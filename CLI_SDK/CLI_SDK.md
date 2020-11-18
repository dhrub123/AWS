## CLI and SDK

To be a developer on AWS we need to know about AWS CLI or AWS command line interface whcih allows us to programmatically access AWS. 
When we created stuff in AWS manually within a console , they exposed standard information from our clients.
+ We could SSH into an EC2 instance.
+ The RDS database was just a standard MySQL or Postgres database and we could just connect using the URL or the connection string.
+ ElastiCache exposes a URL and you can connect to it.
+ ASG or ELB were automated and there were managed services and we really cannot do much with them. We set them up manually.
+ Route 53 was setup manually

Developing against AWS has two components
+ Perform interactions with AWS without using the on line console.
+ Interact with AWS proprietary services like Amazon S3, DynamoDB and others.

When you perform a task in AWS you can do it in several ways.
+ You can use the CLI from your local computer.
+ You can use the CLI from EC2 machines.
+ You can use the SDK from the local computer.
+ You can use the SDK on your EC2 machines.
+ You can use the AWS instance metadata service for EC2.

#### Setting up AWS CLI version 2

+ https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

To check if AWS CLi has been properly installed.

```
Dhrubas-MacBook-Pro:~ dhruba$ aws --version
aws-cli/2.1.2 Python/3.7.4 Darwin/19.6.0 exe/x86_64
```

#### Configure AWS CLI

In this lecture we're going to configure the AWS CLI and there are ways to do it properly. Your computer will access the AWS network in your registered accounts using the CLI or command line interface over the world wide web.
When you connect your computer using the CLI to your AWS accounts, it will check for credentials and permissions **Do not share your AWS access key and secret key with anyone.** They're just like your username and your password
and you should not share them or show it to anyone. To set up our AWS CLI, we need to configure it and for that , we need to get access credentials from the IAM service.
+ Go to IAM Service > click on users, find your user and then click on security credentials. We will put access keys into our AWS configure commands. We create an access key and downlaod it and it can only be viewed or 
  downloaded at creation time.
+ Go to the CLi and type ```aws configure``` and you will be prompted for the access key ID, then the secret access key, then the default region name and finally the output format is none, we can leave it as the default.
  Press Enter and the AWS CLI is configured.
+ If we type ```aws configure``` again, it will show that we have an access key ID, a secret access key, the region name and the format and we can modify them if we want to. When we run the ```aws configure``` command, it goes 
  and creates some files into a small folder called .aws on Linux and mac, and it's also called .aws. We can navigate by typing ```ls ~/.aws```.  It creates two files, one called config and one called credentials, so if you just look at what these files are, if we look at the config file it shows the region I mentioned earlier and that means that my AWS CLI calls will default to this region. Now if I cat the other file called credentials we can see that it has the AWS access key ID and this AWS secret key which it will use to interact with AWS.
+ These files are confidential. To invalidate the configuration, we simply delete the access key or make it temporarily inactive. 
+ When we access CLI from EC2, never use aws configure command. It is a very bad practice to store aws secrets in EC2 because it is insecure. So we have to use IAM roles instead whcih can be attached to EC2 instances. The IAM roles come with
  policy which defines what the EC2 instance should be able to do and by default the policy has no rights. EC2 instances automatically use this IAM roles and profiles without any additional configuration. 
+ So we do as configure but do not enter access key and access id, we just enter region. Then we go to IAM role and create a role. We will create a role for EC2 service, most popular is lambda and ec2. Here we are going to create a role
  for EC2 and look for s3 read only access policy. We will attach this policy to this role and create this role. Now we go to Instance Settings > Attach or repalce IAM role > Attach the role we created. It takes some time to take effect.
  Now if we ssh into the ec2 instance and do ```aws s3 ls``` , it will work. We can also list files within the bucket by using ```aws s3 ls s3://<bucket_name>```. But only read operations will work since we attached a read policy. If we 
  try to do operations like make bucket ```aws s3 mb s3://12345678```, that is not going to work. We can attach the s3 full access policy for full access. We can remove the bucket by doing ```aws s3 rb s3://12345678```

#### CLI with S3

+ https://docs.aws.amazon.com/cli/latest/reference/s3/
+ ```
	List buckets in s3
	------------------
	dhruba$ aws s3 ls
	2020-10-16 17:30:40 dayita-bucket
	2020-11-18 22:26:27 dhruba-bucket-cors
	2020-11-18 22:12:55 dhrubas-bucket	
	List files in a bucket
	----------------------
	dhruba$ aws s3 ls s3://dhruba-bucket-cors
	2020-11-18 22:43:09         69 extra-page.html
	copy or downlaod a file in local computer from s3
	-------------------------------------------------
	dhruba$ aws s3 cp s3://dhruba-bucket-cors/extra-page.html sample.html
	download: s3://dhruba-bucket-cors/extra-page.html to ./sample.html
	Make Bucket
	-----------
	dhruba$ aws s3 mb s3://123mydhruba
	make_bucket: 123mydhruba
	Remove Bucket
	-------------
	dhruba$ aws s3 rb s3://123mydhruba
	remove_bucket: 123mydhruba
  ```

#### IAM Roles and Policies

+ We can use AWS managed policy or create our own. When we create a policy , we have to mention Service, Actions(List, Read, Write, Permission Management and drill down further), Resources(If Specific mention ARN/* or All resources) and  
  Request Conditions and we can also import  manaaged policy. We can also use Policy Generator. e can also add versions of policy.
+ We can also add inline policies to a role.
+ We can test policies using AWS Policy Simulator - https://policysim.aws.amazon.com/home/index.jsp?# We can mention users and roles and test them against Service and specific actions(Amazon S3 , List Bucket for example)

Example : 
AWSS3ReadOnlyAccess policy - This is saying that only get and list operations are allowed on resources of S3.
```

    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": "*"
        }
    ]
}
```

AWSS3FullAccess - This is saying that any action is allowed on any resources in S3
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```

#### AWS Instance metadata

+ It basically allows your EC2 instances to learn about themselves, so they don't have to use an IAM role for that purpose. The Metadata is the info about the EC2 instance where as the user data, was to launch a script of the EC2 instance.
  They are compeltely different.
+ The URL for this is ***http://169.254.169.254/latest/meta-data***. It is basically an internal IP to AWS, it will not work from your computer, it will only work from your EC2 instances.
+ Using this, you can retrieve the IAM role name from the Metadata, but you cannot retrieve the IAM policy.

+ Some commands
  ```
  1.Curl - gives us the verion of api call
  [ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254
	1.0
	2007-01-19
	2007-03-01
	2007-08-29
	2007-10-10
	2007-12-15
	2008-02-01
	2008-09-01
	2009-04-04
	2011-01-01
	2011-05-01
	2012-01-12
	2014-02-25
	2014-11-05
	2015-10-20
	2016-04-19
	2016-06-30
	2016-09-02
	2018-03-28
	2018-08-17
	2018-09-24
	2019-10-01
	2020-10-27

  We will choose latest.
  2.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest
  dynamic
  meta-data
  user-data
 
  3.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest/meta-data
	ami-id
	ami-launch-index
	ami-manifest-path
	block-device-mapping/
	events/
	hibernation/
	hostname
	iam/
	identity-credentials/
	instance-action
	instance-id
	instance-life-cycle
	instance-type
	local-hostname
	local-ipv4
	mac
	managed-ssh-keys/
	metrics/
	network/
	placement/
	profile
	public-hostname
	public-ipv4
	public-keys/
	reservation-id
	security-groups
	services/

  4.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest/meta-data/instance-id
	i-08170f91f62019187
  
  5.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest/meta-data/local-ipv4
	172.31.23.218
  6.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest/meta-data/hostname
	ip-172-31-23-218.ec2.internal
  7.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest/meta-data/iam/info
	{
	  "Code" : "Success",
	  "LastUpdated" : "2020-11-18T18:50:12Z",
	  "InstanceProfileArn" : "arn:aws:iam::140862900615:instance-profile/awss3access",
	  "InstanceProfileId" : "AIPASBTAVJGDUXRHOQWQD"
	}
  8.[ec2-user@ip-172-31-23-218 ~]$ curl http://169.254.169.254/latest/meta-data/iam/security-credentials/awss3access
	{
	  "Code" : "Success",
	  "LastUpdated" : "2020-11-18T18:49:44Z",
	  "Type" : "AWS-HMAC",
	  "AccessKeyId" : "XXXXXXXXXXXXXXXXXXX",
	  "SecretAccessKey" : "XXXXXXXXXXXXXXXXXXXXXXXX",
	  "Token" : "Vey big Alpha Numeric string",
	  "Expiration" : "2020-11-19T01:14:30Z"
	}
	So the EC@ instance gets temporary short lived credentials when they are given access through an IAM access role , in this case awss3access jsut liek we see above.
  ```

#### AWS SDK

SDK is for software development kits. If you want to perform actions on AWS, but not from using the CLI, just from your code in whatever language you're using, you can use the AWS SDK. There is the official SDK, and there's unofficial SDK.
+ Official SDK are for Java, .Net, node.js, PHP, Python, Go, Ruby, C++ and a lot of languages and most languages support the AWS SDK anyway. "boto3 or botocore" is an alternative name for the Python SDK.
+ SDK is really useful when we start coding against AWS services such as DynamoDB.
+ The AWS CLI itself uses the Python SDK, or boto3. It is a wrapper around the Python SDK.
+ If you don't specify or configure a default region, then the API codes will default to the us-east-1 region when you use the SDK.
+ For credentials, because security is a big part of the SDK and CLI, it's recommended to use the default credential provider chain. It works seamlessly with 
	+ your AWS credentials at ```~/.aws/credentials``` (on premise or on local computer) when you run AWS configure
	+ If you use an EC2 machine and you use IAM Roles, then you can use an Instance Profile Credentials, and the SDK again automatically will look for these credentials.
	+ A little bit less recommended, but still used in some place, you can use environment variables (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY)
	+ Never ever store your AWS credentials in your code. It should be abstracted from these credentials, and rely on your aws configure credentials in your machine, or the instance profile credentials. 
	+ You need to use IAM Roles if you work within AWS Services.
+ When you use the SDK, there's something called Exponential Backoff. So when you use a API call, and it fails because it's been called too many times across too many applications of yours, you go into a strategy called Exponential Backoff,
  and that's only for rate limited API. So the SDK usually implements a retry mechanism with Exponential Backoff.Your first API call after failure will wait maybe 10 millisecond, and your second API call will run after 20 millisecond.
  And the third API call, as you can see the double length of the arrow, it will be for 40 millisecond. And then the next API call, if it still fails, it will wait double of that time, and then obviously, the next one, the final one,
  will still wait double of that time. So the Exponential Backoff means that if your API calls still keep on failing , we will wait twice as long as the previous API call to try again, and that ensures that you don't overload
  the API by trying it every millisecond. Exponential Backoff is included in most SDKs.

  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/CLI_SDK/images/EXP_BACKOFF.png" width="60%" height="60%"/> 





