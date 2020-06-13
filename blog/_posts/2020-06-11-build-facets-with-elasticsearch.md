---
layout: post
title: Build Facets With Elasticsearch
comments: true
---

Facet allows the users to navigate through the website by applying filters for categories, attributes and price range etc. It's probably the most common filter that most the ecommerce websites need. In most cases each filter is followed by counter that indicates the number of items each group contains.

From a technical standpoint there are some challenges with these counters. To know the exact count we need to get the filters first than another query against each filter to get the number of count against each. In complex applications this leads to number of queries. Luckily elasticsearch has a solution to this, that we can achieve in one query.

## Facets

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200611/facets.png" style="max-width:635px;" alt=""/>
</figure>

Let's say that we have following data stored in elasticsearch index, and we want to achieve the above result.

<script src="https://gist.github.com/Idnan/bab028aa46b9bbe71c3639129e226048.js"></script>

Use [bulk api](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) to insert the above data. Once the API is executed the documents will stored in the `products` index. Make the below API call and check the count in the response to verify if the documents are stored successfully.

```shell
POST http://127.0.0.1:9200/products/_count
```

Now let's say we want to see the products of brand `Apple` and having processor type `Core i3`.   
```json
POST http://127.0.0.1:9200/products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "terms": {
                        "attributes.brand": [
                            "Apple"
                        ]
                    }
                },
                {
                    "terms": {
                        "attributes.processorType": [
                            "Core i3"
                        ]
                    }
                }
            ]
        }
    }
}
```

Now lets say we want to show user the `brand` and `processorType` filter along with the products count. For this we will use [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)
```json
POST http://127.0.0.1:9200/products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "terms": {
                        "attributes.brand": [
                            "Apple"
                        ]
                    }
                },
                {
                    "terms": {
                        "attributes.processorType": [
                            "Core i3"
                        ]
                    }
                }
            ]
        }
    },
    "aggregations": {
        "brand": {
         "terms": {"field": "attributes.brand"}
      },
      "processorType": {
         "terms": {"field": "attributes.processorType"}
      }
    }
}
```
Running the above query will show user two aggregation buckets one for `processorType` and other with `brand` along with products count against each  
<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200611/aggregation_1.png" style="height: 400px; width: auto;" alt=""/>
</figure>



Feel free to leave your feedback or questions in the comments section below.
