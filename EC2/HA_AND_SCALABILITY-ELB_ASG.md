## Scalability and High Availablity

#### Scalability:

An application can handle greater load by adapting. There are 2 kinds of scalability.
+ Vertical Scalability : We need to increase the size of instance eg. T2.micro to T2.large. Use case - Non distributed systems like databases, RDS and Elasticache. But there is a hardware limit on how much we can scale.
+ Horizontal Scalability(Elasticity) : Increase the number of instances for your application. This is suitable for distributed systems - modern web applications.

#### High Availablity:

Running our application in at least 2 datacenters or availability zones. The goal is to survive a data cneter loss. So in case one goes down, the other is still running. High Availability can be passive like RDS HA or active for horizontal scaling. 

#### EC2 Scaling and HA:
+ Vertical scaling : Increase instance size from T2.nano(.5 gig of RAM, 1 vcpu) to U-12TB1.metal(12.3 TB of RAM, 448 vCPU). This is an example of scaling up.
+ Horizontal scaling : Increase number of instances which is scale out(increase) and scale in(decrease) This is done using ASG and load balancers.
+ High availablity : Run instances of same applications across multiple availability zones and use Multi AZ ASG and Load balancer.

#### ELB:

Load Balancers are servers that forward internet traffic to multiple EC2 instances(servers) downstream. Users have a single point of entry and at the backend, we can have many EC2 instances which can scale and serve traffic. So load balancers do the following:
+ Spread load across multiple downstream instances
+ Expose a single point of entry(DNS) to our application SO the users just need to know the hostname of load balancer.
+ It will seamlessly handle failures  of downstream instances through healthchecks. So it is important to perform regular healthchecks of downstream instances.
+ Load balancers provide SSL termination(https) security for websites.
+ Enforce stickiness with cookies
+ High availability across availability zones which means load balancers can spread across multiple azs.
+ Load balancers separate public traffic(Users to load balancer) from private traffic(Load balancer to EC2).

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ELB.png" width="60%" height="60%"/>

###### Why AWS ELB
+ It is a managed load balancer - AWS guarantees that it will be working and AWS is responsible for maintenance, upgrades , HA etc. 
+ AWS provides configuration knobs.
+ Own load balancer will be much cheaper but we will have to manage a lot of configurations since they are complicated and require a lot of effort.
+ AWS ELB can be integrated with different services.

#### Healthchecks:
They are crucial for load balancers because through them load balancers get to know if downstream servers or instances are healthy (can reply and serve requests)
+ The healthcheck is done on a port and a route(usually /health)
+ If the response is not 200(ok), then the instance is unhealthy. 
+ Health checkups are done every 5 seconds by default but they can be configured.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ELB_HEALTHCHECK.png" width="60%" height="60%"/>

#### Types of managed Load Balancers:
There are 3 types of load balancers.
+ **Classic load balancer(v1 - old generation - 2009)** 
  + They support TCP(Layer 4) , HTTP and HTTPS(Layer 7)
  + Healthchecks are TCP or HTTP based.
  + We get a fixed hostname - XXX.region.elb.amazonaws.com
  + We go to Load Balancers > Create a load balancer > we have 3 choices but we will select classic and click create > Give name , select VPC, internal or 
    external, listener configuration - our ELB is going to listen to port 80(load balancer port) on HTTP and backtalk with instances on port 80(instance port). 
    Then assign security group - create new security group , give name and inbound rule is allow port 80 from anyone > configure health check - ping protocol is
    http, ping port is 80 and ping path is /index.html , then response timeout(5 sec), interval(10 sec), unhealthy threshold(2) and healthy threshold(5) - after
    how many healthy pings, instance can be considered healthy> Add EC2 instances > Create. Now under instance in the load balancer, we can see the instance with
    a status of inService(means it is passing health check). Now if we give the DNS name of the ELB in browser, we can see the response from EC2. But if we give 
    ip adress of EC2 in browser, we will also get a response. So the security group of ec2 is too open. So we will modify the inbound http rule of ec2 and in
    source mention the security group of load balancer to limit incoming http traffic only from loadbalancer. We can have our instances in different availability
    zones.
  + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/CLB.png" width="60%" height="60%"/>
+ **Application load balancer(v2 - new generation - 2016)** - http, https, websocket
+ **Network Load balancer(v2 - new generation - 2017)** - tcp, tls(secure tcp) and udp
It is recommended to use newer load balancers since they have more features. We can setup internal(private - not accessible from public web) and external(public) ELBs.

#### Load Balancer security group:
Users will access ELB from anywhere using http or https. So the inbound rule in load balancer security group should be traffic from 0.0.0.0/0(anywhere) on port 80 and 443. But the downstream EC2 instances should only recieve only HTTP traffic from only ELB. So the application security group will reference ELB security group which will in turn enable HTTP traffic on port 80 from ELB security group.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ELB_SG_REFERENCE.png" width="60%" height="60%"/>

###### Important points
+ Load balancer does not scale instantaneously. So if we anticipate large volume , AWS can be contacted for warmup.
+ Troubleshooting
  + 4XX errors are client induced
  + 5XX errors are application induced
  + 503 means at capacity or no registered target
  + If load balancer cannot conenct to application, please check security groups
+ Monitoring
  + Each ELB access request is logged
  + Cloud watch metrics give us aggregate statistices like connection count
