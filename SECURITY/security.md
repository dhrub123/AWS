## SECURITY and ENCRYPTION

#### Encryption

+ KMS, the Encryption SDK, the Parameter Store, IAM, all these things are essential piece of the exam.
+ Encryption in flight(SSL) 
	+ We want Encryption in flight because if I send a very sensitive secret, for example, my credit card, to a server to make a payment online,
	  I want to make sure that no one else on the way, where my network packet is going to travel, can see my credit card number. And so I wanna make sure that when I make a payment online,
	  I have that green lock, I have that HTTPS website, which guarantees me that it is an SSL-enabled website, and then I will get Encryption in flight.
	+ When you have Encryption in flight the data will be encrypted before I send it, and then the server will be decrypting it after receiving it, but only myself and the server knows how to do these thing.
	+ The SSL certificates are what's going to help with the encryption, and so another way to see it is HTTPS. So any time we've been dealing with an amazon service, and had an HTTPS end point, that guaranteed 
	  us that it was encryption in flight. And now the whole web needs to run on SSL and HTTPS.
	+ When you have this enabled, you are protected against a man in the middle attack(MITM) and so this guarantees that when you have that green lock and the server certificate is valid, no one can retrieve your 
	  sensitive information.
	+ We want to talk an HTTPS website in AWS, could be DynamoDB, could be whatever it wants, and then what we're going to do is that we're gonna have our super secret data, we're going to
	  encrypt it with SSL Encryption, and send this over the network and then the website will receive the data, and know how to decrypt it. All programing languages know how to do SSL encryption and decryption,
	  and all the libraries do this for you, so you don't have to worry about anything; this is not something you have to deal with directly.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/ENCRYPT.png" width="60%" height="60%"/>
+ Server side encryption at rest
	+ Data is encrypted after being received by the server.
	+ Before, the server was receiving data, decrypting it, and using its decrypted form, but here the server's going to store the data on its disk, and so we need to know that the server is storing
	  the data in an encrypted form. Because, in case the server gets hijacked by someone else, we don't want to let someone else to be able to decrypt the data.
	+ The data will be decrypted before being sent back to our client. So, thanks to key, usually called a data key, the data is going to be stored in an encrypted form, and the encryption and decryption keys
	  must be managed somewhere usually called a KMS, or Key Management Service, and the server must have the right to talk to the Key Management Service.
	+ So, here's our object, and we're going to transfer it to EBS, so it's gonna be transferred over whatever mechanism, and EBS will use a data key, and using a data key, it will perform encryption
	  of the data, and now it's stored in an encrypted form, and then the day we need to retrieve the data for whatever reason, then EBS, the AWS Service will do decryption for us, using the data key, again, 
	  and we'll get the encrypted data, and back to us over HTTP or HTTPS for example. So this is how servers side encryption works, and as you can see, the server side itself of the service
	  manages the encryption and decryption, and uses a data key it has access to. Many AWS Services do use that encryption at rest.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/REST.png" width="60%" height="60%"/>
+ Client side encryption
	+ In client side encryption the data will be encrypted by the client, and the client is us. And the server will never be able to decrypt the data.
	+ The data will then be decrypted by a receiving client. So all in all, the data is just stored along the server, but the server doesn't know what the data means.
	+ And the server, as best practice, should never be able to decrypt the data anyway.
	+ And for this, we could leverage something called Envelope Encryption.
	+ We have our objects, and on our clients we're going to use a data key, and we're going to encrypt our data client side. So, we perform encryption, we got data key.
	  Now we send our data to any store of data we want, could be FTP, could be S3, could be whatever you want really. You put your data wherever you want, in amazon or somewhere else.
	  And then when you receive the data back, your client will receive an encrypted object, and if it has access to the data key, if it can manage to retrieve the data key from somewhere, then it will be able to perform
	  a decryption and get the decrypted object as a result. The server does not know how to decrypt or encrypt the data stored, it just receives encrypted data.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/CSE.png" width="60%" height="60%"/>

#### KMS



#### AWS Secrets Manager
+ You can store secrets in AWS Secrets Manager. It's newer service it came after the AWS SSM parameter store was out and really the sole purpose of secrets manager is to be storing secrets.
+ The difference between secrets manager and the parameter store is that secrets manager is more oriented towards secrets and it has a capability to force the rotation of your secrets eEvery X number of days.
+ There is also the capability to automate the generation of secrets on the rotation so it uses AWS Lambda for this integration.
+ On top of it you can integrate secrets manager with RDS to synchronize your secrets between your databases and secrets manager.
+ The secrets are obviously encrypted and you can encrypt it using KMS. 
+ When you go into the exam anytime you see secret storing rotation of secrets and integration with RDS think secrets manager.
+ Handson - Go to Secrets Manager 
	+ With this you can rotate, manage and retrieve secrets throughout their lifecycle.
	+ So the big difference of secrets manager you'll have with something like parameter store with an encrypted value like a secure string is that with secrets manager you can set up some rotation and you can link
	  it to a lambda function that will allow you to rotate your credentials.
	+ It has very tight integration with RDS, Aurora postgres and so on.
	+ So the pricing is that you have 40 cents per secret per month and five cents for 10000 API calls and you get a 30 day free trial available for the secret manager.
	+ It is managed by IAM for access to the secrets.
  Let us store a new secret. The secret type can be Credentials for RDS DB, Credentials for REDSHIFT cluster, Credentials for DocumentDB database, Credentials for other database and other type of secrets.
  If we have a database, we will have to provide username/password but if we have other type of secrets, we will have to provide key value pairs and we can have multiple ones. Then we can encrypt
  and we can use the default encryption key or the KMS key. Then we need to give the secret a name and description. Then we can configure Automatic Rotation and choose a Lambda Function with appropriate role
  to refresh the secret. Sample code is generated to retrieve the secret. And then we click Store. We can also create secret for a RDS database by giving the username and password and then linking the RDS database.
  Thus we can rotate the secret of the database automatically. We can delete the secret when we do not need it and we can define a waiting period as well before the deletion which can be between 7 and 30 days.

#### CloudHSM
+ CloudHSM is another way to perform encryption on your cloud and this is going to be different than KMS.
+ With KMS, AWS is the one that manages the software for the encryption but with CloudHSM, AWS will just provision you the encryption hardware, and you have to use
  your own client to perform the encryption.
+ So HSM(Hardware Security Module) is a dedicated hardware to you. They give you a hardware, the physical thing within Amazon, and you manage all the rest.
+ So you manage your own encryption keys entirely, not AWS, and AWS does not have access to your key, or cannot even recover them.
+ The HSM device is tamper resistant, so no one from the AWS team can touch your device, and it has very high level of compliance(FIPS 140-2 Level 3 Compliance)
+ The HSM clusters can be spread across multiple availability zones, so it's highly available, but you must set this up as well.
+ It supports both symmetric and asymmetric encryption, which can be really helpful if you want to generate some SSL or TLS key.
+ There's no free tier available
+ To use CloudHSM you must use your own CloudHSM Client Software.
+ Services that do integrate with CloudHSM are Redshift, It has support for CloudHSM for database encryption and key management.
+ CloudHSM is also a really good idea if you choose to use SSE-C as an encryption mechanism for S3. And then, in this case, the key management software will be within your HSM.
+ AWS only manages the hardware and AWS cannot recover your keys if you lose your credentials. So really, if you decide to go with HSM, you need to make sure that you manage your keys
  and have a very strong way in place to not lose the keys. And then your CloudHSM clients will have a connection to your HSM device to get right and generate some keys.
+ For IAM, you can create, read, update, and delete an HSM cluster but you cannot manage the keys within. To manage the keys within you need to use the CloudHSM software that is not within the console.
  And that allows you to manage the key and manage the users. But that is not regulated by IAM. This is really up to you to set up your own security within your CloudHSM.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/HSM.png" width="60%" height="60%"/>

#### Shield DDos Protection

+ If you want to protect yourself from DDoS attacks, which are distributed denial-of-service attacks, then you need to use AWS Shield.
+ It comes in two flavors.
	+ AWS Shield Standard which is a free service that is going to be activated by default for every AWS customers, and it's going to protect you from attacks such as SYN/UDP floods, reflection attacks,
	  and other layer three or layer four attacks.
	+ AWS Shield Advanced - which is another level of service, and it gives you a DDoS mitigation service which is $3,000 per month per organization, and it protects you against more sophisticated attacks
	  on your Amazon EC2, ELB, CloudFront, Global Accelerator, and Route 53 services. You also get access 24/7 to a DDoS response team which is called DRP. And finally, in case you have higher fees
	  because you have a Load Balancer and those kind of groups that scales a lot and so on, because you have a DDoS, all the higher fees are being waived because you have opted to choose for AWS Shield Advanced.

#### WAF - Web Application Firewall
+ WAF or Web Application Firewall is a firewall that will protect your web application from common web exploits at the Layer 7.
+ Layer 7 is for HTTP versus Layer 4 which is for TCP. So Layer 7 is a higher level, and has more information into how a request is structured than Layer 4, which is just around the protocol of the transport mechanism.
+ WAF can be deployed on three things
	+ Application Load Balancer - newer generation, HTTP level Load Balancer
	+ API Gateway
	+ CloudFront.
+ To use WAF, you need to define something called a Web ACL, a Web Access Control List and in the Web ACL, there can be rules on multiple things.
	+ And these things include, for example, IP addresses, HTTP headers, HTTP body or URI strings. So for example, we could create a firewall rule which will restrict an IP address from accessing
	  our websites that is deployed on an application Load Balancer.
	+ We can also get protected using WAF from common attacks that can be done at the HTTP level like SQL injection and Cross-Site Scripting or XSS.
	+ Then you can also put some constraints around the size of a query. So maybe you don't want a query to be bigger than five megabytes, in which case, a size constraint ACL will really help
	  decrease the amount of load your website can have due to bad queries. You can also have geo-match to block specific countries from accessing, for example, your API gateway by attaching a web ACL on to your API gateway
	  and blacklisting a country you don't want to be going onto your website.
	+ WAF can also be used to protect yourself against DDoS and so it can give you DDoS protection by including some rate-based rules to count occurences of events. 
	  So in these cases, we don't look at each request individually, we look them in bulk. And so we're going to count the occurrences of events, for example, per IP and we want to say, okay
	  an IP should not do more than five requests per second. Otherwise, it looks like it is a bot, and it's going to attack our infrastructure. And so in this case, using rate-based rules
	  can help for DDoS protection directly within WAF and so we can see WAF can help us with HTTP defense, thanks to the fact that an HTTP request is more structured and we can defend ourselves again for IP addresses,
	  SQL injection, Cross-Site Scripting, geo-match, rate-based rules and DDoS protection.
+ AWS Firewall Manager is to manage all the rules of all your WAFs within an AWS Organization.
	+ You define a common set of security rules 
	+ The WAF rules applies to the exact same thing like ALB, API gateways and CloudFront
	+ It also applies to AWS Shield advanced, which applies to ALB, CLB, Elastic IP and CloudFront
	+ Your security groups for EC2 and ENI resources in VPC.
  So it's a centralized way to manage all these things together within your organization.
+ Reference Architecture for DDoS and other Exploits Protection
	+ https://aws.amazon.com/answers/networking/aws-ddos-attack-mitigation
	+ We have the users that go through route 53 for the DNS service, which is protected by Shield. And then they go for cloudFront Distribution again, which is protected by Shield. We can also install WAF to control the type of request,
	  that get to CloudFront and maybe deny them at the edge. And if they all go through, then they go to your Load Balancer, which is also protected by Shield. And finally, to your auto scaling group that hopefully scales if you're 
	  experiencing an attack.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/DDOS.png" width="60%" height="60%"/>
+ Handson - Go to WAF and Shield. We have the option to create a new WebACL. They cost about 5$ a month. We give it a name and description and a CloudWatchMetric is automatically associated. We will then have to choose the resource type
  it is associated with like Regional Resources(ALB and API Gateway) or CloudFront Distributions. We can choose Cloudfront and add associated Resources to protect which is our CDN. Then we add rules. We can add managed rule groups or our own 
  rule group. If we use managed Rule groups, we have admin protection, amazon ip reputaiton list, core rule set, known bad inputs etc. And then do we want to allow default web ACL action for requests that do not match rules - we click allow.
  Then we set Rule Priority. Then we get Cloudwatch metrics. Then we click Create web acl. We can have IP sets, Regex Pattern Sets, Rule groups and marketpace for WAF. 











