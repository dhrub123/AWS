## Serverless

+ Developers do not provision servers in these services, everything is managed. 
+ Amazon - S3 (users get static content from our S3 buckets delivered as a website)
+ AWS Cognito - It manages user login, users would have their identity stored here.
+ API gateway - It exposes REST API to be invoked
+ Lambda Functions - They store and retrieve data from Dynamo DB
+ DynamoDB
+ SNS and SQS
+ AWS Kinesis Data Firehose
+ Aurora Serverless
+ Step Functions
+ Fargate

#### Reference Architecture
<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/RA.png" width="60%" height="60%"/>

#### AWS LAMBDA

+ Amazon EC2 are virtual servers in the cloud which needs to be provisioned and are limited by the amount of memory and CPU provisioned. They run continuously and can be optimized by starting them and stopping them efficiently.
  Otherwise, they run continuously, regardless of if something's happening or not on your instance. If you want to scale, you can use Auto Scaling groups, but that means that you need to do something to automatically add and remove servers.
+ In AWS Lambda, we have virtual functions, and there are no servers to manage. We just provision the code and the functions run. This is limited by time, so short executions of up to 15 minutes. They run on-demand.
  This means that when you don't use Lambda, your Lambda function is not running, and you only are going to be billed when your function is running and it will run on-demand when it gets invoked and that is a huge shift from Amazon EC2.
  The scaling is automated. If you need more Lambda functions, occurrences, and concurrency, then automatically, AWS will provision for you, more Lambda functions.
+ Benefits of Lambda
	+ Easy Pricing
		+ You're going to pay for the number of requests Lambda receives, so the number of invocations, and your compute time, so how long Lambda was running for.
		+ There is a very generous free tier on Lambda, which is 1 million Lambda requests, and 400,000 gigabyte-seconds of compute time.
	+ It is also integrated with so many AWS services
		+ API Gateway is to create a REST API, and they will invoke our Lambda functions.
		+ Kinesis will be using Lambda to do some data transformations on the fly.
		+ DynamoDB will be used to create some triggers, so whenever something happens in our database, a Lambda function will be triggered.
		+ Amazon S3, we've seen this already, a Lambda function will be triggered anytime, for example, a file is created in S3.
		+ CloudFront, this will be Lambda@Edge
		+ CloudWatch Events or EventBridge, this is whenever things happen in our infrastructure on AWS and want to be able to react to things, for example, we have a CodePipeline state changes, and we want to do some automations based on it
		  we can use a Lambda function.
		+ CloudWatch Logs to stream these logs wherever you want.
		+ SNS to react to notifications in your SNS topics.
		+ SQS to process messages from your SQS queues.
		+ And finally, Cognito to react to whenever, for example, a user logs into your database.
		+ Usecase 1: Serverless thumbnail creation. We have an S3 bucket, and we want to create thumbnails on the fly, so there will be an event, where a new image will be uploaded in Amazon S3. This will trigger through an S3 event 
		  notification, a Lambda function. And that lambda function will have code to generate a thumbnail. That thumbnail may be pushed and uploaded into another S3 bucket or the same S3 bucket, which would be a smaller version of that 
		  image, and also, our Lambda function may want to insert some data into DynamoDB, around some metadata for the image, for example the image name, size, creation date etc. So we have an automated and had a reactive architecture
		  to the event of new app, new images being created in S3.
		  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/U1.png" width="60%" height="60%"/>
		+ Usecase 2: Serverless CRON job - CRON is a way, on your EC2 instances, for example, to generate jobs every five minutes, or every Monday at 10:00 a.m. etc. But you need to run CRON on a virtual server, so an EC2 instance, and 
		  so on. And so while your instance is not running, or at least your CRONs are not doing anything, then your instance time is wasted. So, you can create a CloudWatch event rule, or an EventBridge rule, that will be triggered 
		  every one hour and it will be integrated with a Lambda function that will perform your task. So this is a way to create a serverless CRON, because CloudWatch Events are serverless, and Lambda functions are serverless too.
		  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/U2.png" width="60%" height="60%"/>
	+ We can use a lot of different programming languages on Lambda 
		+ Node.js(JavaScript)
		+ Python
		+ Java, and this is Java 8 compatible
		+ C# or .NET Core
		+ Golang
		+ C# and Powershell
		+ Ruby
		+ There is a new API called the Custom Runtime API where you can run pretty much any language on Lambda, but it has to be community supported, and one of these example is, for example, the Rust language.
		+ **The one thing that does not run on Lambda is Docker.** So, if in the exam you are asked, hey, I want to run a Docker container on AWS, you have to think ECS or Fargate. It is not for Lambda.
	+ There is very easy monitoring integrations through CloudWatch.
	+ Finally, if you want to provision more resources per function, you can provision up to three gigabytes of RAM per function and if you increase the RAM of your function, then it will also improve the quality and performance
	  of your CPU and network.
+ Pricing  - https://aws.amazon.com/lambda/pricing/
	+ Pay per calls
		+ The first 1 million requests are free ,
		+ Then you're going to pay 20 cents for extra 1 million requests, so that makes a very, very cheap request.
	+ Pay per duration in increments of 100 milliseconds.
		+ First 400,000 gigabyte-seconds of compute time per month for free. Gigabyte-seconds means 400,000 seconds of execution, if the function has one gigabyte of RAM. That means you get eight times as more seconds
		  if the function has eight times as less RAM so 128 megabyte of RAM. 
		+ And after that, you're going to pay $1 for 600,000 gigabyte-seconds.
	+ It is very cheap to run code on lambda
+ Handson 
	+ Go to  Lambda console. 
	  Lambda lets you run code without thinking about servers and that make it a serverless service with zero administration. Lambda responds to events.
	  A lot of different event sources can send events to Lambda and the Lambda function will reply and will react to these events. Lambda function scales based
	  on if it receives more events or less events. Lambda scales seamlessly, so as we can see right now as more invocations happen for our Lambda function our cost remains at zero that's because the first
	  1 million invocations are within the free tier. But as soon as we pass the 1 million invocations, I'm going to start getting billed for it, so the more invocations I do, the faster the number of
	  invocations will go up obviously and the higher the cost. But what we can see that the cost grows linearly with the number of Lambda invocations and that is the whole power of Lambda, We only pay for what you use
	  and it scales based on the load you're having. 
	+ We have three ways to create a function, you could be authoring from scratch, blueprint or browsing the serverless app repository. We will use hello world from blueprint. We will configure this
	  blueprint and so we have to give our function a name. Then there is this execution role where we must actually create an IAM role for our Lambda function. This is what our Lambda function will use to perform
	  its integration with other services. I can create a new role, use an existing role or create a new role from policy templates. I will create a new role with basic Lambda permissions and it will have the permissions 
	  to upload logs to Amazon Cloudwatch Logs.I will create this function and its took about 30 seconds. We can invoke it with a test event which is a JSON. Then I have to run my test event, so I click on test yet again
	  and now we can see that execution result has succeeded. The details will give us lot more information. We can see the start of our invocation with the Requestid, the version of our Lambda function that was invoked 
	  so latest, we can see the end of our Lambda function with the Requestid and we can see the report, the duration was 2.35 millisecond, 100 millisecond has been billed and the size of the memory was 128 mega bytes.
	  We get the duration 2.35 millisecond **but we have been billed for 100 milliseconds because our Lambda function is billed in increments of 100 milliseconds.** We've configured 128 mega bytes of RAM but has used 
	  only 48 mega bytes. We cannot go lower than 128 mega bytes. 
	+ The log actually comes from Cloudwatch Logs from my Log groups, aws/lambda/hello-world which is the name of my function. And then I have Log stream and this Log stream corresponds to an invocation.
	+ If our Lambda function is in a file called lambda_function.py and the name of the function is lambda_handler, the handler is called lambda_function.lambda_handler..
	+ There is an integration between Lambda and Cloudwatch Logs and this is automatic. An execution role was created for our Lambda function which gives lambda has access to Amazon Cloudwatch Logs.
	  It can do three actions on two resources, it can create log group on any resources which allowed our function to create the log group aws/lambda/hello-world and it also has the power to create Log streams
	  and put Log events in this Log group and so that means that our Lambda function was able to create these Log streams and push the Logs.
+ AWS Lambda Limits per region
	+ Execution Limits
		+ So Execution is that the memory allocation is between 128 megabytes to 3000 megabytes(And this is in 64 megabytes increments). When we increase the memory, we have more vCPU.
		+ The maximum execution time is 900 seconds, which is 15 minutes, so anything above that is not a good use case for Lambda.
		+ Environment variables, we can only have up to four kilobytes of environment variables, so a limited amount of space, but there is a temp space if you want to pull in
		  some big files while we create another function, and so disk capacity in the /tmp folder is 512 megabytes.
		+ We can have up to 1000 concurrent executions. This can be increased as well if we request. But it's good to use reserve concurrency early on.
	+ Deployments Limits
		+ The max size for your compressed .zip is 50MB, uncompressed(code + dependency) it is 250 MB.
		+ So for anything above that, for example big files, you should use the /tmp space instead. So, in case you need to have big file in start up please use that directory.
		+ The environment variables size is is four kilobytes.
	+ So if exam asks you that we need six gigabytes of RAM, 30 minutes of execution time, and a big file of three gigabytes, then you will know that Lambda is not the right way to run this work load.
+ AWS Lambda@Edge
	+ Lambda@Edge is another type of synchronous invocation of Lambda.
	+ You have deployed a CDN using CloudFront and you wanted to run a global Lambda function, alongside each edge locations. How would you implement request filtering before reaching your application.
	  For this, you can use Lambda@Edge . It is that you deploy Lambda functions, not in a specific region, but alongside each region around the world with your CloudFront CDN. So, why would you do this?
	  We do this to build more responsive applications, 
	+ We don't manage servers so Lambda will be deployed globally.
	+ You can customize whatever goes through your CDN, and you're only going to pay for what you use.
	+ We can use lambda to change CloudFront request and response. 
	+ There's four types of Lambda functions at the edge.
	+ We have our user, talking to your CloudFront edge locations that are going to be talking to your origin. 
		+ So the first thing we can change is called the viewer request. The viewer request is when CloudFront receives a request from a viewer, so from a user, we can modify that request.
		+ The second type of request we can modify is called the origin request, and this request is before CloudFront forwards the request to the origin.
		+ Then we can modify the third request, which is the origin response, which is done when CloudFront received the response from the origin, we can modify that.
		+ And then finally, we can modify the viewer response, which is before CloudFront forwards the response to the viewer. 
		+ So we can also generate responses to viewers, without ever sending any request to the origin, by just using the viewer request and the viewer response Lambda functions.
	  So, Lambda@Edge does allow you to change these four types of requests. 
	+ Usecase : We can create a global application. Our website will be statically hosted onto Amazon S3 in the HTML form, then the user will visit it and will be asked, with some client side JavaScript to do
	  some API requests to CloudFront and CloudFront will be triggering our Lambda@Edge function which is running globally alongside your edge locations, and then your Lambda function may be querying
	  data in Amazon, DynamoDB,which is a global database for your serverless applications.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/LE.png" width="60%" height="60%"/>
	+ We use Lambda@Edge for 
		+ website security and privacy
		+ Dynamic web application at the edge.
		+ Search Engine Optimization or SEO.
		+ You can intelligently route across origins and data centers.
		+ You can mitigate bots at the edge.
		+ You can do real time image transformation.
		+ AB testing
		+ User authentication and authorization prior to reaching your origin.
		+ User prioritization
		+ User tracking and analytics
	+ It is whenever you need to integrate a Lambda function with your CloudFront distribution to modify these four kinds of requests we've seen.

#### DynamoDB

+ DynamoDB is a serverless database. It is a fully managed, highly available database. And it has replication across 3 AZ by default.
+ It's a NoSQL database which means not only SQL.
+ It's not a relational database like RDS where we can do joins.
+ The idea is that it scales to massive workload and it's a distributed database, so it has its own advantages and its own weaknesses.
+ There's millions of requests per second that can be done through DynamoDB. It can scale to trillions of row and hundreds of terabytes of storage. So it's a database that scales and is managed.
+ It's fast and has consistent performance. That means even if you have a lot of data, you can still get low latency on data retrieval.
+ It's integrated with IAM for security, authorization, and administration. This is one of these cloud-native thing on AWS side.
+ It enables event driven programming with DynamoDB Streams.
+ It's low cost and has auto scaling capabilities.
+ Basics od DynamoDB
	+ DynamoDB is made of tables.
	+ Each table has a primary key and you decide that primary key at creation time.
	+ There's no creating of database. The database is already available. You just start directly by creating tables.
	+ Each table can have an infinite number of items, or rows.
	+ And so each row, or each item, will have attributes. And they can be added over time. And these attributes can be null. So an attribute is like a column in a RDS database.
	+ Now the maximum size of an item is 400 kilobytes. So that means that you can store pretty good sized item on that DynamoDB
	+ The data types supported is going to be 
		+ Scalar types - string, number, binary, Boolean, and null.
		+ Document types such as list and map.
		+ Set types, such as string set, number set, and binary set.
		+ So you have a lot of different types of data you can store on DynamoDB, and you're very flexible.
	+ Provisioned Throughput
		+ The table must have provisioned read and write capacity units.
		+ A Read Capacity Unit, an RCU, is basically defining a throughput for a read. So here's the cost for the RCU ($0.00013 per RCU) And one RCU means that you have one strongly consistent read of four kilobyte per second,
		  or two eventually consistent read of four kilobyte per second. So if you have 10 RCUs, you can do, maybe, 10 strongly consistent reads of four kilobyte per second.
		+ And WCU is throughput for writes. It's about five times as expensive than RCU($0.00065 per WCU). So DynamoDB is better when you read more than when you write more. And one WCU is one write of one kilobyte per second.
		  You have to, basically, figure out how many writes you want per second and what's your write size. And then you can figure out and compute the RCU and the WCU.
		+ Now there's an option to set up auto scaling of throughputs to meet the demand over time. And the throughput can be exceeded temporarily using something called burst credits. And if the burst credit are empty,
		  then you're going to get a ProvisionedThroughputException. That means that you're requesting more than what you have. And so, in this case, you can do an exponential back-off retry to, hopefully, get the read or the write 
		  working eventually.
		+ There's also this thing called on-demand DynamoDB.
+ Handson - Go to Dynamo DB, Give table name and Partition key or primary key say user_name. We can also add a sort key for say a timestamp and so we can then search for user_names between a timestamp. We have to select Read/Write capapcity 
  mode and it can be Provisioned or OnDemand. For provisioned we have to give RCU and WCU and we will get a capacity calculator. We can also enable autoscaling and then rcu wcu will be greyed out. We can add encryption. We can create an item with username and address.
+ Advanced Features of DynamoDB
	+ DAX - DynamoDB Accelerator
		+ This is basically a cache that's seamless, and enabled. You don't need to rewrite your application for this and writes to dynamodb will go through DAX.
		+ The reads will be cached, and you have micro seconds latency. So we solve the Hot Key problem, when you get too many reads on one value in DynamoDB. DynamoDB can go a bit crazy
	  	  and you may get provisionthroughputexceptions. Each cache entry has a five minutes TTL by default. So that means that anytime you read something from DynamoBD, it's gonna be cached for five minutes in DAX
	  	  and that really allows you to relieve pressure off of DynamoDB if you need to. 
		+ You can get up to 10 DAX nodes in the cluster. And it could be multi AZ so you need three nodes minimum recommended for production. It is super secure, you get encryption at rest, you get VPC integration,
	 	  IAM, CloudTrail etc. 
	    + We have DynamoDB and we have maybe lots of tables in our application. And will thus directly talk to DynamoDB accelerator or DAX and DAX will talk to our table and basically, will make the caching wherever necessary.
	      So DAX is a great way to speed up reads in your application and start caching your DynamoDB data.
	    + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/DAX.png" width="60%" height="60%"/>
	+ DynamoDB Streams 
		+ It is whenever you want to intercept changes in DynamoDB, for examples whenever something happens to a table, that could create and update or delete of a row, you want it to end up in DynamoDB Stream.
		+ It's going to create a changelog of everything that happened to your table and that changelog can be read by AWS Lamda. And then we can do some really cool integrations.
		+ We can react to changes in real time, for example we can send a welcome email to new users. We can do some analytics in real time. We can create derivative tables and views. We can even insert this into ElasticSearch
		  if you wanted to use another database.
		+ If you do cross region replication, you need to enable Streams for that.
		+ Streams has twenty four hours of data retention. 
		+ Overall, Streams is mostly used with a Lambda function to be integrated.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/DS.png" width="60%" height="60%"/>
	+ New Features
		+ Transactions. It's an all or nothing type of operation. You can coordinate inserts, updates and deletes across multiple tables. And so, either all these things work or none of them work. 
		  You can include up to ten unique items or up to four megabytes of data total. So you really get some good chance to write, to all tables at a time or none. For example, if you have financial transactions,
		  you want to update everything as part of one transaction.
		+ On demand - Now you don't need to do any capacity planning anymore. It scales automatically. You get any reading, any write, working no matter what.
			+ The catch is that it's 2.5x more expensive than provisioned capacity. So you need to use with care.
			+ You basically use it when you have spikes in workload or it's unpredictable and auto scaling doesn't do a good job. or when your application is so low throughput, then it's actually cheaper to use on-demand
			 because you have so may less reads and writes.
	+ Security 
		+ You get VPC Endpoints, to access DynamoDB without the internet.
		+ You can access all the DynamoDB security using IAM policies.
		+ And you get encryption at rest using KMS. You also get encryption in transit, using SSL and TLS. So, it's all encrypted all the way.
	+ Backup and restore feature
		+ We can do a point in time restore like RDS.
		+ And there's no performance impact. 
		+ And you can enable global tables. That means you have multi region, fully replicated, high performance tables, and that may help with latency.
	+ Amazon DMS - it's just a database migration service, we can use it to migrate to DynamoDB, maybe if you have a MongoDB, Oracle, MySQL, S3, etc.
	+ You can actually launch your own local DynamoDB on your computer. If you were to do development against it.
	+ Global Tables - which is cross region replication.
		+ It is active active replication, that means that all the regions will replicate to all the other ones.
		+ You must enable DynamoDB Streams before you get that, and it's very useful when you want low latency or want to have this for recovery.
		+ We have the table an it's a global table, so it's in us-east-1, and ap-southeast-2. And so if some writes happen to our table in us-east-1, they will be automatically replicated to our table in ap-southeast-2. 
		  And likely, if some writes happen on our table in ap-southeast-two, then they will be replicated back to us-east-1. So with this global tables, we have region replication happening from all the regions.
		  And because all the tables are able to be written, to replicate to each other, it is called an active active type of replication.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/GT.png" width="60%" height="60%"/>
	+ Capacity Planning
		+ You can provision your WCU and RCU in advance.
		+ You can also enable auto scaling. 
		+ You only have planned capacity. But if you have, a more erratic, sporadic type of workload, you may want to have on-demand capacity, where you get unlimited WCU and RCU, so no capacity planning to do at all.
		  You will not get any throttle, but it's going to be a lot more expensive.
		+ So the question is, do you want to have some very rare workloads and you just want capacity for that? maybe on-demand is a great option. Or can you have some more predictable, more smooth workloads that can maybe auto scale? In 
		  which case, planned capacity with WCU or RCU and auto scaling would be a great option.

#### API Gateway

We would like our clients to be able to invoke the lambda functions in some way. There are multiple ways of doing it.
+ We can have the client directly invoke the lambda function, that means that the client would need IAM permissions or we can use an applciation layer to expose our lambda function as an HTTP endpoint.
+ We can use an API Gateway which is a serverless offering from AWS and allows us to create REST APIs that are going to be public and accessible for clients.
	+ The clients will talk to the API Gateway, which will then proxy the request to our lambda functions.
	+ So we would use an API Gateway because it provides us more than just an HTTP endpoint, it will provide us a lot of features, such as of authentication, usage plans, development stages etc.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/GATE.png" width="60%" height="60%"/>
+ We can integrate the API Gateway with the AWS lambda, and that gives us a full serverless application, with no infrastructure to manage.
+ We have support for the WebSocket Protocol, so we can do real-time streaming through the API Gateway in two different ways.
+ The API Gateway can handle API versioning, so we can go from version one to version two, and version three, and not break our clients
+ We can handle multiple environments, that includes a dev, a test, and a prod environment
+ Security - There are many ways to implement security on your API Gateway, for both authentication and authorization.
+ We have the ability to create API keys, do request throttling, in case some clients are doing too many requests on our API Gateway.
+ We can also use some common standards, such as swagger, or Open API 3.0 import to quickly define APIs, and also we can export them as swagger and Open API.
+ We can transform and validate requests and response in the API Gateway level, to ensure that the invocations are correct
+ We can generate SDK and API specifications
+ We can cache API responses.
+ API Gateway integrates with
	+ Lambda function - It can invoke a lambda function, and with this integration, it is the most common way to expose a REST API backed by a lambda function to do a full serverless application.
	+ HTTP
		+ We can expose any HTTP endpoints in the backend, so it could be, for example, an HTTP API you have on premises, or it could be an Application Load Balancer you have on your cloud environment.
		  We would use an API Gateway to leverage the rate limiting features, the caching, the user authentication, the API keys, etc. So it's definitely useful to have a layer of API Gateway on top of your HTTP endpoint.
	+ AWS Service
		+ We can expose any AWS API through the API Gateway
			+ We can start a Step Function workflow, We can post a message to SQS directly from an API Gateway API. We would do this to add authentication, deploy some APIs publicly, or do some rate control
			  on some AWS services.
+ There are three ways to deploy your API Gateway. This is called endpoint types.
	+ The first type, which is the default one, is called edge-optimized. This is for your global clients, so that means that your API Gateway is going to be accessible from anywhere in the world,
	  and to be efficient, the requests are going to be routed through all the CloudFront Edge locations, which will improve the latency. Your API Gateway is still only in one region, where you created it,
	  but it's accessible, efficiently, from every Cloud formation Edge location.
	+ Then, there's the regional deployment, so this is when we don't want to use CloudFront Edge locations, so it's when we expect all of our users to be within the same region where we created our API Gateway,
	  and if you wanted to, you could create your own platform distribution, and this will give you the same result as an edge-optimized distribution, but this time, you have more control over the caching strategies,
	  and the CloudFront settings themselves.
	+ And then, finally, the last kind of API Gateway you can do is a private API Gateway, so this time it's not public, so a private API Gateway can only be accessed from within your VPC, and it will use interface VPC endpoints 
	  for your ENIs. And to define access for an API Gateway, you can use a resource policy.
+ Handson 
	+ Go to API Gateway and choose API Type(HTTP or WebSocket or Rest or Rest API Private). We select RestAPI and click on Build, Create a new API, give name and endpoint type. Then we create method say GET. We have to choose
	  integration type(Lambda Function, HTTP, MOCK, AWS Service, VPC Link). We can select Lambda Function and we enable Use Lambda Proxy Integration. Then we choose region and provide lambda function. We can provide timeout,
	  default is 29 seconds. We also have give API gateway capability to invoke lambda function and this will create a resource policy allowing the gateway to access the lambda function and click save. We can test. The GET function
	  at root is now integrated with the lambda function but we could have given any routes. The headers, body and status code are automatically separated and displayed in appropriate places during response. 
	  in API gateway. We can also log the event in the lambda function. In this way, we can integrate different routes of the api gateway to different lambda function or other services mentioned earlier. Now we will deploy 
	  this API and specify stage. We are given an invoke URL to access this API gateway as an HTTP endpoint.
+ Security
	+ There are three aspects of security, IAM Permissions, Lambda authorizers, and Cognito user pool.
	+ IAM permissions 
		+ If you want to give one of your user, one of your role access to your API , then it makes sense to attach an IAM policy to your user and your role, and then what happens is that the API Gateway
		  will verify the IAM permissions when you call your rest API, and it's really good if you wanna provide API access within your own infrastructure.
		+ This is done through Sig v4, or signature v4, and so the IAM credentials are in the header and the header is passed onto the API Gateway.
		+ So we have our client, it's calling API Gateway with Sig v4, and then API Gateway calls IAM , verifies the policies, makes sure it all checks out, and then if it's happy, goes to the back end.
		+ Advantages - there's no added cost to this solution, and any time you see Sig v4 in the exam, think IAM permissions for API Gateway. It's a pretty easy solution, but if you want to give access to users outside of your AWS
		  then you can't use IAM permissions obviously.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/S1.png" width="60%" height="60%"/>
	+ Lambda authorizer also formerly known as Custom Authorizers
		+ It uses Amazon Lambda to validate the token that is being passed in the header of your request. 
		+ And you can cache the result of your authentication, so you don't need to call your Authorizer Lambda every time a request comes in, just once, and then you can cache your result for say, one hour. 
		+ It's used when you have some kind of third party type authentication, OAuth, SAML, etc. So any time you need to evaluate the credentials given by a third party, Lambda authorizer is a great candidate for this.
		+ And the Lambda, as a result of your authorization must return an IAM Policy for the user, and that IAM Policy will define whether or not the user can call the API.
		+ So now our client calls a rest API with a token, a third party token, and our API Gateway will call the Lambda authorizer, passing the token to the Lambda authorizer, and the Lambda will return an IAM Policy,
		  and if everything checks out, then the API Gateway talks to the back end and we're good.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/S2.png" width="60%" height="60%"/>
	+ User pools for Cognito
		+ Cognito will manage the full user lifecycle
		+ The API Gateway will automatically verify the identity from AWS Cognito, and you don't need to implement any custom Lambda function, or anything.
		+ **Cognito only helps with authentication, not authorization.** So Cognito just says, yes the user talking to you right now is indeed the right user.
		+ So the process is that your client calls the Cognito user pool to authenticate, and then the Cognito user pool gives back a token to the client. The client now calls our API Gateway, as a rest API and it passes on the token
		  it just received from the Cognito user pool. The API Gateway will then make sure that the Cognito token is correct, by talking to Cognito directly. And then when it's happy, it says okay we can now talk to the back end.
		  The back end must ensure that you are authorized to make the call, so this time it's a little bit different. So in this type of solution, where you manage your own user pool , it is going to be great when we use Cognito,
		  because we can see how we can enable Facebook, or Google Authentication with that scheme. 
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/S3.png" width="60%" height="60%"/>
	+ Summary
		+ IAM
			+ IAM is for when you have users or roles already within your AWS accounts.
			+ It handles both authentication, and authorization through IAM Policies. 
			+ It leverages Sig v4.
		+ Custom authorizer
			+ It's great when your third party tokens that you don't control
			+ You are very flexible, in terms of what IAM policy is going to be returned.
			+ Handle authentication and authorization because you return an IAM policy
			+ You're going to pay per Lambda invocation but you can use caching to limit the number of calls you do to your Lambda function for authorizing, which if you have one million users then you'll have to call your Lambda function
			  one million times every time the cache gets invalidated.
		+ User pools
			+ You're going to manage your own user pool. You can be backed by Facebook login, Google login etc.
			+ You don't need to write any custom code.
			+ You must implement the authorization layer on the back end.
			+ Cognito will just provide you an authentication pattern, not an authorization pattern.

#### AWS Cognito

+ Cognito is used when we want to give our users an identity so they can interact with our servers and our application. Cognito is different products.
	+ Cognito User Pool
		+ There's a sign in functionality for app users.
		+ Then it's integrated with the API Gateway.
	+ Cognito Identity Pools or Federated Identity.
		+ This is to provide AWS credentials directly to our app users so they can access our AWS resources directly.
		+ This also has an integration with Cognito User Pools as an identity provider.
	+ Cognito Sync 
		+ This is to synchronize data from a device to Cognito
		+ It's also probably deprecated and replaced by AppSync.
+ AWS Cognito User Pools or CUP
	+ This is basically a serverless database of users for your mobile apps. 
	+ It's a simple login, can be username or email and a password combination.
	+ You can verify emails or phone numbers. You can add multifactor authentication. You can have password policies.
	+ You can also enable Federated Identities, and this is where the confusion comes. This is basically saying you can allow your user to login through Facebook, Google, and SAML and get an identity into your User Pool.
	  Do not mistake this for Federated Identities, which is different. But you can still use the Facebook login or Google login to log into your user pool.
	+ What we get back out of it is a JSON Web Token, or JWT, and this token can be used to verify the identity of someone.
	+ This can be integrated with the API Gateway for the authentication part.
	+ We have our app and it wants to authenticate to CUP. It's going to register or login using a password, and CUP, after verifying the login, says, "Okay here is a JWT or JSON Web Token."
	+ Remember, a database of users where you can have email passwords, username passwords, or you can even can have Facebook or Google login.
	+ It is just you provide an identity and to integrate with the API Gateway.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/CUP.png" width="60%" height="60%"/>
+ Cognito Federated Identity Pools
	+ The goal is to provide direct access to our AWS environment from the client side. So, no proxy, no API. Just straight access.
	+ How ? - We log into a Federated Identity Provider, or we can choose to remain anonymous. We have this option.
	+ And, from this we get temporary AWS credentials back from the Federated Identity Pool.
	+ And, then these credentials come with an IAM policy attached to it, and so we can do stuff based on this IAM policy.
	+ For example, say we wanted to provide temporary access to write to an S3 bucket using a Facebook login. This use case would be great for using Federated Identity Pools.
  	  Our app is able to login to an identity provider. The identity provider can be whatever you want. It an be Google, Facebook, Twitter, SAML, OpenID or even the Cognito User Pools we have created from before.
      So, from there our app gets to login and gets a token. Now using this token, we are going to pass it on to our Federated Identity Pool. So, we authenticate using that token to our FIP, and it will verify the token 
      with our identity provider just to make sure we are who we say we are. Once the token has been verified, the Federated Identity will talk to the STS service to get temporary credentials for AWS. Once it has that,
      it will pass on the temporary credentials back to our application, and now that our application has these temporary AWS credentials, it is, for example, able to interact directly with our S3 bucket. And, we have an IAM policy
      which allows us to do certain things and not do other things. This is how Federated Identity Pools work. We trade in a login for an identity provider, and we get, in the end, temporary AWS credentials. So, an access id, a secret id,
      and a temporary session token. 
    + It directly allows us to interact with AWS for our app users.
    + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/FIP.png" width="60%" height="60%"/>
+ Cognito Sync 
	+ It is deprecated, and we should use AWS AppSync now.
	+ It lets you store user preferences, configuration, and the state of our application
	+ It has a cross device synchronization capability. Any platform, could be iOS, Android, etc.
	+ You can do offline stuff. So, if you were to change your preferences offline and then you go back online, then they're synchronized automatically.
	+ To have it working well, you need to use Federate Identity Pools in Cognito , not User Pools.
	+ The data is stored in data sets, and each data set could be up to one megabyte
	+ We can have up to 20 data sets to synchronize.

#### AWS SAM - Serverless Application Model

+ SAM stands for Serverless Application Model
+ It's a framework for developing and deploying serverless applications 
+ All the configuration of your serverless applications is going to be done in YAML code. Through the same framework, you can configure 
	+ Lambda functions
	+ DynamoDB tables
	+ API Gateway,
	+ Cognito User Pools
  and SAM will help you deploy that automatically.
+ SAM can help you run your Lambda functions, your API Gateway, and your DynamoDB tables locally on your computer so you can do some debugging.
+ Finally, SAM allows you to quickly deploy your Lambda functions using the integration with Code Deploy.
+ SAM at a high level is a framework to manage your applications, your serverless applications, and it allows you to deploy your Lambda functions, and DynamoDB tables, API Gateway, and Cognito User Pools.

#### Serverless Architectures

+ Architecture 1
	+ We're going to create a mobile application called MyTodoList and we have the following requirements.
		+ We want to expose a REST API that has HTTPS endpoints.
		+ We want to be serverless architecture
		+ We want the users to be able to directly interact with their own folder in S3 to manage their data if they want to.
		+ Users should be able to also authenticate through a managed serverless service. 
		+ And finally, users can write and read to-dos, but they mostly read them, so maybe there's something to do around performance here.
		+ The database layer should scale, and should have some really high read throughputs.
	+ So we have a mobile client, and we talked about doing a rest HTTPS thing, so let's use Amazon API Gateway for this. In a classic serverless API fashion, API Gateway will invoke a lambda function
	  which basically allows us to scale and use serverless infrastructure. Amazon Lambda needs to be able to store and read to-do from a database. A database that scales really well that is serverless is DynamoDB.
	  So here, we have DynamoDB as our backend. Now we said there was going to be also some kind of authentication layer going on so, for this we can use a serverless technology such as Amazon Cognito.
	  So our mobile client can connect and authenticate to Cognito and then API Gateway along the way will verify the authentication with Cognito.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A1_1.png" width="60%" height="60%"/>
	+ If you want to give a user's access to Amazon S3 Location, we have our mobile clients, that authenticates to Amazon Cognito, and Cognito that can generate temporary credentials for us using AWS STS and 
	  return these credentials to our mobile client. These credentials allow our mobile client to store and retrieve files in Amazon S3, and basically access their own little space in S3.
	  The wrong answer is to store AWS user credentials on your mobile clients. You want to really use Amazon Cognito, STS, and then Amazon S3 with temporary credentials.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A1_2.png" width="60%" height="60%"/>
	+ Next our app is starting to scale. We're starting to get more users and it turns out that we, by looking at the patterns, figure out that we have a very high read throughput, so we have many RCUs and the to-dos 
	  don't really change much, they don't get edited very often. So how can we change this architecture, to improve the read throughput and decrease maybe the cost overall. What we can do is use DAX as a caching layer,
	  so just before DynamoDB, we'll use DynamoDB DAX and this will basically have a caching layer and because we're doing so many reads, now the reads will be cached in DAX, and so DynamoDB won't need as much read 
	  capacity units, maybe will scale better, because so many reads are cached and this is a great way, overall, to keep on improving our architecture in a serverless fashion.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A1_3_1.png" width="60%" height="60%"/>
	  Now there could be another way of doing caching, maybe you want to use DAX, but maybe also we want to start caching the responses at the Amazon API Gateway level.This is also a very good one if you think
	  that the answers never really change, and that you can start caching a few responses for some few API routes.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A1_3_2.png" width="60%" height="60%"/>
	+ So this is the classic serverless REST API architecture, basically leveraging HTTPS, API Gateway, Lambda and DynamoDB and Cognito to generate temporary credentials with STS which gives us access to an extra bucket
	  with a restricted policy. We can use the exact same app pattern using Cognito, so our app can access maybe DynamoDB or Lambda directly or whatever. Caching the reads on DynamoDB can be done using DAX,
	  its a very easy way to enable and you can bring not only performance improvement, but also cost reduction, and caching the REST request can be done at the API Gateway level, if we have very static responses.
	  Security, finally, for the entire thing can be done with Cognito, and Cognito is directly integrated with the API Gateway.
+ Architecture 2
	+ Now we have a serverless hosted website, called myblog.com.
		+ Our website should scale globally, and we rarely write blogs, we often read blogs.
		+ So our blogs is seen by hundreds of thousands of people online, and we rarely add blogs, maybe once a day, once a week, but most of the time these blogs are being read.
		+ And so most of my website is going to be purely static files and maybe a little bit of my website is going to be a dynamic REST API.
		+ I want to implement caching where possible to save cost and save latency, and have a great user experience.
		+ And any new users that subscribes to my website, to my blog, I really want them to receive a warm welcome Email, and this should be serverless.
		+ And any photo uploaded to the blog, I also want to have a thumbnail being generated, also serverless because I really like serverless.
	+ We want to serve content, it's static and it's global. So we have our client, and our static content may be stored in Amazon S3.
	+ So how do we expose that bucket globally ? The Amazon S3 bucket is in specific region. We can use Amazon CloudFront, and Amazon CloudFront is a global distribution CDN, and so basically our clients is going to
	  interact with edge locations on Amazon CloudFront, and it's going to cache data coming straight from Amazon S3.
	+ In terms of security, we have the client it's interacting with CloudFront, and it's a global distribution still, but now, we're going to use OAI, or an Origin Access Identity from CloudFront to S3.
	  Basically saying, okay, we're going to add a bucket policy, and the bucket policy on S3 will say, you only authorize the OAI, so CloudFront, to read, the rest cannot. And so that secures it, because now our clients
	  they cannot go directly to the S3 bucket to get the content. They have to go through CloudFront, and in this way we've secured our infrastructure.
	+ How do we add a public serverless REST API? Well for this we'll have a REST HTTPS cloud talking to Amazon API Gateway, invoking a Lambda function, maybe querying and reading from DynamoDB, and because we have so many reads,
	  maybe DAX is a great caching layer we could use. If we're going global, maybe we could be leveraging DynamoDB global databases to reduce the latencies in part of the world. That also could be a really good way of maybe 
	  speeding up our infrastructure and our architecture.
	+ Now let's talk about the user welcome email flow. When a user subscribes, I want them to be having an email saying, hello, how are you? So for this, maybe in DynamoDB we want to enable streams of changes, so we'll have 
	  DynamoDB stream being created, and that DynamoDB stream will invoke a Lambda function. That Lambda function is going to be very special, it's going to have an IAM role, which allows us to use Amazon SES.
	  Amazon SES is Amazon Simple Email Service, so SES, and it basically allows us to send emails. So here our Amazon Lambda function can use the AWS SDK to send emails from Amazon SES, and here we go, we have a basically 
	  serverless user welcome email flow, and really simple, no infrastructure to manage, it just works and scales really really well.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A2_1.png" width="60%" height="60%"/>
	+ If users upload images, we want thumbnails to be created, so our client, is going to maybe upload to our S3 bucket directly, or maybe we again have an OAI in a CloudFront distribution. In which case our client will 
	  upload photos to CloudFront, and CloudFront will forward them onto the Amazon S3 bucket, and this is called S3 transfer acceleration. So either directly to S3, or using transfer acceleration, and then 
	  whenever a file is added to S3 it's going to trigger a Lambda function, so Lambda can be triggered by S3, and Lambda will be creating a thumbnail and putting that thumbnail into an S3 bucket, 
	  could be a different bucket for example. And just to show you it's possible, Amazon S3 also has triggers to SQS and SNS. So Amazon S3 can invoke either Lambda, SQS, or SNS.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A2_2.png" width="60%" height="60%"/>
	+ This architecture is serverless, it's all scaling globally. Static content being distributed using CloudFront with S3. We leveraged a global DynamoDB table to serve the data globally. We could have used also 
	  Aurora Global Tables, but in this case it wouldn't have been such serverless, it would have been provisioned Aurora. We could also enable DynamoDB streams, basically these streams should tell us about changes
	  to our user tables and then trigger a Lambda function, and that Lambda function had a IAM role attached to it, so it could use SES or Simple Email Service, and this was just to send emails in a serverless way.
	  And S3 we've seen that it could trigger SQS, SNS, Lambda to be notify of events.
+ Architecture 3
	+ Micro services are not exactly the server-less. We want to switch to a micro service architecture and many services will interact with each other maybe using a REST API.
	+ Each architecture for each micro service may vary in form and shape.
	+ We want to use a micro service architecture for the reason that we want to have leaner development lifecycle for each service.
	+ We want each service to scale independently and have it's own code repository.
	+ Our users may be able to talk to our first micro service over HTTPS and we've decided to have an elastic load balancer talking to ECS, and then talking to Dynamo DB.
	  ECS, is for docker, so for writing docker containers on AWS. This micro service usually has a DNS name or URL, so maybe it's service1.example.com, and so to get all that information maybe you will do a DNS Query
	  to route 53, get an alias record back and then we can interact with that service. Maybe we have a second service, and this one is using a classic architecture for server-less, but instead of having Dynamo DB,
	  we have ElastiCache. We can definitelyuse ElastiCache as the back end for Lambda. But maybe that second micro service, service two, also interacts with service one. So the lambda function will make
	  a call to our elastic load balancer because it needs to get some information from the first micro service to be able to make a response. And then maybe we have a third micro service, also using an ELB, but this one
	  is not server-less, it's using Amazon EC2 auto scaling and an Amazon RDS database, so more of the classic architecture we've seen from before. And it turns out that maybe the EC2 instance must make a call to the second 
	  micro service before making a decision, so it's represented here by the dotted lines. Andthe URL for this is going to be service3.example.com.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A3.png" width="60%" height="60%"/>
	+ We are free to design each micro service the way you want, and this is why I had so many different random architectures.
	+ There is two patterns for micro services. 
	 	+ There is a synchronous pattern, so this is when we make explicit calls to other micro service, so API Gateway, Load Balancer are great way to do HTTPS calls to other micro services.
	 	+ But there is also an asynchronous pattern using SQS, or Kinesis, or SNS, or Lambda triggers, or S3. 
	+ There are some challenges with micro services.
		+ You need to have some overhead for creating each new micro service.
		+ You may get issues optimizing server density or utilization.
		+ You may get complexity of running multiple versions of each micro service simultaneously.
		+ You may get a proliferation of client-side code requirements to integrate with many separate services.
	  But most of these challenges are solved by using server-less patterns. For example, API Gateway, Lambda they scale automatically and then you pay for usage. So no need to worry about server utilization.
	  You can easily clone APIs or reproduce environments in API Gateway. And you can, for example, generate client SDK through Swagger integration for the API Gateway. 
	+ Micro service is a design, and you can use any of this for that, and it does solve some problems, and it does add some problems.
+ Architecture 4
	+ In this architecture, we're going to look into the problems and solution of distributing paid content. We sell videos online and the users have to pay to buy videos. 
		+ Each video can be bought by many different customers.
		+ We only want to distribute videos obviously to the users who are premium users, and have paid for these videos.
		+ We have a database of who the users are, of premium users and, so we want to send links to the premium users and they should be short-lived and secure, so that not other users can use these links.
		+ Our application should be global, and actually we want this to be fully serverless.
	+ First, let's design our premium user service. We have our clients, rest https, talking to the the Amazon API Gateway, talking to Lambda function, talking to DynamoDB table. In this DynamoDB table we can
	  read, update, delete a table of users. And in this table of users we can say whether or not a user is premium. We need to add some form of authentication so, for this maybe we want to authenticate
	  with Amazon Cognito and verify authentication with API Gateway. Now, let's add a video storage service. So, our videos are going to be in S3. We know that the users may want to directly view them from S3 
	  but, what we want is to have global and secure distribution. So, we still have our S3 bucket this is where the videos are going to be located but now we're going to have a CloudFront global distribution
	  using an OAI talking to S3 with a bucket policy and we've ensured that now people can only access our content through CloudFront, and it's distributed globally. 
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A4_1.png" width="60%" height="60%"/>
	  We still haven't figured out the problem of how do we allow our client to talk to CloudFront and only get premium users to get access to this content in CloudFront. We're going to want to use the feature 
	  of signed URL of CloudFront and this is a great way to distribute premium content or paid content to users. But, we have to generate that signed URL so, we have to create or own application that will
	  generate signed URLs. So, for this, we'll create a second Amazon API Gateway and it will verify the authentication of the clients to give them a signed URL. Amazon API Gateway will invoke a Lambda Function
	  that has been coded to create a signed URL and for this, first, it's going to have to verify if the users that invoked this API is a premium user by looking up into Dynamo DB, and then, if it is a premium user,
	  then it's going to use the CloudFronts API using the AWS SDK and it's going to generate a signed URL, and maybe the signed URL will have an expiration of five minutes. When it's done, Lambda returns the value 
	  to API Gateway which returns the signed URL to our clients and now, our client can use the signed URL to access our CloudFront global distribution.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A4_2.png" width="60%" height="60%"/>
	+ In this architecture, We've basically implemented a fully serverless solution
		+ Cognito for authentication
		+ Dynamo DB for storing users that are premium
		+ 2 serverless applications
			+ one was used to register these premium users
			+ and the other one was used to generate CloudFront signed URLs.
		+ And then all this static content, all the videos were stored in S3 and S3 is serverless and scalable.
		+ We've integrated with CloudFront with OAI for security and global distribution so that users can not bypass our security.
		+ And CloudFront has been used to generate signed URLs to prevent unauthorized users from accessing it.
		+ S3 signed URLs are not efficient for global access because CloudFront is more efficient for distributing stuff globally and on top of it because we have an OAI and only allow platforms to access S3,
		  then S3 pre signed URLs would have not worked.
+ Architecture 5
	+ Software updates offloading. We have an application running on EC2 and it distributes software updates once in a while. 
		+ So, think about the computers that need to download the patch which sits on EC2. 
		+ When a new software update is out, we are going to get a lot of request because a lot of people want to update. 
		+ And so the content is distributed in mass over the networking.
		+ It cost us so much money.
		+ Additionally, we really don't want to change or re-architect our application. 
		+ We just wanna optimize our cost and CPU.
	+ Application Current State
		+ We have a classic ELB plus ASG type of application running in multi AZ on M5 instances distributing the software updates. These software updates are put into the Amazon EFS. So how do we basically 
		  enable these applications to scale more globally and to reduce CPU utilization and to reduce cost.
		  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A5_CS.png" width="60%" height="60%"/>
	+ Solution
		+ We just put CloudFront in front. There are no changes to our architecture. They will cache software update files at the edge because the software update files don't change, they're not dynamic, 
		  there're static, they're never changing. And so our EC2 instances, even though they're not serverless, CloudFront is and it will scale for us. So that means that our ASG will not scale as much
		  and we'll save tremendously in EC2 cost, network cost, EFS cost and so much. And on top of it, we'll save on availability and so on. So, remember that CloudFront is such an easy way to make an 
		  existing application more scalable and cheaper if it's mostly static content and just using some caching at the edges all around the world.
		  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A5.png" width="60%" height="60%"/>
+ Architecture 6
	+ Big Data Ingestion Pipeline. 
		+ We want the application ingestion pipeline to be fully serverless, fully managed by AWS
		+ We want to collect data in real-time
		+ We want to transform the data
		+ We want to query the transformed data using SQL
		+ The reports we create using these queries, maybe they should be stored in S3
		+ Then we want to load that data into a data warehouse and create dashboards on it.
		+ So the usual big data problems of ingestion, collection, transformations, querying and analysis.
	+ Let's assume that the producers of are IoT Devices. And so there is a service in Amazon Cloud services and it's called IoT Core which helps you manage these IoT devices. Now these devices can send 
	  data in real-time, to IoT Core and IoT Core directly into a Kinesis Data Stream. So data stream for Kinesis, allows to basically pipe big data, in real-time, very fast into this Kinesis service.
	  Now Kinesis can be talking to Kinesis Data Firehose and Firehose allows us to, for example, every one minute, put and offload data into an Amazon S3 Bucket and that will be an Ingestion Bucket.
	  So we have a whole pipeline to get a lot of data from a lot of devices in real-time, and put it every one minute into an S3 Bucket. On top of it, it's possible for us to cleanse or really quickly 
	  transform the data, using an AWS Lambda function, that is directly linked to Kinesis Data Firehose. So now we have that Ingestion Bucket, we can trigger an SQS Queue can trigger an AWS Lambda function 
	  and it's optional because Lambda can be directly triggered by our S3 Bucket. So, Lambda will trigger an Amazon Athena SQL query, and this Athena query will pull data from the Ingestion Bucket and will 
	  do an SQL query that's all serverless and their outputs of this serverless query, will go into a reporting bucket, maybe again in Amazon S3 as different Bucket. So from this we have the data, it's been reported on,
	  it's been cleansed and analyzed, we can either directly visualize it, using QuickSight. QuickSight is a way for us to visualize the data into an Amazon S3 Bucket, or we can load our data into a proper 
	  data warehouse for analytics, such as Amazon Redshift. Redshift is not serverless, and so this Redshift data warehouse can also serve as an end point for QuickSight.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SERVERLESS/images/A6.png" width="60%" height="60%"/>
	  This shows you, overall, what you can expect in a Big Data Ingestion Pipeline at a high level, including real-time ingestion, transformation, serverless Lambda and some data warehousing using Redshift and 
	  visualization using QiuckSight.
	+ In this architecture
		+ IoT Core allows you to harvest data from many IoT devices.
		+ Kinesis is great real-time data collection, 
		+ Firehose helps you with data delivery to S3 in near real-time, so one minute is the lowest frequency you can choose.
		+ Lambda can help Firehose with data transformation,
		+ Amazon S3 can trigger notifications to SQS, SNS or Lambda.
		+ Lambda can subscribe to SQS, but we could have connected S3 to Lambda, 
		+ Athena is a serverless SQL service, and we can store the results of Athena directly back into S3.
		+ And the reporting buckets contain analyzed data and we can use reporting tools, such as QuickSight, for visualization or Redshift, if we want to do more analytics on it.


