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

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ELB.png" width="60%" height="60%"/>
