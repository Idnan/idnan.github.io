---
layout: post
title: Build Facets With Elasticsearch
comments: true
---

Facets allow the users to navigate through the website by applying filters for categories, attributes and price range etc. It's probably the most common filter that most of the ecommerce websites need. In most cases, each filter is followed by counter that indicates the number of items each group contains.

From a technical standpoint there are some challenges with these counters. To know the exact count we need to get the filters first then run another query against each filter to get the count of items against each. In complex applications, this leads to a number of queries and might not be performant. With elasticsearch aggregatio, we can achieve this in one query.

## Facets

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200611/facets.png" style="max-width:635px;" alt=""/>
</figure>

Let's create the index first to store the products with below mappings.
```json
PUT http://127.0.0.1:9200/products
{
    "mappings": {
        "dynamic_templates": [
            {
                "textFields": {
                    "match_mapping_type": "string",
                    "mapping": {
                        "type": "keyword"
                    }
                }
            }
        ]
    }
}
```

Let's say that we have the following data stored in the elasticsearch index called `products` inserted using [bulk api](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) and we want to achieve the above result.

<script src="https://gist.github.com/Idnan/bab028aa46b9bbe71c3639129e226048.js"></script>

You can run the below query to verify if you have the products in the index.

```shell
POST http://127.0.0.1:9200/products/_count
```

Let's say that we want to get all the products of brand `Apple` and having processor type `Core i3`. We can achieve that using the query below.

```json
POST http://127.0.0.1:9200/products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "terms": { "attributes.brand": ["Apple"] }
                },
                {
                    "terms": { "attributes.processorType": ["Core i3"] }
                }
            ]
        }
    }
}
```

Now let's say that we want to show the user, the `brand` and the `processorType` filter along with the products count. For this we will use [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)

```json
POST http://127.0.0.1:9200/products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "terms": { "attributes.brand": ["Apple"] }
                },
                {
                    "terms": { "attributes.processorType": ["Core i3"] }
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
Running the above query will return two aggregation buckets one for `processorType` and other with `brand` along with products count against each

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200611/aggregation_1.png" style="width: auto;" alt=""/>
</figure>

You will notice that it has only the `apple` brand. This is because by default elasticsearch executes its aggregations on the result set. But what about other brands? What if we want to get the list of all the brands? To fix this, we need to instruct Elasticsearch to execute the aggregation on the entire dataset, ignoring the query. We can do this by defining our aggregations as [global](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-global-aggregation.html).

```json
POST http://127.0.0.1:9200/products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "terms": { "attributes.brand": ["Apple"] }
                },
                {
                    "terms": { "attributes.processorType": ["Core i3"] }
                }
            ]
        }
    },
    "aggregations": {
        "filters": {
            "global": {},
            "aggregations": {
                "brand": {
                    "terms": {
                        "field": "attributes.brand"
                    }
                },
                "processorType": {
                    "terms": {
                        "field": "attributes.processorType"
                    }
                }
            }
        }
    }
}
``` 
<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200611/aggregation_2.png" style="width: auto;" alt=""/>
</figure>

This will solve our problem of getting all the filters but there's another problem — doing this will always show customer the same brand and processor type aggregations regardless of our filters. Our aggregations needs to be a bit more complex for this to work, we need to add filters to them. Each aggregation needs to count on the dataset with all the filters applied, except for its own.

```json
POST http://127.0.0.1:9200/products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "terms": { "attributes.brand": ["Apple"] }
                },
                {
                    "terms": { "attributes.processorType": ["Core i5"] }
                }
            ]
        }
    },
    "aggregations": {
        "filters": {
            "global": {},
            "aggregations": {
                "brand_agg": {
                    "filter": {
                        "terms": { "attributes.processorType": ["Core i5"] }
                    },
                    "aggregations": {
                        "brand": {
                            "terms": {
                                "field": "attributes.brand"
                            }
                        }
                    }
                },
                "processorType_agg": {
                    "filter": {
                        "terms": { "attributes.brand": ["Apple"] }
                    },
                    "aggregations": {
                        "processorType": {
                            "terms": {
                                "field": "attributes.processorType"
                            }
                        }
                    }
                }
            }
        }
    }
}
``` 
Now if you run the query above, it will produce the result that we expect.

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200611/aggregation_3.png" style="width: auto;" alt=""/>
</figure>

And that wraps it up. Feel free to leave your feedback or questions in the comments section below.