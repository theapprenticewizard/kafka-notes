# Notes on Kafka

## Kafka Terminlology

* __record__ Are a record of an event, they are immutable.  They are append only and are made of a key value and timestamp.  Records are persisted to DISK and not to memory! It uses the OS's caching system instead of an explicit cache.
* __producers__ produces events for a topic.
* __broker__ represents a node in a kafka cluster, uses lead-follower for cluster distribution.
* __consumers__ read events from a topic
* __topic__ a logical name for a group of _partitions_
* __partition__ in a clustered environment (which is basically every non-dev kafka environment a partition is what gets replicated across brokers (nodes) ordering is guranteed per partiion
* __offset__ a sequential id per partition, the consumers themselves should check the offsets. Can be the 'break off point' determining when kafka will stop storing records of a partition. 
* __leader-follower__ There must always exist a leader of a partition, everyone else (followers) will replicate the records that are saved by the leader. This very similar to "write masters" in MongoDB, and is generally a common theme in clustering/sharding.
* __consumer groups__ allow for load-balancing of consumer groups, it enables exactly-once delivery if required (not always existant).  It essence, it says "hey, we're doing the same thing with the same message, I don't want to do the same work as the other consumer!"
* __delivery gurantees__ Async (no gurantees, really fast!) ... Commited to Leader (the leader did deliver it), Commited to Leader and Quorum (the leader said it got it, Quorum votes that most people received it) - you can see diagnostic info on confluent console.
* __consumer gurantess__ At least once, At most once, Effectively Once (most common) Exactly Once (not recommended in most cases) be careful about transactions!

## Kafka Features
Log Compaction - will optionally compress records or logs into smaller chunks.  Store more stuff on disk!, this is good for some use cases.
Everything is stored on disk - Excplicity caching is lame, why use heap? Your OS will cache anyways.
Pagecache to Socket - Everthing doesn't neccisarily need to go 'through' kafka, makes for some natural caching.
Balanced Particions and leaders - Make sure that one broker is doing all of the work!  Internally uses Zookeeper.
Producer/Consumer quotas. 
Has managed services with it. 

## Clients
* JVM is the official clien
