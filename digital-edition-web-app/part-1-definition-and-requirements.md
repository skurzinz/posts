# A definition of »digital edition«


 The following series of blog posts will guide you step by step through the process of building a web application for digital editions. Such a web application makes digital editions available over the world wide web. A digital edition in the context of this blog posts is understood as a well structured and (therefore) machine readable representation (or model) of basically any kind of text. The language of choice which is used to model the text is XML and this XML should preferably validate against the rules or recommendations set by the Text Encoding Initiative ([TEI](http://www.tei-c.org/)). In general, however, our digital edition web app should be able to handle every well formed XML documents. 

# Web app features

The features of our web application are very straightforward. 

* The users should be able to **browse the content of our digital edition**. This means we have to expose all our XML documents in some kind of table of content of list view. 

* From such a central index page the user should be able to click on a document of their interest to **see the full document**. In a more technical way, a click on e.g. the name of a document should trigger a script, which **transforms the XML document via XSLT to a nicely looking HTML document**. 

* Our most minimalistic web application should also provide a **full text search** over all documents in the edition. 

* And since we are big friends of open access and open source, we also want to give access to our XML documents in a machine readable manner via an application programming interface (**API**). 

# Software and Data requirements

To build such a web application, all we need is 1) the open source xml database [eXist-db](http://exist-db.org/) and 2) a good xml editor. The second is not essential, but since we are dealing a lot with XML, XSL, XQuery and (X)HTML, an xml editor becomes handy.  I will use the [oXygen XML Editor](https://www.oxygenxml.com/) as this one connects smoothly with eXist-db and is packed with features you do not want to miss, when you are planning to do some serious work with XML. Be aware that Oxygen is not for free but you can run a trial version for about 30 days. If you don't want or can't use Oxygen, then eXist-db´s build-in web based xml editor [eXide](http://exist-db.org/exist/apps/eXide/index.html) is also quite sufficient.  

As mentioned in the section above, we will also need some XML/TEI documents to play with. Since I have to rebuild an old application of mine anyway, I will use the documents of the online edition of the correspondence of [Leo von Thun Hohenstein](http://thun-korrespondenz.uibk.ac.at:8080/exist/apps/Thun-Collection/index.html). You can download some or all files using the applications [REST-API](http://thun-korrespondenz.uibk.ac.at:8080/exist/rest/db/files/thun/xml), but for the sake of convenience you can find a zip file containing some documents [here](https://github.com/csae8092/posts/raw/master/digital-edition-web-app/downloads/editions.zip). 

But now it is time to [get started](../part-2-getting-started).

## Disclaimer

For the time being, this tutorial focuses on a windows-based environment. 
