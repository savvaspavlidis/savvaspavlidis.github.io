---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 6) -  OS Install via kickstart
date: 2017-04-19 09:00:00
categories: oracle
comments: true
disqus_identifier: 0012
---

## OS Install via kickstart file (unattended)

First of all, I should note that we have  to prepare two systems, the master and the failover. I use a kind of unattended install with a file that has all the selections that normally an administrator would select on console. But not only that, this file gives us the capability to write and execute scripts in bash (or any other language) that would modify and do all the necessary changes without intervention.

So we have two servers, oratest1 (the master) and oratest2 (the failover). A large part of the kickstart file is almost the same, and the only difference is that the install of the oracle software is done on oratest1. And this is done, because it would be replicated and mirrored via the DRBD, as all the files needed for the Oracle RDBMS, and should be the same on both systems. The specifics of the Oracle installation would be covered in the next chapter. It should be noted that oratest1 and oratest2 for this tutorial are systems with two hard disks, sda and sdb, with same capacity (24GB and 16GB respectively). This can be changed according to your specifications. Also RAM should at least 4GB for each system.

The kickstart file has actually two parts. The first part, is like the selection we do to anaconda, for the installation of the Operating system. The second part, the post-installer script, is actually using bash, so we can right in bash (and in other script languanges like perl or python) everything needed to modify the verbatim installation to our specifications.

I will explain the parts of kickstart file, step by step, although, the comments in most cases are good enough.
```
######################################################################################
# Do install, not upgrade
install

######################################################################################
# Where is the repository for the installation and the access method, ftp or http.
# For ftp, it may be anonymous ftp, or with names
# but then credentials must be given at the command line
url --url ftp://myanonymousftpserver/centos/6/os/x86_64/
```
in place of myanonymousftpserver you should place the name that can be resolved to your anonymous ftp server or its IP address.
```
######################################################################################
# the following are self-explanatory
lang en_US.UTF-8
keyboard us
timezone --utc Europe/Athens

######################################################################################
# Network confuguration. We may provide directly static address, BUT
# by using DHCP, we are sure that it will take a free IP in the pool
# which later will be used as static.... Otherwise we should check
# the availability of the IP that we will give as static.
network --noipv6 --onboot=yes --bootproto dhcp --hostname oratest2
```
the first three lines need no more explanation. Change according to your needs.

But we have to talk a little more about the network configuration. Because Oracle insists (and I didn't want to circumvent it) the static IP of the server it is installed on and be resolved via DNS too, we need that. So prior to install the OS, we must have included an entry in the DHCP Server, that for the specific MAC address of the machine, it will take a specific IP, and that will also be registered in the DNS Server. Thus, prior to installing, the MAC address should be known. If this is not an option, due to remote nature of installing, we may start the install as it is, and we will retrieve the MAC and it's IP on the DHCP Server, and include it on dhcp.conf file. Later, via the use of scripts, we will change it.

Another approach would be to make directly static here, but then we should update the DNS server likewise. Perhaps I will post this scenario, as an extra, later.
```
######################################################################################
# Enable shadow encrypted password file, md5 and give root a password
authconfig --enableshadow --enablemd5
rootpw --iscrypted $6$U/9F9pAN$S5PXDI/z3PNIT3h.42GwtHg1Q5mlmbA6jzsFJ9bwtdJrZ0Yhqhp9WVx4fEKtz/bhNQ/9zdQc7Pz8/LX4/d6/m0
```
This is also easy to explain, it is the default password for the administrator account root. Ofcourse it is encrypted. Because the kickstart file resides in an anonymous ftp server, we don't want everyone to see it, if it was unencrypted. In our case above it is the easy 1234 (what else should I have selected? :-) )

In your case, it is easy to make the initial password to your liking. Just run the following, give the password you want at the prompt, and copy and paste accordingly to the line above in the kickstart file, replacing mine's.
```
grub-crypt --sha-512
```
next we have 
```
######################################################################################
# Allow specific ports on system, 22 for SSH and 1521 for ORACLE
firewall --enabled --port 22:tcp --port 1521:tcp

######################################################################################
# ORACLE is not so well behaved with SELinux, all tutorials say to have it
# either permissive or disabled at all. Someday I will work on it...
selinux --disabled

######################################################################################
# define the bootloader options
bootloader --location=mbr --driveorder=sda --append="crashkernel=auth "

```
Some may cough up with my selinux configuration. You may use also permissive. I haven't tried it yet with enforcing with such a configuration, I mean with DRBD clustering and Oracle. 

```
######################################################################################
# SERVICES
#
# A miniman system like this, loads some services that are not necessary
# for an Oracle RDBMS (strictly an Oracle RDBMS Server, and nothing else!)
# Thus, disable them. Enable sendmail instead of postfix, just because I
# know it well and does the job allright. Change to your liking.
services --disabled auditd,lvm2-monitor,ip6tables,portreserve,cpuspeed,cups,netfs,abrt-ccpp,abrtd,kdump,postfix
services --enabled sendmail

######################################################################################
# DISK PARTITIONING
#
# We assume a good server, thus a hardware raid card, best with RAID-10
# Linux will see one (or perhaps more) disks that are already mirrored.
# logical disks would be managed by hardware raid
zerombr yes
clearpart --all --initlabel --drives=sda,sdb
part /boot      --fstype=ext4   --size=500                      --asprimary --ondisk=sda
part swap       --fstype=swap   --size=4096                     --asprimary --ondisk=sda
part /          --fstype=ext4   --size=18000                    --asprimary --ondisk=sda


```

as for the services, because I have a very good experience with sendmail, I still use this, than postfix. I'm only human, don't put the blame on me!
Several services that are enabled by default, and we don't need them, we disable them right from the start here.

As for the disk partitioning. As I said, in my installment there are two disks, but you may have one or whatever scenario fits your case. We just need to make the right modifications. In this case, I have two disks, initialize both, so no partitioning will be on them. Afterwards, partition the first with a simple partitioning scheme, and enforce all of the partitions to be on disk sda. this is crucial, because the partition manager may try to use both 

```
######################################################################################
# PACKAGES
#
# The minimum required for an Oracle Database Server
# Normally, for the installation of an Oracle Database,
# we need also a graphics desktop, but only for the installation
# (and perhaps some management tools). We can avoid that, and
# not install software just for the installation of the Database,
# because there is a method which a GUI desktop is not required.
%packages
@Server Platform
@Development Tools
binutils
compat-libstdc++-33
compat-libstdc++-33.i686
ksh
elfutils-libelf
elfutils-libelf-devel
glibc
glibc-common
glibc-devel
gcc
gcc-c++
libaio
libaio.i686
libaio-devel
libaio-devel.i686
libgcc
libstdc++
libstdc++.i686
libstdc++-devel
libstdc++-devel.i686
make
sysstat
unixODBC
unixODBC-devel
sendmail
sendmail-cf
yum-plugin-priorities
lshw
libxslt
libxslt-devel
compat-libcap1
ftp
iptraf
-postfix


# Make sure we reboot into the new system when we are finished
reboot
```
here we select which packages should be installed. We can select installement of groups and individual packages, plus, which packages we do not to be installed at all.

Finally, with the reboot, it says that initial configuration for the install, the first part, is finished. and next in kickstart file is the postinstaller phase, after the initial configuration. Here, we can script and do whatever we want.






<div id="disqus_thread"></div>
<script>
  var disqus_config = function () {
    this.page.url = "{{ page.url | prepend: site.url }}";
    this.page.identifier = "{{ page.disqus_identifier }}"; 
  };
  (function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = '//savvaspavlidis.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
