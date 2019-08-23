---
title: PXE & Booting
tags: PXE & Booting , partitions
description: Breif details on PXE & Booting , partitions .
---

#  PXE Network Boot Server on Ubuntu 18.04 LTS and booting with a PXE client
## We start with PXE boot ubuntu server
##### Requirements
- We will use VirtualBox which will provide us with the virtual environement for this configurations.
- We will create a virtual machine and install ubuntu 18.04 LTS on it (which will act as our PXE server)
- We need to configure TFTP and DHCP server.For simplicity reasons, I decided to use DNSmasq which will act as a DHCP server and TFTP server of the network.
- We will use PXElinux for the network boot program (NBP).
- We will also create a virtual container without an OS to test PXE network boot.

---

### 1. Creating and installing Ubuntu 18.04 virtual machine on VirtualBox


- For this task, I used this video tutorial which helped me go through the installation. It is very simple and anyone following it will get to install both the virtual machine and the container without any hurdles.

    {%youtube QbmRXJJKsvs %} 
    *Video on creation and installing VMs using virtualBox on Ubuntu 18.04 LTS.
    source : https://youtu.be/QbmRXJJKsvs***
 - After following the video, we will have something similar to this (see screenshot below). 
 - The two VMS are Ubtuntu for PXE and test. As seen below:
 - ![](https://i.imgur.com/pLTqm0a.png)
 - The Ubuntu for PXE OS when running will look like so:

 ![](https://i.imgur.com/qPwapSr.png)
 
 
 
---

### 2. Installing and configuring DNSmasq (to act as TFTP server + DHCP server ) 
- One way was to configure PXE boot server with a fixed IP address using Netplan .
<style>
code.blue {
  color: #337AB7 !important;
}
code.orange {
  color: #F7A004 !important;
}
</style>
- To do this, using the terminal, we type the following command to conigure the yaml file.
-  <code class="blue">sudo nano /etc/netplan/01-cloud-init.yaml</code>
- Note that the yaml file varies so we will first enter in the /etc/netplan directory to edit our own yaml file.
- The resultant config will look like so : 
 ![](https://i.imgur.com/tTpMvuD.png)
- From the netplan file, we see enp0s3 and enp0s8.These are interfaces and vary per machine. To know ours, we type either "ifconfig -a" or "ip add". The resultant picture shows the interfaces that we will use in our yaml file as above.

- ![](https://i.imgur.com/b8b643W.png)

- But I realised there is this better technique less stressful and easy.This is what we are going to use for this lab.
- This is done by creating a NAT network between the PXE server and connect the client PXE PC to this server via the same NAT network of the server.
- On VirtualBox, we go to file, preference and then network.We edit the addresses and enable DHCP. See picture below.
![](https://i.imgur.com/2A1m1IE.png)

- On our PXE server, we go to settings, network and select adapter NAT network as seen below.
![](https://i.imgur.com/ZrSmytf.png)


<style>
code.blue {
  color: #337AB7 !important;
}
code.orange {
  color: #F7A004 !important;
}
</style>

- Then we install DNSmasq using this command :
- <code class="orange">sudo apt update && sudo apt install -y dnsmasq</code>
![](https://i.imgur.com/4FLWnXb.png)
- After running this command, the first time, sometimes we can have errors, but these can be solved by "apt-get update && apt-get upgrade && apt-get dist-upgrade"
- DNSmasq is installed.
![](https://i.imgur.com/fkbEHCw.png)


- Now, we rename our /etc/dnsmasq.conf file to /etc/dnsmasq.conf.backup.This is done using the following command
![](https://i.imgur.com/n5WYlEb.png)
- Then we create a new /etc/dnsmasq.conf file and configure it as follows:
![](https://i.imgur.com/axJwa66.png)
- We congifure as follows :
![](https://i.imgur.com/ZVsFkL7.png)
- The next thing is to create the TFTP root directory in /tftpboot/tftp that will host our pxelinux.0 boot program.
![](https://i.imgur.com/MDsIMHS.png)
- Then we restart our DNSmasq and see its status using systemctl command.
- ![](https://i.imgur.com/Ac7RuNv.png)

---

### 3. Lets install and configure NFS server

- The Network File System (NFS)  is a distributed file system protocol which allows a server to share directories and files with clients over a network. For more details follow this link.(https://vitux.com/install-nfs-server-and-client-on-ubuntu/).
- We use the following command to install and we can update using "apt-get update"
- ![](https://i.imgur.com/xBfE5Yr.png)
- We then create the NFS directory inside the TFTPBoot directory we previously created.
- ![](https://i.imgur.com/AQmdmDx.png)
- We will open and configure the exports config file as follows which defines the way pxelinux.0 is used.
- We do this by adding this line <code class= "orange">/netboot/nfs(ro,sync,no_wdelay,insecure_locks,no_root_squash,insecure,no_subtree_check)</code> at the end of the export config file as seen below. 
![](https://i.imgur.com/6LFEGlg.png)
![](https://i.imgur.com/Xrb0FH8.png)
- We make it available to PXE clients via the following command : 
![](https://i.imgur.com/jiA5BIO.png)

### 4. Let's install the required PXE boot program (PXElinux)
- We isntall pxelinux via the following : 
![](https://i.imgur.com/Rl1Co7S.png)
- We will now transfer the pxelinux.0 boot program to the tftp root directory so that it can be visible to the PXE clients to boot via it.
- We use the following commands : 
![](https://i.imgur.com/vLR3IEX.png)

- We also copy the ldlinux.c32, libcom32.c32, libutil.c32, vesamenu.c32 files to the tftp directory using the command: 
<code class="orange">sudo cp -v /usr/lib/syslinux/modules/bios/{ldlinux.c32,
libcom32.c32,libutil.c32,
vesamenu.c32} /netboot/tftp</code>
![](https://i.imgur.com/cZXUELz.png)
- next, we create the directory pxelinux.cfg using the command :
<code class="blue">
sudo mkdir /tftpboot/tftp/pxelinux.cfg
</code> 
- We then create the PXE bootloader’s default configuration file /tftpboot/tftp/pxelinux.cfg/default using the command : 
<code class="blue">
sudo touch /tftpboot/tftp/pxelinux.cfg/default
</code>
- ### Voila !
- TFTP server is now able to serve all the required bootloader files over the network.
---


### 5. Preparing Ubuntu 18.04 LTS for PXE Boot:
- Let's first download the iso image from the ubuntu repo via the command : 
<code>
wget http://releases.ubuntu.com/18.04/ubuntu-18.04.3-desktop-amd64.iso
</code>
- We have downloaded the iso image to our local drive : 
![](https://i.imgur.com/pkmkPb7.png)
- Let's mount the ISO file on the /mnt directory.
![](https://i.imgur.com/77Srrd0.png)
- Let's now create directories for Ubuntu 18.04 LTS /tftpboot/nfs/ubuntu1804/ and /tftpboot/tftp/ubuntu1804/ using the following comand : 
![](https://i.imgur.com/AcnER2U.png)
- this creates 2 directories in nfs and tftp folders respectively.
![](https://i.imgur.com/juobzTt.png)


- We copy the contents of the ubuntu ISO file to the NFS directory /tftpboot/nfs/ubuntu1804/
![](https://i.imgur.com/mKHLU7z.png)
- We then copy the vmlinuz and initrd files to the /tftpboot/tftp/ubuntu1804/ directory via the command below:
![](https://i.imgur.com/kStZcld.png)
- Let's finalize this by giving some better permissions to our tftpboot directory.
![](https://i.imgur.com/9lURUEV.png)
- Perfect !
- Let's unmount the Ubuntu 18.04 LTS ISO image and delete it as we don't need it anymore.
![](https://i.imgur.com/pZbVFxg.png)


---


### 6. Final stage is Adding PXE Boot Entry for Ubuntu 18.04 LTS:
- W need to edit the PXE boot menu configuration file  found in /tftpboot/tftp/pxelinux.cfg/default and add the boot entry for ubuntu 1804 LTS.

- Open the PXE boot menu configuration file /netboot/tftp/pxelinux.cfg/default for editing as follows:
![](https://i.imgur.com/RhhT5Ql.png)
- We now add the entry in the default file as seen below.
![](https://i.imgur.com/lUKBxwV.png)
- Now let's test this with our test VM.
{%youtube n2XG6K3_53U %} 
 YAYY!
- Working !! :)


# :100: :muscle: :tada:



---

### Thank you! :sheep: 

You can find me on

- GitHub :<code class="blue"> 
https://github.com/bayegaspard
</code>
- Twitter:<code class="blue"> 
https://twitter.com/bayegaspard
</code>
- or email me <code class="blue">
gaspybaye@gmail.com
</code>


