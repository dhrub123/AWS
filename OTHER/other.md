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
		+ CodeDeploy
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
		      AvailabilityZone: us-east-1a
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

		Resources:
		  MyInstance:
		    Type: AWS::EC2::Instance
		    Properties:
		      AvailabilityZone: us-east-1a
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

		Outputs:
		  ElasticIP:
		    Description: Elastic IP Value
		    Value: !Ref MyEIP
  	  ```