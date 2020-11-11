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

