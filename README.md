# Terraform
## Deploying infrastructure with Terraform
## Create a new user in IAM 
### Create Access Keys 
## Creating Connection in VScode with IAM Credentials 
## Set up
1. Create a **Provider**.<p>
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      #version = "~> 5.0"
    }
  }
}

provider "aws" {
  region                   = "eu-west-2"
  shared_config_files      = ["~/.aws/conf"]
  shared_credentials_files = ["~/.aws/credentials"]
  profile                  = "myterra-vscode"
}
```

 ## Deploying a Virtual Private Network (VPC)
```
resource "aws_vpc" "mtc-vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support = true

  tags = {
    Name = "dev-vpc"
  }
}
```
To privision this resource, we will run:
```
terraform plan
```
and 
```
terrafoem apply
```
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/fb717746-378d-4fef-9b7d-97ec6b3b2f68)<p>
Our **VPC** has been created successfully. Now Let's see the terraform state and  state-backup in our resource files:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/4f1bc728-537b-4cf9-b9ce-f09f831f4a65)
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/3d23c998-0880-47f8-92f8-5c2eba4da56f)<p>

Also, let's compare this to the resource in the AWS Explorer in VScode:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/00ebb30a-07f4-4036-8061-97d19fb8b5f5)<p>
Next, we will view our state list by running the code: 
```
terraform state list
```
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/c99dfd3c-6570-405f-9fbc-ab6d0d85b06d)<p>
The code above returned one list, **aws_vpc.mtc-vpc**, the vpc we created. If we want to return all the componenets of the resource (VPC) created, We must run:
```
terraform state show aws_vpc.mtc-vpc
```
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/8b448071-2e6b-47c2-9a72-36c5b223656d)<p>
If there are numerous resources, we can list all simultanously with by running:
```
terraform show
```
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/207804d0-5ab2-4c99-9018-74c7ded80d71)<p>
This lists every resources we have provisioned. We only have one resource, hence, only one was listed. 

## Unprovisioning Resources
Terraform comes with the ability build and tear down infrastructure. Here, we will delete the VPC created with the **destroy** commanad by running:
```
terraform destroy -auto-approve
```
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/9ee3aed4-047a-4e00-bd8d-5b4e5b95435a)
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/ae61405a-add5-49ba-a496-0445d93475aa)<p>
Lets refresh the VPC panel on AWS to confirm if our **custom VPC** has been deleted.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/4938792c-1a7d-482d-9382-486f1d2e1bed)<p>
Successfully, **Terraform** has teared down the vpc. 

Since we will be needing this VPC to build other infrastructures, we will create the vpc again by running the **apply** command:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/def4870f-d5fc-4a01-a932-0a2b354f6822)<p>
Command executed without error. We will confirm from the VPC pane in the AWS management console.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/da4815d9-ed3a-46f5-9441-791589bcd7a4)<p>
Our custom vpc, **dev-vpc**, has been created as shown above with the VPC-ID, **vpc-0b0e2732a0982c0bb**. Clearly, this is a newly created VPC as compared to the first one we deleted which had the vpc-ID, vpc-04552a2b38121e57**. This is the *power** of terraform, infrastructure comes and go as needed.  

## Deploying a Subnet and Referencing Other Resources
Next, we will deploy a subnet to which we will bdeploy our Elastic Compute Cloud (EC2). Let's add another resource:
```
resource "aws_subnet" "mtc-subnet" {
    vpc_id = aws_vpc.mtc-vpc.id
    cidr_block = "10.0.1.0/24"
    map_public_ip_on_launch = true
    availability_zone = "eu-west-2a"

    tags = {
    Name = "dev-public"
  }
}
```
Next, we will run the plan command to review our infrastructure, then apply the configurations to create the **public subnet** in *AZ eu-west-2*.<p>

![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/5d66d554-07db-4c08-8729-da9ec82d72db)<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/a7613aa6-1978-4fd6-b59c-6d9c159a3ab1)<p>
Having ran the **terraform apply** command, we have created a public subnet, **dev-public**, with the cidr_block *10.0.1.0/24* in a speicified region, *eu-west-2a* as seen below:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/e44bb65f-09a4-4c52-9b84-72d3d83fcf7f)<p>

## Create Internet Gateway
Next, we will give our resources a way to the internet by building an **Internet Gataeway** with the terraform. The internet gateway is a VPC component, that allows communication between vpc and the internet. It supports both IPV4 and IPv6 traffic, hence, it enables resources such as EC2 in the public subnet to connect the internet and vice versa. To provision the gateway, let's run:
```
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.mtc-vpc.id

  tags = {
    Name = "dev-igw"
  }
}
```
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/5c3a0249-f520-4371-8657-8d1ca3cdacb9)<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/0ec25170-c3be-4995-8900-240222688c12)<p>

Terraform has successfully mounted the Internet Gateway. Now, we will create a connection between the **IGW** and the **subnet**.

## Create a Route Table
In this task, we will provision a **route table** to route traffic from the subnet to the internet gateway by running this terraform code:
```
resource "aws_route_table" "mtc_public_rtb" {
  vpc_id = aws_vpc.mtc-vpc.id 
  tags = {
    Name = "dev_public_rtb"
  }
}

resource "aws_route" "default_route" {
  route_table_id = aws_route_table.mtc_public_rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.igw.id
}
```
This time, we will have two resources to be provision:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/ace5cd06-e4aa-41b2-a447-7ab921798fda)<p>
This time let's confirm from the resources in the **AWS Explorer** pane in VScode to confirm whether the route table has been successfully been created.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/888ae217-9aee-404b-9b4f-89b5553b60de)
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/3e15e1cb-0d29-4505-9276-11baf6e403c9)<p>
From the AWS Explorer pane in VScode, it can be observed that all the resources created are found under **Resources** group under **Europe (London)** the default AZ, eu-west-2, where we deploy the infrastcure we are building. Nonetheless, let's confirm from the the AWS Management Console.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/40ad7d82-b23d-4015-9fa7-604aab025d65)<p>

### Providing Route Table Association
By providing a route table association, we aim at linking the subnet to the route table. The following code creates an association between the subnet and route table. 
```
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.mtc-subnet.id
  route_table_id = aws_route_table.mtc_public_rtb.id
}
```

In the route table page, we will select the **Subnet associations** tab and view the **route table association** we just created.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/d5d8168b-86d2-42f5-ad37-5f085e78a55f)

## Create Security Groups
Next, we will create a security group and allow outboun and inbound rules. 
```
resource "aws_security_group" "mtc_sg" {
  name        = "dev-sg"
  vpc_id      = aws_vpc.mtc-vpc.id

  ingress {
    from_port   = 0
    to_port     = 0 // we set this port to the maximum port number if you want to allow all ports
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0 // we set this port to the maximum port number if you want to allow all ports
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
The coode above created the scueirty groups successfully:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/0879725f-8e52-4bb1-8c03-06f2c42695c5)

## Configuring AMI Datasource
Before we will provision the EC2 instance, we will provide and AMI from which we want deploy. We will do this by utilising **data source**. Data source enables us to query an information from external system or an existing resorces that is added to the configuration in terraform. From then **Amazon Machine Images (AMIs), we will extract the **AMI name** and **owner** details into our datasource code as below:
```
data "aws_ami" "webserver-image" {
    most_recent = true
    owners = ["099720109477"]

    filter {
      name = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-arm64-server-*"]
    }
```
From the datasource code, the onwer is **099720109477** whiles the AMI name is **ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-arm64-server-*.** the * after the **server-** is the release date for the AMI but since we want to get the updated version, we used a wild card instead of an actual date.  When we check the **terraform.tfstate** file, we can observe that the defined instace AMI has been populated but with a different AMI and Owner's details. This is the cases because we set the most recent to true and used the wild card for an updated version of the AMI.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/fdc54c15-e74d-4d2a-b9c3-7290a51d26e9)<p>

### Create Key Pair
Next, we will create a key pair then create a terraform resources that uses the keay pair.<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/264a3b73-dd4c-477b-9a0b-359e9edefa65)<p>

We will now proceed to create the key pair with terraform:<p>
```
resource "aws_key_pair" "mtc_auth" {
  key_name = "mtckey"
  public_key = file("~/.ssh/mtckey.pub")
}
```
From the key pair page in the AWS Management console, the resource **mtckey** has been created:<p>
![image](https://github.com/JonesKwameOsei/Terraform/assets/81886509/264ec79f-216b-4d61-b927-6a714b398356)<p>







