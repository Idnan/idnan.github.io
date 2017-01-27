Search as you type is an interesting feature of modern search engines. The basic idea of this feature is to give user instant feedback as they type. Implementation of this feature can vary; sometimes it can be based on popular searches and other times just a preview of results. 

Elasticsearch has a lot of natural language text analysis options that makes it possible to be used as a part of search as you type system. This article is going to be a drill down on how to implement search as you type using Elasticsearch that will search on movies database fulfilling the below requirements.

- **Partial word matching**: The query must match partial words. So typing "Capta Civil War" should return results containing "	Captain Civil War" or "Captain America Civil War" or "Captain America's: Civillian War" etc. 
- **Grouping**: The results must be grouped by the year

Note that I am not going to describe how to setup Elasticsearch in this article and it is just going to be the details on how to make "Search as you type" service.

## Key Components
Lets begin with the key components that we will use for the search:

### a. Tokenizer

[Tokenizing](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html) is a process of scanning a string of characters as the user types and converting that character string into a list of words. Where each item is called `token`. During parsing we wish to deal with tokens as its easier and faster to deal with tokens. So to parse string we must find word boundaries and elasticsearch has plenty of built in tokenizers that can be used. For example a basic tokenizer will do following:

```
Star Trek Beyond
-> Applying Tokenizer ->
[Star] [Trek] [Beyond]
```

We can either write a custom tokenizer or can use one of built-in ones for example `edgeNGram`, `nGram`, `snowBall` etc. For our usecase, we are going to rely upon the built-in tokenizer called `edgeNGram`

`edgeNGram` tokenizer is useful for "as you type" queries. It only generate `nGrams` from the beginning of the words like word `brown` is tokenized into
```
[b] [br] [bro] [brown]
```
We are going to rely upon the following properties of the tokenizer

- **`min_gram`/`max_gram`** to control the minimum and maximum length of `nGram`s by using follwoing properties.
- **`token_chars`** for the characters to keep; it allows [five different character classes](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html#_configuration_17) to be specified as "characters to keep". Elasticsearch will split on character classes not specified. If you don't specify any character classes, then all characters are kept.

Lets take an example for tokenization:

**Query:** `into the forest`
**Filter:**
```
"edge_ngram_tokenizer":{
   "type":"edgeNGram",
   "min_gram":"1",
   "max_gram":"20",
   "token_chars":[
      "letter",        // Do not split on letters and consider words only
      "digit"          // Do not split on digits e.g. `21 Jump street` to not split `21` i.e resulting in `[2]` and `[1]`
   ]
}
```
**Result:**
```
[in] [to] [the] [for] [forest]
```

### b. Token Filters

> Token filters accept a stream of tokens from a tokenizer and can modify tokens (eg lowercasing), delete tokens (eg remove stopwords) or add tokens (eg synonyms). Elasticsearch has a number of built in token filters which can be used to build custom analyzers.


[Token filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html) basically work on token streams from a tokenizer and can modify (e.g. `lowercase`), delete (e.g. remove `stopwords`) or add (e.g. add `synonyms`) tokens. 

Elasticsearch offers a plenty of built-in token filters. We are going to be interested in the below filters for our usecase:

- `lowercase` that converts all token into lowercase.
- `asciifolding` that converts non ASCII characters into there equivalent 127 ASCII characters like `Café Society` into `Cafe Society` etc

### c. Analyzer
[Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) are the way through which `Lucene` (java based search engine that is the backbone of elastic search) processes and indexes the data. Each analyzer is composed of following 
 
- One Tokenizer
- Zero or more Token/Char filters

Here is how the analyzer that I have created for our implementation looks like:

```
"nGram_analyzer":{
   "type":"custom",
   "tokenizer":"edge_ngram_tokenizer",
   "filter":[
      "lowercase",
      "asciifolding",
      "possessive_stemmer"
   ]
}
```

I decided to name it `nGram_analyzer` but you can name it anything that you want. First of all it applies the given tokenizer `edge_ngram_tokenizer` on the search text and then given token filters. The main purpose of it is to do partial matching on the given query string.


## Creating Index and Types

Combining all of the above i.e. tokenizer, filter and analyzer first of all we will create an index or an *elasticsearch database*. While creating an index we specify the mapping which is a set of *tables* for our so called *database*. For example, here we are creating a `movie` type or *table* with properties or *columns* `display_name` and `year`. With each property we can specify an analyzer which will be used by elastic search to filter. If no analyzer is specified, it will be considered a static value and elastic search will not use it during the filter. Here is what our sample mapping looks like

```json
"movie":{
 "properties":{
    "display_name":{
       "type":"text",
       "analyzer":"nGram_analyzer"
    },
    "year": {
      "type": "keyword"
    }
 }
}
```

Putting it all togeter we have below script to create an index called `entertainment_index` having the type `movie` with the above said properties.

```
curl -XPUT "http://localhost:9200/entertainment_index/" -d'
{
   "settings":{
      "analysis":{
         "filter":{
            "nGram_filter":{
               "type":"edgeNGram",
               "min_gram":1,
               "max_gram":20,
               "token_chars":[
                  "letter",
                  "digit"
               ]
            }
         },
         "tokenizer":{
            "edge_ngram_tokenizer":{
               "type":"edgeNGram",
               "min_gram":"1",
               "max_gram":"20",
               "token_chars":[
                  "letter",
                  "digit"
               ]
            }
         },
         "analyzer":{
            "nGram_analyzer":{
               "type":"custom",
               "tokenizer":"edge_ngram_tokenizer",
               "filter":[
                  "lowercase",
                  "asciifolding"
               ]
            }
         }
      }
   },
   "mappings":{
      "movie":{
         "properties":{
            "display_name":{
               "type":"text",
               "analyzer":"nGram_analyzer"
            },
            "year": {
              "type": "keyword"
            }
         }
      }
   }
}'
```

## Populating Sample Data

Below is how the HTTP request to populate data looks like

```
PUT /{index_name}/{type}/{id_to_assign:optional}
```

Lets add some sample data for the testing purposes. 

```
curl -XPUT "http://localhost:9200/entertainment_index/movie/1" -d'
{
   "display_name": "X-Men Origins: Wolverine",
   "year": 2009
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/2" -d'
{
   "display_name": "X-Men: First Class",
   "year": 2011
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/3" -d'
{
   "display_name": "Zombieland",
   "year": 2009
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/4" -d'
{
   "display_name": "Monsters University",
   "year": 2013
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/5" -d'
{
   "display_name": "Monsters",
   "year": 2010
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/6" -d'
{
   "display_name": "Lost in London",
   "year": 2017
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/7" -d'
{
   "display_name": "London Spy",
   "year": 2015
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/8" -d'
{
   "display_name": "London Road",
   "year": 2015
}'
curl -XPUT "http://localhost:9200/entertainment_index/movie/9" -d'
{
   "display_name": "London Fields",
   "year": 2017
}'
```

## Applying Search

Here is hwo the search request looks like

```
POST /{index}/{type}/_search
```

Now lets search against the stored data and test if we are getting the required results.

```
curl -XPOST "http://localhost:9200/entertainment_index/movie/_search" -d'
{
    "aggs": {
        "year": {
            "terms": {
                "field": "year"
            },
            "aggs": {
                "group_docs": {
                    "top_hits": {
                        "size": 10
                    }
                }
            }
        }
    },
    "query": {
    	"match": {
    		"display_name": {
    			"operator": "and",
			"query": "london"
    		}
    	}
    }
}'
```

`aggs` specifies the grouping of results based on the `year` and limits to top 10 results from each of the groups. And in match `operator: and` means that the complete input string must be part of movie name and `query` is the user input. After running the above query you will get following results.

```
2015
    London Spy
    London Road
2017
    Lost in London
    London Fields
```

And this is how we can implement "search as you type" service. Follow up with the questions and comments in the section below. Also, I would love to hear how you are using Elasticsearch in your system.