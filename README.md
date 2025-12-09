📘 Highly Available WordPress Deployment on AWS

This project demonstrates how to architect and deploy a highly available, fault-tolerant, and horizontally scalable WordPress application on AWS following AWS best practices.

The solution leverages multi-AZ deployments, stateless compute, shared storage, and automatic scaling to ensure the application remains operational even during failures of components, servers, or entire Availability Zones.

🏗 Architecture Overview

This architecture distributes the application across two Availability Zones to achieve high availability at every tier:
Application Tier: EC2 instances running WordPress in an Auto Scaling Group
Load Balancing: Application Load Balancer routes traffic to healthy instances
Database Tier: Amazon RDS MySQL in Multi-AZ mode
Shared Storage: Amazon EFS stores /wp-content for persistent files
Networking: Multi-AZ VPC with public & private subnets

<img width="1440" height="900" alt="vpc" src="https://github.com/user-attachments/assets/2e7fd7f1-4a2f-4741-99c2-00eaaa0de064" />
<img width="1440" height="900" alt="vpc map" src="https://github.com/user-attachments/assets/3c0abce8-3e93-46d5-9b57-e65d88d2692b" />


🚀 What This Project Demonstrates

Multi-AZ high availability for web, storage, and database tiers
Automatic scaling based on load
Stateless EC2 WordPress instances
Decoupled architecture (web tier, file storage, database all independent)
Fault tolerance during AZ outages
Fully managed data layer with RDS Multi-AZ

🧩 AWS Services Used

Compute & Scaling

Amazon EC2

Launch Template

Auto Scaling Group (ASG)

Amazon Application Load Balancer

Storage

Amazon EFS (shared file system across AZs)

Amazon RDS MySQL (Multi-AZ)

Networking

Amazon VPC

Public & Private Subnets

Route Tables

NAT Gateway

Security Groups

Monitoring & Security

Amazon CloudWatch

IAM Roles for EC2

ALB Health Checks


⚙️ Deployment Steps (High-Level)

1️⃣ Create Multi-AZ VPC
2 public subnets
2 private subnets
Internet Gateway + NAT Gateway
Route tables configured per subnet type

2️⃣ Deploy Multi-AZ Amazon RDS MySQL
Create DB subnet group
Enable Multi-AZ
Note down DB endpoint

3️⃣ Create Amazon EFS
One EFS filesystem
Mount targets in each AZ
Security group linking EC2 + EFS

4️⃣ Prepare EC2 Launch Template
User data installs WordPress + Apache/PHP and mounts EFS:
#! /bin/bash
# #!/bin/bash

DB_NAME="wordpress"
DB_USERNAME="wpadmin"
DB_PASSWORD=""
DB_HOST="wordpress-workshop.cluster-xxxxxxxxxx.eu-west-1.rds.amazonaws.com"
EFS_FS_ID="fs-xxxxxxxxx"
dnf update -y
#install wget, apache server, php and efs utils
dnf install -y httpd wget php-fpm php-mysqli php-json php amazon-efs-utils
#create wp-content mountpoint
mkdir -p /var/www/html/wp-content
mount -t efs $EFS_FS_ID:/ /var/www/html/wp-content
#install wordpress
cd /var/www
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
rm -f latest.tar.gz

#change wp-config with DB details
cp -rn wordpress/* /var/www/html/
sed -i "s/database_name_here/$DB_NAME/g" /var/www/html/wp-config.php
sed -i "s/username_here/$DB_USERNAME/g" /var/www/html/wp-config.php
sed -i "s/password_here/$DB_PASSWORD/g" /var/www/html/wp-config.php
sed -i "s/localhost/$DB_HOST/g" /var/www/html/wp-config.php

#change httpd.conf file to allowoverride

#  enable .htaccess files in Apache config using sed command
sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# create phpinfo file
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

# Recursively change OWNER of directory /var/www and all its contents
chown -R apache:apache /var/www

systemctl restart httpd
systemctl enable httpd

5️⃣ Create Auto Scaling Group
Attach launch template
Minimum = 2 instances
Spread across two AZs

6️⃣ Create Application Load Balancer
Listener on port 80
Target Group (HTTP health check)
Attach ASG as target

7️⃣ Validate WordPress Setup
Browser access via ALB DNS
Complete WordPress installation wizard

🔄 High Availability Features
✔ Multi-AZ RDS for automatic failover
✔ EFS replicated across multiple AZs
✔ Stateless Web Tier — no data stored on EC2
✔ Auto Scaling adds/removes instances automatically
✔ Application Load Balancer handles health checks & routing
✔ Multi-AZ compute distribution protects from AZ failures

<img width="1440" height="900" alt="load balancer " src="https://github.com/user-attachments/assets/5b89fa45-6b20-4bae-8132-035115a1e37d" />
<img width="1440" height="900" alt="target group" src="https://github.com/user-attachments/assets/20678604-8650-4d42-9667-ee8fa1228142" />
<img width="1440" height="900" alt="autoscaling" src="https://github.com/user-attachments/assets/42cc6360-ea4c-47d7-aea9-249ebcbd12d0" />
<img width="1440" height="900" alt="launch templatepng" src="https://github.com/user-attachments/assets/0ce1eb39-6dea-4153-9217-53b90965caa3" />
<img width="1440" height="900" alt="aurora rds" src="https://github.com/user-attachments/assets/5599543a-b45e-4386-8ac4-853f2b0b73f8" />


📚 What I Learned

How to design and deploy highly available architectures on AWS

Differences between stateless and stateful application layers

How to use ALB + ASG for automatic scaling and resilience

Using Multi-AZ RDS and EFS for data durability

Proper VPC design across multiple Availability Zones


✅ Conclusion

This project demonstrates a production-grade AWS architecture where every tier is built to handle failure gracefully and scale automatically. It follows the same design principles used by real-world enterprise and cloud-native applications.




