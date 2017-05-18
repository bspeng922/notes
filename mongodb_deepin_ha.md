# MongoDB 深入Sharding架构 


> MongoDB distributes data, or shards, at the collection level. Sharding partitions a collection’s data by the shard key.
> 
> To shard a collection, you need to select a shard key. A shard key is either an indexed field or an indexed compound field that exists in every document in the collection. MongoDB divides the shard key values into chunks and distributes the chunks evenly across the shards. To divide the shard key values into chunks, MongoDB uses either range based partitioning or hash based partitioning. 
> 



