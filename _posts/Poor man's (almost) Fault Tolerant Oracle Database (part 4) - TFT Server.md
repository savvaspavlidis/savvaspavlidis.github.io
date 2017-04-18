---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 4) - TFT Server
date: 2017-04-18 15:00:00
categories: oracle
comments: true
disqus_identifier: 0010
---

## TFTP Server 
We must have a server to be the TFTP Server. It may be the same as the DHCP/DNS and anonymous FTP Server, the same box. Or separate. I assume the use of Centos 6.X (or other Redhat Linux Enterprise clone). If not installed, we install the tftp server via the following command.
```
yum -y install tftp-server
```
and will install and all the dependencies. In RHEL7 the TFTP Server runs standalone, but the default behavior in RHEL 6 and clones is to run via the xinetd service. Due to security concerns, we must be carefull of not allowing any other service run via xinetd.


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
