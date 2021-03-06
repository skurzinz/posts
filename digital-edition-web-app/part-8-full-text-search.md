# Introduction and requirements

In the [last part](../part-7-index-based-search) of this series of tutorials we implemented a basic index based search functionality. The code we build so far can be downloaded [here](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/part-7/thun-demo-0.1.xar). In this part now, we are going to build a full text search which will allow users to 

1. search the content (the text) of all of our XML/TEI documents, 

2. inspect the results in a KWIC (**K**ey**W**ords **I**n **C**ontext) preview, 

3. and jump to the HTML representation of the documents which are matching the search query. 

Therefore we need to have to:

1. Create and configure a full text index,

2. build a new html site containing a search form and which will return the (KWIC) search results,

3. Write an xquery function which takes the search string as a parameter and returns the results.

# Create and configure a full text index

Since the nitty gritty details of creating and configuring a (full text) index are described broadly in the [eXist-db documentation](http://exist-db.org/exist/apps/doc/lucene.xml), I won't spend much time on explaining the next steps described below. Only one things should be stressed. For creating a full text index, eXist-db uses 
> Apache Lucene a high-performance, full-featured text search engine library written entirely in Java
(see [https://lucene.apache.org/core/](https://lucene.apache.org/core/)) which is currently practically the tool of choice when it comes to indexing, searching and information retrieval. 

To create and configure a full text index for our XML/TEI documents stored in **data/editions/** we have to rebuild (parts of) the structure of our application below eXist-db’s **db/system/** directory. Or, to be more precise, we have create an XML document named **collection.xconf** and store under `db/system/path/to/collection/you/would/like/to/index`. So to index all our documents stored in `db/apps/thun-demo/data/editions/` we have to create a collection.xconf document at: `/db/system/config/db/apps/thun-demo/data/editions/collection.xconf`.

Doing so in eXide looks like on the screenshot below:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_0.jpg)

The content of this new created collection.xconf documents only contains a few lines:

**/db/system/config/db/apps/thun-demo/data/editions/collection.xconf:**

```xml
<collection xmlns="http://exist-db.org/collection-config/1.0">
    <index xmlns:tei="http://www.tei-c.org/ns/1.0" xmlns:xs="http://www.w3.org/2001/XMLSchema">
        <fulltext default="none" attributes="false"/>
        <lucene>
            <text qname="tei:p"/>
        </lucene>
    </index>
</collection>
```

Important to know is, that we have now indexed the text of all TEI `<p>` elements, including the text in child nodes of `<p>`. But this is a very basic index configuration which contains no explicit rules on how to deal e.g. with tokenizing (splitting the text into small units we would like to call words) or inline elements (e.g. `<p> This is a <strong>b</trong>old capital letter</p>` which would index a ‘word’ ‘b’ and ‘old’ but not ‘bold’). Maybe we will pick this topic up in a later tutorial. 

## Trigger indexing

After we stored this document in the right place, we have to tell eXist-db to start the indexing routine. To do so, open the Collections tab from the [dashboard](http://localhost:8080/exist/apps/dashboard/index.html) browse to the collection you configured an index for (db/apps/thun-demo/data/editions/) and click on the ‘Reindex collection’ icon. Depending on the size and the amount of documents stored in this collection, this may take a while.![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_1.jpg) 

To check if everything worked and eXist-db really created an index on the documents stored in **db/apps/thun-demo/data/editions/** click on the the [dashboard](http://localhost:8080/exist/apps/dashboard/index.html) on the ‘Monitoring and Profiling for eXist’ (monex) or browse to  [http://localhost:8080/exist/apps/monex/index.html](http://localhost:8080/exist/apps/monex/index.html). (It might be, that this eXist-db package is not installed by default. If so, then you have to install it via the ‘Package Manger’.) After you opened monex, click on ‘Indexes’ (or browse to[http://localhost:8080/exist/apps/monex/indexes.html](http://localhost:8080/exist/apps/monex/indexes.html)) where you should see a list of all indexed collections, where you should see an entry ‘/db/apps/thun-demo/data/editions’

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_2.jpg)

 You can now follow this link to inspect your index in detail. 

## Update the index

Once the index is configured and created, it will update automatically whenever someone changes or adds a document to the indexed collection. Only when you change something in the index configuration (`collection.xconf`) you will have to reindex the collection as described above.

## Deploy an Index

Be aware that whenever you want to deploy - and the (re)install your application as eXist-db package, your index and index configuration will be lost. Although there are solutions and methods on how to set up a new index automatically whenever you install an application like one described in the [eXist-db book](http://shop.oreilly.com/product/0636920026525.do) by Retter and Siegl (p. 235) I would opt for slightly less elaborate workflow for deploying an index. 

For a very simple solution just copy&paste the **/db/system/config/db/apps/thun-demo/data/editions/collection.xconf** document to e.g. the application’s data collection. Whenever you want to deploy your application by first packing it to a .xar package and installing it via eXist-db’s Packer Manager, the only thing you should do than is to copy&paste the collection.xconf to the applications system-path which for our example app is: 

**/db/system/config/db/apps/thun-demo/data/editions/collection.xconf.** After this, the only thing left to do is to trigger the indexing as described above. 

# Implement full text search

## ft_search.html

Now we need to provide some simple search form which will take the search string, pass it on to some XQuery script (see next section) and return the results to the users. Therefore, we create a html document **/pages/ft_search.html** and add the following lines:

```html
<div class="templates:surround?with=templates/page.html&amp;at=content">
    <div>
        <form method="get" class="form-inline" id="pageform">
            <div class="form-group">
                <div class="input-group">
                    <input type="text" class="form-control" name="searchexpr"/>
                </div>
                <button type="submit" class="btn btn-primary">Go!</button>
            </div>
        </form>
    </div>
    <hr/>
    <table>
        <thead>
            <tr>
                <th>score</th>
                <th>KWIC</th>
                <th>Link</th>
            </tr>
        </thead>
        <tbody>
            <tr data-template="app:ft_search"/>
        </tbody>
    </table>
</div>
```
Then we add a link to this page in the base tempate **/templates/pages.html:**

```html
...
<li>
    <a href="persons.html">Persons</a>
</li>
<li>
    <a href="ft_search.html">Fulltext Search</a>
</li>
...
```

But of course, when we now want to browse to this page, we will receive an error message complaining about the fact the there is no function called **app:ft_search** in **modules/app.xql**.

## app:ft_search

In **/modules/app.xql** create a new function called app:ft_search. This function has to parse the search string handed, build a full text search query and return the results to ft_search.html in a nice and KWIC way. This all can be achieved with the following few lines of code:

```xquery
declare function app:ft_search($node as node(), $model as map (*)) {
    if (request:get-parameter("searchexpr", "") !="") then
    let $searchterm as xs:string:= request:get-parameter("searchexpr", "")
    for $hit in collection(concat($config:app-root, '/data/'))//tei:p[ft:query(.,$searchterm)]
    let $document := document-uri(root($hit))
    let $score as xs:float := ft:score($hit)
    order by $score descending
    return
    <tr>
        <td>{$score}</td>
        <td>{kwic:summarize($hit, <config width="40" link="{$document}" />)}</td>
        <td>{$document}</td>
    </tr>
 else
    <div>Nothing to search for</div>
 };
```

But trying to browse to [http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html](http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html) now will still result in an error message, but this time complaining about missing namespace definition. This is because we are using eXist-db’s XQuery module [kwic](http://exist-db.org/exist/apps/doc/kwic.xml) but without importing it first. To do so, we have to add the following line below the import statement for the config module: 

```xquery
import module namespace config="http://www.digital-archiv.at/ns/thun-demo/config" at "config.xqm";
import module namespace kwic = "http://exist-db.org/xquery/kwic" at "resource:org/exist/xquery/lib/kwic.xql";
```

Finally we should now be able to browse to [ft_search.html](http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html) and see the following screen:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_3.jpg)

No go ahead and search something for example: "kirche" (german for church, don’t know, why this word came to my mind).

 [http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html?searchexpr=kirche](http://localhost:8080/exist/apps/thun-demo/pages/ft_search.html?searchexpr=kirche) 

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_4.jpg)

## Hit Count

Usually a search result page like the one we are building here also presents a number of matches or hits returned by the search query. Basically there are two ways of fetching and presenting this information. One could calculate the number of hits on the back end with the help of XQuery and display this number to the user using eXist-db's template functions. I guess this is considered to be the right way.

On the other hand our existing search function is already providing all information needed to display the number of hits to the user. Because currently it renders each hit in an HTML `<p/>` element, grouped by the document in which the search term was found (rendered as HTML `tr` elements.) So if you want to know how many hits the query returned with, just go on and count all paragraphs of the KWIC-column in the results table. Of course this is not exactly a user friendly solution. Especially if have the power of computers at disposal and everybody knows that computers are quite good in counting things.

To avoid writing some code which says, please, count all `<p>` elements in each second `<td>` in each `<tr>` element in the first `<table>` we will add a class attribute to the `<td>` element containing the KWIC-result. 
In **modules/app.xql** in the function **app:ft_search** please change this line:

```xquery
<td>{kwic:summarize($hit, <config width="40" link="{$document}" />)}</td>
```

into 

```xquery
<td class="KWIC">{kwic:summarize($hit, <config width="40" link="{$document}" />)}</td>
```

Now we can use a jQuery statement `var hits = ($('td.KWIC').children('p').length);` to count all `<p>` elements which are children of `<td class="KWIC">` elements and store this number in a `hits` variable. 

The next step is to display this value. Therefore we add a `<h1><span id="hitcount"></span> Hits</h1>` to our **pages/ft_search.html** document which will be populated when the page has finished loading with the help of the following little jQuery function: 

```javascript
<script>
$( document ).ready(function() {
    var hits = ($('td.KWIC').children('p').length);
    $("#hitcount").text(hits);
});
</script>
```

When we now search for e.g. 'kirche', we will see the number of hits.

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_5.jpg)

### Hits and Hit

The perfectionists among you might worry about searches returning only one match because of the hard coded "Hit**s**". The easy way out would be to replace "Hits" with something like "Hit(s)". The nicer way of course is to change our code into something like this (and remove "Hits" from the `<h1>`element.)

```javascript
<script>
    $( document ).ready(function() {
        var hits = ($('td.KWIC').children('p').length);
        if (hits == 1){
            $("#hitcount").text(hits+" Hit");
        }
        else {
            $("#hitcount").text(hits+" Hits");
        }
    });     
</script>
```

Searching now for e.g. 'Schreckensruf' returns "1 Hit":

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_6.jpg)

## What did I search for

The last thing (for now) we could do to improve the usability of `ft_search.html` is to display the actual search term. Like the hit count this could be done on the front end as well on the back end. Or, to use different words, on the client or on the server side. Again we will choose the front end. Which means that the users browser will have to do all the work and not the server which hosts the application. 
The task we try to accomplish is quite similar to the task before. First we have to fetch the actual search term from somewhere, store it to a variable and then add the variables value to an element of our choice. So let's start with fetching the search term. Since we are using a GET-request to submit the search term to the server, the serarch term is exposed in the URL as value of the ?searchexpr=` parameter. The only thing we have to do is to write a function which can parse URL params. Since this is a quit common issue, a *google-first* approach spares us from thinking coming up with [this](https://www.sitepoint.com/url-parameters-jquery/) solution amongst others. Go on and include this function and adapt your existing code so it looks like the following snippet in **pages/ft_search.html**:

```javascript
<script>
        $( document ).ready(function() {
        
            $.urlParam = function(name){
                var results = new RegExp('[\?&amp;]' + name + '=([^&amp;]*)').exec(window.location.href);
                if (results==null){
                    return null;
                }
                else{
                    return results[1] || 0;
                }
            }
            
            var hits = ($('td.KWIC').children('p').length);
            if (hits == 1){
                $("#hitcount").text(hits+" Hit ");
            }
            else {
                $("#hitcount").text(hits+" Hits ");
            }
            $("#searchexpr").text(decodeURIComponent($.urlParam("searchexpr")));
        });     
</script>
```

As you can see we are we adding the value of the fetched URL param to an HTML element with the `id="searchexpr"`. Such an element could be a child element of the pages `<h1>` headline which we could modify into something shown below:

```html
<h1>
    <span id="hitcount"/> for <strong><span id="searchexpr"/></strong>
</h1>
```

When we now search for e.g. 'kirche' our result page should like this:

![image alt text](https://raw.githubusercontent.com/csae8092/posts/master/digital-edition-web-app/images/part-8/image_7.jpg)

# Conclusion and outlook

Now we are (almost) done. There is just one thing left to adjust. As you can see, the Keyword in its context is rendered as a link. But clicking on it produces (for now) only a 404 (page not found) error. 

To fix this, we could (once more) copy and paste parts of the code used for the index based search results view. But since this would be now the third time we copy paste very similar functionalities, we should do something more advanced like writing a reusable function. But this we will engage in the [next part](../part-9-code-refactoring). 

