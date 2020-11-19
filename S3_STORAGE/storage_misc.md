#### Cloudfront

+ CloudFront is a content delivery network or CDN. 
+ It improves read performance, because the content is going to be distributed and cached at the edge locations and edge locations are all around the world.
+ There's about 216 points of presence globally and they are growing in number. Edge Locations are much more than the 30 something regions that AWS has.
+ It gives us DDoS protection against distributed denial of service, integration with AWS shield and also AWS web application firewall.
+ It allow you to expose external HTTPS endpoint by loading the certificates and also talk to internal HTTPS backends.
+ https://aws.amazon.com/cloudfront/features/?nc=sn&loc=
+ So in this is a map of the world, there are some orange regions and their edges. If we have an S3 bucket in Australia and some user from America wants to access it,
  it's actually going to access an edge location close to it. So in America and that network is going to be transmitted over the private AWS network, all the way to the S3 buckets, and the content is going to be cached.
  So the idea is that this American user, with the more users you have in America, the more they will want to do the same kind of reads. And they will all have content served directly from America, not necessarily 
  from Australia, because it will be fetched once into America and then served from the cache locally. So another user, maybe in Asia, will talk to a edge location closer to Asia and that edge location again,
  will support traffic to the S3 buckets to get the content and then cache it at the edge.
+ So CloudFront allows you really to distribute your reads all around the world based on these different edge locations and improve latency and reduce the load on your main S3 buckets.
+ Cloudfront Origins
	+ The first one is an S3 bucket 
		+ You would use CloudFront in front of S3 as a very common pattern to distribute your files globally and cache them at the edge.
		+ You also get enhanced security, between CloudFront and your S3 buckets using CloudFront OAI or origin access identity. This allows your S3 bucket to only allow communication from CloudFront and from nowhere else.
		+ You could also use CloudFront as an ingress, to upload files into S3 from anywhere in the world.
	+ Custom Origin(HTTP endpoint) - This can be anything that respects the HTTP protocol
		+ Application load balancer
		+ EC2 instance
		+ An S3 website(But we first must enable the bucket as a static S3 website)
		+ Any HTTP Backend like  our own on-ßpremise infrastructure.
+ At a high level, we have a bunch of edge locations all around the globe and they're connected to the origin we defined, could be an S3 bucket or it could be any HTTP endpoint. Our clients wants to access our 
  CloudFront distribution. For doing this, the client will send an HTTP request directly into CloudFront. And then the edge location will forward the request to your origin and that includes the query strings
  and the headers, and then your origin response to the edge location. The edge location will cache the response based on the cache settings we've defined and return the response back to our clients.
  The next time another client makes a similar request, the edge location will first look into the cache before forwarding the request to the origin.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CF_HIGH.png" width="60%" height="60%"/>
+ S3 as an origin - You have your origin, S3 buckets. And for example, you have an edge location in Los Angeles and some users want to read some data from there. So your edge location is going to fetch the data
  from your S3 buckets over the private AWS network and give you the results from that edge location. The idea here is that for the edge location of CloudFront to access your S3 buckets using an OAI
  or an origin access identity which is an IAM role for your CloudFront origin. And using that role, edge location is going to access your S3 buckets and the bucket policy is going to say yes, this role is accessible and yes,
  send the file to CloudFront. So this works as well for other edge locations for example, in Sao Paulo in Brazil, or Mumbai, or Melbourne. And so all around the world, your edge locations are going to serve cached content
  from your S3 buckets and so we can see how CloudFront can work as a CDN.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CF_ORIGIN.png" width="60%" height="60%"/>
+ ALB or EC2 as Origin - If we use an ALB or EC2 as an origin, the security changes. EC2 instances must be public because they must be publicly accessible from HTTP standpoint and we have our users all around the world.
  So they will access our edge location which will access our EC2 instances and as you can see, it traverses the security group. So the security group must allow the IPs of CloudFront edge locations into the EC2 instance.
  And for this, there is a list of public IP for edge locations that you can get and the security group must allow all these public IP of edge locations to allow CloudFront to fetch content from your EC2 instances.
  If we use an ALB as an origin, we have a security group for the ALB and the ALB must be public to be accessible by CloudFront. But the backend EC2 instances now can be private. And so in terms of security group 
  for the EC2 instances, EC2 allow the security group of the load balancer, And for the edge location, which are again, public locations, it needs to access your ALB through the public network. And so that means that 
  your security group for your ALB must allow the public IP of the edge locations ,the same public IP as we had from before. 
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/ALB_ORIGIN.png" width="60%" height="60%"/>
+ Cloudfront for Geo Restriction - We can restrict who can access our distribution.
	 + We can provide a white list so users from this list of approved countries can go to a CloudFront and access content.
	 + We can provide blacklist, where the users from these countries are not allowed to access our distribution.
	 + The way the country is determined, is by using a third party Geo-IP database where the incoming IP is matched against it to figure out the country.
	 + Use case for jurisdiction will be when you have copyright laws to prevent access to your content. And you want to prove to regulators that you are indeed restricting content access from, say, France if you have content in America.
+ Difference between CloudFront and S3 cross region replication 
	+ CloudFront 
		+ It is using a global edge network.
		+ Files are going to be cached for a TTL(maybe a day).
		+ Great when you have static content that must be available everywhere around the world
		+ We are okay if that content is outdated for a little bit.
		+ This is for caching globally.
	+ S3 cross region replication
		+ It must be set up for each region in which you want to have replication to happen.
		+ The files will be updated in near real time
		+ It's going to be read only so is going to help you with read performance.
		+ It is great if you have dynamic content that needs to be available at low latency in a few regions.
		+ This is for replication into select regions.

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CLOUDFRONT.png" width="60%" height="60%"/>

#### Cloudfront signed URL

We want to make a cloudfront distribution private
+ WE want to give access to people to premium paid shared content all over the world but you want to be able to see and know who has access to what on your CloudFront distribution.
+ For this you can use a CloudFront signed URL or signed cookie. So first when we create a URL and a cookie, you need to attach a policy and you need to tell
	+ when does the URL or the cookie expire
	+ what IP ranges can access this data 
	+ the trusted signers(Which AWS account can create signed URLs for your users)
+ How long should this URL be valid for ? 
	+ If you're sharing a content like movie or music , you can make it very short a few minutes
	+ But if it's content that is private to the user, that they will access in a long period of time, you can make that URL or signed cookie last for years.
+ Difference between a URL and a cookie
	+ A signed URL gives access to individual files so you get one signed URL per file so, if you have a hundred files , you get a hundred URLs
	+ If you have a signed cookie, then you get access to multiple files and the cookie can be reused so this time you have one signed cookie for many files.
	+ Choose whatever you need based on the context.
+ How does signed URL work - We have our CloudFront distribution and a bunch of edge locations which can access our S3 bucket through OAI, Origin Access Identity for full security. And so that means the objects in our S3 bucket
  cannot be accessed by anything else, but CloudFront, but we still want to be able to give people access to their objects through CloudFronts. So we have our clients and our client is going to authorize and authenticate 
  to our application and we have to code that application. And our application will use the AWS SDK to generate a signed URL directly from CloudFront. It will return the signed URL to the clients and then the client will be able 
  to use that signed URL to get the data and files and objects or whatever he needs directly from CloudFront. This works for signed URL, but this also works for signed cookie obviously.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CFSURL.png" width="60%" height="60%"/>
+ Difference between CloudFront signed URL or an S3 pre-signed URL
	+ CloudFront signed URL 
		+ It allows access to a path no matter the origin, so signed URL works not just for S3 as an origin, but HTTP and other backend as well.
		+ It's an account wide-key-pair, so only the root can manage it.
		+ We can filter by IP, path, date, and expiration.
		+ We can leverage all the caching features out of CloudFront.
		+ If you want people to have access to your CloudFront distribution and it's in front of S3, you have to use a signed URL because you cannot access your S3 bucket as you should because there is a bucket policy restricting it 
		  to the OAI
	+ S3 pre-signed URL 
		+ It issues a request as a person who pre-signed the URL. 
		+ If I sign the URL, with my own IAM principal and then use my IAM key to sign this, then the person who has that URL has the same rights as me.
		+ It has a limited lifetime
		+ The client can access directly your S3 bucket directly using that pre-signed URL.
		+ If your users are using directly against S3 and you want to distribute a file in S3 directly without using CloudFront, then pre-signed URL 
		  would be a great use case for it.
		  
|Cloudfront signed URL|S3 Pre signed URL|
|---------------------|-----------------|
|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/CFSU.png" width="60%" height="60%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/S3_STORAGE/images/S3PSU.png" width="60%" height="60%"/>|









