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





