---
title: "Setup_kvm_virt"
date: 2018-09-09T17:21:49+02:00
draft: true
---

  

  

  
## Linux Virtualization with KVM
  

  

You are free to use this I use this blog like a wiki to write things I learn. 

  

Here are some other Open Source Virtualization projects but  

I am going to setup KVM. 

  

  

- KVM - Kernel-based Virtual Machine 

- Xen 

- VirtualBox 

- UML - User Mode Linux 

   

Setup Linux Virtualization with KVM. 

  

  

What are KVM is a hypervisor built into the Linux Kernel and it  

allows Linux desktop or server to simulate multiple piece of hardware. 

  

KVM uses the QEMU virtual machine format so what are the difference between QEMU and KVM. First, they are two separate software projects and QEMU is primary a  

hardware emulator. KVM is a kernel module that is used to expose hardware  

virtualization technologies such as: 

  

- Intel VT-x or AMD SVM 

- KVM then uses QEMU for the device emulation. 

  

When QEMU and KVM work together, KVM arbitrates access to the CPU 

and memory, while QEMU emulates hardware resources like hard disk, video card, 

USBs and more. 

  

Hardware Virtualization  

  

- Full virtualization  

  

  - Complete simulation of the actual hardware to allow software, 
   which typically consists of the guest machine or virtual machine.

  

  - KVM uses full virtualization. 

  

- Para virtualization 

  

  - The hardware environment is not simulated. 

  - XEN Supports this not KVM. 

  

Types of Hypervisors  

  

VMM/hypervisor is a piece of software that is responsible for monitoring 

and controlling virtual machines or guest operations systems 

  

Type 1 hypervisor  

  - The hypervisor runs directly on top of the hardware. 

  

Type 2 hypervisor  

  - The hypervisor acts as a separate layer often on top 

of a base operating system.  

  

Libvirt 

  

Libvirt is a set of API libraries that sites in between the end user and the 

hypervisor 

  

The hypervisor could be built to use any virtualization technology such 

as KVM/QEMU, XEN, LXC, VirtualBox, VMWARE ESX, MS HyperV and 

even Parallels 

  

Libvirt acts as a transparent layer that can take commands from 

users modify them based on the underlying virtualization technology, 

and then executes them on the actual hypervisor 

Tools include the libvirtd daemon, API library, and command line utility 

called virsh 

  

Linux virtualization tips  

  

   - Overcommitting 

  

To allocate more virtualized CPUs or virtual memory than the available 

resources on the host system provides. Overcommitting can cause risk to your host system’s stability. 

  

  - Thin provisioning  

  

Allows you to optimize the available storage space the guest virtual machines. 

Like overcommitting, but only pertains to storage, not CPU and memory and 

can also risk the system stability 

  

Linux Virtualization are also used in cloud solutions. 

-  OpenStack 

-  Eucalytus 

-  Cloudstack 

  

Installing Virtualization Packages 

  

To check for KVM support `lsmod | grep kvm` 

Then check for this for AMD cpu the `kvm_amd`  and for for  `INTEL cpu kvm_intel`  so if these flags show up then we have cpu support for KVM. 

  

  

First, we install these packages, so we can use Linux virtualization. 

The package libguestfs-tool is for virt-builder and is not needed for kvm 

but is usefull and virt-manager is a gui manager for Linux virtualization and  

you can skip it for servers. 

  

  

Centos7   

  

`yum install -y  qemu-kvm libvirt virt-install libguestfs-tool-c virt-manager` 

  

Fedora  

  

`dnf install -y qemu-kvm libvirt virt-install libguestfs-tool-c virt-manager` 

  

Groupinstall for Fedora  

  

`dnf install -y @virtualization`  

  

Ubuntu/debian systems  

  

`apt  install -y  qemu-kvm libvirt virt-install libguestfs-tools virt-manager` 

  

  

To list your machine capabilities `virsh domcapabilities` 

  

  

libvirt-based-tools to manage virtual machines 

  

First tool to manage virtual machines is the virsh shell  

Hers is some commands.  

  

To list all machines running   

  

`virsh list --all` 

  

This only show running virt-machines. 

`virsh list`    

  

To start a virt-machine   

`virsh start machine`

  

It`s important to remember and check on the virtual machine what the name of the disk is called before attaching the disk-image. 

  

To attach a disk to a virt-machine  

`virsh attach-disk  nameofmachine --source /path/to/disk.format --tar get namelike-vdc --persistent` 

  

  

  

To stopp a virt-machine. 

`virsh shutdown yourmachine`   

  

to start a virt-machine. 

`virsh start`  

  

to reboot a virt-machine. 

`virsh reboot`  

  

to pause a virt-machine. 

`virsh suspend`  

  

Since virsh is a command line tool then use  

virt-viewer to see the console or gui inside the  

virtual machine like this. 

`virt-viewer  yourmachine`   

  

Then the last tool to manage virtual machines is the gui tool  

virt-manager and it can start stop take snapshots  

add disk-images manage storage pools and create and delete network bridges. 

Virt-manager can also manage other machines hypervisor and virtual machines  

with ssh. 

  

Tools for managing disk-images and install virtual machine  

  

virt-builder creates ready to use virt-builder creates disk images for the most  

common Linux distros so you not need to install them with a iso. 

  

Here is a virt-builder command for more options see the manpage. 

`virt-builder ubuntu-18.04 --root-password password:yourpassword --size 51G --format qcow2`  

  

  

When managing disk-images you use the utility qemu-img to create new empty images like this and see the manpage for more options. 

  

`qemu-img create -f yourformat diskname.format  1G` 

  

  

To setup a virtual machine to use the disk-image you can use virt-manager or  

use the virt-install command line utility like this.  

  

`virt-install --name distro --ram 1024 --vcpu=1 --network bridge=virbr0 --disk path=/path/to/disk-image.qcow2 --import` 

  

Or you can have more options like when install ing with a iso. 

  

`virt-install --name yourmachine --ram 1100 --disk path=/path/to/disk.format --vcpu 1 --os-type=youros --network bridge=virbr0 --graphics vnc,port=portnumber --console pty,target_type=serial --cdrom /path/to/iso` 

  

  

The tool qemu-img can also convert a disk-image from raw to qcow2 like this.  

  

`qemu-img convert -f raw -O qcow2 yourimage.img  yourimage.qcow2` 

  

snapshots  

     

With libvirt there is internal snapshot and external snapshot 

Internal snapshot is everything inside a qcow2 fileformat. 

  

Internal snapshot does pause the machine, so it's not recommended to use on important production machines when uptime is important and internal snapshot is only for qcow2 disk-format. 

  

It's better to use them when you are having maintenance on a production  

machine. 

  

External snapshot is based on the concept copy on write file-system. Then a  

exsternal snapshot is read only. 

  

Exsternal snapshots do grow in tandem with the virtual-machines and it support all the disk-images formats in libivrt. 

  

To list snapshots  

`virsh snapshot-list yourmachine`.

     

Internal snapshot does this command. 

`virsh snapshot-create yourmachine` 

  

exsternal snapshot like this. 

  

virsh snapshot-create-as diskname  snap1-diskname \ 

  "snap1-diskname-description" \ 

  --diskspec hda,file=/export/vmimgs/snap2-disknameqcow2 \ 

  --disk-only --atomic 

  

  

  

To revert to a snapshot, use this command. 

`virsh snapshort-revert yourmachine --snapshotname "nameofsnap"` 

  

Bridge Networking  

  

Virtual machines need networking to work so then we use bridge networking to  

allow virtual interfaces to connect to the outside network through the physical 

interface, making them appear as normal host to the rest for the network.  

  

When installing Linux virtualization packages, it will setup a standard bridge  

network for you called virbr0 and it's using NAT to allow the virtual machine  

to connect to the outside network. 

  

There are many tools to manage Bridge networking like. 

  

- Brctl 

- Virt-manager -> Edit -> Connection-Details -> Virtual-Networks 

- Systemd-networkd 

- NetworkManager 

  

link to [archlinux-wiki](https://wiki.archlinux.org/index.php/Network_bridge) site to learn more. 

  

  

  

 
