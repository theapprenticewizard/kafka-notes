# Notes on Kafka

## Kafka Terminlology

* __record__ Are a record of an event, they are immutable.  They are append only and are made of a key value and timestamp.  Records are persisted to DISK and not to memory! It uses the OS's caching system instead of an explicit cache.
* __producers__ produces events for a topic.
* __broker__ represents a node in a kafka cluster, uses lead-follower for cluster distribution.
* __consumers__ read events from a topic
* __topic__ a logical name for a group of _partitions_
* __partition__ in a clustered environment (which is basically every non-dev kafka environment a partition is what gets replicated across brokers (nodes)
