When you have OVA file it saves your time to install and configure the VM by yourself. In this article, I have explained all the steps to import OVA to Proxmox.
</br>

## What Is an OVA File?
---
OVA file or a file with OVA extension is known as an Open Virtual Appliance, which means an Archive format of a ready Virtual Machine (VM) that you can import into your Virtualization Platform like Oracle Virtual Box or VMWare.

The file contains OVF as an Archive format with the files VMDKs. OVF is a descriptor XML-Based file. The archive may also contain the ISO or other related resource files.

OVA archive includes OVF and VMDK.
</br>


### Step to Import OVA to Proxmox.
---
Here you don’t need an OVF file but you need VMDK which is the Virtual Disk, you need to you that disk to be used as your Virtual Disk for your VM in Proxmox. 
</br>

#### Step 1 :Download the OVA file from the source.

In my case, I have downloaded OpenPPM OVA Production file.
</br>

#### Step 2 : Upload OVA File to Proxmox

You can use WinSCP or any other preferred tool to upload the file to /home directory on Proxmox


In my case, I have uploaded this file to the /home folder. So make sure you chose your required directory.
</br>

#### Step 3 : Extract OVA file in Proxmox Server

Login to your proxmox Server using SSH, I will be using putty. locate the folder and exact the file to the same location where it has been extracted.

Use the following command to extract (make sure to change the file name.ova)

```tar xvf openppm.ova```
</br>

xvf means : extract –verbose –file . Where x stands for extracting the archive, v for displaying Verbose information, and f for specifying a filename.


#### Step 4 : Import VM Settings to Proxmox

You can create the VM by yourself in Proxmox but this command will automatically create VM in Proxmox with required configuration.

If you see the extracted OVA contains two files. One is OVF that contains the configuration of VM and another is Disk.

First we will create VM using the Configuration file.

Use below command in SSH

```qm importovf 137 ./openppm.ovf proxthin --format qcow2```
</br>

Once method is to create VM by yourself.

#### Step 5 : Import Disk to VM

Finally, I will be importing the VMDK to My Virtual Machine, the VM Number is 136 so I will use below command

```qm importdisk 136 openppm.vmdk local-lvm -format qcow2```

Below is an explanation of the above command

qm : short form of QEMU which means Quick Emulator

importdisk : Command used to import disk

136 – VM ID of Proxmox this is the new machine I created as per step no. 1

openppm.vmdk the file name of VMDK extracted from step 3

local-lvm: storage name of LVM volume from my installation, you can use your installation name.

–format : the format of your disk

qcow2: storage format for virtual disks. QCOW stands for QEMU copy-on-write. The QCOW2 format decouples the physical storage layer from the virtual layer by adding a mapping between logical and physical blocks


#### Step 6 : Add disk to VM

You notice that disk is added to VM but it is not yet added or attached to VM.

Just click the this and add.

#### Step 7 : Start VM

Now start the VM, if you follow all the steps above the VM will for sure work.

#### Step 8 : Add Virtual Network Interface

You will need to add hardware in the hardware section of VM and attach the network card. You may need to change the network settings of the VM. Depending upon the operating system of VM you will need to configure it.

