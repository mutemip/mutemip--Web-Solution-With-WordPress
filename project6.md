## Web Solution With WordPress
Launch an EC2 instance that will serve as "Web Server"

### Step 1: Prepare web server
1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
![volumes](https://user-images.githubusercontent.com/64135078/197396327-98f83fcb-2368-4ef4-ad71-86c2a10f27b6.png)

2. Right click on each volume and click on attach to the Web Server EC2 instance.
![attach](https://user-images.githubusercontent.com/64135078/197396500-6796abce-2c66-4e53-9e3c-9c50c2448597.png)

3. Open Linux terminal to begin configerations and use `lsblk` command to inspect what block devices are attached to the server. There are some newly created devices.
All devices reside in /dev/ directory
![1](https://user-images.githubusercontent.com/64135078/197396744-a1c62e5f-8af5-4767-a5cb-905a3911e2ee.png)

4. Use `df -h` command to see all mounts and free space on your server.
