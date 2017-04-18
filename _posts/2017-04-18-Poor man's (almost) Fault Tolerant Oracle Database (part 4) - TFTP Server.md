---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 4) - TFTP Server
date: 2017-04-18 15:00:00
categories: oracle
comments: true
disqus_identifier: 0010
---

## TFTP Server 
We must have a server to be the TFTP Server. It may be the same as the DHCP/DNS and anonymous FTP Server, the same box. Or separate. I assume the use of Centos 6.X (or other Redhat Linux Enterprise clone). If not installed, we install the tftp server via the following command.
```
yum -y install tftp-server syslinux-tftpboot
```
and will install and all the dependencies. In RHEL7 the TFTP Server runs standalone, but the default behavior in RHEL 6 and clones is to run via the xinetd service. Due to security concerns, we must be carefull of not allowing any other service run via xinetd.
So we go to /etc/xinetd.d directory and change only the tftp file. 
```
perl -npe '/disable/ && s/yes/no/g' -i /etc/xinetd.d/tftp
perl -npe '/server_args/ && s/-s/-c -s/' -i /etc/xinetd.d/ttp
chkconfig xinetd on
service xinetd start
grep disable /etx/xinetd.d/*
```
The first two perl commands make the appropriate changes to the tftp subservice of the xinetd in order to to be started on demand. The chkconfig make the xinetd service startable upon every boot, and the next starts the service, which will run without problems. The last command is just for precaution. We check that no other xinetd subservice will run, and by greping we must see only the tftp to have the disable option to no.

The installation of syslinux-tftpboot make available the needed files for the network boot. Although a bunch of files would be available in the default directory for tftp, /var/lib/tftpboot we basically at this point we need just pxelinux.0 which is the one used for initial network boot.
Some other files will also be needed, by the installation files of Centos, and we will copy them here, plus some configuration.
We need to make a directory for the configuration and inside create the file with the tftp options. Run the following
```
cd /var/run/tftpboot
mkdir pxelinux.cfg
cd pxelinux.cfg
cat <<EOT > default
PROMPT 1
DeFAULT local
DISPLAY messages
TIMEOUT 100

label local
    LOCALBOOT 0

label oratest1
    MENU LABEL oratest1
    KERNEL vmlinuz
    APPEND ks=ftp://myftpserver.example.com/oratest1.ks initrd=initrd.img ramdisk=100000
label oratest2
    MENU LABEL oratest2
    KERNEL vmlinuz
    APPEND ks=ftp://myftpserver.example.com/oratest2.ks initrd=initrd.img ramdisk=100000
EOF
cd ..
chown -R nobody:nobody *
```

Well up to this point, this configuration is to have a system, boot via network, and then proceed to localboot (default) or the user may select to boot with oratest1 or oratest2.

What the oratest1 and oratest2 do? Well it is quite simple, at each option, it says to load the vmlinuz kernel and boot with it, and use the initrd image. The location of the files are on the root of the tftpboot server, which is the /var/lib/tftpboot directory. If you have a lot of images and kernels to boot (I have, even a DOS 6.2 with Norton Ghost!) then you may build a structure with directories inside the /var/lib/tftpboot for easier management and maintanance. As for our case, there is actually just one kernel to boot, we skip that.

but where are these vmlinuz and initrd.img files? We will have them later, when we will clone the whole repository or we may get them directy if needed. For example, for those that can't wait, provided that we have a connection, the following commands will get them (and change owner afterwards).
```
cd /var/lib/tftpboot
rsync -ptv --progress ftp.cc.uoc.gr::centos/6/os/x86_64/isolinux/vmlinuz
rsync -ptv --progress ftp.cc.uoc.gr::centos/6/os/x86_64/isolinux/initrd.img
chown nobody:nobody vlinuz initrd.img
```
This will install these two needed files in place, and modify owner accordingly. In case you are behind a proxy, then you may need to define prior to running rsync, the RSYNC_PROXY environment variable, and perhaps user and password, if authentication to proxy exists.

So far we haven't mentioned what is the other file, which is on the menu. Namely, the ks={something}. Well, this is a file, which does all the magic for the unattended install. It holds all the commands, scripts, and configurations, that we may needed to do on console, and do it all automatically.

This file and all the files of Centos for the install (so no in place technician with a bunch of DVD's or USB sticks may needed) are stored to an FTP server. We may have other options as well, like NFS or even via HTTP. I selected FTP because of the simplicity of it. 



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
