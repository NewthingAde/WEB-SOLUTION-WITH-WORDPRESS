# WEB-SOLUTION-WITH-WORDPRESS

In this project, the tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This project consists of two parts:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is working with disks, partitions and volumes in Linux.

2. Install WordPress and connect it to a remote MySQL database server. This part of the project will deploy Web and DB tiers of Web solution.


##### Three-tier Architecture

Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

1. Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
3. Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

**In this project, i used RedHat OS**


## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.

  ### Prepare a Web Server
  
  - Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
  
  - Connect to your instance
   
  - Use `lsblk` command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. 
 
 <img width="331" alt="Screenshot 2022-05-02 at 12 45 18" src="https://user-images.githubusercontent.com/80678596/166222484-f64ddc88-da49-4718-ae6a-04489d397a65.png">

 
 - Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there – their names will likely be `xvdb, xvdc, xvdd`.

<img width="561" alt="Screenshot 2022-05-02 at 12 44 18" src="https://user-images.githubusercontent.com/80678596/166222315-93f725e8-51c2-42bc-8048-fa428c9685f0.png">

 - Use df -h command to see all mounts and free space on your server

                      df -h
                      
Use `gdisk` utility below to create a single partition on each of the 3 disks. Press `n` to add a new partition, `p` to print the partition table and `w` to write table to disk and exit. Click on yes and you would get something similar to the picture below. You will do this for all the 3 disk

                      sudo gdisk /dev/xvdb
                      
<img width="422" alt="Screenshot 2022-05-02 at 12 50 39" src="https://user-images.githubusercontent.com/80678596/166223216-c4be43db-eb60-4a78-b18e-2d21318cd612.png">
                
 - Use lsblk utility to view the newly configured partition on each of the 3 disks.

  <img width="392" alt="Screenshot 2022-05-02 at 12 54 50" src="https://user-images.githubusercontent.com/80678596/166223456-23d0db01-c284-4210-8d1c-2a92b1152116.png">
                    
