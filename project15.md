#  SET UP A VIRTUAL PRIVATE NETWORK (VPC)

1. Create a vpc

![my-vpc](./images/my-vpc.PNG)

2. Create internet Gateway for our vpc and attach

![my-igw](./images/my-igw.PNG)

3. Create public subnets 1 and 2 on our AZ and Private subnets 1-4.

![pub-sub-01](./images/pub-sub-1.PNG)

![pub-sub-02](./images/pub-sub-2.PNG)

![priv-sub-01](./images/priv-sub-1.PNG)

![priv-sub-02](./images/priv-sub-2.PNG)

![priv-sub-03](./images/priv-sub-3.PNG)

![priv-sub-04](./images/priv-sub-4.PNG)

4. Create Public, Private Route Table, and Associate the Public and Private Subnets we created to the Private And Public Route Table Respectively.

5. Edit the public route table to allow Access From the Internet

![rtb-igw](./images/rtb-igt.PNG)

6. Create NAT gateway in order for us to associate it with our private route table,but before that, we need to allocate an elastic ip to our NAT gateway

![Elastic-ip](./images/Elastic-ip.PNG)

7. Create a NAT GATEWAY and place it on any of our public subnets

![Nat-gw](./images/Nat-gw.PNG)

8. Now edit the private route table and associate it with our NAT gateway

![priv-rtb-nat](./images/priv-rtb-nat.PNG)

9. Create security groups for

![ACS-bastion](./images/acs-bastion-se.PNG)

![Acs-datalayer](./images/acs-datalayer.PNG)

![acs-int-ALB](./images/acs-int-ALB.PNG)

![acs-ext-alb](./images/acs-ext-alb.PNG)

![Acs-nginx-reverse](./images/ACS-nginx-reverse-proxy.PNG)

![acs-webserver](./images/acs-webserver.PNG)

10. Create TLS certificates From Amazon Certificate Manager (ACM) You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

![acm-cert](./images/acm-cert.PNG)

11. Create Elastic File System (EFS)

![efs-system](./images/efs-system.PNG)

12. Create Access points for wordpress and tooling respectively

![tooling-ap](./images/tooling-ap.PNG)

![wordpress](./images/wordpress-ap.PNG)

# Setup RDS Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.
Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless

To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

To configure RDS, follow steps below:

- Create a subnet group and add 2 private subnets (data Layer)

- Create an RDS Instance for mysql 8..

- To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)

- Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
- Configure VPC and security (ensure the database is not available from the Internet)
- Configure backups and retention
- Encrypt the database using the KMS key created earlier
- Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)

1. Setup RDS subnet groups

![acs-rds-sub1](./images/acs-rds-subi.PNG)

![acs-rds-sub2](./images/acs-rds-sub2.PNG)

2. Setup Database

![acs-rds-database](./images/acs-rds-database.PNG)

![acs-rds-data](./images/acs-rds-data.PNG)

## Set Up Compute Resources for Bastion Provision the EC2 Instances for Bastion

- Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server

- Ensure that it has the following software installed

- python

- ntp

- net-tools

- vim

- wget

- telnet

- epel-release

- htop

- Associate an Elastic IP with each of the Bastion EC2 Instances

## Create an AMI out of the EC2 instance

![Nginx-ami](./images/Nginx-ami.PNG)

![Bastion-ami](./images/Bastion-ami.PNG)

![webserver-ami](./images/webserver-ami.PNG)

## Prepare Launch Template For Bastion (One per subnet)

- Make use of the AMI to set up a launch template

- Ensure the Instances are launched into a public subnet

- Assign appropriate security group

- Configure Userdata to update yum package repository and install Ansible and git

## Configure Target Groups

- Select Instances as the target type

- Ensure the protocol is TCP on port 22

- Register Bastion Instances as targets

- Ensure that health check passes for the target group

![acs-nginx-tg](./images/acs-nginx-tg.PNG)

![acs-wordpress-tg](./images/acs-wordpress-tg.PNG)

![acs-tooling](./images/Acs-tooling-tg.PNG)

## Configure Autoscaling For Bastion

- Select the right launch template

- Select the VPC

- Select both public subnets

- Enable Application Load Balancer for the AutoScalingGroup (ASG)

- Select the target group you created before

- Ensure that you have health checks for both EC2 and ALB The desired capacity is 2 Minimum capacity is 2 Maximum capacity is 4

- Set scale out if CPU utilization reaches 90%

- Ensure there is an SNS topic to send scaling notifications

## Set Up Compute Resources for Nginx Provision EC2 Instances for Nginx

- Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)

-Ensure that it has the following software installed:

- python ntp net-tools vim wget telnet epel-release htop

## Create an AMI out of the EC2 instance

- Prepare Launch Template For Nginx (One Per Subnet)

- Make use of the AMI to set up a launch template

- Ensure the Instances are launched into a public subnet

- Assign appropriate security group

- Configure Userdata to update yum package repository and install nginx

![acs-nginx-userdata](./images/acs-nginx-userdata.PNG)

![acs-reverse-conf](./images/acs-reverse-conf.PNG)

- Configure Target Groups

- Select Instances as the target type

- Ensure the protocol HTTPS on secure TLS port 443

- Ensure that the health check path is /healthstatus

- Register Nginx Instances as targets 

- Ensure that health check passes for the target group 

- Configure Autoscaling For Nginx

- Select the right launch template

- Select the VPC

- Select both public subnets

- Enable Application Load Balancer for the AutoScalingGroup (ASG)

- Select the target group you created before

- Ensure that you have health checks for both EC2 and ALB The desired capacity is 2 Minimum capacity is 2 Maximum capacity is 4

- Set scale out if CPU utilization reaches 90%
- Ensure there is an SNS topic to send scaling notifications

## Set Up Compute Resources for Webservers

### Provision the EC2 Instances for Webservers

- Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

- Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

- Ensure that it has the following software installed

- python ntp net-tools vim wget telnet epel-release htop php

- Create an AMI out of the EC2 instance

- Prepare Launch Template For Webservers (One per subnet)
- Make use of the AMI to set up a launch template

- Ensure the Instances are launched into a public subnet
- Assign appropriate security group

- Configure Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)

![acs-wordpress-userdata](./images/acs-wordpress-userdata.PNG)

![acs-tooling-userdata](./images/acs-tooling-userdata.PNG)
## CONFIGURE APPLICATION LOAD BALANCER (ALB)

### Application Load Balancer To Route Traffic To NGINX Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

- Create an Internet facing ALB

- Ensure that it listens on HTTPS protocol (TCP port 443)

- Ensure the ALB is created within the appropriate VPC | AZ | Subnets

- Choose the Certificate from ACM

- Select Security Group

- Select Nginx Instances as the target group

![Acs-ext-alb-listeners](./images/acs-ext-alb-listeners.PNG)

### Application Load Balancer To Route Traffic To Web Servers Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scaled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

Create an Internal ALB Ensure that it listens on HTTPS protocol (TCP port 443) Ensure the ALB is created within the appropriate VPC | AZ | Subnets Choose the Certificate from ACM Select Security Group Select webserver Instances as the target group Ensure that health check passes for the target group

NOTE: Instead for creating another internal load balancer for tooling target group we will add a a rule on our port 443 listeners to foward traffic host header to our tooling target group 

![acs-int-alb-listeners](./images/acs-int-alb-listeners.PNG)

### Configuring DNS with Route53

Earlier in this project you registered a free domain with Freenom and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as CNAME, alias and A records.

NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences.

Create an alias record for the root domain and direct its traffic to the ALB DNS name. Create an alias record for tooling..com and direct its traffic to the ALB DNS name.

![wordpress-dns](./images/domain-name.PNG)

![wordpress-server](./images/wordpress-server.PNG)