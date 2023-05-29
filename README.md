# web401
Objective:

	We will be building a fresh Ubuntu Server 20.04 virtual machine, configuring a RAID 1 array, and creating a dockerized instance of SAMBA (SMB file share). These are all great tools that you could use in your future jobs and career and you should know.

NOTE: this Assignment is dealing with commands that alter the virtual hardware attached to the virtual machine and if you mistype the command there is no taking it back. I suggest taking your time and verify after each step.
 Part 1: Creating a new VM

Download the Ubuntu Server ISO to your computer here
Open the Virtual Box Manager 
You can safely delete your old virtual machines for this course
Once the Ubuntu Server ISO is downloaded Click the New button
Under Name and operating system 
Name: web401-assignment_4
Machine Folder: leave as default
Type: Linux
Version: Ubuntu (64-bit)
Click Continue 
Under Memory size
Leave as the default 1024 MB
Under Hard disk
Select Create a virtual hard disk now
Click Create
Under Hard Disk File Type
Select VDI (VirtualBox Disk Image)
Click Continue
Under Storage on physical hard disk
Select Dynamically allocated
Click Continue
Under File location and size
Leave the default at 10.00 GB
Click Create
Once the VM is created but before it's started, select web401-assignment_4 from the list and click Settings
Under the System tab click the Processor sub-tab
Processor(s): 2
Under the Storage tab
Select the Empty disk under Controller: IDE
Click the disk to the right of IDE Secondary Device 0
A small popover menu will appear, select Choose a disk file... and navigate the file system to find the Ubuntu ISO you've downloaded earlier
Under the left hand column, select Controller: SATA
Click the Hard Drive with a Green PLUS
A menu will pop up, click Create
Under Hard Disk File Type
Select VDI (VirtualBox Disk Image)
Click Continue
Under Storage on physical hard disk
Select Dynamically allocated
Click Continue
Under File location and size
Leave the default at 1.00 GB
Click Create
Under the Not Attached category of disks, click to highly the newly created one
Then Click Choose
Repeat this process 1 more time till you have 3 hard drives attached to your virtual machine
Once completed, click OK
Under the Network tab
Make sure the network adapter is attached to NAT
Click Advanced
Click the Port Forwarding button
In the popup click the Green Plus
Name: SSH
Protocol: TCP
Host IP: blank
Host Port: 2222
Guest IP: blank
Guest Port: 22
Click OK
Once all the settings are set, click OK

Part 2: Installing the Operating System

Select web401-assignment_4 from the list in the Virtual Box Manager
Click the Green Start Button
Once the system has booted into the install menu for Ubuntu use the following Settings
Choose English as your language and hit <enter>
If prompted to update the installer, select Continue without updating and hit <enter>
Under Keyboard Configuration
Make sure the Layout and Variant are set to English (US) and select Done and hit <enter>
Under Network Connections
Leave at the defaults and select Done and hit <enter>
Under Configure Proxy
Leave at the defaults and select Done and hit <enter>
Under Mirror Address
Leave at the defaults and select Done and hit <enter>
Under Guided storage configuration
Make sure Use an entire disk has a (X)
Hit <tab> a few times to move to the bottom
Select Done and hit <enter>
Under Storage configuration
Leave at the defaults and select Done and hit <enter>
When the scary Confirm destructive action popup appears, know that it won't destroy your computer because it's isolated in a VM and select Continue and hit <enter>
Under Profile Setup
Your name: web401
Your Server's name: web401-assignment-4
Pick a username: student
Choose a password: student
Confirm your password: student
Then select Done and hit <enter>
Under SSH Setup
Hit <spacebar> to toggle on Install OpenSSH Server
Hit <tab> till you're at the bottom
Select Done and hit <enter>
Under Featured Server Snaps
Leave at the defaults and select Done and hit <enter>
Now we're installing the Operating System, this may take a few minutes
Once everything is done installing, select Reboot Now and hit <enter>
It should prompt you to hit <enter> to eject the installation medium, do so

Part 3: Configuring MDADM Raid 1

Log into the server
Install MDADM
$ sudo apt-get install -y mdadm nmon htop iotop
Create the partitions for SDB
Let's check that the drives are attached correctly by running a command
$ ls -la /dev/ | grep sdb
You should see output, if blank then you haven't attached the drive correctly to your VM. Review Part 1 and try again
$ sudo fdisk /dev/sdb
Now FDISK is very old and spooky so let's be careful here and after each command you need to hit <enter>
<n> for a new partition
<p> for a primary partition
<1> for the first partition or just hit <enter> for the default
<enter> for the default of 2048
<enter> for the default for the last sector in the drive
<w> to write the changes
Create the partitions for SDC
Let's check that the drives are attached correctly by running a command
$ ls -la /dev/ | grep sdc
You should see output, if blank then you haven't attached the drive correctly to your VM. Review Part 1 and try again
$ sudo fdisk /dev/sdc
Same as before, FDISK is very old and spooky so let's be careful here and after each command you need to hit <enter>
<n> for a new partition
<p> for a primary partition
<1> for the first partition or just hit <enter> for the default
<enter> for the default of 2048
<enter> for the default for the last sector in the drive
<w> to write the changes
Now we are going to create our RAID 1 drives, be very CAREFUL as this is destructive
$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
When prompted type: yes
Let's see how the syncing of the block devices are going
$ sudo mdadm --detail /dev/md0
The state should be clean
Once the RAID 1 is finished building let's create the filesystem and mount it on boot
Create the mount filepath
$ sudo mkdir -p /media/file_server
To create the ext4 file system
$ sudo mkfs.ext4 /dev/md0
To autostart the RAID based on superblocks
$ sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
To verify MDADM loads in on boot
$ sudo update-initramfs -u
To mount this to a path in the filesystem on boot
$ echo '/dev/md0 /media/file_server ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
Now let's mount up
$ sudo mount -a
We now have a mounted RAID 1 pair of drives in our server
To verify: $ df -h and look for our mounting path of /media/file_server


Part 4: Setting up docker

Install docker
Update the repo listings and remove docker if already installed
$ sudo apt-get update
$ sudo apt-get remove -y docker
Now we need to install some base packages and input the docker GPG signing key
$ sudo apt-get install -y ca-certificates curl gnupg lsb-release
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null 
Now that that we have the correct GPG key and the repository in our sources we need to refresh our lists and install docker
$ sudo apt-get update
$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose 
If you don’t allow your user into the docker group it will require you to use sudo to do all docker commands which is suboptimal
$ sudo usermod -aG docker $(whoami)
$ sudo chmod 666 /var/run/docker.sock 
Now we start the docker service and enable it to run on boo
$ sudo systemctl start docker
$ sudo systemctl enable docker
We will now build a docker compose image to run
$ mkdir -p ~/docker/samba
$ cd ~/docker/samba
$ touch docker-compose.yml
$ vim docker-compose.yml
The contents of this file can be found here
A simpler method would be to wget the file down like so:
$ wget -O docker-compose.yml https://gist.githubusercontent.com/araymer-stclair/c8dbd3d936c309c1524b4da373246fb5/raw/3a4ef5201fd4bacee9dc0c04d624dff368164d0d/docker-compose.yml
Let's start up the service and see what happens
$ cd ~/docker/samba
$ docker-compose pull
$ docker-compose up --build -d
To verify it's running run: $ docker ps

Part 5: Submission

We need to make sure that the samba service is running, the raid is correctly configured and error free, and SAMBA has started without issues
Using the following commands, take a screenshot with command output visible
$ sudo echo “hello”; clear; sudo mdadm --detail /dev/md0; docker ps; df -h
$ echo “{{YOUR NAME}}”
$ sudo cat /etc/machine-id
Name the screenshot {{YOUR NAME}} - Assignment 4.png and submit it to blackboard under the Assignment 4 assignment
0 will be given if the name of the image is incorrect
Shut down the VM and have a great day!
