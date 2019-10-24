# NAS Home File Server (Samba)
 This is how I set up my Raspberry Pi as a network drive
 connected with an external drive.
 In this setup, you will create 3 folders in your drive,
 2 of witch will be private and only accessible by assigned users
 and 1 will be public that won't require authentication.
 
 _Some of the general instructions in this setup are based on 
 [this tutorial](https://medium.com/@aallan/adding-an-external-disk-to-a-raspberry-pi-and-sharing-it-over-the-network-5b321efce86a)._
 
 
 ## Pre-requirements:
 Have your Raspberry Pi working and ready for business.
 Meaning, it can be a stock installation
  but should have
   [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)
    or any other debian / ubuntu based OS
    just make sure you can connect to the SSH
 or VNC server if you use it as a headless (without monitor) device.
 
 ### Step 0 - Formatting the drive (optional)
 **BE AWARE, YOU WILL LOSE ALL YOUR DATA ON YOUR EXTERNAL DRIVE IF YOU
 FOLLOW THIS STEP**
 
 We are going to format our external drive to exFat type. (If you would like to
  use other type, feel free to google how to do that, there are plenty of info
   on that.)
 
 1. `sudo apt-get update`   
    `sudo apt-get upgrade`
    
 2. `sudo apt-get install exfat-fuse`  
    `sudo apt-get install exfat-utils`
          
 3. Connect your HDD / SSD to your Raspberry Pi
 4. `sudo umount /dev/sda1`
 5. `mkfs.exfat /dev/sdb1`
 
 Note: You can also format your drive on Windows / Linux / Mac OS the way you normally
 do.
 
 ### Step 1 - Mounting the Disk
 `sudo mkdir /mnt/usb`  
 `sudo chown -R pi:pi /mnt/usb`  
 `sudo mount /dev/sda1 /mnt/usb -o uid=pi,gid=pi`
 
Now we are going to make it mount automatically at startup:

`sudo nano /etc/fstab`

add

`/dev/sda1 /mnt/usb auto defaults,user 0 1`

save file with `ctrl + x`

 ### Step 2 - Installing SMB (Samba Server)
`sudo apt-get install samba samba-common-bin`

### Step 3 - Create new SMB users
We are going to create 2 new SMB users (user1 and user2).
Each will have their own private folder
on the drive (adjust this step based on what you want or skip if you're interested
in public directory only).

`sudo adduser --no-create-home --disabled-password --disabled-login user1`  
`sudo smbpasswd -a user1` - set password for user1  
`sudo adduser --no-create-home --disabled-password --disabled-login user2`  
`sudo smbpasswd -a user2` - set password for user2

### Step 4 - Create folder structure
`chmod 1777 /mnt/usb`

##### Public folder
`sudo mkdir /mnt/usb/public`  
`chmod 1777 /mnt/usb/public`

##### User1 folder
`sudo mkdir /mnt/usb/user1`  
`chmod 1777 /mnt/usb/user1`  
`sudo chown -R user1:users /mnt/usb/user1`

##### User2 folder
`sudo mkdir /mnt/usb/user2`  
`chmod 1777 /mnt/usb/user2`  
`sudo chown -R user2:users /mnt/usb/user2`

### Step 5 - Configure SMB
In general, you can just copy / paste the content 
of the `smb.conf` file in this GIT folder
and replace the content of `/etc/samba/smb.conf` file.

OR

Just add this at the end of your file and adjust based on your
needs:

>[public]  
  comment = public storage  
  path = /mnt/usb/public  
  create mask = 0777  
  directory mask = 0777  
  Public = yes  
  Guest ok = yes

>[user1]  
  comment = User 1 Private Folder  
  path = /mnt/usb/user1  
  browsable = yes  
  read only = no  
  guest ok = no  
  valid users = user1  
  force user = user1  
  create mask = 0777  
  directory mask = 0777  

>[user2]  
  comment = User 2 Private Folder    
  path = /mnt/usb/user2  
  browsable = yes  
  read only = no  
  guest ok = no  
  valid users = user2  
  force user = user2  
  create mask = 0777  
  directory mask = 0777  

Note that this config is for 3 folders and 2 users that we created before. 
If you only want the public folder config, then copy the [public] only.

### Step 6 - Restart SMB
`sudo /etc/init.d/samba restart`


### Conclusion

At this point, you should be done and should be able to connect
 to your external drive via phone / PC / Mac etc.

#### User1 Example Mac OS:
1. Right click on finder
2. Click `Connect to Server`
3. Enter this and click connect:
>smb://user1:*@192.168.0.123/user1

#### User1 Example Windows:
1. Go to This PC
2. Right click and select `Add a network location`
3. Click next until you need to enter URL / Address
3. Enter this and click next:
>\\192.168.0.123\user1

#### Connecting to public directory:
Same steps as above, just use
>smb://192.168.0.123/public

for Mac OS and

>\\192.168.0.123\public

for Windows

Note: change **192.168.0.123** to your Raspberry Pi IP.