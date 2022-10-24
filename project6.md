## Web Solution With WordPress
Launch an EC2 instance that will serve as "Web Server"

## Step 1: Prepare web server
1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
![volumes](https://user-images.githubusercontent.com/64135078/197396327-98f83fcb-2368-4ef4-ad71-86c2a10f27b6.png)

2. Right click on each volume and click on attach to the Web Server EC2 instance.
![attach](https://user-images.githubusercontent.com/64135078/197396500-6796abce-2c66-4e53-9e3c-9c50c2448597.png)

3. Open Linux terminal to begin configerations and use `lsblk` command to inspect what block devices are attached to the server. There are some newly created devices.All devices reside in /dev/ directory
![1](https://user-images.githubusercontent.com/64135078/197396744-a1c62e5f-8af5-4767-a5cb-905a3911e2ee.png)

4. Use `df -h` command to see all mounts and free space on your server.
5. Use `gdisk` utility to create a single partition on each of the 3 disks `sudo gdisk /dev/xvdf`
![gdisk](https://user-images.githubusercontent.com/64135078/197397304-85c036e1-ed0a-45cc-94ad-2f28b8c20761.png)

6. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.
![2](https://user-images.githubusercontent.com/64135078/197397356-96c7d2f0-9478-4c0f-bd91-3752ebb8cb54.png)
7. Install `lvm2` package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.
![lvmds](https://user-images.githubusercontent.com/64135078/197397588-3c7735cb-7de9-471d-b348-dd35bf7bf219.png)

8. Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
```bash
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
9. Verify that your Physical volume has been created successfully by running `sudo pvs`
![pvs](https://user-images.githubusercontent.com/64135078/197417559-6d9a6204-cc71-4ba7-9598-657d31cec8ee.png)
10. Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG **webdata-vg**
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
11. Verify that your VG has been created successfully by running `sudo vgs`
![vgs](https://user-images.githubusercontent.com/64135078/197417854-3cbfa916-5bf3-4800-9a75-d0cbadaa1f68.png)
12. Use `lvcreate` utility to create 2 logical volumes. 
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
13. Verify that your Logical Volume has been created successfully by running `sudo lvs`
![lvs](https://user-images.githubusercontent.com/64135078/197418026-f586d13f-54b3-485e-a6cf-98fdf56ad666.png)

14. Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
```
![lsblk2](https://user-images.githubusercontent.com/64135078/197418124-84e2cf73-83fa-4d34-9851-5cb66c12b1b1.png)

15. Use `mkfs.ext4` to format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
16. Create **/var/www/html** directory to store website files

`sudo mkdir -p /var/www/html`
17. Create **/home/recovery/logs** to store backup of log data

`sudo mkdir -p /home/recovery/logs`
18. Mount **/var/www/html** on **apps-lv** logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`
19. Use `rsync` utility to backup all the files in the log directory **/var/log** into **/home/recovery/logs** (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`
20. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 16 above is very
important)
`sudo mount /dev/webdata-vg/logs-lv /var/log`
21. Restore log files back into **/var/log** directory
`sudo rsync -av /home/recovery/logs/. /var/log`
22. Update **/etc/fstab** file so that the mount configuration will persist after restart of the server.
  The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`
![blkid](https://user-images.githubusercontent.com/64135078/197419017-e9219bcc-75f0-41c0-b827-c539c27d402f.png)

`sudo vi /etc/fstab`
![fstab](https://user-images.githubusercontent.com/64135078/197419282-43bb1877-7a02-41be-aec9-c56549a5b80d.png)

Test the configuration and reload the daemon

 ```
 sudo mount -a
 sudo systemctl daemon-reload
 ```
 Verify your setup by running df -h, output must look like this:
 ![dfh](https://user-images.githubusercontent.com/64135078/197419421-8715f71d-f47d-4238-b226-2227d27bac94.png)

 
## Step 2: Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

## Step 3 — Install WordPress on your Web Server EC2
1. Update the repository

`sudo yum -y update`
2. Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`
3. Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```
4. To install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
5. Restart Apache

`sudo systemctl restart httpd`
6. Download wordpress and copy wordpress to **var/www/html**
```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
```
7. Configure SELinux Policies
```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```

## Step 4 — Install MySQL on your DB Server EC2

```
sudo yum update
sudo yum install mysql-server
```
Verify that the service is up and running by using `sudo systemctl status mysqld` , if it is not running, restart the service and enable it so it will be running even after reboot:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
## Step 5 — Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

## Step 6 — Configure WordPress to connect to remote database

 open MySQL port 3306
 For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32
 
1. Install MySQL client and test that you can connect from your Web Server to your DB server by using `mysql-client`
```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
2. Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.
![web-db](https://user-images.githubusercontent.com/64135078/197424137-dc5d4c23-f244-4984-8629-ff4a7158e6dc.png)
3. Change permissions and configuration so Apache could use WordPress:
4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
5. Try to access from your browser the link to your WordPress `http://<Web-Server-Public-IP-Address>/wordpress/`
![wordp](https://user-images.githubusercontent.com/64135078/197425025-8f3d8cc9-91b1-440b-b59e-2c326e25ef1c.png)

![dashboard](https://user-images.githubusercontent.com/64135078/197425103-7597732e-d4e4-4748-a481-21db5b8183b4.png)



