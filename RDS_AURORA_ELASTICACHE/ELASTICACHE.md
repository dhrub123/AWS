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
  up with an invalidation strategy, so that only the most current and most relevant data is in our cache. This pattern is also called lazy loading.
  
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
  + So Redis can also be used as a database.
  + To get a always available redis cluster, use multi az feature, not read replicas
+ Memcached 
  + It is very different. It uses multiple-node for partitioning of data, which is called sharding. 
  + This is a non persistent cache, so if our Memcached node goes down, then the data is lost. There is no backup and restore features, and it's a multi-threaded 
    architecture. So Memcached is around sharding. A part of the cache is going to be on the first shard, and another part of the cache is going to be on the 
    second shard, and each shard is a Memcached node.
  + Memcached is a pure cache that lives in memory without any backup and restore, no persistence, multi-threaded architecture and revolves around Sharding.

#### Elasticache Hands On
+ Go to Elasticache service > Cluster Engine (Memcached or Redis(Cluster Mode)) - Redis Settings(name, desc, port, parameter group, read replica) - Advanced Redis
  Settings(VPC, Subnet, Security - Encryption at rest and transit(Redis Auth generates token which is used by service to login to redis), Backups, maintenance) 
  > Create
+ We then use the primary endpoint for using it in App

#### Elasticache Security

+ All caches in ElastiCache support SSL in flight encryption 
+ **They do not support IAM authentication.**
+ IAM policies on ElastiCache are only used for AWS API-level security which includes creating a cluster, deleting a cluster, updating the configurations etc.
+ Redis now has authentication and it's called Redis AUTH. We can create something called a password or a token when we create a Redis cluster. This is an extra 
  level of security for your cache on top of security groups. So any client that does not have this password or this token will not be able to connect to our 
  Redis cluster and will be rejected. This is called Redis AUTH.
+ By default, Redis does not have any AUTH and anything or anyone can connect to our Redis cluster.
+ So it is important to use security groups as an extra level of security for our cache to ensure that only the networks we have authorized can access our 
  Redis cluster.
+ From Memcached, there is SASL-based authentication
+ EC2 clients for example or applications running on EC2 are running inside of an EC2 security group and they want to connect to our Redis cluster cache
  and therefore there is going to be a Redis security group around the cache and we are going to make sure that the Redis security group allows the 
  EC2 security group in. And also our clients will be using SSL encryption for encryption of data in transit. And as an extra level of security for Redis, 
  we can enable Redis AUTH to make sure that the EC2 clients will have the right password before connecting to our Redis cache.
  
#### Elasticache patterns

There are three patterns.
+ **Lazy Loading** - We read all the data and once all the data that has been read they are automatically cached. Some data can become stale in the cache.
+ **Write Through** - We add or update data in the cache when it is written to a database, in this case we have no stale data.
+ **Session Store** - We can store temporary session data in the cache. We can use a TTL feature on top of it to expire data in your Session Store.
