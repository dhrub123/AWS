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

