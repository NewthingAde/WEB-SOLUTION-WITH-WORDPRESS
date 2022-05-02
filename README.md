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
  
 - Install lvm2 package using the command below 
  
                     sudo yum install lvm2
  
 - Run the command below to check for available partitions.
  
                      sudo lvmdiskscan
                      
  <img width="382" alt="Screenshot 2022-05-02 at 13 02 55" src="https://user-images.githubusercontent.com/80678596/166224204-cb43b53e-54ae-48fb-ad11-ecf9fd96b0ba.png">

  **In RedHat/CentOS we mostly use yum package manager command**
  
 - Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

                            sudo pvcreate /dev/xvdb1
                            sudo pvcreate /dev/xvdc1
                            sudo pvcreate /dev/xvdd1
                            
 - Verify that your Physical volume has been created successfully by running 

                                sudo pvs
                                
  <img width="422" alt="Screenshot 2022-05-02 at 13 08 13" src="https://user-images.githubusercontent.com/80678596/166224756-1bb03e18-63cf-4b4d-90f3-6cd52aa11e68.png">
  
 - Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

                      sudo vgcreate webdata-vg /dev/xvdd1 /dev/xvdc1 /dev/xvdb1
                      
 - Verify that your VG has been created successfully by running 
 
                                          sudo vgs
                                          
 <img width="521" alt="Screenshot 2022-05-02 at 13 13 02" src="https://user-images.githubusercontent.com/80678596/166225295-f3f72238-58e4-4349-959a-10a87c0f9f7a.png">
 
 - Use `lvcreate` utility to create 2 logical volumes. `apps-lv` (Use half of the PV size), and `logs-lv` Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

                                      sudo lvcreate -n apps-lv -L 14G webdata-vg
                                      
                                      sudo lvcreate -n logs-lv -L 14G webdata-vg
                                      
 - Verify that your Logical Volume has been created successfully by running 
                                                  
                                                  sudo lvs
                                                  
  <img width="565" alt="Screenshot 2022-05-02 at 13 17 23" src="https://user-images.githubusercontent.com/80678596/166225639-4891cc38-e8ba-4e29-9aea-60de52b2fd01.png">

 - Verify the entire setup

                                    sudo vgdisplay -v 

                                       sudo lsblk
                                     
<img width="477" alt="Screenshot 2022-05-02 at 13 19 58" src="https://user-images.githubusercontent.com/80678596/166226230-4d2a5960-0bc4-4375-a843-b1a9cf54b352.png">

 - Use `mkfs.ext4` to format the logical volumes with ext4 filesystem

                              sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
                              
                              sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
                              
 - Create /var/www/html directory to store website files

                                sudo mkdir -p /var/www/html
                                
 - Create /home/recovery/logs to store backup of log data
                              
                              sudo mkdir -p /home/recovery/logs
                              
 - Mount /var/www/html on apps-lv logical volume

                      sudo mount /dev/webdata-vg/apps-lv /var/www/html/
                      
  - Use `rsync` utility to backup all the files in the log directory `/var/log into /home/recovery/logs` (This is required before mounting the file system)

                        sudo rsync -av /var/log/. /home/recovery/logs/

  - Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why its very important to Create /var/www/html directory to store website files)

                                    sudo mount /dev/webdata-vg/logs-lv /var/log

  - Restore log files back into /var/log directory

                                        sudo rsync -av /home/recovery/logs/. /var/log

  - Update `/etc/fstab` file so that the mount configuration will persist after restart of the server. The UUID of the device will be used to update the /etc/fstab file;

                                                      sudo blkid

  <img width="1120" alt="Screenshot 2022-05-02 at 13 37 19" src="https://user-images.githubusercontent.com/80678596/166227698-5d00a156-535d-44ab-b4cf-5cf65361144b.png">

  - oprn the /etc/fstab and update it using your own UUID and rememeber to remove the leading and ending quotes.

                                                       sudo vi /etc/fstab                                    

  <img width="759" alt="Screenshot 2022-05-02 at 13 55 54" src="https://user-images.githubusercontent.com/80678596/166229823-2d90e954-8f71-4e0f-b317-daa5ae56fcf6.png">

  - Test the configuration and reload the daemon

                                      sudo mount -a

                                sudo systemctl daemon-reload

  - Verify your setup by running the command below and the output must look like this:

                                    df -h

  <img width="574" alt="Screenshot 2022-05-02 at 13 54 50" src="https://user-images.githubusercontent.com/80678596/166229691-7ff1cb7a-ac93-4231-9599-3e5ecd98fa6a.png">


  ### Prepare a Database Server
  
  - Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
  
  - Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
  
  <img width="519" alt="Screenshot 2022-05-02 at 15 38 21" src="https://user-images.githubusercontent.com/80678596/166243186-20b7a5ba-95c9-453a-94f2-e8c2850e9774.png">

  ### Install WordPress on Web Server EC2
  
   - Update the repository
                    
                    sudo yum -y update
                    
   -Install wget, Apache and it’s dependencies
   
                  sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
   
   - Start Apache
                          
                          sudo systemctl enable httpd
                          
                          sudo systemctl start httpd

  - Install PHP and it’s depemdencies

                            sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
                            sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
                            sudo yum module list php
                            sudo yum module reset php
                            sudo yum module enable php:remi-7.4
                            sudo yum install php php-opcache php-gd php-curl php-mysqlnd
                            sudo systemctl start php-fpm
                            sudo systemctl enable php-fpm
                            setsebool -P httpd_execmem 1
                            
    - Restart Apache

                            sudo systemctl restart httpd
                            
    - Download wordpress and copy wordpress to var/www/html

                              mkdir wordpress
                              cd   wordpress
                              sudo wget http://wordpress.org/latest.tar.gz
                              sudo tar xzvf latest.tar.gz
                              sudo rm -rf latest.tar.gz
                              sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
                              sudo cp -R wordpress /var/www/html/
                              
    - Configure SELinux Policies

                              sudo chown -R apache:apache /var/www/html/wordpress
                              sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
                              sudo setsebool -P httpd_can_network_connect=1
                              
    ### Step 4 — Install MySQL on your DB Server EC2
    
      - Install mysql
                                
                                sudo yum update
                                
                          sudo yum install mysql-server
                          
      - Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
                            
                              sudo systemctl restart mysqld
                              
                              sudo systemctl enable mysqld
                                  


                              sudo mysql
                              
                              CREATE DATABASE wordpress;
                              CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
                              GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
                              FLUSH PRIVILEGES;
                              SHOW DATABASES;
                              
                              exit
                              
     ### Configure WordPress to connect to remote database
     
     **Hint:** Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32
     
      - Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
      
                                                    sudo yum install mysql
                                                    
                                sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
                                
      - Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.
      
      - Change permissions and configuration so Apache could use WordPress:
      
      - Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
      
      - Try to access from your browser the link to your WordPress 
      
                                      http://<Web-Server-Public-IP-Address>/wordpress/
