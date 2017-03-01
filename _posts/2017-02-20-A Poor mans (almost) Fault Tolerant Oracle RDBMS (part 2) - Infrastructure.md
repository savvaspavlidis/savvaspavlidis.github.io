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

## DHCP Server 

The reason behind this setup is to have the capability to boot and install the system, even wheh we are not present at the machine (someone might be needed though). 

Someone is needed to press the button to start the machine (there is always ofcourse the option of WOL, or if you can have a system that can be managed sideways, for example with iLO or IBM's, that's even better). There is a catch here, that I will explain later. What we need is to start via network (PXE BOOT). Either the machine is configured already to boot first by network (in BIOS, or by remote management), or someone there would select the boot option device manually.

Thus the machine should boot by network as a first option and this should have been configured in BIOS. It adds about half a minute in the boot procedure, but we don't boot very often our linux machines, dont we?
First Step. The machine should be configured to boot first by network (to avoid the case that it has an already bootable hard disk or some other media (bootable CD/DVD etc)

So a DHCP server is needed, and the dhcp server in small routers like home ADSL routers would not suffice. Also a DNS server is needed, mostyly because the Oracle requires a static IP and to be resolved via DNS, otherwise will complain and not install (or install via ignoring system prerequisites).

As we are working with Linux systems surely the most easy way is to make a DHCP Server on a Linux box. The same applies for he DNS Server, and may be on the same machine. Because as I said before we need a static IP address to be resolved via DNS, this means that we should instruct the DHCP Server likewise. We must have the MAC Address of the ethernet port, and on the dhcpd.conf file (/etc/dhcpd/dhcpd.conf on RHEL like systems) we put the following fragment in the pool.

Here in our example, the server is oratest1, the domain is example.com, our subnet is 10.1.1.0/24, and the server's oratest1 IP should be 10.1.1.129

                host oratest1.example.com {
                        hardware ethernet 08:00:27:61:96:bc;
                        fixed-address 10.1.1.129;
                        ddns-hostname "oratest1";
                }
                

Also in the dhcpd.conf file we should give the following directives, before the enclosing parenthesis of the dhcpd.conf for the pool that instructs whenever some machine wants to boot via PXE, where is the next server, to load the files needed (it may be the same machine as the DHCP/DNS Server). The next machine, needs to provide TFTP service and the needed files, described next.

        filename "pxelinux.0";
        next-server 10.1.1.10;

A more complete simpe example dhcpd.conf file, that it may be needed to change according to your needs is provided below 

~~~
    authoritative;
    use-host-decl-names             on;
    default-lease-time              7200;
    max-lease-time                  7200;
    option subnet-mask              255.255.255.0;
    option broadcast-address        10.1.1.255;
    option domain-name-servers      10.1.1.6;
    option domain-name              "example.com";
    option netbios-node-type        8;
    option netbios-name-servers     10.1.1.10;
    ddns-updates                    on;
    ignore client-updates;
    ignore declines;
    ddns-domainname                 "example.com";
    ddns-rev-domainname            "1.1.10.in-addr.arpa";
    ddns-update-style               interim;
    update-static-leases            on;
    allow booting;
    allow bootp;

    subnet 10.1.1.0 netmask 255.255.255.0 {
        range                           10.1.1.50 10.1.1.254;

        group {
                host oratest1.example.com {
                        hardware ethernet 08:00:27:61:96:bc;
                        fixed-address 10.1.1.129;
                        ddns-hostname "oratest1";
                }
        }
        range dynamic-bootp 10.1.1.45 10.1.1.49;
        filename "pxelinux.0";
        next-server 10.1.1.10;
  }       
~~~

Visit this [tutorial](https://tecadmin.net/configuring-dhcp-server-on-centos-redhat/#) for a simple installation and configuration on RHEL clone systems of a DHCP server. 

## DNS Server

In our case, there is a need for DNS Server, as the Oracle installer checks via DNS if the system can be looked up via DNS queries. Although you can skip it, it is not recommended. The DNS server must be authoritative. A simpe authorative solution that accepts requests from the DHCP server would suffice. For a tutorial about installing and configuring a simple DNS Server, you may check this.


## TFTP

The need for a TFTP server is because immediately after booting from PXE, from here it will download the needed parts to proceed to the actual boot (and install). By booting via network with PXE, its like to boot with a CD, only it exists somewhere in the network. This makes maintenance easier, because you can make a remote boot, and have all needed CD's for maintenance or installing operating system available thru network.

I am using a Centos server, so this [tutorial](http://www.bo-yang.net/2015/08/31/centos7-install-tftp-server) might be of help for everyone who would like to setup a TFTP Server. I have a number of bootable images, GHOST (remember DOS Ghost?), UBCD, Clonezilla, Acronis etc, but I will assume that this is your first attempt, so there would be only the Centos install image. 
