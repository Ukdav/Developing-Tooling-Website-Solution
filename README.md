# Devops-Tooling-Website-Solution

Welcome to our DevOps Tooling Website Solution, your one-stop destination for all things related to DevOps tools and solutions. 

In today's fast-paced software development landscape, DevOps practices and tools are essential for streamlining the development, testing, and deployment processes. 

Our website is designed to provide you with valuable insights, resources, and information about the latest DevOps tools and best practices. 

Whether you're a seasoned DevOps professional or just starting on your DevOps journey, we aim to empower you with the knowledge and tools needed to succeed in your continuous integration, continuous delivery, and infrastructure automation endeavors. 

In this project, I will be introducing the concept of file sharing for multiple servers to share the same web content and also a database for storing data related to the website.

## Architectural Diagram

![diagram](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/98bdb2c6-ad05-408a-b565-785e8da43eaf)

In the diagram above we can see a common pattern where several stateless Web Servers share a common database and access the same files using a Network File system (NFS) as shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it looks like a local file system from where they can serve the same files.

This project consists of the following servers:

* Web server(RHEL)
* Database server(Ubuntu + MySQL)
* Storage/File server(RHEL + NFS server)

## Preparing NFS Server

Create an EC2 instance (Red Hat Enterprise Linux 8 on AWS) on which we will set up our NFS(Network File Storage) Server.

On this server, we attach 2 EBS volumes 10GB each as external storage to our instance and create 3 logical volumes on it through which we will attach mounts from our external web servers.

* 3 logical volumes *lv-opt*, *lv-apps* and *lv-logs*
* 3 mounts directory */mnt/opt*, */mnt/apps* and */mnt/logs*
* Webserver content will be stored in /apps, webserver logs in /logs, and /opt will be used by Jenkins

![lsblk](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/fc7aa4c9-289f-44b8-abc7-956f87988219)

Steps taken to create logical volumes are shown in the last project

Install NFS Server, configure it to start on reboot, and make sure it is up and running

* sudo yum -y update
* sudo yum install nfs-utils -y
* sudo systemctl start nfs-server.service
* sudo systemctl enable nfs-server.service
* sudo systemctl status nfs-server.service

![sudo yum update on nfs server](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/318adcf9-e6cc-4520-a5b7-3dcea8fbbba9)

![sudo yum install nfs-utils -y](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/1158b085-ee38-4140-81d4-6201770ef535)

![sudo systemctl status nfs-server service](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/888b707d-7252-468e-adb1-197f88bb0e3a)

Note: In this project, we will be creating our NFS server, web-servers and database server all in the same subnet

Next, we configure NFS to interact with clients present in the same subnet.

We can find the subnet ID and CIDR in the Networking tab of our instances

![subnet cidr](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/beaa9bc6-6259-4b81-8d01-1299e6f3612e)

* sudo vi /etc/exports

On the vim editor add the lines as seen in the image below

* sudo exportfs -arv

Also to check what port is used by NFS so we can open it in security group

![sudo vi and sudo exporttfs](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/b4d28001-1e58-42f8-8c8f-839f6b1df3ec)

The following ports are to be open on the NFS server

![nfs server](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/7d43a36a-5eb2-43c0-85db-8659e72427e7)

## Preparing Database Server

Create an Ubuntu Server on AWS which will serve as our Database. Ensure its in the same subnet as the NFS-Server.

*Install mysql-server*

*sudo apt -y update*

*sudo apt install -y mysql-server*

To enter the db environment run

*sudo mysql*

* Create a database and name it tooling
* Create a database user and name it webaccess
* Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![create databse on mysql ubuntu server](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/9963b98b-3898-4b6b-87ee-2ce470120646)

## Preparing Web Servers

Create a RHEL EC2 instance on AWS which serves as our web server. Also, remember to have it in the same subnet

A couple of configurations will be done on the web servers:

* configuring NFS client
* deploying tooling website application
* configure servers to work with database
  
**Installing NFS-Client**

![nfs client installation](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/e2be46fc-65c1-4bde-aeda-5c5f7ad9fa4d)

We will be connecting our /var/www directory to our web server with the */mnt/apps* on NFS server. This is achieved by mounting the NFS server directory to the webserver directory.

We then need to ensure that our mounts remain intact when the server reboots. This is achieved by configuring the fstab directory.
*sudo vi /etc/fstab*

Add the following line *172.31.18.60:/mnt/apps /var/www nfs defaults 0 0*

![webserver1 sudo mkdir varwww](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/91602781-0b1a-4cf4-a3e2-d27112af1b81)
![webserver 1 mounting logs](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/56c990fa-8bbb-48ca-8c1e-c86bd9cc0988)
![webserver  1 vi config for mount](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/55105d14-1ef2-4b8d-80f6-25a146edc2d9)

## Installing Apache and Php

* sudo yum install httpd -y

* sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

* sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

* sudo dnf module reset php

* sudo dnf module enable php:remi-7.4

* sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

* sudo systemctl start php-fpm

* sudo systemctl enable php-fpm

* sudo setsebool -P httpd_execmem 1

We can see that both /var/www and /mnt/apps contain the same content. This shows that both mount points are connected via NFS.

![webserver 1 create new file](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/b7696cf1-bba8-4885-bce1-677477e6abe8)
![nfs server confirming if the created file is existing](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/1f2ebb69-866d-43c4-a3fe-5187bff12fc2)

On the NFS Server, add web content into the */mnt/apps* directory. This should contain a HTML folder. The same content will be present in the */var/www* directory in the web server. Also, fork the tooling source code from Darey.io Github Account to your Github Account.

Run <public_ip_address>/index.php on a web browser to access the site. Use the public_ip_address of the web server. TCP port 80 should be open on the web browser.

![webserver 1 url login interface](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/ef119d31-beda-4f30-9d70-c6341cb5ca88)
![webserver 1 tooling website](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/c33e15a7-f970-4e56-b10c-ae6d71e82980)

In the */var/www/html* directory , edit the already written PHP script to connect to the database sudo vi /var/www/html/functions.php.

![weberser1 cp into tooling](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/64280946-ec21-4ec9-b2bc-edd1e1cede64)
![webserver config edit connection to db server](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/47fe1e3c-c020-4fbf-bc20-f2ca5b7654d4)

After the modification, connect to the database server from the web server *mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql*

![webserver mysql tooling config](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/da5ad768-a1b2-42d2-a221-365755ff9366)

![show user table](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/5e2ed6e7-d3d4-4916-a2e7-20c4b17615ed)

Simulate a sign-up process by manually adding user credentials to the database

![dbserver insert into](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/10033db4-5fc4-4112-871e-9fcc2c4f57b8)

Login by running the following on the browser <public_ip_address>/login.php.

![web logged in](https://github.com/Ukdav/Developing-Tooling-Website-Solution/assets/139593350/9db1059f-2ce5-4c36-be3c-f27ea7de2784)

In summary, DevOps tooling for a website solution aims to promote collaboration, automation, and best practices throughout the development lifecycle. It accelerates development, enhances deployment reliability, and improves the overall quality and performance of web applications. 

Successful DevOps implementation involves selecting the right tools and tailoring them to the specific needs of your website solution.





















































