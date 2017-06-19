---
layout: post
title:  Template Based Search
date:   2016-12-12 11:40:18 +0800
categories: Elasticsearch
tags: elasticsearch python
author: Abhinav Kumar
mathjax: true
---


Introduction

Have you ever thought why google is so famous. Why its really hard to imagine something without google nowadays, because of its in- depth searching capability. Searching is something that can leverage the user experience and at the same time can add values to your business extent. If a website’s content is not organized properly, an efficient search engine is not only helpful but crucial, even for basic website navigation.An efficient search engine puts users in control of their search for information. But Making efficient search engine is a challenge especially if we are dealing with relational database and on top of that if we are handling millions of rows. Elastic search comes for the rescue here, Elasticsearch is a nosql , distributed full text datastore (having documents instead of tables). It is based on Lucene Engine , having rest apis exposed making it relatively easy to use , scalable and blazingly fast. 

In one of our project, we need to come up with a searching solution that can leverage the user to put a search on multiple fields. Not only search but to give runtime calculations based on values of the fields and based on those values further apply new search criterion. And if that is not enough we need to come up with added calculations and aggregations on different fields. With the given use case we started with a plan to use elastic search , with an idea in mind to achieve all above. Our main database was going to be cassandra but we have decided to use elastic search on top of that as search engine. Our Front end applications will be sending the fields and python is used for query formulation to be digested by elastic search. The request will go to the python api, api will query search engine and the results will be send back. The API abstracts the Elasticsearch details and adds some business logic regarding which content to return beyond simply matching the parameters specified by the front-end. 

Background

For the folks in relational world thats how to relate the things in elastic search 


We need to handle a business scenario where the client will be providing details of customers belonging to a particular organisation. Details are nothing but in the form of large csv (comma separated values) files. We need to take these csv files , process them (processing engine is based on apache spark , well thats whole new story :) ) and then make apache spark to put those records into Elasticsearch.

First thing we needed was to decide on index (Database in relational world). So we represented each organisation with unique identifier (uuid) and made them as elastic search index. And since we are storing nothing but customer details in each index, we decided on “customers” as our doc_type. So we end up with an Elasticsearch index with millions of customers in it. 


And there are lots of other details you have to get yourself acquainted with in order to make a full fledged search engine like mapping files, aliases (especially for reindexing), scripting (for on the fly field generation and search) , aggregations and last but not the least a simple text search. (https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html).

Text search can be implemented in different ways.

Whole code based approach.(power is on the developer).
Template based approach
Template based approach (Why?)

I was under dilemma as i was not able to decide which is the best approach so that i can take input from user and create a query accordingly and pass on to our search engine. After spending sometime online and talking to few of my friends , i figured it out that template based search is the best approach as it gives developer:

Less coding and less maintenance benefit.
More modular programming as you have to write a simple template and then read that as separate file.
Just the modification in the template lets you put search on different fields.
Template based approach (What?)

An Elasticsearch search template is kind of like a stored procedure in a relational database. It is written like a normal query with replacement parameters. Format of the template are expressed as Mustache.

Sample Template:

In order to use template , since we are coding everything in python, we have created a separate file called my_template.py.

We have stored the above template in a variable (COUNT_TEMPLATE) and the only thing we have to do is to include that variable in the main driver code. That thing can be done very easily in python by “import” 
```
{
    "query": {
               "bool":{
                        "must_not": [
                            {"term": {"field1": true}},
                            {"term": {"field2": "1"}}
                                 ],
                         "must":[
                             {"terms": {"field": [{{#var1}} "{{.}}", {{/var1}}]}},
                             {"range": 
                             {“date_field3": {"gt": 0}}
                              }
                                        {{#var2}},\
                                                        {"term": {"field4": "{{var3}}"}}
                                        {{/var2}}\
                                        ]}}\
                        }
```
statement. And in order to explore search_template option by elastic search we have used elastic search client. (https://elasticsearch-py.readthedocs.io/en/master/).

How to pass arrays?
```
from elasticsearch import Elasticsearch
from my_template import COUNT_TEMPLATE

ES_HOSTS = getattr(settings, 'ES_HOSTS', {"host": "localhost", "port": 9200})
es = Elasticsearch(hosts = [ES_HOSTS])

params = {"params": {"var": [listofvalues], "var2": “value”, “var3”: “value”}}

query = {"inline": COUNT_TEMPLATE}
query.update(params)

result = es.search_template(index=organisationid, doc_type=doc_type, body=json.dumps(query),search_type=‘count')

count = result["hits"]["total"]
return {"count": count}
```
In one of our use case we need to pass array of values to a particular field so that any one of the values can be considered as valid for the field. If you are familiar with elastic search that can be done using terms match like this:
```
{“query”: {“terms”: {“code”: [“123, “234”, “454”]}}}
```
search template (arrays):

The value for var1 list will be provide from the code. 

Conditions in search template (if-else):
```
{
                         "query": {
			{"terms": {"field": [{{#var1}} "{{.}}", {{/var1}}]}}
			}
		}
```
Since the generation of the search template is going to be dynamic that means if the front end is sending a particular field then only that will be added as part of search query otherwise not. So we need to create a conditional search template like this :
```
{{#if_var}},\
                                       {"term": {"field": "{{var}}"}}\
                         {{/if_var}}\
```
So it all depends on the value of if_var being sent as part of params in query . if the value of if_var == True then the term filter will be added to the query. 

Apart from this there are lots of other scenarios where we want to check for a condition if the condition is true then include a particular parameter otherwise add some thing else. The if-else block in search template:

Consider this complex scenario where we want to apply a range search on a particular field , so we can start with something like this elastic search query:
```
{“range”: {“salary”: {“gte”: 2000, “lte”: 1000}}}
```
Now what additional freedom we want to give to the user is to apply independent searches too , like he can only search for salary greater than 2000 or salary less than 1000 and even both. So to achieve this we have to use if-else block. But another issue with these scenarios is , we need to take care of comma in between “gte” and “lte” fields. We cannot hardcode comma in our search template as in the scenario where search is being made on say only “get: 2000” scenario then we don’t need the comma. like this
```
{“range”: {“salary”: {“gte”: 2000}}}
```
or
```
{“range”: {“salary”: {“lte”: 1000}}}
```
To achieve the above problem statement we need to put a check on the first field (if the fields exists) and the following field (if also present) then only we will be placing comma in between. And you can see in order to put “lte” (less than equal to) attribute we first check if “s_gte” variable is true , if it is true then only this will add comma other wise (else ,which is represented by ^ ) different condition will be called and executed accordingly.
```
{{#if_fterm}},
                                  {"range": {
                                           "salary": {
                                                  {{#s_gte}}
                                                       "gte": "{{s_gte}}"
                                                  {{/s_gte}}
                                                  {{#s_lte}}
                                                        {{#s_gte}},
                                                            "lte": "{{s_lte}}"
                                                        {{/s_gte}}
                                                        {{^s_gte}}
                                                            "lte": "{{s_lte}}"
                                                        {{/s_gte}}
                                                    {{/s_lte}}
                                                }
                                            }}
                        {{/if_fterm}}
```

CONCLUSION:

How easy the life is if you are using search templates. Infact this is just the beginning . Like that you can go to another level in achieving lots from search templates, you can add scripting , query strings and other different query formats from elastic search. With search templates, we can add new queries and modify existing queries by creating and modifying search templates.

So its always fast and easy to maintain code that you are writing using search templates. If you need to add more fields to your search query you just have to modify your template. 
