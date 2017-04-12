---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 2) - Infrastructure
date: 2017-02-28 14:00:00
categories: oracle
comments: true
disqus_identifier: 0003
---

## Infrastructure
In order to have the ability to do an (almost complete unattended installation of operation system and installation of the oracla software plus all the configurations needed, there must be some preparation ahead of time.

For unattended install, the system must boot thru network, and install everything from network. Thus we need first of all to have a DHCP Server configured for this, not only to give an IP and a subnet mask and a gateway to its clients, but something more. It should give the capability of booting via PXE (network). This can not be accomplished by cheap adsl routers. In our case a linux server is configured accordingly.

Upon the boot process via network the DHCP server instructs the machine to use files on a server with a TFTP server, for booting to start. Thus we need a properly configured TFTP server with the appropriate files. From this server the boot image is loaded and the installation will begin. Here it is configured where to read the kickstart file and where is the repository with the operating system files to install. The kickstart file is a script file that holds the instructions in a textual form of all the options (and more) in order to do the installation.

As the previous instruction of booting, we need a place for the repository and for the kickstart file. There are multiple ways to achieve that, but surely we need a local place for the kickstart file. The repository may be on internet or may be available locallay for faster "download" to install. For a locally available repository, we need first to clone a known one repository and make available the repository and the kickstart file via anonymous FTP (could be also done via NFS, HTTP etc). We use the Anonymous FTP Server not only for the repository and the kickstart file, but also for the Oracle installation files.

In conclusion we need.
* A properly configured DHCP Server 
* A properly configured TFTP Server for PXE boot
* An Anonymous FTP Server making available the repository, kickstart file and Oracle installation files
* two servers, eiter raw iron or VM based, 64bit as this tutorial is for 64bit only installation.



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
