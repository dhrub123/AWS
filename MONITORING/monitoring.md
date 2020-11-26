## Monitoring and Audit

We have a application in cloud and it stops working at 2 am in the morning. We forgot to turn on monitoring which gives us logs , metrics, monitoring, audit etc.

#### AWS Cloudwatch metrics

+ CloudWatch provides metric for every services in AWS and monitoring is at the center of AWS.
+ A metric is a variable to monitor. So it could be CPUUtilization or NetworkIn or number of connections, etc.
+ Metrics belong to namespaces, so they'll be grouped, 
+ A dimension is an attribute of a metric, for example, which instance id has sent the CPUUtilization for EC2, that could be a dimension.
+ WE can have up to 10 dimensions per metric
+ Metrics will have timestamps, because it's a time series. We get a metric over time, so they will have timestamp.
+ We can create a CloudWatch dashboards of metrics.
+ We have some metrics called EC2 Detailed monitoring, its something that we have to enable.
	+ EC2 instances metrics are every five minutes, but if you enable detailed monitoring for a cost, you get data every one minute. If we use detailed monitoring,
	  we can get faster, auto scaling for our ASG, it could be a way of improving the scalability of our application. The AWS Free Tier will allow us to get
	  10 detailed monitoring metrics, so that's quite a nice tier.
	+ EC2 Memory usage is not pushed by default, it must be pushed from the inside of your instance as a custom metric, and for this we've used the monitoring scripts.
+ We can get custom metrics in CloudWatch.
	+ By default you can send your own custom metrics to the CloudWatch, using the CLI or whatever SDK you want.
	+ We can also send or receive dimensions or attributes to segment your metrics, up to 10.
		+ We can send Instance.id, Environment.name, etc. to give dimensions to our metrics.
	+ The metric resolution by default is going to be one minute, so we have a standard resolution of one minute, but you can have high volume, high resolution metrics
	  and you can send them up to one second, so every one second and obviously when you do use that higher resolution, you're going to pay a higher cost, so this is if you need extreme, detailed and extreme
	  information around of how your application is faring every one second.(StorageResolution API Parameter)
	+ Now to just send a metric to CloudWatch, you can use the API called PutMetricData, and that is to forecast metrics 
	+ In case you get any errors,we have to use a throttle with exponential back off, so that means that if you get an error, first you're just going to retry every one second and then two seconds and then four seconds
	  up until it works.
+ Handson - Go to cloudwatch. We will go to All metrics. There we can see metrics group like EC2, S3, Elasticache, Logs, RDS etc. We will launch an EC2 instance. Then we will go inside EC2 metrics group, then per-instance metric 
  and then select a metric for the instanceId say CPU Utilization and we get a graph for CPU Utilization over time.

#### Cloudwatch Dashboards

+ CloudWatch Dashboards are a great way to get access to your key metrics and to get a good overview of how your application is running based on how you design it.
+ Dashboards are global. You can access the Dashboard in any region.
+ We can make a Dashboard include graphs from different regions. Because that means that you can have a graph from Ireland and one from the USA and one from Asia together on the same CloudWatch Dashboard.
+ You can also change the time zone and the time range of the Dashboards straight from Dashboards themselves.
+ You can set automatic refresh. It can be 10 seconds, one minute, two minute up to 15 minutes.
+ You get three Dashboards up to 50 metrics for free. Afterwards it is three dollars per Dashboard, per month. So Dashboards can get quite pricey over time. But they're very, very useful if you have an 
  application to monitor in a very certain way.
+ Handson - We go to Cloudwatch and click on Dashboards. We click on create dashboard and give it a name. We can then add widgets to the dashboard like Line, Stacked Area, Number, Text, Query Results.
  Let us add a line widget and click configure. We will select metric CPU Utilization and select our EC2 instance id in Ireland. We then save the dashboard. We can select any kind of Absolute or relative timeframe. 
  We can refresh or Autorefresh our data. We can select timezone. Now if we switch our region to us-east-1, we still see our dashboard. Now we create an instance in us-east-1, and then in the dashboard, 
  add another line widget and configure it with cpu utilization of indtance in us-east-1. So now the dashbaord shows us both the graphs for the instances in ireland and us-east-1. We have to save the dashboard.

#### AWS Cloudwatch Logs

+ Custom Applications can send logs to CloudWatch using the SDK.
+ But if you wanna collect a log from an AWS service
	+ Elastic Beanstalk will directly collect log from your applications and send them to CloudWatch.
	+ ECS will do the collection from the container logs. 
	+ AWS Lambda, will get the collection from the function log.
	+ VPC Flow Logs will get VPC specific logs.
	+ We also have API Gateway logs.
	+ CloudTrail, if you set up a filter.
	+ CloudWatch if you set up log agents on EC2 machines.
	+ You can get Route 53 and you can log all the DNS queries that are made all around your infrastructure.
  So overall, you get a lot of services that can directly send logs to CloudWatch out of the box.
+ CloudWatch themselves can send the logs to whatever you want.
	+ It could be S3 if you wanted to archive it from time to time.
	+ It could be Elasticsearch cluster, for example, if you wanted to perform further analytics. Because Elasticsearch can have nice searching capabilities for logs.
+ Log Storage Architecture: 
	+ We need to have a log group and that's named whatever you want. Usually it's an application name, but it's really free naming.
	+ Within the log groups, you're gonna have many log streams. And this is what a stream of logs of a specific file, or an application, or a container will be. 
	  So this is usually when you have one log stream per container, one log stream per application, or per log file, that kind of things.
+ Log expiration policy - whether or not your log never expires, expires in 30 days, etc. Because you need to pay for data retention in CloudWatch. So the more data you store in CloudWatch,
  the more you're going to pay. So this is a good strategy maybe to have it for 30 days, export the logs to S3, and then delete in CloudWatch from time to time.
+ And then you can even use the AWS CLI if you want to tail the CloudWatch Logs which is a nice way of seeing how an application is behaving in real time.
+ If you are sending the logs to CloudWatch, some common mistakes is not to get the IAM permissions right.  And so when you don't have the IAM permissions right, obviously things won't work
  and it could be quite tricky to debug sometimes.
+ Logs can be encrypted. You can use KMS for encryption.
+ You can use CloudWatch Logs and you can use filter expressions. 
	+ And so with these filter expressions, for example, we can search for a specific IP inside of a log. So you can basically search for whatever you want.
	+ We can set up a metric filter, basically it should give a metric based on the filter you define. And you can use that metric filter to trigger CloudWatch alarms.
  The idea is that for example you're looking for a specific IP, you set up a metric filter. And then any time that IP appears, it will trigger an alarm and you will know about it right away. This 
  could be to detect an attacker or some shady behavior or whatever you want, really.
+ CloudWatch Log Insights - You can use this to query logs using a common language that's easy to use. And you can add queries directly into your CloudWatch dashboards.
  And so there are some sample queries that are given in the UI. And you can see, for example, the common query says I want the 25 most recently added log events, or the number for exceptions logged
  every five minutes, or the list of log events that are not exceptions. So you can have queries that are common, but also for CloudTrail, Lambda, Route 53, VPC Flow Logs. And you can even write your own.
  It brings a lot of usability and ease to query CloudWatch Logs, that was not easy to do before the filter expressions.
+ Go to cloudwatch logs - quick start guide will tell you how to install and configure the cloudwatch logs agent on a running ec2 linux instance. We can create an EC2 instance. In the IAM role, we should give
  access to cloudwatch by creating a role and to that role attach a policy like CloudWatchLogsFullAccess. Then we are going to install the agent by doing ssh.
  ```
  sudo yum update -y

  //install logs agent
  sudo yum install -y awslogs

  Edit file /etc/awslogs/awslogs.conf file
  
  //change region
  The file /etc/awslogs/awscli.conf points to us-east-1 region by default. For a different region we have to update region = <region name>

  //start logs agent
  sudo systemctl start awslogsd

  //we can check the file below for logs
  /var/log/awslogs.log

  ```
  In cloud watch we will see one log group created for /var/log/messages which contains the log stream of our ec2 instance. We can filter the logs. We can also epire the logs. We can delete the log stream or log group and export the data
  to S3 or stream to lambda or elastic search service.

#### Cloudwatch Agent and Cloudwatch Logs Agent

+ CloudWatch agents are used to take logs from EC2 instances, as well as metrics, and have them onto CloudWatch. By default, no logs are going from your EC2 instance from CloudWatch. 
  For this, you need to create and start an agent, which is a small program on your EC2 instances that will push the log files that you want.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/MONITORING/images/CLA.png" width="60%" height="60%"/>
+ Your EC2 instances will have the CloudWatch logs agent running, sending the logs into CloudWatch logs.
+ For it to work your EC2 instance must have an IAM role that allows it to send the log to CloudWatch logs.
+ CloudWatch log agent can also be set up on-premises servers. So it's possible for you to have your services, virtual servers, like VMware on-premises, and you install the exact same agent, which is a small Linux program,
  and your logs will end up in CloudWatch logs as well.
+ There are two different agents you can find in CloudWatch.
	+ You have the CloudWatch logs agent, which is the older one
	+ The CloudWatch unified agent, which is the newer one.
  They're both for virtual servers, EC2 instances,  on-premises servers etc.
+ The CloudWatch logs agents is the old version and can only send logs to CloudWatch logs.
+ The unified agents will collect additional system level metrics, such as RAM , processes etc. and also send the logs into CloudWatch logs. It's unified and can do both metrics and logs. You can configure that agent very easily
  using the SSM Parameter Store, which is a feature that the previous agent did not have. So you can do centralized configuration for all your unified agents.
  	+ So the CloudWatch unified agent can send logs in CloudWatch logs
  	+ It can also colelct metrics. If you install it on your EC2 instances or your Linux servers, you can collect metrics but at a way more granular level.
  		+ CPU metrics for example, active, guest, idle, system, user, steal.
  		+ Disk metrics like free, used, total and Disk IO in terms of number of writes, reads, bytes, IOPS.
  		+ RAM - free, inactive, used, total, cached.
  		+ Netstat, the number of TCP and UDP connections, net packets, bytes.
  		+ You get some information around the processes. Total number of processes, How many are dead, blocked, idle, running, sleep.
  		+ Swap space, which is some memory spilling on disk. So how much is free, used, and used percentage
  	+ The CloudWatch unified agent allows you to get a lot more metrics and a lot more granular details than the normal monitoring for EC2 instances.
+ Out-of-the-box for EC2, you get some information on disk, CPU, network, not memory, not swap, but all of this at a high level.

#### AWS Cloudwatch Alarms

+ Cloudwatch Alarms are going to be used to trigger notifications from any metric. 
+ Alarms can get attached to Auto Scaling Groups, they can be attached to EC2 Actions, or SNS notifications.
+ There are various options, so you can choose to alarm on the sampling, on a percentage, on a max, on a min. So you can basically customize your alarm on any thing you want.
+ The alarms come in three states. 
	+ It can be an OK state, that means your alarm is not doing anything;
	+ INSUFFICIENT_DATA, this is when you basically don't send enough data for your alarm, so the metric is missing some data points;
	+ ALARM, when your alarm threshold is being passed.
+ In terms of period, basically you have to specify a length of time in seconds to evaluate the metric, and if you use a high resolution custom metric, you can choose 10 seconds or 30 seconds
  as the period for the evaluation of your alarm.
+ Handson - We go to Cloudwatch and then go to Alarms. Here we can see the configured alarms in different states. We can see that alarm is in state alarm because network is less than 2MB for 
  1 datapoint in 5 minutes and so the ASG will need to scale down according to policy stated in ASG. We can create an alarm with networkIn > 100 MB for 5 minutes and 1 datapoint, what to do in case of 
  missing data, and action for alarm state and create alarm.
+ EC2 instance recovery with cloudwatch alarms
	+ On your EC2 instances, you have status check. And it could be an instance status to check the EC2 virtual machine or a system status to check the underlying hardware behind your EC2 instances.
	  So if either of these fail, then you lose your EC2 instance.
	+ We could set up a CloudWatch alarm to monitor a specific EC2 instance. And this CloudWatch alarm will look at the StatusCheckFailed_System metric, for example. If this alarm gets triggered,
	  you can have an action called an EC2 Instance Recovery which triggers some internal mechanism within AWS to recover your instance.
	  That means that you will have the same private IP as before, the same public IP, the same elastic IP if one was attached, the same metadata and the same placement group.
	  So this could be something quite helpful if you need to recover an EC2 instance and make sure it keeps its private IP.
	  Now any data that was stored, for example, on an instance store will not be kept. But if you have EBS volumes then the data will be kept because it will be stored onto EBS volumes.
	  So to recover an instance automatically, you set up a CloudWatch alarm that will be checking for the status checks of the EC2 instance and the action of that CloudWatch alarm
	  can be an EC2 Instance Recovery. And then finally because that alarm can be triggered, it can alert and send a notification into an SNS topic.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/MONITORING/images/IR.png" width="60%" height="60%"/>


#### AWS Cloudwatch Events

+ You can have a cron Job to schedule events in CloudWatch. So notifications would trigger on event.
+ You can have an Event Pattern in CloudWatch Events and that will basically allow you to define rules such as if a service in AWS is doing something, we are going to trigger an event if that event
  matches the rule. So for example if a Cloud CodePipeline state changes, we can detect that using CloudWatch Events and then trigger, Lambda functions or send a message to SQS, SNS, Kinesis.
+ When you have CloudWatch Event, as an output the event will create a small JSON document and will give a little information about the change.
+ Handson - We go to cloudwatch and click on events. There we can create a rule. We can choose a event pattern or schedule. If we select a schedule, we can create an event every 5 minute or 5 hours or whatever 
  interval we like or we can set up a cron expression and then we have to select a target like a Lambda, Kinesis etc. For event pattern, we have to select a service like say CodePipeeline and we 
  have to select event type like CodePipeline Execution State Change and Specific State as Failed and then add a target like lambda or SNS and configure inputs like matched events or part of matched event etc. 
  and then we create the rule with a name and enable it. 
+ We can set up as many rules as we want for different sources and targets.

#### AWS Cloudtrail

+ CloudTrail is one of the simplest service into AWS, but yet, one of the most useful.
+ It provides you governance, compliance, and audit for your AWS accounts. 
+ Because CloudTrail is enabled by default, you will get a history of events and API calls made within your AWS account. So, anytime you do anything in your AWS account, or any application does something,
  you know it will appear in CloudTrail. And so, if you do something from the console, from the SDK, from the CLI, and your AWS services, this will appear in CloudTrail.
+ Additionally in CLoudTrail, you can put all the logs of things that happened in CloudWatch Logs, and that gives you the ability to just keep the events all the time.
+ Now, if a resource is deleted into AWS, we should look into cloudtrail.
+ Handson - We select region and select cloudtrail and see all the recent events in the last 90 days. It tracks user activity and API usage. We can see the event details which include date, time, source IP,
  API etc. We can even filter events by deletes like Delete Bucket.

#### AWS Config

+ AWS Config helps you with auditing and recording compliance of your AWS resources over time.
+ It will record configuration and the changes to your configuration over time, and it will display that to you, into the console.
+ Additionally, you can store all this configuration data over time as well, into S3 and maybe you want to analyze it using Athena in a serverless fashion, to quickly extract the insights you need.
+ So the questions that can be solved by AWS Config would be
	+ Is there an unrestricted SSH access to my security group?
	+ Are my buckets public exposed, do they have any public access?
	+ How has my ALB configuration changed over time?
  Again, I record it, and roll it back.
+ You can receive alerts in the form of SNS notifications for any changes that happened within your configuration.
+ And Config, even though it's a per-region service, you can aggregate it across region and all accounts to get a global view.
+ If it's green, that mean it is compliant and if it's red, that mean it's not compliant.
+ You can view the configuration of a resource over time to see what had changed.
+ There is a deep integration with CloudTrail, if you enable CloudTrail. So that you can see who did the changes, in terms of what changes were done.
+ Then you can use AWS Managed Config Rules(Over 75)
	+ They help you evaluate the compliance.
	+ You can use AWS managed config rules and there are over 75 of them 
	+ You can make your own custom config rule. And for this you must define that rule using AWS Lambda.
		+ So for example, the rules we can make is, evaluate if each EBS disk is of type gp2.
		+ Evaluate is each EC2 instance is type t2.micro.
	+ Rules can be evaluated or triggered based on different factors. Any time there is a config change, so it's a reactive type of rule evaluation, or at the regular time interval, if for example, your config changes are a lot,
	  or very often, it just wants you run it every hour, for example.
	+ You can also trigger CloudWatch Events if a rule is non-compliant, and then chain that with Lambda, and do some remediation.
 	+ Rules can also have a auto remediation setting. And so if a resource is not compliant, you can trigger an auto remediation.
 		+ For example, say you have a rule that checks if instances have tags, and if not, if they're not-approved, maybe your remediation will be to stop these EC2 instances.
 	+ Config Rules do not prevent actions from happening, there is no deny, it's not like IAM, which denies stuff from happening in your accounts. But it helps you evaluate the compliance of your resources over time,
 	  and maybe you remediate to that. But it's not a preventive action, it is something that reacts to changes.
+ Pricing - AWS Config is not free. You have to pay about $2 per active rule per region, per month. There is no partial month.
+ Handson - Go to AWS Config. If we click get started, it asks us to select types of resources to record. We can record all resources in the region and we can include global resources as well. We can also select specific 
  resources. We can then log that change into an AWS S3 bucket which we can analyze later using Athena. We can also send notification using SNS. Then we have to create role for AWS Config. Then we have to choose config rules.
  We will skip this and create the Config. After this is done, all resources in this region will be tracked. We will then add a rule and there are many AWS managed rules. Eg. we can use a compliant rule where only approved
  AMIs are used. Another rule can be restricted SSH which can check if SSH groups are allowing unrestricted incoming SSH traffic. We can define a trigger which will trigger this rule any time a configuration changes and the
  scope is specific resources like EC2 security groups. We can also trigger it periodically. We create the rule. All Security groups in our region which are being tracked will be evaluated now. Suppose we see that one security group
  is non compliant and we make changes to that for remediation. That change will show up in the configuration timeline or the compliance timeline. The dashboard will show us compliance and non compliance details. 


#### Cloudtrail vs Cloudwatch vs Config

+ CloudWatch is for performance metrics, metrics, CPU, network, and to create dashboards. You can also get events and alerts, and finally, we have a log aggregation and analysis tool if we wanted to.
+ CloudTrail is to record API calls made within your account by everyone and everything, and you can define some trails for specific resources, so you can get more information on EC2 only, and it's a global service.
+ AWS Config is to record configuration changes and to evaluate resources configuration against compliance rules. You're going to get a timeline of changes and compliance with its nice UI.

For an Elastic Load Balancer
	+ CloudWatch can monitor the number of incoming connections, can visualize number of error codes as a percentage over time, and maybe we can have a dashboard to get an idea of the load balancer performance,
	  maybe we can have a global dashboard if you have multiple load balancers for a global application.
	+ We will use AWS Config on ELB to track the security group rules for the load balancer, making sure no-one does anything fishy or changes anything, maybe you want to track the configuration 
	  changes for the load balancer itself, to see if anyone modifies the SSL certificate, etc. We also maybe have a rule to say, oh there's always should be an SSL certificate assigned to the load balancer,
	  and maybe we should never allow non-encrypted traffic into the load balancer. So there could be two different compliance rules that you put into Config.
	+ CloudTrail will be to track who made any changes to the load balancer with API calls. So, in case someone changes the security group rules or someone changes the SSL certificate, or removes it, or whatever,
	  then through CloudTrail , we will know who made these changes.



