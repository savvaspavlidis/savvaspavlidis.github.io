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

%pre
```
here we select which packages should be installed. We can select installement of group of packages (thos with @ in front) and individual packages, plus, which packages we do not to be installed at all (those with the minus in front).

Finally, with the %pre, it says that initial configuration for the install, the first part, is finished. and next in kickstart file is the postinstaller phase, after the initial configuration. Here, we can script and do whatever we want.

Now the postinstaller phase begins.
```
######################################################################################
######################################################################################
# Upon completion of the installation, run the script below for
# further customization
# http_proxy is used not only for the changes, but also as for the rpm/wget which
# it uses it. If not named like that, it cannot be used by rpm/wget

%post --log=/root/install-post.log
(
PATH=/bin:/sbin:/usr/bin:/usr/sbin
export PATH
export ANONFTPSERVER=anonftpserver.example.com
export FTPREPOSITORY=ftp://$ANONFTPSERVER
export http_proxy=http://proxy.example.com:8080


######################################################################################
# REPOS
#
# Change repos, Install extra repos, update system to latest
# change repos to use the local ftp repository for installation/update
# change to use proxy for faster downloading
echo "Updating YUM Repositories"
cd /etc/yum.repos.d
perl -npe '/mirrorlist=.*repo=os/ && s/^/#/'            -i CentOS-Base.repo
perl -npe '/mirrorlist=.*repo=updates/ && s/^/#/'       -i CentOS-Base.repo
perl -npe '/mirrorlist=.*repo=extras/ && s/^/#/'        -i CentOS-Base.repo
perl -npe '/^#baseurl=.*\/os\// && s/^#//'              -i CentOS-Base.repo
perl -npe '/^#baseurl=.*\/updates\// && s/^#//'         -i CentOS-Base.repo
perl -npe '/^#baseurl=.*\/extras\// && s/^#//'          -i CentOS-Base.repo
perl -npe '/^baseurl/ && s/http:\/\/mirror.centos.org/$ENV{FTPREPOSITORY}/' -i CentOS-Base.repo
yum -y update

perl -npe '/=centos-release/ && s/=centos-release/=centos-release\nproxy=$ENV{http_proxy}/' -i /etc/yum.conf

rpm --import http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum -y install epel-release.noarch

# instal ELRepo repository for DBRD
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm
# install the DRBD
yum -y install kmod-drbd84 drbd84-utils

# change repos to use priorities in order to avoid conflicts
perl -npe '/gpgcheck=1/ && s/gpgcheck=1/gpgcheck=1\npriority=10/' -i Centos-Base.repo
perl -npe '/gpgcheck=1/ && s/gpgcheck=1/gpgcheck=1\npriority=20/' -i epel.repo
perl -npe '/gpgcheck=1/ && s/gpgcheck=1/gpgcheck=1\npriority=30/' -i elrepo.repo
```
Firstly we define some enviromental variables that would be used throughout the postinstaller phase. You should place your anonymous ftp server in the place of anonftpserver.example.com or its IP. If you are behind a proxy for connection to internet, the http_proxy environment variable is needed, otherwise, you should delete it. If you are using a proxy, replace accordingly with your proxy server the proxy.example.com and the port if needed.

In the first part of the postinstaller phase we make the needed changes to repos, so an update can follow to the latest software (remember, the install previously does not install the updates after the base of the specific version). We make the changes so Centos to use the local (and I pressume always synchronised and updated) repository. Because we will include some other repositories too, the yum-priorities plugin is required in order to avoid conflicts of the same software available from different repositories, and possibly in different versions. 

Now in my case, I have to use a proxy, so there is a command that places the proxy configuration in /etc/yum.conf If you don't use a proxy, ommit it. 

The very known epel repository is installed and also the ELREPO which has the DRBD tools, and finally the priorities are given (the lower the number, the higher the priority).
```
# NETWORK
#
# Make static Networking
echo "Converting DHCP scope to static IP address"
echo "HOSTNAME already is: " `hostname`
HOSTNAME=oratest2.example.com
HOSTNAMESHORT=`echo $HOSTNAME | cut -f1 --delimiter='.'`

DEVICE=`route -n|grep '^0.0.0.0'|awk '{print $8}'`
IPADDR=`ifconfig $DEVICE|grep 'inet addr:'|awk '{sub(/addr:/,""); print $2}'`
NETMASK=`ifconfig $DEVICE|grep 'Mask'|awk '{sub(/Mask:/,""); print $4}'`
NETWORK=`ipcalc $IPADDR -n $NETMASK|awk -F= '{print $2}'`
GATEWAY=`route -n|grep '^0.0.0.0'|awk '{print $2}'`
HWADDR=`ifconfig $DEVICE|grep 'HWaddr'|awk '{print $5}'`

cat <<EOF >/etc/sysconfig/network
NETWORKING=yes
HOSTNAME=$HOSTNAME
GATEWAY=$GATEWAY
EOF

cat <<EOF >/etc/sysconfig/network-scripts/ifcfg-$DEVICE
DEVICE=$DEVICE
BOOTPROTO=static
IPADDR=$IPADDR
NETMASK=$NETMASK
ONBOOT=yes
HWADDR=$HWADDR
EOF


cat <<EOF >>/etc/hosts
$IPADDR         $HOSTNAME $HOSTNAMESHORT
EOF

service network restart
```
This part of the postinstaller phase handles the networking. Makes the network connection as static and places the corresponding IP as static. Plus it informs several configuration files. with this, the NetworkManager service is not required and we can work with the much simpler network service. the only thing needed is the hostname that we want this system to have.

```
# Sendmail configuration
cd /etc/mail

cat <<EOF >>local-host-names
localhost
localhost.localdomain
$HOSTNAMESHORT
$HOSTNAME
EOF


cat <<EOF >>mailertable
.       esmtp:mailserver.example.com
EOF
make mailertable.db


perl -npe '/^dnl MASQUERADE/ && s/^dnl MASQUERADE/MASQUERADE/' -i sendmail.mc

echo "\nSendmail.MC"
cat /etc/mail/sendmail.mc
echo "END OF Sendmail.MC\n"
echo "HOSTNAME:" $HOSTNAME

perl -npe '/^dnl MASQUERADE/ && s/^dnl MASQUERADE/MASQUERADE/' -i sendmail.mc
export MYHOSTNAME=$HOSTNAME
perl -npe '/^MASQUERADE_AS/ && s/mydomain\.com/$ENV{MYHOSTNAME}/' -i sendmail.mc

make sendmail.cf
```
The part above handles the configuration of sendmail. In your case, if you do not use sendmail, ommit it. I pressume another mailserver is used a mail server gateway (smarthost). Instead of modifying the sendmail.mc with the smarthost directive, I do it on the mailertable. 
```
########################################################################
# DRBD Config

if [ ! -d /etc/drbd.d ]; then
        mkdir /etc/drbd.d
fi

cd /etc/drbd.d
cat <<EOF >disk1.res
resource disk1 {
        startup {
                wfc-timeout 30;
                outdated-wfc-timeout 20;
                degr-wfc-timeout 30;
        }

        net {
                cram-hmac-alg sha1;
                shared-secret sync_disk;
        }

        syncer {
                rate 100M;
                verify-alg sha1;
        }

        on oratest1.example.com {
                device /dev/drbd1;
                disk /dev/sdb1;
                address 10.1.1.129:7789;
                meta-disk internal;
        }

        on oratest2.example.com {
                device /dev/drbd1;
                disk /dev/sdb1;
                address 10.1.1.130:7789;
                meta-disk internal;
        }
}
EOF
```
At this point, we make the configuration files for DRBD, although it is not fully configured still. Remember, we will use the second disk, sdb, and its first partition, for network mirroring, and this is named as disk1 in DRBD file above. This means that on the first system, the master, we will access via the /dev/dbrb1 device, which will be mirrored to the failover system. The configuration files are exactly the same on both systems, and that makes it easier. It is via the administrator commands that we specifically assign which one will be the master, and which the slave. I would like to mention the inclusion of the static addresses of both systems here, and is another point, that both system should have preassigned static addresses prior to installation, and these addresses should be applied to the kickstart configuration file.
```
########################################################################
# Crontab the check of time each hour
if [ ! -d /var/spool/cron ]; then
        mkdir /var/spool/cron
fi

cat <<EOF >/var/spool/cron/root
1 * * * * /usr/sbin/ntpdate -s time.windows.com
EOF

ntpdate -s time.windows.com
```
For the proper operation of the DRBD there must be exact timing, and we can't rely on the clock of the system and a manual set. Thus we need to enable the automatic update of time via use of ntp. Here it is assumed that there is the capability to access the Internet for the time protocol directly. Other options would be to have a time server inside the lan.
```
########################################################################
# Make the partition available.
# we do it here, and not with the format of kickstart, because we don't
# want to have it automatically included in fstab. But must be done prior
# to running the drbd commands

(
        echo o
        echo n
        echo p
        echo 1
        echo
        echo
        echo w
) | fdisk /dev/sdb

yes yes | /sbin/drbdadm create-md disk1
/etc/init.d/drbd start
chkconfig drbd on
/sbin/drbdadm secondary disk1
```
As I said earlier, we did not make the partition via the preinstall phase, because it is included in fstab that way. So we do it here, directly using fdisk. Then the partition is prepared, and the DRBD service is started and the disk is marked as the secondary (slave). Remember, firstly we install the secondary (slave) system. We make the service startable upon boot.

Last but not least, we must set up both systems, for Oracle, and I mean the configuration of system variables and user under which Oracle will run. The first part will be to setup the sysctl parameters about memory, changes that are needed by Oracle.
```
########################################################################
# ORACLE


# script to calculate oracle kernel memory parameters
# Latest one, if Huge pages are required, it displays the apropriate kernel parameter
#
# In case of virtualisation, dmidecode does not report correctly always the memory size
memory=$(lshw -quiet -class memory -short | grep System | awk '{print $3}')
memory_type=$(echo $memory | grep -o '[a-Z]\+')
memory_num=$(echo $memory | grep -o '[0-9]\+')
if [ "$memory_type" == "GiB" ]; then
        Total_Memory_M=$(echo $memory_num*1024 | bc)
        Total_Memory_G=$memory_num
else
        Total_Memory_M=$memory_num
        Total_Memory_G=$(echo $memory_num/1024 | bc)
fi

Page_Size=$(getconf PAGE_SIZE)
shmmax=$(echo $Total_Memory_M *1024*1024/2 | bc)
shmall=$(echo \($Total_Memory_M -2048\)*1024*1024/$Page_Size | bc)


cat <<EOF >>/etc/sysctl.conf
net.ipv4.ip_local_port_range = 9000 65500
fs.file-max = 6815744
kernel.shmall = $shmall
kernel.shmmax = $shmmax
kernel.shmmni = $Page_Size
kernel.sem = 250 32000 100 128
net.core.rmem_default=262144
net.core.wmem_default=262144
net.core.rmem_max=4194304
net.core.wmem_max=1048576
fs.aio-max-nr = 1048576
EOF

# should I implement Huge pages? yes if memory is more than 8GB)
SGA_Memory_Size_M=$(echo $Total_Memory_M /2 | bc)
if (( $SGA_Memory_Size_M>8192 )); then
        # Find out the HugePage size
        HPG_SZ=`grep Hugepagesize /proc/meminfo | awk {'print $2'}`
        # Start from 1 pages to be on the safe side and guarantee 1 free HugePage
        NUM_PG=1
        # Cumulative number of pages required to handle the running shared memory segments
        for SEG_BYTES in `ipcs -m | awk {'print $5'} | grep "[0-9][0-9]*"`
        do
                MIN_PG=`echo "$SEG_BYTES/($HPG_SZ*1024)" | bc -q`
                if [ $MIN_PG -gt 0 ]; then
                        NUM_PG=`echo "$NUM_PG+$MIN_PG+1" | bc -q`
                fi
        done

cat <<EOF >>/etc/sysctl.conf
vm.nr_hugepages=$NUM_PG
EOF

fi
########################################################################
# ORACLE


# script to calculate oracle kernel memory parameters
# Latest one, if Huge pages are required, it displays the apropriate kernel parameter
#
# In case of virtualisation, dmidecode does not report correctly always the memory size
memory=$(lshw -quiet -class memory -short | grep System | awk '{print $3}')
memory_type=$(echo $memory | grep -o '[a-Z]\+')
memory_num=$(echo $memory | grep -o '[0-9]\+')
if [ "$memory_type" == "GiB" ]; then
        Total_Memory_M=$(echo $memory_num*1024 | bc)
        Total_Memory_G=$memory_num
else
        Total_Memory_M=$memory_num
        Total_Memory_G=$(echo $memory_num/1024 | bc)
fi

Page_Size=$(getconf PAGE_SIZE)
shmmax=$(echo $Total_Memory_M *1024*1024/2 | bc)
shmall=$(echo \($Total_Memory_M -2048\)*1024*1024/$Page_Size | bc)


cat <<EOF >>/etc/sysctl.conf
net.ipv4.ip_local_port_range = 9000 65500
fs.file-max = 6815744
kernel.shmall = $shmall
kernel.shmmax = $shmmax
kernel.shmmni = $Page_Size
kernel.sem = 250 32000 100 128
net.core.rmem_default=262144
net.core.wmem_default=262144
net.core.rmem_max=4194304
net.core.wmem_max=1048576
fs.aio-max-nr = 1048576
EOF

# should I implement Huge pages? yes if memory is more than 8GB)
SGA_Memory_Size_M=$(echo $Total_Memory_M /2 | bc)
if (( $SGA_Memory_Size_M>8192 )); then
        # Find out the HugePage size
        HPG_SZ=`grep Hugepagesize /proc/meminfo | awk {'print $2'}`
        # Start from 1 pages to be on the safe side and guarantee 1 free HugePage
        NUM_PG=1
        # Cumulative number of pages required to handle the running shared memory segments
        for SEG_BYTES in `ipcs -m | awk {'print $5'} | grep "[0-9][0-9]*"`
        do
                MIN_PG=`echo "$SEG_BYTES/($HPG_SZ*1024)" | bc -q`
                if [ $MIN_PG -gt 0 ]; then
                        NUM_PG=`echo "$NUM_PG+$MIN_PG+1" | bc -q`
                fi
        done

cat <<EOF >>/etc/sysctl.conf
vm.nr_hugepages=$NUM_PG
EOF

fi
########################################################################
# ORACLE


# script to calculate oracle kernel memory parameters
# Latest one, if Huge pages are required, it displays the apropriate kernel parameter
#
# In case of virtualisation, dmidecode does not report correctly always the memory size
memory=$(lshw -quiet -class memory -short | grep System | awk '{print $3}')
memory_type=$(echo $memory | grep -o '[a-Z]\+')
memory_num=$(echo $memory | grep -o '[0-9]\+')
if [ "$memory_type" == "GiB" ]; then
        Total_Memory_M=$(echo $memory_num*1024 | bc)
        Total_Memory_G=$memory_num
else
        Total_Memory_M=$memory_num
        Total_Memory_G=$(echo $memory_num/1024 | bc)
fi

Page_Size=$(getconf PAGE_SIZE)
shmmax=$(echo $Total_Memory_M *1024*1024/2 | bc)
shmall=$(echo \($Total_Memory_M -2048\)*1024*1024/$Page_Size | bc)


cat <<EOF >>/etc/sysctl.conf
net.ipv4.ip_local_port_range = 9000 65500
fs.file-max = 6815744
kernel.shmall = $shmall
kernel.shmmax = $shmmax
kernel.shmmni = $Page_Size
kernel.sem = 250 32000 100 128
net.core.rmem_default=262144
net.core.wmem_default=262144
net.core.rmem_max=4194304
net.core.wmem_max=1048576
fs.aio-max-nr = 1048576
EOF

# should I implement Huge pages? yes if memory is more than 8GB)
SGA_Memory_Size_M=$(echo $Total_Memory_M /2 | bc)
if (( $SGA_Memory_Size_M>8192 )); then
        # Find out the HugePage size
        HPG_SZ=`grep Hugepagesize /proc/meminfo | awk {'print $2'}`
        # Start from 1 pages to be on the safe side and guarantee 1 free HugePage
        NUM_PG=1
        # Cumulative number of pages required to handle the running shared memory segments
        for SEG_BYTES in `ipcs -m | awk {'print $5'} | grep "[0-9][0-9]*"`
        do
                MIN_PG=`echo "$SEG_BYTES/($HPG_SZ*1024)" | bc -q`
                if [ $MIN_PG -gt 0 ]; then
                        NUM_PG=`echo "$NUM_PG+$MIN_PG+1" | bc -q`
                fi
        done

cat <<EOF >>/etc/sysctl.conf
vm.nr_hugepages=$NUM_PG
EOF

fi

/sbin/sysctl -p
```
This is quite a moutfull. The above script does all the job for us, finds out how much memory we have, and configures accordingly the sysctl parameters. Note that for systems, with more than 16GB of RAM, is advised to use huge tables and the script does so.
```
groupadd oinstall
groupadd dba
useradd -g oinstall -G dba -m -d /home/oracle oracle

cat <<EOF >>/etc/security/limits.conf
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
EOF

cat <<EOF >>/etc/pam.d/login
session required pam_limits.so
EOF
``` 
We need also the aforementioned groups and user for the oracle installation, plus to modify the limits for fliles and processes.
```
########################################################################
# Oracle File Architecture.
# Recomended although you may change it to your liking
# https://docs.oracle.com/cd/B28359_01/install.111/b32002/app_ofa.htm#LADBI446
# CHANGE HERE ORACLE SPECIFIC VARIABLES LIKE SID


cat <<EOF >>/home/oracle/.bashrc
export TMP=/tmp
export TMPDIR=$TMP
export ORACLE_HOSTNAME=$HOSTNAME
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=\$ORACLE_BASE/product/11.2.0/db_1
export ORACLE_SID=ORCL
export NLS_LANG=AMERICAN_AMERICA.EL8MSWIN1253
export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH
export INVENTORY_LOCATION=/home/oracle/oraInventory

# export CLASSPATH=$ORACLE_HOME/jdk/jre:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib

if [ "\$USER" == “oracle” ]; then
        if [ \$SHELL = “/bin/ksh” ]; then
                ulimit –p 16384
                ulimit –n 65536
        else
                ulimit –u 16384 –n 65536
        fi
fi
umask 022
EOF
. /home/oracle/.bashrc

```
We setup the user oracle environmental variables, of where to be located the database and the files. In Oracle home is where the installation will be (we can have multiple installations of different versions), and these ara stores in .bashrc so they will be loaded on oracle run.

So far the installation on bot systems is absolutely the same.


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
