# Using a Raspberry Pi as a DHCP Server
* [What DHCP Server Software to Use? dnsmasq vs isc-dhcp](#what-dhcp-server-software-to-use--dnsmasq-vs-isc-dhcp)
* [Installing ISC DHCP Server on Raspberry Pi](#installing-isc-dhcp-server-on-raspberry-pi)
* [Configuring ISC DHCP Server on Raspberry Pi](#configuring-isc-dhcp-server-on-raspberry-pi)
* [Getting an IP address from the server](#getting-an-ip-address-from-the-server)
* [Configuring a Cisco IOS Device as DHCP Client](#configuring-a-cisco-ios-device-as-dhcp-client)
* [Advanced Configuration: Dynamic DNS Updates from ISC DHCP Server -> Bind](#advanced-configuration--dynamic-dns-updates-from-isc-dhcp-server----bind)
* [Final Thoughts](#final-thoughts)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

One of the most fundamental parts of any network is the allocation of IP addresses.  DHCP is key to the automatic, scalable allocation of IPs in a network.  DHCP initially shows up on the [CCNA certification](https://learningnetwork.cisco.com/s/ccna-exam-topics) where candidates must understand its role in the network as well as configure client and relay services.  You'll find DHCP on professional and expert level certifications as well.

There are many options for hosting a DHCP server in your network lab, and I have probably tried them all.  Some of them include: 

* Many network devices can act as a DHCP server, including Cisco switches and routers
* Consumer grade all-in-one firewall/router/wireless devices provide DHCP services 
* Add a server to your lab and run an enterprise DHCP server such as Microsoft DHCP Server or ICS-DHCP on Linux

Each of the above have pros and cons, and depending on your needs in the lab at a given time one might make more sense than the others.  I've always been a fan of the last option, running a "real" DHCP server because that is what most closely lines up with with what I need to work with on the job.  However, the cost, overhead, and complexity of setting up a server in the lab is often a challenge.  The fan noise alone can get to you eventually.  

That's where the Raspberry Pi comes in as a great option.  As a low cost computer, RPis are inexpensive to add to a lab.  And because it runs Linux, you can install and use the same DHCP server that you might run across in the office.  



## What DHCP Server Software to Use? dnsmasq vs isc-dhcp 
If you do some searching for "DHCP Server for Raspberry Pi", you'll find lots of documentation, blog posts, and forum discussions.  I know because I did it.  Most of them seem to suggest using [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html).  This was a bit of a surprise to me, as I've long used [ISC DHCP](https://www.isc.org/dhcp/) as the DHCP server on Linux systems.  

> Note: The ISC DHCP server is often just called `dhcpd`.

I asked myself, "Why all the love for dnsmasq?"  I'd heard of dnsmasq before, but I hadn't really used it.  Not wanting to just go with what I've always done, I did some digging to see if I could learn more. What I found broke down to three main reasons: 

1. dnsmasq provides both DHCP and DNS services in a single package 
1. dnsmasq is simpler to configure (according to opinions I read, not sure of accuracy here)
1. dnsmasq has a lower cpu and memory footprint than the ISC options of DHCP Server and Bind9 (DNS)

So which do we use for our network lab?  

In the end, it's up to you.  The reasons above are compelling, however I decided to stick with ISC DHCP for my lab.  The main reason for this choice was that I'm much more likely to use ISC DHCP in work environments than dnsmasq, and I'd rather lab what I'm going to use in work.  

> There's a secondary reason that most of the examples and documentation written for DHCP related content like ZTP, phones, etc provide ISC DHCP examples. 

## Installing ISC DHCP Server on Raspberry Pi 
The ISC DHCP Server is available in the standard packages for Raspberry Pi, so installation is like any other Linux server. 


```bash
sudo apt-get install isc-dhcp-server

Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libirs-export161 libisccfg-export163 policycoreutils selinux-utils
Suggested packages:
  isc-dhcp-server-ldap
The following NEW packages will be installed:
  isc-dhcp-server libirs-export161 libisccfg-export163 policycoreutils selinux-utils
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,666 kB of archives.
After this operation, 6,698 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
.
.
.
Setting up isc-dhcp-server (4.4.1-2.3) ...
Generating /etc/default/isc-dhcp-server...
Job for isc-dhcp-server.service failed because the control process exited with error code.
See "systemctl status isc-dhcp-server.service" and "journalctl -xe" for details.
invoke-rc.d: initscript isc-dhcp-server, action "start" failed.
● isc-dhcp-server.service - LSB: DHCP server
     Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
     Active: failed (Result: exit-code) since Sat 2022-02-26 14:30:22 EST; 28ms ago
       Docs: man:systemd-sysv-generator(8)
    Process: 2132 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, status=1/FAILURE)
        CPU: 123ms

Feb 26 14:30:20 lab-server dhcpd[2147]: before submitting a bug.  These pages explain the proper
Feb 26 14:30:20 lab-server dhcpd[2147]: process and the information we find helpful for debugging.
Feb 26 14:30:20 lab-server dhcpd[2147]: 
Feb 26 14:30:20 lab-server dhcpd[2147]: exiting.
Feb 26 14:30:22 lab-server isc-dhcp-server[2132]: Starting ISC DHCPv4 server: dhcpdcheck syslog for diagnostics. ...
Feb 26 14:30:22 lab-server isc-dhcp-server[2160]:  failed!
Feb 26 14:30:22 lab-server isc-dhcp-server[2161]:  failed!
Feb 26 14:30:22 lab-server systemd[1]: isc-dhcp-server.service: Control process exited, code=exited, status=1/FAILURE
Feb 26 14:30:22 lab-server systemd[1]: isc-dhcp-server.service: Failed with result 'exit-code'.
Feb 26 14:30:22 lab-server systemd[1]: Failed to start LSB: DHCP server.
Processing triggers for libc-bin (2.31-13+rpt2+rpi1+deb11u2) ...
Processing triggers for man-db (2.9.4-2) ...
```

The installation process includes setting up a default configuration file and trying to start the server.  This fails on my device because the default configuration file won't work. The output suggests looking at a couple of commands for more detail.  The command `journalctl -xe` is one of my favorite troubleshooting commands for Linux.  It lets you look at the systemd journal, basically all the logs for what is going on.  This will provide the full error messages and logs when a service doesn't start.  

Using this command indicates the problem pretty quickly: 

<details><summary>Output: <strong>journalctl -xe</strong></summary>

```
Feb 26 14:39:02 lab-server systemd[1]: Starting LSB: DHCP server...
░░ Subject: A start job for unit isc-dhcp-server.service has begun execution
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ A start job for unit isc-dhcp-server.service has begun execution.
░░ 
░░ The job identifier is 776.
Feb 26 14:39:02 lab-server isc-dhcp-server[3037]: Launching both IPv4 and IPv6 servers (please configure INTER>
Feb 26 14:39:02 lab-server dhcpd[3052]: Wrote 0 leases to leases file.
Feb 26 14:39:02 lab-server dhcpd[3052]: 
Feb 26 14:39:02 lab-server dhcpd[3052]: No subnet declaration for wlan0 (10.192.81.123).
Feb 26 14:39:02 lab-server dhcpd[3052]: ** Ignoring requests on wlan0.  If this is not what
Feb 26 14:39:02 lab-server dhcpd[3052]:    you want, please write a subnet declaration
Feb 26 14:39:02 lab-server dhcpd[3052]:    in your dhcpd.conf file for the network segment
Feb 26 14:39:02 lab-server dhcpd[3052]:    to which interface wlan0 is attached. **
Feb 26 14:39:02 lab-server dhcpd[3052]: 
Feb 26 14:39:02 lab-server dhcpd[3052]: No subnet declaration for eth0 (192.168.192.11).
Feb 26 14:39:02 lab-server dhcpd[3052]: ** Ignoring requests on eth0.  If this is not what
Feb 26 14:39:02 lab-server dhcpd[3052]:    you want, please write a subnet declaration
Feb 26 14:39:02 lab-server dhcpd[3052]:    in your dhcpd.conf file for the network segment
Feb 26 14:39:02 lab-server dhcpd[3052]:    to which interface eth0 is attached. **
Feb 26 14:39:02 lab-server dhcpd[3052]: 
Feb 26 14:39:02 lab-server dhcpd[3052]: 
Feb 26 14:39:02 lab-server dhcpd[3052]: Not configured to listen on any interfaces!
Feb 26 14:39:02 lab-server dhcpd[3052]: 
Feb 26 14:39:02 lab-server dhcpd[3052]: If you think you have received this message due to a bug rather
Feb 26 14:39:02 lab-server dhcpd[3052]: than a configuration issue please read the section on submitting
Feb 26 14:39:02 lab-server dhcpd[3052]: bugs on either our web page at www.isc.org or in the README file
Feb 26 14:39:02 lab-server dhcpd[3052]: before submitting a bug.  These pages explain the proper
Feb 26 14:39:02 lab-server dhcpd[3052]: process and the information we find helpful for debugging.
Feb 26 14:39:02 lab-server dhcpd[3052]: 
Feb 26 14:39:02 lab-server dhcpd[3052]: exiting.
Feb 26 14:39:04 lab-server isc-dhcp-server[3037]: Starting ISC DHCPv4 server: dhcpdcheck syslog for diagnostic>
Feb 26 14:39:04 lab-server isc-dhcp-server[3057]:  failed!
Feb 26 14:39:04 lab-server isc-dhcp-server[3058]:  failed!
Feb 26 14:39:04 lab-server systemd[1]: isc-dhcp-server.service: Control process exited, code=exited, status=1/>
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ An ExecStart= process belonging to unit isc-dhcp-server.service has exited.
░░ 
░░ The process' exit code is 'exited' and its exit status is 1.
Feb 26 14:39:04 lab-server systemd[1]: isc-dhcp-server.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ The unit isc-dhcp-server.service has entered the 'failed' state with result 'exit-code'.
Feb 26 14:39:04 lab-server systemd[1]: Failed to start LSB: DHCP server.
░░ Subject: A start job for unit isc-dhcp-server.service has failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ A start job for unit isc-dhcp-server.service has finished with a failure.
░░ 
░░ The job identifier is 776 and the job result is failed.
```

</details>

The issue is that there are no subnet declarations for the configured networks found on the server and no interfaces are configured to listen for requests.  

## Configuring ISC DHCP Server on Raspberry Pi 
The odds are that you already have a DHCP server running on your home network.  So you need to be careful to NOT add a second DHCP server and cause problems for the rest of your network. I can attest to the fact that breaking access to video streaming services, social media, and video games for the other people who share your home doesn't go over well.  

With this in mind, I want to setup DHCP on my lab network: 

* Only on the `eth0` interface that is connected to my lab 
* Only on the lab subnet

> For the full configuration guide for ISC DHCP go [here](https://kb.isc.org/docs/isc-dhcp-44-manual-pages-dhcpdconf).  What follows is a basic example configuration for a network lab.

> ISC DHCP is often referred to as `dhcpd`.  This will be used going forward.

1. First up, let's backup the default configuration file that is installed with `dhcpd`.  This file is a great resource for seeing example configurations for common setups.  

    ```bash
    sudo mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.backup
    ```

1. Now create a new file with your desired configuration. 

    ```bash
    # dhcpd.conf

    # DNS Configuration for the network lab
    option domain-name "lab.example";
    option domain-name-servers 192.168.192.11;

    # Setting the default lease time to 2 minutes and max to 1 hour
    default-lease-time 120;
    max-lease-time 3600;

    # This is the subnet on my home network. No configuration is 
    # provided because I don't want to listen/reply to requests
    # but adding it avoids errors/warnings from the service
    subnet 10.192.81.0 netmask 255.255.255.128 {
    }

    # This is my network lab segment configured on eth0
    # A range of IPs to assign is provided, as well as the 
    # default gateway/router
    subnet 192.168.192.0 netmask 255.255.255.0 {
    range 192.168.192.101 192.168.192.120;
    option routers 192.168.192.1;
    }

    # This assigns a static IP address to a lab client device
    host lab-client {
    hardware ethernet b8:27:eb:d3:22:e7;
    fixed-address 192.168.192.12;
    }
    ```

1. In addition to the configuration file you need to indicate the interface on which you want the ISC DHCP server to listen.  This is done in the `/etc/default/isc-dhcp-server` file. Just add `eth0` to the `INTERFACESv4` list.  

    > If you are setting up DHCPv6 ad well, add the interface in that line as well. 

    ```bash
    # On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
    #       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
    INTERFACESv4="eth0"
    INTERFACESv6=""
    ```

1. Now restart the `dhcpd` service to apply the configuration. 

    ```
    sudo systemctl restart isc-dhcp-server
    ```

    > If the service reports errors after making the configuration changes and trying to start the service, you may need to restart the RPi (`sudo shutdown -r now`).  I had this happen in one of my tests. Likely some service interaction/conflict on startup was causing issues. 

1. Check the status of the server. 

    ```
    systemctl status isc-dhcp-server.service
    ● isc-dhcp-server.service - LSB: DHCP server
        Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
        Active: active (running) since Sat 2022-02-26 15:31:40 EST; 17s ago
        Docs: man:systemd-sysv-generator(8)
        Process: 813 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, status=0/SUCCESS)
        Tasks: 4 (limit: 780)
            CPU: 154ms
        CGroup: /system.slice/isc-dhcp-server.service
                └─828 /usr/sbin/dhcpd -4 -q -cf /etc/dhcp/dhcpd.conf eth0

    Feb 26 15:31:37 lab-server systemd[1]: Starting LSB: DHCP server...
    Feb 26 15:31:37 lab-server isc-dhcp-server[813]: Launching IPv4 server only.
    Feb 26 15:31:37 lab-server dhcpd[828]: Wrote 0 deleted host decls to leases file.
    Feb 26 15:31:37 lab-server dhcpd[828]: Wrote 0 new dynamic host decls to leases file.
    Feb 26 15:31:37 lab-server dhcpd[828]: Wrote 0 leases to leases file.
    Feb 26 15:31:37 lab-server dhcpd[828]: Server starting service.
    Feb 26 15:31:40 lab-server isc-dhcp-server[813]: Starting ISC DHCPv4 server: dhcpd.
    Feb 26 15:31:40 lab-server systemd[1]: Started LSB: DHCP server.
    ```

## Getting an IP address from the server 
Now that we have the DHCP Server running on one RPi, let's try to get an IP address assigned to our `lab-client` RPi. 

1. I had configured my `lab-client` RPi for static IP address on its `eth0` interface so my first step was to comment out the static IP configuration in `/etc/dhcpcd.conf`

    ```bash
    # Home Network Lab Setup
    #interface eth0
    #static ip_address=192.168.192.12/24
    #static domain_name_servers=192.168.192.11
    ```

1. To apply the changes to `dhcpcd`, just restart the service. 

    ```bash
    sudo systemctl restart dhcpcd
    ```

1. After a few seconds, you can check to see if you have an address on the interface. 

    ```bash
    ip address show dev eth0

    # Output 
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether b8:27:eb:d3:22:e7 brd ff:ff:ff:ff:ff:ff
        inet 192.168.192.12/24 brd 192.168.192.255 scope global dynamic noprefixroute eth0
        valid_lft 229sec preferred_lft 191sec
        inet6 fe80::1f14:692e:59db:d878/64 scope link 
        valid_lft forever preferred_lft forever
    ```

1. Even better, check the logs/journaly for the dhcpcd client. 

    ```bash
    journalctl -u dhcpcd | grep eth0
    Feb 26 15:35:25 lab-client dhcpcd[4513]: eth0: removing interface
    Feb 26 15:35:26 lab-client dhcpcd[11433]: eth0: IAID eb:d3:22:e7
    Feb 26 15:35:26 lab-client dhcpcd[11433]: eth0: soliciting a DHCP lease
    Feb 26 15:35:26 lab-client dhcpcd[11433]: eth0: offered 192.168.192.12 from 192.168.192.11
    Feb 26 15:35:26 lab-client dhcpcd[11433]: eth0: leased 192.168.192.12 for 300 seconds
    Feb 26 15:35:26 lab-client dhcpcd[11433]: eth0: adding route to 192.168.192.0/24
    Feb 26 15:35:26 lab-client dhcpcd[11433]: eth0: adding default route via 192.168.192.1
    Feb 26 15:35:27 lab-client dhcpcd[11533]: eth0: soliciting an IPv6 router
    Feb 26 15:36:11 lab-client dhcpcd[11533]: eth0: no IPv6 Routers available
    ```

    > `-u dhcpcd` limits the messages from the journal just to the DHCP Client.  And by `grep eth0` we can limit to just messages related to the interface we are interested in. 

1. Over on `lab-server` that is the DHCP Server, we can check the logs to see the view from the server side. 

    ```bash
    journalctl -u isc-dhcp-server -n 20

    # Output 
    -- Journal begins at Thu 2022-01-27 22:15:01 EST, ends at Sat 2022-02-26 15:51:25 EST. --
    Feb 26 15:31:37 lab-server dhcpd[828]: Wrote 0 leases to leases file.
    Feb 26 15:31:37 lab-server dhcpd[828]: Server starting service.
    Feb 26 15:31:40 lab-server isc-dhcp-server[813]: Starting ISC DHCPv4 server: dhcpd.
    Feb 26 15:31:40 lab-server systemd[1]: Started LSB: DHCP server.
    Feb 26 15:35:26 lab-server dhcpd[828]: DHCPDISCOVER from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:35:26 lab-server dhcpd[828]: DHCPOFFER on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:35:26 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 (192.168.192.11) from b8:27:eb:d3:22:e7 >
    Feb 26 15:35:26 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:37:56 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:37:56 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:40:26 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:40:26 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:42:56 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:42:56 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:45:26 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:45:26 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:50:26 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:50:26 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:52:56 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.12 from b8:27:eb:d3:22:e7 via eth0
    Feb 26 15:52:56 lab-server dhcpd[828]: DHCPACK on 192.168.192.12 to b8:27:eb:d3:22:e7 via eth0
    ```

    > `-n 20` limits to just the most recent 20 lines.  This avoids seeing all the messages about service startup. And notice that the `DHCPREQUEST` / `DHCPACK` are coming in every 2 minutes. This is from the shorter lease time we configured.

    > Tip: You can also add a `-f` to the `journalctl` command to "follow" and get updates as they come in. 

## Configuring a Cisco IOS Device as DHCP Client 
This is a networking lab discussion, let's configure an interface on a network device to recieve an IP address from the DHCP server.  

> For this example I am using a Cisco Catalyst 3650 switch (WS-C3650-24TS-S).  The lab network is `vlan 192`.

1. Configure the interface for dhcp

    ```
    lab-switch#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    lab-switch(config)#interface vlan 192
    lab-switch(config-if)#ip address dhcp 
    ```

1. After a few seconds the following console output was logged

    ```
    *Feb 26 20:33:00.206: %DHCP-6-ADDRESS_ASSIGN: Interface Vlan192 assigned DHCP address 192.168.192.101, mask 255.255.255.0, hostname lab-switch
    ```

1. And checking the logs/journal on the DHCP server shows the full `DISCOVER > OFFER > REQUEST > ACK` process.

    ```
    Feb 26 16:00:05 lab-server dhcpd[828]: DHCPDISCOVER from a0:ec:f9:ab:39:d3 via eth0
    Feb 26 16:00:06 lab-server dhcpd[828]: DHCPOFFER on 192.168.192.101 to a0:ec:f9:ab:39:d3 (lab-switch) via eth0
    Feb 26 16:00:06 lab-server dhcpd[828]: DHCPREQUEST for 192.168.192.101 (192.168.192.11) from a0:ec:f9:ab:39:d3 (lab-switch) via eth0
    Feb 26 16:00:06 lab-server dhcpd[828]: DHCPACK on 192.168.192.101 to a0:ec:f9:ab:39:d3 (lab-switch) via eth0
    ```

## Advanced Configuration: Dynamic DNS Updates from ISC DHCP Server -> Bind 
If you are truly going to get the benefits of DHCP and DNS in your lab, you'll likely want DNS to work for clients who receive IP addresses via DHCP as well.  

Checkout the [Dynamic DNS - dhcpd to bind](dynamic-dns.md) guide for details on that setup.

## Final Thoughts 
Now we have a full featured DHCP server running in our lab.  This doc just scratched the surface of DHCP configuration and exploration you might do.  From here somethings you can explore include: 

* DHCP Relay configuration from other VLANs/networks to the server 
* PXE or ZTP configuration using dhcp options 
* IP Phone configuration of settings like TFTP server
* DHCPv6 configuration 
* DHCP debugging 