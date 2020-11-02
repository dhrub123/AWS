## AWS Regions

AWS is a global service and it has datacenters all over the world. They are called regions.
https://aws.amazon.com/about-aws/global-infrastructure/

The blue dots are the existing regions and the orange dots are the upcoming ones. 
Each region is a set of datacenters. They follow a naming convention like us-east-1(N. Virginia), us-east-2(Ohio), us-west-1(N. California), us-west-2(Oregon), eu-west-3(Paris).

Most of the AWS services are region scoped( This means we can have a service in a specific region and that service in any other region will not have data 
replicated or synchronized). One such service is EC2 which is a regional service but there is a notable exception like IAM which is a global service.

**AWS AVAILABILITY ZONES** Each region has many availability zones. (Usually 3 availability zones, minimum 2, maximum 3).

Example: 
ap-southeast-2 is a region. This has 3 availability zones.
+ ap-southeast-2a
+ ap-southeast-2b
+ ap-southeast-2c
+ The letters a,b,c,d,e and f at the end represent availability zones.
+ Each availability zone is made up of one or more discreet data centers with redundant power, networking and connectivity.
+ They are seperated from one another and thus isolated from disaster.
+ They are connected with high bandwidth, ultra low latency networking.

A region physically nearer will have relatively low latency.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/REGIONS/Region_Basics.png" width="35%" height="35%"/>
  

