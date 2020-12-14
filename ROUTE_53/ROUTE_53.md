## Route 53 service

It is a managed DNS service. DNS stands for Domain Name System. DNS is a collection of rules and records which will help clients like a web browser 
to reach a server through its domain name. In AWS, there are four common records.

+ A record - It maps a host name, for example myapp.example.com to an IPv4.
+ AAAA record - It maps a host name to an IPv6 address.
+ CNAME - It maps a host name to another host name.
+ Alias - It maps a host name to an AWS resource.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/A_RECORD.png" width="50%" height="50%"/>

We have a web browser and it wants to access our application which is on an application server with IPv4 of 32.45.67.85. So our web browser is going to make
a DNS request to AWS DNS system which is Route 53 in this instance which tells us where myapp.mydomain.com is located. So, we send a host name and Route 53 
will tell us the IP we should be looking at. It is 32.45.67.85 and an A record because we have mapped a host name to a IP. And then, the web browser does
a HTTP request to the server which receives the request and send back HTTP response. So a web browser makes a DNS query to a DNS system such as Route 53,
and then, the web browser is able to reach our server where it is located.

Route 53 can use different things.
+ It can use public domain names that we own or buy.
+ It can use private domain that can only be resolved by our instances within our VPC. like application1.company.internal. This is not something we can 
  purchase on the internet. We will have to make this a private domain, that only our applications can resolve.
  
Route 53 also has a lot of advanced features like load balancing and health checks. It also has routing policies like simple, failover, geolocation, latency, 
weighted and multi value. We have to pay 0.50 dollars per month per hosted zone. It is not free. To buy a domain name, it costs twelve dollars.
**Route 53 is a global service.**

#### DNS TTL

DNS TTL is DNS Records Time to Live. TTL is basically a way for web browsers and clients to cache the response of a DNS query to avoid overloading the DNS.
So make a DNS request to Route 53 from myapp.mydomain.com. and Route 53 will send back the IP: 32.45.67.85, which is a A record, because it's domain to IP.
And then, it's going to also send back the TTL which we can configure for example, we can set it to 300 seconds. The web browser will cache that DNS request 
and the response for the TTL duration. As soon as we receive that reply, it's going to be valid for 300 seconds. Any time we request myapp.mydomain.com, the 
web browser will just look internally, and not ask Route 53 again. So after this TTL happens, if we have something changing on the Route 53 side, for example, the IP is now 195.23.45.22, then our cache will be updated, only after the TTL has expired and then DNS Record will be updated in our web browser. So as soon as we make a change on the Route 53 DNS Record, that doesn't mean necessarily that all the clients will see that change right away. They have to wait for the TTL to expire before they can see that change.
+ High TTL is considered to be something like 24 hours which means that we get way less traffic on your DNS, so Route 53 will have less queries because records 
  are cached for 24 hours. But there's a possible chance of out-dated records, especially if you change them on Route 53.
+ Low TTL, for example, 60 seconds, will incur a lot more traffic on your DNS and more cost, but also, the records will be out-dated for less time. And it will be
  very easy to change the records.
+ **It is mandatory for each DNS record to specify a TTL.**

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/DNS_TTL.png" width="50%" height="50%"/>

#### CName vs Alias

If you have an AWS Resource like a Load Balancer or CloudFront, it will expose a AWS hostname like lb1-1234.us-east-two.elb.amazonaws.com. This URL is controlled
by Amazon Web Services , not us. We want to expose our application as myapp.mydomain.com and point it to the Load Balancer. In this case we need a CNAME.

+ CNAMEs point a hostname to any other hostname for example, app.mydomain.com to blabla.anything.com. But **CNAME only works for non root domain** like 
  something.mydomain.com. , not just mydomain.com.
+ So we have Alias records which are very similar to CNAME but **they point a hostname to an AWS Resource** like app.mydomain.com to blabla.amazonaws.com. In 
  this instance, it has to point to an AWS Resource, specifically, whereas CNAME can point to anything. **They work for both root domain and non root domain.** 
  Alias records are free of charge and have capability for native health checks. 
+ Things to remeber : if we have a root domain then you have to use an Alias , If it's a non root domain, we can use either, and usually it's always going to be 
  an Alias anyway, because we point to an AWS Resource which will be free of charge and better.
+ CNAME is paid and Alias is free
+ In case of Alias, we can evaluate health directly by pointing to healthcheck URL of load balancer.
  
#### Routing Policies

+ Simple routing policy
  + Web browser wants to know where's foo.example.com and route 53 will reply, it is an A record and the IP is 11.22.33.44. We use this when we need to redirect
    to a single resource. 
  + But we cannot attach health checks to a simple routing policy.
  + We can return multiple values to a client, in which case the client sees all the values and will choose a value at random to use.
+ Weighted routing policy
  + Weighted routing policy controls the percentage of the requests that will go to specific endpoints. We have Route 53, and we're going to assign different IP
    addresses, maybe linked to each of two instances, and weights like 70, 20, and 10. The sum does not have to be 100. Whatever weight you put, whatever the sum
    is, the average will be calculated and a percentage will be derived from it. So Route 53 will send 70% of the answers back from one EC2 instance, 20% of the
    answer back from the second one and 10% of the answer back to the third one. So our clients will send 70% of the traffic to the first instance, 20% of the 
    traffic to the second instance and 10% of the traffic to the last instance. SO different weights are assigned to different parts. So for example, to deploy a 
    new application version and you wanted to test only 1% of the traffic on this new app version for example, we can do this with this weighted policy. 
  + We can use this to split traffic between two regions and this is super quick because you can also associate this with health checks, so if one EC2 instance 
    is not working properly, no traffic will be sent to it.
+ Latency routing policy
  + This is the most useful routing policy. Latency, as this name indicates, will redirect the user to the server that has the least latency. That's super 
    helpful when latency for the users is your priority. And latency is going to be evaluated in terms of the user directly to the AWS region. That means that if 
    a user is in Germany, If the us-east-1a, for example, region is the least latency for that user, then that's where it is going to be redirected. We have two 
    EC2 instances, one on the west of the United States and one in Sydney in Australia. And we have users all around the world. Based on the Latency Routing 
    Policy, the four users on the left-hand side of the map will be redirected to the US, while users on the right-hand side of the map will be redirected to 
    Australia.
+ Failover routing policy
  + We have Route 53 in the middle, and we have two EC2 instances. One will be called a primary EC2 instance, and the other one will be a secondary EC2 instance
    meant to be used for disaster recovery only if the primary fails. This is the failover routing policy. We need to use a health check and that's mandatory. So
    Route 53 will have a health check pointing to the primary instance , associated with the primary record and it will check the health all the time. In case 
    that health check fails, automatically, Route 53 will failover to the secondary instance when there is a DNS query. So when our web browser does a DNS 
    request, basically the answer that Route 53 will give it is either the primary if the health check works, but if the health check doesn't pass, then 
    automatically Route 53 is smart enough to send back the secondary disaster recovery response back to the web browser.
+ Geolocation routing policy
  + The geo location Routing Policy is different from Latency based, it is routing based on the user location. So if the traffic originates from the U.K., it 
    should go to this specific IP defined in the a record. We should also create a default policy in case, for example, we get a user from Germany but we haven't 
    specified a routing policy specifically for Germany. Then we still get that default if you go somewhere else.
  + This is to route based on the user location and to redirect traffic from a specific country. So all the traffic that comes from the U.K. should go to 
    11.22.33.44, whereas all the traffic that comes from France should go to 22.33.44.55, and then by the way, there's a default and the default says if you don't 
    have traffic originating from the U.K. or France, then the default response is going to be 33.44.55.66. And that's how geo location works.
+ Multi value routing
  + This is used when you want to route traffic through multiple resources and you want to also associate Route 53 health checks with our records. So it is 
    an improvement over simple routing policy.
  + It will return up to eight healthy records for each multi value query. So you get up to eight values returned even if you have 50 records in the backend.
  + And although it looks like a good replacement for ELB, it's not a substitute, it's different.
  + But it really helps to do some kind of load balancing on the client side. 
  + In this case dig will return upto 8 answers and browser can choose any one of them to send request to.
    
|Simple Routing|Weighted Routing|Latency Routing Policy|
|--------------|----------------|----------------------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/SIMPLE_ROUTING.png" width="50%" height="25%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/WEIGHTED_ROUTING.png" width="50%" height="25%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/LATENCY_ROUTING.png" width="50%" height="25%"/>|

|Failover Routing|Geolocation Routing|
|----------------|-------------------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/FAILOVER_ROUTING.png" width="40%" height="25%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/ROUTE_53/images/GEOLOCATION_ROUTING.png" width="50%" height="25%"/>|

#### Healthchecks
There are health checks in Route 53, and the idea is that if an instance is unhealthy, just like an ELB, Route 53 will not send traffic to that instance.
+ So how do we know if a health check failed? Well, basically, an instance is, or an IP or URL, whatever you want, is deemed unhealthy if it fails three health 
  checks in a row, and it's deemed healthy if it passes three health checks in a row.
+ The default health check interval is 30 second, but there is something called a fast health check, which is 10 seconds, but those will lead to higher cost.
+ When you do a health check, somehow AWS basically launches about 15 health checkers in the background that will check the health endpoint which means if you 
  have a 30 second interval and 15 health checkers that will check the endpoint health, you get on average a request every two seconds. If you have 10 seconds as
  a period of health checks and you have 15 health checkers, then it will be more than one request per second.
+ In terms of health checks, you got lots of options. You get HTTP, TCP, HTTPS health checks. Although, when you use HTTPS health checks, you'd get no SSL 
  certificate verification.
+ You can intregrate these health checks with CloudWatch if you wanted to.
+ Once you have these health checks defined in Route 53, they can be directly linked to the record sets and the DNS queries, and they will basically change the
  behavior of Route 53.

#### Integration of 3rd party domain names with route 53

Route53 is also a registrar. 
+ A registrar is basically an organization that manages the reservation of internet domain names. Some famous registrars are GoDaddy, Google Domains, etc. 
  Route53 on AWS is also a registrar. So We can buy domain names on a registrar and on AWS.
+ Domain registrar is different from a DNS. With Route53, we get both these features. One to offer DNS and the other one to offer domain registration.
+ It is also possible to use a third party domain registrar with AWS Route53. So if you buy your domain on another website, you are still able to use Route53
  to define all the rules.
  + We create a hosted zone in Route53.
  + We update the name server or NS records on the third party website to use the Route53 name servers. We will have option to use separate name servers for our 
    domain or use their name servers(in this case Google Domain name servers) in our Registrar e.g. Google Domain. So if I use the Google Domain name servers, 
    then I  choose my DNS to be the one that comes with the interface in Google Domain. But if I use custom name server, I have to insert the AWS name servers 
    here and then from AWS **I'll create a public hosted zone** and configure my DNS records in AWS. I go back to Route 53, click on Hosted zones, and
    then I would need to create a new hosted zone for my domain name. Then I would click on the radio button beside the domain name and on the right hand side, 
    as we can see, there are name servers and these four name servers, all of these four URLs, is what I would have to put in my Google Domain name server and 
    then I would click on Save, and then automatically now, for my domain, it will be using this specific public hosted zone, in Route53. **So we have to    
    **create a public hosted zone and update the 3rd party registrar ns records.**

#### Handson
Route 53 > Hosted Zones > for new account nothing. So go to Registered Domains > Register Domain(Choose Domain - 12$) and provide
Details  and enable Privacy Protection. It may take an hour to get ready. We can also Transer Domain. Now we go to Hosted Zones > The domain shows up. Click
on the domain and start creating records. We will create a Record Set, give name and set type(For type A-Record, we have to provide IP Adress). So how do we
check that it works ? we do ```nslookup www.abc.com``` in windows and ```dig www.abc.com``` in macos and the address should be shown as the ip adress provided earlier. The TTL is also shown as output of dig command. IF we repeat dig command, we will see TTL decreasing.

We now create few instances in different regions. Then create an internet facing application load balancer(ELB is region scoped) and add the instance in the load balancer region to the load balancer. Now in the A Record, we created earlier, we will give the ip adress of the instance in ireland and a TTL value. Now if we click on www.abc.com, we will be routed to our EC2 instance in Ireland. No, if we change A Record to point to instance in US, that will be effective after the TTl time because the browser will not make a DNS rquest if TTL has not expired. 

Now we delete the A Record and create a CNAME record out of sample.abc.com and copy the load balancer hostname to the value of CNAME and give TTL value. So if we click on www.sample.abc.com, we will be routed to the load balancer in ireland which will point to the EC2 instance behind it. We can also create an Alias and give it a name alias.abc.com. Alias is more efficient here because we are pointing to an AWS resouce, the load balancer and copy the load balancer hsot name to the value. This will result in the same effect. We can also use the Alias as root by creating an alias called abc.com and giving it the target of alias.abc.com.
This is result in this alias pointing to earlier alias which points to the load balancer and the ec2 behind it in ireland. We cannot create a CNAME with abc.com.

If we create an A record and add one IP adress, this is **simple routing**. We can also add multiple IP adresses in separate lines. Then all the ip adresses will be returned and the web browser will choose between the IP adresses. This is an example of client side browser load balancing. dig command will return multiple
entries. 

To achieve **weighted routing**, we have to do the following. We create an A record with www.weighted.abc.com and give value as our ireland ip and select routing policy as weighted and define weight for example 70 and ID say IRELAND. We create another A record with www.weighted.abc.com and give value as our US ip and select routing policy as weighted and define weight for example 20 and ID say US. Now we see there are 2 A records created for www.weighted.abc.com and their weight and id is displayed under respective columns. We can create another A record for our tokyo ip and give it a weight of 10. So when we first hit www.weighted.abc.com, we are routed to any of the instances in US, IR or TOKYO and if we try to reach the URL again, we will not be routed to a new instance until we are past the TTL. But we have 70 percent chance of landing in IR, 20 percent chance of landing in US and 10 percent chance of landing in TOKYO. The dig command is also going to give us back only one IP so no one is aware of this weighted policy.

To achieve **latency routing**, we again have to create multiple A records with routing policy as latency and give ip for the various ec2 instances, their ip and their id. Now the routing happens based on latency.

Route 53 Health checks - Go to Route 53 > Healthchecks > Create healthchecks(Here we have concepts likeDNS failover and Availablity and Performance Monitoring).> We configure healthchecks - Name, what to monitor(Endpoint, status of other healthchecks or a cloudwatch alarm, we select endpoint) - In monitor endpoint, we have to specify IP or DNS name, protocol, ipadress or hostname, port, path, Advanced - healthcheck interval - regular or fast, failure threshold, string matching, latency graph, invert healthcheck, regions, pricing(50 cents per health per month for aws and 75 cents per healthcheck per month for non-aws endpoints, 1$ per month per healthcheck per additional feature like string matching, latency, fast interval and https and 2$ for additional features on non aws endpoint). > Create an alarm when healthcheck fails > Create. First 50 healthchecks are free. Once health checks are created, we can see the healthchecks under them which will ping our instance every 30 seconds.

In **failover routing**, we create multiple a records and select routing policy as failover. Then we have to select failover record type. We can select primary or secondary. For primary failover record type, we have to associate the record with a healthcheck like the one we created earlier. For secondary failover record type, there is no such mandate.

In **geolocation routing**, we create multiple a records and select routing policy as geolocation.  Then we have to set location for incoming traffic like northamerica, france etc and give an id. We can also select default so that when the originating traffic is from a location other than those defined in the routing policy, then this will be triggered.

In **multivalue answer routing**, we create multiple a records and select routing policy as multivalue answer. Then we set ip of instance and associate healthcheck to each a record. Now the browser will get a maximum of 8 records back per DNS query and it can use any of them. **If TTL is updated in any one of these multi value record , the ttl is eefctive for all the records.**
