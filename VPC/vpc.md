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




