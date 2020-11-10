#### EBS(Elastic block storage)

An EC2 machine loses its root volume(main drive) when it is manually terminated. Unexpected terminations may happen. So we need a way to store data safely.
So we nned to store important data not on root volume but on an attached volume. This is an EBS(Elastic Block Store) volume. It is a network drive that we can attach to instances while they run. We can use this to let our instances pesist data.

+ It is a network drive, not a physical drive. It uses netowrk to communicate with the instance and hence there may be latency.
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
  


