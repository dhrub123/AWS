#### EBS(Elastic block storage)

An EC2 machine loses its root volume(main drive) when it is manually terminated. Unexpected terminations may happen. So we need a way to store data safely.
So we nned to store important data not on root volume but on an attached volume. This is an EBS(Elastic Block Store) volume. It is a network drive that we can attach to instances while they run. We can use this to let our instances pesist data.

+ It is a network drive, not a physical drive. It uses netowrk to communicate with the instance and hence there may be latency.
+ 1 EBS can be accessed from only 1 EC2.
+ It can be detached from an instance and attached to another one very quickly as log as they are in the same AZ.
+ It is locked to an Availability Zone. It is not available in other regions. To move a volume across AZs, we have to snapshot it.
+ When we create a EBS volume, we have to provide a provisioned capacity like size in GB and IOPS.
  + We get billed for the provisioned capacity, not what we use. So if we use 1 GB but we have provisioned 1 TB, we still get billed for 1 TB.
  + We can increase the capacity of the disk over time and start small.
+ Volume types
  + GP2(SSD) - General purpose SSD volume that balances both price and performance for a variety of workloads. Comparatively cheaper.
  	+ Have a burst aspect to it
  	+ Recommended for most workloads, system boot volumes, virtual desktops, low latency interactive apps and dev and test environments
	+ Size can range between 1 GB to 16 TB(16384 GB)
	+ Small GP2 volumes can burst IOPS upto 3000
	+ Max IOPS is 16000. GP2 has a rule(Baseline is 3 IOPS per GB with a minimum of 100 IOPS). This means at 5334 GB we are at max IOPS. The burst happens
	  below 1000 GB. So at 999 GB we get 2997 IOPS with burst upto 3000 IOPS but from 1000 GB 3000 IOPS and so on until we reach 5334 where we reach max
	  possible 16000 IOPS. We cannot set IOPS manually here. **+1TB = +3000IOPS**
	+ Locked to AZ
	+ **Throughput is not applicable**
  + IO1(SSD) - Highest performance SSD volume for mission critical low latency or high throughput workloads. This is for critical business applications which
    require sustained IOPS performance or more than 16000 IOPS per volume(gp2 limit). Expensive
    	+ Used for large database workloads like MongoDB, Cassandra, Microsoft SQL Server, MYSQL, POSTGRE, ORACLE
	+ Size ranges from 4 GB to 16 TB
	+ IOPS is provisioned(PIOPS) - Min 100 and Max 64000 (Nitro Instances)  else Max 32000 (Other instances)
	+ The maximum ratio of provisioned IOPS to requested volume size in GB is 50:1
	+ Here we have to define both IOPS and Size ( And the ratio of 50 to 1 must be maintained with a minimum of 100)
	+ **Throughput is not applicable**
  + ST1(Throughput optimized HDD) - Low cost HDD volume designed for frequently accessed, throughput inensive workloads
  	+ This is for streaming workloads that require consistent fast throughput at a low price
	+ Big data, Data warehouse, log processing, Apache Kafka
	+ This cannot be a boot volume
	+ Size ranges from 500 GB to 16 TB
	+ Max IOPS is 500(It cannot be set and is shown as NA)
	+ Max throughput of 500 MB/s - can burst. The baseline throughput is 40 MB/s per TB and can burst upto 500
  + SC1(HDD Cold) - Lowest cost HDD volume designed for less frequently accessed workloads. This is a cold volume.
  	+ Throughput oriented storage for large volumes of data that is infrequently accessed
	+ Scenarios where lowest storage cost is important
	+ Cannot be boot volume
	+ Size ranges from 500 GB to 16 TB
	+ Max IOPS is 250(It cannot be set and is shown as NA)
	+ Max throughput of 250 MB/s - can burst. The baseline throughput is 12 MB/s per TB and can burst upto 250
	
+ Each volume is charecterized in Size | Throughput | IOPS (I/O Ops per sec)
+ **Only GP2 and IO1 can be used as boot volumes**

#### Attach and EBS volume and mount it 

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html

We add 2 EBS volumes in an EC2 instance , one is a root volume , the other is a 2 GB extra volume. We ssh to the EC2 and execute below commands.

**lsblk** - This gives us a list of disks attached to the EC2 instance
```
[ec2-user@ip-172-31-3-26 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk 
└─xvda1 202:1    0   8G  0 part /
xvdb    202:16   0   2G  0 disk 
```
**sudo file -s /dev/xvdb** - If data is returned that means there is no file system on the device and we must create one. 
```
[ec2-user@ip-172-31-3-26 ~]$ sudo file -s /dev/xvdb
/dev/xvdb: data
```
If there is file system on device and it is formmatted, it will return 
```
[ec2-user@ip-172-31-3-26 data]$ sudo file -s /dev/xvdb
/dev/xvdb: Linux rev 1.0 ext4 filesystem data, UUID=789dd1ad-c517-4539-9b05-30c037db250d (needs journal recovery) (extents) (64bit) (large files) (huge files)
```
**sudo mkfs -t ext4 /dev/xvdb** - This is to create a volume out of the device
```
[ec2-user@ip-172-31-3-26 ~]$ sudo mkfs -t ext4 /dev/xvdb
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131072 inodes, 524288 blocks
26214 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```
**sudo mkdir /data** - We will create a data folder
**sudo mount /dev/xvdb /data** - We will mount our directory in the data folder

Now if we run lsblk, we can see that xvdb drive has been mounted to a data folder
```
[ec2-user@ip-172-31-3-26 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk 
└─xvda1 202:1    0   8G  0 part /
xvdb    202:16   0   2G  0 disk /data
```

Now we navigate to data and create hello.txt
```
[ec2-user@ip-172-31-3-26 ~]$ cd /data
[ec2-user@ip-172-31-3-26 data]$ sudo touch hello.txt
```
We write data into file
```
[ec2-user@ip-172-31-3-26 data]$ sudo nano hello.txt
[ec2-user@ip-172-31-3-26 data]$ cat hello.txt
hello world 
```

**To mount this file during every system reboot we need to add an entry to /etc/fstab.orig**
```
[ec2-user@ip-172-31-3-26 data]$ sudo cp /etc/fstab /etc/fstab.orig
[ec2-user@ip-172-31-3-26 data]$ sudo nano /etc/fstab
Add this line /dev/xvdb /data	ext4 defaults,nofail 0 2
[ec2-user@ip-172-31-3-26 data]$ sudo cat /etc/fstab
#
UUID=b24eb1ea-ab1c-47bd-8542-3fd6059814ae     /           xfs    defaults,noatime  1   1
/dev/xvdb /data ext4 defaults,nofail 0 2
```
To exit nano , ctrl+x

**Unmount /data and remount**
```
[ec2-user@ip-172-31-3-26 /]$ sudo umount /data
[ec2-user@ip-172-31-3-26 /]$ sudo mount -a
[ec2-user@ip-172-31-3-26 /]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk 
└─xvda1 202:1    0   8G  0 part /
xvdb    202:16   0   2G  0 disk /data
```
#### EBS Snapshots:
+ Incremental - only backup changed blocks
+ EBS backups use IO of EBS volume, so we should not backup when there is aheavy load in our application, we should try to backup during quieter or downtime.
+ Snapshots are internally stored in S3 but we cannot directly see the. We get billed for the storage.
+ Not necessary but recommended to detach volume to do snapshot.
+ Max 100,000 snapshots per account
+ Can copy snapshots across AZ or region
+ Can make AMI from snapshots
+ EBS volumes restored by snapshots need to be pre warmed using fio or dd command to read entire volume for optimal performance
+ Snapshots can be automated using Amazon Data Lifecycle manager
+ Right click on volume > Create Snapshot - Give description > Create - Snapshot backs up only the blocks that are used. Backup takes a little time.
+ We can create an Image or a volume from a snapshot, copy it to a different region or encrypt it
+ **Lifecycle manager** helps schedule and manage creation of EBS snapshots - Create Snapshot Lifecycle Policy > Target volumes with tags( the policy applies to
  any volumes with mentioned tags), Define schedule an start time, retention rule( number of snapshots of a target volume to retain), Copy tags or Add tags,
  IAM Role. This provides automated backup solution.
  
#### EBS Operation: Volume Migration
+ EBS volumes are locked to specific AZ. To migrate it to a different AZ or region, we have to snapshot the volume, copy it to a different region and create a 
  volume out of that snapshot in the AZ of choice
+ Snapshots > RIght Click on volume > Create Volume > Select AZ - and we get the snapshot in the new AZ

#### EBS Encryption
When we create an encrypted EBS volume, we get the following.
+ Data at rest encrypted inside volume
+ All the data in flight moving between instance and volume is encrypted
+ All snapshots are encrypted
+ All volumes created from snapshots are encrypted
+ Encryption and Decryption is handled transparently - we have to do nothing and has a minimal impact on latency
+ EBS Encryption leverages KMS keys(AES-256)
+ When we copy an unencrypted snapshot, we can enable encryption
+ Snapshots of encrypted volumes are encrypted

To encrypt an unencrypted EBS volume attached to an EC2 instance,
+ We have to create a snapshot of the volume
+ Encrypt the EBS snapshot using the copy command
+ Create a new volume from the snapshot which is encrypted now
+ Attach the encrypted volume to the original EC2 instance.
+ Unencrypted EBS volume --> results in unencrypted snapshot --> Creating a volume from it results in encrypted volume

#### EBS vs Instance Store
Some instances do not come with root EBS volume, they come with an Instance store or ephemeral storage
+ Instance store is a physically disk(**very high IOPS**) attached to the physical server where our EC2 is vs EBS which is a network drive
+ PROS : Better I/O performance, good for buffer/cache/scratch data or temporary content, Data survives reboots
+ CONS : On stop or termination isntance store is lost, we cannot resize instance store and backups must be operated by user, no right click> backup here
+ Disks up to 7.5 TB which can change over time and can be stripped to reach 30 TB but once we setup a disk in local instance store, we cannot resize.
+ It is block storage just like EBS so we can have file system on it.
+ It cannot be resized and **there is risk of data loss if hardware fails so we need to have data replication**
+ It shows up as ephemeraln in console

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ISTORE_IOPS.png" width="60%" height="60%"/>

#### EBS RAID CONFIGURATIONS

EBS has raid option. But it is already redundant storage(replicated within an AZ). So what if we want to increase IOPS to 100000 IOPS or mirror EBS volumes.
We will mount volumes in parallel in RAIS settings as long as OS supports it. Some RAID options are RAID0, RAID1, RAID5 and RAID6(5 and 6 are not recommended for EBS). **This is done in OS level , not in console **
+ **RAID 0 means increased performance**. We have an EC2 instance which has one logical volume backed by 2 or more EBS volumes. So when we do a write, it may go
  to EBS volume 1 or EBS volume 2. So they get distributed between the 2 volumes. So when we combine 2 volumes, we get total disk space and IO of the volumes.
  But if one of these disks fail, all the data is failed. Use cases are applications that need a lot of IOPS but does not need fault tolerance like a database
  which already has replication built in. Using this we can have a big disk with lots of IOPS. We can go 100000 IOPS by combining 10 volumes of 10000 IOPS each.
  For example if we have 2 500 GB Amazon EBS IO1 volumes with 4000 provisioned IOPS each, it will create a 1000 GB RAID0 array with an available bandwidth of 
  8000 IOPS and 1000 MB/s of throughput.
+ **RAID 1 means fault tolerance**. Here we write to both volumes at the same time. RAID 1 = mirroring a volume to one another. If one of the volume fails, the
  other one is still working. We have to send the data to 2 EBS volumes at the same time so 2x network throughput. Use case is applications that need increased
  volume fault tolerance and applications where you need to service disks. For example if we have 2 500 GB Amazon EBS IO1 volumes with 4000 provisioned IOPS
  each, it will create a 500 GB RAID1 array with an available bandwidth of 4000 IOPS and 500 MB/s of throughput.

|RAID0|RAID1|
|-----|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/RAID0.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/RAID1.png" width="50%" height="50%"/>|

#### EFS - Elastic File System
+ It is a managed NFS(Network file system) that can be mounted on many EC2 instances **across multiple availability zones**.
+ It is highly available, scalable, expensive(3xGP2), pay for what we use(We only pay for what we store)
+ We attach a security group to EFS to manage incoming connections(different ec2 instances across different AZs) and they will be mounting the same EFS on to
  their file system and access the same files. So security group is used to control access to EFS.
+ Use case : content management, web serving, data sharing, wordpress
+ Uses NFSv4.1 protocol
+ Encryption at rest using KMS
+ **Compatible with only linux based AMIs not windows**. POSIX file systems - Linux that has a standard file API
+ File system scales automatically, pay-per-use, no capacity planning
+ Performance and Storage classes
	+ EFS Scale - It is built for 1000s of concurrent NFS clients, 10 GB+ /s throughput. It can grow to Petabyte scale file system automatically.
	+ Performance mode set at creation time
		+ General Purpose - default, latency-sensitive use cases like web server, cms etc
		+ MAX I/O - higher latency, throughout, highly parallel(big data, media processing)
	+ Storage Tiers (lifecycle management feature - move file after N days)
		+ Standard - For frequently accessed files
		+ Infrequent access (EFS-IA) - cost to retrieve files, lower price to store
+ Go to EFS console > Give name > Customize > Automatic backups, Lifecycle Mangement which helps us to move to EFS-IA if unused for more than 30 days,
  Performance mode - General Purpose or Max I/O, Throughput mode - Bursting or Provisioned(Mention Throughput), Enable encryption at rest > Network > 
  We can mount in different AZ and for each AZ we need a security group. So we create a security group and define inbound rule - NFS and source as security
  group EC2_EFS_SG defined for EC2 isntance below.> File system policy > Create
+ We then create 2 instances in 2 AZs and create a security group(EC2_EFS_SG) for EFS and attach it. We have to install amazon-efs-utils package in our ec2.
  ```
  sudo yum install -y amazon-efs-utils
  ```
+ We go to EFS console and click attach. This gives us info on how to mount EFS drives to our EC2. So we ssh into EC2 and execute following commands.
  ```
  mkdir efs
  sudo mount -t efs -o tls fs-5dafeeac:/ efs(we get this command from EFS console - attach - mount via DNS) - this needs the sg configs
  sudo touch hello-world.txt( This will create the file and it will be visible in all other instances which have mounted this EFS)
  ```
<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/EFS.png" width="50%" height="50%"/>

#### EBS vs EFS - Elastic block store
|EBS|EFS|
|---|---|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/EBS_1.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/EFS_1.png" width="50%" height="50%"/>|

+ In EBS we are charged for provisioned capacity, not what we use but EFS is pay per use which we can leverage for cost savings
+ EFS is for NFS to be mounted across multiple instances, EBS is for a network volume mounted to a single EC2 instance locked in AZ and instance store means 
  maximum amount of IO.
