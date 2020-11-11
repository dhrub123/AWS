## AWS RDS

RDS stands for relational database service. It is a managed database service fir datbase engines and it uses SQL as query language.
It allows us to create databases in the cloud that are managed by AWS and supports various DB engines like Postgres, MySQL, MariaDB, Oracle, Microsoft SQL Server 
and Aurora(This is a AWS proprietary database).

#### Advantages of RDS vs deploying Database on EC2

RDS is a managed service and so many addon services are provided by AWS on top of the database.
+ Automated provisioning and OS patching
+ Continuous backups and restoration to a specific timestamp(Point in time restore)
+ Monitoring Dashboards
+ Read replicas for improved read performance
+ Multi AZ setup for Disaster Recovery
+ Maintenance windows for upgrades
+ Scaling capabiity(vertical by changing instance type and horizontal by adding read replicas)
+ Storage backed by EBS(IO1 or GP2)
+ **We cannot SSH into our RDS instances** because this is a service provided by AWS and it does not allow access to underlying resources

#### RDS backups - They are automatically enabled by AWS
+ Automated backups
  + Daily full bakup of database(duing maintenance window)
  + Transaction logs are backed up by RDS every 5 minutes
  + Both of these together gives us the ability to restore to any point in time(from oldest to 5 minutes ago)
  + 7 day retention by default but can be increased to 35 days
+ DB Snapshots
  + This has to be manually triggered by user
  + Retention of snapshot backup as long as needed like may be 6 months and so on

#### Read replicas 

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/READ_REPLICA.png"/>

Read Replicas help us to scale our reads. An application may need to scale out reads due to heavy load. In that case we can use read replicas.
+ Upto 5 read replicas
+ Read replicas within AZ, Cross AZ or Cross Region
+ Replication is asynchronous, so reads are eventually consistent. This means that if there are 2 read replicas, they will need some time to get data synced from 
  main DB. If a read request is routed to them before the sync, there is a possibility of getting old data.
+ Replicas can also be promoted to their database. They are then out of the replication mechanism.
+ In order to leverage Read Replicas, the main application must update its connection string to leverage list of read replicas in RDS cluster.

|Usecase1|Usecase2|
|--------|--------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/UC1.png"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/UC2.png"/>|

#### Multi AZ(Disaster Recovery)

+ Synchronous replication - a change in master DB also needs to be synchronously replicated to standby DB to be accepted.
+ So we get one DNS name, in case of failure there will be automatic failute to standby DB instance. In case of loss of AZ, loss of network, loss of instance or 
  storage failure of master, we will have failover and standby AZ will become the new master.
+ Availability is increased
+ No manual intervention in apps even in case of failure
+ Not used for scaling, the standby db is just for standby, no one can read or write to it.
+ Read replicas can also be setup as Multi AZ for disaster recovery

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/MULTI_AZ.png"/>





