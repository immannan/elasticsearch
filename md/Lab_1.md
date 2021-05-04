<img align="right" src="./images/logo.png">


Lab 1. Introducing Elastic Stack
---------------------------------------------



In this lab, we will cover the following topics:


-   What is Elasticsearch, and why use it?
-   A brief history of Elasticsearch and Apache Lucene
-   Elastic Stack components
-   Use cases of Elastic Stack




Downloading and installing
--------------------------------------------



Now that we have enough motivation and reasons to learn about
Elasticsearch and the Elastic Stack, let\'s
start by downloading and installing the key components. Firstly, we will
download and install Elasticsearch and
Kibana. We will install the other components as we need them on the
course of our journey. We also need Kibana because, apart from
visualizations, it also has a UI for developer tools and for interacting
with Elasticsearch.

Starting from Elastic Stack 5.x, all Elastic Stack components are now
released together; they share the same version and are tested for
compatibility with each other. This is also true for Elastic Stack 6.x
components. 

At the time of writing, the current version of Elastic Stack is 7.0.0.
We will use this version for all components.



### Installing Elasticsearch



Elasticsearch can be downloaded as a ZIP,
TAR, DEB, or RPM package. If you are on Ubuntu, Red Hat, or CentOS
Linux, it can be directly installed using `apt` or
`yum`.

We will use the ZIP format as it is the least intrusive and the easiest
for development purposes:


1.  Go to <https://www.elastic.co/downloads/elasticsearch> and download
    the ZIP distribution. You can also download an older version if you
    are looking for an exact version. 
2.  Extract the file and change your directory to the top-level
    extracted folder. Run `bin/elasticsearch` or
    `bin/elasticsearch.bat`.
3.  Run `curl http://localhost:9200` or open the URL in your
    favorite browser.


You should see an output like this:


![](./images/885ab56e-e6d5-4041-8711-abea7fb2d49f.png)


Congratulations! You have just set up a single-node Elasticsearch
cluster.

### Installing Kibana



Kibana is also available in a variety of
packaging formats such as ZIP, TAR.GZ, RMP, and DEB for 32-bit and
64-bit architecture machines: 


1.  Go to <https://www.elastic.co/downloads/kibana> and download the ZIP
    or TAR.GZ distribution for the platform that you are on. 
2.  Extract the file and change your directory to the top-level
    extracted folder. Run `bin/kibana` or
    `bin/kibana.bat`.
3.  Open `http://localhost:5601` in your favorite browser.


Congratulations! You have a working setup of Elasticsearch and Kibana.



Summary
-------------------------



In this lab, we started Elasticsearch and Kibana to begin
the journey of learning about the Elastic Stack.

In the next lab, we will understand the core concepts of
Elasticsearch. We will learn about indexes, types, shards, datatypes,
mappings, and other fundamentals. We will also interact with
Elasticsearch by using **Create**, **Read**,
**Update,** and **Delete** (CRUD)
operations, and learn the basics of searching.
