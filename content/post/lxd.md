---
title: "Lxd"
date: 2018-05-14T21:03:41+02:00
draft: false
---

## LXD setup

I am now trying out LXC and LXD both are cool technologies. Lxd  it is a· 

hyper-visor for lxc containers(linux containers). 

So why LXD/LXC?  That is a good question and it can have many answers. 

It has benefits like being closer to the metal. And it behaves more like a 

virtual machine and for me it is easier understand concept behind it. 

Lxd is also more like Freebsd jails and the solaris zones. 

And lxd/lxc consisting of  namspaces to isolate process. And cgroups to· 

controll the resources of the processes. 

But one may think this is a competitor to docker but in my eyes no.· 

Docker is an app container and lxd is a system container. 

In fact, you can use ubuntu, centos as system container in lxd and they will· 

operate more like a virtual machine. So docker and lxd are both super cool tech· 

and i use them both. I am currently trying to learn more of both container technologies. 

So how do one get started with lxd the easy way is to use ubuntu. 

But other Linux distributions also can support lxd but you must do more manual· 

setup. I am going to concentrate on using ubuntu since that is my main Linux distro.  


So, to install lxd to ways either use the deb binary 

```
sudo apt install -y lxd

```

or use.

```
snap install lxd

```

After this use this command it the terminal with sudo permissions. 

```
sudo lxd init

```

 After that command the setup process will begin. And I am going to show how
 it looks like under her. 

Her I say yes it is easier  this way. 

 ```

Do you want to configure a new storage pool (yes/no) [default=yes]?

```

Now it is time to choose the file system for storing and running containers. 
If I can I choose zfs but one can also use the default 
file system on your disk then that is dir.

```

Name of the storage backend to use (dir or zfs) [default=dir]:

```

  If you want, you can have lxd exposed over the network, so you can control it·
  with other lxd client like your laptop. I am going to show it but the default· 
  is no so if you want type yes

```

Would you like LXD to be available over the network (yes/no) [default=no]?

```

There you can choose a specific ip address if you have more than one. 
The default is all. 

```
Address to bind LXD to (not including port) [default=all]:

```

Now you can choose a port the default is port 8443 tcp.

```
Port to bind LXD to [default=8443]:
```

Now you can setup a password for connecting to this lxd client and choos a 
good password.

```
Trust password for new clients: 
```

Then it is time to setup the lxd bridge so your containers can have network 
access.

```
Do you want to configure the LXD bridge (yes/no) [default=yes?
```
Then it will prompt you trough setting up the ip4. And ip6 subnet for the 
containers you can mostly press enter for the default are good.

Then you are ready to use lxd and the last message should be.

```
LXD has been successfully configured.
```

To show your running containers the command is.

```
lxc list
```

Result if no container are running.

```

+------+-------+------+------+------+-----------+ 
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+ 


```

If a container is running it looks like this.
```
+------+---------+--------------------+----------------------------------------------+------------+-----------+
| NAME |  STATE  |        IPV4        |                     IPV6                     |    TYPE    | SNAPSHOTS |
+------+---------+--------------------+----------------------------------------------+------------+-----------+
| t1   | RUNNING | 10.196.76.9 (eth0) | fdcf:4fe8:8c3f:de07:216:3eff:fe3e:222 (eth0) | PERSISTENT | 0         |
+------+---------+--------------------+----------------------------------------------+------------+-----------+

```

To launch a container the command is.

```
lxc launch ubuntu:16.04 yourcontainername
```

To stop container.

```
lxc stop yourcontainer
```

To list images 

```
lxc image list
```

To make a snapshot of the container.

```
lxc snapshot youtcontainer snapshotname
```

This is a basic setup of lxd on ubuntu 16.04.


