---
author: 'Raymond Dinkin'
user: 'theapprenticewizard'
created-on: 2019-03-31
---

# Elastic Search



## Architecture

### 1. Indices

_An Index_ is a type of collection of documents that can be searched.  Each index can have many documents.  Previously there were types as well - however types are now deprecated.  Coming from a MongoDB background, indices are mostly just a 

### 2. Types [deprecated]

Although, types are now officially deprecated they still remain in some older implementations of elastic search.  Types were once a way to think about nested documents inside of a single index.  As in you were once able to have two types inside of the same index as indices can be expensive and aggregating across indices may have been complicated.  However, now documents can easily be nested and the barrier for querying across indices is no longer as high.  Therefor types have become confusing hence their deprecation.  

The second part after of the path of a resource in elastic search denotes the type.  However, since types are deprecated you should use only `_doc` instead to denote it as a document type (as types, being deprecated are optional).

example.  ` POST /people/_doc { "name" : "Raymond" }`

Note: after the index name used to be the type name such as `/products/type_name` however that is no longer the case.  Instead you access documents using the `_doc` prefix.  such as `/products/_doc/1`

### 3. Clustering

Elastic search is a clustered application meaning that more than one physical server can host an elastic search application.  An elastic search cluster is made of the following parts.

​	a.) __Cluster:__ A name for the whole application spread across multiple nodes. 

​	b.) __Node:__ A server instance responsible for executing queries and managing _shards_.

​	c.) __Shard:__ A portion of the data, as say 1TB may not fit into single physical server it will be sharded and distributed across multiple noted.  This as well will increase horizontal scalability as each node can process their own shard in parallel. Sharding seems to work in a similar fashion to a RAID0 configuration, however it is possible for shard replicas to increase availability and fault tolerance as well through read replication. 

​	d.) __Replicas and Replica Groups:__ Replicas are read-only copies of a primary shard.  A replica may not be on the same node as the shard it's replicating.  A replica is updated by the primary of it's own _replica group_ after the primary has completed handling the update request. Replicas greatly increase high availability and scalability of an elastic search cluster. 

## The Elastic Stack

The elastic stack is made up of 5 main components.  Those components are...

### Elastic Search

Duh.

### Logstash

Allows for uniform ingestion into an elastic search cluster, meaning it allows for the processing of data before it gets sent into elastic search.  It has connection from multiple sources.

### Kibana

An elastic search visualization tool as well as sandbox and management UI.  Kibana does not add any 'server' level features.  It is simply a web server that acts as a much nicer way to interact with elastic search.  It's configuration is stored on the elastic search cluster that it connects to. 

### Beats

Beats are pre-configured ingestion plugins for elastic search.  Beats allow for collecting logs and metrics automatically, or with very little hassle from the user.  An example of a beat is File Beat which collects logs from files and sends them to elastic search.

### Elastic Extensions [formerly XPack]

Extensions for elastic search that add new features that enhance elastic search itself.  Some come pre-bundled with elastic search now!  Used to be closed source and a way to reward enterprise customers for using elastic search through the elastic company. But now it's fully open soured.

## Installation and Configuration

Download [here](https://www.elastic.co/downloads/elasticsearch) or use the following `docker-compose.yml`.  If installing on windows it's recommended to install elastic search as a `service` so you can have automatic startup and manage it though the dedicated `services` window;

For docker, if installed; simply run `docker-compose up` to bring up the service and `docker-compose down` to take down the service as well.  This will expose elastic search at the standard port of  `9200` and Kibana at `5601`.

```yml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.0
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
  kibana:
    image: docker.elastic.co/kibana/kibana:6.7.0
    ports:
      - "5601:5601"
```

## Managing documents

Elasticsearch uses a RESTful API to manage documents, so in order to create, read, update or delete you need to make am HTTP request, each HTTP method's semantic meaning is used (except patch).

### Managing Indexes

#### Create

In order to add a new index all you need to do is send an HTTP PUT request to a new URL.

```http
PUT /new_index

# response
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "new_index"
}
```

#### Delete

In order to delete an index simply perform a `DELETE` request on the same path you created the index on.

```http
DELETE /new_index

# response
{
  "acknowledged" : true
}
```

#### CRUD for ducuments

#### Create

In order to create a documents simply just `POST` the document into to elastic search, do note.  The second path variable is the _type_ of the document you wish to create, however - as mentioned above it is __deprecated__.  Instead of posting to a custom type name instead have the second path variable be `_doc` to indicate it is a document.  Another note, when you `POST` a new document to an index that hasn't been created yet  - it will automatically create the index.  If this behavior is not desirable you can simply disable it in the settings.

```http
POST people/_doc
{
  "name" : "Raymond",
  "age" : 24
}

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "NpK61GkBCMHpv4E0rPH0",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

#### Read

In order to read a single document (without performing a query) you'll want to simply make a `GET` request. And use the id of the document as a path variable.

```http
GET people/_doc/NpK61GkBCMHpv4E0rPH0

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "NpK61GkBCMHpv4E0rPH0",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Raymond",
    "age" : 24
  }
}
```

#### Updating

Updating can happen in two forms - however it's important to note that event though updating with PATCH semantics exists - it's "more" thread safe but it's not entirely thread safe as the PATCH semantics update mechanism in elastic search simply fetches the document save a new version and re-indexes it just as if we made a PUT request. However, this is generally safe - as it occurs without the overheard of the network, decreasing the odds of a concurrent update breaking our system.

for details concerning concurrency read the following [guide in the docs](<https://www.elastic.co/guide/en/elasticsearch/guide/current/concurrency-solutions.html>)

##### PUT semantics

Updates the version of the document as well as the source!  Example...

```http
PUT people/_doc/NpK61GkBCMHpv4E0rPH0
{
  "name" : "Someone Else",
  "age" : 48
}

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "NpK61GkBCMHpv4E0rPH0",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Someone Else",
    "age" : 48
  }
}
```



##### PATCH semantics

Also updates the version (as it's doing the same thing internally!)

```http
POST people/_doc/NpK61GkBCMHpv4E0rPH0/_update
{
  "doc" : {
    "age" : 33
  }
}

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "NpK61GkBCMHpv4E0rPH0",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

##### PATCH Semantics with scripting

Important to note that the script is just a text field.  As well, to update the "source object" (more on this later) you need to specify `ctx._source` first, with the assumption that `ctx` is short for _context_ as there needs to be some object to specify!

```http
POST people/_doc/NpK61GkBCMHpv4E0rPH0/_update
{
  "script" : "ctx._source.age += 5"
}

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "NpK61GkBCMHpv4E0rPH0",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

#### Upserting

If you want to insert or update a document where you're not sure if the document is yet to exist than you want to perform an upsert.  Note, in the below example if you try to update using the script a document that doesn't have the field necessary to update, even though the field is in your upsert it will cause an exception.  Be careful of this! 

```http
POST people/_doc/NpK61GkBCMHpv4E0rPH0/_update
{
  "script" : "ctx._source.age += 1",
  "upsert" : {
      "age" : 10
  }
}

 # response
 {
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "NpK61GkBCMHpv4E0rPH0",
  "_version" : 5,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 9,
  "_primary_term" : 1
}

```

and here's the same request with nothing at that id.

```http
POST people/_doc/new_id/_update
{
  "script" : "ctx._source.age += 1",
  "upsert" : {
      "age" : 10
  }
}

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "new_id",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

# here's the state of the new document
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "new_id",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "age" : 10
  }
}
```

#### Deleting Documents

In order to delete a document simply just make a `DELETE` request to the URL of the the document you've created.

example

```http
DELETE people/_doc/new_id

# response
{
  "_index" : "people",
  "_type" : "_doc",
  "_id" : "new_id",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

##### Deleting By Query

```http
POST /people/_delete_by_query
{
  "query" : {
    "match" : {
      "name" : "Raymond"
    }
  }
}

# response
{
  "took" : 41,
  "timed_out" : false,
  "total" : 1,
  "deleted" : 1,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```



#### Batch Ops

##### Create

For creating documents in bulk (rather than making an http request machine gun) use the bulk API as shown like this.

```http
POST /people/_doc/_bulk
{ "index" : { "_id" : 100 } }
{ "name" : "Raymond", "age" : 24 }
{ "index" : { "_id" : 101 } }
{ "name" : "Other Person", "age" : 27 }

# response
{
  "took" : 6,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "people",
        "_type" : "_doc",
        "_id" : "100",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "people",
        "_type" : "_doc",
        "_id" : "101",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```

##### Update

It's also possible to perform batch updates as well, these have PATCH semantics!

```http
POST /people/_doc/_bulk
{ "update" : { "_id" : 100 } }
{ "doc" : { "age" : 55 } }
{ "update" : { "_id" : 101 } }
{ "doc" : { "age" : 58 } }

# response
{
  "took" : 0,
  "errors" : false,
  "items" : [
    {
      "update" : {
        "_index" : "people",
        "_type" : "_doc",
        "_id" : "100",
        "_version" : 3,
        "result" : "noop",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "status" : 200
      }
    },
    {
      "update" : {
        "_index" : "people",
        "_type" : "_doc",
        "_id" : "101",
        "_version" : 2,
        "result" : "noop",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "status" : 200
      }
    }
  ]
}
```

## Mapping

Elasticsearch uses mapping to determine the type of a field of a document. This mapping by default is _dynamic_ however sometimes it is necessary to provide manual mappings of a document.  However, it's important to note once an index has used a document with a given field mapped, you cannot re-map the field to another type.  Therefor it is important to determine at index creation the types of index mapping you want to use (if you want to override the defaults).  Using dynamic mapping can be a way to the increase performance of a cluster or decrease the amount of bits used to store the information.  Or, as well - it can be used to store more semantic types.

### Determine mapping of an existing index

to view the mapping of an existing index you need to access the _mapping route of an index. Note the `include_type_name` by default will be false in the future as types are deprecated.

```http
GET /people/_doc/_mapping?include_type_name

# response
{
  "people" : {
    "mappings" : {
      "_doc" : {
        "properties" : {
          "age" : {
            "type" : "long"
          },
          "name" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}

```

### Meta Fields

In Elastic search there exists many fields that hold meta-values.  These are values that don't directly apply to the object itself.  In other document databases (like MongoDB) these meta fields are stored in the ID as a hash. However, as Elastic Search's indexing is quite a bit more complicated (being a search engine!) the fields that contain the meta data are explicitly.  That allows us to also use any type of ID we want instead of just using a custom UUID (like MongoDB).

Some common meta-fields on all documents are as follows. 

#### _index

Stores the index that the document lives in.

#### _type

The type of the document, though types are deprecated so it will likely forever be `_doc`

#### _id

The id either specified by the user or generated by elastic search. When generated by elastic search a UUID/GUID is used of a custom format.

#### _version

The current version of the data, this should basically only be used for optimistic locking updates.

#### _source

Contains the document that was provided directly.  This is basically the the pure document that you are create/read/updating/deleting without the extra overhead of indexing.

#### _routing

Stores how the document should be routed by shard.

#### _meta

Contains general meta information, this can be used as an application specific store of non-source data in the database.  Perhaps data like create_by or tenant.

### Types

Elastic search has a wide variety of types, most of them should be familiar if you've coded with any programming language before.  Some of the common types that are available are.

a list of compatible data types can be found [here](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/mapping-types.html)

#### Text Data (_text_)

Text data is indexed by the search engine in order to perform searches. Usually this data represents a block of text such as a blog post or a product description.

#### Keywords (_keyword_)

Keywords are _not_ analyzed, instead they are used for match queries. Generally they are small pieces of text such as a product category.

#### Numeric Types 

float, long, short, integer, double, byte, scaled float and half float. 

where scaled float is a float that is stored in the database in scientific notation. IE.  3.4 with scaled factor of 2 is 3.4 * 10^2 which is 340. This allows us to store arbitrarily precise floating point numbers efficiently. Half float is just a float with half the amount of bits available (16 instead of the standard 32).

#### Dates (_date_)

despite the name, dates can be dates or date-times.  Dates can be dynamically mapped in many ways, including mapping to more than one way at a time.  For example you can map a date by epoch_ms AND an ISO date string at the same time.  Elastic search has multiple ways of mapping date times built in.

#### Binary Data (_binary_)

Elasticsearch can keep track of BLOBs in order to store complex serialized data if so required.  This can be such things like images or other data.  Obviously you cannot index or analyze BLOB data, that would be too inefficient. Likely you'll want to attach some metadata to the BLOB in the same object.

#### Range Data Types (_range_)

In elastic search it is possible to store a range as a type.  An example of a range is below.

```json
{ "gte" : 100, "lte" : 200 } // a number between 100 and 200 inclusive
```

#### Objects (_object_)

Objects are actually kind of fake inside of elastic search, as nesting is not efficient in most cases. When you use an object (which can have their own  types) they are simply mapped to a simple single line with `.` separators.

example:

```json
"person" : { "name" : "Raymond", "age" : 24 } 
```

In the above example name will be mapped to "person.name" and age will map to "person.age". like the following.

`"person.age" : 24`

#### Arrays as not a type

In elastic search all types are arrays, and so each value of each key can inherently contain more than one value.  However the types must remain constant.  So you can't mix numbers and strings for example.

__IMPORTANT NOTE__ because arrays of objects have no inherent meaning the association of the values will get lost.

IE. 

```json
"people" : [
    {
        "name" : "Raymond",
        "age" : 24
    },
    {
        "name" : "other guy",
        "age" : 35
    }
]

// will turn into the following mapping

"people.name" : [ "Raymond", "other guy" ],
"people.age" : [ 24, 35 ]
```

#### Nested Data Types (_nested_)

Instead of directly nesting objects Elasticsearch has a better concept for handling nested objects (as most document database naturally would).  Nested documents are "hidden" and separately indexed, as well they require nested queries to be looked up. 

#### Geographic Data Types

It is possible to store geographic information inside of elastic search. Geo data types have many subtypes

##### Geo Point (_geo_point_)

Can be represented by a geo hash or latitude or longitude. as well there are other ways to specify lat and long such as an array or a single string. 

examples:

```json
{
    "location" : {
        "lat" : 33.23413,
        "lon" : 24.23425
    }
}

// or

{
    "location" : "asdfa3223dsdd"
}
```

##### Geo Shapes (geo_shape)

Can store a polygon containing information to define a set of coordinates as a polygon area. 

#### Specialized Data Types

Elastic Search has several specialized data types for storing useful information in an optimized way these include

##### IP Addresses (_ip_)

Elastic search can store IP Addresses as a native type, they can be either IPV4 or IPV6 this is likely very useful for logging IP addresses rather than using DNS for logging as that can change.

##### Completion Data  Type (_completion_)

Provides search as you type data (similar to google) auto-complete is pretty cool IMO. Usually stores a small amount of data, as it needs to have super efficient lookup!

##### Attachments (_attachment_)

Stores an index of an index, this is analyzed by the search engine. Uses Apache Tikka for lexical analysis. Attachment support is provided by a plugin.

### Mapping Existing Indices

To change the mapping of a field simply make a call to the `_mapping` route and `PUT` the mappings you wish to add.  Note, as mentioned before this will cause an error if there is already a mapping created for that key! There are some exceptions to this rule, however - you may update the index of an object inside of a document if it hasn't been added yet. (as objects aren't really real in Elasticsearch)

#### Updating an existing Index

```http
PUT /products/_doc/_mapping
{
    "properties" : {
        "discount" : {
            "type" : "double"
        }
    }
}

# response
{
  "acknowledged" : true
}
```

#### Updating the type of an existing field

You can't update the type of an existing field in an index as mentioned above.  However, you can delete the index and create a new one.

### Setting a new index with manual mapping

When you create an index it is possible to set the new index manually. And disable the automatic mapping here is an example of how to do that. 

```http
PUT /products
{
    "mappings" : {
    	"default" : {
            "dynamic" : false,
            "properties" : {
                "in_stock" : {
					"type" : "text"
				},
				"is_active" : {
                    "type" : "boolean"
				}
            }
    	}
    }
}

# response
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "products"
}

```

### Common Mapping Parameters

As seen above the `mapping` object can take many different kinds of parameters.  though, it's not necessary to memorize all of these a few of them will come in handy in many cases.

#### Coerce (_coerce_)

can be used to force type coercion on a mapping for example, "5" can be mapped to the integer 5.  Without type coercion non-valid types are ignored.

#### Copy To (_copy_to_)

can be used to create a read-only copy of the data that is added. For example, if you have a first name and a last name and you want to provide a name field based on the first name + the last name separated by a space.

```json
{
    "first_name" : {
        "type" : "text",
        "copy_to" : "full_name"
	},
    "last_name" : {
        "type" : "text",
        "copy_to" : "full_name"
	},
    "full_name" : {
        "type" : "text"
	}
}
```

#### Dynamic (_dynamic_)

Disabled or enables (on by default though) dynamic mapping to determine the types used automatically.

#### Properties (_properties_)

Shows the properties of a given document.

```json
"properties" : {
	"in_stock" : {
		"type" : "text"
	},
    "is_active" : {
    	"type" : "boolean"
	}
}
```

#### Store Normalization Factors (_norms_)

Norms are used to calculate relevancy, this property determines if norms should be added for a given text field.  This is on by default.

```json
"properties" : {
	"description" : {
        "type" : "text",
        "norms" : false
	}
}
```

#### Format (_format_)

Used for specifying a format, formats or custom format for usually dates. By default for dates, the format will allow either `epoch_millis` or `strict_date_optional_time`

```javascript
"properties" : {
	"timestamp" : {
        "type" : "date",
        "format" : "strict_date_optional_time||epoch_millis"
	}
}
```

#### Null Value (_null_value_)

Allows for determining the null value that a document will coalesce to for example, if a number is null you can have it be 0 by default. 

```javascript
"properties" : {
	"in_stock" : {
        "type" : "long",
        "null_value" : 0
	}
}
```

#### Fields (_fields_)

Used to specify how to store a field in another way.  using "fields" allows us to specify that a property has more than one type. This is often the case for text fields as they sometimes should be indexed as keywords.

```javascript
"properties" : {
	"tags" : {
        "type" : "text",
        "norms" : false,
        "fields" : {
            "keyword" : {
                "type" : "keyword"
            }     
        }
	}
}
```

### Custom Date Formats

It is possible to specify custom date formats as well in elastic search. The following is used to support Yoda date formats. (Year mOnth DAte)

```http
PUT /products/_doc/_mapping
{
    "properties" : {
        "created" : {
            "type" : "date",
            "format" : "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd"
        }
    }
}
```

### Updating Mappings By Query

It's not always required that you have a mapping for each object.  Sources, for example are just sometimes stored in text with no mapping provided yet.  So it's entirely possible to update all indices providing an explicit mapping after data has been provided for a mapping that hasn't existed yet.  To do this you can perform a query like so.

```http
POST /products/_update_by_query?conflicts=proceed
{
    "query" : {
        "term" : {
            "discount" : 20
        }
    }
}
```

## Text Analysis

When text data is added to elastic search, it goes through an analyzer.  These analyzers standardize and index the data for searching. this allows for the search engine to function efficiently. The analyzers also perform normalization on the data as well in order to allow for a consistent format to be scanned.  The default analyzer, which is acceptable in most cases takes all the text and tokenizes it into terms and then stores them in the "inverted index" in order to be accessed later by the search engine.

### The analysis process

An analyzer is made up of three components, a character filter, a token filter and a tokenizer. The default tokenizer is called the "standout" tokenizer and ignores most special characters.  In addition to removing special characters the tokenizer also marks the position of each token in a stream of text. Which is used for proximity searches and fuzzy finding. After the character filter occurs, the token filter will perform it's job. The most common token filter is the lowercase filter which simply converts all of the text into lowercase, as the case of the search generally doesn't matter. Secondly, another use case for token filters is the "stop" filters which removes stop words from the token stream.  Examples of these words are prepositions, "the", "at", "around" as they don't usually have any semantic meaning for searching things. Finally, another token filter that is common is the "synonym" filter, its used to mark tokens that have the same meaning.  For example "great" may have the same meaning as "awesome" and so forth, so searching by "awesome" may increase the likelihood of finding a document with the keyword "great" being found.

### Using the Analyzer API

You can analyzer a piece of text and view the generated tokens from text by using the analyzer API.  The main reason for doing this is to check some sample text against different analyzers, allowing you to mix and match filters in your search engine.

```http
POST /_analyze
{
  "tokenizer": "standard", 
  "filter" : ["lowercase"],
  "text" : "Hello World!"
}

# response
{
  "tokens" : [
    {
      "token" : "hello",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "world",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```

You can also provide a whole analyzer like so.

```http
POST /_analyze
{
    "analyzer" : "standard",
    "text" : "Lorem Ipsum set Dolor"
}

# response
{
  "tokens" : [
    {
      "token" : "lorem",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "ipsum",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "set",
      "start_offset" : 12,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "dolor",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 3
    }
  ]
}
```



 ### What is the Inverted Index?

An inverted index is an index holding the information of what tokens are available within a full text field of a document. Having an inverted index allows the search engine to determine things like relevancy or if the search matches a specific search term. The data stored in the inverted index can be one of either of two things.  Either meta-data regarding terms, such as the number of times "pizza" is shown in total across an index under the "product_description" field. Or, it can also mean the table of mentions of pizza for each document in the index.  For example, in one description, pizza could be mentioned 5 times, that would mean that pizza is more relevant to that specific products.  This however, can be a problem due to "keyword stuffing" if users get to choose their own search keyword, but hopefully there are tweaks you can make in the analyzer for that.

### Common Character Filters

#### HTML Strip Character Filter (_html_strip_)

Strips out html tags, and turns escaped Unicode characters into a normalized form.

#### Mapping Character Filter (_mapping_)

Replaces one character or set of characters with another set of characters.  This is applied with a dictionary of characters to interchange. For example, for all text occurrences of the word "banana" you can have it replaced with "potato".

#### Pattern Replace Filter (_pattern_replace_)

Replaces a set of characters using a regular expression. For example, this could be used for a profanity filter as well. And can replace the profanity with a friendly version of the text.  This will also allow you to use capture groups as well!

### Common Tokenizers

There are three kinds of tokenizers: Word oriented tokenizers, Partial Word Tokenizer and Structured Text Tokenizers. 

#### Word Oriented Tokenizers

Word oriented tokenizers work at the word level and allow for human-readable searches. 

##### Standard Tokenizer (_standard_)

Separates whitespace into tokens and is the default tokenizer provided for search.

##### Letter Tokenizer (_letter_)

Divides texts into tokens by looking for letters exclusively runs of letters become tokens.  For example I'm becomes two tokens because an apostrophe does not constitute as a letter.

##### Lowercase tokenizer (_lowercase_)

Performs the same as the letter tokenizer but lowercases as it goes. 

##### Whitespace Tokenizer

Tokenizes based on whitespace, will preserve special characters however such as periods and exclamation points!

##### UAX URL Email Tokenizer (_uax_url_email_)

Performs the same as the standard tokenizer but preserves the details of an email.

#### Partial Word Tokenizers

Breaks words into pieces for indexing.

##### N-Gram Tokenizer (_ngram_)

Tokenizes based on an N-Gram which allows for words to be broken up.  The N-Gram tokens contain the original words as well as partial pieces of the word based on a sliding window that is based on the "N" you set.  For example an N-Gram of the word "hello" will become [ "he", "el", "lo", "hel", "hell", "ell", "ello" "hello"] as an N-Gram of 2.  This is great if you want to map partial words such as morphemes.

##### Edge N-Gram Tokenizer (_edge_ngram_)

Breaks text into words when encountering certain characters then splits those up using an N-Gram that always starts from index.  This creates fewer tokens than a standard n-gram. Example. [ "he", "hel", "hell", "hello"] would be an n-gram value of two which is less than the above example.

#### Structured Text Tokenizers

Structured text tokenizers work with structured texts such as zip codes, IP addresses and more.

##### Keyword Tokenizer (_keyword_)

A no-op tokenizer used for keywords (obviously) such as "health insurance" rather than tokenizing everything. 

##### Pattern Tokenizer (_pattern_)

A tokenizer that tokenizers based on a given regular expression's capture group.

##### Path Tokenizer (_path_hierarchy_)

Tokenizes text based on a standard path hierarchy, useful for file paths.

### Common Token Filters

#### Standard Token Filter (_standard_)

Doesn't do anything, for future uses.  This is also the default token filter.

#### Lowercase Token Filter (_lowercase_)

Turns the tokens all into lowercase.

#### Uppercase Token Filter (_uppercase_)

Turns all the tokens into uppercase.

#### NGram token filter (_nGram_)

Does the same things as the nGram token filter, but does it at a later layer in order to make the api as flexible as possible.

#### Edge NGram Token Filter(edgeNGram)

Does the same thing as the tokenizer with the same name, for the same reason as the nGram token filter.

#### Stop Token Filter (_stop_)

Removes "stop words" such as prepositions and articles. This has support for MOST languages!

#### Word Delimiter Token Filter (_word_delimiter_)

Turns words into sub words such as turning WiFi into [ "wi", "fi"]

#### Stemmer Token Filter (_stemmer_)

Stems words into their own form, for example. driving will be stemmed into drive.

#### Keyword Marker Filter (_keyword_marker_)

Protects specified words from being stemmed.

#### Snowball Token Filter (_snowball_)

Applies the _snowball_ stemming algorithm to your inverse index.

#### Synonym token filter (_synonym_)

Is used to apply synonyms as part of the filter. Synonyms are stored at the same level for phrase searches.

#### Trim Filter (_trim_)

Trims whitespace

#### Length Filter (_length_)

Filters out words that are too long or too short.

#### Truncate Filter (_truncate_)

Truncates words that are too long

### Analyzers

#### Standard Analyzer (_standard_)

Is the default analyzer, removes punctuation and separates text using a letter character filter.

#### Simple Analyzer (_simple_)

Uses the character token filter, and lowercases all of the letter.s

#### Stop Analyze (_stop_)

Works the same way as the stop analyzer but also  removes stop words. (like the stop token filter)

#### Language Analyzers (_english... and other ones_)

Preconfigured analyzers for specific languages including synonym support.  This is probably the easiest, most powerful one.

#### Keyword Analyzer (_keyword_)

Doesn't do anything! Used for determining keywords.

#### Pattern Analyzer (_pattern_)

Splits items into tokens based on a regex.

#### Whitespace Analyzer

Analyzes text by just tokenizing based on the whitespace character filter.  Doesn't do anything really!

### Setting a Custom Analyzer

```http
PUT /existing_analyzer_config
{
  "settings" : {
    "analysis": {
      "analyzer": {
        "english_stop" : {
          "type" : "standard",
          "stopwords" : "_english_"
        }
      },
      "filter" : {
        "my_stemmer" : {
          "type" : "stemmer",
          "name" : "english"
        }
      }
    }
  }
}

# response
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "existing_analyzer_config"
}

```

##### And testing our custom analyzer

```http
POST /existing_analyzer_config/_analyze
{
  "analyzer": "english_stop",
  "text" : "I'm in the mood for drinking semi-dry wine today!"
}

# response
{
  "tokens" : [
    {
      "token" : "i'm",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "mood",
      "start_offset" : 11,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "drinking",
      "start_offset" : 20,
      "end_offset" : 28,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "semi",
      "start_offset" : 29,
      "end_offset" : 33,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "dry",
      "start_offset" : 34,
      "end_offset" : 37,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "wine",
      "start_offset" : 38,
      "end_offset" : 43,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "today",
      "start_offset" : 44,
      "end_offset" : 49,
      "type" : "<ALPHANUM>",
      "position" : 9
    }
  ]
}

```



## Searching with Elastic Search

### Performing a query

````http
 GET /products/_doc/_search
 {
     "query" : {
         "match" : {
             "description" : "Red Wine"
         }
     }
 }
````

#### Query String Queries

```http
GET /products/_doc/_search?q=Red+Wine
```

It's also possible to use an asterisk to match all documents. Using simply `*`

```http
GET /products/_doc/_search?q=*
```

It's also possible to add Boolean logic to your queries. Like this.

```http
GET /products/_doc/_search?q=tag:potato AND description:banana
```

Finally it's also possible to analyze just a single document to determine how it is searched. Here's an example.

```http
GET /products/_doc/1/_explain
{
  "query" : {
    "term" : {
      "name" : "lobster"
    }
  }
}
```

### Contexts

When dealing with elastic search queries you need to keep in mind that there are two different ways to search for something.  

#### Query Context

Is used when you want to search something that is indexed as a text document, mostly all textual data should go under this - and this is the main feature of the search engine.

#### Filter Context

Filtering is when you want to filter out results, but you don't want to effect relevancy.  For example you can filter out items that contain "tomato" in the search result. Only items with a "tomato" match will be shown but, how many times or how relevant to "tomato" the texts are will not effect the search results.  As an additional bonus filters can be cached while queries cannot.  (this is because queries have much more complicated logic going on!)



### Term Queries vs Match Queries (full text)

#### Term Queries

Term queries perform a textual search but do not apply an analyzer to the query term itself.  This makes for a faster, but sometimes unexpected query as things like synonyms or case case sensitivity will not be processed.  (if the search index potato, and you searched Potato - it may not match!)

Example:

```http
GET /products/_doc/_search
{
  "query" : {
    "term" : {
      "name" : "Lobster"
    }
  }
}
```



#### Match Queries

Perform a match on the target data, this will use the analyzer to make the query so whatever analyzer that is applied to the _inverse index_ of a search field will intern be applied to the query.



Example.

```http
GET /products/_doc/_search
{
  "query" : {
    "match" : {
      "name" : "lobster"
    }
  }
}
```



