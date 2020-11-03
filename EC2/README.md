## EC2
Amazon EC2(Elastic Compute Cloud) is a webservice that provides resizable compute capacity in cloud. It reduces the time required to obtain and boot new server instances to minutes, allowing to quickly scale capacity both upwards and downwards based on changing computing requirements.

It has changed the economics of computing by allowing to pay for what is being used. It provides the tools needed to build a failure resistent application.
It gives us the capability to 
+ rent virtual machines(EC2) in cloud
+ store data on virtual drives (EFS)
+ distribute load across machines (ELB)
+ scale services using autoscaling group (ASG)

EC2 > launch instance > choose an AMI(Amazon Machine Image) - software or operating system that will be launched on the virtual machine. We can use Amazon Linux 2 AMI. It is free tier eligible. > Then we have to selct type of machine( t2.micro - free tier eligible). The type of machine is determined by how many vcpus , how much memory and many other parameters like these. > Then we configure instance details - use default vpc and subnet( subnet is the availability zone we want to launch instance in) , define shutdown behavior > Then we have to define storage - When we start the instance up , it has to store its os somewhere, that is the disk EBS volume(8gb, ssd). We delete on termination. We can also add tags to describe the instance. If we use a Name tag, the value show up in UI under instance name. > Then we configure security group (firewall around the instance) - we create a new security group and allow ssh , define source as 0.0.0.0/0(which means from anywhere) > Then launch. We can create a key pair to ssh to instance. We have to download the .pem file and then click launch instance.

Once the instance is in running state, billing will start. We can stop the instance to stop billing. Rebooting the instance will start billing. Terminate will **discard instance and delete data**. **Billing only happens when instance is in running state**.

#### SSH overview 

|OS|SSH|Putty|Instance Connect|
|--|---|-----|----------------|
|Mac|X||X|
|Linux|X|X|X|
|Windows < 10||X|X|
|Windows 10|X|X|X|

**EC2 instance connect** allows to connect to AWS instance directly from browser. Instance > Connect > EC2 Instance Connect. Aws will upload key to instance to access it temporarily. The port 22 inbound rule for SSH needs to be enabled in security group associated with instance for this. It only works for Amazon Linux 2 AMI.

#### SSH using LINUX/MAC
It allows us to control the remote EC2 machine from command line, through the port 22 allowed during security group configuration of instance.
|Command|Result|
|-------|------|
|ssh ec2-user@35.180.100.144|Permission denied because we did not provide a pem file|
|ssh ec2-user@35.180.100.144 -i ec2.pem|Permission 0644 for pem too open - bad Permission|
|chmod 400 ec2.pem and then ssh ec2-user@35.180.100.144 -i ec2.pem Then if we do whoami , we will get ec2-user|Logged in|
|exit|exit or ctrl+d|

#### SSH using windows < 10

Use Putty.
+ Use Puttygen to convert .pem to .ppk file.
+ In Putty in hostname field, input ec2-user@35.180.100.144 and under connection > SSH > auth > browse > load ppk file , then login.

#### SSH using Windows 10

+ Go to .pem properties > security > advanced > make yourself owner - disable inheritance and remove other owners > Give yourself full control
+ Use Powershell ssh ec2-user@35.180.100.144 -i ec2.pem

#### Common issues during SSH

+ connection timeout - issue is related to security groups and firewall
+ connection refused - ssh utility not running or application error - restart or create new instance

#### Security Groups

Security groups form the fundamental of network security in aws. They control how traffic is allowed into and out of EC2 machines. They can be used to allow inbound and outbound ports.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/SG_1.PNG" width="60%" height="60%"/>

Network Security > Security Groups : 

They have inbound and outbound tabs. In inbound, by default there is no rule, nothing is allowed. We have to add a rule. If no rule is added, timeout will happen if we try to ssh into instance. In outbound tab, all traffic out of the instance is enabled by default. Each rule has 5 things :
+ Type - SSH/HTTP
+ Protocol - TCP
+ Port Range - 22
+ Source - Custom 0.0.0.0/0
+ Description

The security groups act as a firewall and regulate
+ access to ports
+ authorised ip ranges
+ control of inbound network ( from other to instance) 
+ control of outbound network ( from instance to other)
+ **security groups can be attached to multiple instances and one instance can have multiple security groups**
+ security groups are locked down to vpc/region combination - They have to be created in a different region or a different vpc.
+ They live outside EC2. When they block traffic, EC2 is not even aware of the blocked traffic.
+ It is recommended to maintain a separate security group for SSH access.
+ **By default all inbound traffic is blocked and all outbound traffic is allowed**

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
  

