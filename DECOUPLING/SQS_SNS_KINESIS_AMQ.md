## AWS Integration and Messaging :

#### Patterns of communication
+ Synchronous - The different services are directly connected. The problem is here one service can overwhelm one another. If we have a service that calls a encoding service which has a capacity of 10 per minute, and then there is a 
  sudden unpredictable spike of 1000 videos , then the encoding service will be overwhelmed and there will be outages.
+ Asynchronous or Event based - There is middleware like a queue in between the services.Whenever there is upredictable worload, it is better to decouple the services through a middleware and scale that middleware.
	+ SQS: queue model
	+ SNS : pub/sub model
	+ Kinesis : real time streamign model
	+ Using this, the services can scale independently.

#### Amazon SQS (Simple Queuing Service)
+ It autoscales
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/SQS.png" width="60%" height="60%"/>
+ A queue has messages.
+ One or more producers send messages to the queue and one or more consumers poll the queue for messages. If the consumers get a message , they process the message and delete the message from the queue.
  There can be multiple consumers.
+ Amazon SQS - Standard queue
	+ It is a fully managed service for decoupling applications.
	+ A queue has unlimited throughput and we can have unlimited number of messages in queue.
	+ A message is shortlived. It has a default retention of 4 days and a maximum of 14 days which means that once the message is sent to the queue, a consumer needs to consume and process it within that  
	  duration, else it will be lost.
	+ Low latence(<10 ms on publish and receive)
	+ Limitiation of 256 KB per message sent.
	+ Can have duplicate messages(policy of at least once delivery) which means a message can be delivered twice. So when we write application for consuming the messages, we have to take that into account.
	+ It can have messages which are out of order (It follows best effort ordering).
+ Message Production
    + Messages upto 256 KB can be sent to SQS using AWS SDK - send message API
    + Default retention is 4 days upto a maximum of 14 days
    + A message can be an order which has an orderid, a customer id and other order details
    + The message is persisted in SQS until a consumer will read and process the message and delete it.
    + SQS standard has unlimited throughput 
+ Message Consumption
    + Consumers are applications hosted in EC2 instancs, AWS lambda functions or applications hosted in on-premise servers
    + They poll sqs queues for messages and can recieve upto 10 messages at a time.
    + After receiving the messages, consumers can do a variety of things like inserting the records from the messages in a database.
    + After processing the message, consumers delete the message from the queue using DeleteMessageAPI so that no other consumers can read the message and the entire cycle is complete.
    + We can scale out consumption by increasing the number of consumers. Each consumer will receive a set of different messages and process them in parallel. If a consumer cannot process a message fast 
      enough, there is a possibility that another consumer will receive that message. These type of scenarios need to be handled by application.  In this way, we can increase processing throughput by scaling out consumers horizontally. 
    + To scale horizontally, we can also use SQS with ASG. So we can place EC2 instances inside an ASG and use a cloudwatch metric like queue length which is the approximate number of messsages. Whenever it 
      goes up or down  by x amount, a cloudwatch alarm will be triggered whcih will increase or decrease the number of ec2 instances or the consumers.
      <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/SQS_ASG.png" width="60%" height="60%"/>
+ Use case - We have a front end app which processes videos and stores them in s3 buckets. Now this can get overwhelmed during a surge in traffic. So we can decouple this usecase using SQS.
  We can use a frontend layer(small EC2 instances) inside a ASG which just takes the request and creates a message and sends it to SQS. Then we create a consumer layer inside another ASG
  and this time we use more powerful EC2 instances becuase video encoding is heavy. These consumer apps processes the message(encodes the video) and stores it in an S3 bucket.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/SQS_USE_CASE.png" width="60%" height="60%"/>
+ SQS Security
	+ Encryption
		+ In transit encryption using HTTPS api during producing and consuming messages is enabled by default
		+ At rest encryption through KMS
		+ Client side encryption is not supported out of the box but clients can encrypt or decrypt messages
	+ Access Controls : IAM policies to regulate SQS API
	+ SQS Access Policies - useful for cross account access to SQS queues or giving access to other services to write to SQS queues.
+ Handson - We go to create queue and get 2 options , standard and FIFO. We can choose standard. Then we have to give queue name and configuration details like visibility timeout(amount of time message is not
  visible to other queue - 0 second to 12 hous), size(1 to 256 KB), retention period(mac 14 days, default 4 days), delivery delay and and recieve message wait time. Then we have to choose access policy - basic(defines who can recieve or send messages to queue - can be aws account number, iam roles etc.) or advanced(json policy). Then we can set encryption - in flight is enabled by default and at rest can be enabled with KMS. If a message is sent to this queue, the message will have details(ID,size, hash, sender accountid, sent time), body(content) and atributes(can be sent with message). If no consumer is able to consumer and process the message(process + delete from queue) within the visibbility timeout duration, the message will be made available to all consumers again and the recieve count will increase. If the message is deleted from queue by consumer, it is a signal that the message has been processed and the cycle will end. A queue also has different metrics under the monitoring tab.
+ SQS message visibility timeout 
	+ After a message is picked up by a consumer, it has the duration of visibility timeout to process the message.
	+ During that time , the message is invisible to other consumers and after that time has elapsed, the message is again "put back" in queue for reprocessing if the message is not deleted from queue by the 
	  consumer.
	+ The default message visibility timeout is 30 seconds which means the consumer has 30 seconds to process the message.
	+ if the consumer cannot process the message within 30 seconds or the visibility timeout period, it is recommended that it calls the ChangeMessageVisibilityAPI to change the visibility of the message.
	+ If MessageVisibilityTimeout is set to a high value (hours), it will take a long time to reporcess the message if there is a crash on failure in consumer side. If value is too low (seconds), we will have 
	  occurences of duplciate processing. So the value needs to be reasonable.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/SQS_VIS.png" width="60%" height="60%"/>
+ Dead Letter Queue - If a consumer fails to process the message wothin the message visibility timeout period, the message comes back to the queue for reprocessing. But if there is an issue with this message,
  this process will go on and on in a loop.
  	+ So we need to threshold , how many times the message can go back to the queue. After that threshold, the message is sent to a dead letter queue for further analysis.
  	+ The threshold is called MxximumReceives and is defined within DeadLetterQueue settings in a queue.
  	+ A deadletter queue is useful for debugging and messages which expire in a DLQ are permanently lost. So they should be analysed before they expire.
  	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/DLQ.png" width="30%" height="10%"/>
+ Delay Queue - Here the message is not send by consumers , as soon as it is sent by producers to queue. Instead there is a delay upto 15 minutes before the consumers can see the messages.
	+ The delay can be set at the queue level using the Delivery Delay setting. The default is 0 which means the consumers can see the messages right away.
	+ The delay at the queue level can be overridden during send using the DelaySeconds parameter.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/DQ.png" width="60%" height="60%"/>
+ FIFO Queue - Here the messages are processed in the order they are recieved. So if 4 messages 1,2,3 and 4 are receieved in that order, consumers will recieve message 1 first then 2,3 and 4.
	+ So messages are processed in order by consumer.
	+ Capability to send exactly once by removign duplicates
	+ This results in limited throughput of the queue - 300 messages/second or 3000 messages/second non-batched.
	+ The name of a fifo queue must end with .fifo like demoqueue.fifo and there is option for content based deduplication which when enabled will prevent duplicate messages being sent within a 5 minute 
	  window.
	+ The messages have message body, message group id and message deduplication id.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/FIFO.png" width="60%" height="60%"/>
+ SQS with ASG - We will have the EC2 instances within ASG push custom metrics like total number of messages / number of instances. Then we will create 2 cloud watch alarms based on this value going above or 
  below a threshold and scale EC2 instances or consumers according to the scaling policy. 
  |D1|D2|
  |--|--|
  |<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/1.png" width="60%" height="60%"/>|<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/2.png" width="60%" height="60%"/>|

#### Amazon SNS - Simple Notification Service - This is to send messages to many recievers

+ We can have one application sending the message to many applcations one after the another. But that would require us to take care of all the integrations. Instead we can do pub/sub(publish subscribe) where the service will send or publish 
  a message to a topic(an SNS topic) which will have many subscribers and all of them will get the message.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/PS.png" width="60%" height="60%"/>
+ The event producer will send the messages to only one SNS topic.
+ The event recievers(subscribers) will listen to this topic notifications.
+ Each subscriber will get all the messages from the topic by default. There is a new feature to filter messages.
+ We can have upto 10 million subscriptions per topic and a limit of 100,000 topics.
+ Subscribers can be SQS, HTTP ot HTTPS backend with delivery retries in case of failures, Lambda, Email notification, SMS notification, Mobile notification
+ Cloudwatch, ASG notifications, S3 notifications on bucket events, Cloud formation(upon state changes like failed to build) all can send to SNS directly.
+ AWS SNS - how to publish
	+ We will use the SDK Topic Publish to create a topic, then create one or more subscriptions to the topic and then publish to the topic.
	+ We have to use Direct Publish SDK for mobile apps ADK to create a platform application, then create a platform endpoint and then publish to the platform endpoint. Subscribers can be Google GCM, Apple APNS or amazon ADM.
+ Security
	+ Encryption - In flight default and at rest by KMS. 
	+ Access Controls - IAM policies to regulate access to SNS API
	+ SNS access policies - useful for cross account access to SNS topics and allowing other services like S3 to write to SNS topic.
+ Handson - Go to SNS, create a topic(It can be FIFO or a standard topic). Set Encryption, Access Policy, Delivery Retry Policy and create topic. Then create a subscription for that topic. The subscription can be for email, SMS, 
  HTTP, HTTPS, Amazon SQS, Lambda etc. Now we can send messages to the topic and the message will have a Subject, TTL and body.
+ SNS + SQS : Fan out pattern
	+ When we want to send a message to different SQS queues, we use SNS. We send the message to SNS and all SQS queues which are subscribers will recieve the message. So all consumers which are polling those SQS queues will
	  get those messages.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/FAN.png" width="60%" height="60%"/>
	+ Fully decoupled without any data loss.
	+ We would like to feed the messages to SQS for delayed processing, data persistence, retries etc. 
	+ We can add more SQS subscrivers over time.
	+ SQS queue access policy needs to allow SNS topics to write to them. There us a SNS subscription tab in SQS queues.
	+ **SNS cannot send messages to SQS FIFO Queues.**
	+ **S3 Events to multiple Queues**
		+ For the same combination of event type(object create) and prefix(images/) , we can have only one S3 Event rule. So if we want to send the same S3 event to many SQS queues, we have to use fan out. So the s3 event will
		  send messages to SNS topic which is subscribed by multiple SQS queues or even a Lambda function.
		  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/FAN_S3.png" width="60%" height="60%"/>

#### Kinesis - Managed Alternative to Apache kafka(keywords are big data and realtime)

+ It is a big data streaming tool which ais great for colelcting logs, metrics, IOT, clickstreams etc.
+ It is suitable for real time big data and is compatible with stream processing frameworks like Apache Spark and Nifi. These frameworks can perform computations on real time data coming through streams.
+ Data is automatically replicated to 3 AZ
+ There are 3 main sub products of Kinesis 
	+ Kinesis streams - low latency ingesting of streams at scale (All clickstreams, IOT devices, metrics and logs produce data directly to kinesis streams)
	+ Kinesis analytics - perform real time analytics on streams using SQL or data processing
	+ Kinesis Firehose - load streams to other parts of AWS like S3, Redshift, Elastic Search etc. or storing of processed data
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/KINESIS.png" width="60%" height="60%"/>
+ Kinesis Streams
	+ Stream are divided into ordered shards or partitions which are just one level queues. A producer will produce a kinesis stream and may have 3 shards. Data can go into any shard and consumers can consume from any shard as well.
	  So if we want to scale up our stream, we can increase number of shards.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/KS.png" width="60%" height="60%"/>
	+ Data retention in shards is 1 day by default but can go up to 7 days(costs more). Kinesis is just a data pipe where you want to process data asap. Hence the data retention in shards are for short durations.
	+ Ability to reporcess/ replay data even after it has been processed unlike SQS
	+ Multiple applications can consume the ssame stream sort of like SNS
	+ Real time processing with scale of throughput
	+ Once data is inserted in Kinesis , it cannot be deleted(immutable). 
+ Kinesis Streams Shards
	+ One stream is made up of many shards
	+ 1 shard represents 1 MB/s or 1000 messages/s at write per shard. So the producer can write at 1 MB/s or 1000 messages/s. We scale accordingly.
	+ 2 MB/s at read per shard. We scale accordingly.
	+ Shards are provisioned , so billing is per shard provisioned. So we can have as many shards as we want but if we are not using them at full capapcity, we are overpaying. Similarly, if we have less shards, we will have throughput issues.
	+ We have ability to batch messages and calls. This allows us to push data efficiently into kinesis.
	+ The number of shards can evolve over time(reshard - increasing shards/merge - decreasing shards). We can add or remove shards and can enable some auto scaling for shards. The producers and consumers know how to react to resharding and 
	  merging based on consumption patterns.
	  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/SHARDING.png" width="60%" height="60%"/>
	+ Records are ordered per shard.
+ AWS Kinesis API - Put Records
	+ On producer side, we have PutRecord API.We have to send data and partition key that is hashed to get shard id.
	+ The key is a way for us to send data to specific shard. The same key always goes to the same partition. When a message is produced, it knows which shard to go to because of the message or partition key whose hashed value is the shard 
	  id.
	+ Messages that are sent get a sequence number which is increasing always.
	+ So we need to choose a partition key that is highly distributed and helps prevent a hot partition otherwise all our data will end up on the same shard and that shard will be overwhelmed causing a hot partition.
		+ user id is a great key if there are large number of users
		+ if country id is a field and 90 percent are from same country, then that is not a good message key
	+ We can use batching to reduce cost and increase throughput. When we get over the limit, we get ProvisionedThroughputExceeded and we can use retries and exponential backoffs for them.
	+ We can use CLI, AWS SDK or producer libraries from various frameworks to produce messages.
+ AWS Kinesis API - Exceptions
	+ We get ProvisionedThroughputExceeded when we are sending more data than provisioned throughput(exceeds MB/s or TPS for any shard). We have to make sure that we do not have a hot partition.
	+ Solution is to retry of increase shards or ensure that we have a good partition key
+ AWS Kinesis API - consumers
	+ Normal consumers liek CLI or SDK.
	+ Kinesis client library in java, node, pyhton, ruby, .net
		+ It uses DynamoDB to checkpoint the offsets
		+ It uses DynamoDB to track other workers and share work among shards.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/CONSUME.png" width="60%" height="60%"/>
+ AWS Kinesis Security
	+ Control access and authorization to Kinesis using IAM policies
	+ Encryption in flight using HTTPS endpoints
	+ Encryption at rest using KMS
	+ Possibility to encrypt/decrypt data client side
	+ VPC Endpoints avaialble for Kinesis to access within VPC.
+ Handson 
	+ It is not free tier. We can create a data stream / delivery stream / analytics app / video stream etc. We will create a data stream. We will give a stream name. Then we have to give number of shards. 
      We have an account limit of 200 shards. We then click create. We can also have shard level metrics. 
    + We will now produce data for the stream.
      ```
      aws kinesis list-streams // we list streams
      aws kinesis describe-stream --stream-name my-stream // we describe the stream. It gives us shard details, streamARN, name, status, retention period, monitoring, encryption etc.
      aws kinesis put-record --stream-name my-stream --data "user signup" --partition-key user_123 // we get back a shard id and sequence number
      aws kinesis put-record --stream-name my-stream --data "user login" --partition-key user_123 // we get back a shard id and another sequence number

      ```
    + We will now retrieve these records
      ```
      aws kinesis get-shard-iterator --stream-name my-stream --shard-id <enter shard id> --shard-iterator-type TRIM_HORIZON // we get a shard iterator
      aws kinesis get-records --shard-iterator <enter shard iterator frin previous command> // We get back all the records and the data is base 64 encoded.

      ```
+ Kinesis Data Firehose
	+ It is fully managed serverless service, no administration and it autoscales
	+ It loads data into redshift, s3, elastic search, splunk etc. 
	+ Near real time
		+ 60 seconds latency minimum for non full batches or minimum 32 MB of data at a time
	+ Supports data formats like CSV, JSON etc . conversions, transformation, compressions etc.
	+ We only pay for the amount of data going through Firehose. Both Kinesis Agent and Kinesis data streams can send data to Kinesis Firehose.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/FIREHOSE.png" width="60%" height="60%"/>
	+ Streams are when you write custom code for producers and consumers. It is real time(about 200ms) and we must manage scaling(shard splitting /merging) and do capacity planning ahead, data storage upto 7 days, 
	  replay capability and multi consumers vs Firehose which is fully managed, sends data to s3, splunk, redshift and Elastic search, does serverless data transformations with lambda, near real time(lowest buffer time is 1 minute),
	  auto scaling and no storage, we cannot replay data from firehose.
+ Kinesis Data Analytics
	+ It can take data from kinesis data streams or kinesis data firehose and perform complex real time analytics. 
	+ It has autoscaling, it is managed, no servers to provision and continuous or real time.
	+ We pay for actual consumption rate
	+ We can create new streams out of the real time queries to be consumed by consumers or firehose etc.



#### Ordering of Data into Kinesis vs SQS FIFO Queue
+ Kinesis - We have 100 trucks on road, each sending their GPS positions to AWS regularly. How can we use kinesis to consume data fo each truck and accurately track their movememnt ? 
  We use a partion_key value of truck_id and if we have 5 shards, each shard will have data for 20 trucks. And the data with same key for same truck will always go to particular shard.
  	+ Trucks will have data ordered within each shard 
  	+ We can have a maximum of 5 consumers in parallel.
  	+ Can receive 5 MB/s of data
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/KO.png" width="60%" height="60%"/>
+ FIFO Queue - If we have no group ID, messages are consumed in the order they are send with one consumer. But if we want to scale the number of consumers, we can group the messages unto message groups 
  when they are related to each other. https://aws.amazon.com/blogs/compute/solving-complex-ordering-challenges-with-amazon-sqs-fifo-queues/
  	+ we can have one FIFO queue
  	+ We have 100 message groups, one for each truck
  	+ We can have upto 100 consumers due to 100 group ids
  	+ We have upto 300 messages per second or 3000 if we use batching. 

#### SQS vs SNS vs KINESIS

<img src="https://raw.githubusercontent.com/dhrub123/AWS/master/DECOUPLING/images/DIFF.png"/>

#### Amazon MQ - This is for migrating exisiting MQ apps in premise without rewriting code

+ SQS, SNS are cloud native, the use proprietary AWS protocols and we do not manage them, they are serverless.
+ Traditional on premise applications use open protocols lile MQTT, AMQP, STOMP, OpenWire, WSS. To migrate them , we use AMAZOM MQ which is a managed Apache ActiveMQ
+ It does not scale as much as SQS or SNS because we have to provision it.
+ It runs on a dedicated machine and can run in HA with failover doing multi AZ.
+ It has both queue and topic features.
+ Go to Amazon MQ - We can use single instance broker, Active/standby broker(Multi-AZ). We can use blueprints(clustering) for a mesh network of single isntance brokers or a mesh network of active/standby brokers.
  Give broker name, broker type for instance type, Engine is Apache ActiveMQ, Username and Password and Engine Version. We can also select networking and securty and maintenance. It takes about 15 minutes to create.
  We have access to console and we are provided Endpoints  as well. We can see that it supports most protocols like AMQP, Openwire, MQTT, STOMP, WSS etc. We have to add security group to access.(For admin console,
  port is 8162 and allow IP).









