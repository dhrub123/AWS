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





