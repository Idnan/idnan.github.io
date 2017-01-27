Search as you type is an interesting feature of modern search engines. The basic idea of this feature is to give user instant feedback as they are typing. Implementation of this features can vary; sometimes it can based on popular searches and other times just preview of results. So elasticsearch has a lot of natural language text analysis options that makes it possible to use it as a part of search as you type system.  

So lets prepare a script that will search on movies database. And here are the requirements that it should fulfill

- Partial word matching: The query must match partial words. So typing "Capta Civil War" should return results containing "	Captain Civil War". 
- Grouping: The results must be grouped by the year

## Token Filters
[Token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html) basically works on token streams and then these filters can modify, add or delete text the input. Elasticsearch offers a plenty of built in token filter. E.g. `lowercase` token filter converts each input to lowercase. So given below are the filters explained that i used.

### lowercase
As the name suggests it converts all token into lowercase.

### asciifolding
A token filter of type asciifolding converts non ascii characters into there equivalent 127 ascii characters like `Café Society` into `Cafe Society`

## Tokenizer
[Tokenizing](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html) is a process of scanning a string og characters as the user types and converting that character string into a list of words. Where each item is called `token`. During parsing we wish to deal with token as its easier and faster to deal with tokens. So to parse string we must find word boundaries and elasticsearch has plenty of built in tokenizer that can be used.  E.g. a `basic tokenizer` will do following

```
Star Trek Beyond
-> Applying Tokenizer ->
[Star] [Trek] [Beyond]
```

The tokenizer that i used are explained below.

### edgeNGram
`edgeNGram` tokenize is useful for as to type queries. It only generate nGrams from the beginning of the word like word `brown` is tokenized into
```
[b] [br] [bro] [brown]
```
and you can control the minimum and maximum length of nGram by using follwoing properties `min_gram` and `max_gram`. And `token_chars` allows [five different character classes](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenizer.html#_configuration_17) to be specified as characters to "keep". Elasticsearch will split on characters not specified. If you don't specify any character classes, then all characters are kept. 
Lets take an example

Query: "a brown fox"
Filter:
```
"edge_ngram_tokenizer":{
   "type":"edgeNGram",
   "min_gram":"1",
   "max_gram":"20",
   "token_chars":[
      "letter",
      "digit"
   ]
}
```

Result:
```
[a] [q] [qu] [qui] [quic] [quick] [b] [br] [bro] [brow] [brown] [f] [fo] [fox]
```

### Analyzer
[Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) are the way through which `Lucene` process and indexes the data. Each analyzer is composed of following 
 
- zero or more Token/Char filters
- one Tokenizer

First analyzer are registered in the analysis module then they are used inside mapping definition. So here's my analyzer that i used


### nGram_analyzer
`nGram_analyzer` is a custom analyzer that is built using the above tokenizer and filters. First it applies the given tokenizer (`edge_ngram_tokenizer`) and then token filters. The main purpose of that analyzer is to do partial matching on the given query string.
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

So combining all of the above tokenizer, analyzer and filter i came up with this script. Run the script to create index with given mapping  

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

Next, we add the documents

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

Now lets search against the stored data and test if we are getting required results.

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

Here's few things to notice. First `aggs` its doing the grouping of results based on the `year` and getting top 10 results from each group. And in match `operator: and` means that the complete input string must be part of movie name and `query` is the user input. After running the above query you will get following results.

```
2015
    London Spy
    London Road
2017
    Lost in London
    London Fields
```
    
