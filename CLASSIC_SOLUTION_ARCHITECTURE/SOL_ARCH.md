## CLASSIC SOLUTION ARCHITECTURE

#### UseCase1 : WhatIsTheTime.com

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

|Step 1|Step 2|Step 3|Step4|
|------|------|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_1.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_2.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_3.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_4.png" width="50%" height="50%"/>|

|Step 5|Step 6|Step 7|Step8|
|------|------|------|-----|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_5.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_6.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_7.png" width="50%" height="50%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SOLUTION_ARCHITECTURE/images/U1_8.png" width="50%" height="50%"/>|


An Well-Architected Framework in AWS, has five pillars to it and they are cost, performance, reliability, security, and operational excellence.
+ **cost** - we're scaling up our instance vertically, Maybe we're using ASG to just have the right amount of instances based on the load, and maybe we wanna reserve instances as well to optimize cost.
+ **performance** - we've seen vertical scaling. We've seen ELBs, we've seen auto-scaling groups, basically how we can adapt to performance over time.
+ **reliability** -  we have seen how Route 53 can be used to basically reliably direct the traffic to the right EC2 instances, and maybe using multi-AZ for the ELB and multi-AZ for the ASG as well.
+ **security** - we can use security groups to basically link the load balancer to our instances reliably.
+ **operational excellence** - we can evolve from a very clunky, manual process all the way to having it fully automated with auto-scaling groups.