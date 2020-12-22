#### EC2 FAQ
+ We have a limit of 20 on demand instances. After that a request needs to be raised with AWS. AWS is transitioning to a VCPU based limit from an instance count limit and 
  this is regionally different. VCPU is virtual central processing units and the limits define the number of vcpus that can be assigned for each family of instances
  for each account.This is only applocable to on demand. And the limits change over time based on usage. Still it can be increased from limits page on the AWS EC2 instance console.
+ We have the APIS
    + RunInstances
    + TerminateInstances
    + StopInstances - Release resources but preserve data in EBS boot partition
    + StartInstances
    + DescribeInstances
+ AWS is going to throttle email traffic on port 25 by default in ec2 instances due to increased spam. This can be removed on request.
+ SLA - 99.99% monthly uptime for EC2 and EBS in a region
+ EBS volume data will persist restarts if DeleteOnTermination is set to N. Instance store data does not persist. They need to be backed up in S3.
+ We can multiattach an EBS io1 volume to upto 16 EC2 nitro instances within the same availablity zone.
+ We can take a snapshot of EBS in real time but that can exclude any local caches. It is always recommended to detach the EBS volume and take a clean snapshot of it
  especially if it is a boot volume but that is not mandatory.
+ EFA (Elastic Fabric Adapter) which provided high performance networkign by bypassing kernel can only be enabled during launch or attached to a stopped instance.
+ Enhanced networking - SR IOV with HVM AMI , no additonal fee 
+ We cannot enable hibernation on an existing instance - running or stopped.If hibernation fails because, ebs volume is not large enough to store ram data, the instance will be 
  shut down. Hibernate stores ram state.
+ 
