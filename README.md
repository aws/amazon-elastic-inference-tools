## Amazon Elastic Inference Tools

Amazon Elastic Inference tools and utilities.

## License

This library is licensed under the Apache 2.0 License. 
Amazon Elastic Inference Setup Tool for EC2 

## Amazon Elastic Inference Setup Tool for EC2 

Authors: Shaikh Eftiquar, SW Dev Engineer, EI; Satadal Bhattacharjee, Principal Product Manager, EI

The Amazon Elastic Inference (EI) setup tool is a python script that enables you to quickly get started with EI

Amazon Elastic Inference allows you to attach low-cost GPU-powered acceleration to Amazon EC2 and Amazon SageMaker instances to reduce the cost of running deep learning inference by up to 75%. But, if you are using EI for the first time, setting up your environment to launch instances to EI can be onerous. You'll need to setup a number of dependencies including Amazon Web Services (AWS) PrivateLink VPC Endpoints, IAM policies, and security group rules before you can use EI accelerator. The EI setup tool makes it easy for you to get started by creating the necessary resources that will help you launch EI accelerators in minutes.

##### At a high level, the script does the following:

1. It creates an IAM role for the instance with an IAM policy that lets you connect to the AWS Elastic Inference service. 
2. It creates a security group with the necessary ingress and egress rules to allow the instance to communicate with the accelerator.
3. It creates an AWS PrivateLink VPC Endpoint within your desired subnet.
4. Launches the desired EC2 instance and EI accelerator with the AWS Deep Learning AMI (https://docs.aws.amazon.com/dlami/latest/devguide/what-is-dlami.html) (DLAMI) of your choice.

##### How to use it:

The script has a dependency on the following software:

1. Python 3 installed on your local machine where you expect to run the tool. 
2. The AWS SDK for Python (Boto3 (https://boto3.amazonaws.com/v1/documentation/api/latest/index.html))
3. A VPC in the region where you are launching the instance (could be your default VPC)
4. Subnet where you'd like to launch the instance 
5. EC2 Key Pair

You can then download theamazonei_setup.py (https://github.com/aws/amazon-elastic-inference-tools/blob/master/amazonei_setup.py) script to your local machine and run it from your terminal using the cmd:

	$ python amazonei_setup.py

##### What the tool creates on your behalf:

The script creates following AWS resources: 

##### Instance role with an Amazon EI Policy:
This role is created the first time the script is run. In all subsequent runs, script reuses this IAM role. If this role is deleted, script will recreate the role next time it is run. The IAM role has following properties :   

	Role Name : "Amazon-Elastic-Inference-Connect-Role"
    Policy Name : "Amazon-Elastic-Inference-Connect-Policy"
    Instance Profile Name :"Amazon-Elastic-Inference-Instance-Profile"

The Policy description is as follows :

	{ "Version": "2012-10-17", 
	  "Statement": [
       {
            "Effect": "Allow", 
            "Action": [ 
            "elastic-inference:Connect", 
            "iam:List*",
            "iam:Get*",
            "ec2:Describe*",
            "ec2:Get*" 
            ],
            "Resource": "*"
        } 
    ] 
	}


##### Security Group(SG): 

The security group associated with EC2 instance needs to allow inbound traffic to port 443 as required by Amazon EI service. We also need inbound rule to allow traffic to port 22 for SSH. If a security group matching these rules is found it is used. However if no matching SG is found, a new SG with required rules is created. The outbound rules are set to allow traffic to all ports. The new SG name is “*amazon_ei_security_group”* with the description “*Security Group for accessing Amazon EI service”*

* *Interface VPC endpoint (AWS Private Link) :* The script scans for existing endpoint associated with Amazon EI service for the region and VPC chosen by user. For example, for the us-west-2 region, the script looks for  the endpoint with name “*com.amazonaws.us-west-2.elastic-inference.runtime”* in the given VPC ID.  If the endpoint is not found, the script will create one. Also the script sets following attributes of the VPC endpoint to true, as required by Amazon EI: 
    * EnableDnsSupport
    * EnableDnsHostNames
    * The script will modify the endpoint and add SG and chosen subnet if they are missing from the discovered endpoint 
* The script discovers latest AWS DLAMI based on the operating system chosen by the user 
* If all steps succeed, the script launches an instance and reports the instance ID  
* The script tries to obtain pubic DNS name after the instance is launched and is in running state 
* Even if the instance is running, it may not be ready for accepting SSH connection and users may want to wait until the instance is fully initialized. EC2 console or AWS CLI can be used to query the initialization state, using the instance ID that is reported by the script for the newly launched instance.   

*What to expect when you run the tool*

* The script can be launched from the command prompt as:
 
		$ python amazonei_setup.py --region us-west-2 --instance-type m5.xlarge

The script needs AWS credentials to create or modify AWS resources. It uses Boto3, AWS SDK for Python. In order to be able to configure and manage AWS resources, the script needs user credentials. If the script is run without appropriate credentials, it will report the error below: 

	$ python amazonei_setup.py --region us-west-2 --instance-type m5.xlarge
	Error setting up Amazon EI configuration - 
	Failed to retrieve VPC endpoints for us-west-2 : An error occurred (RequestExpired)
	when calling the DescribeVpcEndpointServices operation: Request has expired.


The solution is to configure AWS credentials using one of the methods described here (https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html). Once the credentials are in place, the script is able to proceed. 

* *Choosing the Operating System:* The script prints informative message and prompts for choosing the OS. It also informs that entering 'q' will cause the script to exit. Choosing '1' takes us to next step. 

		$ python amazonei_setup.py --region us-west-2 --instance-type m5.xlarge

This script launches Amazon EC2 instances with Amazon Elastic Inference accelerators.
Performs the following functions:
 1. It uses the Deep Learning AMIs preconfigured with EI-enabled deep learning 
 frameworks to launch the instances.
 2. It creates security groups for the instance and VPC endpoint.
 3. It creates the VPC endpoint needed for your instances to communicate with EI 
 accelerators.
 4. It creates an IAM Instance Role and Policy with the permissions needed to 
 connect to accelerators.

 To begin, please choose the Operating System for your instance by typing its index :

	 0: Amazon Linux
	 1: Ubuntu

	Type 'q' to quit.
	amazonei-wizard>

* *Choosing Accelerator size:* The script discovered latest DL AMI for Ubuntu, it also discovered one keypair. If it discovers multiple keypairs, it will list those and ask the user to choose desired keypair by typing its index.  In general if there are multiple eligible inputs, the script will list them as indexed list and let user choose an item by typing its index. Thus, script lists supported accelerator sizes and lets user choose. 

		amazonei-wizard>1 
		 Using Image ID: ami-0027dfad6168539c7,Image Name: Deep Learning AMI (Ubuntu) Version 21.2
		 Using instance type: m5.xlarge
		 Using Key Pair: Efti-Default-KeyPair

		Please type index of the accelerator type to use:

		 0: eia1.medium (1 GB of accelerator memory)
		 1: eia1.large (2 GB of accelerator memory)
		 2: eia1.xlarge (4 GB of accelerator memory)

		Type 'q' to quit.
		amazonei-wizard>

* *Choosing VPC :* 
 
 As illustrated, user chose option '1' for Accelerator size and the script confirmed the Accelerator size chosen and proceeded to discover IAM role.Subsequently, it presents list of available VPCs.   

	amazonei-wizard> 1 
 	Using Amazon EI accelerator type: eia1.large

 	Found an IAM role configured for connecting to Amazon EI service. Name - Amazon-Elastic-Inference-Connect-Role, ARN - arn:aws:iam::326228132093:role/Amazon-Elastic-Inference-Connect-Role

	Please select the VPC to use by typing the desired VPC index. Type 0 for default VPC.

	 0: VPC Id 'vpc-d7d218af'
	 1: VPC Id 'vpc-0c2496c51925ff1be'

	Type 'q' to quit.
	amazonei-wizard>

* *Proceeding to launch instance :* 

Once user chooses the VPC ID, the script found a security group with matching inbound rules associated with chosen VPC, it also found one subnet associated with the chosen VPC ID. Additionally it found VPC endpoint for Amazon EI service. As the script has all the details it needs to launch an EC2 instance, the script summarizes all the parameters it will use to launch the instance. 

	amazonei-wizard>1 
 	Using VPC ID: vpc-0c2496c51925ff1be
 	Using Security Group: sg-00aec97685affb306
 	Using Subnet: subnet-04881d24764d6e73f

 	Discovered VPC endpoint for Amazon EI service, ID: vpce-0d2942a8147305240

 	The script will now launch new instance with following configuration. Type 'y' to continue. 

 	Accelerator Type: eia1.large
 	Region: us-west-2
 	Image-ID: ami-0027dfad6168539c7 - (Deep Learning AMI (Ubuntu) Version 21.2)
 	Instance Type: m5.xlarge
 	Key Pair: Efti-Default-KeyPair
 	Security Group ID: sg-00aec97685affb306
 	Subnet ID: subnet-04881d24764d6e73f
 	Instance Profile: Amazon-Elastic-Inference-Instance-Profile

	Type 'y' to continue. Type 'q' to quit.
	amazonei-wizard>

* *Launch and wait for the instance to reach running state:*

As the user typed 'y', the script proceeded to launch the instance. Please note that the script also printed _*probable*_ SSH command. The script infers the SSH command based on the OS type, key pair chosen and the pubic DNS name. Actual command will differ based on location of pem file. The script also warns that the instance may not be immediately accessible via SSH, even though it is in running state. The instance needs to be initialized fully, specifically  the SSH daemon needs to be started before it can accept SSH connections. If the pem file is correctly located the user should be able to access the instance and proceed with using Amazon Elastic Inference. 

	amazonei-wizard>y

 	Launching Instance ..

	 Launched instance successfully. The instance ID is 'i-0969820364c038cca'.

	 Waiting for instance to reach running state ...

	 You can use the following sample SSH command to connect to your instance: ssh -i "Efti-Default-KeyPair.pem" ubuntu@ec2-52-13-194-188.us-west-2.compute.amazonaws.com


	 Note: Please wait until instance is fully initialized and ready to accept SSH connections. You may check instance status at EC2 console.
	 Also please locate your private key file 'Efti-Default-KeyPair.pem'.

	8c85903f96de:amazon-elastic-inference-tools $ 




