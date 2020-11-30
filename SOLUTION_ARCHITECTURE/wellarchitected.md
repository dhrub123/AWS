## WHitepapers and well Architected
+ Architecting for the Cloud: AWS Best Practices
+ AWS Well-Architectetd Framework
+ AWS Disaster Recovery(https://aws.amazon.com/disaster-recovery/)
+ FAQ

#### Well Architected Framework General Guiding Principles
+ Stop guessing your capacity needs, use auto scaling when you can.
+ You need to test your systems at production scale. In cloud , you can scale massively and pay for it and then scale down and stop paying for it.
+ You need to also automate to make architectural experimentation easier. The more you automate, the quicker you can iterate, the quicker you can experiment and the better will be your architecture.
+ You need to also allow for your architectures to evolve over time. So, this is not your on premise data center, this is a cloud and you can change everything at your fingertips.
	+ You need to design based on changing requirements and requirements will change and architectures will change.
+ You need to look at your data. You're going to drive architecture changes by looking at patterns of your application and usage.
+ Finally, you need to improve for game days
	+ That means, when you have a huge sale, simulate it before you go into it. For example, if you plan on running a Valentine's Day sale and you haven't planned for that capacity
	  or you don't know how your application will behave, then you may crash and not have a great Valentine's Day experience. So definitely test your systems production at scale,
	  simulate with game day, so just simulate a massive amount of traffic, and see if your architecture does hold ground.
+ The architected framework itself is five pillars or five categories.
	+ Operational Excellence
	+ Security
	+ Reliability
	+ Performance Efficiency
	+ Cost Optimization
+ These things are not something to balance. For example, you don't usually try security for performance or whatever. They're not trade offs. If done correctly, all of these things enhance each other
  and they become a synergy. It is not like Do I want to really improve security, but then have to pay more or whatever. We will have to pay more, but in the end, you won't run into security issues
  and so that will decrease your cost overall. They work together to improve each other.
+ We can look into the Well-Architected Tool which is a small AWS service.
  https://console.aws.amazon.com/wellarchitected 
+ Well-architected tool is an AWS service and here we can start to define a workload and over time we'll be able to track our performance and evolve our architecture.
  So let's define our workload and see how we are. We give name as demo workload and description as a social media network.. Let us mention the industry type as education and the industry
  as technologies and tools. Then we mention the environment as production and AWS regions as us-east-1 and us-west-1. We can also add non-aws region to add where it's actually running
  outside of AWS, so if I was running on premise I would say, on premise, or Azure or Google Clouds. Then all the account IDs, so where the workload is spanning. These are just metadata fields
  decribing how your workload is defined. Then, you need to start a review by answering questions for each pillar. At the end of it, you can generate a report. So you click on Start Review
  and this review asks you all the questions. For example, you have nine questions under Operational Excellence, 11 questions around Security, nine questions on Reliability, eight on Performance Efficiency
  and nine on Cost Optimization. Then it will give you is a report and you can look at your solution architecture over time, you can have milestones, you can have improvement plans, the report gives you 
  what is a high risk and what is a medium risk in your architecture and so on.

#### Operational Excellence

+ Operational excellence includes the ability to run and monitor systems to deliver business values and to continuously improve supporting processes and procedures.
  It's about having operations that are great.
+ Design principles
	+ Perform Operations as code - All your operations should be code, infrastructure as code is going to be the cornerstone of operational excellence, and infrastructure as code is CloudFormation.
	+ You need to annotate documentation - When you create documentation, make sure it's annotated, make sure it's commented, make sure it's clean and generated after every build of your application.
	+ You need to make frequent and small reversible changes, so that, in case of failures, you can reverse it. If you make huge changes every three months, this is not going to go well.
	+ You need to refine operations procedures frequently, so as the team gets more familiar with the operations, they refine it, automate more, and make improvements.
	+ Anticipate failure, that will always happen, and you need to learn from all these failures.
+ Operational Excellence transcribed in AWS Terms
	+ Prepare
		+ So, Prepare is going to be how do we basically prepare everything to have operational excellence. So you should use run-books, you should have good infrastructure standards,
		  you should do mock deployments, and for this, we could use AWS CloudFormation to prepare everything as a nice infrastructure as code.
		+ Config would be really nice as well to evaluate the compliance of your CloudFormation templates.
	+ Operate
		+ You need to operate and basically automate as much as you can, you should release fast, you should avoid manual processes.
		+ CloudFormation would be a great tool for this.
		+ Config as well.
		+ CloudTrail to make sure to track all the API codes that are done, and make sure nothing is done on purpose, or nothing is changed manually.
		+ CloudWatch to basically monitor the performance over time of your stack, and make sure that if you need to operate it, you know what operations to do.
		+ X-Ray will be able to trace API calls, to trace HTTP requests and to make sure that they're working correctly, and if they're not working correctly, it will point us to where the incorrect stuff happened.
	+ Evolve
		+ CloudFormation is a centerpiece of this whole operational excellence pillar, but also you need to be able to evolve over time.
		+ CICD tools basically CodeBuild, CodeCommit, CodeDeploy, CodePipeline, allow you to basically iterate quickly, deploy quickly, deploy often, deploy small changes.

#### Security

+ Security includes the ability to protect information, systems and assets while delivering business value through risk assessment and mitigation strategies.
  When your application is secure, you're really minimizing a risk over time and you save cost from disasters and you really don't want to have a risk, or a security issue in your company.
+ Design Principles
	+ Implement a strong identity foundation
		+ So we want to centralize how we manage user accounts. We want to rely on least privilege and maybe IAM is going to be one of these services to help us do that.
	+ Enable traceability
		+ We need to look at all the logs, all the metrics and store them and automatically respond and take action, every time something looks really weird.
	+ Apply security at all layers
		+ You need to secure every single layer, such as if one fail maybe the next one will take over. So edge network, VPC, subnet, load balancer, every EC2 instance you have, the OS, patching it,
		  the application, making sure it's up to date.
	+ Automate security best practices
		+ Security is not something you do manually, it's mostly done well, when it's automated.
	+ Protect data in transit and at rest
		+ That means always enable encryption, always do SSL, always use tokenization and do access control.
	+ Keep people away from data
		+ Why is someone requesting data? Isn't that a risk when you allow someone to access data, do they really need it or is there a way to automate the need for that direct access in that manual processing of data?
	+ Prepare for security events.
		+ Security events must happen some day in every company, so run response simulations, use tools to automate the speed of detection, investigation and recovery
+ Security transcribed in AWS Terms
	+ Identity and access management
		+ IAM
		+ STS for generating temporary credentials
		+ Multi-Factor Authentication token
		+ AWS Organization to manage multiple AWS accounts centrally.
	+ Detective controls - So how do we detect when stuff goes wrong?
		+ AWS Config for compliance
		+ CloudTrail to look at API calls that look suspicious.
		+ CloudWatch to look at metrics and things that may go completely out of norm.
	+ Infrastructure Protection
		+ CloudFront is going to be a really great first line of defense against DDoS attacks.
		+ Amazon VPC to secure your network and making sure you set the right ACLs.
		+ Shield to protect your AWS account from DDoS.
		+ WAF which is a Web Application Firewall
		+ Inspector to basically look at the security of our EC2 instances.
	+ Data protection
		+ KMS to encrypt all the data at rest.
		+ S3, which has a tons of encryption mechanism, we have SSES3, SSEKMS, SSEC or client side encryption. On top of it we get bucket policies and all that stuff.
		+ Elastic Load Balancer can be enabled to expose a HTPS end point.
		+ EBS volumes can be encrypted at rest
		+ RDS instances also can be encrypted at rest and they have SSL capability.
	+ Incident response - what happens when there's a problem?
		+ IAM is going to be your first good line of defense if there is an account being compromised. Just delete that account or give it zero privilege.
		+ CloudFormation will be great if someone deletes your entire infrastructure
		+ Automate incident response - If someone deletes a resource, maybe we should alert, CloudWatch Events could be a great way of doing that.

#### Reliabililty

+ Reliability is the ability of a system to recover from infrastructure or service disruptions, dynamically acquire computing resources to meet demand, and mitigate disruptions such as misconfigurations
  or transient network issues. So it's about making sure your application runs no matter what.
+ Design principles
	+ Test recovery procedures
		+ You need to use automation to simulate different failures or to recreate scenarios that led to failures before.
	+ Automatically recover from failure
		+ That means that you need to anticipate and remediate failures before they occur.
	+ Scale horizontally in case you need to have increased system availability, or increased load.
		+ Distribute requests across multiple, smaller resources to ensure that they do not have a common point of failure.
	+ Stop guessing capacity
		+ Maintain optimal level to satisfy demand without over or under provisioning. Use auto scaling wherever you can to make sure you have the right capacity at any time.
	+ Automation
		+ You need to change everything through automation, and this is to ensure that your application will be reliable or you can roll back.
+ Reliability in terms of AWS Services
	+ Foundations of reliability
		+ IAM to ensure that no one has too many rights to basically wreak havoc on your account.
		+ Amazon VPC, this is a really strong foundation for networking.
		+ Service Limits, making sure that you do set appropriate service limits. Not too high, and not too low, just the right amount of service limits, and you monitor them over time.
		  Such as if your application has been growing, and growing, and growing, and you're about to reach that service limit. You don't want to get any service disruptions, so you would contact AWS,
		  and increase that service limit over time.
		+ Trusted Advisor is also great.
	+ Change management, so how do we manage change overall?
		+ Auto Scaling is a great way. 
			+ Basically if my application gets more popular over time and I have set up auto scaling then I don't need to change anything, which is great.
		+ CloudWatch is a great way also of looking at your metrics. For your databases , your application making sure everything looks reliable over time, and if the CPU utilization starts to ramp up
		  maybe do something about it.
		+ CloudTrail in terms of are we secure enough to track our API calls?
		+ And Config
	+ Failure Management - so how do we manage failures?
		+ Backups - To make sure that your application can be recovered if something really really bad happens.
		+ CloudFormation to recreate your whole infrastructure at once
		+ S3 to backup all your data or S3 Glacier for archives that you need to touch once in a while.
		+ A reliable, highly available global DNS system, so Route 53 could be one of them. And in case of any failures, maybe you want to change Route 53 to just point to a new application stack
		somewhere else and really make your your application has some kind of disaster recovery mechanism.

#### Performance Efficiency

+ Performance Effciency includes the ability to use computing resources efficiently to meet system requirements, and to maintain that efficiency as demand changes and technologies evolve.
  So it's all about adapting and providing the best performance.
+ Design Principles
	+ Democratize Advanced technologies - You need to use advanced technologies and as the services become available, track them.
	+ You need to be able to go global in minutes so if you need to deploy in multiple regions it shouldn't last days, it should last minutes , maybe using cloud formation.
	+ Use serverless infrastructure. So that's the golden state. So that means you don't manage any servers, and everything scales for you, which is really awesome.
	+ Experiment more often - Maybe you have something working really well today, but you think it won't scale to 10 times the load. Experiment maybe try serverless architectures.
	+ Mechanical sympathy - Be aware of all the AWS services.
+ AWS services for performance efficiency
	+ Selection - Auto Scaling, Lamda, EBS, S3, RDS so you have so many choices of technology that scale at different patterns. So choose the right one for you.
	  Definitely, for example, Lamda needs to serverless, Auto Scaling is going to be for more EC2, EBS is when you know you need to have a disc, but you can sort of manage performance
	  over time using GB2 or I01. S3 if you want to scale globally, RDS, maybe you wanna use it to provision a database and maybe you want to migrate to Aurora. So you have a lot of things to select from.
	+ Review - so how do we review our performance
		+ Cloud formation, making sure we get exactly what we need before we create it. 
		+ And how do we keep updated on all this performance improvements? - AWS News Blog 
	+ Monitoring - So how do we know we are performing really well and as expected?
		+ CloudWatch with CloudWatch Alarms, CloudWatch metrics. CloudWatch dashboards can help you understand better how things work.
		+ AWS Lamda, as well. Making sure that you don't throttle your application . Lambda function runs in a minimal time
	+ Tradeoffs - So how do we make sure that we are doing the right performance decision?
		+ So RDS maybe versus Aurora.
		+ ElastiCache if you want to improve real performance,
		+ Snowball - Snowball will give us a lot of data moving very fast, but it will take maybe a week for the data to arrive. So the tradeoff is, do we want the data right away
		  in the cloud and use all our network capacity or do we wanna move that data through a truck and get this in a week from now. 
		+ With ElastiCache, do I want to have possibly outdated, stale data in a cache but really improve performance? Or do I wanna get the latest and not use ElastiCache? 
		+ CloudFront same thing. It does cache stuff around the edges. So if you use CloudFront, yes, you go global in minutes, but you have the possibility of everything being cached
		  for one day on people's laptops. So when you release an update to your websites, maybe it will take time for people to get the new stuff.
+ Performance should be really in the middle of your thought process as you do Solution Architecture.

#### Cost Optimization

+ Cost Optimization is the ability to run systems to deliver business value but at the lowest price point possible.
+ Design principles 
	+ Adopt a consumption mode - Pay for only what you use. So, for example, AWS Lambda is one of these services, if you don't use AWS Lambda you don't pay for it,
	  where as RDS, if you don't use your database you still pay for it because you've provisioned your database.
	+ Measure overall efficiency - use CloudWatch, Are you using your resources effectively? 
	+ Stop spending money on data center operations because AWS has the infrastructure for you, and they just allow you to focus on your applications, your systems,
	+ Analyze and attribute expenditure, so that means if you don't use tags on your AWS resources, you're going have a lot of trouble figuring out which application is costing you a lot of money,
	  so using tags ensures that you are able to track the cost of each application and optimize them over time and get a ROI based on how much money you generate from your business.
	+ Use managed and application level services to reduce the cost of ownership. So that means because managed services operate a cloud scale, they can offer a lower cost
	  per transaction or service.
+ Cost optimization in terms of AWS Services
	+ Expenditure Awareness - Making sure we know what costs us something. so Budgets, Cost and Usage Reports, Cost Explorer and for example Reserved Instance Reporting, making sure that if we do reserve an instance,
	  we're actually using them and not just paying for unused reserved instances.
	+ Cost-Effective Resources - are we using the right stuff, for example can we use Spot Instances, they are considerably cheaper, yes they do have some trade offs but can we use them?. If we know we're using a EC2 
	  instance for over a year, maybe three years because we provisioned a database on it , can we use reserved instances? That'll be a great way of saving money. AWS Glacier, so are we basically putting our archives
	  in the lowest price point possible and Glacier is the lowest price point possible.
	+ Matching supply and demand - So are we not over provisioning. So again, auto scaling or maybe AWS Lambda if you're using serverless infrastructure 
	+ Are we optimizing over time - so getting information from trusted advisor, or again looking at our cost and usage report. 
		+ A ELB feature allowed to use HTTP and HTTPS traffic going in but you couldn't do redirect of HTTP to HTTPS before and so you have to spin up an application that was doing the redirect behind the scenes
		  and that application was costing me a little bit of money, but then reading the news blog they said now you can straight from the ELB configure redirect of HTTP to HTTPS and that was great, it saved me a
		  bit of money every month just for that one feature, so reading the news does allow you to optimize your costs and make sure you have the right price point.
		+ If you're running an application on DynamoDB, but it's really inactive and you don't need to use a lot of operations. Maybe you're way better of using the on demand feature of 
		  DynamoDB instead of using the reserved capacity that'll use your RCU and so on, okay.

#### AWS Trusted Advisor

+ Trusted Advisor is a service and there is nothing to install.
+ It's going to give you a high level AWS account assessment and you'll get recommendations straight from UI.
+ How does it work? - Advisor will go ahead and analyze our AWS accounts and provide recommendations on many different topics
	+ Cost Optimization
	+ Performance
	+ Security
	+ Fault Tolerance
	+ Service Limits
+ Trusted Advisor will tell us if we're reaching a service limit. But Trusted Advisor is not really free.
	+ Core Checks and recommendations, that's free for all customers, but a lot of it is disabled unless you have a Business Support plan.
	+ You can get weekly email notifications directly, and you can enable that from the console.
	+ If you're a Business & Enterprise customer, you get full Trusted Advisor, and you can even set CloudWatch alarms when you start reaching your limits.
+ Handson - Go to Trusted Advisor and in the dashboard there are 5 topics. Cost Optimization, Performance, Security, Fault Tolerance and Service Limits.
	+ In Cost Optimization, nothing is available unless we upgrade our Support plan. If you're a corporation, for sure you would have a Support plan, and then you take advantage
	  of it, but without that, I don't have any cost optimization checks. Here we can see things like Low Utilization of Amazon EC2 Instances, e.g. if their CPU is less than 10%,
	  Idle Load Balancers, so basically the balancers that don't do much, Underutilized EBS Volumes, Unassociated Elastic IPs etc. So basically, all the very common ways
	  of wasting moneys on AWS. It complements the billing features. It doesn't supplement them.
	+ Performance which is yet again, locked and you have to upgrade your Support plan. Here there are things like High Utilization of Amazon EC2 Instances,
	  That means that there is more than 90% CPU instance daily on your instances. That means you're pretty much stuck, or you need to upgrade them or something's happening.
	  Same for EBS Volumes. Large Number of Rules in an EC2 Group. Basically that can lead to performance issues, but we're talking about a very large number
	  of security group rules, etc.
	+ Security is available. Here we can find things like which security groups have unrestricted ports. That means that for now, port 22 is unrestricted,
	  and that can be pretty, pretty bad. The idea is that here, I have all my groups here that have enabled port 22 unrestricted, and I could go and act upon it.
	  Tt sort of overlaps with Config. Config was more around compliance. This is more around recommendations. They're a little bit different, but they check
	  for the exact same thing, at least for security groups. We can also see Public Snapshots, RDS Public Snapshots, Bucket Permissions , IAM Use, and whether or not we have MFA enabled 
	  on root accounts. We get a lot more information if we do upgrade our Support plan.
	+ Fault Tolerance - We need to upgrade to the Support plan for this as well. Here we get things like information around the edge of the snapshots, Amazon EC2 availability Zone Balance
	  to make sure that you're really balanced across all the AZ, optimization for your load balance configuration etc.
	+ Service Limits - this has some features unlocked for non business users as well. We can check how we are doing for each service and if we're getting close to the limits.
	  If we're at 80% of the service limits, we'll get a little warning. Trusted Advisor is the way to get alarms and stuff for service limits.
	+ You do have your Preferences, where you can disable Trusted Advisor if you wanted to. But more importantly, you can set weekly email notification preferences,
	  and you can set email address for Billing Contact, Operations Contact, Security Contact. The idea is that if you want to just receive weekly emails
	  around the recommendations you can do to improve your Cost, Performance, Security, Fault tolerance and Service Limits, this will be the way. To enable these weekly email notification preferences,
	  you have to do this through the Trusted Advisor UI. 
	+ Trusted Advisor is a very simple service, and to leverage it fully ,you need support plan for your AWS accounts.

#### Architectures

+ https://aws.amazon.com/architecture/
+ https://aws.amazon.com/solutions/


