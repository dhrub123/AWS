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

#### KMS - Key Management Service
+ KMS is encryption within an AWS service. With KMS keys, you can easily control who and what can access your data
+ AWS will manage all the encryption keys for us.
+ KMS is fully integrated with IAM for authorization so that makes the management of these rules very simple and centralized.
+ KMS is at the center of AWS and it is integrated with different AWSServices.
	+ EBS : We encrypt EBS volumes
	+ AWS S3 : Server side encryption of objects
	+ Amazon Redshift : encrypt your data
	+ RDS : encrypt your underlying data
	+ SSM : parameter store to perform and encrypt some secrets etc.
+ We can use KMS not only with the AWS services but also with the CLI or any AWS SDK.
+ KMS - Customer Master Keys(CMK). There two types of KMS.
	+ Symmetric keys(AES-256-bit) of encryption keys.
		+ They were the first offering from AWS
		+ You use a single encryption key for encrypting and decrypting data and this is why it's called a symmetric encryption key.
		+ A lot of services that are integrated with KMS actually used under the hood this symmetric customer master keys.
		+ They're also necessary for envelope encryption
		+ With KMS, when you generate a symmetric key, you actually never get access to the key unencrypted . You must use the KMS API to use that key and you actually never see the key.
	+ Asymmetric key (They belong to the RSA and ECC key pairs)
		+ Here you have two keys, you have a public key for encryption and the private key for decryption
		+ We can use them for encryption and decrypting, so, with a public key, we encrypt and with a private key, we decrypt or also a new use case in KMS called sign/verify operations.
		+ The public key is something you can download because it is public but again, access to the private key is impossible.
		+ The use case of asymmetric keys would be to do an encryption outside of AWS by users who can't call the KMS API.
		+ Asymmetric keys have a public and private key that is used to encrypt, decrypt or sign/verify operations and the use case is to do encryption outside of AWS
		  by users who cannot call the KMS API.
+ Symmetric Keys
	+ We're able to fully manage the keys and the policies.
		+ So you can create these keys,
		+ you can define rotation polices,
		+ disable them, enable them,
	+ Through CloudTrail integration, you're able to audit the key usage, so see who use the keys and when.
	+ There are three types of customer master keys.
		+ AWS managed service default customer master key, which is free, so this is the idea when you go into EBS volume and use the AWS/EBS key, this is going to be free.
		+ Then, if you create your own keys, this is $1 per month and there is no free tier
		+ If you import your own keys, so if you have to generate them outside of KMS, even though it's not very recommended, then it's also going to be $1 per month.
	+ And then, for KMS, you're going to pay for each API call done to KMS so we're talking about 3 cents per 10,000 calls.
+ KMS Usage
	+ Anytime you need to share some sensitive information, you would use KMS 
		+ Database passwords
		+ credentials for an external service,
		+ a private key of SSL certificates, 
	+ The real value is that we actually don't see the keys to encrypt the data or decrypt so the whole security belongs with AWS. We can send data to KMS to decrypt and encrypt
	  and KMS can rotate these keys for extra security.
+ Encryption
	+ You would never store any of your secrets like database password etc. in plaintext, and especially in your code.
	+ You should encrypt these secrets instead, so you can encrypt them and then you can store them in code or environment variables
	+ KMS has a limit, and you can only encrypt up to 4 kilobytes of data per call
	+ If you want to have more data encrypted, then you need to use something called envelope encryption.
	+ To give access to KMS to someone, we need to make sure 
		+ the key policy allows the user to access the key
		+ the IAM policy allows the API calls
	  When these two things are together, then you get access to KMS key.
+ KMS keys are bound to a specific region. When you create a KMS key in region A, it cannot be transmitted over to region B.
  So let's say we have an encrypted EBS volume with KMS and a KMS key in the region eu-west-2 and we'd like to copy that volume across to a new region for example, ap-southeast-2.
  Because KMS keys are linked to a specific region , you would need to do a specific operation. So here is the process and this applies to EBS volumes, Redshift snapshots and pretty much 
  everything that is encrypted with a AWS KMS. So, first you would create a snapshot of your volume and any snapshot made from an encrypted volume is also encrypted with KMS and the same key.
  Then, you need to copy that snapshot over to the new region but you will specify a new KMS key to re-encrypt the data with so  KMS Key B in the other region and now you have a snapshot encrypted with KMS
  in the other region but with a new key. And finally, when you recreate a volume from that snapshot then that volume will be encrypted with a new KMS Key B. Because KMS keys are bound to a specific region
  when you copy snapshots over they have to be re-encrypted.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/KMS.png" width="60%" height="60%"/>
+ KMS key policies
	+ To control access to KMS, we saw there were IAM policies and key policies. So key policies are similar to S3 bucket policies or any kind of resource policies
	  but the difference is that you cannot control access without them. So if you don't specify a key policy, then no one can access your key.
	+ The default KMS key policy, by default, is very permissive, it's created by default if you don't provide a specific KMS key policy
	+ It's going to give access to the root user which means that your entire AWS account has the right to use this KMS key, if they have the IAM policy available.
	+ To give users access to KMS keys using this default KMS key policy, you just create the correct IAM policy and attach it to the user.
	+ If you define a custom KMS key policy, then you would specifically define the users and the roles they can access with this specific KMS key and define who can administer the key
	  and this is going to be very helpful when you do cross-account access of your KMS key.
+ Cross-account copying of snapshots
	+ When you create a snapshots, it would be encrypted with your own CMK
	+ then you would attach a key policy to authorize cross-account access on that key.
	+ In this example key policy , we allow the target account to read our KMS key then we would share the encrypted snapshots and in the target account, we would create a copy of the snapshots
	  because we have access to the KMS key in our original accounts and then finally, we would create a volume from that snapshots and this is how we create and copy a snapshot across accounts. 
	+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/KMS_1.png" width="60%" height="60%"/>
+ Handson - Go to KMS (Key Management Service). We have AWS Managed Keys, Customer Managed Keys and Custom Key Stores which is for AWS HSM cluster.. In AWS Managed Keys, we see that they start with aws/.
  These are free and we cannot change them , delete them or rotate them or use them outside the service. Then we have Customer managed Keys for which we will have to pay 1$ per month.  When we cretae
  a CMK, we have to configure CMK - symmetric or asymmetric. We can choose symmetric and we will choose KMS as origin. We can also provide external. We need to then give alias and description for key.
  THen we have to provide key administrators. if we do not do anyhting here, we ae going along with default key policy.  Then we have to give key usage permissions. If we do not do anything, then default.
  We can provide access to other accounts as well. THen finish. The CMK is created and enabled. We can disable the key and schedule key deletion. The default key policy allows everyone in. The CMK 
  can be rotated every year.
  ```
  # 1) encryption
    # This will give us base 64 file with encrypted content.
	aws kms encrypt --key-id alias/tutorial --plaintext fileb://ExampleSecretFile.txt --output text --query CiphertextBlob  --region eu-west-2 > ExampleSecretFileEncrypted.base64

	# base64 decode for Linux or Mac OS  - This will give us a binary file and this is somethign we share.
	cat ExampleSecretFileEncrypted.base64 | base64 --decode > ExampleSecretFileEncrypted

	# base64 decode for Windows
	certutil -decode .\ExampleSecretFileEncrypted.base64 .\ExampleSecretFileEncrypted


	# 2) decryption - we get back the base 64 encoded value

	aws kms decrypt --ciphertext-blob fileb://ExampleSecretFileEncrypted   --output text --query Plaintext > ExampleFileDecrypted.base64  --region eu-west-2

	# base64 decode for Linux or Mac OS - we get the value
	cat ExampleFileDecrypted.base64 | base64 --decode > ExampleFileDecrypted.txt


	# base64 decode for Windows
	certutil -decode .\ExampleFileDecrypted.base64 .\ExampleFileDecrypted.txt
  ```

#### SSM Parameter Store

+ SSM Parameter Store is to securely store your configuration and secrets in AWS and you can have optional encryption with KMS. 
  So you can store your secretsand have them KMS encrypted directly from within the SSM Parameter Store.
+ It is a serverless service, it's scalable, durable and the SDK is super easy to use
+ You have versioning of your configuration and secrets.
+ All your security for your configuration management is done using path and IAM policies.
+ You can get notifications with CloudWatch Events
+ Integration with CloudFormation
+ We have an application and it has a parameter stored in the Parameter Store, it could be a plain-text configuration in which case, if we request that configuration, then the Parameter Store
  will check with IAM permissions to make sure that we can get them and then return them to us, or we can also ask for encrypted configurations, in which case the parameter Store will also ask IAM
  but on top of it, check the KMS permissions and if so, call the decrypt API from the KMS service to give us our decrypted secret.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/SSMPS.png" width="60%" height="60%"/>
+ This is way to store your parameters in your Parameter Store. So you can create a hierarchylike below
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/HI.png" width="60%" height="60%"/>
  It's sort of like a folder structure like a file system.
+ you're also able, using the Parameter Store, to reference secrets from the Secrets Manager - ```/aws/reference/secretsmanager/secret_ID-in_Secrets_Manager```
+ you're also able to reference parameters directly from AWS. - ```/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2``` e.g. : this allows you to retrieve the latest AMI ID
  for Amazon Linux 2 from AWS.
+ So if a lambda function wants to access your dev parameters, then you would set an environment variable and then your lambda function will get your parameters by path and retrieve them.
  And if you have a prod lambda function with another environment variable, then it would retrieve automatically the prod values. This is how we could use lambda and the Parameter Store.
  <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/LAM.png" width="60%" height="60%"/>
+ We have two tiers of parameters in the Parameter Store. We have the standard tier and the advanced tier and the standard tier is going to be free, advanced tier's going to be paid.
	+ For the standard tier, you have up to 10000 parameters per account, which is a large amount of parameters. The maximum size is four kilobytes and you don't have the parameters policies available.
	+ If you're using the advanced tier, then you get up to 100,000 parameters. They can be up to eight kilobytes, you do get parameters policies and you do have to pay for your parameters.
+ Parameter Policies
	+ They're only for advanced parameters and they allow you, e.g. to assign a TTL, so a Time to Live to a parameter, which effectively creates an expiration date and the idea behind this is to 
	  force updating or deleting sensitive data such as passwords in your Parameter Store.
	+ You can assign multiple policies at a time.
	+ We have 3 examples here
		+ The first one is the Expiration, to delete a parameter. So in this example, I'm going to say hey, my parameter expires in December 2020.
		+ Then we have an ExpirationNotification, so you're saying hey, this one, send me a notification through Cloud Watch Events 15 days before the expiration happens.
		+ And here is a NoChangeNotification. So this is saying, if my parameter hasn't been changed in 20 days then send me a notification through CloudWatch Events.
		+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/PP.png" width="60%" height="60%"/>
	+ So this are the kind of policies you can attach to your advanced parameters to trigger some sort of automation and to force yourself to change them quite often.
+ <img src="https://raw.githubusercontent.com/dhrub123/AWS/master/SECURITY/images/PS.png" width="60%" height="60%"/>
+ Handson - Go to systems manager and then parameter store tab on the left.
  + We create a parameter called /my-app/dev/db-url and put a string. The parameter is created and we can see that it is versioned. 
  	We then create a parameter called /my-app/dev/db-password and put a secure string(we will encrypt the value this time using KMS). We can use default key or a key we created in KMS.
  	We create for prod now /my-app/prod/db-url and /my-app/prod/db-password.
  + Now we will try to use them using CLI.

    ```
    # GET PARAMETERS
	aws ssm get-parameters --names /my-app/dev/db-url /my-app/dev/db-password

	{
    "Parameters": [
	        {
	            "Name": "/my-app/dev/db-password",
	            "Type": "SecureString",
	            "Value": "AQICAHi92FVTJO2Gcsn6nCqxSDLm/f8fY6T++aAr3xOGX4Cg0wFgALiUYB6AqfFipzFnWD0PAAAAazBpBgkqhkiG9w0BBwagXDBaAgEAMFUGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM05nVVdQYCBR4+GgxAgEQgCjsnccpZgnuf7YfVArsNvGWzN6S3Z9PTIltVkre0rn9OdipDQwdY8ZT",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:09:06.034000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/dev/db-url",
	            "Type": "String",
	            "Value": "devurl.dhruba:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:05:28.600000+05:30",
	            "ARN": "arn:aw s:ssm:us-east-1:140862900615:parameter/my-app/dev/db-url",
	            "DataType": "text"
	        }
       ]
    }

	# GET PARAMETERS WITH DECRYPTION
	aws ssm get-parameters --names /my-app/dev/db-url /my-app/dev/db-password --with-decryption

	{
    "Parameters": [
	        {
	            "Name": "/my-app/dev/db-password",
	            "Type": "SecureString",
	            "Value": "devdbpassword",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:09:06.034000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/dev/db-url",
	            "Type": "String",
	            "Value": "devurl.dhruba:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:05:28.600000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-url",
	            "DataType": "text"
	        }
    	]
	}

	# GET PARAMETERS BY PATH
	aws ssm get-parameters-by-path --path /my-app/dev/

	{
    "Parameters": [
	        {
	            "Name": "/my-app/dev/db-password",
	            "Type": "SecureString",
	            "Value": "AQICAHi92FVTJO2Gcsn6nCqxSDLm/f8fY6T++aAr3xOGX4Cg0wFgALiUYB6AqfFipzFnWD0PAAAAazBpBgkqhkiG9w0BBwagXDBaAgEAMFUGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM05nVVdQYCBR4+GgxAgEQgCjsnccpZgnuf7YfVArsNvGWzN6S3Z9PTIltVkre0rn9OdipDQwdY8ZT",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:09:06.034000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/dev/db-url",
	            "Type": "String",
	            "Value": "devurl.dhruba:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:05:28.600000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-url",
	            "DataType": "text"
	        }
	    ]
	}

	# GET PARAMETERS BY PATH RECURSIVE
	aws ssm get-parameters-by-path --path /my-app/ --recursive

	{
    "Parameters": [
	        {
	            "Name": "/my-app/dev/db-password",
	            "Type": "SecureString",
	            "Value": "AQICAHi92FVTJO2Gcsn6nCqxSDLm/f8fY6T++aAr3xOGX4Cg0wFgALiUYB6AqfFipzFnWD0PAAAAazBpBgkqhkiG9w0BBwagXDBaAgEAMFUGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM05nVVdQYCBR4+GgxAgEQgCjsnccpZgnuf7YfVArsNvGWzN6S3Z9PTIltVkre0rn9OdipDQwdY8ZT",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:09:06.034000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/dev/db-url",
	            "Type": "String",
	            "Value": "devurl.dhruba:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:05:28.600000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-url",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/prod/db-password",
	            "Type": "SecureString",
	            "Value": "AQICAHi92FVTJO2Gcsn6nCqxSDLm/f8fY6T++aAr3xOGX4Cg0wGF9vD6yvifHYVt9etXnpIVAAAAaTBnBgkqhkiG9w0BBwagWjBYAgEAMFMGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMeT8svPRCuE1SKKSZAgEQgCZ9dGOE8ElgP3VWvOUXdfgcnq/6PsBIZoUjv0MRYFPko641RASzaw==",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:11:38.531000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/prod/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/prod/db-url",
	            "Type": "String",
	            "Value": "produrl:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:11:10.553000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/prod/db-url",
	            "DataType": "text"
	        }
	    ]
	}

	# GET PARAMETERS BY PATH WITH DECRYPTION
	aws ssm get-parameters-by-path --path /my-app/ --recursive --with-decryption

	{
    "Parameters": [
	        {
	            "Name": "/my-app/dev/db-password",
	            "Type": "SecureString",
	            "Value": "devdbpassword",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:09:06.034000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/dev/db-url",
	            "Type": "String",
	            "Value": "devurl.dhruba:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:05:28.600000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/dev/db-url",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/prod/db-password",
	            "Type": "SecureString",
	            "Value": "prdpassword",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:11:38.531000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/prod/db-password",
	            "DataType": "text"
	        },
	        {
	            "Name": "/my-app/prod/db-url",
	            "Type": "String",
	            "Value": "produrl:3306",
	            "Version": 1,
	            "LastModifiedDate": "2020-11-28T09:11:10.553000+05:30",
	            "ARN": "arn:aws:ssm:us-east-1:140862900615:parameter/my-app/prod/db-url",
	            "DataType": "text"
	        }
	    ]
	}

    ```

  + SSM Parameter store with Lambda
  ```
  #Python Lambda Program

    import json
	import boto3
	import os

	ssm = boto3.client('ssm', region_name="us-east-1")
	dev_or_prod = os.environ['DEV_OR_PROD']

	def lambda_handler(event, context):
	    db_url = ssm.get_parameters(Names=["/my-app/" + dev_or_prod + "/db-url"])
	    print(db_url)   
	    db_password = ssm.get_parameters(Names=["/my-app/" + dev_or_prod + "/db-password"], WithDecryption=True)
	    print(db_password)
	    return "worked!"


  # Then we have to edit IAM  og Lambda to add inline policy to allow getParameters for my-app/* and add another inline policy for KMS and allow decrypt.

  # Then we go to environment variables for lambda function and add a variablecalled DEV_OR_PROD where we give value of DEV. Then this 
  function will retrieve dev values and if we give prod, then we get prod values.

  ```






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











