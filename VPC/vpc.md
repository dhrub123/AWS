## VPC Intro

VPC is networking in AWS.
<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/vpc.png"/>

#### CIDR - IPv4(Classless Inter-Domain Routing)
+ CIDR are used for security group rules or AWS networking. e.g. 0.0.0.0/0 or 122.149.196.85/32
+ They help to define an IP adress range.
	+ WW.XX.YY.ZZ/32 == one ip
	+ 0.0.0.0/0 == all ips 
	+ There can also be 192.168.0.0/26 : 192.168.0.0 - 192.168.0.63 (64 ips)
+ A CIDR has 2 components
	+ The base IP (XX.XX.XX.XX) - it represents an ip contained in the range
	+ The subnet mask(/26) - it defines how many bits can change in the iP.
+ The subnet mask can take 2 forms
	+ 255.255.255.0 which is less common
	+ /24 which is more common
+ Subnet mask allows parts of the underlying ip to get additional next values from the base ip. 
	+ /32 == 1 IP == 2^0 (no ip can change)
	+ /31 == 2 IPS == 2^1 
	+ /30 == 4 IPS == 2^2
	+ /29 == 8 IPS == 2^3
	+ /28 == 16 IPS == 2^4
	+ /27 == 32 IPS == 2^5
	+ /26 == 64 IPS == 2^6
	+ /25 == 128 IPS == 2^7
	+ /24 == 256 IPS == 2^8(last ip number can change)
	+ /16 == 65536 IPS == 2^16(last 2 ip numbers can change)
	+ /8 == last 3 ip numbers can change
	+ /0 allows for all IPs = 2^32(all ip numbers can change)
	+ /n == 2^(32-n) IPS
+ 192.168.0.0/24 = 192.168.0.0 - 192.168.0.255 (256 IPS)
+ 192.168.0.0/16 = 192.168.0.0 - 192.168.255.255 (65536 IPS)
+ 134.56.78.123/32 = 1 IP
+ 0.0.0.0/0 = All ips
+ https://www.ipaddressguide.com/cidr

#### Difference between public and private ips
+ Allowed Ranges - IANA(Internet Assigned Numbers Authority) established certain blocks of IPv4 addresses for use of Private (LAN) purposes and rest are public (Internet) adresses.
+ Private IP ranges
	+ 10.0.0.0 to 10.255.255.255 (10.0.0.0/8) for in big networks
	+ 172.16.0.0 to 172.31.255.255(172.16.0.0/12) - This is the default one for default VPC.
	+ 192.168.0.0 to 192.168.255.255(192.168.0.0/16) - This is for home networks
+ All the rest of the Ips are public ips

#### Default VPC

+ All new accounts in AWS have a default VPC.
+ New instances are launched into default VPC if no subnet is specified.
+ Default VPC ahs internet connectivity and all instances have a public IP
+ we also have a public and private DNS name which was configured in the default VPC.
+ In the default VPC, we have been given the following things
	+ VPC - 1
		+ There is 1 CIDR block is 172.31.0.0/16
	+ Subnets - 2
		+ There are 2 subnets, one with CIDR block of 172.31.16.0/20 and another with CIDR block of 172.31.64.0/20
		  Each of these CIDR blocks should have 4096 IP adresses but we only see 4091 IPs available.
		  This is because 5 ips are reserved. The CIDR blocks of both these subnets are non overlapping and 
		  they are a subset of the VPC CIDR block. All traffics and all outbound traffics are allowed by default.
	+ Route Table - 1
		+ The route table belongs to the default VPC and is associated with the 2 subnets mentioned earlier.
		+ The routes define how the subnets can access the internet. We can see that one of the target is an internet gateway.
	+ Internet Gateway - 1
		+ The internet gateway is attached to the VPC and gives access to internet.
	+ DHCP option sets - 1
	+ Network ACL - 1
	+ Security Group - 1

#### VPC creation

+ VPC means Virtual Private Cloud
+ You can have a maximum of 5 vpcs per region which is a soft limit. You can place a request with AWS for more.
+ Each VPC can have 5 CIDR blocks. For each CIDR
	+ Min size is /28 or 16 ips
	+ Max size is /16 or 65536 ips
+ Because VPCs are private only private ip ranges are allowed.
	+ 10.0.0.0  - 10.255.255.255 (10.0.0.0/8)
	+ 172.16.0.0 - 172.31.255.255(172.16.0.0/12) 
	+ 192.168.0.0 - 192.168.255.255(192.168.0.0/16)
+ **Your VPC CIDR should not overlap with your corporate networks**. Else there will be issues connecting to your corporate network from cloud because
  of overlapping ip adress.
+ Handson
	+ We go to VPC and create VPC. We give CIDR range between /16 and /28. e.g. 10.0.0.0/16 We have to mention tenancy as default(shared) or dedicated. 
	  Then we click create. A main route table and a main network ACL comes with the VPC.
	+ We can also edit the CIDRs to add new CIDR blocks like 10.1.0.0/16.

#### Subnets
+ Now we need to add subnets to our VPC. Subnets are tied to specific availability zones. We will have 2 AZs for high availability.
  And in each AZ, we are going to have 2 subnets(one public and one private) - so 4 subnets in total.
+ We are going to create different subnets of different sizes because public subnets are usually smaller than private subnets. Public subnets usually have only
  the load balancers while private subnets have all the applications.
+ So we will create a subnet , give it a name of publicsubnetA , choose the vpc we created earlier which had a CIDR range of 10.0.0.0/16, choose the avalability zone of 
  us-east-1b and then give the subnet a CIDR range of 10.0.0.0/24 which is 256 ips(10.0.0.0 to 10.0.0.255)
+ We will create another subnet , give it a name of publicsubnetB , choose the vpc we created earlier which had a CIDR range of 10.0.0.0/16, choose the avalability zone of 
  us-east-1f and then give the subnet a CIDR range of 10.0.1.0/24 which is 256 ips(10.0.1.0 to 10.0.1.255)
+ We will create another subnet , give it a name of privatesubnetA and this is going to be much larger, choose the vpc we created earlier which had a CIDR range of 10.0.0.0/16,
  choose the avalability zone of us-east-1b and then give the subnet a CIDR range of 10.0.16.0/20 which is 4096 ips(10.0.16.0 to 10.0.31.255)
+ We will create another subnet , give it a name of privatesubnetB and this is going to be much larger, choose the vpc we created earlier which had a CIDR range of 10.0.0.0/16,
  choose the avalability zone of us-east-1f and then give the subnet a CIDR range of 10.0.32.0/20 which is 4096 ips(10.0.32.0 to 10.1.63.255)
+ When we create a subnet, we see that there are 5 less ips in the available ips e.g. 4091 instead of 4096 and 251 instead of 256.
  This is because AWS reserves the 1st 4 and last ip adress of each subnet. These 5 ip adresses are not available for use and cannot be assigned to an instance.
  e.g. in a CIDR block of 10.0.0.0/24 the reserved ips are
  	+ 10.0.0.0 - reserved for network adress
  	+ 10.0.0.1 - reserved for vpc router
  	+ 10.0.0.2 - reserved for mapping to Amazon provided DNS
  	+ 10.0.0.3 - reserved for future use
  	+ 10.0.0.255 - Network broadcast adress. AWS does not support broadcast in VPC. So the adress is reserved
+ We need 29 adresses, so we cannot choose a subnet size of /27(32 ip adresses). Because we have only 27 ip adresses available. We have to choose /26(64 ip adresses)
+ In the public subnets, we created we can modify auto assign public ipv4 adress and enable it.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/SUBNET.png" width="60%" height="60%"/>

#### Internet Gateways
+ Now that we have created subnets, we will launch an ec2 instance using one of the public subnets we created. we will give a sceuirty group which allows port 22 inbound.
+ Now if we try to SSH into the ec2 instance using the public ip of the instance, we will not be able to connect. This is not because of a security group issue but because of lack of internet connection.
+ So we will need an internet gateway.
	+ Internet gateways help our VPC instances connect with the internet.
	+ It scales horizontally and is HA and redundant.
	+ Must be created separately from VPC.
	+ One VPC can be attached to one internet gateway and viceversa.
	+ Internet gateway is also a NAT(network adress translation or a way to map multiple private up adresses to one public ip address) for the instances that have a public IPV4.
	+ Internet gateways do not allow internet access on their own. Route tables need to be edited for them.
	+ So we create an internet gateway from the internet gateway tab under vpc and it is detached now. We will attach it to our VPC.
	+ Then we have to edit the route table for the public subnet so that it points to the internet gateway for a specific IP range and then from there, our EC2 will be routed to our Internet Gateway and
	  will get internet access.
	+ So we will create 2 route tables under VPC, one private and one public. We will then associate the public route table with both our public subnets and the private route route table
	  with both our private subnets.
	+ Then we have to create routes in the route table. There will be existing routes like 10.0.0.0/16 - local - active which means that anytime we hit an ip within the CIDR 10.0.0.0/16,
	  the target is going to be a local network. This is fine for a private route table.
	+ But for public route table we are going to edit the route table to add another route which is 0.0.0.0/0 - select Internet gateway - active. This means that anytime, you hit an ip within
	  10.0.0.0/16 look in local network but for any other ip, talk to internet gateway. Save the routes and our ec2 is connected to internet.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/IG.png" width="60%" height="60%"/>

#### Nat Instances
+ Now we have an instance in our public subnet which can access the internet through internet gateway but they are also accessible. In our private subnet, our instances also need to be connected to
  internet to run updates etc but they cannot be accesible publicly(we should not be able to SSH into them). So we cannot use internet gateway.
+ So we need a NAT or network address translation. NAT comes in 2 flavors 
	+ NAT instances. This is outdated but still appears in exam.
		+ Allows instances in private subnets to connect to internet.
		+ Must be launched in a public subnet so that they have internet conenctivity
		+ Must disable EC2 flag - source / destination check
		+ Must have elastic IP attached to it because our route table will go to a fixed ip
		+ Route table must be configured to route traffic from private subnets to NAT instance.
		+ Modify networking > source/destination check and disable source - destination check.
		+ We are going to launch a amzn-ami-vpc-nat instance in public subnet A. Create a new security group allowing SSH, HTTP(from our VPC i.e. 10.0.0.0/16) and HTTPS(from our VPC i.e. 10.0.0.0/16) and launch.
		  If you want to allow ping, add ICMP rule from VPC.
		+ We will launch private instance in private subnet A and in SG, allow port 22 from VPC(10.0.0.0/16) and launch instance.
		+ We will then ssh to private instance from public instance. 
		+ Then if we ping google.com, it will not work.
		+ So we will edit private route table and add a route 0.0.0.0/0 - NAT instance - active which means anytime, you hit an ip within 10.0.0.0/16 look in local network but for any other ip, talk to nat instance.
		+ Exam points
			+ Preconfigured Amazon linux NAT ami is avaailable
			+ Not HA or resilient out of box, may need to create ASG in multi AZ + resilient user-data script
			+ Internet bandwidth depends on type of instance
			+ Must manage security groups and rules.
				+ Inbound - HTTPS, HTTP from VPC
				+ Outbound - HTTP, HTTPS
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/NATI.png" width="60%" height="60%"/>
	+ NAT gateways
		+ A better alternative to NAT instances is NAT gateway because AWS will manage NAT gateway, we will get higher bandwidth, better availability and no administration is needed.
		+ We will be paying by the hour for usage and bandwidth
		+ NAT is created in a specific AZ, uses an EIP but we need not manage that.
		+ Cannot be used by an instance in the same subnet as the NAT gateway, only can be used by instances in other subnets
		+ Requires an Internet Gateway(Private Subnet will talk to NAT gateway which will talk to IGW)
		+ We get 5 Gbps of bandwidth with autoscaling upto 45 Gbps
		+ No Security group needs to be managed or are required
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/NATG.png" width="60%" height="60%"/>
		+ A NAT gateway is resilient within a single AZ. For fault tolerance, we must create multiple NAT gateways in multiple AZ. So we must have one NAT gateway in each AZ. 
		  Then no cross AZ failover is needed, because if an AZ goes down, then the other AZ has its own NAT gateway.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/NATG2.png" width="60%" height="60%"/>
		+ When we delete the NAT instance, the route in Private subnet poiniting to the Nat instance will have a status of blackhole.
		+ We will create a NAT gateway in public subnet A and allocate or create a new elastic ip. Then we will edit pruvate route table to point to nat gateway. Then we will get internet access in our private instances.
		+ The NAT gateway must be in public subnet
	+ Differences between NAT gateway and NAT instances
		+ https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html

#### DNS Resolution in VPC and Route53 Private Zones
+ DNS Resolution in VPC
	+ enableDnsSupport = DNS resolution setting
		+ Default is true
		+ Helps decide if DNS resolution is supported for the VPC
		+ If true, there is a AWS DNS Server at 169.254.169.253 which will be queried automatically as primary DNS.
	+ enableDnsHostname = DNS hostname setting
		+ False by default for newly created VPC, true by default for default VPC
		+ Not effective if enableDnsSupport is not true.
		+ if true, it will assign public hostname to ec2 instances, it the instance has public ip.
	+ if you use custom DNS names in a privat zone in route 53, you must set both these attributes to true for your instances to resolve that private zone.
	+ If we rightclick on VPC and edit DNS resolution, it is enabled by default and will resolve AWS DNS for us. If we rightclick and edit DNS hostname, it is disabled by default.
	  The public ec2 instances in this VPC has a public ip but does not have a public DNS hostname. If we enable this, the instance will now have a public DNS hostname.
	+ We will go to route 53 now and create a private hosted zone(foobar.internal). We will select our VPC and click create. This will cost 50 cents. So private hosted zone is an internal hosted zone which we do not own
	  and we can create a CNAME record for our private hosted zone like this(demo.foobar.internal -> google.com). If we do a dig to demo.foobar.internal from a private ec2 now, demo.foobar.internal will send us to google.com.

#### NACL and security groups
+ We have an EC2 instance and a security group around it. Our EC2 instance resides within a subnet. 
+ Outside the subnet level, we have a NACL or network access control list. Before traffic even enters the subnet, the NACL resides.
+ Lets evaluate an incoming request. Let us assume that our EC2 is a web server exposing Apache application on port 22. 
    + The incoming request will come from the right hand side and will be evaluated by the NACL inbound rules. 
    + If the request passes the evaluation of the NACL inbound rules, it will then be evaluated by the inbound rules of EC2 security group.
    + If that evaluation passes as well, the request will go to the web server hosted in EC2 and will be served with a webpage. 
    + Then the webpage will need to be send back to the requester. 
    + The outbound will be allowed no matter what in security group level because outbound security group rules are stateful which means that if inbound request is allowed, 
      the outbound request will be allowed even if there is a specific deny rule in ec2 sg level.
    + Then the request reaches NACL. Now NACL outbound rules are stateless which means they will be evaluated.
    + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/NACLIN.png" width="60%" height="60%"/>
+ Let us evaluate an outbound request now. Our EC2 instance is making an outbound request this time.
    + The Security Group outbound rules will be evaluated to make sure that the request can leave the EC2 instance. 
    + Then the request will be evaluated by the NACL outbound rules and if it passes, it will reach the website and get a response.
    + Then the inbound NACL rule will get evaluated because it is stateless.
    + It goes back into our EC2 instance but the security group inbound rules is stateful this time and so the response will be allowed in no matter what.
    + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/NACLOUT.png" width="60%" height="60%"/>
+ NACL is like a firewall which controls traffic to and from subnet.
+ Default NACL allows everything in and out.
+ One NACL per subnet, new subnets are assigned the default NACL.
+ NACL rules
    + Rules have a number(1 - 32766) and a lower number has higher precedence. e.g. If we have a rule 100-AllowIp and another rule 200-DenyIp, then the ip will be allowed.
    + Last rule in a NACL is an astersik(*) which denies the request in case of no rule match
    + AWS recommends adding rules in increments of 100 so that we can add rules in between later.
+ Newly created NACLs deny everything
+ NACLs are a great way of blocking a specific ip at subnet level.
+ Handson
    + We go to demo see and there is a default NACL created by default and it is associated will all 4 of our subnets.
    + In inbound rules, we see that there is a rule 100-0.0.0.0/0-Allow and another rule asterisk-0.0.0.0/0-Deny which means that allow all traffic from anywhere in first rule and 
      in the asterisk rule, deny all traffic when it does not match first rule. Same for outbound rules.
    + So we will run apache in our public ec2 instance, and allow 22 and 80 in security group of the ec2 from everywhere. Now if we hit public ip of ec2 instance, we will get response because it
      is using default NACL. If we delete outbound rule in security group of ec2, then also we will get response, because security group rules are stateful.
    + If we deny http in default NACL, by adding a rule like 80 - TCP - Port 80 - 0.0.0.0/0 - DENY, then our request to ec2 instance will be timing out, because 80 takes precedence over 100.
      If we edit this rule and make the number as 200 for the rule, then again we will get resposne because 100 takes precedence over 200.
    + We can also block specific ips from accessing EC2 by adding a lower numbered deny rule with specific ip range. This is also the primary usage of NACL.
    + In outbound rule of NACL, if we click deny, then also we will not get response from EC2 because NACL rules are stateless and both inbound and outbound rules get evaluated.
+ Difference between security groups and NACL
	+ https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html#VPC_Security_Comparison
	+ https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html

#### VPC Peering

+ It allows us to conenct 2 vpc directly, privately using AWS' network. It makes them behave as if they are in the same network.
+ The 2 VPCs must have non overlapping network.
+ To connect 2 VPCs, we must have a VPC peering connection between the two and update the route tables in each VPCs subnets so that instances in each VPC can communicate with one another.
+ VPC peering conenction is not transitive which means a peering connection must be set up for each VPCs that communicate with each other.
  So if we set up a peering conenction between VPC A and VPC B and another peering connection between VPC B and VPC C, that does not mean VPC A and VPC C are conencted. 
  To connect them we have to create a peering connection between them.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/VPCP.png" width="60%" height="60%"/>
+ VPC peering works inter region cross account
+ We can reference a security group of a peered VPC(also works cross account)
	+ HTTP - TCP - 80 - sg-00d2b0f(This security group rule references another secuity group)
	+ HTTP - TCP - 80 - sg-00d2b0f/12345678(This security group rule references another secuity group from another account)
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/VPCP1.png" width="60%" height="60%"/>
+ Now we are going to peer withour VPC with our default VPC. Our VPC has a CIDR of 10.0.0.0/16 and our default VPC has a CIDR of 172.31.0.0/16.
  So we will create a peering connection and the requestor VPC is the VPC we created and the acceptor VPC is the default VPC. Then we have to accept the connection.
+ Now we have to add routes in route table. So in the route table we created and associated with our VPC, we will add a route - 172.31.0.0/16 - select peering connection - active
  And then in the route table of our default VPC , we will add a route 10.0.0.0/16 - select peering connection - active

#### VPC Endpoints

+ How do we talk to different AWS Services like S3, DynamoDB which are inside AWS Cloud from an instance in our private subnet ?
	+ The instance will talk to a nat gateway which will talk to an internet gateway and then there is a internet route through public internet directly into dynamodb. But that is not optimal
	  because we need this traffic to stay private. So we will use VPC endpoints.
	+ VPC endpoints are used to access AWS services within a private network. This VPC endpoint may create an endpoint to S3 or cloudwatch etc and our instances from private network through 
	  some route table will be able to directly access S3 or Cloudwatch or other services privately. 
+ VPC endpoints allow us to connect to AWS services using a private network instead of a public internet.
+ They scale horizontally and are redundant.
+ They remove the need of IGW, NAT etc. to access AWS services.
+ There are 2 kinds of VPC endpoints.
	+ Interface VPC endpoint : They provide an ENI or a private IP address as an entry point and we must attach a security group to it.
	+ Gateway : This is to provision a target and it must be used in a route table - S3 and DynamoDB are 2 services that require a VPC gateway endpoint. 
+ In case of issues : 
	+ Check DNS resolution setting in VPC
	+ Check route tables
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/VPCE.png" width="60%" height="60%"/>
+ Handson
	+ We are going to start our private instance and attach to it an IAM role that has full access to S3.
	+ We will ssh to private instance from our public instance and if we do aws ls s3, we get all the buckets.
	+ Now we will cut off internet from the private instance by modifying route table and removing the nat gateway rule. Now aws s3 ls will not work any more.
	+ So we will create a VPC endpoint. We will select aws service as service category and choose service as s3 which is of type gateway. If we have a interface type service, we have to define subnet for endpoint
	  and whether we want to enable private DNS name and enableDNSHostname and enableDNSSupport must be true for our VPC. We also need to assign a security group for the endpoint. But for s3 we will need gateway type.
	  Here we will need to select a VPC and the route table(our private route table) and a rule will be added to the route table automatically. The rule will say that anytime we hit the s3 url,
	  go to the vpc endpoint target. Policy can be full or custom access. Still things will not work because default region of CLI is us-east-1 but the endpoint was provisioned in a different region. So if we run 
	  aws s3 ls --region eu-west-1 , we will get result.

#### VPC Flowlogs + Athena

+ Flowlogs help us capture information about IP traffic going into our interfaces. There are 3 kinds  of flow logs.
  	+ VPC Flow log which applies to VPC. If we define VPC flow logs, it will include subnet and ENI flow logs as well. 
  	+ Subnet Flow log applying to subnet
  	+ Elastic Network Interface Flow log applying to one ENI
+ It helps us debug (monitor and troubleshoot) connectivity issues.
+ Flow logs can go to cloudwatch or s3 bucket.
+ When we enable flow logs, it captures information about what we own as well as AWS managed services like ELB, RDS, Elasticache, Redshift, Workspaces etc. 
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/FL.png" width="60%" height="60%"/>
+ Flow Log Syntax
	+ <version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>
	+ <srcaddr> and <dstaddr> will help identify problematic ips
	+ <srcport> and <dstport> will help identify problematic ports
	+ Action - success or failure(accept or reject in flow logs) of the request due to security/nacl
	+ Can be used for analytics on usage patterns or malicious behavior.
	+ https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html#flow-log-records
	+ Query vpc flow logs using Athena on S3 or Cloudwatch Logs insights
+ Go to VPC and under flow logs, create flow log. The filter can be Accept/Reject or ALL. Destination can be cloudwatch or S3. 
	+ In cloudwatch, we will create log group and select the log group name in create flow log page. Then we will create an IAM role by clicking on setup permisisions. The click create. Takes some time to reflect. 
	+ If we select s3, we have to give bucket ARN and a resource based policy will be created and attached to the bucket automatically.
+ Now if we look at the cloudwatch log, we will see logs and if we open the logs , we will see a lot of rejects and some accept. To analyse this, we can go to Athena and create a table.
  https://docs.aws.amazon.com/athena/latest/ug/vpc-flow-logs.html
  ```
	CREATE EXTERNAL TABLE IF NOT EXISTS vpc_flow_logs 
	(
	  version int,
	  account string,
	  interfaceid string,
	  sourceaddress string,
	  destinationaddress string,
	  sourceport int,
	  destinationport int,
	  protocol int,
	  numpackets int,
	  numbytes bigint,
	  starttime int,
	  endtime int,
	  action string,
	  logstatus string
	)
	PARTITIONED BY (`date` date)
	ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ' '
	LOCATION 's3://your_log_bucket/prefix/AWSLogs/{account_id}/vpcflowlogs/{region_code}/'
	TBLPROPERTIES ("skip.header.line.count"="1");
  ```
  Now run the query and add a partition.
  ```
  	ALTER TABLE vpc_flow_logs
	ADD PARTITION (date='YYYY-MM-dd')
	location 's3://your_log_bucket/prefix/AWSLogs/{account_id}/vpcflowlogs/{region_code}/YYYY/MM/dd';
  ```
  Now we can run sql queries and analyse the data.

#### Bastion Hosts

+ We have our bastion host users and ssh into out bastion host in a public subnet and then ssh into our private instances from it.
+ So we can use a Bastion Host to SSH into our private instances.
+ The bastion host sits in the public subnet becuase the public subnet is conencted to all other private subnets.
+ Bastion Host security group must be very restricted to only allow the ips which we want to go in. We have to make sure that bastion host only has port 22 traffic allowed from the ip
  we need to SSH from, not from the security groups of other instances.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/BH.png" width="60%" height="60%"/>

#### Site to Site VPN, Virtual Private Gateway and Customer Gateway

+ We need to connect to our corporate data center to AWS cloud directly or indirectly. So to do a site to site vpn, which will establish a virtual private network that will behave as our corporate network and
  it will also that the AWS vpc is part of our corporate network, we have to create a customer gateway into the corporate data center and we will have to set this up as a software or hardware.
+ On the VPC side we will provision a VPN gateway. Then in between the VPN gateway and the customer gateway, we will set up a site to site VPN connection, that will link the two and our corporate data center
  and AWS VPC will be able to talk to each other.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/S2S.png" width="60%" height="60%"/>
+ Virtual Private Gateway
	+ VPN concentrator on the AWS side of VPN connection
	+ VGW(virtual private gateway) is created and attached to VPC from which we want to create the site-to-site VPN conenction. So it is at VPC level.
	+ It is possible to customize the ASN.
	+ So we will have to enter a tag and give ASN (Amazon Default or custom)
+ Customer Gateway
	+ Software application or physical device on customer side of VPN connection.
	+ https://docs.aws.amazon.com/vpc/latest/adminguide/Introduction.html#DevicesTested
	+ IP Address
		+ Use static, internet-routable ip adress of your customer gateway device
		+ If the gateway is behind a NAT with NAT-T enabled, we need to use the public ip adress of the NAT. The NAT is the NAT on customer side. 
	+ So we will need to enter a name, routing (static or dynamic) and ip address.
+ Site-to-Site VPN connection
	+ We will have to enter a name, select a virtual private gateway, the select a customer gateway and then give tunnel instructions. We can have 2 tunnels.

#### Direct connect and Direct Connect Gateway

+ The site to Site VPN internet traffic goes through public internet.
+ Instead we can also use a a dedicated private connection from a remote network i.e. our corporate data center to our VPC.
+ The dedicated conenction must be setup between our DC and one of AWS direct connect locations. 
+ On AWS side we need to set up a Virtual Private Gateway to use this.
+ Direct Conenct allows us to access public resources like S3 and private resources like EC2 on the same connection.
+ Use cases 
	+ Increase bandwidth throughput when working with large data sets and want lower cost on bandwidth
	+ More consistent network experience for applications using real-time data feeds.
	+ Hybrid environments of on prem and cloud
+ Supports both IPv4 and IPv6.
+ https://docs.aws.amazon.com/directconnect/latest/UserGuide/images/direct_connect_overview.png
+ Here our data center connects to a partner router which uses the AWS direct connect endpoint to communicate with AWS VGW. Usign the partner router, our corporate data center
  can talk to public resources like S3, Glacier etc. as well as private resources like EC2.
+ If we want to connect to multiple VPCs, we have to use a direct connect gateway.
	+ If we want to setup a DirectConnect to one or more VPCs in many different regions of same account, we use a direct connect gateway.
	+ https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-gateways-intro.html
	+ So using this, we can connect multiple VPCs in AWS to our corporate DC. These VPCs are not connected to each other and must not have overlapping CIDRs.
+ Direct Connect has 2 types of connections.
	+ Dedicated connection - 1 Gbps and 10 Gbps capacity
		+ There is a physical ethernet port dedicated to a customer
		+ A request needs to be made first to AWS for the ethernet port and then  contact AWS Direct Conenct partners to establish the connection.
		+ It takes a lot of time
	+ Hosted Connection - 50 Mbps, 500 Mbps to 10 Gbps
		+ Connection requests are made via AWS Direct Connect partners 
		+ Capacity can be added or removed on demand.
		+ So if we have a big migration to do, we can upgrade the connection for a week and increase capacity and then remove that capacity
		+ 1,2,5,10 Gbps is available at seelct AWS direct connect partners.
	+ Lead times are often very long for direct connect and longer than 1 month often to establish a new connection.
+ Direct Connect - Encryption
	+ Data in transit is not encrypted but it is private
	+ We can combine AWS Direct Connect with a VPN solution that provides IPsec encrypted private connection. This is good for extra level security but complex.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/DCE.png" width="60%" height="60%"/>
+ Handson
	+ Create a Virtual Private gateway with name tag and ASN
	+ Go to DirectConnect service, give a name and select a location and give speed for port.
	+ We can also create a direct connect gateway and give a name and ASN.

#### Egress Only Internet Gateway

+ This means outgoing only internet gateway. THis is for IPv6 only.
+ It is similar to NAT but NAT is for IPv4.
+ The Egress Only Internet Gateway will allow IPv6 instances to acess internet but they cannot be accessed from internet.
+ IPv6 are all public adresses. So all our IPv6 instances are all publicly accessible. So we have to use Egress Only Internet Gateway so that our IPv6 instances can 
  access internet but they are not reachable from internet.
+ We have to edit the route table.
+ We click on Egress Only Internet Gateway , select a VPC and click create. Now edit the route table, edit route, add route and add rule ::/0(any IPv6) and select target as egress only
  internet gateway. So we have added an outbound route for IPv6 adresses and the target is the Egress Only Internet Gateway.

#### AWS Privatelink - VPC Endpoint Services

+ We have a VPC in our account and inside that we have a service in which we created our application. We want to expose that application to other VPCs in other accounts.
	+ Option 1. We make our application public
		+ So we create an ALB and make it public , so all traffic goes through public internet.
		+ It is difficult to manage access.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/1.png" width="60%" height="60%"/>
	+ Option 2. Do VPC Peering
		+ We must create many peering relations between our VPC and customer VPC.
		+ That opens up the whole network. So all other applications in our VPC will be accessible form other VPCs.
		+ This is also not scalable.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/2.png" width="60%" height="60%"/>
+ So AWS came up with AWS Private link(VPC Endpoint Services)
	+ This is the most scalable and secure way to expose a service to 1000s of VPCs(own or other accounts)
	+ Does not require anything like VPC Peering, NAT, route tables, internet gateway etc.
	+ So we will have a NLB in Service VPC and ENI in customer VPC. These 2 will be connected through a AWS private link.
	+ Also if we have our NLB in multiple AZ and ENI in Multiple AZ, then our service becomes fault tolerant.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/PL.png" width="60%" height="60%"/>
+ We have to go to Endpoint Services under VPC where the application is and create Endpoint Service. Then we have to select the NLB in our account and create the service. Then we go to 
  Endpoints in VPC of customer and choose FindServiceByName and enter name of service.

#### AWS Classic Link and EC2-Classic(deprecated)

+ EC2-Classic : Instances run in a single network shared with other customers. This is not available any more.
+ Amazon VPC : Your instances run logically isolated in your AWS account.
+ ClassicLink allows us to link EC2-Classic instances to a VPC in our account.
	+ Must associate a security group
	+ Enables communication using private IPv4 adresses.
	+ Removes the need to use public IPv4 adresses or elastic IP.

#### VPN Cloudhub

+ Provides secure communications between sites if we have multiple VPN connections. So when we have 3 customers DCs(DC1, DC2 and DC3) all connected to a VPN Cloudhub, they are connected to each other as well.
+ Low cost hub and spoke model for primary or secondary network connectivity between locations.
+ It goes over the public internet but is encrypted.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/VCH.png" width="60%" height="60%"/>

#### Transit Gateway

+ This is for complex network topologies
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/COM.png" width="60%" height="60%"/>
+ So transit gateway is used to solve this kind of problem.  
	+ It is a star connection between all your VPCs and on premise infrastructure. So all VPCs in AWS, AWS Direct Connect Gateway and VPN connecttions are all conencted to the transit gateway.
	+ It is for transitivie peering between thousands of VPCs and on premise , hub and spoke star connection. So everything connected to Transit gateway can conenct to one another as well.
	+ The transit gateway is a regional resource but can work cross region as well.
	+ We can share it across accounts using RAM or resource access manager.
	+ We can peer transit gateways across regions.
	+ Route tables defines in transit gateway determiens which VPC can talk to other VPCs or which VPC can talk to on premise infrastructure.
	+ It works with direct connect gateways , vpn connections and any kind of network conenctions in AWS
	+ Supports IP multicast which is not supported by any other AWS services.
	+ Pricing - There are 2 components
		+ Pricing per AWS Transit gateway attachment - 5 cents per hour in US east -2 
		+ Pricing per GB of data processed - 2 cents in US east -2 
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/TG.png" width="60%" height="60%"/>
+ Under VPC, Go to transit gateway, then click create transit gateway. Enter name , description, ASN and enable/disable DNS support, VPN ECMP suppport, default route table association and propagation
  and create Transit gateway. Then go to Transit Gateway attachments to manage and create attachments. The attachments can be VPC, VPN or a peering connection and we have to give transit gateway ID.
  Then we go to Transit gateway route tables to define how each of these attachments can talk to each other. The Network Manager option helps us to manage our networks in AWS and on premise centrally and
  visualize them in a central dashboard. In ResourceAccessManager, we are able to share our transit gateway across multiple accounts by creating a resource share and selecting Transit Gateway.

#### VPC Summary

+ CIDR : IP Range
+ VPC : Virtual Private Cloud and we could define a list of IPv4 and/or IPv6 CIDR as needed.
+ Subnets : Tied to one specific Availability Zone and then within the subnet, we define a CIDR where our instance would be defined.
+ Internet Gateway :  we created one at the VPC level and that was used to provide IPv4 and IPv6 Internet Access, but that Internet Gateway did not work on it's own.
+ Route Table : Must be edited to add routes from the subnets directly to the Internet Gateway, VPC peering conenctions, VPC Endpoints etc.
+ Nat instances : Provide Internet Access for private instances in private subnets but the NAT Instances is the old way of doing things. We need to manage a CIDR entirely on our own
  and also disable the source destination check flag.
+ NAT Gateway : Managed by AWS and it provides scalable internet access to private instances but it is only for IPv4.
+ Private DNS + t 53 : We created private zone in Route 53 and we saw that this private zone would only work if we enable DNS resolution and DNS hostname at the VPC level.
+ Network ACL : They are stateless and they are subnet level rules and they are going to filter inbound and outbound traffic and when we define Network ACL, we shouldn't forget ephemeral ports,
  otherwise some traffic would not go through. Network ACL is a global type of firewall.
+ Security Groups : Stateful which means that if a traffic can go in, automatically the response can go out and the security group operates at the EC2 instance level, so they're more like 
  the second-line of defense.
+ VPC Peering : To connect 2 VPCs and they need to have non overlapping CIDR. It is non transitive. That means that if you peer A and B and B and C, A and C cannot talk to each other
  until they are also peered.
+ VPC Endpoints : To provide private access to AWS Services such as S3, DynamoDB, CloudFormation, SSM within our VPC without the help of an Internet Gateway.
+ VPC Flow Logs : Can be setup either at the VPC, Subnet, or ENI Level and we can filter for ACCEPT or REJECT type of request and this will help us identify attacks
  and we can analyze this using Athena because data will be in S3, or using CloudWatch Log insights if the data is in CloudWatch.
+ Bastion Host :  they were public instances that we would set up in our public Subnets and we would be able to SSH into those and then these public instances would have SSH connectivity
  directly to our private subnets and this is why it's called a Bastion Host.
+ Site to Site VPN : To connect our data center directly into the VPC and make it seem like they're part of one network and for this when you set up a Customer Gateway on our Data Center, 
  a Virtual Private Gateway on the VPC, and then we can link them using a site-to-site VPN connection. But this one is over the public internet.
+ As an alternative, more expensive feature, we have Direct Connect, where we also setup a Virtual Private Gateway on VPC and this time, we establish a direct connection to an AWS Direct Connect Location,
  so we have to actually set up the wired connection between our Data Center and this location, but then we would have a Private Connection directly into AWS' Network.
+ And then, if you wanted to do this, but on many different VPC, then, instead of setting up many different Direct Connects, we use Direct Connect Gateway, and basically that helps us connect to 
  many different VPCs in different regions. But, this is not the same as VPC Peering.
+ Finally, we wanted to provide NAT Gateway Facilities, but for IPv6, we would use an Egress only Internet Gateway. And this would allow us to provide our IPv6 Instances Internet Access while making sure they're
  not Internet reachable.
+ Private Link : Also called VPC Endpoint Services, which allowed us to connect privately our services from a service VPC to a customer VPC and it doesn't need VPC peering, it doesn't go over the public internet,
  it does not use NAT gateway, it does not use route tables, so it's very elegant and to use it, we must have a Network Load Balancer and an ENI.
+ ClassicLink to connect an EC2-Classic Instances privately into our Amazon VPC, which is, by the way, a deprecated service.
+ VPN CloudHub : to create hub-and-spoke VPN model to connect your own data centers through the public Internet.
+ Transit Gateway : transitive star type peering connection between VPC, VPN and Direct Conenct gateway or Direct Connect.

#### Networking Costs in AWS per GB

+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/VPC/images/NC.png" width="60%" height="60%"/>
+ We have a region and 2 availability zones in that region. 
	+ Let's assume we have an EC2 instance in the first AZ, so any traffic going into your EC2 instances is going to be free. So any incoming traffic onto EC2 is free.
	+ Let's assume we have a second EC2 instance and it is in the same availability zone. Then in that case because availability zone represents a set of multiple data centers
	  that are geographically located within one another, then any traffic between your 2 EC2 instances will be free, assuming that they are using their private IP adresses
	  to communicate, by using the private IP they will go over the network that they are connected with.
	+ Now let's include an EC2 instance in another AZ, and this time we want to have these 2 EC2 instances across two different AZ within the same region to communicate 
	  with one another. 
	  	+ One approach would be to use a public IP, or an elastic IP. And if we do so, we're going to pay 2 cents per gigabyte if using a public IP or elastic IP.
	  	  This is because the traffic has to leave the AWS network and go back in for our two instances to communicate and so AWS will charge us for that.
	  	+ Instead, if we are using a private IP, then we're going to be charged half as less, because we're now using the internal AWS network to link between these two 
	  	  availability zones.
	  So a take away here is that if you want to make your instances communicate one - faster, and two - at a lesser price, then use as much as possible the private IP
	  and not the public IP.
	+ Next, let's consider another region with another availability zone, in which case the traffic will need to go from one region to another. This is going to be 2 cents
	  per gigabyte. So that means that inter region traffic can be quite expensive.
+ So we should use a private IP instead of a public IP if you want have good savings and better network performance because you will automatically be in AWS private network.
  So do not use the public IP to communicate between two instances in the same region in 2 AZs.
+ If you have a cluster that does some computation and does require a lot of communication between your EC2 instances, then you may want to use the same availability zone
  for a maximum amount of saving. This cost saving come at a cost that you're not highly available anymore which means if your AZ goes down, then you don't have any fail over available,
  so here you have to balance between high availability and cost.
+ We have an RDS database and we want you to create a read replica and do some analytics on top of this read replica. How do we create a read replica for the cheapest amount of money.
  Well, if you create that read replica in the same AZ, then you're not going to be charged anything to replicate one database to another, in terms of network costs,
  but if you create that read replica in another AZ, then you're going to pay one cent per gigabyte of data transfer between the two databases.




      





























