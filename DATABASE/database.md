## Databases

+ Choosing the right database is tough because there's a lot of manage databases on AWS to choose from and you need to ask yourself the right questions.
+ These questions come from the AWS white paper
	+ Is your workload more read-heavy like a lot of reads or write-heavy a lot of writes or is it more of a balanced workload? What are your throughput needs? Maybe you need high throughput or low throughput.
	  Will it change? Will it fluctuate over time or will it need to scale over time? 
	+ How much data do you store and for how long? Will it grow and what's your average object size? Is it really small? is it really big? Is it average? And how is your data accessed?
    + Is there some security needs around it?
    + Data durability, do you need your data to be there for a week or forever? Is your database going to be a source of truth for all your data sets?
    + What are the latency requirements? How many users at a time will you get?
    + And what's the data model? How will you query your data? Do you query by primary key? Do you join? Is it structured? Is it semi-structured? Do you need to search it?
    + And do we need strong schema or more flexibility? Do we need reporting, search? Do you want it to be RDBMS or NoSQL?
    + And finally is there any license cost? Can you switch to a Cloud Native database such as Aurora in the case of license cost say on Oracle?
  These questions will help you steer your decision towards the right database.
+ Database types on AWS
	+ RDBMS, that means you're using SQL and its OLTP so online transaction processing and this is usually going to be your principal database like RDS if you going to use Postgres, MySQL, MariaDB,
	  MySQL server, Oracle and Aurora if you wanted to jump to a more enterprise grade database. And so these databases are great to power a website, they're great for joins, they're great for normalized data.
	  Basically any time you see data in a tabular form, RDS and Aurora are gonna be the way to go.
	+ Then there's NoSQL databases and there are many on AWS. There is DynamoDB which stores sort of like JSON documents, ElastiCache for key value pairs, high performance. Neptune, is for graphs and so in these 
	  databases we can't really do joins and there's no SQL language to query your database so it's more of a way to organize your data that's going to be different and you get performance benefits out of it.
	+ ObjectStore : Maybe you want to have objects such as Object Store so S3 is gonna be here for big objects so on DynamoDB , you can only store maybe 400 kilobytes of data per row whereas on the S3 you can go 
	  up to five terabytes of data per object so its really really big. And then Glacier is going to be for backups and archives but you can still store data there as well. So S3 and Glacier they don't seem like databases
	  at the first glance but they actually are databases because we put data on them and we retrieve it
	+ Data warehousing so this is when you wanna do analytics and BI based on SQL and so for this we can use Redshift, Redshift is going to be an OLAP so online analytical processing and so this is basically
	  helping you doing your data warehousing and all that stuff. And Athena , it can be used to query your data in S3 and so it can be considered as a data warehouse for analytics and BI purposes.
	+ Search capability - ElasticSearch(JSON) where you can like search around free text, unstructured searches so it has a lot of good querying and search capabilities as the name indicates.
	+ Graphs - Neptune may be considered as a graph database and so it will be used to display relationships between data.

#### RDS

+ RDS is managed PostgreSQL, MySQL, Oracle, MariaDB, SQL Server
+ We must provision an EC2 instance and EBS Volume type and size, so that happens behind the scenes, Amazon does that for us. We tell Amazon the instance type we want, and the EBS Volume type and size.
  Then, Amazon provides us a database and we don't have direct access to this underlying EC2 instance, but this is what happens in the back-end.
+ Now this has support for Read Replicas and Multi AZs, so Read Replica allows us to scale reads, Multi AZ allows us to save and recover from disaster.
+ And security is done through IAM, Security Groups, KMS for encryption at rest, and SSL for encryption in transit. and sometimes, Postgres and MySQL will have IAM authentication as a feature.
+ There is also a feature of doing backups, snapshots, point in time recovery feature. 
+ There is managed and scheduled maintenance so our database is kept up-to-date by AWS.
+ The monitoring can happen through CloudWatch 
+ Use cases would be RDBMS/OLTP, so to store relational data sets. So remember OLTP, Online Transactional Processing. And then, from it, we can perform SQL queries, ad-hoc queries basically, and we can do transactions.
  So, transactional inserts, updates, deletes, is an available feature.
+ RDS from a solutions architect perspective
	+ A well architectured framework has five pillars and we will compare RDS to the five pillars.
	+ Operations -  we have a very small downtime when a failover happens, and when maintenance happens. And when we scale reads, or EC2 instance, and EBS restore that implies manual intervention. So that means that we still have
	  to do a few operations on our RDS databases. And then when there is a change, maybe we have to do an application change as well.
	+ In terms of security, AWS will be responsible for OS security, for the EC2 instance security, but we are responsible for letting know to use KMS, to configure the security groups correctly, to set up the IAM policies correctly,
	  and authorizing users in our database, and enforcing SSL encryption.
	+ In terms of reliability, there is this multi AZ feature, and it's done automatically for us, so there will be a failover in case of failures. And that makes RDS particularly reliable. Now, if you don't use multi AZ,
	  then you have a risk obviously, of having some outages.
	+ Performance will depend basically on the EC2 instance type you provisioned for your RDS instance, and as well as the EBS volume type, so do you want to use a gp2 or io1? And then if you want to scale and perform some reads,
	  then you need to add Read Replicas. And RDS, in that case, doesn't auto-scale. It's not something that's super cloud native, it's still something that we have to provision and scale manually, and basically, adapt based on our workload.
	+ In terms of cost, it is very simple, you're going to pay per the hour, based on the provisioned EC2 instance type, and the EBS volume.

#### Aurora

+ Aurora is a database managed by AWS, it's under the RDS service for now and the API is compatible for either PostgreSQL or MySQL so we get compatibility for these two type of databases.
+ The data though is very different, it's held six replicas across three AZ so there is a lot of durability for our data and it has some auto healing capabilities
+ We do not exactly know about the underlying architecture of Aurora but we know it's really really good and there is some kind of auto healing capabilities so even if we lose one replica , Aurora and Amazon will automatically recover that 
  for us.
+ There is multi AZ available by default and auto scaling of read replicas so it gives as less operations to do and we get seemingly better performance
+ The read replicas can be global which is really nice and you can also have a global Aurora database which can be used for disaster recovery purposes or latency decrease purposes.
+ Overall the auto scaling of storage is a really nice feature on Aurora so no need to worry about sizing the storage and auto scaling can go from 10 gigabytes all the way to 64 terabytes of data so it's a consistent sizable amount of data
  that we can have in Aurora.
+ We can define an EC2 instance type still for the Aurora instances which we can change that in the future.
+ Then we get the same security, monitoring and maintenance features as RDS.
+ Finally there is this option for Aurora Severless which removes all the need to manage servers.
+ Overall the use case is gong to be the same a RDS but here you are looking for more at a enterprised grade database maybe replacing Oracle and you want less maintenance more flexibility and more performance but it's gonna be a little 
  bit pricier. Overall AWS is pushing for Aurora to be used instead of RDS instances to create a enterprise grade application.
+ From a solutions architect perspective based on the five pillars of the well architected framework
	+ Operations - Compared to RDS, we get less operations and there is auto scaling of storage so we basically don't need to think too much about aurora, we just set up the aurora database and setup auto scaling for maybe read
	  replicas and we are done. We will have a database that will not need many operations.
	+ In terms of security, well we have the same security for RDS and we can authorize basically users in database , use SSL same as RDS.
	+ Reliability - it's more reliable, it's multi AZ, highly available, we have six replicas of the data so it's possibly more highly available than RDS and then we even have Aurora Serverless option if you want to have even more reliable
	  and less operations type of option.
	+ Performance - AWS Aurora boasts five X performance due to the architectural optimization they've done around how they designed Aurora and we can have up to 15 read replicas verses five for RDS so seemingly we get much better perfomance
	  and much better scaling on aurora than we do for RDS.
	+ Finally for cost we still pay per hour based on the EC2 instance, type of operation and the storage usage but it is possibly a much lower cost compared to an enterprised grade database such as Oracle for even better performance.
+ Amazon is pushing for Aurora to be used on RDS because aurora is less maintenance more performance more reliability and possibly similar costs or a little bit ahead of cost but probably worth all the performance increase.

#### Elasticache

+ ElastiCache is Managed Redis or Memcached. ElastiCache is for cache what RDS is for Postgres, for example. 
+ A cache is an in-memory data store, and has really, really good performance. We're talking about sub-millisecond latency, so it's really high performance.
+ It works similar to RDS. We need to provision an EC2 instance type. 
+ We have support for Clustering so it's called Multi AZ, Read Replicas as in RDS it's called Clustering in Redis and Read Replicas is provided through these clustering. It's called sharding in the ElastiCache world.
+ Security is done through IAM, Security Groups, KMS, and there is no IAM authentication to the database, but we can use something called Redis Auth if we talk to Redis.
+ Now there's still the feature of Backup, Snapshot, and Point in time restore if we wanted to
+ We get Managed and Scheduled maintenance.
+ All the monitoring is done through CloudWatch, so very similar to RDS.
+ We would use ElastiCache not as a SQL database. It is an in-memory Key/Value store. And so it has very different Use Cases. We can use it as a Key/Value store, when we have frequent reads and less writes.
  Maybe want to cache database results for queries, so it's called a write-through pattern. Maybe we want to store session data for websites. And we cannot use SQL in ElastiCache, it's really here to improve performance,
  or used as a cache, So it's very different from a data base. It's Key/Value store, so that means that you retrieve data by key. You can't query data on fields.
+ From a Solutions Architect perspective
	+ Operations are exact same as RDS
	+ Security, the same thing as RDS, except this time we don't get IAM authentication to ElastiCache. We get Redis Auth if we wanted to.
	+ Reliability, we will get this Clustering feature, sharding feature, and Multi AZ.
	+ Performance, it's really good for a cache with sub-millisecond performance, in memory, and you have read replicas for sharding. So, it's a very popular cache option. So, if you see sub-millisecond performance, at the exam,
	  in memory, think ElastiCache.
	+ The cost is going to be a similar pricing as RDS, So, we're going to pay per hour based on the EC2 instance type, that we provision and the storage usage. 

#### DynamoDB

+ DynamoDB is pure cloud-based technology. It's proprietary to AWS.
+ It is a managed, NoSQL database and it is serverless, you provision the capacity, you get auto scaling on top of it, if you want it to, and now you can get on demand capacity.
  So it really scales, based on your workload, and gives you optimal pricing and performance.
+ It can be a replacement for ElastiCache, as a key value store for storing session data. Since it's serverless, you don't have to worry about provisioning ElastiCache and maintaining it.
  But you do not get sub-millisecond performance on DynamoDB, you get single digit, so between 1 and 9 millisecond of performance, which is good and that makes it a very acceptable
  keyvalue store for storing session data.
+ It's highly available, Multi AZ by default. The Reads and the Writes are decoupled, so you can provision read capacity units and write capacity units, and if you want to have a caching 
  technology in front of it, you can enable DAX, which is DynamoDB accelerated cluster, and that will give you a read cache, which will improve the performance and decrease the number of 
  reads that actually happen on the DynamoDB database itself.
+ Because it's a distributed database, you have two options for reads, either you choose eventual consistence, or strong consistence. That means, are you willing to accept all data,
  stale data or do you want to get always the newest data.
+ Security, authentication and authorization, is all done through IAM. So it gives you a one stop shop to manage security on DynamoDB.
+ If you wanted to integrate with AWS Lambda, you can use DynamoDB Streams to stream all the changes that happen in DynamoDB all the way over to your function, and react, based on some 
  events that happened.
+ You get a backup and restore feature, just like any database on AWS, and you even have this feature of a Global Table but only if you've enabled DynamoDB Streams.
+ You get monitoring integration with CloudWatch
+ One downside of DynamoDB is that you can only query on primary key, sort key or indexes. This means that you need to be very careful when you start designing your DynamoDB tables,
  and it does require some practice as a developer. 
+ Usecase : We use it when we have serverless application development and when the actual rows are small, we're talking less than 400 kilobytes, and we want maybe, a distributed, serverless 
  cache. It does not have the SQL query available, and now it has transactions capability. 
+ From a Solutions Architect's perspective and the five pillars
	+ Operations - You don't need to do any operations, if you enable auto scaling or if you enable on demand, and it's serverless, so we do not worry about managing servers.
	+ Security - It is done through IAM policies, you can enable KMS encryption at rest, and you get SSL in flight, which means it is a one stop shop for you to manage all your access to your tables,
	+ Reliability - You have Multi AZ and backups, which means that you really will survive AZ going down and if you want it to have some disaster recovery, well you can enable backups,
	  and restore from these backups later on.
	+ Performance, is single digit millisecond performance, and that's why you can use it as a cache in some ways, and if you wanted to enable an actual cache, for your database reads,
	  then you can have DAX as well. DynamoDB, is made so that performance won't degrade if your application scales. So DynamoDB can scale to hundreds of terrabytes in your database, and you will still 
	  get that really good, single digit millisecond performance.
	+ Cost - You're going to pay for the provision capacity and storage usage, and so if you use auto scaling, you don't guess in advance the capacity. It will scale down if you don't need it,
	  and scale up if you do need it, so in the end, you can expect to cause a cost to be paid for exact usage, because if you have an application that does nothing, you don't pay much, if it does a lot,
	  you pay more, but at least you don't need to think about provisioning in advance.

#### S3

+ S3 is not a conventional database but it is a database because we put data in S3 and we retrieve it.
+ S3 is a key value store for objects, and so that means that when we do create a key, an object, you know it's a long key, the full path to the object, and then the value itself is the object itself.
+ The recommendation on S3 is that it's great for big objects, big files, because there is more latency. It's not so great if you have many small objects, so in that case S3 does not replace RDS,
  or it dos not replace DynamoDB, because in RDS and DynamoDB you get small objects.
+ But if you want to have infinite storage, serverless, and you want to have object size up to five terabytes, then S3 is a great database for these things.
+ You're gonna query maybe a movie, so you're gonna store all your movies there. It's gonna be your movie database, and maybe each movie's gonna be 500 megabytes.
+ We get eventually consistency for writes, overwrites, and deletes, but for new objects, we get strong consistency.
+ In terms of tiers, we have S3 Standard, S3 IA for Infrequently Accessed, One Zone IA and then we can use Glacier for backups.
+ We get many features like Versioning, Encryption, Cross Region Replication, and so on.
+ Security, it's all managed through IAM, but on top of it we can add Bucket Policies and ACLs at the object level, and for encryption we get SSE-S3, SSE-KMS, SSE-C, and client side encryption,
  and for in transit we can use SSL or HTTPS.
+ We use S3 for static files, but also as a key value store for big files or website hosting.
+ From a solutions architect perspective
	+ Operations - We don't need to do anything. It's there, it's available, it's all the time here. You don't need to provision servers.
	+ Security - It is up to you to manage it - so please define your IAM policies correctly, your Bucket Policies, your ACL, make sure that encryption is done correctly 
	  based on your requirements on server and clients, and then make sure you're using SSL for encryption in flight.
	+ Reliability is huge, so we have 99.999999 durability and 99.99 availability, so its makes it a really reliable store for your data. You also have Multi AZ, so by default
	  all you did is replicated across Multi AZ, and you get CRR for Cross Region Replication, if you wanted to put all your Bucket contents into another region just in case.
	+ Performance is amazing, you can scale to thousands of reads and writes per second. You can get transfer acceleration if you use CloudFront, and you use multi-part for big files to make sure 
	  they're reliably put into S3.
	+ For cost, you're going to only pay for the storage you actually use, so you don't need to think about how much storage you want to provision. That makes S3 an infinite storage store,
	  and then you're only going to pay as well for network cost, so the bandwidth to transfer and retrieve the data, and then finally, if you do a lot of requests on S3, you're going to get billed for that as well.


#### Athena

+ Athena is not a database in the terms that it holds data, but it does provide a query engine on top of S3, so you can see it as the SQL layer on top of S3.
+ AWS advertises it as a fully serverless query engine. It has SQL capabilities so you can start querying your data in S3.
+ You're going to pay for each query you run based on how much capacity you're using and how much data you're analyzing etc. and you can output the results back optionally to S3.
+ It's all secured through IAM so you have to ensure that you have access to the bucket you're querying or the data you're querying.
+ Use Case is going to be for one time SQL query - so exploratory work and maybe serverless queries on S3 or log analytics, that kind of stuff. So if you do light weight queries
  which are not too complicated, not too many joins, Athena is a great candidate
+ From a Solutions Architect's perspective
	+ Operations: It's serverless so no operations.
	+ Security is IAM + S3 security, so there is an S3 security component in there usually through bucket policies.
	+ Reliability: It's a managed service and it uses Presto as an engine, which is a very high performance engine and on top of it, all the queries are done in a
	  highly available fashion, so you're pretty much sure that they will succeed.
	+ Performance: The queries will scale based on the data size, so you expect a big data chunk to be analyzed and Athena will basically scale accordingly.
	+ Cost, you're going to pay per query which are per terabyte of data scanned, so that means that you only pay only for the actual usage and that's what makes it as well, a serverless offering.
+ So you can use Athena to analyze extra logs, ELB logs, VPC logs and it's quite a cool query engine to use SQL on top of S3.

#### Neptune

+ Neptune is a fully managed graph database. Graph database are a little bit new and different kind of database. 
+ We use graphs when you have a high relationship data like social networking, for example, Users are friends with other Users, they reply to your comment on the post of a user
  and likes other comments. So this is kind of data that's really highly linked in a whole graph. Another good graph that we know about is Wikipedia because in every Wikipedia article,
  you have links to other Wikipedia articles and that makes a giant graph.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DATABASE/images/GRAPH.png" width="60%" height="60%"/>
  All this is graph data because all these things are linked together and gives us a massive graph.
+ Neptune is when you see graphs at the exam.
+ Neptune is highly available across 3 AZ and you get up to 15 read replicas.
+ You also get point-in-time recovery and continuous backup to S3.
+ It supports obviously KMS encryption and HTTPS.
+ From a Solutions Architect's perspective
	+ Operations is similar to RDS.
	+ Security is similar to RDS, but you also get IAM authentication to Neptune.
	+ Reliability is great. You get Multi-AZ and clustering.
	+ Performance is at its best when you use it for storing graph data and you use clustering as well to improve performance.
	+ Cost is similar to RDS. You basically provision the node and you pay for them.

#### ElasticSearch

+ Elasticsearch is an open source technology and it is a service on AWS.
+ In DynamoDB you can only find data when it has primary key or has indexes that you create on top of it. So, you can only find data for the exact match of the primary key and the 
  exact match of indexes, and that makes doing queries on DynamoDB very optimized. But if you're doing a search, then it's not really flexible and good at all.
+ Elasticsearch is meant for search. We can search any field and you can even search for partial matches. So, if your name is John but you just look for Jo, the first two letters of John,
  its going to return you a match as well.
+ So it's a really complimentary to use Elasticsearch as a compliment to another database.
+ You would use maybe DynamoDB to store all your data, but then to provide a search capability in your application you would use something like Elasticsearch.
+ Elasticsearch is also now commonly used for big data applications. 
+ You can provision a cluster of instances and it has built-in integrations for data ingestion. So you can get Kinesis Data Firehose, AWS IoT, and CloudWatch Logs.
+ Finally, you get security through Cognito and IAM, KMS encryption, SSL & VPC security.
+ It also comes with two other tools, called Kibana for visualization and Logstash for log ingestion. And together Elasticsearch, Kibana and Logstash makes the ELK stack.
+ So Elasticsearch allows you to search any field.
+ From a solutions architect perspective
	+ Operation is going to be similar to RDS
	+ Security is done through Cognito, IAM, VPC, KMS and SSL.
	+ Reliability - we get Multi-AZ and clustering 
	+ Performance, it's really hard to evaluate but it's based on the Elasticsearch project which is open source and really, really good performance
	  and AWS says that the Elasticsearch service can scale two petabytes of data.
	+ Cost is going to be pay per node provisioned. So based on how many instances in your cluster you provision you're going to pay for them, so its similar to RDS.
+ Elasticsearch is greatly used for search and indexing capability.

#### Redshift

+ Redshift is a database based on PostgreSQL. But it is not used for OLTP or Online Transaction Processing. It is not a replacement for RDS.
  Redshift is OLAP or Online Analytical Processing. So it is used for analytics and data warehousing.
+ Redshift is going to have 10 times better performance than other data warehouse technologies. It is meant to scale to petabytes of data. One petabyte is 1000 terabytes.
+ The way Redshift stores data and what makes it good for analytics and not good for transaction processing, is that the data is stored in columns. So, it's called a columnar 
  storage of data, whereas Postgres and RDS and MySQL will be using row. So even though Redshift is based on Postgres, it's a heavily modified version of Postgres. Here 
  data is stored in a column.
+ It is a great optimization when you do analytics. 
+ It's highly distributed. So when you run a query, it's going to run it in parallel across many instances and many cores. So it's called a Massively Parallel Query Execution, or MPP.
  And that makes the database highly available as well because it's distributed.
+ You're going to pay as you go based on the instance provisioned. So here, you have to choose the right cluster size, and use it as much as you can, because you don't pay per the query.
  You pay for whatever you provisioned, and Redshift can use SQL interface for performing the queries. So we can use SQL to perform your analytics.
+ Redshift is a great tool to use when you have a BI, so business intelligence tool, to use on top of it such as AWS Quicksight for doing visualizations, or Tableau if you're looking 
  for another more professional or different type of technology that's not aws native.
+ We can load data into Redshift. Redshift has integrations to pull data from S3 or DynamoDB, but you can use DMS, and that's Database Migration Service, or even you can pull in from other 
  databases. So you can even pull for example, from the Read Replica in RDS. It's a really great way, for example, when have an RDS database but you want to do analytics on it,
  to create a Read Replica to pull the data from the Read Replica into Redshift, and to do the analytics into Redshift.
+ In terms of Redshift scale, it can go from one node up to 128 nodes. And each node has many cores. And each node by itself has 160 gigabytes of space.
	+ There's two types of nodes in Redshift, there's a leader node, and the leader node is used for planning the queries and aggregating results across all the compute nodes.
	+ The compute nodes are going to actually be performing the queries and they will send the results back to the leader.
+ If you have one node, then that node is both a leader node and a compute node.
+ If we use Redshift spectrum, you don't need to load the data into Redshift first. You can perform the queries directly against S3, so it's a great way to do ad hoc queries.
  But it's not like Athena because in Athena it's server less, whereas in Redshift, you still need to provision your cluster. And then, if you use Redshift Spectrum,
  then you can perform the queries directly against S3.
+ There's backup and restore.
+ There's going to be security at the VPC, IAM, and KMS level.
+ You're going to get cloud watch monitoring out of the box.
+ Redshift Enhanced VPC Routing - So if you enable it, basically, all the copy of data from whatever stores you want into Redshift, or UNLOAD, which is from Redshift back to S3, goes through VPC.
  But it gives you enhanced, basically, security and maybe better performance, as well as your data doesn't go over the public internet.
+ Redshift in regards to the snapshots and disaster recovery - Snapshots are point-in-time backups of your cluster, and they will be stored internally in Amazon S3, and they will be incremental.
  So, only the stuff that has changed in your Redshift cluster will be saved. You can restore a snapshot into a new cluster, so this is all very similar to RDS, and you can have automated snapshots 
  as well every eight hours, or every five gigabytes, or on a schedule, and you would set the retention for this. You can also do manual snapshots, and in case you do that,
  the snapshot will be retained until you manually delete it. You can configure Redshift to automatically copy snapshots, either automated or manual, of a cluster to another AWS region.
  This provides you a good disaster recovery strategy. You just do these snapshots, configure them to be in another region, and if an entire region goes down, you can restore your Redshift cluster 
  very quickly elsewhere.
+ Redshift spectrum - We can query data that is already in Amazon S3 without loading it in advance into your Redshift cluster. So, you query the data while it's in place in S3,
  but to use it, you must have a Redshift cluster available to start the query. It is not like Athena, where you can just go ahead and query data in S3. First, you have to create the Redshift cluster.
  And then, you need to launch your query against S3 with Redshift Spectrum. And then the query will be submitted to thousands of Redshift Spectrum nodes that will perform that massively parallel 
  query for you. So you have your Amazon Redshift cluster in the middle, and you're going to do a query on S3, so this is going to be a Redshift Spectrum query, And automatically, 
  Redshift will decompose that query and send it to Redshift Spectrum nodes, which can be thousands of them. And these nodes will automatically, talk to Amazon S3, and run your query for you,
  and the results will be aggregated back into your Redshift cluster, which will give you your response and your right data. It is somewhat serverless, and it allows you to scale to massive queries
  without having a huge Redshift cluster in the beginning. https://aws.amazon.com/blogs/big-data/amazon-redshift-spectrum-extends-data-warehousing-out-to-exabytes-no-loading-required
+ From a solutions architect's perspective
	+ Operations is going to be similar to RDS
	+ Security as well is going to be similar to RDS (IAM, VPC, KMS, SSL)
	+ Reliability - we get highly available clusters. And there's a auto healing feature, so basically, once you provision a Redshift cluster, you can expect it to be pretty solid.
	+ Performance is going to be huge, so this is meant for analytics, and so, at least Amazon says, there's 10x performance increase compared to other data warehousing solution. And on top of it, 
	  if you want, in Redshift you can use compression, so you can compress the data into very, very small amounts, and that will also help you with performance, and your storage space, and your performance queries.
	+ Cost - you're going to pay per node provisions. You don't pay by the query like for Athena, here, you pay by the node provision, like for RDS. 
	  Even though Redshift is expensive, according to AWS, it is 1/10 the cost of other traditional enterprise data warehouses.
+ So Redshift is a really good data warehouse for you if you use the Cloud. Redshift is going to be used for analytics, business intelligence, and data warehousing,




























