# Project 10
## Spin up an Ubuntu 24.04 EC2 instance we shall be using it for this project

Connect your instance via ssh


Clone the project repository to your instance
```
git clone https://github.com/TobiOlajumoke/prometheus-observability-stack
```
The project code is present in the prometheus-observability-stack folder. cd into the folder.
```
cd prometheus-observability-stack
```
## To do this project you need to have the following.

1. A Valid AWS account with full permissions to create and manage AWS VPC service.
2. Setup terraform on an ec2 instance, ensure you have a valid IAM role attached to the instance with VPC provisioning permissions.


We are going to spin up an ec2 instance and attach the following IAM roles to it:

- AmazonVPCFullAccess
- AmazonEC2FullAccess

 ![text](<p10.png>)
 ### Step 1: Install Terraform on the EC2 instance
 ##  Connect to your instance via ssh 

- ssh into the instance
- install terraform
```
sudo snap install terraform --classic

```

# Provision Server Using Terraform
Modify the values of ec2.tfvars file present in the terraform-aws/vars folder. You need to replace the values highlighted in bold with values relevant to your AWS account & region.


```
cd terraform-aws/vars
```
If you are using us-west-2, you can continue with the same AMI ID else change the AMI ID 
```
vi ec2.tfvars
```
 ![text](<p10a.png>)
  ![text](<p10-1.png>)

Now we can provision the AWS EC2 & Security group using Terraform.

```
cd ../prometheus-stack

```


```
terraform fmt
```

```
terraform init
```

```
terraform validate
```
![text](<p10-2.png>)
![text](<p10-3.png>)
![text](<p10-4.png>)
![text](<p10-5.png>)
![text](<p10-6.png>)

Execute the plan and apply the changes.

```
terraform plan --var-file=../vars/ec2.tfvars
```

```
terraform apply --var-file=../vars/ec2.tfvars
```

Before typing ‘yes‘ make sure the desired resources are being created. After running Terraform Output should look like the following:

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

![text](<p10-7.png>)
![text](<p10-8.png>)
Now we can connect to the AWS EC2 machine just created via ssh. 

# CONNECT TO YOUR NEW INSTANCE 

- ssh into an instance

![text](<p10-9.png>)

- let's run the following command on our instance

We will check the cloud-init logs to see if the user data script has run successfully.

```
tail /var/log/cloud-init-output.log
```
![text](<p10-10.png>)
Let’s verify the docker and docker-compose versions again.

```
sudo docker version

```
![text](<p10-11.png>)

```
sudo docker-compose version
```
![text](<p10-12.png>)

Now that we have the instance ready with the required utilities, let’s deploy the Prometheus stack using docker-compose.

# Deploy Prometheus Stack Using Docker Compose
First, clone the project code repository to the server.
```
git clone https://github.com/TobiOlajumoke/prometheus-observability-stack
```
```
cd prometheus-observability-stack
```
Execute the following make command to update server IP in prometheus config file. Because we are running the node exporter on the same server to fetch the server metrics. We also update the alert manager endpoint to the servers public IP address.
```
make all
```
![text](<p10-13.png>)
## Bring up the stack using Docker Compose. It will deploy Prometheus, Alert manager, Node exporter and Grafana

```
sudo docker-compose up -d
```

On a successful execution, you should see the following output saying Running 5/5
Now, with my servers IP address you can access all the apps on different ports.

1. Prometheus: http://my-public-ip-address:9090
2. Alert Manager: http://my-public-ip-address:9093
3. Grafana: http://my-public-ip-address:3000
4. Now the stack deployment is done. The rest of the configuration and testing will be done the using the GUI.

# Validate Prometheus Node Exporter Metrics
If you visit http://my-public-ip-address:9090, you will be able to access the Prometheus dashboard as shown below.

Validate the targets, rules and configurations as shown below. The target would be Node exporter url.

![text](<p10-14.png>)

### Validating prometheus rules and targets
Now lets execute a promQL statement to view node_cpu_seconds_total metrics scrapped from the node exporter.
```
avg by (instance,mode) (irate(node_cpu_seconds_total{mode!='idle'}[1m]))
```
You should be able to data in graph as shown below.

executing a promQL statement to get graph
![text](<p10-15.png>)
![text](<p10-16.png>)
![text](<p10-17.png>)
![text](<p10-18.png>)

# Configure Grafana Dashboards
Now lets configure Grafana dashboards for the Node Exporter metrics.

Grafana can be accessed at: http://your-ip-address:3000

Use admin as username and password to login to Grafana. You can update the password in the next window if required.

Now we need to add prometheus URL as the data source from Connections→ Add new connection→ Prometheus → Add new data source.

# Configure Node Exporter Dashboard
Grafana has many node exporter pre-built templates that will give us a ready to use dashboard for the key node exporter metrics.

To import a dashboard, go to Dashboards –> Create Dashboard –> Import Dashboard –> Type 10180 and click load –> Select Prometheus Data source –> Import

![text](<p10-19.png>)
![text](<p10-20.png>)
![text](<p10-21.png>)
![text](<p10-22.png>)
![text](<p10-23.png>)
![text](<p10-24.png>)

# Configure Alert Manager
# Simulate & Test Alert Manager Alerts
You can access the Alertmanager dashbaord on http://your-ip-address:9093
Alert rules are already backed in to the prometheus configuration through alertrules.yaml. If you go the alerts option in the prometheus menu, you will be able to see the configured alerts as shown below.
 http://your-ip-address:9090
![text](<p10-25.png>)

As you can see, all the alerts are in inactive stage. To test the alerts, we need to simulate these alerts using few linux utilities.

You can also check the alert rules using the native promtool prometheus CLI. We need to run promtool command from inside the prometheus container as shown below.
run the commands below in the prometheus server

```
sudo docker exec -it prometheus promtool check rules /etc/prometheus/alertrules.yml
```
# Test: High Storage & CPU Alert
```
dd if=/dev/zero of=testfile_16GB bs=1M count=16384; openssl speed -multi $(nproc --all) &
```

Now we can check the Alert manager UI to confirm the fired alerts.
![text](<p10-26.png>)
Now let’s rollback the changes and see the fired alerts has been resolved.

```
rm testfile_16GB && kill $(pgrep openssl)
```
![text](<p10-27.png>)

# Cleanup The Setup
To tear down the setup, execute the following terraform command from your workstation.
```
cd prometheus-observability-stack/terraform-aws/prometheus-stack
```
```

terraform destroy --var-file=../vars/ec2.tfvars
```
enter 'yes'

## Terminate the  ec2 instances

# THE END