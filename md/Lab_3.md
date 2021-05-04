
<img align="right" src="./images/logo.png">



Lab 3. Searching - What is Relevant
------------------------------------------------



We will cover the following topics in this lab:


-   The basics of text analysis
-   Searching from structured data
-   Writing compound queries
-   Searching from full-text
-   Modeling relationships




### Understanding Elasticsearch analyzers



The main task of an analyzer is to take the value of a field and break
it down into terms. In Lab 2, [*Getting Started with
Elasticsearch*], we looked at the structure of an inverted index. The job of the analyzer is to take
documents and each field within them and extract terms from them. These
terms[* *] make the index searchable, that is, they can help
us find out which documents contain particular search terms.

The analyzer performs this process of breaking up input character
streams into terms. This happens twice: 


-   At the time of indexing
-   At the time of searching


The core task of the analyzer is to parse the document fields and build
the actual index.

Every field of `text` type needs to be analyzed before the
document is indexed. This process of analysis is what makes the
documents searchable by any term that is used at the time of searching.

Analyzers can be configured on a per field basis, that is, it is
possible to have two fields of the `text` type within the same
document, each one using different analyzers.

Elasticsearch uses analyzers to analyze text data. An analyzer has the
following components:


-   **Character filters**: Zero or more
-   **Tokenizer**: Exactly one
-   **Token filters**: Zero or more


The following diagram depicts the components of an analyzer:


![](./images/17fb404f-a755-4cbf-8dd2-3bf8bc373376.png)


Figure 3.1: Anatomy of an analyzer

Let\'s understand the role of each component one by one.



#### Character filters



When composing an analyzer, we can configure
zero or more character filters[*. *] A character filter works
on a stream of characters from the input field; each
character filter can add, remove, or change
the characters in the input field.

Elasticsearch ships with a few built-in character filters, which you can
use to compose or create your own custom analyzer. 

For example, one of the character filters that Elasticsearch ships with
is the Mapping Char Filter. It can map a character or sequence of
characters into target characters.

For example, you may want to transform emoticons into some text that
represents those emoticons:


-   `:)` should be translated to `_smile_`
-   `:(` should be translated to `_sad_`
-   `:D` should be translated to `_laugh_`


This can be achieved through the following character filter. The short
name for the Mapping Char Filter is the mapping filter:

```
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            ":) => _smile_",
            ":( => _sad_",
            ":D => _laugh_"
          ]
        }
      }
```

When this character filter is used to create an analyzer, it will have
the following effect:

`Good morning everyone :)` will be transformed in to
`Good morning everyone _smile_`.

`I am not feeling well today :(` will be transformed in to
`I am not feeling well today _sad_`.

Since character filters are at the very beginning of the processing
chain in an analyzer (see [*Figure 3.1*] ),[* *] the
tokenizer will always see the replaced characters. Character filters can
be useful for replacing characters with something more meaningful in
certain cases, such as replacing the numeric characters from other
languages with English language decimals, that is, digits from Hindi,
Arabic, and other languages can be turned into 0, 1, 2, and so on.

You can find a list of available built-in character
filters here:
<https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html>.

#### Tokenizer



An analyzer has exactly one tokenizer. The responsibility of a
tokenizer is to receive a stream of
characters and generate a stream of tokens. These tokens are used to
build an inverted index. A token is roughly
equivalent to a word. In addition to breaking down characters into words
or tokens, it also produces, in its output, the start and end offset of
each token in the input stream.

Elasticsearch ships with a number of tokenizers that can be used to
compose a custom analyzer; these tokenizers are also used by
Elasticsearch itself to compose its built-in analyzers.

You can find a list of available built-in tokenizers here: <https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html>.

Standard tokenizer is one of the most popular tokenizers as it is
suitable for most languages. Let\'s look at what standard tokenizer
does.



##### Standard tokenizer



Loosely speaking, the standard tokenizer
breaks down a stream of characters by
separating them with whitespace characters and punctuation.

The following example shows how the standard tokenizer breaks a
character stream into tokens:

```
POST _analyze
{
  "tokenizer": "standard",
  "text": "Tokenizer breaks characters into tokens!"
}
```

The preceding command produces the following output; notice
the `start_offset`, `end_offset`, and
`positions` in the output:

```
{
  "tokens": [
    {
      "token": "Tokenizer",
      "start_offset": 0,
      "end_offset": 9,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "breaks",
      "start_offset": 10,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "characters",
      "start_offset": 17,
      "end_offset": 27,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "into",
      "start_offset": 28,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "tokens",
      "start_offset": 33,
      "end_offset": 39,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

This token stream can be further processed by the token filters of the
analyzer.


#### Token filters



There can be zero or more token filters in an
analyzer. Every token filter can add, remove, or change tokens in the
input token stream that it receives. Since it
is possible to have multiple token filters in an analyzer, the output of
each token filter is sent to the next one until all token filters are
considered.

Elasticsearch comes with a number of token
filters, and they can be used to compose your own custom analyzers.

Some examples of built-in token filters are the following:


-   **Lowercase token
    filter**:[* *] Replaces all
    tokens in the input with their lowercase
    versions.
-   **Stop token
    filter**:[* *] Removes stopwords,
    that is, words that do not add more meaning to the context. For
    example, in English sentences, words
    like [*is*], [*a*], [*an*], and
    [*the*], do not add extra meaning to a sentence. For many
    text search problems, it makes sense to remove such words, as they
    don\'t add any extra meaning or context to the content.


You can find a list of available built-in
token
filters here: <https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html>.

Thus far, we have looked at the role of character filters, tokenizers,
and token filters. This sets us up to understand how some of the
built-in analyzers in Elasticsearch are composed. 


### Using built-in analyzers



Elasticsearch comes with a number of built-in
analyzers that can be used directly. Almost all of these analyzers work
without any need for additional configuration, but they provide the
flexibility of configuring some parameters.

Some analyzers come packaged with Elasticsearch. Some popular analyzers
are the following:


-   **Standard analyzer**: This is the default analyzer in Elasticsearch. If not
    overridden by any other field-level,
    type-level, or index-level analyzer, all fields are analyzed using
    this analyzer.
-   **Language analyzers**: Different
    languages have different
    grammatical rules. There are differences
    between some languages as to how a stream of characters is tokenized
    into words or tokens. Additionally, each language has its own set of
    stopwords, which can be configured while configuring language
    analyzers.
-   **Whitespace analyzer**: The whitespace
    analyzer breaks down input into tokens wherever it finds a whitespace token such as
    a space, a tab, a new line, or a carriage return. 


You can find a list of the available built-in
analyzers
here: <https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html>.



#### Standard analyzer



Standard Analyzer is suitable for many
languages and situations. It can also be customized for the underlying
language or situation. Standard analyzer comprises of the
following components:

**Tokenizer**:


-   **Standard tokenizer**: A tokenizer that splits tokens at
    whitespace characters


**Token filters**:


-   **Standard token filter**: Standard token filter is used
    as a placeholder token filter within the Standard Analyzer. It does
    not change any of the input tokens but may be used in future to
    perform some tasks.
-   **Lowercase token filter**: Makes all tokens in the input
    lowercase.
-   **Stop token filter**: Removes specified stopwords. The
    default setting has a stopword list set to `_none_`, which
    doesn\'t remove any stopwords by default.


Let\'s see how Standard analyzer works by
default with an example:

```
PUT index_standard_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std": { 
          "type": "standard"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text": {
        "type": "text",
        "analyzer": "std"
      }
    }
  }
}
```

Here, we created an index, `index_standard_analyzer`. There
are two things to notice here:


-   Under the `settings` element, we explicitly defined one
    analyzer with the name `std`. The `type` of
    analyzer is `standard`. Apart from this, we did not do any
    additional configuration on Standard Analyzer.
-   We created one type called `_doc` in the index and
    explicitly set a field level analyzer on the only
    field, `my_text`[*.*] 


Let\'s check how Elasticsearch will do the analysis for
the `my_text` field whenever any document is indexed in this
index. We can do this test using the `_analyze` API, as we saw
earlier:

```
POST index_standard_analyzer/_analyze
{
  "field": "my_text", 
  "text": "The Standard Analyzer works this way."
}
```

The output of this command shows the
following tokens:

```
{
  "tokens": [
    {
      "token": "the",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "standard",
      "start_offset": 4,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "analyzer",
      "start_offset": 13,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "works",
      "start_offset": 22,
      "end_offset": 27,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "this",
      "start_offset": 28,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "way",
      "start_offset": 33,
      "end_offset": 36,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

Please note that, in this case, the field level analyzer for the
`my_field`[* *] field was set to Standard Analyzer
explicitly. Even if it wasn\'t set explicitly for the field, Standard
Analyzer is the default analyzer if no other analyzer is specified.

As you can see, all of the tokens in the output are lowercase. Even
though the Standard Analyzer has a stop token filter, none of the tokens
are filtered out. This is why the `_analyze` output has all
words as tokens.

Let\'s create another index that uses English language stopwords:

```
PUT index_standard_analyzer_english_stopwords
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std": { 
          "type": "standard",
"stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text": {
        "type": "text",
        "analyzer": "std"
      }
    }
  }
}
```

Notice the difference here. This new index is
using `_english_` stopwords[*. *] You can also
specify a list of stopwords directly, such as `stopwords`:
([*a*], [*an*], [*the*] ). The
`_english_` value includes all such English words.

When you try the `_analyze` API on the new index, you will see
that it removes the stopwords, such as `the` and
`this`:

```
POST index_standard_analyzer_english_stopwords/_analyze
{
  "field": "my_text", 
  "text": "The Standard Analyzer works this way."
}
```

It returns a response like the following:

```
{
  "tokens": [
    {
      "token": "standard",
      "start_offset": 4,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "analyzer",
      "start_offset": 13,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "works",
      "start_offset": 22,
      "end_offset": 27,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "way",
      "start_offset": 33,
      "end_offset": 36,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

English stopwords such as `the` and `this` are
removed. As you can see, with a little configuration, Standard Analyzer
can be used for English and many other languages.

Let\'s go through a practical application of creating a custom analyzer.


### Implementing autocomplete with a custom analyzer



In certain situations, you may want to create
your own custom analyzer by composing
character filters, tokenizers, and token filters of your choice. Please
remember that most requirements can be fulfilled by one of the built-in
analyzers with some configuration. Let\'s create an analyzer that can
help when implementing autocomplete functionality.

To support autocomplete, we cannot rely on Standard Analyzer or one of
the pre-built analyzers in Elasticsearch. The analyzer is responsible
for generating the terms at indexing time. Our analyzer should be able
to generate the terms that can help with autocompletion. Let\'s
understand this through a concrete example.

If we were to use Standard Analyzer at indexing time, the following
terms would be generated for the field with the
`Learning Elastic Stack 7` value:

```
GET /_analyze
{
  "text": "Learning Elastic Stack 7",
  "analyzer": "standard"
}
```

The response of this request would contain the terms
`Learning`, `Elastic`, `Stack`,
and `7`. These are the terms that Elasticsearch would create
and store in the index if Standard Analyzer was used. Now, what we want
to support is that when the user starts typing a few characters, we
should be able to match possible matching products. For example, if the
user has typed `elas`, it should still recommend
`Learning Elastic Stack 7` as a product. Let\'s compose an
analyzer that can generate terms such as [*el*],
[*ela*], [*elas*], [*elast*],
[*elasti*], [*elastic*], [*le*],
[*lea*], and so on:

```
PUT /custom_analyzer_index
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "custom_analyzer": {
            "type": "custom",
            "tokenizer": "standard",
            "filter": [
              "lowercase",
              "custom_edge_ngram"
            ]
          }
        },
        "filter": {
          "custom_edge_ngram": {
            "type": "edge_ngram",
            "min_gram": 2,
            "max_gram": 10
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "product": {
        "type": "text",
        "analyzer": "custom_analyzer",
        "search_analyzer": "standard"
      }
    }
  }
}
```

This index definition creates a custom
analyzer that uses Standard Tokenizer to create the tokens and uses two
token filters -- a `lowercase` token filter and
the `edge_ngram` token filter. The `edge_ngram`
token filter breaks down each token into
lengths of 2 characters, 3 characters, and 4 characters, up to 10
characters. One incoming token, such as elastic, will generate tokens
such as [*el*], [*ela*], and so on, from one
token. This will enable autocompletion searches.

Given that the following two products are indexed, and the user has
typed `Ela` so far, the search should return both products:

```
POST /custom_analyzer_index/_doc
{
  "product": "Learning Elastic Stack 7"
}

POST /custom_analyzer_index/_doc
{
 "product": "Mastering Elasticsearch"
}

GET /custom_analyzer_index/_search
{
 "query": {
   "match": {
     "product": "Ela"
   }
 }
}
```

This would not have been possible if the index was built using
Standard Analyzer at indexing time. We will
cover the `match` query later in this lab. For now, you
can assume that it applies Standard Analyzer (the analyzer configured as
`search_analyzer`) on the given search terms and then uses the
output terms to perform the search. In this case, it would search for
the term `Ela` in the index. Since the index was built using a
custom analyzer using an `edge_ngram` token filter, it would
find a match for both products.

In this section, we have learned about analyzers. Analyzers play a vital
role in the functioning of Elasticsearch. Analyzers decide which terms
get stored in the index. As a result, what kind of search operations can
be performed on the index after it has been built is decided by the
analyzer used at index time. For example, Standard Analyzer cannot
fulfill the requirement of supporting the autocompletion feature. We
have looked at the anatomy of analyzers, tokenizers, token filters,
character filters, and some built-in support in Elasticsearch. We also
looked at a scenario in which building a custom analyzer solves a real business problem regarding supporting
the autocomplete function in your application.

Before we move onto the next section and start looking at different
query types, let\'s set up the necessary index with the data required
for the next section. We are going to use product catalog data taken
from the popular e-commerce site
[www.amazon.com](http://www.amazon.com/). The data is
downloadable from
<http://dbs.uni-leipzig.de/file/Amazon-GoogleProducts.zip>.

Before we start with the queries, let\'s create the required index and
import some data:

```
PUT /amazon_products
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {}
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "title": {
        "type": "text"
      },
      "description": {
        "type": "text"
      },
      "manufacturer": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      },
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      }
    }
  }
}
```

The `title` and `description` fields are analyzed
text fields on which analysis should be
performed. This will enable full-text queries on these fields. The
`manufacturer` field is of the `text` type, but it
also has a field with the name `raw`[*.*]  The
`manufacturer` field is stored in two
ways, as `text`, and `manufacturer.raw` is stored as
a `keyword`. All fields of the `keyword`
type internally use the keyword analyzer. The keyword analyzer consists
of just the keyword tokenizer, which is a **noop** tokenizer,
simply returning the whole input as one token. Remember, in an analyzer,
character filters and token filters are optional. Thus, by using
the `keyword` type on the field, we are choosing a noop
analyzer and hence skipping the whole analysis process on that field.

The `price` field is chosen to be of
the `scaled_float` type. This is a new type introduced with
Elastic 6.0, which internally stores floats as scaled whole numbers. For
example, 13.99 will be stored as 1399 with a scaling factor of 100. This
is space-efficient as `float` and `double` datatypes
occupy much more space.

To import the data, please follow the
instructions in the book\'s accompanying source code repository at
GitHub: [https://github.com/fenago/elasticsearch](https://github.com/fenago/elasticsearch/tree/v7.0/) in
the branch
[v7.0](https://github.com/fenago/elasticsearch/tree/v7.0/).

The instructions for importing data are
in [lab-03/products\_data/README.md](https://github.com/fenago/elasticsearch/tree/v7.0/lab-03/products_data#import-product-data-into-elasticsearch).

After you have imported the data, verify that it is imported with the
following query:

```
GET /amazon_products/_search
{
  "query": {
    "match_all": {}
  }
}
```

In the next section, we will look at structured search queries.



Searching from structured data
------------------------------------------------



In certain situations, we may want to find out whether a given document
should be included or not; that is, a simple binary answer. On the other
hand, there are other types of queries that
are relevance-based. Such relevance-based queries also return a score
against each document to say how well that document fits the query. Most
structured queries do not need relevance-based scoring, and the answer
is a simple yes/no for any item to be included or excluded from the
result. These structured search queries are also referred to as **term-level queries**.

Let\'s understand the flow of a term-level query\'s execution:


![](./images/3808134c-ce62-4db1-9e50-5d3add4c5b84.png)


Figure 3.2: Term-level query flow

As you can see, the figure is divided into
two parts. The left-hand half of the figure depicts what happens at the
time of indexing, and the right-hand half depicts what happens at the
time of a query when a term-level query is executed.

 

Looking at the left-hand half of the figure, we can see what happens
during indexing. Here, specifically, we are looking at how the inverted
index is built and queried for the `manufacturer.raw` field.
Remember, from our definition of the index, the
`manufacturer.raw` field is of
the `keyword` type.[* *] The `keyword`
type fields are not analyzed; the field\'s value is directly stored as a
term in the inverted index. 

At query time, when we search using a `term` query that is a
term-level query, we see the flow of execution on the right-hand half of
the figure. The `term` query, as we will see later in this
section, is a term-level query that directly passes on
the `victory multimedia`search term, without breaking it down
using an analyzer. This is how term-level queries completely skip the
analysis process at query time and directly search for the given term in
the inverted index.

These term-level queries create a foundation layer on which other,
high-level, full-text queries are built. We will look at high-level
queries in the next section.

We will cover the following structured or term-level queries:


-   Range query
-   Exists query
-   Term query
-   Terms query

### Range query



Range queries can be applied to fields with
datatypes that have natural ordering. For example, integers, logs, and
dates have a natural order. There is no ambiguity in deciding whether
one value is less, equal to, or greater than the other values. Because
of this well-defined datatypes order, a
`range` query can be applied.

We will look at how to apply range queries in the following ways:


-   On numeric types
-   With score boosting
-   On dates


Let\'s look at the most typical `range` query on a numeric
field.



#### Range query on numeric types



Suppose we are storing products with their
prices in an Elasticsearch index and we want to get all products within
a range. The following is the query to get products in the range of \$10
to \$20:

```
GET /amazon_products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```

The response of this query looks like the following:

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total" : {
"value" : 201,                                   1
      "relation" : "eq"
    },                                     
"max_score": 1.0,                                  2
    "hits": [
      {
        "_index": "amazon_products",
        "_type": "_doc",
        "_id": "AV5lK4WiaMctupbz_61a",
"_score": 1,                                   3
        "_source": {
"price": "19.99",                            4
          "description": "reel deal casino championship edition (win 98 me nt 2000 xp)",
          "id": "b00070ouja",
          "title": "reel deal casino championship edition",
          "manufacturer": "phantom efx",
          "tags": []
        }
      },
```

Please take a note of the following:


-   The `hits.total.value` field in the response shows how
    many search hits were found. Here, there were `201` search
    hits.
-   The `hits.max_score` field shows the score of the best
    matching document for the query. Since a `range` query is
    a structured query without any importance or relevance, it is
    executed as a filter. It doesn\'t do the scoring. All documents have
    a score of one.
-   The `hits.hits` array lists all the
    actual `hits`[*.*]  Elasticsearch doesn\'t
    return all `201` hits in a single pass by default. It just
    returns the first 10 records. If you wish to scroll through all
    results, you can do so easily by issuing multiple queries, as we
    will see later.
-   The `price` field in all search hits would
    be within the requested range, that is,
    10: `<= price <= 20`.


#### Range query with score boosting



By default, the `range` query assigns a score of `1` to each matching document. What if you are
using a `range` query in conjunction with some other query and
you want to assign a higher score to the resulting document if it
satisfies some criteria? We will look at compound queries such as the
`bool` query, where you can combine multiple types of queries.
The `range` query allows you to provide a `boost`
parameter to enhance its score relative to other query/queries that it
is combined with:

```
GET /amazon_products/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "range": {
      "price": {
        "gte": 10,
        "lte": 20,
"boost": 2.2
      }
    }
  }
}
```

All documents that pass the filter will have a score of `2.2`
instead of 1 in this query.

 

#### Range query on dates



A `range` query can also be applied
to date fields since dates are also inherently ordered. You can specify
the date format while querying a date range:

```
GET /orders/_search
{"query":{"range":{"orderDate":{"gte":"01/09/2017","lte":"30/09/2017","format":"dd/MM/yyyy"}}}}
```

The preceding query will filter all the orders that were placed in the
month of September 2017.

Elasticsearch allows us to use dates with or without the time in its
queries. It also supports the use of special terms,
including `now`to denote the current time. For example, the
following query queries data from the last 7 days up until now, that
is, data from exactly 24 x 7 hours ago till now with a precision of
milliseconds:

```
GET /orders/_search
{"query":{"range":{"orderDate":{"gte":"now-7d","lte":"now"}}}}
```

The ability to use terms such as `now` makes this easier to
comprehend.


### Note

Elasticsearch supports many date-math operations. As part of its date
support, it supports the special keyword `now`. It also
supports adding or subtracting time with different units of measurement.
It supports single character shorthands such as `y` (year),
`M` (month), `w` (week), `d` (day),
`h`or `H` (hours), `m` (minutes), and
`s` (seconds). For example, `now - 1y` would mean a
time of exactly 1 year ago until this moment. It is possible to round
time into different units. For example, to round the interval by day,
inclusive of both the start and end interval day, use
`"gte": "now - 7d/d"` or`"lte": "now/d"`.
Specifying `/d` rounds time by days.


The `range` query runs in filter context by default. It
doesn\'t calculate any scores and the score is always set to
`1` for all matching documents.


### Exists query



Sometimes it is useful to obtain only records
that have non-null and non-empty values in a certain field. For example,
getting all products that have description
fields defined:

```
GET /amazon_products/_search
{
  "query": {
    "exists": {
      "field": "description"
    }
  }
}
```

The `exists` query turns the query into a filter; in other
words, it runs in a filter context. This is
similar to the `range` query where the scores don\'t matter. 


### Note

What is a **Filter Context**? When the query is just about
filtering our documents, that is, deciding whether to include the
document in the result or not, it is sufficient to skip the scoring
process. Elasticsearch can skip the scoring process for certain types of
queries and assign a uniform score of 1 to each document, which passes
the filter criteria. This not only speeds up the query (as the scoring
process is skipped) but also allows Elasticsearch to cache the results
of filters. Elasticsearch caches the results of filters by maintaining
arrays of zeros and ones.


### Term query



How would you find all of the products made
by a particular manufacturer? We know that the manufacturer field in our
data is of the string type. The name of a
manufacturer can possibly contain whitespaces. What we are looking for
here is an exact search. For example, when we search
for `victory multimedia`, we don\'t want any results that have
a manufacturer that contains just `victory` or
just `multimedia`. You can use a `term` query to
achieve that.

When we defined the `manufacturer` field, we stored it as
both `text` and `keyword` fields. When doing an
exact match, we have to use the field with the `keyword` type:

```
GET /amazon_products/_search
{
  "query": {
    "term": {
      "manufacturer.raw": "victory multimedia"
    }
  }
}
```

The `term` query is a low-level query in the sense that it
doesn\'t perform any analysis on the term. Also, it directly runs
against the inverted index constructed from the mentioned
`term` field; in this case, against
the `manufacturer.raw` field. By default, the `term`
query runs in the query context and hence calculates scores.

The response looks like the following (only the partial response is
included):

```
{
  ...
  "hits": {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
"max_score": 5.965414,
    "hits": [
      {
        "_index": "amazon_products",
        "_type": "products",
        "_id": "AV5rBfPNNI_2eZGciIHC",
"_score": 5.965414,
   ...
```

As we can see, each document is scored by default. To run the
`term` query in the filter context without scoring, it needs
to be wrapped inside a `constant_score` filter:

```
GET /amazon_products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "manufacturer.raw": "victory multimedia"
        }
      }
    }
  }
}
```

This query will now return results with a
score of one for all matching documents. We will look at the
`constant_score` query later in the lab. For now, you can
imagine that it turns a scoring query into a non-scoring query. In all
queries where we don\'t need to know how well
a document fits the query, we can speed up the query by wrapping it
inside `constant_score` with a `filter`. There are
also other types of compound queries that can help in converting
different types of queries and combining other queries; we will look at
them when we examine compound queries[*.*] 



Searching from the full text
----------------------------------------------



Full-text queries can work on unstructured
text fields. These queries are aware of the analysis process. Full-text
queries apply the analyzer on the search terms before performing the
actual search operation. That determines the right analyzer to be
applied by first checking whether a field-level
`search_analyzer` is defined, and then by checking whether a
field-level analyzer is defined. If analyzers at the field level are not
defined, it tries the analyzer defined at the index level.

Full-text queries are thus aware of the analysis process on the
underlying field and apply the right analysis process before forming
actual search queries. These analysis-aware queries are
also called **high-level
queries**. Let\'s understand how the high-level query flow
works.

 

Here, we can see how one high-level query on the
`title`field will be executed. Remember from our index
definition earlier that the `title` field is of
the `text` type. At indexing time, the value is analyzed using
the analyzer for the field. In this case, it was a Standard Analyzer,
and hence the inverted index contains all of the broken down terms, such
as **gods**, **heroes**, **rome**, and
so on, as depicted in the following figure:


![](./images/b43def01-15c4-4e0d-b6f5-ec81be749142.png)


Figure 3.3: High-level query flow

 

**At query time** (see the right-hand half of the figure), we
issue a `match` query, which is a high-level query. We will
cover `match` queries later in this section. The search terms
passed to a `match` query are analyzed using standard
analyzer. The individual terms after applying standard analyzer are then
used to generate individual term-level queries.

The example here results in multiple term queries---one for each term
after applying the analyzer. The original search term was **gods
heroes**, resulting in two terms, **gods** and
**heroes**, which are used as individual terms in their own
term queries. The two term queries are then combined using
a `bool` query, which is a compound query. We
will also look at different compound queries in the next section.

We will cover the following full-text queries in the following sections:


-   Match query
-   Match phrase query
-   Multi match query

### Match query



A `match` query is the default
query for most full-text search requirements. It is one of the
high-level queries that is aware of the analyzer used for
the underlying field. Let\'s get an
understanding of what this means under the hood.

For example, when you use the `match` query on
a `keyword` field, it knows that the underlying field is
a `keyword` field, and hence, the search terms are not
analyzed at the time of querying:

```
GET /amazon_products/_search
{
  "query": {
    "match": {
      "manufacturer.raw": "victory multimedia"
    }
  }
}
```

 

The `match` query, in this case, behaves just like a
`term` query, which we understand from the previous section.
It does not analyze the search term\'s `victory multimedia` as
the separate terms `victory` and `multimedia`. This
is because we are querying a `keyword` field,
`manufacturer.raw`. In fact, in this particular case, the
`match` query gets converted into a `term` query,
such as the following:

```
GET /amazon_products/_search
{
  "query": {
    "term": {
      "manufacturer.raw": "victory multimedia"
    }
  }
}
```

The `term` query returns the same scores as the
`match` query in this case, as they are both executed against
a `keyword` field.

Let\'s see what happens if you execute a `match` query against
a `text` field, which is a real use case for a full-text
query:

```
GET /amazon_products/_search
{
  "query": {
    "match": {
      "manufacturer": "victory multimedia"
    }
  }
}
```

When we execute the `match` query, we expect it to do the
following things:


-   Search for the terms `victory` and
    `multimedia` across all documents within the
    `manufacturer` field.
-   Find the best matching documents sorted by score in descending
    order.
-   If both terms appear in the same order, right next to each other in
    a document, the document should get a higher score than other
    documents that have both terms but not in the same order, or not
    next to each other.
-   Include documents that have either `victory` or
    `multimedia` in the results, but give them a lower score.


The `match` query with default parameters does all of these
things to find the best matching documents in order, according to their
scores (high to low).

By default, when only search terms are
specified, this is how the `match` query behaves. It is
possible to specify additional options for the `match` query.
Let\'s look at some typical options that you would specify:


-   Operator
-   Minimum should match
-   Fuzziness

#### Operator



By default, if the search term specified
results in multiple terms after applying the analyzer, we need a way to
combine the results from individual terms. As we saw in the preceding
example, the default behavior of
the `match`[* *] query is to combine the results
using the [*or*]  operator, that is, one of the terms has to
be present in the document\'s field.

This can be changed to use the `and` operator using the
following query:

```
GET /amazon_products/_search
{
  "query": {
    "match": {
      "manufacturer": {
        "query": "victory multimedia",
        "operator": "and"
      }
    }
  }
}
```

In this case, both the terms `victory` and
`multimedia` should be present in the document\'s
`manufacturer` field.

#### Minimum should match



Instead of applying the `and`
operator, we can keep the [*or*]  operator and specify at
least how many terms should match in a given document for it to be
included in the result. This allows for finer-grained control:

```
GET /amazon_products/_search
{
  "query": {
    "match": {
      "manufacturer": {
        "query": "victory multimedia",
        "minimum_should_match": 2
      }
    }
  }
}
```

The preceding query behaves in a similar way to the `and`
operator, as there are two terms in the query and we have specified
that, as the minimum, two terms should match. 

With `minimum_should_match`, we can specify something similar
to at least three of the terms matching in the document.

#### Fuzziness



With the `fuzziness` parameter, we can turn the`match` query into a `fuzzy` query. This
fuzziness is based on the Levenshtein edit distance, to turn one term
into another by making a number of edits to the original text. Edits can
be insertions, deletions, substitutions, or the transposition of
characters in the original term. The `fuzziness` parameter can
take one of the following values: `0`, `1`,
`2`, or `AUTO`.

For example, the following query has a misspelled
word, `victor` instead of `victory`.
Since we are using a `fuzziness` of `1`, it will
still be able to find all `victory multimedia` records:

```
GET /amazon_products/_search
{
  "query": {
    "match": {
      "manufacturer": {
        "query": "victor multimedia",
        "fuzziness": 1
      }
    }
  }
}
```

If we wanted to still allow more room for errors to be correctable, the
`fuzziness` should be increased to `2`. For example,
a `fuzziness` of `2` will even match
`victer`. `Victory` is two edits away from
`victer`:

```
GET /amazon_products/_search
{
  "query": {
    "match": {
      "manufacturer": {
        "query": "victer multimedia",
        "fuzziness": 2
      }
    }
  }
}
```

The `AUTO` value means that the `fuzziness` numeric
value of `0`, `1`, `2` is determined
automatically based on the length of the original term. With
`AUTO`, terms with up to 2 characters have
`fuzziness` = `0` (must match exactly), terms from 3
to 5 characters have `fuzziness` = `1`, and terms
with more than five characters have `fuzziness` =
`2`.

Fuzziness comes at its own cost because Elasticsearch has to generate extra terms to match against. To control the
number of terms, it supports the following additional parameters:


-   `max_expansions`: The maximum number of terms after
    expanding.
-   `prefix_length`: A number, such as 0, 1, 2, and so on. The
    edits for introducing `fuzziness` will not be done on the
    prefix characters as defined by the `prefix_length`
    parameter.



### Match phrase query



When you want to match a sequence of words, as opposed to separate terms
in a document, the `match_phrase` query can be useful.

For example, the following text is present as part of the description for one of the products:

```
real video saltware aquarium on your desktop!
```

What we want are all the products that have this exact sequence of words
right next to each other: `real video saltware aquarium`. We
can use the `match_phrase`[* *] query to achieve it.
The `match`[* *] query will not work, as it doesn\'t
consider the sequence of terms and their proximity to each other.
The `match`[* *] query can include all those
documents that have any of the terms, even when they are out of order
within the document:

```
GET /amazon_products/_search
{
  "query": {
    "match_phrase": {
      "description": {
        "query": "real video saltware aquarium"
      }
    }
  }
}
```

The response will look like the following:

```
{
  ...,
  "hits": {
    "total": 1,
    "max_score": 22.338196,
    "hits": [
      {
        "_index": "amazon_products",
        "_type": "products",
        "_id": "AV5rBfasNI_2eZGciIbg",
        "_score": 22.338196,
        "_source": {
          "price": "19.95",
          "description": "real video saltware aquarium on your desktop!product information see real fish swimming on your desktop in full-motion video! you'll find exotic saltwater fish such as sharks angelfish and more! enjoy the beauty and serenity of a real aquarium at yourdeskt",
          "id": "b00004t2un",
          "title": "sales skills 2.0 ages 10+",
          "manufacturer": "victory multimedia",
          "tags": []
        }
      }
    ]
  }
}
```

The `match_phrase`[* *] query also supports
the `slop` parameter, which allows you to specify an integer:
0, 1, 2, 3, and so on. `slop` relaxes the number of
words/terms that can be skipped at the time of querying.

For example, a `slop` value of `1` would allow one
missing word in the search text but
would still match the document:

```
GET /amazon_products/_search
{
  "query": {
    "match_phrase": {
      "description": {
        "query": "real video aquarium",
        "slop": 1
      }
    }
  }
}
```

A `slop` value of `1` would allow the user to search
with `real video aquarium` or
`real saltware aquarium` and still match the document that
contains the exact phrase `real video saltware aquarium`. The
default value of `slop` is zero.

### Multi match query



The `multi_match` query is an extension of
the `match` query. The `multi_match` query
allows us to run the `match` query
across multiple fields, and also allows many
options to calculate the overall score of the documents.

The `multi_match` query can be used with different options. We
will look at the following options:


-   Querying multiple fields with defaults
-   Boosting one or more fields
-   With types of `multi_match` queries


Let\'s look at each option, one by one.



#### Querying multiple fields with defaults



We want to provide a product search
functionality in our web application. When the end user searches for
some terms, we want to query both the `title`
and `description` fields. This can be done using
the `multi_match` query.

The following query will find all of the documents that have the terms
`monitor` or `aquarium` in the `title` or
the `description` fields:

```
GET /amazon_products/_search
{
  "query": {
    "multi_match": {
      "query": "monitor aquarium", 
      "fields": ["title", "description"]
    }
  }
}
```

This query gives equal importance to both fields. Let\'s look at how to
boost one or more fields.

#### Boosting one or more fields



In an e-commerce type of web application,  the user intends to search for an item, and they might search
for some keywords. What if we want the `title` field to be
more important than the description? If one or more of the search terms
appears in the `title`, it is definitely a more relevant
product than the ones that have those values only in the description. It
is possible to boost the score of the document if a match is found in a
particular field.

Let\'s make the `title` field three times more important than
the `description` field. This can be done by using the
following syntax:

```
GET /amazon_products/_search
{
  "query": {
    "multi_match": {
      "query": "monitor aquarium", 
      "fields": ["title^3", "description"]
    }
  }
}
```

The `multi_match` query offers more control regarding how to
combine the scores from different fields. Let\'s look at the options.

#### With types of multi match queries



In this section, we have learned about
full-text queries, which are also known as high-level queries. These
queries find the best matching documents according to the score.
High-level queries internally make use of some term-level queries. In
the next section, we will look at how to write compound queries.



Writing compound queries
------------------------------------------



This class of queries can be used to combine
one or more queries to come up with a more complex query. Some compound
queries convert scoring queries into non-scoring queries and combine
multiple scoring and non-scoring queries.

We will look at the following compound queries:


-   Constant score query
-   Bool query

### Constant score query



Elasticsearch supports querying both
structured data and full text. While full-text queries need scoring mechanisms to find the best matching documents,
structured searches don\'t need scoring. The constant score query allows
us to convert a scoring query that normally runs in a query context to a
non-scoring filter context[*. *] The constant score query is a
very important tool in your toolbox.

For example, a `term` query is normally run in a query
context. This means that when Elasticsearch executes a `term`
query, it not only filters documents but also scores all of them:

```
GET /amazon_products/_search
{
  "query": {
    "term": {
      "manufacturer.raw": "victory multimedia"
    }
  }
}
```

Notice the text in bold. This part is the actual `term` query.
By default, the `query` JSON element that contains the bold
text defines a query context[*.*] 

The response contains the score for every document. Please see the
following partial response:

```
{
  ...,
  "hits": {
    "total": 3,
    "max_score": 5.966147,
    "hits": [
      {
        "_index": "amazon_products",
        "_type": "products",
        "_id": "AV5rBfasNI_2eZGciIbg",
        "_score": 5.966147,
        "_source": {
          "price": "19.95",
   ...
}
```

Here, we just intended to filter the documents, so there was no need to
calculate the relevance score of each document.

The original query can be converted to run in a filter context using the
following `constant_score` query:

```
GET /amazon_products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "manufacturer.raw": "victory multimedia"
        }
      }
    }
  }
}
```

As you can see, we have wrapped the original highlighted
`term` element and its child. It assigns a neutral score of
`1` to each document by default. Please note the partial
response in the following code:

```
{
  ...,
  "hits": {
    "total": 3,
"max_score": 1,
    "hits": [
      {
        "_index": "amazon_products",
        "_type": "products",
        "_id": "AV5rBfasNI_2eZGciIbg",
"_score": 1,
        "_source": {
          "price": "19.95",
          "description": ...
      }
   ...
}
```

It is possible to specify a `boost` parameter, which will
assign that score instead of the neutral score of `1`:

```
GET /amazon_products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "manufacturer.raw": "victory multimedia"

        }
      },
"boost": 1.2
    }
  }
}
```

What is the benefit of boosting the score of
every document in this filter to `1.2`? Well, there is no
benefit if this query is used in an isolated
way. When this query is combined with other queries, using a query such
as a `bool` query, the boosted score becomes important. All
the documents that pass this filter will have higher scores compared to
other documents that are combined from other queries.

Let\'s look at the `bool` query next.

### Bool query



The `bool` query in Elasticsearch is your Swiss Army knife. It can help you write many types of
complex queries. If you are come from an SQL background, you already
know how to filter based on
multiple `AND`[* *] and `OR` conditions in
the `WHERE`[* *] clause. The `bool` query
allows you to combine multiple scoring and
non-scoring queries.

Let\'s first see how to implement simple `AND` and
`OR` conjunctions. 

A `bool` query has the following sections:

```
GET /amazon_products/_search
{
  "query": {
    "bool": {
      "must": [...],        scoring queries executed in query context
      "should": [...],      scoring queries executed in query context
      "filter": {},         non-scoring queries executed in filter context
      "must_not": [...]     non-scoring queries executed in filter context
    }
  }
}
```

The queries included in `must` and `should` clauses
are executed in a query context unless the whole `bool` query
is included inside a filter context.

The `filter` and `must_not` queries are always
executed in the filter context. They will always return a score of zero
and only contribute to the filtering of documents.

Let\'s look at how to form a non-scoring query that just performs a
structured search. We will gain an understanding of how to formulate the
following types of structured search queries using the `bool`
query:


-   Combining `OR` conditions
-   Combining `AND` and `OR` conditions
-   Adding `NOT` conditions

#### Combining OR conditions



To find all of the products in the price
range `10` to `13`, `OR` manufactured by
`valuesoft`:

```
GET /amazon_products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "should": [
            {
              "range": {
                "price": {
                  "gte": 10,
                  "lte": 13
                }
              }
            },
            {
              "term": {
                "manufacturer.raw": {
                  "value": "valuesoft"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

Since we want to `OR` the conditions, we have placed them
under `should`. Since we are not interested in the scores, we
have wrapped our `bool`[* *] query inside a
`constant_score` query.

#### Combining AND and OR conditions



Find all products in the price range
`10` to `13`, `AND` manufactured by
`valuesoft` or `pinnacle`:

```
GET /amazon_products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "price": {
                  "gte": 10,
                  "lte": 30
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "manufacturer.raw": {
                  "value": "valuesoft"
                }
              }
            },
            {
              "term": {
                "manufacturer.raw": {
                  "value": "pinnacle"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

Please note that all conditions that need to be ORed together are placed
inside the `should` element. The conditions that need to be
ANDed together, can be placed inside the `must` element,
although it is also possible to put all the conditions to be ANDed in
the `filter` element.

#### Adding NOT conditions



It is possible to add `NOT` conditions, that is, specifically
filtering out certain clauses using
the `must_not` clause in the `bool` filter. For
example, find all of the products in the price range `10` to
`20`, but they must not be manufactured
by `encore`[*. *] The following query will do just
that:

```
GET /amazon_products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "price": {
                  "gte": 10,
                  "lte": 20
                }
              }
            }
          ],
          "must_not": [
            {
              "term": {
                "manufacturer.raw": "encore"
              }
            }
          ]
        }
      }
    }
  }
}
```

The `bool`[* *] query with the `must_not`
element is useful for negate any query. To negate or apply
a `NOT` filter to the query, it should be wrapped inside the
`bool` with `must_not`, as follows:

```
GET /amazon_products/_search
{
  "query": {
    "bool": {
      "must_not": {
.... original query to be negated ...
      }
    }
  }
}
```

Notice that we do not need to wrap the query in a
`constant_score` query when we are only using
`must_not` to negate a query. The `must_not` query
is always executed in a filter context.

This concludes our understanding of the different types of compound
queries. There are more compound queries supported by Elasticsearch.
They include the following:


-   Dis Max query
-   Function Score query
-   Boosting query
-   Indices query


Besides the full-text search capabilities of Elasticsearch, we can also
model relationships within Elasticsearch. Due to the flat structure of
documents, we can easily model a one-to-one type of relationship within
a document. Let\'s see how to model a one-to-many type of relationship
in Elasticsearch in the next section.



Modeling relationships
----------------------------------------



We saw in the previous sections how to model
and store products and run various queries on products. The product data
had partly structured data and partly textual data. What if we also had
detailed features of the products available to us? We may have many
different types of products and each product may have completely
different types of detailed features. For example, for products that
fall into the **Laptops** category, we would have features
such as screen size, processor type, and processor clock speed.

At the same time, products in the **Automobile GPS Systems**
category may have features such as screen size, whether GPS can speak
street names, or whether it has free lifetime map updates
available.[] 

Because we may have tens of thousands of products in hundreds of product
categories, we may have tens of thousands of features. One solution
might be to create one field for each feature. As you may remember from
our earlier analogy between the field and database column, the resulting
data would look very sparse if we tried to show it in tabular format:

![](./images/3.PNG)


As you can see, products in the **Laptops** category have a
different set of columns populated (those columns are the features
related to that category) and products in the **GPS Navigation
Systems** category have a different set of columns populated.
If we were to model all products of all
categories like this, we may end up with tens of thousands of fields
(imagine tens of thousands of columns to make it a very wide table). If
the data was modeled in this way, it would be hard to generate certain
types of queries, as we have many different fields.

Instead, we could model this relationship between a product and its
features as a one-to-many relationship as we would in a relational
database. Let\'s see how we would have modeled it in a relational
database.

The product table would be modeled as follows:

![](./images/4.PNG)

 

The features would be modeled as a separate table, where the ProductID
and Feature may be a composite primary key:

![](./images/5.PNG)

 
When we model a similar type of relationship in Elasticsearch, we can
use the `join` datatype
(<https://www.elastic.co/guide/en/elasticsearch/reference/7.x/parent-join.html>)
to model relationships. To import the data,
follow the steps mentioned in
[lab-03/products\_with\_features\_data](https://github.com/fenago/elasticsearch/tree/v7.0/lab-03/products_with_features_data).

Here, we want to establish a relationship between
products[* *] and features. When using the `join`
datatype, we still need to index everything into a single Elasticsearch
index within a single Elasticsearch type. Remember, we can\'t have more
than one type within a single index. The `join` datatype
mapping that establishes the relationship is defined as follows:

```
PUT /amazon_products_with_features
{
  ...
  "mappings": {
    "doc": {
      "properties": {
        ...
"product_or_feature": {
          "type": "join",
          "relations": {
            "product": "feature"
          }
        },
        ...
       }
     }
   }
}
```

The highlighted `product_or_feature`field is of
type `join` and it defines the relationship
between `product` and `feature`[***.***] 
The product is on the left-hand side, which is analogous to conveying
that the `product` is the parent of the feature(s).

When indexing `product` records, we use the following syntax:

```
PUT /amazon_products_with_features/doc/c0001
{
  "description": "The Lenovo ThinkPad X1 Carbon 20K4002UUS has a 14 inch IPS Full HD LED display which makes each image and video appear sharp and crisp. The Thinkpad has an Intel Core i5 6200U 2.3 GHz Dual-core processor with Intel HD 520 graphics and 8 GB LPDDR3 SDRAM that gives lag free experience. It has a 180 GB SSD which makes all essential data and entertainment files handy. It supports 802.11ac and Bluetooth 4.1 and runs on Windows 7 Pro 64-bit downgrade from Windows 10 Pro 64-bit operating system. The ThinkPad X1 Carbon has two USB 3.1 Gen 1 ports which enables 10 times faster file transfer and has Gigabit ethernet for network communication. This notebook comes with 3 cell Lithium ion battery which gives upto 15.5 hours of battery life.",
  "price": "699.99",
  "id": "c0001",
  "title": "Lenovo ThinkPad X1 Carbon 20K4002UUS",
  "product_or_feature": "product",
  "manufacturer": "Lenovo"
}
```

The value of `product_or_feature`, which is set
to `product` suggests that this document is referring to a `product`.

A `feature` record is indexed as follows:

```
PUT amazon_products_with_features/doc/c0001_screen_size?routing=c0001
{
  "product_or_feature": {
    "name": "feature",
    "parent": "c0001"
  },
  "feature_key": "screen_size",
  "feature_value": "14 inches",
  "feature": "Screen Size"
}
```

Notice that, while indexing a `feature`, we need to set which
is the parent of the document
within `product_or_feature`[*.*]  We also need to
set a routing parameter that is equal to the document ID of the parent
so that the child document gets indexed in the same shard as the
parent. Please follow the instructions at
[lab-03/products\_with\_features\_data](https://github.com/fenago/elasticsearch/tree/v7.0/lab-03/products_with_features_data) to
load some sample data, that has products and features.

Once we have some products and features populated, we can query from
products while joining the data from features. For example, you may want
to get all of the products that have a certain feature. 



### has\_child query



If you want to get products based on some condition on the features, you can use `has_child` queries.

The outline of a `has_child` query is as follows:

```
GET <index>/_search
{
  "query": {
    "has_child": {
      "type": <the child type against which to run the following query>,
      "query": {
        <any elasticsearch query like term, bool, query_string to be run against the child type>
      }
    }
  }
}
```

As you can see in the preceding code, we mainly need to specify only two
things:


1.  The type of the child against which to execute the query
2.  The actual query to be run against the child to get their parents


Let\'s learn about this through an example. We want to get all of the products where
`processor_series` is `Core i7` from the example
dataset that we loaded:

```
GET amazon_products_with_features/_search
{
  "query": {
    "has_child": {
      "type": "feature",                                     1
      "query": {
        "bool": {                                            2
          "must": [
            {
              "term": {                                      3
                "feature_key": {
                  "value": "processor_series"
                }
              }
            },
            {
              "term": {                                      4
                "feature_value": {
                  "value": "Core i7"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

Please note the following points about the query:


-   The `has_child` query is used to query based on the child
    type `feature`.
-   The actual query to be used on the child type is specified under
    the `query` element. We can use any query supported by
    Elasticsearch under this `query` element. Whichever query
    is used will be run against the child type that we specified. We are
    using a `bool` query because we want to filter by all features
    where---`feature_key` =
    `processor_series`[* *] **AND `feature_value`
    = `Core i7`[*.*] 
-   We have used a `bool` query, as explained in the previous
    point. The first of the conditions is included as a `term`
    query here under the `must` clause. This `term`
    query checks that we filter for `feature_key` =
    `processor_series`[*.*] 
-   This is the second condition under
    the `must`[* *] clause, in which the
    `term` query checks that we filter
    for `feature_value` = `Core i5`.


The result of this `has_child` query is that we get back all
the products that satisfy the query mentioned under
the `has_child` element executed against all the features. The
response should look like the following:

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "amazon_products_with_features",
        "_type" : "doc",
        "_id" : "c0002",
        "_score" : 1.0,
        "_source" : {
          "description" : "The Acer G9-593-77WF Predator 15 Notebook comes with a 15.6 inch IPS display that makes each image and video appear sharp and crisp. The laptop has Intel Core i7-6700HQ 2.60 GHz processor with NVIDIA Geforce GTX 1070 graphics and 16 GB DDR4 SDRAM that gives lag free experience. The laptop comes with 1 TB HDD along with 256 GB SSD which makes all essential data and entertainment files handy.",
          "price" : "1899.99",
          "id" : "c0002",
          "title" : "Acer Predator 15 G9-593-77WF Notebook",
          "product_or_feature" : "product",
          "manufacturer" : "Acer"
        }
      }
    ]
  }
}
```

Next, let\'s look at another type of query that can be utilized when we
have used the `join` type in Elasticsearch.

### has\_parent query



In the previous type of query, `has_child`,[* *] we
wanted to get records of the `parent_type` query based on a
query that we executed against the child type. The
`has_parent` query is useful for exactly the opposite purpose,
that is, when we want to get all child type of records based on a query
executed against the `parent_type`. Let\'s learn about this
through an example.

We want to get all the features of a specific product that has the
product id = `c0003`. We can use a `has_parent`
query as follows:

```
GET amazon_products_with_features/_search
{
  "query": {
    "has_parent": {
      "parent_type": "product",                                  1
      "query": {                                                 2
        "ids": {
          "values": ["c0001"]
        }
      }
    }
  }
}
```

Please note the following points, marked with the numbers in the query:


1.  Under the `has_parent` query, we need to specify
    the `parent_type` against which the query needs to be
    executed to get the children (features).
2.  The `query` element can be any Elasticsearch query that
    will be run against the `parent_type`. Here we want
    features of a very specific product and we already know the
    Elasticsearch ID field (`_id`) of the product. This is why
    we use the `ids` query with just one value in the array:
    `c0001`.


The result is all the features of that product:

```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "amazon_products_with_features",
        "_type" : "doc",
        "_id" : "c0001_screen_size",
        "_score" : 1.0,
        "_routing" : "c0001",
        "_source" : {
          "product_or_feature" : {
            "name" : "feature",
            "parent" : "c0001"
          },
          "feature_key" : "screen_size",
          "parent_id" : "c0001",
          "feature_value" : "14 inches",
          "feature" : "Screen Size"
        }
      },
      {
        "_index" : "amazon_products_with_features",
        "_type" : "doc",
        "_id" : "c0001_processor_series",
        "_score" : 1.0,
        "_routing" : "c0001",
        "_source" : {
          "product_or_feature" : {
            "name" : "feature",
            "parent" : "c0001"
          },
          "feature_key" : "processor_series",
          "parent_id" : "c0001",
          "feature_value" : "Core i5",
          "feature" : "Processor Series"
        }
      },
      {
        "_index" : "amazon_products_with_features",
        "_type" : "doc",
        "_id" : "c0001_clock_speed",
        "_score" : 1.0,
        "_routing" : "c0001",
        "_source" : {
          "product_or_feature" : {
            "name" : "feature",
            "parent" : "c0001"
          },
          "feature_key" : "clock_speed",
          "parent_id" : "c0001",
          "feature_value" : "2.30 GHz",
          "feature" : "Clock Speed"
        }
      }
    ]
  }
}
```


### parent\_id query



In the `has_parent` query example, we already knew the ID of
the parent document. There is actually a handier query to get all
children documents if we know the ID of the parent document.

You guessed correctly; it is the `parent_id` query, where we
obtain all children using the parent\'s ID:

```
GET /amazon_products_with_features/_search
{
  "query": {
    "parent_id": {
      "type": "feature",
      "id": "c0001"
    }
  }
}
```

The response of this query will be exactly the same as
the `has_parent`[* *] query we wrote earlier. The
key difference between the `has_parent`[* *] query
and the `parent_id` query is that the former is used to get
all children based on an arbitrary Elasticsearch query executed against
the `parent_type` query, whereas the latter
(`parent_id` query) is useful if you know the ID.

Covering all types of queries is beyond the scope of this book. Having
learned different types of full-text queries, compound queries, and
relationship queries in depth, you are now well equipped to try other
queries that aren\'t covered here. Please refer to the Elasticsearch
reference documentation to learn about their usage.



Summary
-------------------------



In this lab, we took a deep dive into the search capabilities of
Elasticsearch. We looked at the role of analyzers and the anatomy of an
analyzer, saw how to use some of the built-in analyzers that come with
Elasticsearch, and saw how to create custom analyzers. Along with a
solid background on analyzers, we learned about two main types of
queries---term-level queries and full-text queries. We also learned how
to compose different queries in more complex queries using one of the
compound queries.

This lab provided you with sound knowledge to get a foothold on
querying Elasticsearch data. There are many more types of queries
supported by Elasticsearch, but we have covered the most essential ones.
This should help you get started and enable you to understand other
types of queries from the Elasticsearch reference documentation.

In Lab 4, [*Analytics with Elasticsearch*], we will learn
about the analytics capabilities of Elasticsearch. With that lab
under your belt, we will conclude by learning the core components of the
Elastic Stack and Elasticsearch, and you will be well-equipped to
understand the other components of the Elastic Stack.
