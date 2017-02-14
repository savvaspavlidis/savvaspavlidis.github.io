---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 1)
date: 2017-02-14 14:00:00
categories: tutorial
---
# Introduction
Despite the fact that today's hardware (and software) is far better than before, more reliable, still there are the cases
that life could be miserable, by hitting a severe hardware (or software) failure.
<p>
Some of these can be minimized by having for example RAID disk subsystem, but still, what can you do, when your main production 
server has a motherboard failure? The disks maybe OK (or not), but the whole system is unavailable. When this production 
system is an Oracle Database Server, what choises do you have? (it happaned to us, and indeed was a horror story)
<p>
Well, there are plenty of solutions, and the most known (and implemented) are at a hefty price. For example you can have a standby
system, always ready to take action, when the primary hits the ground, and is made with the optional package 
[Oracle Data Guard](https://en.wikipedia.org/wiki/Oracle_Data_Guard). Alas, with a hefty price! First, its available only
with the Oracle Enterprise Database edition. What about those folks that could not afford the Enterprise edition in the first
place? Surely Data Guard is out of the question! And change to some other, in the open source realm, database, like say PostgreSQL
maybe also out of the question if the application used has a lot of the intricacies of Oracle, or logic in the database in 
the form of procedures/triggers and functions plus reports that all would need change. This is also a not a cost-effective solution
in most cases.
<p>
So, what is left?
<p>
Well, all is not lost. If we can't use DataGuard, then surely we can use some other mechanism to have an ALMOST
fault tolerant system. The main idea is to have the main production system, with the database, replicated to another, almost
same server (same architecture). It may be of lesser capacity, because in case of emergency we can allow only the most critical
services to be used on this standby system. If we can afford the same hardware, then ofcourse the standby system could handle 
all the load. The only downside in the case of emergency would be the time to startup the database on the standby system (and 
do the recovery on files needed, but about this later)
<p>
# The Solution
This is a project that it will be covered in a series of posts. Our server is a standalone, not a VM machine. Our servers are
build on Linux Operating System (64bit), and specifically Centos 6.X (Redhat clone). 

1. Introduction 
2. Infrastrucure required
3. Installation of Operating system via kickstart (unattended)
4. Installation of Database software via response file (unattended)
5. DRBD configuration 
6. Simulating a real disaster - Testing fault tolerance
<p>
Wait for the posts. Be patient, as time is scarce resource.
<p>
PS. This post(and future posts too) may change in time, because it is also to keep it as a tutorial. 
Any feedback would be highly appreciated. 
