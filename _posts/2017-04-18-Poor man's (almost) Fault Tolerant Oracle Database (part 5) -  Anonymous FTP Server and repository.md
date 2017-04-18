---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 5) -  Anonymous FTP Server and repository
date: 2017-04-18 21:00:00
categories: oracle
comments: true
disqus_identifier: 0011
---

## Anonymous FTP Server

On one server (remember, it may be even the same with DHCP/DNS/TFTP) we must have an anonymous ftp server. This is actually a normal FTP server which allows anonymous logins (to avoid using authentication of username and passwords for something so trivial like a Centos linux repository). We can install the vsftpd which is a known and very efficient server, and configure it accordingly. We must have a directory, lets say /home/public/anon with the appropriate permissions, and also there must be enough free space (several GBs will be needed, don't be skroutz).
```
yum -y install vsftpd
if [!-d /home/public/anon] then; mkdir /home/public/anon; fi
chmod 0500 /home/public/anon
perl -npe '/write_enable/ && s/YES/NO/' -i /etc/vsftpd/vsftpd.conf
chkconfig vsftpd on
service vsftpd start
```

The script above installs the vsftpd, if not already installed, and creates the directory (change to your taste but be sure to change it to TFTP default configuration too) and makes the default configuration of VSFTPD which allows anonymous logins, to not allow uploads by users, for security reasons. Next we make the service startable upon every boot, and start it.

## Creating the Centos repository

We must populate the Anonymous FTP server with the needed files. Firstly we need to download a Centos install locally. Although if we have a fast internet connection, especially via a proxy (which will cache most of our internet downloads), the local repository completely cloned it is more safe and fast option.
We use the rsync way of download, which can be used also in crontab, to update the repository we have to be always the latest, downloading only the needed files and not the whole repository each time.
```
cd /home/public/anon
mkdir centos
cd centos
rsync -rptlv --progress --delete-after --exclude '**/i386/**' --exclude '**/isos/**' --exclude '**/cr/**' --exclude '**/sclo/**' --exclude '**/6.[012345678]/**' --bwlimit=600 ftp.cc.uoc.gr::/centos/6*
cd /home/public/anon
chmod -R 0777 centos
```

Well, what I may have to explain here is the rsync command. The --progress and the v option in the -rptv make the command to be verbose, and you may not want to use them. You shouldn't use them if you put the command in a cron job anyway.

The bwlimit command is to policy the bandwidth used in order not to saturate the internet connection. Use it according to your specifications or ommit completely if not needed. Remember that the number is in KBs (kilobytes per second).

Now for the repository clone, we specifically ommit the 386 option, as we use only the x86_64 architecture.  Also we dont need to download the ISO images, and ofcourse some options (more exist) that also are uneccessary. Lastly, because several sub versions exist in the Centos6, we ommit the previous versions, versions 6.0 thru 6.8 at this point of writing, because now we are at 6.9
In the installation there must be afterwards a soft link, 6, which points always to the latest version.

Use a mirror near you, that has also the rsync option. Here in Greece, I used the mirror at the University of Crete.

## Getting the Oracle installation files

Here things can be quite messy for some. I used the 11.2.0.4 version, which is not available on the OTN but on Metalink. You need to have an account there to download that version, or, you may use the 11.2 version which is available on OTN and modify scripts accordingly.
Due to the nature of these (I mean Metalink), the download is done manually, and the files are placed on the /home/public/anon directory. Do not unzip them, don't do anything else. Of the 7 files that exist for the 11.2.0.4 version (patchset), we need only the first and second. So you may skip all the other zip files.

Perhaps some will say, that it would be better, to unzip them, and provide the files via NFS for example. If that pleases you, you may do it. I tried to keep it as simple as possible. But this kind of setup is a viable alternatice with its merits.

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
