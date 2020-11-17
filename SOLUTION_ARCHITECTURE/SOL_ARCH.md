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
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_5.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_6.png" width="50%" height="50%"/>|

|Step 7|Step8|
|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_7.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_8.png" width="50%" height="50%"/>|


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

Go to Elastic Beanstalk > Get started > Give application name, platform and upload code. > Configuration presets - Low cost(Free tier Elegibile - creates 1 ec2 instance with public ip on it), High Availability(It will create a load balancer, Auto scaling group etc.) and Custom Configuration. We select high availability and then we get a whole panel to configure like software(platform), instances(type of intance, IOPS and security group), capacity(no of instances, load balancing, autoscaling, AZ), load balancer(type, listener, process and rules), Updates and deployments, security, monitoring, managed updates, notifications, network, database and tags. > Create app. It takes 5 minutes and completes. We see health okay and an URL where our application is being served. The process has cretaed an instance, an ASG with 3 azs and an ALB, target groups, health checks and necessary security groups. To terminate go to actions > terminate and then delete the app.