## EC2
Amazon EC2(Elastic Compute Cloud) is a webservice that provides resizable compute capacity in cloud. It reduces the time required to obtain and boot new server instances to minutes, allowing to quicly scale capacity both upwards and downwards based on changing computing requirements.

It has changed the economics of computing by allowing to pay for what is being used. It provides the tools needed to build a failure resistent application.

#### Types of Instances based on Pricing
+ On Demand - allows to pay by the hour or second(Linux is by second and Windows is by hour)
+ Reserved - Reservation for 1 or 3 years, certain or entire amount upfront, but large discount compared to on demand price.
+ Spot - enables to bid a price for instance capacity, if application has flexible timings, this can lead to significant savings.
+ Dedicated host - These are physical ec2 server dedicated for use. They allow to bring exisitng server-bound software licenses over to aws and thus save costs.

##### On Demand Instances
+ Perfect for users who want low cost and flexibility of AWS EC2 without any long term commitment or contract
+ Applications with Unpredictable workloads that cannot be interrupted
+ Development and testing

##### Reserved Instances
+ Applications with steady state or predictable usage that require reserved capacity
+ Users willing to make upfront payment to reduce computing cost even further
  + Standard RIs(upto 75% off on demand) - if entire payment is made upfront and contract is for 3 years
  + Convertible RIs(upto 54% off on demand) - capability to change attribute of instance from say compute to memory intensive provided the     exchange is of equal or greater value 
  + Scheduled RIs - available to launch within a scheduled time window. It allows to obtain compute capacity within a certain recurring     schedule. For example if a company has large sales during fridays, then it will go for RIs scheduled on every friday.

##### Spot Instances
+ Applications with flexible start and end times which are feasible only at a very low cost. e.g. : Genomics and Pharma companies use this to perform research by running resource intensive apps on say a sunday at 4 am when price is very low.
+ Users who suddenly need additional compute capacity
+ Spot instances are terminated by AWS if spot price for that capacity increases. However Amazon does not charge for partial usage of hour in that case. However, if instance is terminated by customer, then whole hour is charged for.

##### Dedicated host
+ Used for regulatory requirement or to save licensing costs which do not allow multi tenant virtualization
+ Can be purchased on demand
+ Can be purchased as a reservation which saves about 70% compared to on demand

#### Types of instances based on hardware

|Family|Speciality|Usecase|
|------|----------|-------|
|F1|Field gate programmable array|Genomic research, Video processing, Financial analytics, big data|
|I3|High Speed Storage|No SQL DB and Data Warehousing|
|G3|Graphics Intensive|Video Encoding|
|H1|High Disk throughput|Distributed file systems like HDFS and Map Reduce based workloads|
|T2|Lowest Cost General Purpose| Web Server, Small DBs|
|D2|Dense Storage|File Servers, Data Warehouse, Hadoop|
|R4|Memory Optimized|Memory Intensive Apps, DB|
|M5|General Purpose|Application Servers|
|C5|Compute Ompimized|CPU Intensive Apps/DBs|
|P3|General Purpose , Graphics Intensive|Bit coin, Machine Learning|
|X1|Memory Optimized|SAP HANA, Apache Spark|

Remember by **FIGHT DR MCPX**

#### EBS(Elastic block storage)

This is a virtual disk just like EC2 is a virtual machine.  It allows to create storgae volumes and then add to EC2 instance. Once attached we can create a file system , run a database etc. They are placed in a psecific availability zone and are automatically replicated to protect from failure.

##### SSDs
+ General Purpose SSD(GP2) - 
  + General purpose, balances both price and performance
  + 3000 IOPS per gig with upto 10000 IOPS and ability to burst upto 3000 IOPS for extended period of time for volumes at 3334 Gib and above. 
+ Provisioned IOPS SSD(IO1)
  + Designed for I/O intensive apps like large relational or NOSQL databases.
  + Used for more than 10000 IOPS
  + Can provision upto 20000 IOPS per volume.
  
##### Magnetic volumes
+ Throughput optimized HDD(ST1)
  + Big Data/ Data warehousing/ Log  processing
  + Can only be an additional volume and **not a boot volume**
+ Cold HDD(SC1)
  + Lowest cost for infrequently accessed workloads
  + Usage may be a file server
  + Can only be an additional volume and **not a boot volume**
+ Magnetic(standard)
  + Lowest cost per gigabyte for all EBS volumes that is **bootable**
  + Ideal for workloads where data is accessed infrequently and where emphasis is on lowest storgae cost.
  + Previous generation
  

