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
