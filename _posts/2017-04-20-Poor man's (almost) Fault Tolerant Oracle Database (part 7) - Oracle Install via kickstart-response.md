
---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 7) -  Oracle Install via kickstart-response
date: 2017-04-19 09:00:00
categories: oracle
comments: true
disqus_identifier: 0013
---

## Enabling DRBD
There is one difference from the previous chapter explaining the kickstart file. In the master there is no declaration of the DRBD disk as secondary, but the opposite. we declare it as a primary. And afterwards we make a file system on it, and proceed with the installation of Oracle software, which will be replicated either now, or afterwards (preferably the latter). It is assumed that the secondary slave (failover) system has already run the installation and is already up. All the installation will be, again, in the kickstart file, although, it will run on the next boot, after the finish of the postinstall phase.

instead of the command
```
/sbin/drbdadm secondary disk1
```
we should have the following
```
/sbin/drbdadm -- --overwrite-data-of-peer primary disk1
mkfs.ext4 /dev/drbd1
if [ ! -d /u01 ]; then
        mkdir /u01
fi
mount /dev/drbd1 /u01
cat <<EOT >>/etc/fstab
/dev/drbd1       /u01                   ext4    defaults,,noatime,nodiratime        1 2
EOT

# if no mount point created thru partitioning, then make the directory
# apply owner
if [ ! -d $ORACLE_BASE ]; then
        mkdir -p $ORACLE_BASE
fi
chown -R oracle:oinstall $(echo $ORACLE_BASE | cut -f1,2 --delimiter='/')

```
these commands make available firstly the DRBD device, and so whatever it is written there, it would be replicated to the corresponding device in secondary system. We make an ext4 filesystem and create the structure with the permissions for Oracle to be installed according to OFA. Next will be the download of the software and the installation of it. Remember, everything is on this kickstart file, including the response file for the unattended oracle installation.
```
# get Oracle Installation files from the anonymous FTP Server
# unzip installer files..
cd /home/oracle
wget --no-proxy ftp://$ANONFTPSERVER/public/p13390677_112040_Linux-x86-64_1of7.zip
wget --no-proxy ftp://$ANONFTPSERVER/public/p13390677_112040_Linux-x86-64_2of7.zip
unzip p13390677_112040_Linux-x86-64_1of7.zip
unzip p13390677_112040_Linux-x86-64_2of7.zip
rm -f p13390677_112040_Linux-x86-64_?of7.zip
```
First the easy part. We download and unzip the software. remove to keep some space the zips.

```
# change the cv/admin/config so it will default to OEL6 (and not OEL4). Needed because otherwise it will request pdksh.
# and fail install
perl -npe '/^CV_ASSUME_DISTID/ && s/OEL4/OEL6/' -i database/stage/cvu/cv/admin/cvu_config
```
Now, Oracle 11G has some trouble, when installed unattended, because it thinks that the default is RHEL4 (OEL4 ror Centos4 respectively) and in that version it requires the pdksh (which now in version 6, is ksh, the Korn Shell). So we change what it thinks is the default version, from 4 to 6.
```
# make the oracle software install response file
cat <<EOF > /home/oracle/db_install.rsp
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=oratest1.example.com
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/home/oracle/oraInventory
SELECTED_LANGUAGES=en,el
ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=SE
oracle.install.db.EEOptionsSelection=false
oracle.install.db.optionalComponents=
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
oracle.install.db.CLUSTER_NODES=
oracle.install.db.isRACOneInstall=false
oracle.install.db.racOneServiceName=
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=
oracle.install.db.config.starterdb.SID=
oracle.install.db.config.starterdb.characterSet=
oracle.install.db.config.starterdb.memoryOption=false
oracle.install.db.config.starterdb.memoryLimit=
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.password.ALL=
oracle.install.db.config.starterdb.password.SYS=
oracle.install.db.config.starterdb.password.SYSTEM=
oracle.install.db.config.starterdb.password.SYSMAN=
oracle.install.db.config.starterdb.password.DBSNMP=
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.gridcontrol.gridControlServiceURL=
oracle.install.db.config.starterdb.automatedBackup.enable=false
oracle.install.db.config.starterdb.automatedBackup.osuid=
oracle.install.db.config.starterdb.automatedBackup.ospwd=
oracle.install.db.config.starterdb.storageType=
oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=
oracle.install.db.config.starterdb.fileSystemStorage.recoveryLocation=
oracle.install.db.config.asm.diskGroup=
oracle.install.db.config.asm.ASMSNMPPassword=
MYORACLESUPPORT_USERNAME=
MYORACLESUPPORT_PASSWORD=
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true
PROXY_HOST=
PROXY_PORT=
PROXY_USER=
PROXY_PWD=
PROXY_REALM=
COLLECTOR_SUPPORTHUB_URL=
oracle.installer.autoupdates.option=SKIP_UPDATES
oracle.installer.autoupdates.downloadUpdatesLoc=
AUTOUPDATES_MYORACLESUPPORT_USERNAME=
AUTOUPDATES_MYORACLESUPPORT_PASSWORD=
EOF
```
All the above is the Oracle response file, for unattended install. We may also do it on the command line, but it is clearer this way I think, for maintance. Now in the home of the oracle user there is the response file ready to be used.
```
cat <<EOF > /home/oracle/oraInst.loc
inventory_loc=/home/oracle/oraInventory
inst_group=oinstall
EOF

chown -R oracle:oinstall /home/oracle
```
make the necessary permissions, because everything that we had done under root, have the root as owner.
```
########################################################################
# Make the oracle installation script as the first to run on next boot
#
cat <<EOZ > /root/oracle-install.bash
#!/bin/bash
# install the database software
# firstly, exclude it from rc.local, so it would not run again in subsequent boots
cat /etc/rc.d/rc.local | grep -v "oracle-install.bash" > /etc/rc.d/rc.local
echo "*********************************************************************************************************************"
echo "JUST BEFORE THE ORACLE INSTALLER"
su - oracle -c "/home/oracle/database/runInstaller -silent -responseFile /home/oracle/db_install.rsp -waitforcompletion"
echo "JUST AFTER THE ORACLE INSTALLER"

if [ $(tail -1 $INVENTORY_LOCATION/logs/silent*) == "The installation of Oracle Database 11g was successful." ]; then
        echo "Running $ORACLE_HOME/root.sh"
        $ORACLE_HOME/root.sh
fi
```
Now, the difficult part, at least what kept me some long hours to figure it out.The oracle installer wants to run in runlevel 3 to 5. In runlevel 3, with no desktop, is where you can do the installation with the response file. But when in postinstaller phase, there is no explicit runlevel, because it has not booted yet as a system. Thus, NO RUNLEVEL.

No Runlevel, means the installer cannot run. Perhaps it would run, if I explcitly requested to ignore all these dependencies on the system, but then we would not know if something went wrong. I tried to fool the installer, but with no success. Finally, the only way I could do it, is to make the system run the installer immediately upon reboot, like the firsboot. This is done, by pacing the commands and scripts I want in rc.local, which will run, after reboot. A caution must be made, that upon running, it should remove that line, so it would not run every time.
```
########################################################################
# Make the oracle installation script as the first to run on next boot
#
cat <<EOZ > /root/oracle-install.bash
#!/bin/bash
# install the database software
# firstly, exclude it from rc.local, so it would not run again in subsequent boots
cat /etc/rc.d/rc.local | grep -v "oracle-install.bash" > /etc/rc.d/rc.local
echo "*********************************************************************************************************************"
echo "JUST BEFORE THE ORACLE INSTALLER"
su - oracle -c "/home/oracle/database/runInstaller -silent -responseFile /home/oracle/db_install.rsp -waitforcompletion"
echo "JUST AFTER THE ORACLE INSTALLER"

if [ $(tail -1 $INVENTORY_LOCATION/logs/silent*) == "The installation of Oracle Database 11g was successful." ]; then
        echo "Running $ORACLE_HOME/root.sh"
        $ORACLE_HOME/root.sh
fi

# configure the network listener (netca)

# configure the database
EOZ

chmod +x /root/oracle-install.bash


#Now place the script for oracle install in the next boot of system...
cat <<EOF >> /etc/rc.local
ntpdate time.windows.com
/root/oracle-install.bash
EOF
```


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
