## DISASTER RECOVERY AND MIGRATIONS

#### DR

+ A disaster is any event that has a negative impact on a company's business continuity or finances.
+ A disaster recovery is about preparing and recovering from these disasters.
+ What kind of disaster recovery can we do on AWS or on general?
	+ We can do on-premise to on-premise. That means we have a first data center, maybe in California, another data center, maybe in Seattle,
	  and so this is traditional disaster recovery and it's actually very, very expensive.
	+ We can start using the cloud and do on-premise as a main data center and then if we have any disaster, use the cloud. So this is called a hybrid recovery.
	+ If you're just all in the cloud then you can do AWS Cloud Region A to Cloud Region B, and that would be a full cloud type of disaster recovery.
+ There are two key terms for DR.
	+ RPO, recovery point objective
		+ The first one is the RPO, recovery point objective, and that is how often you run backups, how back in time can you to recover.
		  And when a disaster strikes, basically, the time between the RPO and the disaster is going to be a data loss. For example, if you back up data every hour
		  and a disaster strikes then you can go back in time for an hour and so you'll have lost one hour of data. So the RPO, sometimes it can be an hour,
		  sometimes it can be maybe one minute. It really depends on our requirements, but RPO is how much of a data loss are you willing to accept in case 
		  a disaster happens?
	+ RTO, recovery time objective
		+ RTO on the other end is when you recover from your disaster. And so, between the disaster and the RTO is the amount of downtime your application has.
		  So sometimes it's okay to have 24 hours of downtime, sometimes maybe just one minute of downtime. So basically optimizing for the RPO and the RTO
		  does drive some solution architecture decisions, and obviously the smaller you want these things to be, usually the higher the cost.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/DR.png" width="60%" height="60%"/>
+ Disaster recovery strategies
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/DR1.png" width="60%" height="60%"/>
	+ Backup and restore 
		+ Backup and restore has a high RPO.
		+ If you have a corporate data center, for example, and here is your AWS Cloud and you have an S3 bucket, and so if you want to backup your data over time,
		  maybe we can use AWS Storage Gateway and have some lifecycle policy put data into Glacier for cost optimization purposes, or maybe once a week you're sending a ton of data
		  into Glacier using AWS' Snowball. So if you use Snowball, your RPO is gonna be about one week because if your data center burns or whatever and you lose all your data 
		  then you've lost one week of data because you send that Snowball device once a week. 
		+ If you're using the AWS' Cloud instead, maybe EBS volumes, Redshift and RDS, if you schedule regular snapshots and you back them up 
		  then your RPO is going to be maybe 24 hours or one hour based on how frequently you do create these snapshots. And then when you have a disaster strike you
		  and you need to basically restore all your data, then you can use AMIs to recreate EC2 instances and spin up your applications or you can restore straight from a snapshot
		  and recreate your Amazon RDS database or your EBS volume or your Redshift.
		+ And so that can take a lot of time as well to restore this data and so you get a high RTO as well. But the reason we do this is actually it's quite cheap to do backup and restore.
		  We don't manage infrastructure in the middle, we just recreate infrastructure when we need it, when we have a disaster and so the only cost we have is the cost of storing these backups.
		  So Backup and restore is not too expensive and you get high RPO, high RTO.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/BR.png" width="60%" height="60%"/>
	+ Pilot light
		+ With pilot light, a small version of the app is always running in the cloud and so usually that's going to be your critical core or pilot light.
		+ It's very similar to backup and restore but faster because your critical systems, are already up and running and so when you do recover, you just need to add on all the other systems that are not as critical.
		+ We have a data center, it has a server and a data base, and AWS Cloud. We are going to do continuous data replication from your critical database into RDS which is going to be running at any time
		  so you get an RDS database ready to go running. But your EC2 instances, they're not critical just yet. In case you have a disaster happening, Route 53 will allow you fail over from your server
		  on your data center, recreate that EC2 instance in the cloud and make it up and running, but your RDS database is already ready. 
		+ Here we get a lower RPO and a lower RTO and we still manage costs. We still have to have an RDS running, but just the RDS database is running, the rest is not and your EC2 instances only are brought up,
		  are created when you do a disaster recovery. So pilot light is a very popular choice. and it is only for critical core assistance.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/PL.png" width="60%" height="60%"/>
	+ Warm standby
		+ Warm standby is when you have a full system up and running but at a minimum size so it's ready to go, but upon disaster, we can scale it to production load.
		+ We have our corporate data center. It is a little more complicated. We have a reverse proxy, an app server, and a master database, and currently our Route 53 is pointing the DNS
		  to our corporate data center. And in the cloud, we'll still have our data replication to an RDS Slave database that is running. And maybe we'll have an EC2 auto scaling group,
		  but running at minimum capacity that's currently talking to our corporate data center database. And maybe we'll have an ELB as well, ready to go. And so if a disaster strikes you,
		  because we have a warm standby, we can use Route 53 to fail over to the ELB and we can use the failover to also change where our application is getting our data from.
		  Maybe it's getting our data from the RDS Slave now, and so we've effectively stood by and then maybe using auto scaling, our application will scale pretty quickly. 
		  So this is a more costly thing to do now because we already have an ELB and EC2 Auto Scaling running at any time, but again, you can decrease your RPO and your RTO doing that.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/WS.png" width="60%" height="60%"/>  
	+ Hot site or multi site approach
		+ It's very low RTO, we're talking minutes or seconds but it's also very expensive. But you get to full production scale running on AWS and On Premise. So that means we have your On Premise data center,
		  full production scale, you have your AWS data center, full production scale with some data replication happening. And so here what happens is that because you have a hot site that's already running, your 
		  Route 53 can route request to both your corporate data center and the AWS Cloud and it's called an active, active type of setup. And so the idea here is that the failover can happen.
		  Your EC2 can failover to your RDS Slave database if need be, but you get full production scale running on AWS and On Premise, and so this costs a lot of money, but at the same time, you're ready to fail over,
		  you're ready and you're running into a multi DC type of infrastructure.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/MS1.png" width="60%" height="60%"/>  
		+ If you wanted to go all cloud, it would be the same kind of architecture. It will be a multi region so maybe we could use Aurora here because we're really in the cloud.
		  So we have a master database in a region and then we have your Aurora Global database that's been replicated to another region as a Slave and so these both regions are working for me
		  and when I want to failover, I will be ready to go full production scale again in another region if I need to.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DR/images/MS2.png" width="60%" height="60%"/>  
+ Important Points
	+ Backups
		+ You can use EBS Snapshots, RDS automated snapshots and backups, etc.
		+ You can push all these snapshots regularly to S3, S3IA, Glacier. You can implement a Lifecycle Policy. You can use Cross Region Replication if you wanted to make sure these backups
		  will be in different regions.
		+ If you want to share your data from On-Premise to the cloud, Snowball or Storage Gateway would be great technologies.
	+ High availability
		+ using Route 53 to migrate DNS from a region to another region is really, really helpful and easy to implement.
		+ We can also use technology to have multi-AZ implemented, such as RDS Multi-AZ, ElastiCache Multi-AZ, EFS, S3, all these things are highly available by default.
		+ If you're talking about the high availability of your network, maybe you've implemented Direct Connect to connect from your corporate data center to AWS.
		  But what if the connection goes down for whatever reason? Maybe you can use Site to Site VPN as a recovery option for your network.
	+ Replication
		+ In terms of replication, you can use RDS Replication Cross Region, Aurora, and Global Databases.
		+ Maybe you can use a database replication software to do your on-premise database to RDS
		+ You can use Storage Gateway as well.
	+ Automation
		+ Cloudformation/Elastic Beanstalk can help recreate whole new environments in the cloud very quickly.
		+ If you use CloudWatch, we can recover or reboot our EC2 instances when the CloudWatch alarms fail.
		+ AWS Lambda can also be great to customize automation. So they're great to do rest API but they can also be used to automate your entire AWS infrastructure, and so overall,
		  if you can manage to automate your whole disaster recovery then you are really, really well-set for success.
	+ Chaos testing
		+ We create disasters e.g. a widely quoted scenario now in the AWS' world is that Netflix, runs everything on AWS, and they have created something called a simian-army, and they randomly terminate
		  EC2 instances in production ensuring that their infrastructure is capable to survive failures. 

#### Database Migration Service(DMS)

+ We want to migrate a database from our on-premise system to the AWS cloud. We should use DMS for database migration service.
+ It's a quick and secure database service that allows you to migrate your database from on-premise to AWS.
	+ It's resilient and it's self healing.
	+ And all alongside the migration the source database remains available
	+ It supports many types of engines
		+ Homogenous migrations - from Oracle to Oracle or Postgres to Postgres.
		+ Heterogeneous migrations - e.g. if you want to migrate from Microsoft SQL Server all the way to Aurora.
	+ It supports continuous data replication using CDC or change data capture.
	+ To use DMS, you need to create an EC2 instance and that EC2 instance will perform the replication tasks for you.
	+ So very simply, your source database may be on-premise, and then you're running an EC2 instance that has the DMS software and they will pull the data
	  from the source database and continuously and put it in the target database.
	+ So  what are the sources and what are the targets?
		+ SOURCES:
			+ On-premise and EC2 database instances, such as Oracle, Microsoft SQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, and DB2.
			+ Azure : Azure SQL databases
			+ RDS which is all the databases including Aurora.
			+ Amazon S3
		+ TARGETS:
			+ On-premise and EC2 instances, such as Oracle, Microsoft SQL Server, MySQL, MariaDB, Postgres and SAP.
			+ Amazon RDS, Redshift, DynamoDB, S3, ElasticSearch Service, Kinesis, and Document DB.
	+ You can transform a source, which is on-premise, to a target, which is usually AWS and these targets are all the common databases on AWS.
	+ If the source database and the target database do not have the same engine
		+ We need to use something called AWS SCT or schema conversion tool And it will convert the database schema from one engine to another.
			+ If you're using an OLTP, we can migrate from SQL Server or Oracle to MySQL, PostgreSQL, or Aurora.
			+ You can transform for analytic processes(OLAP) such as Teradata or Oracle all the way to Amazon Redshift.
		+ If the source database has a different engine than the target database, we have to use DMS and SCT but we do not need to use SCT if we're migrating to same database engine.
		  So if you're doing on-premise PostgreSQL to RDS PostgreSQL, it is the same database agent, PostgreSQL, and so this for you will not use SCT. But if you're doing something 
		  such as Oracle to Postgres, then you will need to use SCT. The database engine is PostgreSQL, but RDS is just the platform that we're using to run the database engine.
+ Handson
	+ Go to DMS and create replication instance, give a name and description. We have to give the EC2 instance class , Engine version, storage(for buffer and log files),
      vpc to deploy, we can choose multi az and publicly accessible or not. Encryption and click create. 
    + Then we can create a database migration task , give it an identifier and choose replication instance(source and target database details), migrate exisiting data and create task.
      If the endpoints is different, it will use SCT internally.

#### On-premise strategies with AWS

+ We have the ability to download the Amazon Linux 2 AMI as a virtual machine in ISO format and load this iso image into common softwares used to create the VMs.
	+ That includes VMWare, KVM, Virtual Box(Oracle VM) or Microsoft Hyper-V. This would allow you to run Amazon Linux 2 on your on premise infrastructure directly 
	  using that VM. We can make it work with some user data and so on.
+ We have a feature called VM import and export. 
	+ This allows you to migrate your existing VMs and applications into EC2 directly using this feature. 
	+ You can also create a disaster recovery repository strategy if you had a lot of on premise VMs but you wanted to back them up into the cloud
	+ We can export back the VMS from EC2 to your on premise environment.
+ Application discovery service
	+ It allows you to gather information about your on premise servers and plan a migration at a high level 
	+ It does give you some server utilization information and dependency mappings and that could be quite helpful when you want to do a massive migration from on premise to the cloud.
	+ Finally you can track all that migration using the AWS migration hub.
+ AWS data base migration service or DMS 
	+ It allows you to replicate from On-premise => AWS, AWS => AWS , AWS => On-premise.
	+ If you had a mysql or a Postgres database running on premise and you wanted to start moving your workload into AWS, you could use DMS to replicate the database.
	+ Works with various database technologies that includes Oracle, mysql, dynamodb etc.
+ AWS server migration service or SMS
	+ This is for incremental replication of on premise Live servers to AWS. You can replicate the volumes directly into AWS. And this is used for more ongoing type of incremental 
	  replication.

#### Datasync

+ AWS DataSync is going to be a service used to move large amount of data from on-premise to AWS , but here we're talking about more file and file systems.
+ It can synchronize to Amazon S3, EFS or FSx for Windows
+ It can move data from your on-premise NAS or file systems that are compatible with the NFS and SMB protocols.
+ We would have our on-premise NFS or SMB server, it would be talking to the AWS DataSync agent, which is an agent we have to install on-premise
  that will monitor your on-premise data source for changes and the DataSync agent will do its work to transmit it to the DataSync service which in turn will send the data to Amazon S3,
  EFS file system, or Amazon FSx for Windows file server.
+ The replication tasks from the DataSync agent all the way to the DataSync service can be scheduled hourly, daily or weekly
+ You need to install this DataSync agent on-premise to make things work.
+ It works for EFS to EFS replication  by using the DataSync agents on the EC2 machine that monitors your source Amazon EFS file system and it will send it to the DataSync service endpoint
  that will replicate it automatically into another EFS service.
+ DataSync agent is for replicating data and it goes two ways and finally, the source can be an NFS or an SMB server or NAS, all the way into S3, EFS and FSx for Windows,
  or you can reuse it for replicating EFS to EFS.
+ https://docs.aws.amazon.com/datasync/latest/userguide/how-datasync-works.html

#### Transfering Large Datasets into AWS

+ We want to transfer 200 Terabytes of data into the cloud, and what we have currently is  a 100 Megabits per second internet connection.
	+ Option number one would be to use the internet, the public internet, or establish a Site-to-Site VPN, which also uses the public internet.
		+ The advantage of this is that it's immediate to set up, we can leverage our connection right away.
		+ If we do a quick computation, what we get is ```[200(TB)*1000(GB)*1000(MB)*8(Mb)]/100 Mbps = 16,000,000 seconds = 185 days```, so almost half a year to transfer 200 terabytes of data
		  over a 100 megabits internet connection. 
	+ If I want to transfer, now, over direct connect, and say we have provisioned a one gigabits per second line,
		+ It's going to be a long, one-time setup, it will take about a month to get this connection established
		+ Once I have this connection, it will take about, if we do the exact same computation, ten times faster than my first connection, so it's going to end up being 18.5 days.
		+ So a lot quicker, but still quite long.
	+ If I use Snowball
		+ I need to order my snowballs, and then I need to get about two to three snowballs and I can order them in parallel so they will arrive at my facility at the same time.
		+ Then it takes about one week, all in all, from the snowball to being delivered, to being loaded, to being packed, sent back to AWS, and data being transferred,
		  it will take about one week for the end-to-end transfer.
		+ If there was a database that was being transferred through Snowball, it could be combined with DMS to transfer the rest of the data, aferwards.
	+ For ongoing replications we could use techniques such as Site-to-Site VPN, again, because we'll have less data to transfer on an ongoing basis or use direct connect, or use DMS,
	  or use a service like DataSync. 
	+ So all these things would allow us to transfer the data on an ongoing basis, continuously or not, through some more reasonable internet's lines,
	  such as Site-to-Site VPN or direct connect. Snowball is going to be used more for one-off, large transfers, and as we can see, Snowball, in this case, is really, really helpful into speeding up
	  our first data transfer into AWS.
