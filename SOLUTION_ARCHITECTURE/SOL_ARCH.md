## CLASSIC SOLUTION ARCHITECTURE

#### UseCase1 Stateless: WhatIsTheTime.com

WhatIsTheTime.com allows people to know what time it is. We do not need a database and each instance, each server knows what time it is, and we want to start small. We're willing to accept downtime now 
but when our app will get more and more popular,  we'll need to scale vertically and horizontally and remove downtime.

+ 1) You have a t2.micro instance and you have a user, and the user says what time it is, and say okay, it's 5:30 p.m. This is my app. So we have a public EC2 instance, and because we wanna make the EC2 
     instance have a static IP address in case something happens and we need to restart it, then I will attach an elastic IP address to it.
+ 2) Now our application is getting more and more traffic, and certainly, the t2.micro instance isn't enough, and so, as a solution architect, we should replace that t2.micro instance by something a 
     little bit bigger to handle the load, so that's called vertical scaling. Maybe we'll make it an m5.large type of instance. So we stop the instance, change the instance type, and then, we start again 
     the instance. So now there is an M5 type of instance which has the same public IP because it has an elastic IP address, so people are still able to access our application, but we have experienced downtime
     while upgrading to an M5, and so, our users were not really happy since during that moment, they were not able to access our application.
+ 3) Next, we're going really popular, and it's time to scale horizontally, so we start adding EC2 instances all m5.large, and they all have an elastic IP attached to it. So now, on top of having three EC2 
     instances, we have three elastic IP, and so our users, they need to be aware of the, the exact values of these three elastic IP to talk to our instances, and so, that's called horizontal scaling.
     We are starting kinda reach some limits. Now the users need to be aware of more and more IPs, and we have to manage more infrastructure.
+ 4) Let's change the approach. Now we have three EC2 instances, M5, and let's remove elastic IP because it's something that we can't really manage and There's only five elastic IP per region per account
     by default so it's not real lot, and so, instead, our users, are going to leverage Route 53, so we've set up Route 53, and the website URL is api.whatisthetime.com, and we've decided it's going to be 
     an A Record with a TTL of one hour. An A Record means that from a DNS like this, it's going to give me a list of IPs. The users query Route 53, and then, they get the IP addresses of our EC2 instances
     and they can change over time. Route 53 will get updated and so, our users are now able to access our EC2 instances, and we don't have any elastic IP to manage anymore. But we wanna be able to scale,
     and be able to add and remove instances on the fly, and so, when we do remove an instance, it seems like the users that were talking to the m5.large instance will not get a response because it's gone,
     and it turns out that if they do a Route 53 query, because the TTL was one hour, they're using the same response for one hour. So for one hour, they'll try to connect to that instance, and that 
     instance is gone, and after one hour, these users will be able to connect to a new instances, and they think that our application is down.
+ 5) So let us add a load balancer. We don't have public instances anymore. We have private EC2 instances, and we're going to launch them in the same Availability Zone manually. We have three m5.large 
	 instances. We will use a load balancer and add health checks such as if one instance is down or not working, at least we won't send traffic from our users to the instance. So my ELB is going to be 
	 public-facing, whereas my private EC2 instances are in the back, and so, they restrict traffic between these two using maybe a security group rule that we've seen before, using security group as a 
	 reference. Our users, are going to query for WhatIsTheTime.com, but this time, it cannot be a A Record because a load balancer has its IP changing all the time, and so, instead, because it's a load 
	 balancer, we can use an Alias Record, and this Alias Record will point from Route 53 to the ELB. So users connect to our load balancer, and our load balancers redirects us to our EC2 instances
	 and balances the traffic out and it's really great 'cause now we can add and remove these instances without downtime for our users thanks to the health checks feature of ELB.
+ 6) But now, adding and removing instances manually is pretty, pretty hard to do, so we'll launch an auto-scaling group. So we have the same thing, a Route 53, ELB, Availability Zone, private EC2 instances,
     but they're going to be managed by an auto-scaling group which basically scales on demand. So now we have an application, no downtime, auto-scaling, load-balanced. It is going to provide us with even
     better costing because we will have just the right amount of ec2 instances at any time.
+ 7) But There's an earthquake that happens, and Availability 1 goes down, Zone 1 goes down, and our application is entirely down because we haven't implemented a multi-AZ application for highly available app.
	 So now we're gonna have to ELB, and on top of having health checks, it's also going to be multi-AZ, and it's going to be launched on AZ 1 to 3, so three AZs for this ELB, and our auto-scaling group as well
	 is going to span across multiple AZ, and this allows us maybe to have two instances in AZ 1, two instances in AZ 2, and one instance in AZ 3. So if AZ 1 goes down, we'll still have AZ 2 and AZ 3 to serve 
	 our traffic to our users, and we've effectively made our app multi-AZ and highly available and resilient to failure.
+ 8) Now we have two AZ, and we know that at least one instance will be running in each AZ, so we can reserve capacity and diminish the cost of our application because we know that for sure, two instances must 
	 be running at all time during the year. And so, by reserving instance, maybe for the minimum capacity of our auto-scaling group, then we're going to save a lot of cost in the future whereas the new instances 
	 that get launched, maybe they're gonna be temporary, so on demand is fine, or if we're a bit crazy, we could even use spot instances for a less, less cost, but we might have the instances being terminated.

|Step 1|Step 2|
|------|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_1.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_2.png" width="50%" height="50%"/>|


|Step 3|Step4|
|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_3.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_4.png" width="50%" height="50%"/>|


|Step 5|Step 6|
|------|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_5.png" width="70%" height="25%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_6.png" width="70%" height="25%"/>|

|Step 7|Step8|
|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_7.png" width="75%" height="25%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_8.png" width="75%" height="25%"/>|


An Well-Architected Framework in AWS, has five pillars to it and they are cost, performance, reliability, security, and operational excellence.
+ **cost** - we're scaling up our instance vertically, Maybe we're using ASG to just have the right amount of instances based on the load, and maybe we wanna reserve instances as well to optimize cost.
+ **performance** - we've seen vertical scaling. We've seen ELBs, we've seen auto-scaling groups, basically how we can adapt to performance over time.
+ **reliability** -  we have seen how Route 53 can be used to basically reliably direct the traffic to the right EC2 instances, and maybe using multi-AZ for the ELB and multi-AZ for the ASG as well.
+ **security** - we can use security groups to basically link the load balancer to our instances reliably.
+ **operational excellence** - we can evolve from a very clunky, manual process all the way to having it fully automated with auto-scaling groups.

####  Usecase2 Stateful: Myclothes.com

MyClothes.com allows people to buy clothes online and there is a shopping cart when you navigate MyClothes.com and we're having hundreds of users at the same time so all these users are navigating the website and we wanna be able to 
scale, maintain horizontal scalability and keep our application web tier as stateless as possible. So even though there is a state of shopping cart, we want to be able to scale our web application as easily as possible. This means
that users should not lose their shopping cart while navigating our website. They will also have their details such as address in a database that we can store effectively and make accessible from anywhere.

+ 1) We have our user, Route 53, Multi AZ ELB, Auto Scaling group three AZ. Our application is accessing our ELB and our ELB says "Alright, you're gonna talk to this instance." And you create a shopping cart, and then the next request 
	 is going to go not to the same instance but to another instance, so now the shopping cart is lost and the user says "Oh, there must just be a little bug, I'm going to try again." So he adds something into the shopping cart
	 and it gets redirected to the third instance which doesn't have their shopping cart. So basically the user is going crazy.
+ 2) We can introduce Stickiness or Session Affinity and that's an ELB feature so we enable ELB Stickiness and now our user talks to our first instance, adds something into the shopping cart and then the second request goes
	 to the same instance because of Stickiness and the third request also goes to the same instance and actually every request will go to the same instance because of Stickiness.This works really well but if an ec2 instance
	 gets terminated for some reason, then we still lose our shopping cart.
+ 3) Now, let's look at the completely different approach and introduce user cookies. So instead of having the ec2 instances store the content of the shopping cart, let's say that the user is the one storing the shopping cart content 
	 and so every time it connects to the load balancer, it basically is going to say "By the way, in my shopping cart I have all these things." and that's done through web cookies. So now if it talks to the first server,
	 the second server or the third server, each server will know what the shopping cart content is because the user is the one sending the shopping cart content directly into our ec2 instances. We achieved statelessness because now 
	 each ec2 instance doesn't need to know what happened before. The user will tell us what happened before but the HTTP request, they are getting heavier. So because we sent the shopping cart content in web cookies
	 we're sending more and more data every time we add something into our shopping cart. Additionally, there is some level of security risk because the cookies, they can be altered by attackers maybe and so maybe our user may have
	 a modified shopping cart all of a sudden. So, when we do have this kind of architecture make sure that your ec2 instances do validate the content of the user cookies. And then, the cookies overall, they can only be so big.
	 They can only be less than four kilobytes total so there's only a little information you can store in the cookies. This is actually a pattern that many web application frameworks use.
+ 4) Let's introduce this concept of server session. So now, instead of sending a whole shopping carts in web cookies, we're just going to send a session ID and in the background, we're gonna have to maybe in the ElastiCache cluster
	 and what will happened is that when we send a session ID, we're gonna talk to ec2 instance and say we're going to add this thing to the cart and so the ec2 instance will add the cart content into the ElastiCache and the ID to 
	 retrieve this cart content is going to be a session ID. So when our user basically does the second request with the session ID and it goes to another ec2 instance, that other ec2 instance is able using that session ID
	 to look up the content of the cart from ElastiCache and retrieve that session data. ElastiCache, is sub-millisecond performance so all these things happen really quickly. An alternative, by the way, for storing session data is 
	 called DynamoDB. This pattern is more secure because now ElastiCache is a sourceof truth and no attackers can change what's in ElastiCache. 
+ 5) We wanna store user data in the database, we wanna store the user address. So again, we're gonna talk to our ec2 instance and this time, it's going to talk to an RDS instance. And RDS is for long term storage and so we can 
	 store and retrieve user data such as address, name etc. directly by talking to RDS. And each of our instances can talk to RDS and we effectively get, again, some kind of Multi AZ stateless solution. 
+ 6) Now we have more and more users and we realized that most of the thing they do is they navigate the website. They do reads, they get product information, all that kind of stuff. So how do we scale reads? We can use an RDS 
	 Master which takes the writes but we can also have RDS Read Replicas with some replication happening. And so anytime we read stuff, we can read from the Read Replica and we can have up to five Read Replicas in RDS.
	 And it will allow us to scale the reads of our RDS database.
+ 7) There's an alternative pattern called Write Through where we use the cache and so the way it works is that our user talks to an ec2 instance. It looks in the cache and said, "Do you have this information?" If it doesn't have it 
	 then it's going to read from RDS and put it back into ElastiCache so just this information is cached. And so the other ec2 instances, they're doing the same thing but this time when they talk to ElastiCache, they will have the information and they get a cache hit and so, they directly get the response right away because it's been cached. And so this pattern allows us to do less traffic on RDS. Basically, decrease the CPU usage on RDS and improve performance at the same time. But we need to do cache maintenance now and it's a bit more difficult and again this has to be done application side.
+ 8) Now, we have our application, it's scalable, it has many many reads but we wanna survive disasters. Our user talks to our Route 53 but now we have a Multi AZ ELB. Route 53 is already highly available. Our auto scaling group is 
	 Multi AZ and then RDS, there's a Multi AZ feature. The other one is going to be a standby replica that can just takeover whenever there's a disaster. And ElastiCache also has a Multi AZ feature if you use Redis.
+ 9) Now for security groups, we wanna be super secure. So maybe we'll open HTTP, HTTPS traffic from anywhere on the ELB side. For the ec2 instance side, we just wanna restrict traffic coming from the load balancer and maybe for my 
	 ElastiCache, we just wanna restrict traffic coming from the ec2 security group and from RDS, we want to restrict traffic coming directly from the ec2 security group.

|Step 1|Step 2|
|------|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_1.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_2.png" width="50%" height="50%"/>|


|Step 3|Step4|
|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_3.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_4.png" width="50%" height="50%"/>|


|Step 5|Step 6|
|------|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_5.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_6.png" width="50%" height="50%"/>|

|Step 7|Step8|
|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_7.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_8.png" width="50%" height="50%"/>|

|Step 9|
|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U2_9.png" width="50%" height="50%"/>|

Architecture:
+ We have used ELB sticky sessions, web client for storing cookies and making our web app stateless or using maybe a session ID and a session cache for using ElastiCache and as an alternative, we can use DynamoDB.
+ We can also use ElastiCache to cache data from RDS in case of reads and we can use Multi AZ to be surviving disasters but we pay more in that case.
+ RDS, we can use it for storing user data, so more durable type of data. 
+ Read replicas can be used for scaling reads  and we pay more for that or we can also use ElastiCache and then we have Multi AZ for disaster recovery.
+ And on top of it, Tight Security for security groups referencing each other.

####  Usecase3 Stateful: Wordpress.com

Here we are trying to create a fully scalable WordPress website. We want to access that website and correctly display picture uploads. WordPress will store the pictures somewhere on some drive and then basically
all your instances must access that data as well. Our user data, the blog content and everything should be stored in a MySQL database and we want this to scale globally.
+ 1) We create a layer that has RDS in the back end, its Multi AZ it's going to be kind of get through all of these 2 instances but what if I just wanna go and really scale up, maybe I want to replace this layer
	 with Aurora MySQL and I can have Multi AZ, read replicas even global databases if I wanted to. Aurora scales better and is easier to upgrade.
+ 2) Now lets talk about storing images. So let's go back to the very simple solution architecture when we have one EC2 instance and it has one EBS volume attached to it. So it's in one AZ and so we're gonna get to
	 our loaded answer and so our user wants to send an image to our loaded answer and that image makes it all the way through to EBS so the image is stored on EBS so now it works really well. We only have one EC2 instance
	 and so it goes straight the the EBS Volume and we're happy. If we wanted to read that image, same thing, the image can be read from the EBS Volume and sent back to the user so very good, right? The problem arrives 
	 when we start scaling so now we have two EC2 instances and two different AZ and each of these EC2 instances have their own EBS Volumes and so what happens is that if I send an image right here from this instance
	 and it gets stored on that EBS Volume if I want to read that image maybe I'll make it this way and yes, I can read it or, very common mistake I can read that image and it will go here and here on the bottom 
	 there is no image present and so because it's not the same EBS Volume and so here I won't be able to access my image. So the problem with EBS Volumes is that it works really well when you have one instance but when you 
	 start scaling across multiple AZ or multiple instances then it's starting to become problematic. 
+ 3) So we can use EFS so let's use the exact same architecture but now we are recording in EFS Network File System Drive so EFS is NFS and so EFS basically creates ENI's for Elastic Network Interface and it creates these ENI's 
	 into each AZ and this ENI can be used for all our EC2 instances to access our EFS drive and the really cool thing here is that the storage is shared between all the instances so if we send an image to the M5 instance 
	 to the ENI, to EFS so the image is stored in EFS now if you wanna read the image, it goes all the way to the bottom and through the ENI and it's going to read on EFS and yes EFS has that image available so we can send it back 
	 and so this is a very common way of scaling website storage across many different EC2 instances to allow them all to have access to the same files regardless of their availability zone and how many instances we have.

|Step 1|Step 2|
|------|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U3_1.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U3_2.png" width="50%" height="50%"/>|


|Step 3|
|------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U3_3.png" width="50%" height="50%"/>|

Architetcure:
+ Aurora Database has less operations and have multi AZ and read replicas
+ We have talked about storing data in EBS which works great when we're in a single instance application but it doesn't work really great when we have many.
+ So maybe we can use EFS then to have a distributed application across multi AZ.
+ Costing aspect - EBS is cheaper than EFS but we do get a lot of advantages by using EFS especially in distributed use cases

#### Quick instantiation of applications

We install and deploy this application onto EC2 instances to basically run websites, and so when you launch your full stack, It can take a lot of time to install applications, insert or recover data, configure everything and 
then launch the application. How to speed things up. 

+ Golden AMI - A golden AMI means that you install your applications , you're OS dependencies etc, everything beforehand, and then you create an AMI from it and then for the future EC2 instances, you just launch them directly 
  from the Golden AMI. The reason we do this is that we don't have to reinstall the applications , OS dependencies etc. We could just launch with everything already installed and ready to go, and that's the fastest way we can 
  start up our EC2 instance. So golden AMI is very common pattern in the Cloud to have. 
+ Bootstrapping - We can also use user data to bootstrap our instances. Bootstrapping means basically configuring the instance when it first starts. And so bootstrapping can also be done to install application or OS dependency's etc.
  But this is going to be very slow and we don't want each application to do the exact same thing the other one did if it can be repeated. But for dynamic configuration for example maybe retrieving the URL for our data base and the 
  password etc. , We can use bootstrapping using the EC2 user data and so we can basically have a hybrid mix of a golden AMI.
+ We can also use Elastic Beanstalk. It uses the same principle of the hybrid, where we can configure an AMI and then we add on some user data.
+ For RDS databases we can restore from the snapshots and then the database will have the schemas and the data ready which is much better than may be running a big insert statement they will take forever to start RDS databases.
  That may be a way to go be quicker when you want to retrieve data.
+ For EBS volumes we can restore from a Snapshot so we don't have to have a disc that's empty and not formatted, we can retrieve from a snapshot and the snapshot will already be formatted properly and have the data we need.

#### Elastic Beanstalk

In traditional architecture, there was Elastic Load Balancing, auto-scaling, Multi-AZ, a Database RDS,  a cache for the ElastiCache, and all these things we had to create manually.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/WEB_APP_ARCHITECTURE.png" width="50%" height="50%"/>

If we have to deploy so many apps, Java, Go, Python, whatever languages you are using, even Docker. And we had every time to create that load balancer, to configure that auto scaling, and that RDS database, and on top of this,
well, we need to deploy into several environments, such as DEV, TEST, and PROD, and maybe we want to have different versions at the same time. It will be a complete nightmare. As  a developer, you don't want to manage infrastructure.
You just want to deploy code. You want to configure the databases, load balancers once and just be done with it. You want it to scale - You want your deployment to be valid for one instance, as well as for 100 instances. As a developer, 
I want my code to run. I really don't care about the architecture. And I possibly want consistency across all my applications and environment. I want a one-stop shop to manage my stuff which is provided by Elastic BeanStalk.

+ Elastic BeanStalk is like a developer centric view, so you can deploy applications on AWS. It will leverage all the components we've seen before like EC2, ASG, ELB, RDS, etc. It's one view that's super easy to make sense of.
  So, we'll see all the configuration and development. But we'll still have full control over how all these components get configured. 
+ Elastic BeanStalk is free, and you are going to pay only for the underlying instances. So, BeanStalk is a managed service. That means that the instance configuration, all of the operating system, will be handled by BeanStalk.
  The deployment strategy, you can configure it, but again it will be performed by BeanStalk. And just the application code is your responsibility. We can always customize stuff. 
+ There's three architecture models for BeanStalk. 
  + You have a single instance deployment, which is good for DEV. So, you'll have a whole environment, and it's going to be one instance.
  + Then you have a load balancer and an auto-scaling group. That's great when you do production or pre-production for your web applications and you want to see how they react at scale.
  + And then, if you just want to have ASG load balancers, it is great when you do non-web apps in production, such as your workers or other kind of models that don't need a load balancer or don't need to be accessible.
+ So, BeanStalk has three components.
  + It has an an application version, so every time you upload new code, you'll get an application version. 
  + It also has an environment name, so you're going to deploy your code to DEV, TEST, and PROD. And you're free to name your 
    environments just the way you want, and have as many environments as you wish. You're going to deploy applications to your environments, and basically will be able to promote these application versions to the next environments.
    So, We're going to create an application version, get it from DEV to TEST to PROD, etc. There's a rollback feature, as well, so we can roll back to a previous application version, which is quite handy. And we've got full control over
    the life cycle of environments. So, the idea is that you create an application, and you create an environment or multiple environments, and then, you're going to upload a version and you're going to give it an alias, so, just a name that you want.
  + And then, this alias, you will release it to environments.
+ So, what can we deploy in BeanStalk? Well, it has support for tons of platforms. Go, Java, Java Tomcat, .NET, Node, PHP, Python, Ruby, Docker - Single Container Docker, Multicontainer Docker, Preconfigured Docker, and if your platform is  
  not supported, you can write your own custom platform.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/EB.png" width="50%" height="50%"/>

#### Elastic Beanstalk handson

Go to Elastic Beanstalk > Get started > Give application name, platform and upload code. > Configuration presets - Low cost(Free tier Eligible - creates 1 ec2 instance with public ip on it), High Availability(It will create a load balancer, Auto scaling group etc.) and Custom Configuration. We select high availability and then we get a whole panel to configure like software(platform), instances(type of intance, IOPS and security group), capacity(no of instances, load balancing, autoscaling, AZ), load balancer(type, listener, process and rules), Updates and deployments, security, monitoring, managed updates, notifications, network, database and tags. > Create app. It takes 5 minutes and completes. We see health okay and an URL where our application is being served. The process has cretaed an instance, an ASG with 3 azs and an ALB, target groups, health checks and necessary security groups. To terminate go to actions > terminate and then delete the app.

#### Event Processing in AWS - different possibilities

+ One way is to use a SQS queue with a Lamda function. The events are going to be inserted into our SQS queue and the Lamdda service is going to poll the SQS queue.
  In case there are issues, it's going to put back the messages into the SQS queue and retry and this can go into some sort of infinite loop. So we can set up a dead letter queue 
  and after say five tries send the message to the dead letter queue to avoid this.
+ Another way is to use SQS FIFO and Lamda Function. In this case, FIFO means first in, first out which means that the messages are going to be processed in order. So our Lamda function is going to 
  try and retry to get the messages from the queue, but because it needs to process them in order, in case one message doesn't go through, it is going to be a blocking never ending process
  and the whole queue processing will be blocked, in which case yet again, we can set up a dead letter queue to send these error messages away from the SQS queue to the DLQ.
+ Another option is to use SNS with Lambda. The message goes through SNS and is sent asynchronously to Lambda. In case the lambda can not process the message, it is going to retry processing internally,
  and that retry will be three times and if the message is not processed successfully it will be discarded or we can set up a DLQ but this time at the Lambda service level, to send that message in to,
  for example, a SQS queue for later processing. So with SQS, the DLQ is set up on the SQS side. And for SNS, the DLQ is set up on the Lambda side.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/EP.png" width="50%" height="50%"/>
+ Fan Out Pattern - How do you deliver a data to multiple SQS queues?
	+ One option is that our application has the AWS SDK installed on it and we have three SQS queues to deliver a message to. So we will write our application to first send a message to the first queue
	  and then send a message to the second queue and then send again the same message to the third queue. That will work but will not be reliable. Because if our application crashes after we send a message 
	  to the second queue, then the third queue will never receive the last message. And the content of each queue will be different. 
	+ Instead we can use a Fan Out Pattern, which is to combine SQS queue with an SNS topic in the middle. Our SQS queues are going to be subscribers of our SNS topic in the middle.
	  And what will happen is that any time we send a message to the SNS topic, it will be sent by the SNS service into all the SQS queues, which has a higher guarantee.
	  So from our application perspective now, we just do a PUT into our SNS topic and automatically the SNS service will fan out that message into the other SQS queues. This is a 
	  very common design pattern on AWS.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/FOP.png" width="50%" height="50%"/> 
+ We can combine this with eventing on S3. We have events happening in our S3 buckets and we want to react to them happening in real time. We can react to S3 object created
  or S3 Object Removed, S3 Object Restore or S3 Replication. And you can even have a filter to just be able to react to certain name in particular. For example, I want to filter
  so that all the events that are created corresponds to a JPEG file or image. And the use case for this would be, for example, to generate thumbnail images of images that have been uploaded to Amazon S3 in real time.
  So let's take an example, we upload an object into our Amazon S3 buckets and we can react in three different ways.
  	+ The first one is to send a notification directly into an SNS topic using S3 events.
  	+ We can send a message into an SQS queue
  	+ We can asynchronously invoke a Lambda function.
  	+ These are the only three possible destinations for your Amazon S3 events. If it's SNS, we can combine from the Fan Out Pattern to just send it to to multiple SQS queues.
  	  So using one S3 events, and one SNS topic, we could send the data out to say 10 SQS queues. If we have an SQS queue, we can also, for example, create a Lambda function or an EC2 instance to read the messages
  	  of that SQS queue, and the benefit we get out of using an SQS queue would be, for example, that if the Lamda function fails, then our message remains in our SQS queue and can be processed by another application.
  	  And finally, if we are using Amazon Lambda, then we can do whatever we want. In case our Lambda fails because it has been invoked asynchronously, then we can define a dead letter queue on a letter function,
  	  such as if it fails to process it three times, then we can send the message to SQS, for example, for later processing.
  	+ You can create as many S3 events as desired on your S3 bucket so you could have as many events as you want. So you can create one event for SNS, one event for SQS, one event for Lambda, whatever you want.
  	+ The S3 event notifications are typically delivered within seconds, but sometimes can take a minute or longer. 
  	+ If two writes are made to a non versioned single object at the same time, then it's possible you only see one S3 event and not two. If you wanted to get an event for each and every single object that has been 
  	  uploaded into S3, then you need to enable versioning and each version will create its own notification. 
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/S3E.png" width="50%" height="50%"/> 

#### Caching Strategies in AWS

+ We have CloudFront in front of an API Gateway which is in front of our application logic(could be EC2, could be Lambda). Our application stores and uses data from the database, and maybe you will use an internal cache
  such as Redis, Memcached and DAX. This is the route for Dynamic Contents.
+ There be will a static content route, in which our client goes through a CloudFront and the CloudFront will source data from S3.
+ Caching Strategies
	+ CloudFront does caching at the edge. That means that they're doing caching as close as possible to our users. If we enable caching in CloudFront, and the users do hit the cache, then they get a response
	  right away, it's very very quick. But because it's at the edge, there is a chance that things have changed in the back end, and is outdated here. So we can use a TTL to make sure the cache is often renewed,
	  and the new stuff is picked up from backend. So we have a balancing act to do between how much we want to cache at the edge, versus how much we want to cache in the app logic.
	+ API gateway also has some caching capabilities, so it doesn't have to be used with CloudFront. API gateway is a regional service and so in case we do use the cache at the API gateway, then the cache will be 
	  regional. There's going to be a network lag between the clients and the API gateway, if the cache is hit here. 
	+ Then we have our app logic which usually doesn't do any caching, but it will do caching in a cache that we could use such as Redis, Memcached, or DAX if we have DynamoDB. And the idea here is that we don't want to 
	  repeatedly hit our database, which does not have any caching capability. We want to make sure that the result of frequent or complex queries is stored into shared cache that can be accessed by your app logic more easily.
	  And so here, we are saving by using caching here. We are saving pressure on our database, and we are augmenting the read capacity. 
	+ There is no caching capability in your database , in Amazon S3 and so on. 
	+ As we move along this line of caching, the more we move to the right, the more there's gonna be a cost for computation and increase in latency.
	+ The scenario will tell where the caching is going to be most appropriate and most efficient.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/CSAWS.png" width="50%" height="50%"/> 

#### Blocking an IP Address in AWS

When a request comes from the clients and goes all the way to your application, many things happen. Sometimes we may want to block an IP address from a client because it is a bad actor, 
and it is trying to access our application. So we want to know the line of defenses available.
+ First let's start from a very simple solution architecture where we have an EC2 Instance, in a security group, in a VPC, and that instance has a public IP so is publicly accessible,
  and this is how our clients get into our EC2 Instance. So say you wanted to block that client, the first line of defense would be the network ACL in our VPC, which is at a VPC level,
  and in this network ACL, we can create a deny rule for this client IP address and the client will just be ejected. Then for the security group of the EC2 Instance, we can not have deny 
  rules. We can only have allow rules, so if we know that only a subset of authorized clients can access our EC2 Instance, then in our security group we can define a subset of IP
  to allow into our EC2 Instance. But if our application is global, we obviously don't know all the IP addresses that will access our application and so the security group here 
  will not be very helpful. Finally, you could run an optional firewall software on your EC2 to block from within your software the request from the client. Now obviously, if the 
  request has already reached your EC2 Instance, then it will have to be processed and there will be a CPU cost for processing that request. 
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/IP1.png" width="50%" height="50%"/>
+ Now lets introduce an Application Load Balancer, so again this ALB is defined within our VPC and we still have our EC2 Instance, but now we have two security groups, we have the ALB security group
  and we have the EC2 security group. And so in this case our Load Balancer in this architecture is going to be in between our clients and our EC2 and it will do something called Connection Termination.
  The clients actually connect to the ALB which will terminate the connection and initiate a new connection from the ALB into our EC2 Instance. In this case, our EC2 security group must be configured 
  to allow the security group of the ALB, because the EC2 Instance can be deployed in a private subnet with a private IP and the source of the traffic it sees comes from the ALB, not the client,
  so from a security group perspective here, we only allow the ALB security group and we're safe on this side. Now for the the security group of the ALB, we need to allow the clients and again
  if we have a range of IP we know that we can configure the security group but if it's a global application, we have to allow everything and their line of defense is going to be at the Network ACL level.
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/IPALB.png" width="50%" height="50%"/> 
+ Now lets look at the NLB. The Network Load Balancer does not do Connection Termination. The traffic actually goes through our Network Load Balancer and so as such there's no such thing
  as a security group for a Network Load Balancer, the traffic is passed through so that means that the client's originating IP is going to go all the way to our EC2 Instance even if our EC2 Instance sits within
  a private subnet and has a private IP. So this can be complicated, but the idea here is that if we again know the source IPs of all the clients , we can define it in the EC2 security group but if we are trying 
  to deny one IP address for our clients, the only line of defense here we have is our Network ACL. So the Network Load Balancer does not have a security group and all the traffic goes through it and 
  so our EC2 Instance sees the client public IP at the edge.
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/IPNLB.png" width="50%" height="50%"/> 
+ Now lets get back to our simpler case which was to have the ALB and here something we can do to deny an IP is to install WAF or Web Application Firewall. 
  This WAF is going to be a little bit more expensive because this is an additional service and a firewall service but in here, we are able to do some complex filtering on IP addresses and we can establish rules
  that will count the requests to prevent a lot of requests going on at the same time from the clients, and so we have more power over our security or our ALB. So WAF is not a service in between your client
  and your ALB, it is a service we have installed on the ALB and we can define a bunch of rules. So this is one more line of defense.
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/WAF.png" width="50%" height="50%"/> 
+ Similarly if we use CloudFront in front of the ALB, CloudFront sits outside our VPC. So as such, our ALB needs to allow all the CloudFront's public IPs coming from the edge locations and there's a list of it online,
  but that's it, so coming from the ALB, it does not see the client IP, what it sees is the CloudFront public IP, and so as such the Network ACL here which sits at the boundary of our VPC is not helpful at all
  because it can not help us block the client IP address. And so in this case if we are trying to block a client from CloudFront we have two possibilities: say we are attacked from a specific country; then we can use
  the cloudfront's geo-restriction feature to restrict all the country from our clients to be denied on CloudFront. Or if there's one specific IP that annoys us we can again use WAF, or Web Application Firewall
  to induce some IP address filtering just like we did before.
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/CFS.png" width="50%" height="50%"/> 

#### High Performance Computing - HPC on AWS

+ The cloud is the perfect place to perform High-Performance Computing. HPC is a combination of service and different options used to maximize the potential of computation within AWS.
	+ This is because you can create a very high number of resources in no time and speed up the time by adding more resources.
	+ You only pay for what you've used. Once you're done you can destroy the entire infrastructure and not be billed any more.
	+ So we can have an extremely high number of instances performing computations for us and then be done with it and just pay for what we used.
	+ We use HPC to perform genomics, computational chemistry, financial risk modeling, weather prediction, machine learning, deep learning, autonomous driving and so on.
	+ Different Services help us do this.
		+ Data Mangement and Transfer
			+ Direct connect to move data GB/s per second of data into the cloud over a private secure network.
			+ Snowballs and snowmobile to move petabytes of data to the cloud through a physical route. They're usually great for big transfers or one off transfers.
			+ DataSync here we have to install the DataSync agents and they will help us move large amounts of data between on-premise NFS or SMB systems into S3, EFS or FSx for Windows.
	    + Compute and networking
	    	+ EC2 instances - We have CPU optimized or GPU optimized instances based on the type of computations we're trying to do. We can also leverage Spot Instances
	    	  or Spot Fleets for huge cost saving and Auto Scaling to automatically scale our fleet based on the computation we're doing.
	    	+ EC2 instances need to talk to one another and perform some computation in a distributed fashion. Then, using an EC2 placement group of type cluster
	    	  is great to get the best network performance. In which case we have a low latency 10 Gbps network. For the cluster placement group everything's
	    	  on the same rack, everything is on the same AZ. 
	    	+ To further improve the performance of our EC2 instances we have 
	    		+ EC2 Enhanced Networking also called SR-IOV. This gives you higher bandwidth, higher PPS or packet per second and lower latency.
	    		+ We get EC2 Enhanced Networking by using
	    			+ Elastic Network Adapter which delivers you a network speed of up to 100 Gbps. ENA is for EC2 enhancement networking, and gives you
	    			  higher bandwidth. higher packet per second and lower latency.
	    			+ Complicated stuff from Intel called 82599VF and that gives you up to 10 Gbps. Its legacy.
	    			+ You can push this further by using the Elastic Fabric Adapter or EFA. This is an improved ENA dedicated for HPC for high performance computing.
	    			  And it only works for Linux. And it's great when you have inter-node communication, or tightly coupled workloads. So think about distributed computation.
	    			  It's going to leverage something called MPI or the Message Passing Interface standard which will bypass the underlying Linux OS to provide even lower-latency 
	    			  and more reliable transport. 
	    + Storage
	    	+ Instance-attached storage, so EBS and this can scale up to 64000 IOPS if you use io1.
	    	+ It could be an instance store, and we've seen this can scale to million of IOPS. And it's linked to the EC2 instance so it's on a hardware
	    	  it's going to be lower latency, but we can lose it if we lose our instance.
	    	+ Then we can use network storage such as Amazon S3 to store a large blob of data. It's not a file system it's to store large objects.
	    	+ EFS where the IOPS is going to be scaled based on the total size of your file system.
	    	+ We can use a provisioned IOPS mode on the EFS to get higher IOPS.
	    	+ We've seen there's a file system that's dedicated to HPC, which was called FSx for Lustre. And Luster was for Linux and Cluster. And it's going to be HPC optimized,
	    	  gives you millions of IOPS and in the backend it's backed by S3.
	    + Automation and orchestration
	    	+ AWS Batch which is a support service to perform multi-node parallel jobs and enables you to run jobs that span multiple EC2 instances. They're batch jobs, and it's very easy to schedule
	    	  these jobs and launch the EC2 instance accordingly. They will be managed by the batch service, so batch is a very popular choice for HPC.
	    	+ We can use something called AWS ParallelCluster which is an open source cluster management tool to deploy HPC on AWS. It's configurable with text files and it automates the creating of VPCs, Subnets,
	    	  clusters types and instance types.

#### EC2 Instance High Availability

An EC2 instance, by default, it's launched in one Availability Zone, and it's not really highly available, but we can engineer something to make it highly available.
+ So we have a Public EC2 instance that's running a web server, and we wanna be able to access the web server . 
	+ So we will attach an Elastic IP to that EC2 instance, so our users can access our website directly through this Elastic IP, and they will be directly talking to the EC2 instance and get a result from our web server. 
	+ But now we want to have a Standby EC2 instance, just in case things go wrong, that makes our EC2 instance highly available. 
	+ We know if something goes wrong by using monitoring. We're going to create a CloudWatch Event or a CloudWatch Alarm, based on an event. We may have a CloudWatch Event,
	  if an instance is getting terminated. or if we are having a web server, and we know the CPU can go all the way to 100%, maybe you want to have a CloudWatch Alarm that monitors the CPU,
	  and if we see the CPU is at 100%, maybe the EC2 instance has gone wrong, and we want to trigger an alarm based on that. So there's different ways of monitoring your EC2 instance,
	  based on what your requirements may be. 
	+ Then, from the Alarm or the CloudWatch Events, you could go ahead and trigger a Lambda function. 
	+ That Lambda function, will allow you to do whatever you want, like issue API calls to start the standby instance if it hasn't been started yet and 
	  issue another API call to attach the Elastic IP to my Standby instance. So now the Elastic IP will be attached, and it will be obviously detached from the other EC2 instance,
	  because an Elastic IP can only be attached to one instance at a time, and the other EC2 instance, can be terminated.
	+ So we have effectively failed over to a new Standby EC2 instance. But our users did not see anything due to the failover using the Elastic IP.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/HAEC2.png" width="50%" height="50%"/> 
+ We can also do failover with an Auto Scaling Group.
	+ So, we have an ASG in two Availability Zones, and again, we're using the same concept, where a user is going to be talking to our application 
	  using an Elastic IP because it makes things a little bit simpler.
	+ So now we configure our Auto Scaling Group by saying that the minimum amount of instances is one, the maximum is one, and we want one desired, and we specify over two Availability Zones.
	  This means that we're going to get only one EC2 instance, and that EC2 instance may be in the first AZ. On the user data of the EC2 instance, when it does come up, its going to acquire and attach
	  the Elastic IP address based on Tags. So this user data will issue API calls and the Elastic IP will be attached to our Public EC2 instance, and our users will be able to talk to our web server.
	+ But now, if this instance is being terminated and it goes down, the ASG will create a Replacement EC2 instance in another AZ, and the second instance will run it's EC2 user data scripts
	  and attach the Elastic IP.
	+ And we have effectively failed over without a CloudWatch Alarm or a CloudWatch Event.
	+ The Auto Scaling Group as soon as it sees that one instance has been terminated, will create a new EC2 instance and another AZ. And the reason we have one min, one max and one desired
	  is that we'll never get more than one instance running at the same time in our entire ASG. 
	+ Finally, because our EC2 instance does do API calls directly, to attach this Elastic IP Address, then we need to make sure that the EC2 instance has an instance role, that allows it to issue API calls
	  to attach this Elastic IP Address.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/HAEIP.png" width="50%" height="50%"/> 
+ We can extend this pattern.
	+ Our EC2 instance, can be stateful and have an EBS volume.
	+ So we have an Auto Scaling Group, two AZ, our Public EC2 instance, and an Elastic IP and an EBS Volume attached to our EC2 instance. Maybe our EC2 instance is a database and we are trying to 
	  make that database highly available.
	+ All of our data is onto our EBS Volume and is locked into a specific Availability Zone.
	+ If our EC2 instance is being terminated, the Auto Scaling Group can use lifecycle hooks, and create a script to take that EBS Volume and create an EBS Snapshot from it.
	+ It will be triggered as soon as the EC2 instance goes down, and so we know that the EBS volume will be free.
	+ So we have an EBS Snapshot, and we tag it properly, and the ASG will be launching a Replacement EC2 instance, and now, by properly configuring our Auto Scaling Group
	  to create a lifecycle hook on the Launch event, we can create an EBS Volume out of this EBS Snapshot into the correct Availability Zone, and then attach it to the Replacement EC2 instance.
	+ Then the user script can also attach the Elastic IP address directly, and we need to make sure that we have an EC2 instance role for the API calls.
	+<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/HAASG.png" width="50%" height="50%"/> 


#### High Availability Bastion Host

+ We have a VPC and we have two AZ.
+ Each AZ will have a public subnet and a private subnet.
+ The bastion host is used to SSH into our EC2 instances that sits in a private subnet. And to do so our clients, which is in the public Internet needs to access a bastion host being deployed
  in the public subnets. They will do SSH into the bastion host on port 22. And then from the bastion host, SSH into the private subnets. We can also have a EC2 instance in the private subnet of 
  another AZ and have the bastion host also do SSH directly into these instances. But we want to make this Bastion host highly available.
+ So it is an EC2 instance, it's self managed and you could be in one AZ, you could be in another AZ and we're going to be able to is make it highly available by creating a new Bastion host in another AZ.
  So once we have two Bastion host, we have to make our clients talk to one Bastion host or another one, without really being concerned about them.
  	+ We can create a network load balancer which will transmit at the TCP level. It's layer four because it's TCP, the TCP traffic will go into the bastion host in each AZ.
  	+ This network load balancer can be deployed in two different public subnets. By connecting to the network load balancer which is a fixed IP directly, we can get rerouted to the bastion host.
  	  So the bastion hosts are going to be registered with the NLB which will be directly connecting to the bastion host and then connecting to the EC2 instances behind them.
    + Because we have a bastion host, we cannot use 
    	+ An application load balancer because an application load balancer works at HTTP level and SSH is lower level , it is TCP. And so we would need to use something like an NLB.
    	+ We have different options here for the bastion host. We can run two across two AZ and then we have a network load balancer with it.
    	+ We can run one across two AZ with one ASG running in one, one nodes. So one min, one max, one desired 
    + To route to the bastion host, we have multiple options.
    	+ If we only have one Bastion host, then we could go back to what we see before and just using the elastic IP and then use a EC2, user datascript to access your Bastion hosts.
    	+ If we have two Bastion hosts or more, we have a a very highly available fleet of Bastion hosts with multiple running in multiple AZ, then it's a great idea to use a network load balancer
    	  which is layer four deployed in multiple AZ and then our clients will just connect directly into the network load balancer. And with it, actually our bastion hosts can live in the private subnets 
    	  directly. 
    	+ So if we had a network load balancer, the bastion host can be pushed into the private subnets and that would even simplify our task and make it even more secure.
    	+ We cannot use an ALB because the ALB is layer seven, which is HTTP protocol. And the SSH protocol requires you to be layer four because it's TCP based only.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/BAHA.png" width="50%" height="50%"/> 


