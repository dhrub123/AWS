## AWS Elasticache - managed cache

We get managed cache of Redis and Memcached using ELasticache the same way we get managed database using RDS.

+ Caches are basically in-memory databases, so they run on RAM, and have really high performance and really low latency.
+ They help reduce the load off of databases, by caching data. The read-intensive workloads read from the cache, instead of reading from the database.
+ It also helps make a bunch of application stateless, by storing states in a common cache.
+ It has a Write Scaling capability using sharding and it has a Read Scaling capability using Read Replicas.
+ It has Multi AZ capability with failover.
+ AWS takes care of OS maintenance, patching, optimizations, setup, configuration, monitoring, failure recovery and backups.
+ It is an RDS for caches, and it is called ElastiCache.
+ Things to remember : Write Scaling, Read Scaling, and Multi AZ.

#### How does it work ?

+ **DB Cache**
  We have our application and it communicates to RDS. But we're also going to include an ElastiCache. Our application will first query ElastiCache.
  And if what we query for is not available, then we'll get it from RDS and store it in ElastiCache. We will call this cache hit, when you get into ElastiCache, 
  and it works. So we have an application, it has a cache hit, and we get the data straight from ElastiCache. In that case, the retrieval was fast, and RDS did 
  not see a thing. But sometimes our application requests data, and it doesn't exist in cache. This is a cache miss. So when we get a cache miss, our application
  needs to go ahead and query the database, and write the result back to the cache. So if another application, or the same application will ask for the same 
  query, it will be cache hit. So the cache just caches data and help relieve the load in RDS, usually the read load, definitely. And the cache must also come 
  up with an invalidation strategy, so that only the most current and most relevant data is in our cache.
  
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/DB_CACHE.png" width="50%" height="50%"/>
  
+ **User Session Store**
  A user logged in to our application, and our application is stateless, so that means there is a bunch of applications running and all of them need to know 
  that the user is logged in. So the user logs in to one of the application, and then the application will write the session data into ElastiCache.
  Now the user hits another instance of our application, Then that application needs to know that our user is logged in. And for this, it's going to retrieve 
  the session off of Amazon ElastiCache and find out if there is a session exisiting for the user. All the instances can retrieve this data and make sure 
  the user doesn't have to re-authenticate every time. This pattern relieves load off a database, and shares some states, such as the user session store 
  into a common ground, so that all the applications can be stateless and yet retrieve and write these sessions in real time.
  
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/USER_SESSION.png" width="50%" height="50%"/>
  
#### Difference between memcached and redis

+ Redis
  + Redis is going to have a Multi AZ feature, so we can have it in multiple availability zone, with Auto-Failover feature. So if one AZ is down, we can failover 
    automatically to another one. 
  + We can also scale our reads by creating Read Replicas so we have more reads and high availability. 
  + We can enable Data Durability using AOF persistence. If our cache is stopped and then restarted, we can still have our data that was in the cache before 
    stopping it, and this is available to us because of AOF persistence. So we can backup, and restore our Redis clusters. 
  + In Redis, remember two instances, one being the primary, the second being the replica and data persistence, backup and restore and similar to RDS.
  + Redis has more industrial RDS type features, multi AZ and revolves around replication using Read replicas
+ Memcached 
  + It is very different. It uses multiple-node for partitioning of data, which is called sharding. 
  + This is a non persistent cache, so if our Memcached node goes down, then the data is lost. There is no backup and restore features, and it's a multi-threaded 
    architecture. So Memcached is around sharding. A part of the cache is going to be on the first shard, and another part of the cache is going to be on the 
    second shard, and each shard is a Memcached node.
  + Memcached is a pure cache that lives in memory without any backup and restore, no persistence, multi-threaded architecture and revolves around Sharding.
