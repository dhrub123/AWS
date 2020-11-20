#### Cloudfront

+ CloudFront is a content delivery network or CDN. 
+ It improves read performance, because the content is going to be distributed and cached at the edge locations and edge locations are all around the world.
+ There's about 216 points of presence globally and they are growing in number. Edge Locations are much more than the 30 something regions that AWS has.
+ It gives us DDoS protection against distributed denial of service, integration with AWS shield and also AWS web application firewall.
+ It allow you to expose external HTTPS endpoint by loading the certificates and also talk to internal HTTPS backends.
+ https://aws.amazon.com/cloudfront/features/?nc=sn&loc=
+ So in this is a map of the world, there are some orange regions and their edges. If we have an S3 bucket in Australia and some user from America wants to access it,
  it's actually going to access an edge location close to it. So in America and that network is going to be transmitted over the private AWS network, all the way to the S3 buckets, and the content is going to be cached.
  So the idea is that this American user, with the more users you have in America, the more they will want to do the same kind of reads. And they will all have content served directly from America, not necessarily 
  from Australia, because it will be fetched once into America and then served from the cache locally. So another user, maybe in Asia, will talk to a edge location closer to Asia and that edge location again,
  will support traffic to the S3 buckets to get the content and then cache it at the edge.
+ So CloudFront allows you really to distribute your reads all around the world based on these different edge locations and improve latency and reduce the load on your main S3 buckets.
+ Cloudfront Origins
	+ The first one is an S3 bucket 
		+ You would use CloudFront in front of S3 as a very common pattern to distribute your files globally and cache them at the edge.
		+ You also get enhanced security, between CloudFront and your S3 buckets using CloudFront OAI or origin access identity. This allows your S3 bucket to only allow communication from CloudFront and from nowhere else.
		+ You could also use CloudFront as an ingress, to upload files into S3 from anywhere in the world.
	+ Custom Origin(HTTP endpoint) - This can be anything that respects the HTTP protocol
		+ Application load balancer
		+ EC2 instance
		+ An S3 website(But we first must enable the bucket as a static S3 website)
		+ Any HTTP Backend like  our own on-ßpremise infrastructure.
+ At a high level, we have a bunch of edge locations all around the globe and they're connected to the origin we defined, could be an S3 bucket or it could be any HTTP endpoint. Our clients wants to access our 
  CloudFront distribution. For doing this, the client will send an HTTP request directly into CloudFront. And then the edge location will forward the request to your origin and that includes the query strings
  and the headers, and then your origin response to the edge location. The edge location will cache the response based on the cache settings we've defined and return the response back to our clients.
  The next time another client makes a similar request, the edge location will first look into the cache before forwarding the request to the origin.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CF_HIGH.png" width="60%" height="60%"/>
+ S3 as an origin - You have your origin, S3 buckets. And for example, you have an edge location in Los Angeles and some users want to read some data from there. So your edge location is going to fetch the data
  from your S3 buckets over the private AWS network and give you the results from that edge location. The idea here is that for the edge location of CloudFront to access your S3 buckets using an OAI
  or an origin access identity which is an IAM role for your CloudFront origin. And using that role, edge location is going to access your S3 buckets and the bucket policy is going to say yes, this role is accessible and yes,
  send the file to CloudFront. So this works as well for other edge locations for example, in Sao Paulo in Brazil, or Mumbai, or Melbourne. And so all around the world, your edge locations are going to serve cached content
  from your S3 buckets and so we can see how CloudFront can work as a CDN.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CF_ORIGIN.png" width="60%" height="60%"/>
+ ALB or EC2 as Origin - If we use an ALB or EC2 as an origin, the security changes. EC2 instances must be public because they must be publicly accessible from HTTP standpoint and we have our users all around the world.
  So they will access our edge location which will access our EC2 instances and as you can see, it traverses the security group. So the security group must allow the IPs of CloudFront edge locations into the EC2 instance.
  And for this, there is a list of public IP for edge locations that you can get and the security group must allow all these public IP of edge locations to allow CloudFront to fetch content from your EC2 instances.
  If we use an ALB as an origin, we have a security group for the ALB and the ALB must be public to be accessible by CloudFront. But the backend EC2 instances now can be private. And so in terms of security group 
  for the EC2 instances, EC2 allow the security group of the load balancer, And for the edge location, which are again, public locations, it needs to access your ALB through the public network. And so that means that 
  your security group for your ALB must allow the public IP of the edge locations ,the same public IP as we had from before. 
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/ALB_ORIGIN.png" width="60%" height="60%"/>
+ Cloudfront for Geo Restriction - We can restrict who can access our distribution.
	 + We can provide a white list so users from this list of approved countries can go to a CloudFront and access content.
	 + We can provide blacklist, where the users from these countries are not allowed to access our distribution.
	 + The way the country is determined, is by using a third party Geo-IP database where the incoming IP is matched against it to figure out the country.
	 + Use case for jurisdiction will be when you have copyright laws to prevent access to your content. And you want to prove to regulators that you are indeed restricting content access from, say, France if you have content in America.
+ Difference between CloudFront and S3 cross region replication 
	+ CloudFront 
		+ It is using a global edge network.
		+ Files are going to be cached for a TTL(maybe a day).
		+ Great when you have static content that must be available everywhere around the world
		+ We are okay if that content is outdated for a little bit.
		+ This is for caching globally.
	+ S3 cross region replication
		+ It must be set up for each region in which you want to have replication to happen.
		+ The files will be updated in near real time
		+ It's going to be read only so is going to help you with read performance.
		+ It is great if you have dynamic content that needs to be available at low latency in a few regions.
		+ This is for replication into select regions.
+ We are going to create a bucket and a cloudfront distribution in front of that bucket. We are also going to create an Origin Access Identity so this is a user of CloudFront that will be accessing our S3 bucket and we'll limit the S3 bucket 
  to be only accessed using this identity/user. So effectively, we are ensuring that no one can access our S3 bucket except if they go through CloudFront. We can do this for many reasons like monitoring because maybe you have cookies,
  maybe because of some policies, etc. So we create a bucket and upload some files. Then we go to cloudfront and create a distribution of type web. The origin domain name will be our bucket and we will give an id to the origin.
  Then we can restrict bucket access and in that case we have to create n Origin access Identity and give it a name. Then we need to grant read permission of the S3 bucket to the OAI. We have more options and then we create the distribution
  which takes about 10 minutes to get created. If we go to the origin s3 bucket and look at the bucket policy, we can see that a new policy has been automatically created for us which states that the OAI we created earlier
  can get object from the bucket. We can update the policy to add a new rule to deny any requests coming from anything other than the OAI. Now for sometime cloudfront will keep redirecting us to the s3 bucket as the DNS takes some time to propagate properly. It is a common problem and not a bucket. We can temprarily make the files public in our S3 bucket. But once DNS propagation is complete and cloudfront stops redirecting to S3, we can make them private again and we will
  be able to access the files from cloudfront.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CLOUDFRONT.png" width="60%" height="60%"/>

#### Cloudfront signed URL

We want to make a cloudfront distribution private
+ WE want to give access to people to premium paid shared content all over the world but you want to be able to see and know who has access to what on your CloudFront distribution.
+ For this you can use a CloudFront signed URL or signed cookie. So first when we create a URL and a cookie, you need to attach a policy and you need to tell
	+ when does the URL or the cookie expire
	+ what IP ranges can access this data 
	+ the trusted signers(Which AWS account can create signed URLs for your users)
+ How long should this URL be valid for ? 
	+ If you're sharing a content like movie or music , you can make it very short a few minutes
	+ But if it's content that is private to the user, that they will access in a long period of time, you can make that URL or signed cookie last for years.
+ Difference between a URL and a cookie
	+ A signed URL gives access to individual files so you get one signed URL per file so, if you have a hundred files , you get a hundred URLs
	+ If you have a signed cookie, then you get access to multiple files and the cookie can be reused so this time you have one signed cookie for many files.
	+ Choose whatever you need based on the context.
+ How does signed URL work - We have our CloudFront distribution and a bunch of edge locations which can access our S3 bucket through OAI, Origin Access Identity for full security. And so that means the objects in our S3 bucket
  cannot be accessed by anything else, but CloudFront, but we still want to be able to give people access to their objects through CloudFronts. So we have our clients and our client is going to authorize and authenticate 
  to our application and we have to code that application. And our application will use the AWS SDK to generate a signed URL directly from CloudFront. It will return the signed URL to the clients and then the client will be able 
  to use that signed URL to get the data and files and objects or whatever he needs directly from CloudFront. This works for signed URL, but this also works for signed cookie obviously.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CFSURL.png" width="60%" height="60%"/>
+ Difference between CloudFront signed URL or an S3 pre-signed URL
	+ CloudFront signed URL 
		+ It allows access to a path no matter the origin, so signed URL works not just for S3 as an origin, but HTTP and other backend as well.
		+ It's an account wide-key-pair, so only the root can manage it.
		+ We can filter by IP, path, date, and expiration.
		+ We can leverage all the caching features out of CloudFront.
		+ If you want people to have access to your CloudFront distribution and it's in front of S3, you have to use a signed URL because you cannot access your S3 bucket as you should because there is a bucket policy restricting it 
		  to the OAI
	+ S3 pre-signed URL 
		+ It issues a request as a person who pre-signed the URL. 
		+ If I sign the URL, with my own IAM principal and then use my IAM key to sign this, then the person who has that URL has the same rights as me.
		+ It has a limited lifetime
		+ The client can access directly your S3 bucket directly using that pre-signed URL.
		+ If your users are using directly against S3 and you want to distribute a file in S3 directly without using CloudFront, then pre-signed URL 
		  would be a great use case for it.
		  
|Cloudfront signed URL|S3 Pre signed URL|
|---------------------|-----------------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CFSU.png" width="60%" height="60%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/S3PSU.png" width="60%" height="60%"/>|

#### AWS Global Accelerator

+ The specific problem we are trying to solve here is that we have deployed an application and it's global and you've global users who want to access it directly. But our application is only deployed in one region.
  For example here in India, I have deployed a public ALB. But my users are all over the world. They're in America, in Europe, in Australia.
+ As they access the application, they have to go over the public internet.And that can add a lot of latency due to many hops through the routers. 
+ These hops introduce a bit of risk because connection can get lost, they also add a little bit of latency, and they're not as direct as possible into our amazon infrastructure.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/PROBLEM.png" width="60%" height="60%"/>
+ So we want to go as fast as possible through the AWS network to minimize latency. 
+ Unicast IP vs Anycast IP
	+ Unicast IP : One server holds one IP address. So our clients when they talk to two servers, one has starting IP address  of 12 and the other one has a starting ip of 98. 
	  If you refer to the IP address that begin with 12 , we will be sent to that particular server. And for the other one if you use the other IP , then we will go to the other server.
	+ Anycast IP : All servers will hold the same IP address and the client will be routed to the nearest one. So our client has two servers and these two servers have the same IP. When our client when it tries to connect 
	  to this Anycast IP, it will be sent to the server that is the closest to itself. 
+ Global Accelerator uses that Anycast IP concept to work.
	+ We're able to leverage the AWS internal global network to route to our application.
	+ We want to route to India and we have users all around the globe. Instead of sending it through the public internet in America, it's going to come to the closest edge location. And from edge location,
	  it's going to go all the way straight to our ALB in India through the internal AWS network. Same for Australia, so it goes to closest edge location near to Australia and then it goes over the private AWS network to get to the ALB in India and same for Europe. 
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/SOLUTION.png" width="60%" height="60%"/>
	+ We're going to use an Anycast IP and there's actually going to be 2 of those that are going to be created for your application and they're global.
	+ The Anycast IP will send the traffic directly to the closest edge location of your users. 
	+ The edge location will then send the traffic to you application, through the private AWS network which is much more stable, has less latency and so on.
	+ It is unique because it really allows to give two static IP addresses all around the globe for the users for whatever application you may have. 
	+ And right now I'm showing one ALB in one region but it could be global as well, it could be multiple ALBs in multiple regions.
	+ It works with Elastic IP, EC2 instances, Application Load Balancer, Network Load Balancer and they can either be public or private.
	+ There is consistent performance because we go over the network
		+ We have intelligent routing to the lowest latency edge location and we'll have fast regional failover in case anything goes wrong.
		+ There's no issue with client cache because the client doesn't cache anything. The IPs, the two Anycast IP we're using don't change.
		+ It's internal AWS network to go after the edge location
	+ We have health checks.
		+ The Global Accelerator will perform a health check on your applications.
		+ It will ensure that the application is global and if health check fails for one ALB and one region , then there is automated failover in less than one minute to a healthy end point.
		+ Great for disaster recovery, thanks, to the health checks.
	+ Security - It is appropriately secured
		+ We only have two external IPs that needs to be whitelisted by your clients.
		+ We get DDoS protection automatically through the Global Accelerator by AWS Shield.
+ Difference between global accelerator and cloudfront
	+ Global Accelerator and CloudFront both use the same global network and they will both use edge locations all around the globe that AWS has created.
	+ They both integrate with Shield for DDoS protection 
	+ CloudFront 
		+ Improve the performance for both cacheable content such as images and video
		+ Dynamic content such as, API acceleration and dynamic site delivery
		+ The content is going to be served from the edge locations. So once in a while the edge locations are going to fetch the content from the origin 
		  but most of the time hopefully CloudFront will deliver cache content from the edges. So here the users are getting content from the edges.
	+ Global Accelerator
		+ It improves the performance of a wide range of applications over TCP or UDP.
		+ The packets are being proxied from the edge locations to the applications running in two one or more AWS regions. So in that case, all the request still make it 
		  to our application end. There is no caching available.
		+ So it's a really good fit if you have non-HTTP uses cases, such as gaming, IoT or Voice over IP 
		+ It's also really good if you have HTTP use cases that require static IP addresses globally.
		+ You need to have deterministic and fast regional failover.
+ Go to global accelerator and you will be taken to Oregon(us-west-2), no matter the region. We are going to create a EC2 instance in US-east-1 with a httpd service showing some data. 
  We can create another instance in mumbai. We do not need to associate key pairs with these instances because we will not be doing ssh into it. Then we create accelerator and we have to add
  listeners(port and protocol - 80 and TCP). THen we have to add endpoint groups. We will have to give region and traffic dial(amount of traffic) and healthcheck. So we add 2 endpoint groups 
  here - one for us-east-1 and one for mumbai(ap-south-1). The endpoint type can be ALB, NLB, EC2 or Elastic IP. So we specify our EC2 instance ID in endpoint info and add endpoint. Then we create the 
  global accelerator. After the accelerato is created, we can see 2 static anycast ip adresses to access our application. We can use the accelerator DNS name to access our apps. If we access the DNS from india,
  the EC2 in india will serve traffic and if we access from US, we will get the EC2 in us-east-1 serving traffic. If one of our ec2 instances are unhealthy, the failover to other region will be instant.
  We will disable and delete the accelerator. We have a fixed fee of $ 0.025 for every full or partial hour until it is deleted and a data transfer fee depending on region and destination.

#### Some questions

+ You are creating an application that is going to expose an HTTP REST API. There is a need to provide request routing rules at the HTTP level. Due to security requirements, your application can only be exposed through the use of two static 
  IPs. How can you create a solution that validates these requirements?
	+ Global Accelerator will provide us with the two static IP, and the ALB will provide use with the HTTP routing rules. So we will use Global Accelerator and ALB.
+ You are hosting highly dynamic content in Amazon S3 in us-east-1. Recently, there has been a need to make that data available with low latency in Singapore. What do you recommend using?
	+ S3 CRR
+ You would like to provide your users access to hundreds of private files in your CloudFront distribution, which is fronting an HTTP web server behind an application load balancer. What should you use? 
	+ Cloudfront signed cookies because they provide access to multiple files.

#### Snowball

+ Snowball is a huge box and that allows you to basically physically transport data in and out of AWS. The scale of data is terabytes or petabytes of data.
+ It's basically an alternative to moving data over the network and you avoid paying network fees.
+ If you have a lot of data, if it's huge and you need to transfer it, from on premise, all the way to the Amazon Cloud, maybe sometimes it's better to actually use this giant box called a Snowball,
  send it to AWS and load it this way.
+ It is secure, tamper resistant, there is KMS 256 bit encryption on it.
+ There's a tracking using SNS and text messages and there's a E-ink shipping label
+ You're going to pay per data transfer job.
+ The use case of Snowball would be to do large cloud migrations or decommission a DC or do disaster recovery.
  If it takes more than a week to transfer over the network, you're probably better off using a Snowball device.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/SNOWBALL.png" width="60%" height="60%"/>

How does snowball work?
+ You're going to request a Snowball device from Amazon Console and it's going to be delivered to you.
+ Then you install the Snowball client onto your servers.
+ You connect the snowball to your servers and then you copy the file using the Snowball client.
+ Then you ship back the device when you're done. It goes right away to the right AWS facility thanks to the E-ink shipping label.
+ The data will be loaded for you into an S3 bucket.
+ And then the snowball will be completely wiped, so that no one can access your data.
+ All the tracking is done, using SNS, text messages and the AWS console.

To visualize the process - 
You can directly upload the data to S3, over the internet, you have a 10 gigabyte per second broadband, but maybe it's not enough. Maybe you have petabytes of data
and so you order a snowball, it comes to you, you load the client into your servers and connect snowball to your servers and then you ship the Snowball box. You have the import export feature
directly done by AWS and your data will be in an Amazon S3 bucket.

#### Snowball Edge

It is a Snowball but it's improved.
+ It adds computational capability to the snowball device.
+ You can have 100 terabytes of capacity.
	+ You can either have it storage optimized - so you get 24 vCPUs
	+ Compute optimized -  52 vCPUs and optional GPU.
+ You can load a custom EC2 AMI on it and custom Lambda functions, and so you can support, basically perform computations while your data is moving.
+ Your Snowball device will actually perform computations for you and save you time while moving in a truck.
+ It's quite useful if you want to pre-process the data while the thing is moving.
+ The use case would be data migration, image collation, IoT capture or machine learning.

#### Snowmobile

+ If you have more than 100 terabytes say petabytes of data, then there is a Snowmobile which is a truck.
+ You can transfer exabytes of data with it.(1 EB = 1,000 PB = 1,000,000 TB)
+ And each Snowmobile itself will have 100 petabytes of capacity and we can have multiple of these trucks come into your facility in parallel. 
+ So it's usually better than Snowball if you transfer more than 10 petabytes of data at a time.
+ But obviously now you need to have a truck on your facility and load the data into there.

#### Important Points
+ If you want to transfer a very large amount of data, you should use a Snowball and if it's an insanely high amount of data then probably a Snowmobile.
+ How do we get data from Snowball into Glacier? - **Snowball can not import data into Amazon Glacier directly.** We have to use Amazon S3 first, and then S3 lifecycle policy to transition that data directly
  and immediately into glacier. So Snowball will import data into Amazon S3 because Amazon S3 is the only place where Snowball can drop its data  and then, using an S3 lifecycle policy, we will migrate that 
  data into Amazon Glacier and effectively, we'll have made Snowball do put data into Amazon Glacier, but that involves an extra step into Amazon S3 in the middle. 

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/SNOWBALL_S3.png" width="60%" height="60%"/>

#### Snowball Handson

Go to Snowball > Create a job(Import into S3 / Export from S3 / Local Compute and Storage only) - Import - Shipping Adress and Shipping Speed >  Job Name and Snowball Type, Give bucket name, You can also load an AMI into 
Snowball > Set Security > Permission(IAM Role - which allows snowball to import data into s3) and Encryption > Notifications(Email or SNS) > Create Job

#### Storage gateway

+ Hybrid Cloud means that 
	+ Part of your infrastructure will be on the cloud on AWS 
	+ Part of your infrastructure will also be on-premise.
+ This can be due to many reasons 
	+ long cloud migration
	+ security requirements or compliance requirements
	+ your strategy to be half and half
+ S3 is a proprietary storage technology. It's not like NFS or EFS, which is standardized. So how do we expose the S3 data when we are working with on-premise servers or on-premise computers?
+ So that's the whole idea behind AWS Storage Gateway. It's going to give us access to S3 through a gateway which will expose standard API's.
+ We have 3 types of cloud native storage now.
	+ Now we have Block Storage which is EBS or EC2 Instance Store, that's basically our volumes.
	+ Then we have file storage. That's when we dealt with EFS and we're storing files on a network file system.
	+ Then we have object when we were storing files and objects directly on S3 and Glacier.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/TYPES.png" width="60%" height="60%"/>
+ Storage Gateway will bridge between on premise data and cloud data in s3
+ The use cases are where we want to bridge on-premise data into S3 ,disaster recovery, back up and restore, or maybe tiered storage.
+ We get three types of Storage Gateway
	+ File Gateway which will basically allow us to view files on our local file system on-premise, but it will be backed by S3.
	+ There will be Volume Gateway to do the exact same thing, but with volumes.
	+ There will be Tapes for backups and recovery.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/BRIDGE.png" width="60%" height="60%"/>
	+ All these things are going into the Storage Gateway and it will go straight into EBS, S3 or Glacier, but behind the scenes. Storage Gateway will do this for us.
+ File Gateway is when you have S3 buckets and you want them to be accessible using maybe the NFS protocol or the SMB protocol. So NFS is for Network File System, and here it will
  expose an S3 bucket using NFS. 
  	+ This will support S3 standard, S3 IA, One Zone IA
  	+ Each bucket will need to be accessed by the File Gateway and it will have it's own IAM role.
  	+ The most recently used data will be cached into the File Gateway. The File Gateway will take our most active S3 objects and cache them locally.
  	+ This File Gateway, because it's NFS, it can be mounted on many servers.
  	+ We have our data center and our Application Server. It will to use the NFS protocol for Network File System, maybe v3 or v4.1. and it's going to talk to the File Gateway.
  	  So we have to setup a File Gateway on-premise. The File Gateway automatically goes and talks to the AWS cloud to S3, S3IA, or Glacier and basically gets the files we need
  	  and caches them locally onto the File Gateway. From our applications perspective, it seems like we're talking to a local network file system, but the File Gateway actually does
  	  some magic behind the scenes and talks to S3 or Glacier.
  	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/FG.png" width="60%" height="60%"/>
 + Volume Gateway - This is when you want to have Block storage.
 	+ Block storage using iSCSI protocol and they will be backed by S3.
 	+ The EBS snapshots will be made from time to time and they will be in S3. This will help us restore on-premise volumes if we wanted to.
 		+ Cached volumes - They give you a low latency access with the most recent data on your volumes
 		+ Stored volume which is going to be an entire dataset that will be on-premise and it will have scheduled backups to S3. So from time to time it'll go to S3.
 		+ We have our customer premise and usually we mount volume using the iSCSI protocol. Our application server is going to mount a volume from the Volume Gateway
 		  and for on-premise, it will look like a local volume, but the magic happens again with the Volume Gateway. It basically will store this as Amazon EBS snapshots backed by S3.
 		  We provide on-premise servers access to Volume Storage, Block storage, and Volume Gateway scales becuase of being backed by Amazon Cloud.
 		  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/VG.png" width="60%" height="60%"/>
 + Tape Gateway - Some companies still have processes to use physical tapes. 
 	+ With Tape Gateway, basically you use the same processes but it's going to be backed in the cloud.
 	+ You build a VTL, or Virtual Tape Library and it will be backed by Amazon S3 and Glacier.
 	+ The idea is that if you have existing tape-based processes or software and they use sometimes the iSCSI interface then it will work as well with Tape Gateway.
 	+ Works wil leading backup software vendors.
 	+ Tape Gateway is for a backup reason and so we would have a backup server and a Backup software and it will be connecting directly using iSCSI to the Tape Gateway and automatically
 	  the Tape Gateway is smart and will create a Virtual Tape library stored in S3 or Glacier.
 	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/TG.png" width="60%" height="60%"/>
 + Exam Tips
 	+ General question around we need to bridge on-premise data into the cloud, think Storage Gateway at a high level.
 	+ Then if it says we want to have file access or NFS, think File Gateway which is backed by S3.
 	+ If we want volume or Block Storage and there is iSCSI in the question, think Volume Gateway, it will be backed by S3 with EBS snapshots,
 	+ If you see VTL Tape solution, if you see Backup, the word Backup with iSCSI, think Tape Gateway.
 	+ Storage Gateway bridges on-premise data to the cloud.
 	+ File access or NFS will be File Gateway.
 	+ Volume, Block Storage, iSCSI will be Volume Gateway.
 	+ VTL Tape Solution, Backup with iSCSI will be Tape Gateway.
 + File Gateway Hardware Appliance
 	+ While using the file gateway, you need to use visualization, but you have an alternative.
 	+ You can also install a file gateway hardware appliance directly on premises to synchronize your files from on premises into AWS.
 	+ In the console, you'll see the host platform. The first four options are going to be virtual appliances, but the last one is a hardware appliance,
 	  and you can but it at amazon.com.
 	+ You can plug it in your own small data center and have it do the job of transferring your files from your on premises environment into AWS through the NFS.
 	+ The use case for it is that if you have a small data center with no visualization capability and you still need to perform a daily NFS backup, then you need to buy a hardware appliance
 	  and plug it onto your small data center, and using the hardware appliance, you can do your daily NFS backup.
  + Go to Storage Gateway and choose type of gateway. We can choose 2 types for volume gateway - cached volume and stored volume. For file gateway, we need to select host platform and give IP adress.
    Then activate it and configure local disks.

#### Amazon FSx

We have a newer kind of storage offering from AWS, which is called Amazon FSx. It in two different flavors.
	+ Amazon FSx for Windows File Server
		+ EFS is a shared POSIX file system. It can only be used by Linux EC2 instances, or on-premise machines. Therefore, you can not use EFS with your Windows servers.
		  So how do you share storage between your Windows servers? 
		+ Amazon came up with FSx for Windows. It's a fully managed Windows file system shared drive.
		+ It supports the SMB protocol and Windows NTFS.
		+ It supports Active Directory integration , ACLs, and user quotas.
		+ It's built on top of SSD, it has a massive scale, it can scale to 10s of GB/s, millions of IOPS, and 100s of PBs of data.
		+ So it's a scalable distiributed file system for for Windows, that is managed by AWS.
		+ It can also be accessed from your on-premise infrastructure.
		+ It can be configured to be Multi-AZ and gets high availability.
		+ Data is backed up daily to Amazon S3, so you can always recover your file system directly from S3.
		+ Anytime you have shared storage for your Windows instances, it is Amazon FSx for Windows.
		+ FSx for Windows is going to be for a distributable file system for your Windows instances
	+ Amazon FSx for Lustre
		+ Lustre is a type of parallel distributed file system for large-scale computing. Lustre is derived from the term Linux and cluster.
		  So Lustre is for Linux instances, and because it comes from cluster it's meant for large-scale computing.
		+ We use Lustre for Machine Learning, High Performance Computing or HPC.
		+ Anytime you need a file system to perform these High Performance Computing, use Lustre.
		+ We can also do video processing, financial modeling, electronic design animation, anything that requires a high level of distribution for your file system and your computation.
		+ It scales up to 100 of GB/s, millions of IOPS, and has sub-millisecond latencies. So it is really meant for High Performance Computing, or HPC, and has a seamless integration with S3.
		+ We can read your S3 like it's a file system through FSx for Lustre, and you can write the output of whatever computation you're doing back to S3, again through using FSx for Lustre.
		+ So FSx for Lustre is a way to expose your S3 buckets, as a file system as well, to your Linux instances.
		+ It can also be used from on-premise servers.
		+ It is going to be for Linux, and it's going to have a cluster, a High Performance Computing cluster, that has a file system that is shared, with high IOPS, high throughputs,
		  very low latency, and integration with S3 as a backend
	+ Go to sx > Create File System - 2 options(Windows and Lustre), File System Name, Deployment Type(Multi AZ or Single AZ for HA), Storage Capacity(For windows min is 32 GB and max is 65536 GB),
	  specify throughput capacity, we can use recommended throughput capacity or specify throughput capacity(We can go up from 8 MB/s to 2048 MB/s). The FSx for Windows is not elastic, we have to provision
	  this in advance. We have to attach a Security group and we have a preferred or standard subnet. Then we have Windows Authentication(AWS or self managed), Encryption.
	  For Lustre, Storage capacities are 1200 GB, 2400 GB or increments of 3600 GB. Throughput capacity = storage capacity in TB * 200 MB/s/TB, so for storage capacity of 10 TB, we will get a throughput of 2 GB/s.
	  We can increase capacity and insane throughput as a result. We then have network and security(SG, Default VPC and subnet), this is encrypted by default at rest with keys managed by FSx using
	  XTS-AES-256 block cipher, we can integrate with S3 for data source of file system.

#### Comparison among different file systems

+ S3 is going to be an object storage, it's going to be serverless, you don't have to provision capacity ahead of time. It has some deep integration with many database services.
+ Glacier is going to be for object archival. So this is when we want to store objects for a long period of time, Retrieve it very very rarely, and when we retrieve these objects, 
  they're going to be taking a lot of time to get back to us because they are archived.
+ EFS is Elastic File System, and this is a network file system for Linux instances. It is a POSIX file system so that means for Linux again. And it is accessible from all your EC2 instances at once.
  It is something that is going to be shared and across AZ. 
+ FSx for Windows is the same thing as EFS, but for Windows. So it's a network file system for your Windows servers.
+ FSx for Lustre is Linux and cluster, so it's for High Performance Computing Linux file system. This is where you're going to do your HPC running. You have insanely high IOPS, insanely big capacity.
  And it has integration with S3 in the back end.
+ EBS volumes is your network storage for one EC2 instance at a time only. And it is bound to a specific availability zone that you create it in. And in case you wanted to change the AZ, you will need to 
  create a snapshot, move that snapshot over, and create a volume from it.
+ Instance Storage is going to be physical storage for your EC2 instance. Because it's attached from the hardware, then it's going to have a much higher IOPS than EBS. EBS volumes
  is up to 16,000 IOPS or 64,000 IOPS for io1. 
+ But for Instance Storage, because it is physically attached to your EC2 instance, you can get, for some, millions of IOPS. It's going to be very high. But the risk is that if your EC2 instance goes down,
  then you will lose that storage permanently.
+ Storage Gateway is going to be transporting files from on premise to AWS. So we have File Gateway, Volume Gateway for cache and stored, and Tape Gateway.
+ Snowball/Snowmobile to move large amount of data to the cloud physically into S3.
+ Database which is a way of storing data. It's for more specific workloads, and usually it's going to be mixed with some indexing and some querying.






