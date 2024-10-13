# Project 8
We are going to spin up an ec2 instance and attach the following IAM roles to it:

- AmazonVPCFullAccess
- AmazonEC2FullAccess
 ![text](<p8.png>)
 ![text](<p8_1.png>)
 ![text](<p8_2.png>)
 ![text](<p8_3.png>)
 ![text](<p8_4.png>)

## Now create an EC2 instance, Ubuntu 22.04
let us the attach the role created earlier to the instance. And click on **Action** then, **Security group** and **Modify IAM role**
![text](<p8_5.png>)
![text](<p8_6.png>)

##  Connect to your instance via ssh 

- ssh into the instance
- install terraform
```
sudo snap install terraform --classic

```
![text](<p8_7.png>)

## Step 1:  Clone thge repo into the instance and CD in the Cloned Repository
- clone it using the following command.
```
git clone https://github.com/TobiOlajumoke/Terraform-VPC.git
```
![text](<p8_8.png>)

- Then cd in to the terraform-vpc folder
```
cd terraform-vpc
```

## Step 2
- cd vars/dev/vpc.tfvars 
- using any text editor of your choce Nano or VIM and change the #tag owner to your name eg "Andrew" in this demo i used "DevOps"

![text](<p8_9.png>)

## Step 2: Initialize Terraform and Execute the Plan



- Now cd in to infra/vpc folder and execute the terraform plan to validate the configurations.


```
cd infra/vpc
```
This command is used to change into the directory where your Terraform files for creating the VPC are located. In this case, you're moving into the infra/vpc directory.

- Firstly initialize Terraform

```
terraform init
```
This step is necessary to set up Terraform for the project.
It will download any plugins (like AWS) Terraform needs and prepare your project.
You only need to do this once when you start working with Terraform in a new directory.

![text](<p8_10.png>)

- Execute the plan

```
terraform plan -var-file=../../vars/dev/vpc.tfvars
```
The plan command shows you what changes Terraform will make to your AWS resources.
The `-var-file` is pointing to the file that contains specific values (like network ranges) for the development environment (dev).
After running this command, you’ll get an output showing what Terraform plans to create or modify (like the VPC, subnets, and gateways).

You should see an output with all the resources that will be created by terraform.
![text](<p8_11.png>)

## Step 3: Create VPC With Terraform Apply

Lets create the VPC and related resources using terraform apply.

```
terraform apply -var-file=../../vars/dev/vpc.tfvars
```
This is where you tell Terraform to actually create the resources on AWS.
It will use the values from the `vpc.tfvars` file and make the necessary changes.
You’ll see a summary of the changes before proceeding, and you have to confirm (by typing `yes`) to allow Terraform to create the VPC, subnets, internet gateways, etc.

![alt text](<p8_12.png>)
![alt text](<p8_13.png>)
![alt text](<p8_14.png>)

# Step 4: Validate VPC
Head over to the AWS Console the check the Resource Map of the VPC.

Click on the created VPC and scroll down to view the Resource Map.

You should see 15 subnets , 6 route tables, internet gateway and NAT gateway as shown below.
![alt text](<p8_15.png>)

# Step 5: Cleanup the Resources

- clean up the resources created by Terraform, execute the following command
```
terraform destroy  -var-file=../../vars/dev/vpc.tfvars
```
![alt text](<p8_16_1.png>)
![alt text](<p8_16.png>)

- You can go ahead and terminate your instance you created also

# End of Project