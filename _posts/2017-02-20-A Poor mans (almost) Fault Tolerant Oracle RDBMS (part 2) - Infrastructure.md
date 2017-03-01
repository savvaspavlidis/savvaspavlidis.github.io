---
layout: default
title: Poor man's (almost) Fault Tolerant Oracle Database (part 2) - Infrastructure
date: 2017-02-28 14:00:00
categories: oracle
comments: true
disqus_identifier: 0003
---

In order to have a system that would boot and install itself with the minimum of a sysadmin intervention, we would need:

1. DHCP Server
2. TFTP Server
3. An anonymous FTP (or HTTP Server)

## DHCP Server & DNS Server

The reason behind this setup is to have the capability to boot and install the system, even wheh we are not present at the machine (someone might be needed though). 

Someone is needed to press the button to start the machine (there is always ofcourse the option of WOL, or if you can have a system that can be managed sideways, for example with iLO or IBM's, that's even better). There is a catch here, that I will explain later. What we need is to start via network (PXE BOOT). Either the machine is configured already to boot first by network (in BIOS, or by remote management), or someone there would select the boot option device manually.

Thus the machine should boot by network as a first option and this should have been configured in BIOS. It adds about half a minute in the boot procedure, but we don't boot very often our linux machines, dont we?
First Step. The machine should be configured to boot first by network (to avoid the case that it has an already bootable hard disk or some other media (bootable CD/DVD etc)

So a DHCP server is needed, and the dhcp server in small routers like home ADSL routers would not suffice. Also a DNS server is needed, mostyly because the Oracle requires a static IP and to be resolved via DNS, otherwise will complain and not install (or install via ignoring system prerequisites).

As we are working with Linux systems surely the most easy way is to make a DHCP Server on a Linux box. The same applies for he DNS Server, and may be on the same machine. Because as I said before we need a static IP address to be resolved via DNS, this means that we should instruct the DHCP Server likewise. We must have the MAC Address of the ethernet port, and on the dhcpd.conf file (/etc/dhcpd/dhcpd.conf on RHEL like systems) we put the following fragment in the pool
'''
                host oratest1.the.yalco.gr {
                        hardware ethernet 08:00:27:61:96:bc;
                        fixed-address 10.1.1.129;
                        ddns-hostname "oratest1";
                }
'''


[tutorial](https://tecadmin.net/configuring-dhcp-server-on-centos-redhat/#)

## TFTP

The need for a TFTP server is because immediately after booting from PXE, from here it will download the needed parts to proceed to the actual install.
I am using a Centos server, so this [tutorial](http://www.bo-yang.net/2015/08/31/centos7-install-tftp-server) might be of help for everyone who would like to setup a TFTP Server. 
