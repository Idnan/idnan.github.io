Search as you type is an interesting feature of modern search engines. The basic idea of this feature is to give user instant feedback as they are typing. Implementation of this features can vary; sometimes it can based on popular searches and other times just preview of results. So elasticsearch has a lot of natural language text analysis options that makes it possible to use it as a part of search as you type system.  

So Last week i had to work on preparing a search as you type script that had to search on 3 million records. And here's the requirements that were given to me.

- Partial word matching: The query must match partial words. So typing "dub united emir" should return results contaning "Dubai United Emirates".
- Multiple search fields: The query must match across multiple fields. So typing "Dubai" should match against these fields hotel name and address. 
- Grouping: The results must be grouped by the location type like "Hotel, City, Landmark"

## Filters
So following were the custom [token filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html) that i created.

### stemmer
As some of the hotels had `'s` in there name like `Traveller's Inn Douglas & Blanshard`. So i used the possessive stemmer to remove `'s` from the tokens.
```
"possessive_stemmer":{
   "type":"stemmer",
   "name":"possessive_english"
}
```

#### word_delimiter
A filter of type `word_delimiter` split words into subwords and performs addtional transformation on each subgroup with the following rules.
- `split_on_numerics` If `true` then splits `2Home` into `[2] [Home]`
- `split_on_case_change` If `true` then splits `OakWood` into `[Oak] [Wood]`
- `generate_word_parts` If `true` causes parts of words to be generated like "PowerShot" into `[Power] [Shot]`
- `generate_number_parts` If `true` causes number subwords to be generated like "500-42" into `[500] [42]`
- `catenate_all` If `true` then catenates all words like `exit-142` into `[exit142]`
- `stem_english_possessive` If `true` causes trailing "'s" to be removed for each subword like "O'scia" into `[O] [scia]`

```
"strip_underscore":{
   "type":"word_delimiter",
   "split_on_numerics":false,
   "split_on_case_change":false,
   "generate_word_parts":false,
   "generate_number_parts":false,
   "catenate_all":true,
   "stem_english_possessive":false
}
```

### lowercase
As the name suggests it converts all token into lowercase.

### asciifolding
A token filter of type asciifolding converts non ascii characters into there equivalent 127 ascii characters like `Á` into `A`

## Tokenizer
Given below are the [tokenizers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html) that i used to split a query string into tokens

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

### keyword
`keyword` tokenizer basically converts enitire input as a single output. Like `Burj Al Arab` into `BurjAlArab`

### Analyzer
I used two [analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) for analyzing the query strings.

### nGram_analyzer
`nGram_analyzer` is a custom analyzer that is built using the above tokenizer and filters. First it applies the given tokenizer (`edge_ngram_tokenizer`) and then given token filters. The main purpose of that analyzer was to do partial matching on the given query string.
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

### analyzer_keyword
`analyzer_keyword` used to generate a single word output like `A brown fox jumps over the lazy dog` into `abrownfoxjumpsoverthelazydog`. I am basically using for exact match. So the exact match has more preference then the partial match.

So combining all of the above tokenizer, analyzer and filter i came up with this script.   

``` json
{
   "settings":{
      "analysis":{
         "filter":{
            "possessive_english_stemmer":{
               "type":"stemmer",
               "name":"possessive_english"
            },
            "nGram_filter":{
               "type":"edgeNGram",
               "min_gram":1,
               "max_gram":20,
               "token_chars":[
                  "letter",
                  "digit"
               ]
            },
            "strip_underscore":{
               "type":"word_delimiter",
               "split_on_numerics":false,
               "split_on_case_change":false,
               "generate_word_parts":false,
               "generate_number_parts":false,
               "catenate_all":true,
               "stem_english_possessive":false
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
                  "asciifolding",
                  "possessive_english_stemmer"
               ]
            },
            "analyzer_keyword":{
               "tokenizer":"keyword",
               "filter":[
                  "strip_underscore",
                  "lowercase",
                  "asciifolding"
               ]
            }
         }
      }
   },
   "mappings":{
      "location_search":{
         "properties":{
            "hotel_name":{
               "type":"text",
               "analyzer":"nGram_analyzer",
               "fields":{
                  "exact_match":{
                     "type":"text",
                     "analyzer":"analyzer_keyword"
                  }
               }
            }
         }
      }
   }
}
```