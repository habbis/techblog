---
title: "Setup a server with zfs and docker"
date: 2018-05-18T10:34:11+02:00
draft: false
---



You are free to use this as you wish I am also using this as my own wiki to remember 
how to do stuff. I hope this can help other people.

I am going to set up a [Digitalocean](https://www.digitalocean.com/) vps with 
docker and testing out zfs as [docker](https://docs.docker.com/storage/storagedriver/zfs-driver/) storage driver.

I am going to use Ubuntu 16.04 since it have a native zfs binary.

First login on digital-ocean server via [openssh](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) via root like this:

```
    ssh root@yourip
```
- Then add a user and on ubuntu you can use two different commands. in these·
 examples I am going to create a use called youruser. Example one adduser is a·
 more user-friendly way you do not need to add flags but if you need read the manpage.·

```
    adduser youruser
```

- Example two is user add and this command is on all Linux distros and the flags·
 will be the same. But on the other UNIX-like FreeBSD or smartos the flags are different in my experience.·
 Here is with the short flags. I like to use user add with scripts since it is·
 non-interactive and therefore easier to automate.·



```
    useradd -m -s /bin/bash youruser
    useradd --create-home --shell /bin/bash youruser
```

- To remove user and home directory use this command·

```
    userdel -r youruser
    userdel --remove youruser

```

- Then I use usermod utility to change user accounts. In this example I am·
 going to add a user to the sudo group.·


```
    usermod -a -G sudo youruser
    usermod --append --groups sudo youruser

```

- Then I use usermod utility to change user accounts. In this example i am·
 going to add a user to the sudo group. Here in this example sudo is enabled·
 because you see the %sudo.  If it's disabled, you will see #%sudo.·


```
    # User privilege specification
    root    ALL=(ALL:ALL) ALL

    # Members of the admin group may gain root privileges
    %admin ALL=(ALL) ALL

    # Allow members of group sudo to execute any command
    %sudo   ALL=(ALL:ALL) ALL

    # See sudoers(5) for more information on "#include" directives:

    #includedir /etc/sudoers.d
```
- Then after setting up the users on the system then we are going to set up the 
ssh-keys for the user youruser. Then since we have already setup ssh-keys via digital-ocean. 
The root user directory already has the necessary keys installed. 
So, on a system the ssh-key is inside the .ssh directory on each user that you allow somebody to ssh into. 
on this system  root only have ssh-key it's in the dir /root/.ssh . 
In openssh the keys are in the file called authorized_keys in the .ssh directory. 
Then I recommend copying the .ssh dir with all files to the home folder to /home/youruser/. 
So, I use this command and remember there is always the alternative command.·


```
    cp -r /root/.ssh /home/youruser/
```
- After you have done that then I like to make user the right permission is one 
the files and folder in .ssh. 
Sometimes i have failed to login to new user since the owner of the folder .ssh 
is root. So, I set it to youruser with this command.·


```
    chown habbis:habbis /home/youruser/.ssh

    # Then ls -al the see who owns the folder
    drwx------ 2 youruser youruser 4096 Mar 14 10:14 .ssh
```

 - It is the best practice to have only root user can delete, read, write, execute with the .ssh 
folder on all users on a system. So, we set the permissions with this command.·


```
    chmod 700 /home/youruser/.ssh
    # the ls -al to see
    drwx------ 2 youruser youruser 4096 Mar 14 10:14 .ssh
    # And it must not look like this
    drw-rw-rw- 2 youruser youruser 4096 Mar 14 10:14 .ssh
```
 - And on the authorized_keys file uses the same command. But here to this root read, write and user read, 
and another read. And i set the owner of the authorized_keys to root:root. 
So, it is going to look like this.·

```
    chmod 644 /home/youruser/.ssh/authorized_keys
    # then ls -al to see
    -rw-r--r-- 1 root   root    762 Mar 14 10:14 authorized_keys
```


 And on the authorized_keys file uses the same command. But here to this root read, write and user read, and another read. And i set the owner of the authorized_keys to root:root. So, it is going to look like this. They are [firehol](https://firehol.org/) , [ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04) , [firewalld](https://fedoraproject.org/wiki/Firewalld?rd=FirewallD) and plain old [iptables](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands).

I am going to use ufw and i am going to show some commands to get started.
We are going for the standard server way of denying incoming and allow outgoing.
First allow ssh so you can connect to the server. 
Then do not start the firewall before you have allowed ssh.
So, you don't lock yourself out of the server.

```
    ufw default deny incoming
    ufw default allow outgoing

    # then allow ssh or port
    ufw allow ssh
    uwf allow 22

    # to deny port or service or ipadress
    ufw deny yourting

    # to start
    ufw enable

    # to see status
    ufw status or ufw status verbose
``` 
  

-   Now we are going to setup the disks and installing zfs and [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

```
    apt install -y git zfs-initramfs apt-transport-https ca-certificates curl software-properties-common

    # then curl the docker gpg-key for ubuntu
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

    # then add the repositoru for ubuntu
    sudo add-apt-repository \
               "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
                  $(lsb_release -cs) \
                     stable"
    # Then update and install
    apt update -y && apt install -y docker-ce
```

- Now we are going to partition the disk. I am going to use the gdiks utility, 
but you can also use fdisk utility to partition the disk so when using 
the zfs on Linux I use the long disk alias. 
Because the sdb naming may change if you add more disks.

```
    # when to partition the disk the command looks like this
    sgdisk     -n1:0:0      -t1:BF01 /dev/disk/by-id/yourdisk

    # The disk alias may use looks like this and when is is partitioned it will
    # have the -part1 in the end.
    /dev/disk/by-id/
     scsi-0DO_Volume_volume-fra1-02-part1
```
-  Now we are going to format the disk to zfs. And when making the pool the
command is zfs create. We are also going set so
me flags when creating the zfs pool.·

- The first flag is -o property=value and between argument we are going to use
the -O flag to perform overlay mount see mount in zfs man pages to learn more.
We are going to set ashift=12 because of physical sectors size will change in the future.
On the vps this setting is not that important. Then setting atime=off because
atime controls whether the access timer for files is update when they are read.
Turning this off avoids producing write traffic when reading files and can
result in significant performance gain. And you can learn more read the zfs man pages.·

- The next flag we are going to use is compression=lz4 because it is a fast compression 
and also one of the defaults for zfs. And using compression saves disk space 
this is why zfs i also one of the best file systems we have. Then use the 
flag normalization=formD because it eliminates some corner cases relating 
to UTF-8 filename normalization. So, the next flag is setmountpoint=yourmount 
this flag says it all with its name. The last thing is not a flag, but you set 
the name of the pool and set if you want to mirror or raidz the disks.··

 - But if you say nothing then your zfs disks will be one big storage pool with no redundancy but. 
If you have only had one disk, then and you still can use zfs as your file system. 
But you miss out some features like error repairing and you can also mirror two partition on a disk with zfs. 
So, then you can have some cool zfs repair and redundancy features·

```
    # This how i all looks when we are ready to format the disks.
    zpool create -o ashift=12 \
          -O atime=off -O compression=lz4 -O normalization=formD \
          -O mountpoint=/mnt/mypool  \
          dockpool mirror \
     /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-01-part1 /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-02-part1

    # then to check if the pool is up use the this command.
    zpool list

    # it will look like this
    NAME       SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
    dockpool  31.8G   324K  31.7G         -     0%     0%  1.00x  ONLINE  -

    # to see more info use the command
    zfs status

    # the output will looke somthing like this.
    pool: dockpool
    state: ONLINE
     scan: none requested
    config:

           NAME                                      STATE     READ WRITE CKSUM
           dockpool                                  ONLINE       0     0     0
             mirror-0                                ONLINE       0     0     0
               scsi-0DO_Volume_volume-fra1-01-part1  ONLINE       0     0     0
               scsi-0DO_Volume_volume-fra1-02-part1  ONLINE       0     0     0

    # then we create the dataset for docker
    zfs create dockpool/docker_stor 

```

-  Now when the disks are ready we are going to change default storage drive on docker to zfs. 

```
    # First stop docker with this command
    sytsemctl stop docker

    # then use this command to check status
    systemctl status docker

    # Then we are going to take backup of the docker directory.
    # Then you have backup of your docker images.
    cp -au /var/lib/docker /var/lib/bk.docker

    # Then delete all file in docker dir.
    rm -rf /var/lib/docker/*

    # Then set the mountpoint for the zfs dataset docker.
    zfs mountpoint=/var/lib/docker dockerpool/docker_stor

```

- Now set zfs as a storage driver to do that create a file in the dir /etc/docker called daemon.json.

```
    # i use vim to create the file.
    vim /etc/docker/daemon.json

    # In the file place this
    {
      "storage-driver": "zfs"
    }

    # Then start docker again
    systemctl start docker

    # then run this
    docke info | grep zfs

    # you should see this output.
    Storage Driver: zfs

    # ubuntu and debian system you can see this errori when running docker info.
    WARNING: No swap limit support
```

- To remove this warning you can enable some settings and here is a link the [docs](https://docs.docker.com/install/linux/linux-postinstall/#your-kernel-does-not-support-cgroup-swap-limit-capabilities)

```
    # in the /etc/defaultg/grub write this to enable the feauter realted no swap error
    GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"

    # then update grub with this command.
    update-grub
```

-   Now we are ready to use zfs with docker.
