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
  + GP2(SSD) - General purpose SSD volume that balances both price and performance for a variety of workloads.
  + IO1(SSD) - Highest performance SSD volume for mission critical low latency or high throughput workloads
  + ST1(HDD) - Low cost HDD volume designed for frequently accessed, throughput inensive workloads
  + SC1(HDD Cold) - Lowest cost HDD volume designed for less frequently accessed workloads. This is a cold volume.
+ Each volume is charecterized in Size | Throughput | IOPS (I/O Ops per sec)
+ **Only GP2 and IO1 can be used as boot volumes**
  
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
  


