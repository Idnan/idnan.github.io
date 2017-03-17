---
layout: post
title: Elasticsearch - C Style Comments In Mapping 
comments: true
---

If you have worked with large elasticsearch mappings then you may have encountered this problem, that there is no possibility to comment your mapping. As the elasticsearch mapping is defined in JSON, which does not allow to add comments. 

I thought the same until i stumbled upon following elasticsearch [issue](https://github.com/elastic/elasticsearch/issues/1394). Where a single line was [added to the configuration](https://github.com/elastic/elasticsearch/commit/6f7253c5242e7fb94d959ce291c88f93887e3bde).

```
jsonFactory.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
```

Ain't that cool. This happened only as the elasticsearch uses the JSON parser from [FasterXML](https://github.com/FasterXML). Which supports this feature to do comments in JSON. Check there [wiki page](https://github.com/FasterXML/jackson-core/wiki/JsonParser-Features) where it's described as follows.
 
 > ALLOW_COMMENTS (default: false) (for textual formats with concept of comments)
   For textual formats that do not have official comments, but for which "de facto" conventions exist (like JSON), determines whether use of such unofficial comments is allowed or not  
   Supported for: JSON  
   For JSON: enabling the feature allows recognition and handling of "C comments" (/* ... */) and "C++ comments" (// ....)
   
This feature of doing comments on elasticsearch works well but with some exceptions.
* Comments at the head of the JSON document, before the first opening `{` bracket are not working
* Single line comments with `//` only work at the end of the JSON document after the closing `}`

So following mapping is absolutely fine.

```
{
  /* mapping, used for all indices prefixed with "te" */
  "template": "te*",
  "settings": {
    /* only 1 shard is needed */
    "number_of_shards": 1
  },
  "mappings": {
    "type1": {
      "_source": {
        /* source not needed */
        "enabled": false
      },
      "properties": {
        "host_name": {
          "type": "string",
          /* host_name is used as filter, do not analyze */
          "index": "not_analyzed"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z YYYY"
        }
      }
    }
  }
}
```