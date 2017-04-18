---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 5) -  OS Install via kickstart
date: 2017-04-19 21:00:00
categories: oracle
comments: true
disqus_identifier: 0012
---

## OS Install via kickstart file (unattended)

First of all, I should note that we have  to prepare two systems, the master and the failover. I use a kind of unattended install with a file that has all the selections that normally an administrator would select on console. But not only that, this file gives us the capability to write and execute scripts in bash (or any other language) that would modify and do all the necessary changes without intervention.

So we have two servers, oratest1 (the master) and oratest2 (the failover). A large part of the kickstart file is almost the same, and the only difference is that the install of the oracle software is done on oratest1. And this is done, because it would be replicated and mirrored via the DRBD, as all the files needed for the Oracle RDBMS, and should be the same on both systems. The specifics of the Oracle installation would be covered in the next chapter. It should be noted that oratest1 and oratest2 for this tutorial are systems with two hard disks, sda and sdb, with same capacity (24GB and 16GB respectively). This can be changed according to your specifications. Also RAM should at least 4GB for each system.

The kickstart file has actually two parts. The first part, is like the selection we do to anaconda, for the installation of the Operating system. The second part, the post-installer script, is actually using bash, so we can right in bash (and in other script languanges like perl or python) everything needed to modify the verbatim installation to our specifications.






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
