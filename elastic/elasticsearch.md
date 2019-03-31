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

### _routing

Stores how the document should be routed by shard.

#### _meta

Contains general meta information, this can be used as an application specific store of non-source data in the database.  Perhaps data like create_by or tenant.

### Types

Elastic search has a wide variety of types, most of them should be familiar if you've coded with any programming language before.  Some of the common types that are available are.

#### Text Data (_text_)

Text data is indexed by the search engine in order to perform searches. Usually this data represents a block of text such as a blog post or a product description.

#### Keywords (_keyword_)

Keywords are _not_ analyzed, instead they are used for match queries. Generally they are small pieces of text such as a product category.

#### Numeric Types 

float, long, short, integer, double, byte, scaled float and half float. 

where scaled float is a float that is stored in the database in scientific notation. IE.  3.4 with scaled factor of 2 is 3.4 * 10^2 which is 340. This allows us to store arbitrarily precise floating point numbers efficiently. Half float is just a float with half the amount of bits available (16 instead of the standard 32).

#### Dates (_date_)

despite the name, dates can be dates or date-times.  Dates can be dynamically mapped in many ways, including mapping to more than one way at a time.  For example you can map a date by epoch_ms AND an ISO date string at the same time.  Elastic search has multiple ways of mapping date times built in.

### Binary Data (_binary_)

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

