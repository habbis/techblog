---
title: "Server With Zfs"
date: 2018-05-10T19:46:34+02:00
draft: false 
---

- You are free to use this as you wish I am also using this as my own wiki to remember how to do stuff. I hope this can help other people.

- I am going to sett up a [Digitalocean](https://www.digitalocean.com/) vps with
docker and testing out zfs as [docker](https://docs.docker.com/storage/storagedriver/zfs-driver/) storage driver.

- I am going to use ubuntu 16.04 since it have a native zfs binary.

- First login on digital-ocean server via [openssh](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) via root like this:

```
ssh root@yourip
```

- Then add a user and on ubuntu you can use two diffrent commands.
in these exampels I am going to create a use called youruser.

- Eksample one adduser is a more userfriendly way
you do not need to add flags but if you need read the manpage.

```
adduser youruser
```

- Eksample two is useradd and this command is sipported on all linux distros
and the flags shall be the same. But on the other unix-like opertingsystem
freebsd, smartos the flags is diffrent in my experience.

- Here is with the short flags. I like to use useradd with scripts since
it is non interactive and therfor eayser to automate.
```
useradd -m -s /bin/bash youruser
useradd --create-home --shell /bin/bash youruser
```

- To remove user and home directory use this command
```
userdel -r youruser
userdel --remove youruser
```

- Then I use usermod utillity to modify user accouts.
Inn this example i am going to add a user to the sudo group.

```
usermod -a -G sudo youruser
usermod --append --groups sudo youruser

```

- In ubuntu the sudo group is called sudo but in other unix like os it
often called wheel.The best way enable sudo in a unix like
os use the command visudo.

- Here in this exsample sudo is neabled beacuse you see %sudo
if was disabled you will se #%sudo.

```
# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d

```

- Then after setting up the users on the system then we are going to set up
the ssh-keys for the user youruser. Since we have allready setup ssh-keys
via digital-ocean the root user directory allready have the nessescary keys
installed.

- So on a system the ssh-key are stored in the.ssh directory on each user that
you can ssh into ofcourse.Since root only have ssh-key here then in the dir
/root/.ssh are the keys. The ssh-keys are always in openssh
stored in a file called authorized_keys in the.ssh directory.

- Then i recommend copying the.ssh dir with alle files to the home folder to
/home/youruser/. So i use this command and remmember there is always the
alternativ command.

```
cp -r /root/.ssh /home/youruser/

```

- After you have done that then i like to make user the right permission is one
the files and foler in .ssh. Sometimes i have failed to login to new user since
the owner of the folder .ssh is root so i set it to the user youruser with this
commmand.

```
chown habbis:habbis /home/youruser/.ssh

# Then ls -al the see who owns the folder
drwx------ 2 youruser youruser 4096 Mar 14 10:14 .ssh

```

- It is the best practice to have only root user can delete,read,write,execute
with the .ssh folder on all users on a system. So we set the permissions with
this commamnd.

```
chmod 700 /home/youruser/.ssh
# the ls -al to see
drwx------ 2 youruser youruser 4096 Mar 14 10:14 .ssh
# And it shall not look like this
drw-rw-rw- 2 youruser youruser 4096 Mar 14 10:14 .ssh
```
- And on the authorized_keys file use the same command but here
root read,write and user read and other read.And i set the owner of
the authorized_keys to root:root. So it is going to look like this

```
chmod 644 /home/youruser/.ssh/authorized_keys
# then ls -al to see
-rw-r--r-- 1 root   root    762 Mar 14 10:14 authorized_keys
```
- After setting up ssh then we need to start the firewall
and on ubuntu you have foure options as i know of but all
of then is based on iptables and they are [firehol](https://firehol.org/)
, [ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04) and plain old [iptables](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands).

- I am goint to use ufw and i am just going to show some of the commands to get
started.

- We are going for the standard way deny incoming
and allow outgoing. And allow ssh so you can connect to
the server and do not start the firewall before you have
allowed ssh connection so you do not to lock yourself out of
the server.

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
