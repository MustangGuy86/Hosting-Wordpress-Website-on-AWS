# WordPress Website Deployment on AWS


![Architecture Diagram](2._Host_a_WordPress_Website_on_AWS.png)


## Overview

This project involves hosting a WordPress website on AWS using various services and configurations. The deployment is designed to ensure high availability, scalability, security, and reliability of the web application.

## Architecture

The architecture of the WordPress deployment on AWS consists of the following components:

1. **Virtual Private Cloud (VPC)**:
   - Configured with public and private subnets across two availability zones (AZs).

2. **Internet Gateway**:
   - Facilitates connectivity between VPC instances and the wider Internet.

3. **Security Groups**:
   - Network firewall mechanism for controlling traffic to EC2 instances.

4. **Availability Zones**:
   - Utilized for enhanced system reliability and fault tolerance.

5. **Public Subnets**:
   - Houses infrastructure components like NAT Gateway and Application Load Balancer (ALB).

6. **EC2 Instance Connect Endpoint**:
   - Enables secure connections to assets within public and private subnets.

7. **Private Subnets**:
   - Hosts web servers (EC2 instances) for enhanced security.

8. **NAT Gateway**:
   - Allows instances in private subnets to access the Internet.

9. **Website Hosting**:
   - WordPress website hosted on EC2 Instances.

10. **Load Balancing**:
    - Application Load Balancer (ALB) distributes web traffic to Auto Scaling Group across multiple AZs.

11. **Auto Scaling Group**:
    - Manages EC2 instances for availability, scalability, fault tolerance, and elasticity.

12. **Version Control**:
    - Web files stored on GitHub for version control and collaboration.

13. **Certificate Manager**:
    - Secures application communications using AWS Certificate Manager.

14. **Simple Notification Service (SNS)**:
    - Alerts about activities within Auto Scaling Group.

15. **DNS Configuration**:
    - Registers domain name and sets up DNS records using Route 53.

16. **Shared File System**:
    - Amazon EFS used for a shared file system.

17. **Database**:
    - Amazon RDS utilized for the database.


This architecture ensures a robust and scalable WordPress deployment on AWS.


## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

```bash
# create to root user
sudo su

# update the software packages on the ec2 instance 
sudo yum update -y

# create an html directory 
sudo mkdir -p /var/www/html

# environment variable
EFS_DNS_NAME=fs-0f10e779829ff7ce1.efs.us-east-1.amazonaws.com

# mount the efs to the html directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 

# download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# restart the webserver
sudo service httpd restart
```

### Script for Auto Scaling Group Launch Template

This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.


```bash
#!/bin/bash

# update the software packages on the ec2 instance 
sudo yum update -y

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# environment variable
EFS_DNS_NAME=fs-0f10e779829ff7ce1.efs.us-east-1.amazonaws.com

# mount the efs to the html directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# set permissions
chown apache:apache -R /var/www/html

# restart the webserver
sudo service httpd restart
```

## Deployment Steps

1. **Virtual Private Cloud (VPC)**:
   - Configured a VPC with public and private subnets across two availability zones (AZs).

2. **Internet Gateway**:
   - Deployed an Internet Gateway to facilitate connectivity between VPC instances and the wider Internet.

3. **Security Groups**:
   - Established Security Groups as a network firewall mechanism.

4. **Availability Zones**:
   - Leveraged two AZs for enhanced system reliability and fault

 tolerance.

5. **Public Subnets**:
   - Utilized Public Subnets for infrastructure components like NAT Gateway and Application Load Balancer (ALB).

6. **EC2 Instance Connect Endpoint**:
   - Implemented EC2 Instance Connect Endpoint for secure connections to assets within public and private subnets.

7. **Private Subnets**:
   - Positioned web servers (EC2 instances) within Private Subnets for enhanced security.

8. **NAT Gateway**:
   - Enabled instances in private subnets to access the Internet via the NAT Gateway.

9. **Website Hosting**:
    - Hosted the WordPress website on EC2 Instances.

10. **Load Balancing**:
    - Utilized Application Load Balancer (ALB) and a target group for evenly distributing web traffic to an Auto Scaling Group across multiple AZs.

11. **Auto Scaling Group**:
    - Employed an Auto Scaling Group to manage EC2 instances automatically, ensuring availability, scalability, fault tolerance, and elasticity.

12. **Version Control**:
    - Stored web files on GitHub for version control and collaboration.

13. **Certificate Manager**:
    - Secured application communications using AWS Certificate Manager.

14. **Simple Notification Service (SNS)**:
    - Configured SNS to alert about activities within the Auto Scaling Group.

15. **DNS Configuration**:
    - Registered the domain name and set up DNS records using Route 53.

16. **Shared File System**:
    - Used Amazon EFS for a shared file system.

17. **Database**:
    - Utilized Amazon RDS for the database.

## Prerequisites

- AWS account with necessary permissions.
- Basic understanding of AWS services and networking concepts.
- Access to the GitHub repository for scripts and configurations.

## Usage

1. Clone the GitHub repository.
2. Modify scripts and configurations as per your requirements.
3. Execute the scripts to deploy the WordPress website on AWS.

## Repository Structure

```
wordpress-aws-deployment/
├── scripts/
│   ├── install-wordpress.sh
│   └── launch-template.sh
├── diagrams/
│   └── architecture-diagram.png
├── README.md
└── LICENSE
```

## Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [WordPress Official Documentation](https://wordpress.org/support/)
- [GitHub Actions for CI/CD](https://docs.github.com/en/actions)

## License

This project is licensed under the [MIT License] - see the LICENSE file for details .

---
