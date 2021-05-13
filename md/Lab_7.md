
<img align="right" src="./images/logo.png">



Lab 7. Visualizing Data with Kibana
------------------------------------------------


Kibana is an open source
web-based analytics and visualization tool that lets you visualize the
data stored in Elasticsearch using a variety of tables, maps, and
charts. Using its simple interface, users can easily explore large
volumes of data stored in Elasticsearch and perform advanced analysis of
data in real time. In this lab, let\'s explore the various
components of Kibana and explore how you can use it for data analysis.

We will cover the following topics in this lab:


-   Downloading and installing Kibana
-   Preparing data
-   Kibana UI
-   Timelion
-   Using plugins





### Running Kibana


Switch to `elasticsearch` user and type `kibana` to start the service if not running already.

```
$  su elasticsearch
$  kibana
```



Kibana is a web application and, unlike Elasticsearch and Logstash,
which run on the JVM, Kibana is powered by Node.js. During bootup,
Kibana tries to connect to Elasticsearch running on
`http://localhost:9200`. Kibana is started on the default port
`5601`. Kibana can be accessed from a web browser using the
`http://localhost:5601` URL. You can navigate to the
`http://localhost:5601/status` URL to find the Kibana server
status.

The status page displays information about
the server\'s resource usage and lists the installed plugins, as shown in the following screenshot:


![](./images/adc252fb-443b-4999-8123-851bc67ab164.png)



### Configuring Kibana


When Kibana was started, it started on port `5601`, and it
tried to connect to Elasticsearch running on
port `9200`. What if we want to change some of these settings?
All the configurations of Kibana are stored in a file called
`kibana.yml`, which is present under
the `config` folder, under `$KIBANA_HOME`.



Preparing data
--------------------------------



When you launch Kibana, it comes with predefined options to enable
loading of data to Elasticsearch with a few clicks and you can start
exploring Kibana right away. When you launch Kibana by accessing
the `http://localhost:5601`link in your browser for the first
time, you will see the following screen:


![](./images/32e487fd-8950-47e9-9437-d8d32047d247.png)


Clicking on the `Try our sample data` button will take you to the following screen:


![](./images/437cd832-47ab-4351-bb0a-515115ff3740.jpg)


#### ProTip:


You can search Kibana features from the search menu:

![](./images/s3.png)


Go ahead and click on `Add data` for `Sample eCommerce orders`.
It should load data, visualizations, and dashboards in the background.
Once ready, you can click on `View data`, which will take you to the
eCommerce dashboard: 


![](./images/38890879-9c3c-4510-b23b-6245864e4cfb.jpg)


The following screenshot shows the dashboard:


![](./images/6c394399-9207-45bb-9e1a-cd808b32e021.png)


The actual data that is powering these dashboards/visualizations can be
verified in Elasticsearch by executing the following command. As seen,
the sample data is loaded into
the `kibana_sample_data_ecommerce` index, which has `4675` docs:

```
curl localhost:9200/_cat/indices/kibana_sample*?v
```


### Note

If you click on the `Remove` button, all the dashboards and data
will be deleted. Similarly, you can click on the `Add data` button
for the other two widgets if you want to explore sample flight and
sample logs data.


If you want to navigate back to the home page, you can always click on
this icon ![](./images/c20e47c9-78f3-4761-ba4d-5c353e3c3d99.png)



### Loading Custom Data

In this lab, rather than relying on the sample default data shipped
out of the box, we will load custom data which we will use to follow the
tutorial. One of the most common use cases is
**log analysis**. For this tutorial, we will be loading
Apache server logs into Elasticsearch using Logstash and will then use
it in Kibana for analysis/building visualizations.

<https://github.com/elastic/elk-index-size-tests> hosts a dump of Apache
server logs that were collected for
the [www.logstash.net](http://www.logstash.net/) site during the
period of May 2014 to June 2014. It contains 300,000 log events.



Create a config file named `apache.conf` in the `$LOGSTASH_HOME/bin` folder, as shown in the following code block:

```
input 
{ 
 file {
 path => ["/home/elasticsearch/Lab07/data/logs"]
 start_position => "beginning"
 sincedb_path => "NUL"
 }
}

filter 
{
 grok {
 match => {
 "message" => "%{COMBINEDAPACHELOG}"
 }
 }
 mutate {
 convert => { "bytes" => "integer" }
 }
 date {
 match => [ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" ]
 locale => en
 remove_field => "timestamp"
 }
 geoip {
 source => "clientip"
 }
 useragent {
 source => "agent"
 target => "useragent"
 }
}

output 
{ 

 stdout { 
 codec => dots
 } 
 elasticsearch { 
 index => "logstash-%{+yyyy-MM-dd}"
  }
}
```

Start Logstash, shown as follows, so that it can begin processing the
logs, and index them to Elasticsearch. Logstash will take a while to
start and then you should see a series of dots (a dot per processed log
line):

```
cd $LOGSTASH_HOME/bin

logstash -f apache.conf
```

<span style="color:red;">Note: Logstash will take few minutes to import the data. Please wait for import to complete</span>


Let\'s verify the total number of documents (log events) indexed into
Elasticsearch:

```
curl -X GET http://localhost:9200/logstash-*/_count
```

In the response, you should see a count of 300,000.



Kibana UI
---------------------------



In this section, let\'s understand how data
exploration and analysis is typically performed and how to create
visualizations and a dashboard to derive insights about data.




### Configuring the index pattern



Open up Kibana from the browser using
the [http://localhost:5601](http://localhost:5601/) URL. In the
landing page, search  `Index pattern` and click it. 

![](./images/s2.png)


Click "Create Index" button and type in `logstash-*` in the `Index pattern` text field and click on the `Next step` button, as shown in the following screenshot:


![](./images/f94ce1a0-921b-4a50-bdb9-b52c51f07cb8.png)


On the `Create Index Pattern` screen, during the configuration of an
index pattern, if the index has a datetime field (that is, it is a
time-series index), the `Time Filter field name` dropdown is visible
and allows the user to select the appropriate datetime field; otherwise,
the field is not visible. As the data that we loaded in the previous
section contains time-series data, in the `Time Filter field name`,
select `@timestamp` and click `Create`, as follows:


![](./images/4a525814-6462-46d7-81f1-f324fa537318.jpg)


Once the index pattern is successfully created, you should see the
following screen:


![](./images/d07d92aa-261e-4f5f-aafe-52d0465fab12.jpg)


### Discover


By default, the `Discover` page displays the events of the last 15
minutes. As the log events are from the period May 2014 to June 2014,
set the appropriate date range in the time filter. Navigate to
`Time Filter` \| `Absolute Time Range` and set `From` as
`2014-05-28 00:00:00.000` and \>`To `to
`2014-07-01 00:00:00.000`. Click `Update`, as shown in the
following screenshot:


![](./images/103d7059-34fc-4dfc-a84c-f98c288be45f.jpg)


The `Discover` page contains the sections shown in the following screenshot:


![](./images/6c56eac0-f9be-474c-a5cb-653db5af3a63.jpg)




**Document Table**: This section shows the actual
    document data. The table shows the **500** most recent
    documents that match the user-entered query/filters, sorted by
    timestamp (if the field exists). By clicking the `Expand` button
    found to the left of the document\'s table entry, data can be
    visualized in table format or JSON format, as follows:



![](./images/814550e4-bd8d-41f4-965c-fb6cde7f011f.jpg)


Added field columns replace the **\_source** column in the
**Documents** table. Field columns in the table can be
shuffled by clicking the right or left arrows found when hovering over
the column name. Similarly, by clicking the remove button,
**x**, columns can be removed from the table, as follows:


![](./images/140dd4ab-6413-4ca6-8ae6-f85e6017f9d1.png)




#### Elasticsearch query string/Lucene query


This provides the ability to perform various
types of queries ranging from simple to complex queries that adhere to
the Lucene query syntax. In the query bar, by default, KQL will be the
query language. Go ahead and disable it as shown in the following
screenshot. Once you disable it, `KQL` changes to `Lucene` in
the query bar, as follows:


![](./images/cdff8cb1-d9e2-488d-aef5-cc0a6a1bcb9e.png)


Let\'s see some examples:

**Free Text search**: To search for text present in any of the fields, simply enter a text string in
the query bar:


![](./images/331852be-8b94-47e9-8539-0bf3f002d56f.png)


If you are doing an exact phrase search, that is, the documents should
contain all the words given the search criteria, and the words should be
in the same order, then surround the phrase with quotes. For
example, `file logstash` or `files logstash`.

**Field search**: To search for values against a specific field, use the `syntax` field: `value`:


![](./images/926cab44-beba-4cb1-9eef-c192046bf3aa.png)


**Boolean search**: You canmakeuse of Boolean operators such as `AND`,
`OR`, and `-` (Must Not match) to build complex
queries. Using Boolean operators, you can combine the
field:`value`and free text as well.

`Must Not` match: The following is a screenshot of a
`Must Not` operator with a field:


![](./images/02fb07a0-81bc-4ed7-8fa5-e2cf7c12389c.png)


The following is an example of a `Must Not` operator with free
text:


![](./images/7d58de53-dd3a-4035-af5f-a40bf4755b67.png)



### Note

There should be no space between the `-` operator and the
search text/field.


**Grouping searches**: When we want to build complex queries, often, we have to group the search
criteria. Grouping both by field and value is supported, as shown in the
following screenshot:


![](./images/074cc007-8d86-4958-9bca-28209bfec1e0.png)


**Range search**: This allows you
to search within a range of values. Inclusive ranges are specified with
square brackets---for example, `[START_VALUE TO END_VALUE]`,
and exclusive ranges with curly brackets---for
example, `{ START _VALUE TO END_VALUE }`. Ranges can be
specified for dates and numeric or string fields, as follows:


![](./images/8647f233-62e5-4261-ad9a-78a4f03570b6.png)



### Note

The `TO` operator is case-sensitive and its range values
should be numeric values.


**Wildcard and Regex search**: By using the `*` and `?` wildcards
with search text, queries can be
executed; `*` denotes zero or more matches and `?`
denotes zero or one match, as shown in the following screenshot:


![](./images/435e9034-e748-47d0-a783-37de1fb00199.png)





#### Elasticsearch DSL query



By using a DSL query, queries can be
performed from the query bar. The query part of a DSL query can be used
to perform searches.

The following screenshot is an example of searching for documents that
have `IE` in the `useragent.name` field and
`Washington` in the `geoip.region_name` field:


![](./images/19377d20-d71f-41eb-bb94-1e1840dfa815.png)


**Hits**: Hits represent the total number of documents that
match the user-entered input query/criteria.



#### KQL

KQL doesn\'t allow spaces between expressions and the same thing would
have to be written as
`response:200 or geoip.city_name:Diedorf`, as shown in the
following screenshot:


![](./images/71bec5de-b944-4e91-86f8-b1020cab024a.png)


Similarly, you can have `and not` expressions too and group
expressions as shown in the following screenshot:


![](./images/b83013de-0b85-4e00-ab34-b49f03364213.png)



### Note

The operators `and`, `or`, and `not` are
case-insensitive.


**Histogram**

![](./images/1558ff78-915c-4db2-8b23-7b40660f174e.png)




**Toolbar**

The user can refer to existing stored searches later and modify the
query, and they can either overwrite the existing search or save it as a
new search (by toggling the `Save as new search` option in the
`Save Search` window), as follows:


![](./images/5ec1c4be-72be-4952-ab88-f553efe9230e.png)


Clicking the **Open** button displays the saved searches, as shown in the following screenshot:


![](./images/91505d6f-dcf4-41ab-9904-0b0ae5ae80a9.png)


In Kibana, the state of the current page/UI is stored in the URL itself,
thus allowing it to be easily shareable. Clicking the
`Share` button allows you to share the
`Saved Search`, as shown in the following screenshot:


![](./images/91a0ee8d-deeb-4f7c-a41b-dc67f74cda24.png)


The **Inspect** button allows to view query statistics such
as total hits, query time, the actual query fired against ES, and the
actual response returned by ES. This would be useful to understand how
the Lucene/KQL query we entered in the query bar translates to an actual
ES query, as shown in the following screenshot:


![](./images/5a13f7d9-533d-42e2-9ebc-c58d3e3a163d.png)


**Time Filter** provides the following options to
select time periods. Click
on `Time Filter` (`calendar icon`)/ **Date fields**
to access the following options:


-   **Quick time filter**: This helps you to filter quickly based on some already available
    time ranges:



![](./images/20ecadb5-ada0-4a90-b1d9-9c04db9c4a1a.png)



-   **Relative time filter**: This helps you to filter based on the relative time with respect to
    the current time. Relative times can be in the past or the future. A
    checkbox is provided to round the time:



![](./images/2d9d5a6f-4026-49a4-af12-7993cd95b657.png)



-   **Absolute time filter**: This helps you to filter based on input start and end times:



![](./images/cd880977-2eac-44ce-b116-9f05221392cf.png)



-   **Auto Refresh**: During the
    analysis of real-time data or data that is continuously generated, a
    feature to automatically fetch the latest data would be very useful.
    Auto Refresh provides such a functionality. By default, the refresh
    interval is turned off. The user can choose the appropriate refresh
    interval that assists their analysis and click
    the **Start** button, as shown in the following
    screenshot:



![](./images/8000bded-368b-4c9c-82f8-9ec60cdc190c.png)




You can add field filters from **Fields list** or
**Documents table**, and even manually add a filter. In
addition to creating positive and negative filters, **Documents table** enables you to determine whether a field is present. 

To add a positive or negative filter, in `Fields List` or
`Documents Table`, click on the positive icon or negative icon
respectively. Similarly, to filter a search according to whether a field
is present, click on the  `*`  icon (the exists
filter), as follows:


![](./images/15f62a44-a808-43a9-b4e1-73d723b5e9ce.png)


You can also add filters manually by clicking the
`Add a Filter` button found below the query bar. Clicking on the
button will launch a popup in which filters can be specified and applied
by clicking the `Save` button, as follows:


![](./images/49e497d6-98c0-4c30-a247-6807f91c1510.png)



The following screenshot displays the preceding actions that can be
applied:


![](./images/f2956a69-5c61-4d0e-8cb0-468d8e6b1c5f.png)


You can perform the preceding actions across multiple filters at once
rather than one at a time by clicking on the filter settings icon, as
follows:


![](./images/cd28aec0-ebcb-4b28-834e-702583fa7390.png)



### Visualize


All visualizations in Kibana are based on the aggregation queries of
Elasticsearch. Aggregations provide the multi-dimensional grouping of
results---for example, finding the top user agents by device and by
country. Kibana provides a variety of visualizations, shown as follows:


![](./images/30b40f8a-8bb3-4781-91a9-63af2327b1e8.png)




### Creating a visualization


The following are the steps to create visualizations:


1.  Navigate to the `Visualize` page and click
    the `Create a new Visualization` button or the `+`  button
2.  Select a visualization type
3.  Select a data source
4.  Build the visualization


The **Visualize** Interface looks as follows:


![](./images/7125bfe0-88de-47df-b600-bf37ba76e403.png)




### Visualizations in action



Let's see how different visualizations can
help us in doing the following:


-   Analyzing response codes over time
-   Finding the top 10 requested URLs
-   Analyzing the bandwidth usage of the top five countries over time
-   Finding the most used user agent



### Note

As the log events are from the period May 2014 to June 2014, set the
appropriate date range in the time filter. Navigate to **Time
Filter** \| **Absolute Time Range** and set
**From** as `2014-05-28 00:00:00.000` and
**To** to `2014-07-01 00:00:00.000`; click
`Go`.


#### Response codes over time



This can be visualized easily using a bar
graph.

Create a new visualization:


1.  Click on `New` and select `Vertical Bar`
2.  Select `Logstash-*` under `From a New Search`,
    `Select Index`
3.  On the [*x*]  axis, select `Date Histogram` and
    `@timestamp` as the field
4.  Click `Add sub-buckets` and select `Split Series`
5.  Select **Terms** as the `Sub Aggregation`
6.  Select **response.keyword** as the field
7.  Click the `Play` (Apply Changes) button


The following screenshot displays the steps to create a new
visualization for response codes over time:


![](./images/3d4287eb-8326-4126-843b-6d60869fb354.png)


Save the visualization as`Response Codes By Time`.

As seen in the visualization, on a few days, such as June 9, June 16,
and so on, there is a significant amount of **404**. Now, to
analyze just the **404** events, from the
**labels**/**keys** panel, click on `404 `and
then click `positive filter`:


![](./images/db6f026b-3e25-4a00-bab3-002497d85b64.png)


The resulting graph is shown in the following
screenshot:


![](./images/f6283fe3-8b84-43a8-a734-95e3074603b7.png)



### Note

You can expand the labels/keys and choose the colors from the color
palette, thus changing the colors in the visualization. Pin the filter
and navigate to the `Discover` page to see the requests resulting in
**404**s.

 


#### Top 10 requested URLs



This can be visualized easily using a data
table.

The steps are as follows:


1.  Create a new visualization
2.  Click on `New` and select `Data Table`
3.  Select `Logstash-*` under
    `From a New Search, Select Index`
4.  Select **Buckets** type as `Split Rows`
5.  Select `Aggregation` as **Terms**



6.  Select the **request.keyword** field
7.  Set the `Size` to `10`
8.  Click the `Play` (Apply Changes) button


The following screenshot displays the steps to create a new
visualization for the top 10 requested URLs:


![](./images/e4fbe9aa-c942-46b0-b690-26612dddd0dd.png)


Save the visualization as`Top 10 URLs`.


### Note

`Custom Label` fields can be used to provide meaningful names
for aggregated results. Most of the visualizations support custom
labels. Data table visualizations can be exported as a `.csv`
file by clicking the `Raw` or `Formatted` links found under the
data table visualization.


#### Bandwidth usage of the top five countries over time



The steps to demonstrate this are as follows:


1.  Create a new visualization
2.  Click on `New` and select `Area Chart`
3.  Select `Logstash-*` under
    `From a New Search, Select Index`
4.  In `Y axis`, select **Aggregation** type and **Sum
    of bytes** as the field
5.  In `X axis`, select `Date Histogram` and `@timestamp` as
    the field
6.  Click `Add sub-buckets` and select `Split Series`
7.  Select **Terms** as the `Sub Aggregation`
8.  Select **geoip.country\_name.keyword** as the field
9.  Click the `Play` (Apply Changes) button


The following screenshot displays the steps to create a new
visualization for the bandwidth usage of the top five countries over
time:


![](./images/4753145b-bb3f-4966-8254-59a906dccce4.png)


Save the visualization
as `Top 5 Countries by Bandwidth Usage`.

What if we were not interested in finding only the top five countries?
Rearrange the aggregation and click `Play`, as follows:


![](./images/a1163f50-2eb1-4ea8-8925-4112cf35394e.png)



### Note

The order of aggregation is important.


#### Web traffic originating from different countries



This can be visualized easily using a
coordinate map.

The steps are as follows:


1.  Create a new visualization
2.  Click on `New` and select `Coordinate Map`
3.  Select `logstash-*` under
    `From a New Search, Select Index`
4.  Set the bucket type as` Geo Coordinates`
5.  Select the **Aggregation** as **Geohash**
6.  Select the **geoip.location** field
7.  In the **Options** tab, select `Map Type` as
    `Heatmap`
8.  Click the `Play` (`Apply Changes`) button:



![](./images/2dcd1806-abc7-4ed6-8439-6a8ac6d2bfe2.png)


Save the visualization as `Traffic By Country`.

Based on this visualization, most of the traffic is originating from
California.

For the same visualization, if the metric is changed to `bytes`, the
resulting visualization is as follows:


![](./images/38944859-bc3e-49a5-8ee2-38d15d3a95cd.png)



### Note

You can click on the `+/-` button found at the top-left of the map
and zoom in/zoom out. Using the `Draw Rectangle` button found at the
top-left, below the zoom in and zoom out buttons, you can draw a region
for filtering the documents. Then, you can pin the filter and navigate
to the `Discover` page to see the documents belonging to that
region.


#### Most used user agent



This can be visualized easily using a variety
of charts. Let\'s use **Tag Cloud**.

The steps are as follows:


1.  Create a new visualization
2.  Click on `New` and select `Tag Cloud`
3.  Select `logstash-*` under `From a New Search, Select Index`
4.  Set the bucket type to **Tags**
5.  Select the **Terms** aggregation
6.  Select the **useragent.name.keyword** field
7.  Set the `Size` to `10` and click
    the `Play` (`Apply Changes`) button:



![](./images/fc7da7e3-06a6-424f-b479-db03874baf76.png)


Save the visualization as`Most used user agent`. Chrome,
followed by Firefox, is the user agent the majority of traffic is
originating from.


### Dashboards



**Dashboards** help you bring
different visualizations into a single page. By using previously stored
visualizations and saved queries, you can
build a dashboard that tells a story about the data.

A sample dashboard would look like the following screenshot:


![](./images/3d31c4e7-1ba4-44bf-a62f-f2423071ccb3.png)


Let\'s see how we can build a dashboard for our log analysis use case.



#### Creating a dashboard



In order to create a new dashboard, navigate
to the `Dashboard` page and click the `Create a Dashboard`
button:


![](./images/d0716f25-4915-43dc-8ad1-4a9afc0eb3bb.png)


On the resulting page, the user can click the `Add` button, which
shows all the stored visualizations and saved searches that are
available to be added. Clicking on
`Saved search`/`Visualization` will result in them getting added
to the page, as follows:


![](./images/391fb56c-4a8e-430a-a27d-30a8dcebff41.png)


The user can expand, edit, rearrange, or remove visualizations using the buttons available at the top corner
of each visualization, as follows:


![](./images/c50d115c-dc70-4418-b6b3-36e71a070988.png)



### Note

By using the query bar, field filters, and time filters, search results
can be filtered. The dashboard reflects those changes via the changes to
the embedded visualizations. For example, you might be only interested
in knowing the top user agents and top devices by country when the
response code is **404**. Usage of the query bar, field
filters, and time filters is explained in the [*Discover*] 
section.


#### Saving the dashboard 



Once the required visualizations are added to
the dashboard, make sure to save the dashboard by clicking the
`Save` button available on the toolbar and provide a title. When a
dashboard is saved, all the query criteria and filters get saved, too.
If you want to save the time filters, then, while saving the dashboard,
select the `Store time with dashboard` toggle button. Saving the
time along with the dashboard might be useful when you want to
share/reopen the dashboard in its current state, as follows:


![](./images/eb4ecb22-cbca-4d89-ac5c-32e90e0a541b.png)


#### Cloning the dashboard



Using the **Clone** feature, you can copy the current dashboard, along with its queries and
filters, and create a new dashboard. For example, you might want to
create new dashboards for continents or countries, as follows:


![](./images/116010c5-a259-4289-9933-7ec646a3d1f1.png)



### Note

The dashboard background theme can be changed from light to dark. When
you click the `Edit` button in the toolbar, it provides a button
called `Options`, which provides the feature to change the dashboard
theme.


#### Sharing the dashboard 



Using the **Share** feature, you can either share a direct link to a Kibana dashboard with
another user or embed the dashboard in a web page as aniframe:

![](./images/12.PNG)




### Timelion 


`Timelion` is available just like any other
visualization in the **New Visualization** window, as follows:


![](./images/ddb108d6-691c-4480-bea2-712c4c8a49b5.png)


The main components/features of Timelion visualization
are `Timelion expressions,` which allow you to define expressions
that influence the generation of graphs. They allow you to define
multiple expressions separated by commas, and also allow you to chain
functions.

### Timelion expressions



The simplest Timelion expression used for generating graphs is as follows:

```
.es(*) 
```


If you\'d like to restrict Timelion to data within a specific index (for
example, `logstash-*`), you can specify the index within the
function as follows: 

```
.es(index=logstash-*) 
```

As Timelion is a time-series visualizer, it uses
the `@timestamp` field present in the index as the time field
for plotting the values on an [*x *] axis. You can change it
by passing the appropriate time field as a value to
the `timefield` parameter.

Timelion\'s helpful autocompletion feature will help you build the
expression as you go along, as follows:


![](./images/2a3ee06d-6f55-49a1-b7a6-3ad9998f93b7.png)


Let\'s see some examples in action to understand Timelion better. 


### Note

As the log events are from the period May 2014 to June 2014, set the
appropriate date range in the time filter. Navigate to `Time Filter`
\| `Absolute Time Range` and set `From` to
`2014-05-28 00:00:00.000` and `To` to
`2014-07-01 00:00:00.000`; click `Go.`


Let\'s find the average bytes usage over time for the US. The expression
for this would be as follows:

```
.es(q='geoip.country_code3:US',metric='avg:bytes')
```

The output is displayed in the following screenshot:


![](./images/65ae185d-3ac7-4ffe-9b17-ca22f0966831.png)


Timelion allows for the plotting of multiple graphs in the same chart as
well. By separating expressions with commas, you can plot multiple
graphs.

Let\'s find the average bytes usage over time for the US and the average
bytes usage over time for China. The expression for this would be as
follows:

```
 es(q='geoip.country_code3:US',metric='avg:bytes'), .es(q='geoip.country_code3:CN',metric='avg:bytes')
```

The output is displayed in the following
screenshot:


![](./images/a3fe5738-b852-4ca7-a147-d930b8fad305.png)


Timelion also allows for the chaining of functions. Let\'s change the
label and color of the preceding graphs. The expression for this would
be as follows:

```
.es(q='geoip.country_code3:US',metric='avg:bytes').label('United States').color('yellow'), .es(q='geoip.country_code3:CN',metric='avg:bytes').label('China').color('red')
```

The output is displayed in the following
screenshot:


![](./images/40946626-761e-41b3-833e-f7c159ca429c.png)


One more useful option in Timelion is using offsets to analyze old data.
This is useful for comparing current trends to earlier patterns. Let\'s
compare the sum of bytes usage to the previous week for the US. The
expression for this would be as follows:

```
.es(q='geoip.country_code3:US',metric='sum:bytes').label('Current Week'), .es(q='geoip.country_code3:US',metric='sum:bytes', offset=-1w).label('Previous Week')
```

The output is displayed in the following screenshot:


![](./images/b68cb6d4-dec8-4365-9d98-041e21ab9137.png)


Timelion also supports the pulling ofdatafrom
external data sources using a public API. Timelion has a native API for
pulling data from the World Bank, Quandl, and Graphite.




Using plugins
-------------------------------



**Plugins** are a way to enhance the
functionality of Kibana. All the plugins that
are installed will be placed in the `$KIBANA_HOME/plugins`
folder. Elastic, the company behind Kibana, provides many plugins that
can be installed, and there are quite a number of public plugins that
are not maintained by Elastic that can be installed, too.



### Installing plugins



Navigate to `KIBANA_HOME` and execute
the `install` command, as shown in the following code, to
install any plugins. During installation, either the name of the plugin can be given (if it\'s hosted by Elastic
itself), or the URL of the location where the plugin is hosted can be
given:

```
$ KIBANA_HOME>bin/kibana-plugin install <package name or URL>
```

For example, to install `x-pack`, a plugin developed and
maintained by Elastic, execute the following command:

```
$ KIBANA_HOME>bin/kibana-plugin install x-pack
```

To install a public plugin, for example, LogTrail
(<https://github.com/sivasamyk/logtrail>), execute the following
command:

```
$ KIBANA_HOME>bin/kibana-plugin install https://github.com/sivasamyk/logtrail/releases/download/v0.1.31/logtrail-6.7.1-0.1.31.zip 
```




### Removing plugins



To remove a plugin, navigate to `KIBANA_HOME` and execute the
`remove` command followed by the
plugin name:

```
$ KIBANA_HOME>bin/kibana-plugin remove x-pack
```



Summary
-------------------------

In this lab, we covered how to effectively use Kibana to build
beautiful dashboards for effective storytelling about your data.

We learned how to configure Kibana to visualize data from Elasticsearch.
We also looked at how to add custom plugins to Kibana.
