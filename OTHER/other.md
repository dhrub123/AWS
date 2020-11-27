#### CICD

+ Continuous Integration
	+ Developers usually when they do code, the push the code to a repository - GitHub or Bitbucket but on AWS it's called CodeCommit.
	  Developers they push the code often.
	+ Then there should be a testing or build server that basically checks the code as soon as it's pushed. It tests the code. We have Jenkins but on AWS it's called CodeBuild.
	  It is going to get the code, build it and test it.
	+ The reason we do this is so the developer gets feedback about the test and checks if it passed or failed.
	+ So basically as a developer, the more often you push code the faster you know if it's good code if it's passed the build and the test.
	  So you can find bugs early and fix them early.
	+ You can deliver faster as the code is tested often and you deploy often.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/CI.png" width="60%" height="60%"/>
+ Continuous Delivery
	+ It's to ensure that the software that has been created and built can be released reliably whenever needed.
	+ The deployments happen often and very quick. 
	+ You shift from the mindset of a release every three months to five releases a day, maybe once per hour as well.
	+ That means automated deployment. So the technologies on AWS that allow you to do this
		+ CodeDeploy - You cna define strategy, how fast the rollout of new code will be.
		+ Jenkins CD
		+ Spinnaker
	+ As a developer again we push code often, the build server which has CodeBuild will get the code, build and test and then every time there's a passing build
	  maybe you want to deploy this through deployment server and by using CodeDeploy. Then we get our application server V1 and then after the upgrade V2.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/CD.png" width="60%" height="60%"/>
+ So we have code, build, test, deploy and provision. To basically store codes, we have AWS CodeCommit which is similar to GitHub or a third party code repository.
  Then to build and test we use CodeBuild and it's kind of similar to Jenkins CI. Then for deploying and provisioning, we have two options.
  We have Beanstalk, which does kind of both. You provision both the infrastructure and then on top of it, there is an API to deploy often. Or you can manage your own EC2 fleet,
  using CloudFormation and then to deploy to the EC2 fleet, you would use something like AWS CodeDeploy. Because you need to orchestrate all these tools one after the other,
  maybe you need something like a pipeline orchestrator and this is called AWS CodePipeline.
  	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/OR.png" width="60%" height="60%"/>

#### Cloudformation

+ Currently we've been doing a lot of manual work, where we create everything and it's very tough to reproduce.
	+ So if we wanted to do this in another region, in another AWS account, or within the same region but everything was deleted it'd be really hard.
+ If we create too many EC2 instances, security groups, Route 53 routes, it will be cumbersome.
+ What if we had our infrastructure be in code. What if you could just have code and that code will be deployed and created, updated, deleted all the time to make our infrastructure evolve.
  That is the concept behind infrastructure as code.
+ CloudFormation is a declarative way of outlining your AWS infrastructure for any resource and most of them are supported. So that means that within a CloudFormation template , you'll say
	+ I want a security group
	+ I want two EC2 machines using this security group
	+ I want two Elastic IPs for these machines
	+ I want an S3 bucket
	+ I want a load balancer in front of these machines
+ When you specify all of these things in a CloudFormation template and run it on CloudFormation, your CloudFormation service automatically creates all of these things for us, in the right 
  order with the exact configuration that we specify.
+ Benefits
	+ Infrastructure as code
		+ No resources will be manually created , which is excellent for control.
		+ The code can be version controlled using git for example.
		+ Any changes to the infrastructure are reviewed through code.
	+ Cost
		+ Each resource within the stack will be tagged with a CloudFormation identifier so we can easily see how much a whole CloudFormation stack will cost us, which is really nice.
		+ There's a small feature to estimate the cost of your resources using the CloudFormation templates.
		+ So a saving strategy for example will be, in Dev you automate the deletion of all the templates at 5 p.m. and you recreate them all at 8 a.m. safely and that could be a way to save cost
		  because you're not working at night.
	+ Productivity
		+ You have the ability to destroy and recreate an infrastructure on the cloud, on the fly.
		+ And you get automated generation of Diagram for your templates if you wanted to just create some PowerPoint slides. 
		+ And you get declarative programming, so you don't need to figure out which is the ordering, or the orchestration, or what goes before what.
	+ Separation of concern
		+ You can create as many stacks as you want. So usually one stack per app, and many layers.
			+ You get VPC stacks
			+ Network stacks
			+ App stacks.
	+ you don't need to reinvent the wheel
		+ you can leverage existing templates on the web
		+ you can just leverage the documentation.
+ How does it work ? 
	+ Templates have to be uploaded in S3 and then you reference them in CloudFormation.
	+ And to update a template, you cannot really edit the previous one , you have to re-upload a new version of the template to AWS.
	+ These CloudFormation stacks will be identified by name.
	+ So when you delete a stack in CloudFormation, it deletes every single thing that was created by the CloudFormation stack. So it's a very clean way of deleting resources.
+ Deploy CloudFormation templates
	+ Manual way 
		+ Edit the template in the CloudFormation Designer
		+ Use the console to input parameters etc.
	+ Automated way
		+ Edit the template in a YAML file
		+ Use the CLI to deploy the templates if you wanted to fully automate your flow
+ Building blocks in CloudFormation
	+ Template components
		+ Resources and it's a mandatory one and this declare the AWS resources that will be used in a template.
		+ Parameters if you want to have dynamic inputs for our template.
		+ Mappings, which is static variables for our template.
		+ Outputs, which is references to what has been created.
		+ Conditionals, list of conditions to perform resource creation.
		+ Metadata
	+ Templates helpers
		+ References 
		+ Functions
+ CloudFormation is a great way to have infrastructure as code, or to reproduce a stack into another environment. 
+ Handson - Go to Cloudformation , select region and create a stack. We get 3 options - Template is ready, use sample template or create template in designer. If we choose , template is ready, 
  we have to upload it to S3 or give s3 URL. We can view the file in designer. Give a name to the stack and create stack. We are going to get updates in events window. Cloudformation also add some tags.
  We can update the stack and preview the changes. We can look at resources in resources tab. If we delete stack, everything created by cloud formation is deleted.
  	+ This will create an EC2
  	  ```
		---
		Resources:
		  MyInstance:
		    Type: AWS::EC2::Instance
		    Properties:
		      AvailabilityZone: us-east-1b
		      ImageId: ami-a4c7edb2
		      InstanceType: t2.micro
  	  ```
  	+ This is a little more complex
  	  ```
  	    ---
		Parameters:
		  SecurityGroupDescription:
		    Description: Security Group Description
		    Type: String
        #Resources
		Resources:
		  MyInstance:
		    Type: AWS::EC2::Instance
		    Properties:
		      AvailabilityZone: us-east-1b
		      ImageId: ami-a4c7edb2
		      InstanceType: t2.micro
		      SecurityGroups:
		        - !Ref SSHSecurityGroup
		        - !Ref ServerSecurityGroup
		  # an elastic IP for our instance
		  MyEIP:
		    Type: AWS::EC2::EIP
		    Properties:
		      InstanceId: !Ref MyInstance
		  # our EC2 security group
		  SSHSecurityGroup:
		    Type: AWS::EC2::SecurityGroup
		    Properties:
		      GroupDescription: Enable SSH access via port 22
		      SecurityGroupIngress:
		      - CidrIp: 0.0.0.0/0
		        FromPort: 22
		        IpProtocol: tcp
		        ToPort: 22
		  # our second EC2 security group
		  ServerSecurityGroup:
		    Type: AWS::EC2::SecurityGroup
		    Properties:
		      GroupDescription: !Ref SecurityGroupDescription
		      SecurityGroupIngress:
		      - IpProtocol: tcp
		        FromPort: 80
		        ToPort: 80
		        CidrIp: 0.0.0.0/0
		      - IpProtocol: tcp
		        FromPort: 22
		        ToPort: 22
		        CidrIp: 192.168.1.1/32
        #outouts if any
		Outputs:
		  ElasticIP:
		    Description: Elastic IP Value
		    Value: !Ref MyEIP
  	  ```

+ Stacksets 
	+ It is an advanced feature and it allows you to create, update or delete CloudFormation stacks across multiple accounts and regions at the same time.
	+ The administrators need to create a StackSet and then you need to define trusted accounts that can create, update or delete Stack instances from StackSets 
	+ When you update your StackSet, all the associated stack instances are updated in all the accounts and all the region. 
	+ So it's like an administrative portal into multiple regions and multiple accounts for CloudFormation.
	+ If we have the admin account where we define our StackSets and it's going to deploy into the Account in us-east-1 and in Account A in ap-south-1 and to Account B in eu-west-2.
	+ We can have also different accounts.
	+ Deploying a CloudFormation stack globally or across accounts means using StackSets.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/SS.png" width="60%" height="60%"/>

#### ECS

+ ECS is a container orchestration service 
+ It helps you to run Docker containers on EC2 machines
+ ECS is made of multiple things 
	+ ECS Core or the original ECS where you run Docker container and ECS on user-provisioned EC2 instances. Here you provision your own server.
	+ Fargate and we run ECS again on AWS-provisioned compute, so it's more server-less. You run it on Amazon servers, so it's server-less.
	+ EKS to run ECS on Kubernetes. That's provided by AWS. So if you wanna use Kubernetes as an orchestration service, you can use EKS.
	+ ECR is the Docker Container Registry hosted by AWS.
+ ECS and Docker are very popular when you wanna have microservices. So anytime you see microservice, you either think Lambda or ECS.
+ IAM security is supported and and the roles are at the ECS task level.

**Docker is basically a container technology.** 
	+ We run a containerized application on any machine that has Docker installed.
	+ So first, you containerize your application and then it can run anywhere.
	+ The portability that is provided to us that's really amazing. Having containers allow our application to work the same way anywhere, whether it be on your local laptop,
	  on an EC2 machine, on an on-premise server, really, anywhere is what makes Docker super popular.
	+ When containers run on a machine, they're isolated from each other, so it's secure.
	+ You can control how much memory and CPU will be allocated to each container.
	+ You're also able to restrict network rules, so how these containers communicate with one another.
	+ It's more efficient than virtual machines. 
	+ You can scale containers up and down very quickly in a matter of seconds.

+ Use cases for ECS
	+ Running microservices.
		+ So you can run multiple containers on the same machine
		+ You can do service discovery features to enhance communication
		+ You have a direct integration with a load balancer
		+ And you have auto-scaling capability.
	+ You can also run batch processing or schedule tasks.
	 	+ So you can schedule ECS containers to run on on-demand, reserved, or spot EC2 instances.
	+ And you can also help migrate applications to the cloud with ECS
		+ Dockerize is the action of making a container, a Docker container out of an application.
		+ If you Dockerize a legacy application that's running on-premise, then you can quickly move it after that to ECS because your Docker container can run on ECS as well.
+ Concepts
	+ You're going to run, for example, three EC2 instances.
	+ And the first thing we're going to define is an ECS cluster. And your ECS cluster is going to be a set of these EC2 instances.
	+ On top of it, you're going to define ECS services. And so services is basically an application, so it's an application definition. And you're going to be running on your ECS cluster
	  and this is where you're going to say how many applications you want running and the auto-scaling rules etc.
	+ And your ECS service is going to create ECS task, alongside their definition. And so the tasks are going to be basically a Docker container running on your EC2 machine.
	  And so they can be running on any machine of that cluster. e.g. : I have six task containers running on three EC2 machines. It is possible for the same service
	  to run multiple times on the same EC2 machine and that's provided as a capability by Docker. You can run not just one ECS service, but you can run another ECS service. 
	  And you don't need to know that it runs across all your EC2 machines. It can run maybe across two EC2 machines if you only have two task containers that you need.
	+ Ech task container can have a IAM role. So each task container can be isolated and secure and have its own set of permissions.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/ECS.png" width="60%" height="60%"/>
+ ECS - ALB integration.
	+ ALB is application load balancer and it's very popular to use ALB with ECS because of a feature called port mapping.
	  And that is a new feature of the application load balancer absent in classic load balancer. So we can run multiple instances of the same application
	  on the same EC2 machine.
	+ Use case of this is that you have increased resiliency even if you have only one EC2 instance. You can maximize your utilization of CPU and cores.
	  And you're able to perform rolling upgrades without impacting application uptime. 
	+ We have an ECS instance and it's going to be running our application. So maybe we have four containers and they all run a node.js. And so we have control over 
	  what the container does, but then Docker automatically assigns a dynamic port for us. So here, we have four ports, they're all different and they're all dynamic.
	  And so the idea is that we want to expose this application to the world. So for this, we have an application load balancer and it's going to be exposed on port 80 for HTTP
	  or 443 for HTTPS. And so our application load balancer, if it was a classic one, it would just map to one port, 6789 and say, okay, there's only one application I can reach.
	  But because of this dynamic port mapping feature, the application load balancer is super smart and it will just redirect the traffic to each container with the right port.
	  So using this architecture, you're basically able to run the same application, the same container many, many times over the same ECS instance, leveraging the dynamic port mapping
	  of the application load balancer. 
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/ECS_1.png" width="60%" height="60%"/>
+ ECS Setup and Config File
	+ You have to run an EC2 instance, And on it, you will install the ECS agent alongside the ECS config file.
	+ If you use an ECSS-ready Linux AMI,you already have the ECS agent installed on it and running, but you will still need to modify the config file.
	+ The config file is going to be at /etc/ecs/ecs.config. There are 4 configs that you need to know about out of, like, maybe 35 configs you have for this file.
		+ ECS_CLUSTER=MyCluster - We have to assign EC2 instance to an ECS cluster. It's to say to the EC2 instance to which cluster it belongs.
		+ ECS_ENGINE_AUTH_DATA={...} - If you want to authenticate and pull images from private registries.
		+ ECS_AVAILABLE_LOGGING_DRIVERS=[...] - To enable CloudWatch logging
		+ ECS_ENABLE_TASK_IAM_ROLE=true which is to enable IAM roles for ECS tasks and so you need to have this variable to be true.
	+ We have to remember that we have to assign an EC2 instance to a cluster, pull images from private registry, enable CloudWatch Logs integration and finally
	  enable IAM task roles for your ECS task.
+ ECR - Elastic Container Registry
	+ Your Docker containers , they need to be stored somewhere before they can be run and that is ECR which stores, manages and deploys your containers on AWS.
	+ It's fully integrated with IAM and ECS
	+ The data is sent over HTTPS, so there is encryption in flight
	+ Your images, when they're stored, they're also encrypted at rest
	+ Your ECR is going to run two images, Application A, so image A, and application B, image B. And you have an ECS instance and maybe you want to run the image B,
	  so you're going to do a request to pull the Docker images from ECR. ECR is going to check with IAM that you can indeed pull the Docker images. And then your images will be pulled from the ECR
	  onto your ECS instance, which will run your containers in return. We push the containers to ECR using CLI or use CodeBuild for your CICD to automate this task. You push images to ECR and image
	  will get pulled by the ECS instance when they run your application.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/ECR.png" width="60%" height="60%"/>
+ IAM Task Roles 
	+ We have an EC2 instance. And ECS agents on it because it's using ECS. It's running three tasks. So it's going to run Application A Task 1, Application B task 1
	  and Application A task 2. So in this case, our Application A has two task instances running on the EC2 instance, and our Application B has one. So the EC2 instance,
	  should have an IAM role, allowing it to access the ECS service so that the ECS agent can do what it needs. So we're going to create an EC2 instance role.
	  And we're going to give it enough access for the ECS agent to call the ECS API. Now that we have defined that EC2 instance role, each task is going to inherit the
	  EC2 instance role in terms of permissions. And each task needs to do something different, maybe integrate to DynamoDb, to S3 and so on. So how can we make sure
	  that each task can exactly do what it needs to do. And for this, we need to create what's called an ECS IAM task role, to perform each API calls for the application.
	  So we're going to create a task role specifically for Application definition A. And it's going to be attached to the two tasks, and a different task role will be
	  attached for Application B. So, this task role, will allow us to get the maximum amount of security, meaning that our Application A is going to have a role dedicated to it,
	  allowing A to do what it needs to do, for example, talk to DynamoDb, and Application B as well, who get a seperate task role, allowing it to use IAM for talking for example, to S3.
	  And so this way, we have two different kinds of instance roles. For this example, we have an EC2 instance role attached the EC2 level, and they will help be helpful for the ECS agent,
	  and then each application will have a task role attached to it. **We define this task role in the task definition using a taskRoleArn parameter**, which allows us to just reference that task role.
	  + <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/TR.png" width="60%" height="60%"/>
+ Fargate
	+ When you launch an ECS Cluster, you have to create your own EC2 instances. And if you need to scale you need to add EC2 instances, so we manage infrastructure.
	+ But if you use the Fargate service, it's going to be all serverless. So we don't provision any EC2 instances, you just create the task definitions maybe you need to reference an IAM task role.
	  And then we'll launch our service and AWS automatically, will run the container for us, and it will just find where to run it. So we don't have any EC2 instances to manage.
	  It's all on AWS. 
	+ To scale, we just increase the number of tasks that we want within our service. We don't have any EC2 instance to manage. So if we see Docker images, ECS and serverless seamless scaling , think Fargate.

#### EKS

+ EKS stands for Amazon Elastic Kubernetes Service.
+ It is a way to launch managed Kubernetes clusters on AWS.
+ Kubernetes is an open-source system to do automatic deployment, scaling, and management of containerized application, usually they're dockerized, so they're running under Docker containers.
  Both ECS and EKS have the same goals, to run your Docker containers onto AWS, but it's a different API. ECS is going to be specific to AWS, whereas Kubernetes is open-source, it's shared by 
  many cloud providers, for example, Google cloud provider, Azure, AWS and also on-premises infrastructure.
+ This could be a great way for you to migrate to AWS by using EKS, if you're already running Kubernetes.
+ EKS supports EC2.  You can run your EKS clusters on EC2 to deploy worker nodes, so you deploy EC2 instances, and they subscribe to your EKS cluster, and you can run your containers on them.
  This is very similar to ECS classic. Fargate allows you to deploy serverless containers onto EKS. 
+ EKS and ECS have the exact same capability. You have ECS classic to run on EC2 and you have ECS Fargate, and so for EKS you get EKS EC2 and also EKS Fargate. The only difference is that ECS 
  is going to run Docker containers the Amazon way, whereas EKS will run them the Kubernetes way.
+ Use case for EKS is if your company is already using Kubernetes on-premises or from another cloud, and you want to migrate to AWS then, you should start using EKS. 
  This way you don't have to reinvent the wheel.

#### Step Functions and SWF

+ Step Functions are a way to have a serverless visual workflow to orchestrate your Lambda functions.
  So if you have a ton of Lambda functions and you do a lot of things that need to be orchestrated, to run different Lambda functions and you can use Step Functions to orchestrate all of that.
+ All the flow is represented as a JSON state machine, so any time you see state machine in your exam, this is probably when Step Functions are going to be used.
+ You can use a sequence of Lambda functions, parallel Lambda Functions, conditions, timeouts, error handling, so you get a lot of flexibility.
+ You can also integrate with other services, but it's less popular, such as EC2, ECS, On premise servers, and API Gateway. Step Functions really help with Lambda functions overall.
+ The maximum execution time is one year, so you get a lot of time to complete the workflows.
+ And you have the possibility to implement human approval feature.
+ Use cases
	+ Order fulfillment
	+ Data processing
	+ Web applications
	+ Any workflow
+ Any time it says workflow, state machine, orchestration of Lambda functions, think Step Functions.
+ Visual workflow in step functions
	+ You're going to create a JSON state machine and it's going to give you a beautiful graph. We have a 'Start' and an 'End',
	  and so you have 'Submit Job', 'Wait a number of seconds', 'Get a job status', and 'Is the job complete? If so, come back to wait',
	  'If it's complete then go to the next thing', then you get 'Is the job failing' or 'Is the job succeeded?' and then you go to the end.
	+ It's just a visual way of representing how to orchestrate everything, and so all these little boxes may be Lambda functions or a specific Step Function's features.
	  Now, when you actually start a job, you can see it as it happens, going through the chart. So you can see right now it's been submitted, and then it goes into the 'Wait X Seconds',
	  so it's in progress and it's blue. And so it goes along and does all the things, and then when it's done, you can see the complete path of what it took, which branch it went into,
	  and you can look into the status of all these steps.
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/STEP.png" width="60%" height="60%"/>
+ SWF - Simple Workflow Service
	+ It coordinates work amongst applications.
	+ The code runs on EC2, so it's not exactly serverless.
	+ There's a one year maximum runtime
	+ There's a concept of 'activity step' and 'decision step'
	+ It has a built-in 'human intervention' step.
	+ Example - Order fulfillment from the web to warehouse to delivery as a feature.
	+ The difference with Step Functions is that SWF is older, and this is pretty much not supported any more by AWS.
		+ The documentation and the FAQ recommend you to use Step Functions for new applications except 
			+ if you need external signals to intervene in the process
			+ to have child processes that return values to parent processes.
	+ So if you see SWF and Step Functions in the same question, just ask yourself the question, "Do I need external signals or child processes?"

#### EMR

+ EMR stands for Elastic MapReduce and MapReduce is a big data term.
+ EMR helps creating Hadoop clusters, which is used for big data. 
  Big data is used to basically analyze and process a lot of data like petabytes or terabytes of data. 
+ These clusters that you can create through EMR, they can be made of hundreds of EC2 instances.
+ The benefit of EMR is that EMR knows how to coordinate and configure all these instances to work together so you don't have to worry about setting up a Hadoop cluster.
+ You can just use all the engines, including Spark, HBase, Presto, and Flink to analyze your data and process it.
+ EMR provisions everything and it configures everything.
+ It can be cost conscious, so there's auto-scaling that works with EMR.
+ It's integrated with Spot Instances, so that means if you want to pay a lower price point for your big data analysis, you can use Spot Instances.
+ Use cases for EMR are data processing, machine learning, web indexing, and big data, all this kind of stuff.
+ Handson 
	+ EMR will give us a Hadoop framework, and we can create cluster, then we upload some data, maybe coming from S3. Then we create all the information to input it,
	  and then we can monitor the health of our cluster and see how the processing is going. And, we can retrieve the output of our processing in S3.
	+ If you want to create a cluster, you would name your cluster, you would say where you want logging to be. You would want the launch mode to be either a cluster thats long running 
	  or just step execution, which is when you want to do an analysis and then shut down the cluster. Then you have to select the release of EMR you want and each release comes
	  with different sets of technologies. Then we specify the instance type for our EMR cluster, and how many instances we want. So 3 right now, but you can go up to, say 100 if you wanted to.
	  Then security and access settings like if you wanted to provide a key pair, some permissions, and then you would click on create cluster, and you have cluster with all the technologies that are specified here.

#### AWS GLue

+ AWS Glue does ETL which stands for Extract, Transform and Load and that means that you are able to move data from various data sources into other data targets and through this process you're able
  to transform it, clean it, change the data format etc., You can maybe do analytics on it.
+ Glue means ETL
+ Because it's managed, you can automate all the time consuming steps of preparation for data analytics 
+ It's serverless, it's pay as you go, fully managed, and in the back it works with Apache Spark which has excellent performance.
+ It has a capability to crawl data sources. So if you provide a database and it will be able to find all the data sources, all the tables, all the data formats using schema inference.
+ It gives you automated code-generation if you want to customize the Apache Spark code.
+ Sources - Aurora RDS Redshift and S3
+ Sinks - S3, Redshift, etc.
+ It supports pretty much anything to anything.
+ There is glue data catalog and it basically comprises all the metadata like the definition and the schema of all the source tables in your infrastructure and that can be used by EMR as well.
+ So Glue is fully managed ETL, it's serverless, it provides code-generation and there is Glue data catalog.

#### AWS OpsWorks

+ Chef and Puppet are two open source softwares and they allow you to perform server configuration automatically or you can perform repetitive actions.
+ They work ly with EC2 and On Premise VM
+ In the AWS world, Opsworks is Managed Chef and Puppet. So Opsworks is just another name for Chef and Puppet.
+ And overall, it's kind of an alternative to AWS SSM.
+ Chef and Puppet
	+ They basically help managing configuration as code where all the stuff you run onto your VMs are coded.
	+ That helps in having consistent deployments because the same code can be applied on many different machines.
	+ That works with Linux and Windows and you can automate a lot of things such as user account, cron jobs, NTP, packages, services, etc.
	+ In Chef it is called Recipes and in Puppet it is called Manifest. So they leverage Recipes or Manifests, which is basically configuration as a code.
	+ They have a lot of similarities with some AWS products like SSM, Beanstalk, CloudFormation.
	+ But we use Chef and Puppet in AWS because they're open source products, and they work across different clouds. So if you have Amazon Cloud, Microsoft Azure, and maybe Google Cloud,
	  they provide one way of managing all your instances.

#### Elastic Transcoder

+ So Elastic Transcoder is a way to convert media files, so video and music. They're stored into S3, and you can transform them into various formats, different size, different templates,
  different formats physically and they can be optimized for tablets, PCs, Smartphone and TV.
+ We use Elastic Transcoder becuase it has a lot of features that come out of the box - it can optimize the bit rates, create thumbnails, create watermarks for your video, creates captions,
  creates protections with DRM, progressive download and even encryption.
+ $ components of Elastic Transcoder
	+ The first one is a job and the job describes what the work of the Transcoder is.
	+ The pipeline, which is a queue to manage all the transcoding jobs.
	+ The presets, which is physically the templates for converting media from one format to another. So you would say, this size, this quality, this bit rate
	+ Then finally notifications, if you wanted to have SNS notifications, when a transcoding job is done.
+ Finally, this is kind of this server-less service, so you pay for what you use. It scales automatically and it's fully managed.

#### AWS Workspaces

+ It is a managed secure cloud desktop, 
+ If on premise you are provisioning VDI or Virtual Desktop Infrastructure, it is a lot of management to deploy these environments
+ WorkSpaces allow you to just get On Demand a cloud desktop, and you're going to pay just for usage. So if you use it for one hour, you're going to pay for just one hour of this secure cloud desktop.
+ So it just provides you On Demand access to virtual desktops.
+ It's going to be secure, encrypted, network isolation
+ It's integrated with Microsoft Active Directory. That means that if you're using a Microsoft account to access your VDIs, then you can use that same account to access your secure cloud desktop with WorkSpaces.
+ That means that you as a user can get a desktop provisioned through WorkSpaces. We have a desktop being provisioned, most likely it's going to be Windows, but now there's support for Linux as well.
  and so from that desktop, which is a secure connection between you and desktop, maybe you can access securely your corporate data center infrastructure, or maybe you can access securely your AWS Cloud resources.
  So it is a great way of having a tight management, a tight security, and making sure that your users can work from anywhere securely using virtual desktops.
+ So at the exam when you see the word VDI or how do you place a VDI infrastructure think WorkSpaces
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/OTHER/images/VDI.png" width="60%" height="60%"/>

#### Appsync

+ AppSync is a service to store and sync data across mobile and web apps in real time.
+ It synchronizes data between your mobile and web apps in real time.
+ It makes the use of GraphQL. AppSync is a managed GraphQL.
+ So the client code can be generated automatically.
+ There is integration with DynamoDB, Lambda.
+ There is real time subscriptions, offline data synchronization, and that's an alternative for Cognito Sync
+ You get Fine Grained Security.
+ When you see synchronizing data between mobile and web apps in real time it is going to be either Cognito Synch or AppSync, and if
  you see GraphQL, then it is going to be AppSync for sure.

#### Cheatsheet

+ CodeCommit: service where you can store your code. Similar service is GitHub
+ CodeBuild: build and testing service in your CICD pipelines
+ CodeDeploy: deploy the packaged code onto EC2 and AWS Lambda
+ CodePipeline: orchestrate the actions of your CICD pipelines (build stages, manual approvals, many deploys, etc)
+ CloudFormation: Infrastructure as Code for AWS. Declarative way to manage, create and update resources.
+ ECS (Elastic Container Service): Docker container management system on AWS. Helps with creating micro-services.
+ ECR (Elastic Container Registry): Docker images repository on AWS. Docker Images can be pushed and pulled from there
+ Step Functions: Orchestrate / Coordinate Lambda functions and ECS containers into a workflow
+ SWF (Simple Workflow Service): Old way of orchestrating a big workflow.
+ EMR (Elastic Map Reduce): Big Data / Hadoop / Spark clusters on AWS, deployed on EC2 for you
+ Glue: ETL (Extract Transform Load) service on AWS
+ OpsWorks: managed Chef & Puppet on AWS
+ ElasticTranscoder: managed media (video, music) converter service into various optimized formats
+ Organizations: hierarchy and centralized management of multiple AWS accounts
+ Workspaces: Virtual Desktop on Demand in the Cloud. Replaces traditional on-premise VDI infrastructure
+ AppSync: GraphQL as a service on AWS
+ SSO (Single Sign On): One login managed by AWS to log in to various business SAML 2.0-compatible applications (office 365 etc)




















