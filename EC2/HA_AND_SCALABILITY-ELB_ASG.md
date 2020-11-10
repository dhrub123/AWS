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
+ **Healthcheck timeout must be less than interval**

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
+ **Application load balancer(v2 - new generation - 2016)** 
  + They support http, https, websocket(layer 7 only)
  + They allow to route to multiple http applications across machines and these machines are grouped into target groups
  + They allow to load balance multiple application within same instance for example containers
  + Support redirect from http to https
  + Routing tables to different target groups.
    + Routing based on path in URL (example.com/users amd example.com/posts). /users and /posts are different paths.
    + Routing based on hostname in URL(example.com and other.example.com)
    + Routing based on query strings and headers (example.com/users?id=123&order=false)
  + ALB are great for microservices and container based apps(Docker and Amazon ECS). This is because they have port mapping features to redirect to dynamic port
  in ECS.In comparsion we will need multiple classic loadbalancers for this.
  + Application load balancer(V2) target groups
    + EC2 instances , can be managed by an auto scaling group - HTTP
    + ECS tasks - HTTP
    + Lambda Functions - HTTP request is translated into a JSON event.
    + IP Adresses - must be private ips
    + ALB can route to multiple target groups and the healthchecks can be defined at target group level.
  + Fixed hostname - XXX.region.elb.amazonaws.com
  + The application servers do not see the IP of the client directly
    + The true ip of the client is inserted in the header X-Forwarded-For, the port in X-Forwarded-Port and the protocol in X-Forwarded-Proto 
    <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ALB_1.png" width="60%" height="60%"/>
  + We select Application load balancer > Give name, scheme is internet-facing and ip adress type is IPV4. > Listeners - HTTP on port 80 > Availability zones -
  select VPC and appropriate availability zones. > Security groups - Use the earlier ELB security group > Target Group - Give name and choose target type from
  Instance, IP or Lambda Function, Protocol is HTTP and port is 80, Healthcheck Protocol is HTTP and path is / and usual paremters like Healthy Threshold,
  Unhealthy Threshold, TImeout, Interval ans success code is 200 > Register targets - register instances as targets. We can create multiple target groups and add
  them in ALB > Listeners > View/Edit rules(Here we can add many rules like for particular path, route to specific target group or maybe for particular path
  return a fixed message.
    + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ALB_RULES.png" width="60%" height="60%"/>
  + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ALB.png" width="60%" height="60%"/>
  
+ **Network Load balancer(v2 - new generation - 2017)** 
  + These are layer 4 load balancers which means they forward tcp, tls(secure tcp) or udp traffic to instances. So it operates at a lower level.
  + It allows us to handle millions of requests per second. So they are high performance. Latency is very low about 100 ms compared to ALB which is 400 ms.
  + NLB supports one static IP per AZ which helps us whitelist specific IPs and supports assigning Elastic IPs. So it is different from the others because the 
    others had static host names
  + They are not included in EC2 free tier.
  + Select Network Load Balancer > Give name, Choose Internet Facing , Under Load Balancer Protocol , choose from TCP, UDP, TLS and TCP_UDP and for AZ we can
    select multiple AZs. We can select IPV4 adress assigned by AWS or choose an Elastic IP.> Configure Routing - Create Target group - Give name , select target
    type(Instance or IP), protocol(TCP), port and health check settings (Protocol, Port, Healthy Threshold, Unhealthy Threshold, Timeout and Interval) > Register
    Targets - select instances to register as targets > Create Load Balancer. NLB does not have a security group. So in the earlier EC2 instance security group,
    we add a rule which allows TCP traffic on port 80 from anywhere.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/NLB.png" width="60%" height="60%"/>
  
It is recommended to use newer load balancers since they have more features. We can setup internal(private - not accessible from public web) and external(public) ELBs.

#### Elastic Load Balancer - Stickiness

+ Stickiness means that a client will always be redirected to the same instance behind a load balancer. It can be implement in classic and application load
  balancer.
+ The "cookie" used for stickiness has an expiration date that we control. Usecase : make sure that user does not lose session data
+ Enabling stickiness brings some imbalance to the load in backend EC2 instances. This may result in overloading of a particular instance.
+ If we enable stickiness in target group for application load balancer and mention a expiration time, we will be redirected to the same ec2 instance for the
  mentioned duration.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ELB_STICKINESS.png" width="40%" height="40%"/>

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

#### Elastic Load Balancer : Cross Zone load balancing

When cross zone load balancing is enabled, the traffic is distributed evenly across all registered instances in different AZs. This is the best way to spread loads. But if this is not present, each load balancer node will distribute traffic evenly across registered instances in the availability zone.
+ Classic Load Balancer - By default disabled but can be enabled in attributes. There is not charge for Inter AZ data if this is enabled.
+ Application Load Balancer - Always on and cannot be disabled. There is not charge for Inter AZ data.
+ Network Load Balancer - By default disabled but can be enabled in attributes. **There is some charge for Inter AZ data if this is enabled for NLB.**

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/CROSS_ZONE_ELB.png" width="50%" height="50%"/>

#### ELB SSL Certificates

SSL/TLS Certificates
+ An SSL(Secure Sockets Layer) Certificate allows traffic between clients and load balancer to be encrypted in transit. This is called in flight encryption. The 
  data can only be decrypted by sender and receiver. TLS(Transport Layer Security) is a newer version of SSL. Mainly TLS certificates are used nowadays,
+ Public SSL certificates are issued by certificate authorities(CA) like Comodo, Symantech, GoDaddy, GlobalSign, Digicert etc. We can encrypt the connection 
  between client and load balancer using these public certificates.
+ SSL certificates have an expiration date and they must be renewed regularly.
+ The load balancer uses an X509 certificate which is a SSL or a TLS certificate. These certificates can be managed in AWS using **ACM(AWS Certificate** 
  **Manager)**. We can also upload our own certificates.
+ HTTPS listener : 
  + We must specify a default certificate 
  + We can add an optional list of certificates for multiple domains.
  + Client can use **SNI(Server name indication)** to specify the hostname they reach
  + Ability to specify a security policy to support older versions of SSL/TLS(Legacy Clients)

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/SSL_ELB.png" width="40%" height="40%"/>

#### SNI
It solves the problem of loading multiple SSL certificates onto one web server to serve multiple websites.
+ It is a newer protocol and requires the client to mention the hostname of the target server in initial SSL handshake. The server will then find the correct
  certificate or return the default one.
+ It only works for ALB and NLB (Newer generation), Cloudfront, does not work with CLB

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/SNI.png" width="40%" height="40%"/>

#### ELB SSL Certificate main points
+ CLB(V1) 
  + Supports only one SSL certificate
  + Must use multiple CLB for multiple hostname with multiple SSL certificates
+ ALB(V2)
  + Supports multiple listeners with multiple SSL certificates using SNI
+ NLB(V2)
  + Supports multiple listeners with multiple SSL certificates using SNI
+ For CLB Go to Load Balancer > Listeners > Edit > Add HTTPS listener and we need to select a security cipher(protocols we want to support) > setup SSL 
  certificates and we can import them directly or use ACM. **We can have only one certificate for a CLB**.
+ For ALB, we can add listener for HTTPS:443 > Set security policy - Select Default SSL certificate (ACM or Import). So for each rule , we can have a different
  SSL certificate
+ For NLB, we can add listener for TLS:443 > Set security policy - Select Default SSL certificate (ACM or Import). So for each rule , we can have a different
  SSL certificate
  
#### ELB Connection Draining

This feature is called connection draining in case of a CLB or deregistration delay in case of target groups(ALB and NLB). It is the time to complete in flight requests while the instance is deregistering or is unhealthy.
+ As soon as the instance is in draining mode(deregistering), the ELB will stop sending new requests to it.
+ The ELB will wait for existing connections to be completed for the duration of the connection draining period which is 300 seconds by default.
+ The deregistration delay is 300 seconds by default but can be between 1 to 3600 seconds. We can also disable it by setting a value of 0.
  + If requests are short(1 to 5 seconds), set a low value.
  + But if requests take a long time, then the value can be set to a higher value to give in flight requests a chance to complete.
  + We can also disable it and send back a error when a request comes but EC2 is deregistering.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/EC2/images/ELB_CONN_DRAIN.png" width="40%" height="40%"/>
