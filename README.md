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















