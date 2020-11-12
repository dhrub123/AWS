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

#### Handson
Route 53 > Hosted Zones > for new account nothing. So go to Registered Domains > Register Domain(Choose Domain - 12$) and provide
Details  and enable Privacy Protection. It may take an hour to get ready. We can also Transer Domain. Now we go to Hosted Zones > The domain shows up. Click
on the domain and start creating records. We will create a Record Set, give name and set type(For type A-Record, we have to provide IP Adress). So how do we
check that it works ? we do ```nslookup www.abc.com``` in windows and ```dig www.abc.com``` in macos and the address should be shown as the ip adress provided earlier. The TTL is also shown as output of dig command. IF we repeat dig command, we will see TTL decreasing.

We now create few instances in different regions. Then create an internet facing application load balancer(ELB is region scoped) and add the instance in the load balancer region to the load balancer. Now in the A Record, we created earlier, we will give the ip adress of the instance in ireland and a TTL value. Now if we click on www.abc.com, we will be routed to our EC2 instance in Ireland. No, if we change A Record to point to instance in US, that will be effective after the TTl time because the browser will not make a DNS rquest if TTL has not expired. 
