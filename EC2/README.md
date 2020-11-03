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

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/SG_2.PNG" width="80%" height="80%"/>

#### Referencing other security groups:
Security groups can refer other security groups, ip adresses, cidr blocks but **not DNS names**.
If we have a security group 1 which authorizes inbound traffic from security group 1 and security group 2, then that helps us in the following way. Suppose there are 4 instances. I1 has SG1 attached. I2 has SG2 attached. I3 has SG3 attached. I4 has SG1 attached. So inbound traffic is allowed to I4 from I1 and I2.
Inbound traffic is allowed to I1 from I4 and I2. 

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/SG_REFERENCED.PNG" width="80%" height="80%"/>

#### Public vs Private vs Elastic IP

Networking is of 2 sorts - IPv4 and IPv6. Both are supported by AWS.
+ IPV4 - 1.160.10.240. It is most common.It allows for 3.7 adresses in public space. The format is [0-255]:[0-255]:[0-255]:[0-255]
+ IPV6 - 3ffe:1900:4545:3:200:f8ff:fe21:67cf. It is mainly used for IOT.

+ The **Public IPs** can talk to one another over internet. Machine with public ip can be identified on the internet. It is unique, no two machines can have same public ip. They can be geolocated easily.
+ **Private IPs:** When a company has a private network, it has a private ip range. All computers in that private network can talk to each other using their private ip. But to talk to outside public ip, they will need a internet gateway which has a public ip. This is a common AWS pattern. Machines with private ip can be identified in private network only. They are only unique across private network. 2 different private networks can have same private ip. Machines with private ip connect to wwww using NAT + internet gateway(proxy). Only specified range of ips can be used as private ips.
+ **So public ips are accessible all over the internet and private ips are only accessible in private network.**
+ **Elastic IPs:** When we start and stop an EC2 instance, its public ip changes. If a fixed public ip is needed for our instance, we will need an elastic ip. It is a public ipv4 which we own as long as we do not delete it. We can atatch this elastic ip to one instance at a time. With an elastic ip, one can mask failures by quickly remapping it to another instance in their account. We can have **5 elastic ips in our account but this can be increased by asking AWS.**
+ It is recommended to avoid elastic ips. Instead 1) we can us a public ip and register dns name to it or 2) use a load balancer instead of a public ip.
+ By default, our EC2 instance comes with a private ip for internal AWS network and a public ip for www. SSH with public ip, not private ip. If instance is stopped and started, **public ip changes.**
+ If we ssh into instance using public ip , we will see a prompt ec2-user@<private-ip> which means that the ec2 machine is identified by the private ip in aws network. If we do ifconfig -a, eth0 or the virtual ethernet interface will also give us the private ip as inet.
+ Now, if we stop the instance, the public ip will disappear and if we start again ,the public ip will be different. But the **private ip is still the same**.
+ If we want to persist ips between restarts, go to network and security > Elastic ip > Allocate elastic ip from Amazon's pool of IPV4 adresses or bring in your own pool. > Allocate
+ Then associate elastic ip to ec2 instance. Now if we start and stop, public ip is retained as it is an elastic ip. **Elastic IPs get charged for when not associated with an EC2 instance**
  
<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/PUBLIC_VS_PRIVATE_IP.PNG" width="60%" height="60%"/>

#### Install apache on EC2.
```
# 1) ssh to instance
ssh ec2-user@35.180.100.144 -i ec2.pem
# 2) Elevate to super user
sudo su
# 3) Update without prompting
yum update -y
# 4) Install apache
yum install -y httpd.x86_64
# 5) Start apache service
systemctl start httpd.service
# 6) enable across reboots
systemctl enable httpd.service
# 8) curl -> we should see page content
curl localhost:80
# 9) create index.html. If we do a http://<public_ip>:80 , we will see a page displaying hello world. Please note that we need to add HTTP inbound port 80 rule in # sg for instance.
echo "Hello World" > /var/www/html/index.html
# 10) create index.html. If we do a http://<public_ip>:80 , we will see a page displaying hello world from <private ip>
echo "Hello World from $(hostname -f)" > /var/www/html/index.html
```

#### EC2 User Data :
+ This is used to boot strap instance with user data script. Bootstrapping means launching commands when instance starts.
+ The script is run only once at the first start of instance.
+ It is mainly used to automate boot tasks like installing updates, software, downloading files etc. 
+ The larger the user data script, the more it does, the larger the bootup time. It is run with root user sudo rights.
+ Configure Instance Details > Advanced Details > User Data - The script gets encoded to base 64.

```
#!/bin/bash
sudo su #admin privileges
#install httpd(Linux 2 version)
yum update -y
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
echo "Hello World from $(hostname -f)" > /var/www/html/index.html
```

#### Types of Instances based on Pricing
+ On Demand - allows to pay by the hour or second(Linux is by second and Windows is by hour) - For short workload, predictable pricing
+ Reserved - Reservation for 1 or 3 years, certain or entire amount upfront, but large discount compared to on demand price. These are ideal when the amount of time is known beforehand. for example say 1 year for a database. They are for long workloads.
  + Convertible Reserved Instances : long worload with flexible instance types. m4xlarge today, c5xlarge tomorrow.
  + Scheduled Reserved Instances : say every thursday between 3 am and 5 am (run a job for a year at a certain time).
+ Spot - enables to bid a price for instance capacity, if application has flexible timings, this can lead to significant savings. They are less reliable because we can lose these instances. These are very cheap and are for short workloads.
+ Dedicated host - These are physical ec2 server dedicated for use. They allow to bring exisitng server-bound software licenses over to aws and thus save costs. They allow us to control instance placement.
+ Dedicated instance - No other customer will share hardware on AWS.

##### On Demand Instances
+ Billing per second after the first minute
+ Pay for what we use
+ highest cost but no long term commitment
+ Perfect for users who want low cost and flexibility of AWS EC2 without any long term commitment or contract
+ Applications with unpredictable and short workloads (elastic workloads) that cannot be interrupted
+ Development and testing

##### Reserved Instances
+ Traditional IT
+ Upro 75% discount compared to on demand
+ upfront payment with long term commitment
+ Reserved for 1 or 3 years
+ We can reserve a specific instance type
+ Recommended for applications with steady state or predictable usage that require reserved capacity
+ Users willing to make upfront payment to reduce computing cost even further
  + Standard RIs(upto 75% off on demand) - if entire payment is made upfront and contract is for 3 years
  + Convertible RIs(upto 54% off on demand) - capability to change attribute of instance from say compute to memory intensive provided the exchange is of equal or greater value , expensive but flexible
  + Scheduled RIs - available to launch within a scheduled time window. It allows to obtain compute capacity within a certain recurring schedule. For example if a company has large sales during fridays, then it will go for RIs scheduled on every friday. Not available in all regions.

##### Spot Instances
+ 90% discount compared to on demand. But we can lose them at any point of time if current spot price goes over the max price
+ Applications with flexible start and end times which are feasible only at a very low cost. e.g. : Genomics and Pharma companies use this to perform research by running resource intensive apps on say a sunday at 4 am when price is very low.
+ They are very cost efficient for workloads which has resiliency to any kind of failure , for example, batch jobs ,image processing, data analysis and any jobs which can be retrieved.
+ They are not recommended for critical apps like databases.
+ Users who suddenly need additional compute capacity
+ Spot instances are terminated by AWS if spot price for that capacity increases. However Amazon does not charge for partial usage of hour in that case. However, if instance is terminated by customer, then whole hour is charged for.

##### Dedicated host
+ We get physically dedicated EC2 server for use which gives full control over instance placement.
+ It gives visibility to physical cores/ sockets of hardware and is great for licensing purposes - BYOL(Bring your own license).
+ Used for regulatory requirement or to save licensing costs which do not allow multi tenant virtualization
+ Can be purchased on demand
+ Can be purchased as a reservation of 3 years which saves about 70% compared to on demand

###### Dedicated Instances
+ Instances running on dedicated hardware or shared with instances in my account.
+ No control over instance placement. Can move hardware after stop / start.

** A great combo is reserved instances for baseline capacity(webapps) , on demand for unpredictable work and spots for peaks.** This results in more agility and cost savings.  

|Which host | When |
|-----------|------|
|On Demand|Pay Full Price|
|Reserved|Plan Ahead, use for a long time ,good discount|
|Spot|Bid for unused instances on spot. Highest bidder keeps instances. Can lose any time|
|Dedicated Host|Entire server|
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
  

