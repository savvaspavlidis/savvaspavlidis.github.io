---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 1)
date: 2017-02-14 14:00:00
categories: tutorial
---
# Introduction
Despite the fact that today's hardware (and software) is far better than before, more reliable, still there are the cases
that life could be miserable, by hitting a severe hardware (or software) failure.

Some of these can be minimized by having for example RAID disk subsystem, but still, what can you do, when your main production 
server has a motherboard failure? The disks maybe OK (or not), but the whole system is unavailable. When this production 
system is an Oracle Database Server, what choises do you have? (it happaned to us, and indeed was a horror story)

Well, there are plenty of solutions, and the most known (and implemented) are at a hefty price. For example you can have a standby
system, always ready to take action, when the primary hits the ground, and is made with the optional package 
[Oracle Data Guard](https://en.wikipedia.org/wiki/Oracle_Data_Guard). Alas, with a hefty price! First, its available only
with the Oracle Enterprise Database edition. What about those folks that could not afford the Enterprise edition in the first
place? Surely Data Guard is out of the question! And change to some other, in the open source realm, database, like say PostgreSQL
maybe also out of the question if the application used has a lot of the intricacies of Oracle, or logic in the database in 
the form of procedures/triggers and functions plus reports that all would need change. This is also a not a cost-effective solution
in most cases.

So, what is left?

Well, all is not lost. If we can't use DataGuard, then surely we can use some other mechanism to have an ALMOST fault tolerant system. The main idea is to have the main production system, with the database, replicated to another, almost same server (same architecture). It may be of lesser capacity, and in that case, in case of emergency we can allow only the most critical services to be used on this standby system. If we can afford the same hardware, then ofcourse the standby system could handle all the load as the normal production server. The only downside in the case of emergency would be the time to startup the database on the standby system (and do the recovery on files needed, but about this later)

# The Solution

The solution is based on DRBD, a kind of network RAID, and so we can have a replicated, via network, of our database, to another, working system which is available either locally or somewhere remote (for extra security, provided there is the bandwidth needed). DRBD runs under linux, so our servers are base on Linux, and to be more specific, on Centos 6.X which essentially the same as Redhat (the same applies to Oracle Linux and Scientific Linux, which are also almost Redhat). In this tutorial I used Oracle 11.0.2.4 for Linux (available thru Metalink if you have an account there) but the same applies to newer editions. Perhaps later I will try it with Centos 7.X and Oracle 12c.

In order to have an installation that can be applied again, and we need raw iron, not a VM, the solution of a preconfigured VM image for some kind of virtualization, like Vsphere/ESXi or other, was not an option. And this, because there may be problems in a big production server running as a VM. 

To make matter more interesting I made the whole system to install with the minimum of operating system software needed, no graphical, no desktop, because I also wanted to be installable with the minimum of an administrator/dba interference. So the operating system is to be installed via network, by the use of a kickstart file, and the database with a response file, without the use of desktop. This means that a certain infrastructure is required to be available before.

This would be a series of posts for this project, and most of it is already done, but now waits, to be documented here, and refined somehow.

  1. Introduction 
  2. Infrastrucure required
  3. Installation of Operating system via kickstart (unattended)
  4. Installation of Database software via response file (unattended)
  5. DRBD configuration 
  6. Simulating a real disaster - Testing fault tolerance

Wait for the posts. Be patient, as time is scarce resource.

PS. This post(and future posts too) may change in time, because it is also to keep it as a tutorial. 
Any feedback would be highly appreciated. 

<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() {  // REQUIRED CONFIGURATION VARIABLE: EDIT THE SHORTNAME BELOW
        var d = document, s = d.createElement('script');
        
        s.src = '//EXAMPLE.disqus.com/embed.js';  // IMPORTANT: Replace EXAMPLE with your forum shortname!
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

