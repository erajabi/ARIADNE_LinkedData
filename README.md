ARIADNE_LinkedData
==================

Welcome to the ARIADNE_LinkedData wiki!

ARIADNE has created a standards-based technology infrastructure that allows the publication and management of digital learning resources in an open and scalable way. The vision that drives the continuous development of this infrastructure is to provide flexible, effective and efficient access to large-scale educational collections in a way that goes beyond what typical search engines provide. ARIADNE was initially set up by a network of European stakeholders, expanding now into a global network of member institutions sharing the same vision.

#How we exposed the learning resource materials as Linked Data:
### 1. We converted the ARIADNE metadata to a relational database by a JAVA program

A Java program was written that converts all the XML files into a MySQL database according to the following circumstances: 
* The XML file contains learning object metadata which is based on [IEEE LOM Standard](http://ltsc.ieee.org/wg12/files/LOM_1484_12_1_v1_Final_Draft.pdf). You can find a sample XML file at [here](http://www.erajabi.com/uploads/lom.xml)

* XML files may exist in one or several folders
* The MYSQL schema is available at [here](http://www.erajabi.com/uploads/blankAriadne.sql)

### 2. We created an RDF dump using D2RQ mapping service

The dump-rdf tool uses [D2RQ](http://d2rq.org/) to dump the contents of the whole relational database into a single RDF file. This can be done with or without a D2RQ mapping file. If a mapping file is specified, then the tool will use it to translate the database contents to RDF. If no mapping file is specified, then the tool will use the default mapping of generate-mapping for the translation.

In the case of ARIADNE, we create a mapping file the maps the RDB structure to RDF triples. The mapping file is available at [here](http://www.erajabi.com/uploads/ariadne_keywords.n3). In this mapping file, we exposed the following learning object elements: Title, Language,Keyword, Coverage, Technical.location, Right.cost, and Right.Copyright

We used the following code to create the RDF dump:

`dump-rdf -f N-TRIPLE -b http://localhost:2020/ -m ariadne.n3 > ariadne_dump.nt`

* -f format #
<br>The RDF syntax to use for output. Supported syntaxes are “TURTLE”, “RDF/XML”, “RDF/XML-ABBREV”, “N3”, and “N-TRIPLE” (the default). “N-TRIPLE” works best for large databases.
* -b baseURI #
<br>The base URI for turning relative URIs and URI patterns into absolute URIs.
* -o outfile #
<br>Name of the destination file. Defaults to standard output.
* --verbose #
<br>Print extra progress log information.

### 3. We imported the RDF dump into a triple store (Jena Fuseki) by making use of TDBloader command

Fuseki is a SPARQL server. It provides REST-style SPARQL HTTP Update, SPARQL Query, and SPARQL Update using the SPARQL protocol over HTTP. <br>
TDB is also a component of Jena for RDF storage and query. It support the full range of Jena APIs. TDB can be used as a high performance RDF store on a single machine. This documentation describes the latest version, unless otherwise noted.

Now we are using [apache-jena-2.11.1](http://jena.apache.org/download/index.cgi), which includes a "bat" folder and TDB commands are available at there. We also used jena-fuseki-1.0.1-distribution as our triple store manager.

First we loaded the RDF dump into a triple store by making use of tdbloader command: 

`tdbloader --loc=DB ariadne_dump.nt`

Remember to make bin folder in Jena executable and set the PATH to bin folder. 

* DB is an existing TDB database folder. Create an empty one if it does not exist.
* The second parameter is the RDF dump

Then we ran [Jena Fuseki ](http://jena.apache.org/documentation/serving_data/) by the following command at top of the our triple store:

`fuseki-server --loc=DB /ds` 

For more information about configuring Jena Fuseki you can read [here ](http://jena.apache.org/documentation/serving_data/)

Some notes for developing:

- If you are using a Linux system, you can redirect your website to fuseki page (at 3030) by installing [nginx] (http://nginx.org/)
- For redirect, you have to change the config file at /etc/ngnix/ngnix.conf , the following lines:
`http{
    ...
   server {
	  listen 80 default;
	  location / {
		 proxy_pass http://IPaddress:3030;
	  }
	}   
  ....
}`
- Then reload the ngnix : /etc/init.d/ngnix reload
