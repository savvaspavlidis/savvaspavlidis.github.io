---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 4) - DNS Server
date: 2017-04-12 15:00:00
categories: oracle
comments: true
disqus_identifier: 0009
---

## DNS Server 

In our case, there is a need for DNS Server, as the Oracle installer checks via DNS if the system can be looked up via DNS queries. Although you can skip it, it is not recommended. The DNS server must be authoritative. A simpe authorative solution that accepts requests from the DHCP server would suffice. For a tutorial about installing and configuring a simple DNS Server, you may check this.


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
