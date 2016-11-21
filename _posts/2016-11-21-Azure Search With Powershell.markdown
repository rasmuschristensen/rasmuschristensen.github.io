---
layout: post
title:  "Azuer Search With Powershell"
description:  "How to use Azure Search from Powershell"
date:   2016-11-21
---

<p class="intro">
<span class="dropcap">A</span>zure Search is one of the newer kids in the Azure growing family. You can like any other Azure item, interact with it directly from the Azure portal. As you need to do frequent changes in the start of a project or just want to be able to automatically setup your datasource, index and indexes, the portal becomes less usefull. Luckily Azure Search provides a REST API we can easily interact with. 
</p>

### A basic Azure Search setup
The scope for this post will be a setup where data from an Azure hosted SQL Database will be exposed by an Index in Azure Search. To do this we need 3 parts, a __datasource__, an __index__ and an __indexer__. The datasource is our SQL database, the index contains the data to expose and the indexer is the "scheduler" to add data to the index.
As mentioned Azure Search provides a [REST API](https://docs.microsoft.com/en-us/rest/api/searchservice/?redirectedfrom=MSDN). To use the API you need to provide an __admin__ key, not a __query__ key, this key will be part of each request to authenticate.

### First step, Create A datasource
The first thing we need is a datasource. Here we use Azure Sql Server and one thing you need to think about is how you want to detect changes. Azure Search support a soft-delete approach and "Sql Server Integrated Change Tracking policy". We'll use the latter to have items removed from the index, when they are removed from the SQL Server. You need to ensure this is enabled on the SQL Server __BEFORE__ you create the datasource in Azure Search.

{% highlight powershell %}
$dbBody = @'
{
    "name": 'peopleDb',
    "description": 'people index db',
    "type": 'azuresql',
    "credentials": {"connectionString":'YOUR SQL Connection'},
    "container": {name:'Employees'},
    "dataChangeDetectionPolicy":{ "@odata.type": '#Microsoft.Azure.Search.SqlIntegratedChangeTrackingPolicy'}   
}
'@

$url = "https://company.search.windows.net/datasources?api-version=2015-02-28"

$headers = @{ "api-key" = "YOUR AZURE SEARCH API KEY"}

& Invoke-WebRequest  -Method POST -Uri $url -Body $dbBody -ContentType "application/json" -Headers $headers

{% endhighlight %}

Starting from the top, we first define the body of the request. it contains:

* __name__ the database name.
* __description__ A textual description of the index.
* __type__ database type.
* __creadentials__ the connectionstring to the database.
* __container__ the table we want to index.
* __dataChangeDetectionPolicy__ how we want to handle updates.

Next we define the __URL__, where ___company___ is the name of our Azure Search endpoint. The keyword in the url is __datasources__ and also important is the __api-version__. The value is taken directly from the Azure documentation, it might change in the future.

As mentioned we need an API key for all requests. This must be included as a header in the request, see the __$headers__ in the script above.

Finally we execute the request as __POST__ and define the ContentType as __application/json__ 


### Second step, Create the Index
The index we create will make use of the __container__ we created above. We need to define which fields in the database should be retrieved, searchable, filterable and facetable by the index and also which field is the __Key__. We kan make the same set as before this time, we just target the __index__ of the REST API.

{% highlight powershell %}

$body = @'
{
    "name":'employeeIdx',  
    "fields":
    [
        {"name":'Id',"type":'Edm.String',"key":'true',"searchable":'false',"sortable":'false',"facetable":'false'},
        {"name":'Firstname',"type":'Edm.String'},
        {"name":'Department',"type":'Edm.String',"filterable":'false',"sortable":'false',"facetable":'false'}
        ]          
}
'@

$url = "https://company.search.windows.net/indexes?api-version=2015-02-28"

& Invoke-WebRequest  -Method Post -Uri $url -Body $body -ContentType "application/json" -Headers $headers

{% endhighlight %}

You might notice that I omitted the api-key field setup. This is because all of this will be executed in the same script in the real world.

### third step, Create the indexer
The third and final step is creating the indexer, a bit name confusing. The indexer is the part of Azure Search which schedules how often our __index__ will be updated from the datasource. 

{% highlight powershell %}

$indexerBody = @'
{
    "name": 'employeeindexer',
    "description": 'indexer for the index employeeIdx. Runs every 5 minute',
    "dataSourceName": 'peopleDb',
    "targetIndexName": 'employeeIdx',
    "schedule": {"interval": 'PT5M', "startTime":'2016-10-11T00:00:00Z'}
    }
'@

$url = "https://company.search.windows.net/indexers?api-version=2015-02-28"

& Invoke-WebRequest  -Method Post -Uri $url -Body $indexerBody -ContentType "application/json" -Headers $headers

{% endhighlight %}

This time the target is __indexers__. Besides specifying the datasource and target index, we also specify the schedule. Here we specify it as __interval__ with a frequency of every 5 min, and a starttime.


### build and tear down your indexes
With the script above you are able to fast and consistenly creating indexes and also persisting the script in your version control. One of the reasons I moved into powershell was because I wanted a faster development cycle and this also means I need to be able to tear down my datasource, index and indexer again to make changed and then create it all over and over.

This part is just as easy as the above, a compact edition looks like the following. 

{% highlight powershell %}

#remove indexer
$url = "https://compNY.search.windows.net/indexers/employeeindexer?api-version=2015-02-28"
& Invoke-WebRequest  -Method DELETE -Uri $url -ContentType "application/json" -Headers $headers

#remove index
$url = "https://company.search.windows.net/indexes/employeeIdx?api-version=2015-02-28"
& Invoke-WebRequest  -Method DELETE -Uri $url -ContentType "application/json" -Headers $headers

#remove datasource
$url = "https://company.search.windows.net/datasources/peopleDb?api-version=2015-02-28"
& Invoke-WebRequest  -Method DELETE -Uri $url -ContentType "application/json" -Headers $headers

{% endhighlight %} 

This time the name of the indexer, index and datasource is specified as ___part of the URL___ and the Request method is specified as __DELETE__.

With these scripts you are able to build and tead down the basic elements in Azure Search easily.


