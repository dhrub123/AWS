## AURORA

AWS Aurora is a proprietary technology from AWS. It is not open sourced.
+ POSTGRE and MYSQL are both supported as AURORADB  which means we can conenct to a AURORA DB using POSTGRE and MYSQL drivers.
+ It is AWS Cloud optimized and claims 5x performance over MYSQL on RDS and 3x performance over POSTGRE on RDS.
+ Aurora Storage starts at 10 GB and then automatically grows in increments of 10 GB up to 64 GB
+ Aurora can have 15 read replicas while MYSQL has 5, and the replication process is faster (sub 10 ms replica lag)
+ Failover in AUrora is instanntaneous because it is cloud native by default.
+ Aurora costs more than RDS - about 20 percent more but it is more efficient.

#### Aurora high Availablity and scaling

Aurora stores 6 copies of our data any time we write anything across 3 AZs.
+ It only needs 4 copies out of 6, for writes which means if one AZ is down, you're fine
+ It only needs 3 copies out of 6 for reads which means that it's highly available for reads.
+ Some kind of self healing process happens where if some data is corrupted or bad, then it does self healing with peer-to-peer replication in the back end.
  We don't rely on just one volume, we rely on hundreds of volumes and it just happens automatically and is managed by AWS. We don't actually interface with
  the storage.
+ Aurora is like multi AZ for RDS. There is only one instance that takes writes. So, there's a master in Aurora and that is where we'll take writes.
+ If the master doesn't work, the failover will happens in less than 30 seconds, on average. It's really really quick failover.
+ On top of the master, we can have up to 15 Read Replicas all serving reads. This is how we scale your Read work load. Any of these can Read Replicas can become 
  the master, in case the master fails. By default, you only have one master.
+ Read Replicas support Cross Region Replication.
+ Things to remember : 1 master, multiple Read Replicas and their storage is going to be replicated, self healing, auto expanding.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/AURORA_HA.png" width="50%" height="50%"/>

#### Aurora DB Cluser

How Aurora works with clients ? How do we interface with all these instances ?

+ We have a shared storage volume and it's auto expanding from 10GB to 64TB.
+ Master is the only thing that will write to your storage. Because the master can change and failover, Aurora provides us with a Writer Endpoint.
  It's a DNS name, a Writer Endpoint, and it's always pointing to the master. Even if the master fails over, your client still talks to the Writer Endpoint 
  and it is automatically redirected to the right instance.
+ We also have a lot of Read Replicas and we can have auto scaling on top of these Read Replicas. We can have 1 up to 15 Read Replicas and we can set up auto 
  scaling such that we always have the right number of Read Replicas. Now, because we have auto scaling, it can be really really hard for our applications 
  to keep track of the Read Replicas and their URL details. So **There is something called the Reader Endpoint.**
+ A Reader Endpoint has the exact same feature as a Writer Endpoint. It helps with connection load balancing and connects automatically to all the Read Replicas.
  Anytime the client connects to the Reader Endpoint, it will get connected to one of the Read Replicas and load balancing is automatically done. The 
  load balancing happens at the connection level, not the statement level.
+ Important points :  Writer Endpoint, Reader Endpoint, auto scaling and shared storage volume that auto expands.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/AURORA_CLUSTER.png" width="50%" height="50%"/>

#### Aurora Features

All of these things are handled for us by AWS

+ Automatic failover
+ Backup and recovery 
+ Isolation and security
+ Industry compliance 
+ Push-button scaling 
+ Automated patching with zero downtime
+ Advanced monitoring
+ Routine maintenance
+ Backtrack, which is giving you the ability to restore data at any point of time like go back to yesterday at 4 PM. It actually doesn't rely on backups.

#### Aurora Security

For security, it is similar to RDS because it uses the same engine.

+ We have Postgres and MySQL. We get encryption as rest using KMS.
+ We have automated backups, snapshots, and replicas that are also encrypted.
+ Encryption in flight using SSL and this is the exact same process we have for MySQL and Postgres, if we wanted to enforce it.
+ We have, also, authentication using IAM tokens, which is the exact same method we have seen for RDS, thanks to the integration with MySQL and Postgres RDS.
+ We are responsible still for protecting the instance with security groups and you cannot SSH into our instance.

#### Aurora Serverless

Aurora Serverless is an automated database instantiation with auto scaling feature capability based on actual usage of Aurora. It is suitable for
infrequent, intermittent, or unpredictable workloads. 
+ We do not have to do any capacity planning. It can be a lot more efficient because we pay per second and we have a lot of cost savings associated with it.
+ We have a shared storage volume and when our clients want to access our Aurora database, this happens in the back end.
  + There will be an Amazon Aurora created by Aurora Serverless and there is a proxy fleet, managed by Aurora that our client will connect to
    and then transfer it to our Amazon Aurora database. If we get more load, more Amazon Aurora databases will be created for us automatically. If we get 
    less load, then less databases will be created all the way up to 0 Aurora databases, if we don't have any usage.
+ Aurora Serverless really gives us the power of a relational database while giving us some serverless attitude because we have no scaling to do. We don't do 
  any capacity planning and it will scale based on demand.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/AURORA_SLESS.png" width="50%" height="50%"/>

#### Global Aurora

There are 2 ways to have Global Aurora across multiple regions.
+ Aurora Cross Region Read Replicas
  + They're useful for disaster recovery and they're very simple to put in place. We just create a Read Replica in another region.
+ Aurora Global Database - recommended
  + We have one primary region, where all the reads and the writes happen, and then we have up to 5 secondary read-only regions, where the replication lag
    is going to be less than 1 second. We can have up to 16 Read Replicas per secondary region. That's a lot of replication we can do all around the world.
    It will definitely help decrease latency. If we ever wanted to promote another region for disaster recovery, because the main region had a massive outage,
    then we have a RTO, which is recovery time objective of less than 1 minute. That means, in less than 1 minute, our new Aurora database in our secondary 
    region will become primary and will be ready to take on writes. For example, in us-east-1, which is our primary region, we have our database, the main one,
    and our applications do read and write to it. Then, we want to define a secondary region in eu-west-1 and so we'll define an Aurora Global database, 
    which will perform some replication with up to 1 seconds lag, and this is asynchronous replication. Our applications in eu-west-1 can directly read from 
    this database and perform read only workloads.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/AURORA_GLOBAL.png" width="50%" height="50%"/>

#### Handson

+ RDS > Create Database - Standard Create > Aurora(MySQL or POSTGRE SQL) >  We can choose version and database region(Regional or Global) - Database Features 
  (One writer multiple reader or One writer multiple reader parallel query or Multiple writers or Serverless) - Templates(Production or Dev Test) - Settings 
  -> DB Cluster Identifier, master username and password - DB Instance size - Availability and Durability(Multi-AZ : Read Replica in a different AZ) - Connectivity -> VPC , 
  publicly accessible ? , security group - Additional settings -> Backups, Encryption, Backtrack, Monitoring - Maintenance - Deletion Protection. So once created, we have 
  a writer and a reader endpoint. We can also add readers and add auto scaling.
