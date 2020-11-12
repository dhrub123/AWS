## AWS RDS

RDS stands for relational database service. It is a managed database service fir datbase engines and it uses SQL as query language.
It allows us to create databases in the cloud that are managed by AWS and supports various DB engines like Postgres, MySQL, MariaDB, Oracle, Microsoft SQL Server 
and Aurora(This is a AWS proprietary database).

Got to RDS Servvice > Database tab > Create database > Standard Create or Easy create , Engine types - Aurora(not free tier), MySQL, POSTGRE, Oracle, Microsoft SQL Server, MariaDB. We can select Mysql, Templates - Production, Dev/Test or Free Tier(This comes with values set up for us), Settings - DB name or DB Instance identifier, credentials - master username and password, DB instance size - instance class(standard, memory optimized or burstable), Storage - type amd allocation size, we can also enable storage autoscaling, Availabilty and Durability - Multi AZ, Connectivity - VPC and Publicly Accesible - Yes or No and Security Group, Backups - retention period, Maintenance - upgrade and window, Deletion protection - if we enable this , we will not be able to delete without disabling this first.

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

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/UC2.png" width="50%" height="50%"/>

Read Replicas help us to scale our reads. An application may need to scale out reads due to heavy load. In that case we can use read replicas.
+ Upto 5 read replicas
+ Read replicas within AZ, Cross AZ or Cross Region
+ Replication is asynchronous, so reads are eventually consistent. This means that if there are 2 read replicas, they will need some time to get data synced from 
  main DB. If a read request is routed to them before the sync, there is a possibility of getting old data.
+ Replicas can also be promoted to their database. They are then out of the replication mechanism.
+ In order to leverage Read Replicas, the main application must update its connection string to leverage list of read replicas in RDS cluster.

|Usecase1|Usecase2|
|--------|--------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/UC1.png"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/READ_REPLICA.png"/>|

#### Multi AZ(Disaster Recovery)

+ Synchronous replication - a change in master DB also needs to be synchronously replicated to standby DB to be accepted.
+ So we get one DNS name, in case of failure there will be automatic failute to standby DB instance. In case of loss of AZ, loss of network, loss of instance or 
  storage failure of master, we will have failover and standby AZ will become the new master.
+ Availability is increased
+ No manual intervention in apps even in case of failure
+ Not used for scaling, the standby db is just for standby, no one can read or write to it.
+ Read replicas can also be setup as Multi AZ for disaster recovery

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/MULTI_AZ.png" width="50%" height="50%"/>

#### RDS Encryption and security

+ At rest encryption
  + Possibility to encrypt the master and read replicas with AWS KMS - AES-256 Encryption
  + Encryption has to be defined at launch time
  + **If master is not encrypted, read replicas will also not be encrypted**
  + Transparent data encryption - TDE is available for ORACLE and SQL server.
+ In-Flight Encryption
  + SSL certificates to encrypt data to RDS in flight
  + Provide SSL options with trust certificate when connecting to database
  + To enforce SSL 
    + POSTGRE SQL - rds.force_ssl = 1 in AWS RDS console - parameter groups
    + MYSQL - Within the DB : GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;
+ Encryption operations
  + Encrypting RDS backups
    + Snapshots of un encrypted RDS databases are un encrypted
    + Snapshots of encrypted RDS databases are encrypted
    + We can copy snaphot of an un encrypted database to an encrypted one
  + Encrypt an Un Encrypted database
    + Create a snapshot of un encrypted database
    + Copy the snapshot and enable encryption for the snapshot
    + Restore the database from the encrypted snapshot
    + Migrate applications to the new database and delete the old database
    
#### RDS Security - Network and IAM
+ Network Security 
  + RDS databases are usually deployed within a private subnet not a public one
  + The security works by leveraging security groups which controls what ips or security groups can communicate to the database
+ Access Management
  + IAM policies help control who can manage AWS RDS(create/delete database and read replicas) through the RDS API
  + Traditional username and password is used to login to datbase
  + IAM based database authentication can be used to login to RDS mySQL and POSTGRE SQL
    + We do not need a password for this, instead we get an authentication token through IAM and RDS API calls.
    + The token has a life of 15 minutes
    + Benefits - a) Network in/out must be encrypted using SSL b) Users are centrally managed in IAM instead of within DB c) IAM roles and EC2 instance profiles
      can be leveraged for easy integration
    + So the EC2 which calls the RDS db will have an IAM role which it will use to call RDS service to get a 15 min valid token. Then it will call the DB using
      that token.
    + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/RDS_AURORA_ELASTICACHE/images/RDS_SEC.png" width="50%" height="50%"/>
      
###### RDS Security important points : 
+ Encryption at rest 
  + It is done when we first create the DB instance
  + unencrypted DB => snapshot => copy as encrypted => create DB from snapshot
+ Our Responsibility
  + Check IP / Ports / Security Group inbound rules in DBs SG
  + In-database user creation and permissions or manage through IAM for POSTGRE an MYSQL
  + Create database with or without public access(in public or private subnets)
  + Ensure parameter groups or DB is configured to allow SSL connections
+ AWS Responsibility
  + NO SSH access
  + No manual DB or OS patching
  + No way to audit underlying instance
  



