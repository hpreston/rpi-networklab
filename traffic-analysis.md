# Traffic Analysis and Capture
- [tcpdump - The basic network packet analyzer](#tcpdump---the-basic-network-packet-analyzer)
  * [Installing tcpdump](#installing-tcpdump)
  * [Viewing "normal" traffic to/from the RPi](#viewing--normal--traffic-to-from-the-rpi)
  * [`tcpdump` "verbosity"](#-tcpdump---verbosity-)
- [Span / Monitor Sessions on a Switch](#span---monitor-sessions-on-a-switch)
  * [Configuring the Monitor Session](#configuring-the-monitor-session)
  * [Capture Example 1: DNS Resolution](#capture-example-1--dns-resolution)
  * [Capture Example 2: Telnet](#capture-example-2--telnet)
  * [Capture Example 3: Creating a PCAP file](#capture-example-3--creating-a-pcap-file)
- [Final Thoughts](#final-thoughts)
  * [What about Wireshark?](#what-about-wireshark-)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


An often repeated comment in network engineering is "packets don't lie", or maybe the request to "show me the pcap".  There is nothing quite like looking at the packets as they flow across the network to help you understand how things work.  And a great place to begin explore this side of network engineering is in your home lab.  

As an inexpensive computer, a Raspberry Pi makes a great network sensor.  In this guide we'll look at how we can get started quickly with this exploration.  

> Note: Entire courses, books, videos, etc exist on the topic of packet analysis.  This short guide is not meant to replace them, but rather show a quick example of how to get started on your Raspberry Pi. 

# tcpdump - The basic network packet analyzer 
tcpdump is a command line packet analyzer that is great for packet analysis work because it is small, lightweight, and available nearly anywhere Linux exists.  With tcpdump you can monitor the traffic seen by your computer with powerful filters to drill in and view only the data you want.  Filtering is important because network, particularly production networks, are VERY busy.  And even a lightly used lab network may shock you with how much traffic is happening.  

Some examples of ways filters can be useful include: 

* Capturing traffic with specific source or destination IP addresses or MAC addresses 
* Capturing traffic from certain TCP or UDP ports 
* Capture traffic based on network/mask matching 
* Capturing all broadcast or multicast traffic 
* Capturing packets of a certain size

## Installing tcpdump 
tcpdump can be installed like most other Linux programs, using `apt-get`. 

```bash
sudo apt-get install tcpdump
```

<details><summary>Output: Install tcpdump</summary>

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libpcap0.8
Suggested packages:
  apparmor
The following NEW packages will be installed:
  libpcap0.8 tcpdump
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 588 kB of archives.
After this operation, 1,740 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://deb.debian.org/debian bullseye/main arm64 libpcap0.8 arm64 1.10.0-2 [151 kB]
Get:2 http://deb.debian.org/debian bullseye/main arm64 tcpdump arm64 4.99.0-2 [437 kB]
Fetched 588 kB in 1s (762 kB/s)  
Selecting previously unselected package libpcap0.8:arm64.
(Reading database ... 35664 files and directories currently installed.)
Preparing to unpack .../libpcap0.8_1.10.0-2_arm64.deb ...
Unpacking libpcap0.8:arm64 (1.10.0-2) ...
Selecting previously unselected package tcpdump.
Preparing to unpack .../tcpdump_4.99.0-2_arm64.deb ...
Unpacking tcpdump (4.99.0-2) ...
Setting up libpcap0.8:arm64 (1.10.0-2) ...
Setting up tcpdump (4.99.0-2) ...
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.31-13+rpt2+rpi1+deb11u2) ...
```

</details>

Once installed, you can check the version and basic help info for the command. 

```bash
tcpdump --help

# Output
tcpdump version 4.99.0
libpcap version 1.10.0 (with TPACKET_V3)
OpenSSL 1.1.1k  25 Mar 2021
Usage: tcpdump [-AbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ] [--count]
                [ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
                [ -i interface ] [ --immediate-mode ] [ -j tstamptype ]
                [ -M secret ] [ --number ] [ --print ] [ -Q in|out|inout ]
                [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
                [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
                [ --time-stamp-precision precision ] [ --micro ] [ --nano ]
                [ -z postrotate-command ] [ -Z user ] [ expression ]
```

> For full details on using tcpdump, you can explore `man tcpdump`, or search online for one of many great Introductions/Tutorials on tcpdump
>
> There are are also many "tcpdump cheatsheets" like [this one by Jeremy Stretch (of NetBox fame)](https://packetlife.net/media/library/12/tcpdump.pdf)

## Viewing "normal" traffic to/from the RPi 
For our first examples, we will look at traffic reaching the RPi without any special network configuration on the switch.  This is simply what is being sent to the RPi as a regular host on the network.  

First up, you will often want to limit tcpdump to specific interfaces on the computer.  Let's see what interfaces are available to tcpdump. 

```bash
tcpdump -D

# Output
1.eth0 [Up, Running, Connected]
2.wlan0 [Up, Running, Wireless]
3.any (Pseudo-device that captures on all interfaces) [Up, Running]
4.lo [Up, Running, Loopback]
5.bluetooth0 (Bluetooth adapter number 0) [Wireless, Association status unknown]
6.bluetooth-monitor (Bluetooth Linux Monitor) [Wireless]
7.nflog (Linux netfilter log (NFLOG) interface) [none]
8.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
9.dbus-system (D-Bus system bus) [none]
10.dbus-session (D-Bus session bus) [none]
``` 

See anything surprising?  The `eth0` and `wlan0` interfaces are probably expected, but look at all those other options you can explore later.  

We'll stick with `eth0` for our exploration as that is the one connected to the network-lab. 

We'll start simple listen command. 

```bash
tcpdump -i eth0

# Output
tcpdump: eth0: You do not have permission to capture on that device
(socket: Operation not permitted)
```

***Important: Capturing traffic from a network interface requires admin rights.  So you'll need to `sudo` it.***

```bash
sudo tcpdump -i eth0
```

> Note: Stop the capture by pressing `Cntl-c`

***Output***

```text
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
20:29:18.374804 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:20.375247 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:20.792884 Loopback, skipCount 0, Reply, receipt number 0, data (40 octets)
20:29:22.376125 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:24.377202 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:26.378899 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:28.380114 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:30.381316 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:30.795250 Loopback, skipCount 0, Reply, receipt number 0, data (40 octets)
20:29:32.381832 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:34.382630 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:36.385373 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:38.385190 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:40.385632 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:40.794914 Loopback, skipCount 0, Reply, receipt number 0, data (40 octets)
20:29:42.386416 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:44.386368 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:45.337471 IP 192.168.192.11 > 192.168.192.102: ICMP echo request, id 1, seq 1, length 64
20:29:45.337589 IP 192.168.192.102 > 192.168.192.11: ICMP echo reply, id 1, seq 1, length 64
20:29:46.338820 IP 192.168.192.11 > 192.168.192.102: ICMP echo request, id 1, seq 2, length 64
20:29:46.338915 IP 192.168.192.102 > 192.168.192.11: ICMP echo reply, id 1, seq 2, length 64
20:29:46.386796 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:48.388350 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:50.387681 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
20:29:50.546748 ARP, Request who-has 192.168.192.102 tell 192.168.192.11, length 46
20:29:50.546810 ARP, Reply 192.168.192.102 is-at b8:27:eb:d3:22:e7 (oui Unknown), length 28
20:29:50.556861 ARP, Request who-has 192.168.192.11 tell 192.168.192.102, length 28
20:29:50.557542 ARP, Reply 192.168.192.11 is-at b8:27:eb:18:a6:8c (oui Unknown), length 46
20:29:50.792716 Loopback, skipCount 0, Reply, receipt number 0, data (40 octets)
20:29:52.389203 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 80c0.a0:ec:f9:ab:39:80.800b, length 36
^C
30 packets captured
30 packets received by filter
0 packets dropped by kernel
```

Check that out!  What can we see in the output

1. Spanning-tree BPDU's from the network switch every 2 seconds 
1. Some loopback messages {shrug}
1. ICMP echo requests and replys
    * While the capture was running I sent 2 `pings` from the other RPi in the lab to generate some traffic
1. ARP traffic from the other RPi requesting MAC address for this RPi's IP address 

Not to bad for a first capture.  But you might be wondering... "is that all that is on the network"?  The answer is no.  

## `tcpdump` "verbosity"

By default tcpdump provides only certain details on the "dump line" for each packet.  But you can adjust this detail with options.  A basic one is the "verbosity option" which is `-v`.  It has three levels of increasing verbosity.  

Let's see the difference in output for DHCP traffic to compare. 

> We will use a `port` based filter to select the DHCP traffic.  We'll grab any traffic on `port 67 or port 68`.  
> 
> Filters are referred to as "expressions" and can be crafted to be very specific for the type of traffic you are interested in.

* Single level of verbosity 

    <details><summary>sudo tcpdump -i eth0 -v port 67 or port 68</summary>

    ```
    tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
    20:46:02.181634 IP (tos 0x0, ttl 64, id 40790, offset 0, flags [DF], proto UDP (17), length 328)
        192.168.192.102.bootpc > 192.168.192.11.bootps: BOOTP/DHCP, Request from b8:27:eb:d3:22:e7 (oui Unknown), length 300, xid 0xde321e7e, secs 65535, Flags [none]
            Client-IP 192.168.192.102
            Client-Ethernet-Address b8:27:eb:d3:22:e7 (oui Unknown)
            Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message (53), length 1: Request
                Client-ID (61), length 7: ether b8:27:eb:d3:22:e7
                MSZ (57), length 2: 1472
                Hostname (12), length 10: "lab-client"
                Unknown (145), length 1: 1
                Parameter-Request (55), length 14: 
                Subnet-Mask (1), Classless-Static-Route (121), Static-Route (33), Default-Gateway (3)
                Domain-Name-Server (6), Hostname (12), Domain-Name (15), MTU (26)
                BR (28), Lease-Time (51), Server-ID (54), RN (58)
                RB (59), Unknown (119)
    20:46:02.194360 IP (tos 0x0, ttl 64, id 39778, offset 0, flags [DF], proto UDP (17), length 328)
        192.168.192.11.bootps > 192.168.192.102.bootpc: BOOTP/DHCP, Reply, length 300, xid 0xde321e7e, secs 65535, Flags [none]
            Client-IP 192.168.192.102
            Your-IP 192.168.192.102
            Client-Ethernet-Address b8:27:eb:d3:22:e7 (oui Unknown)
            Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message (53), length 1: ACK
                Server-ID (54), length 4: 192.168.192.11
                Lease-Time (51), length 4: 300
                Subnet-Mask (1), length 4: 255.255.255.0
                Default-Gateway (3), length 4: 192.168.192.1
                Domain-Name-Server (6), length 4: 192.168.192.11
                Domain-Name (15), length 11: "lab.example"

    ^C
    2 packets captured
    2 packets received by filter
    0 packets dropped by kernel
    ```

    </details>

* Double level of verbosity 

    <details><summary>sudo tcpdump -i -vv port 67 or port 68 </summary>

    ```
    tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
    20:46:58.728952 IP (tos 0x0, ttl 64, id 16856, offset 0, flags [none], proto UDP (17), length 328)
        192.168.192.102.bootpc > 255.255.255.255.bootps: [udp sum ok] BOOTP/DHCP, Request from b8:27:eb:d3:22:e7 (oui Unknown), length 300, xid 0xe4ccbcfa, Flags [none] (0x0000)
            Client-Ethernet-Address b8:27:eb:d3:22:e7 (oui Unknown)
            Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message (53), length 1: Request
                Client-ID (61), length 7: ether b8:27:eb:d3:22:e7
                Requested-IP (50), length 4: 192.168.192.102
                MSZ (57), length 2: 1472
                Hostname (12), length 10: "lab-client"
                Unknown (145), length 1: 1
                Parameter-Request (55), length 14: 
                Subnet-Mask (1), Classless-Static-Route (121), Static-Route (33), Default-Gateway (3)
                Domain-Name-Server (6), Hostname (12), Domain-Name (15), MTU (26)
                BR (28), Lease-Time (51), Server-ID (54), RN (58)
                RB (59), Unknown (119)
    20:46:58.787673 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
        192.168.192.11.bootps > 192.168.192.102.bootpc: [udp sum ok] BOOTP/DHCP, Reply, length 300, xid 0xe4ccbcfa, Flags [none] (0x0000)
            Your-IP 192.168.192.102
            Client-Ethernet-Address b8:27:eb:d3:22:e7 (oui Unknown)
            Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message (53), length 1: ACK
                Server-ID (54), length 4: 192.168.192.11
                Lease-Time (51), length 4: 300
                Subnet-Mask (1), length 4: 255.255.255.0
                Default-Gateway (3), length 4: 192.168.192.1
                Domain-Name-Server (6), length 4: 192.168.192.11
                Domain-Name (15), length 11: "lab.example"
    ^C
    2 packets captured
    2 packets received by filter
    0 packets dropped by kernel
    ```

    </details>    

* Triple level of verbosity 

    <details><summary>sudo tcpdump -i eth0 -vvv port 67 or port 68 </summary>

    ```
    tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
    20:48:14.780990 IP (tos 0x0, ttl 64, id 48085, offset 0, flags [none], proto UDP (17), length 328)
        192.168.192.102.bootpc > 255.255.255.255.bootps: [udp sum ok] BOOTP/DHCP, Request from b8:27:eb:d3:22:e7 (oui Unknown), length 300, xid 0xaad2fc4c, Flags [none] (0x0000)
            Client-Ethernet-Address b8:27:eb:d3:22:e7 (oui Unknown)
            Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message (53), length 1: Request
                Client-ID (61), length 7: ether b8:27:eb:d3:22:e7
                Requested-IP (50), length 4: 192.168.192.102
                MSZ (57), length 2: 1472
                Hostname (12), length 10: "lab-client"
                Unknown (145), length 1: 1
                Parameter-Request (55), length 14: 
                Subnet-Mask (1), Classless-Static-Route (121), Static-Route (33), Default-Gateway (3)
                Domain-Name-Server (6), Hostname (12), Domain-Name (15), MTU (26)
                BR (28), Lease-Time (51), Server-ID (54), RN (58)
                RB (59), Unknown (119)
                END (255), length 0
                PAD (0), length 0, occurs 6
    20:48:15.462075 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
        192.168.192.11.bootps > 192.168.192.102.bootpc: [udp sum ok] BOOTP/DHCP, Reply, length 300, xid 0xaad2fc4c, Flags [none] (0x0000)
            Your-IP 192.168.192.102
            Client-Ethernet-Address b8:27:eb:d3:22:e7 (oui Unknown)
            Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message (53), length 1: ACK
                Server-ID (54), length 4: 192.168.192.11
                Lease-Time (51), length 4: 300
                Subnet-Mask (1), length 4: 255.255.255.0
                Default-Gateway (3), length 4: 192.168.192.1
                Domain-Name-Server (6), length 4: 192.168.192.11
                Domain-Name (15), length 11: "lab.example"
                END (255), length 0
                PAD (0), length 0, occurs 13

    2 packets captured
    2 packets received by filter
    0 packets dropped by kernel
    ```

    </details>        

If you look at the three example above you might find a few difference, but there aren't a lot.  If we look at the man page for the option you'll see there are specific protocols where the different levels matter more. In most cases, a single `-v` is probably enough, but don't be afraid to crank it up. 

```
-v     When  parsing and printing, produce (slightly more) verbose output.  For example, the time to
        live, identification, total length and options in an IP packet are printed.  Also enables ad‐
        ditional packet integrity checks such as verifying the IP and ICMP header checksum.

-vv    Even more verbose output.  For example, additional fields are printed from NFS reply packets,
        and SMB packets are fully decoded.

-vvv   Even  more  verbose output.  For example, telnet SB ... SE options are printed in full.  With
        -X Telnet options are printed in hex as well.
```

# Span / Monitor Sessions on a Switch
The real fun of packet analysis is when you monitor traffic sent on the network to hosts other than the one you are running your capture tool on.  This is when your RPi becomes a "network sensor" or "tap".  Let's setup a span session on the network switch to send all traffic destined to the `lab-server` to the `lab-client` RPi. 

> `lab-server` is another Raspberry Pi that is configured to host DNS, DHCP, and provide other services to the network. 

> For this example I am using a Cisco Catalyst 3650 switch (WS-C3650-24TS-S).  

## Configuring the Monitor Session
This configuration creates a `monitor session` on my switch that will copy all traffic (`rx` and `tx`) on port `GigabitEthernet 1/0/1` where `lab-server` is connected to `GigabitEthernet 1/0/11` where `lab-client` is connected.  

```
monitor session 1 source interface Gi1/0/1
monitor session 1 destination interface Gi1/0/11
```

> Note: When a switch interface is setup as a `monitor destination`, normal operation of the interface is disrupted.  In effect the port becomes ***JUST*** a span port for monitoring traffic.  Some network devices support forwarding on destination ports as well, but that may require extra configuration and is out of scope for this guide. 

Let's check the configuration. 

```
show monitor session 1

# Output 
Session 1
---------
Type                     : Local Session
Source Ports             : 
    Both                 : Gi1/0/1
Destination Ports        : Gi1/0/11
    Encapsulation        : Native
          Ingress        : Disabled
```

## Capture Example 1: DNS Resolution
Let's look at the DNS resolution process at the packet level.  

> We filter `port 53` because DNS runs on `tcp/53` and `udp/53`

```bash
sudo tcpdump -i eth0 -v  port 53
```

Then with this running, I attempt to ping the `lab-switch.lab.example` from another host on the network.  This results in a name lookup to `lab-server`.

```
# Output 
21:30:37.250480 IP (tos 0x0, ttl 254, id 0, offset 0, flags [none], proto UDP (17), length 79)
    192.168.192.101.16766 > 192.168.192.11.domain: 28104+ [1au] A? lab-switch.lab.example. (51)
21:30:37.251467 IP (tos 0x0, ttl 64, id 21124, offset 0, flags [none], proto UDP (17), length 95)
    192.168.192.11.domain > 192.168.192.101.16766: 28104* 1/0/1 lab-switch.lab.example. A 192.168.192.101 (67)
```

Two packets are seen, first for the `request` and then for the `reply`.

## Capture Example 2: Telnet
One of my favorite things to look at with packet captures are network applications to show why encryption is so important.  In this example we'll monitor telnet traffic to/from the `lab-server` while it logs into the network switch.  But to truly understand what is being sent, we'll want to look at the full payload of the traffic.

```bash
sudo tcpdump -i eth0 -v -X port 23
```

> The `-X` option will display the frame in both HEX and ASCII format.  The ASCII format makes it easier on us humans to recognized data in the packets.  

With this running, I telnet into the switch and login.  Here is the captured output from the login process.  There is ***ALOT*** of packets because of some important facts about how telnet works for switch CLI. 

1. Each letter/button you press is sent to the switch, not just when you press "enter". 
    * So typing the command `show version` will send 12 packets. One for each letter and one for "ENTER". 
1. Telnet is a TCP protocol, so each packet is checked and acknowledged.  You can follow that in the packets as well

<details><summary>Packets from the Login Process Over Telnet with Comments</summary>

```
### Setting up connection

21:45:29.749649 IP (tos 0x10, ttl 64, id 6795, offset 0, flags [DF], proto TCP (6), length 60)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [S], cksum 0x5de6 (correct), seq 1740542972, win 64240, options [mss 1460,sackOK,TS val 1750529267 ecr 0,nop,wscale 7], length 0
        0x0000:  4510 003c 1a8b 4000 4006 1e5f c0a8 c00b  E..<..@.@.._....
        0x0010:  c0a8 c065 944a 0017 67be 93fc 0000 0000  ...e.J..g.......
        0x0020:  a002 faf0 5de6 0000 0204 05b4 0402 080a  ....]...........
        0x0030:  6856 f4f3 0000 0000 0103 0307            hV..........
21:45:29.751820 IP (tos 0x10, ttl 254, id 3085, offset 0, flags [none], proto TCP (6), length 44)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [S.], cksum 0xd33e (correct), seq 2262866935, ack 1740542973, win 4128, options [mss 1460], length 0
        0x0000:  4510 002c 0c0d 0000 fe06 aeec c0a8 c065  E..,...........e
        0x0010:  c0a8 c00b 0017 944a 86e0 9bf7 67be 93fd  .......J....g...
        0x0020:  6012 1020 d33e 0000 0204 05b4 0000       `....>........
21:45:29.752258 IP (tos 0x10, ttl 64, id 6796, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0x002b (correct), ack 1, win 64240, length 0
        0x0000:  4510 0028 1a8c 4000 4006 1e72 c0a8 c00b  E..(..@.@..r....
        0x0010:  c0a8 c065 944a 0017 67be 93fd 86e0 9bf8  ...e.J..g.......
        0x0020:  5010 faf0 002b 0000 0000 0000 0000       P....+........
21:45:29.752869 IP (tos 0x10, ttl 64, id 6797, offset 0, flags [DF], proto TCP (6), length 64)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0xa3b9 (correct), seq 1:25, ack 1, win 64240, length 24 [telnet DO SUPPRESS GO AHEAD, WILL TERMINAL TYPE, WILL NAWS, WILL TSPEED, WILL LFLOW, WILL LINEMODE, WILL NEW-ENVIRON, DO STATUS]
        0x0000:  4510 0040 1a8d 4000 4006 1e59 c0a8 c00b  E..@..@.@..Y....
        0x0010:  c0a8 c065 944a 0017 67be 93fd 86e0 9bf8  ...e.J..g.......
        0x0020:  5018 faf0 a3b9 0000 fffd 03ff fb18 fffb  P...............
        0x0030:  1fff fb20 fffb 21ff fb22 fffb 27ff fd05  ......!.."..'...
21:45:29.753930 IP (tos 0x0, ttl 254, id 3086, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xeafb (correct), ack 25, win 4104, length 0
        0x0000:  4500 0028 0c0e 0000 fe06 aeff c0a8 c065  E..(...........e
        0x0010:  c0a8 c00b 0017 944a 86e0 9bf8 67be 9415  .......J....g...
        0x0020:  5010 1008 eafb 0000 0d0a 80c0 0000       P.............
21:45:29.755211 IP (tos 0xc0, ttl 254, id 3087, offset 0, flags [none], proto TCP (6), length 52)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0xd7cb (correct), seq 1:13, ack 25, win 4104, length 12 [telnet WILL ECHO, WILL SUPPRESS GO AHEAD, DO TERMINAL TYPE, DO NAWS]
        0x0000:  45c0 0034 0c0f 0000 fe06 ae32 c0a8 c065  E..4.......2...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9bf8 67be 9415  .......J....g...
        0x0020:  5018 1008 d7cb 0000 fffb 01ff fb03 fffd  P...............
        0x0030:  18ff fd1f                                ....
21:45:29.755558 IP (tos 0x10, ttl 64, id 6798, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0x0013 (correct), ack 13, win 64228, length 0
        0x0000:  4510 0028 1a8e 4000 4006 1e70 c0a8 c00b  E..(..@.@..p....
        0x0010:  c0a8 c065 944a 0017 67be 9415 86e0 9c04  ...e.J..g.......
        0x0020:  5010 fae4 0013 0000 0000 0000 0000       P.............
21:45:29.755828 IP (tos 0x10, ttl 64, id 6799, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x0320 (correct), seq 25:37, ack 13, win 64228, length 12 [telnet DO ECHO, SB NAWS IS 0xb5 0 0x1c SE]
        0x0000:  4510 0034 1a8f 4000 4006 1e63 c0a8 c00b  E..4..@.@..c....
        0x0010:  c0a8 c065 944a 0017 67be 9415 86e0 9c04  ...e.J..g.......
        0x0020:  5018 fae4 0320 0000 fffd 01ff fa1f 00b5  P...............
        0x0030:  001c fff0                                ....


### Switch sends request for login.  
### It looks like this:

##############################
#  User Access Verification
#
#  Username: 
##############################

21:45:29.765599 IP (tos 0xc0, ttl 254, id 3088, offset 0, flags [none], proto TCP (6), length 80)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x6545 (correct), seq 13:53, ack 37, win 4092, length 40
        0x0000:  45c0 0050 0c10 0000 fe06 ae15 c0a8 c065  E..P...........e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c04 67be 9421  .......J....g..!
        0x0020:  5018 0ffc 6545 0000 0d0a 5573 6572 2041  P...eE....User.A
        0x0030:  6363 6573 7320 5665 7269 6669 6361 7469  ccess.Verificati
        0x0040:  6f6e 0d0a 0d0a 5573 6572 6e61 6d65 3a20  on....Username:.
21:45:29.765825 IP (tos 0xc0, ttl 254, id 3089, offset 0, flags [none], proto TCP (6), length 46)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0xd2cc (correct), seq 53:59, ack 37, win 4092, length 6 [telnet SB TERMINAL TYPE SEND SE]
        0x0000:  45c0 002e 0c11 0000 fe06 ae36 c0a8 c065  E..........6...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c2c 67be 9421  .......J...,g..!
        0x0020:  5018 0ffc d2cc 0000 fffa 1801 fff0       P.............
21:45:29.766025 IP (tos 0xc0, ttl 254, id 3090, offset 0, flags [none], proto TCP (6), length 43)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0xcab7 (correct), seq 59:62, ack 37, win 4092, length 3 [telnet DONT TSPEED]
        0x0000:  45c0 002b 0c12 0000 fe06 ae38 c0a8 c065  E..+.......8...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c32 67be 9421  .......J...2g..!
        0x0020:  5018 0ffc cab7 0000 fffe 2073 6572       P..........ser


### Note: Several packets related to communication removed from output


### Typing username of "tacacsadmin" begins

### Send "t" - seen at end of output 'P.......t.....'
21:45:31.400501 IP (tos 0x10, ttl 64, id 6808, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x8be9 (correct), seq 57:58, ack 74, win 64167, length 1
        0x0000:  4510 0029 1a98 4000 4006 1e65 c0a8 c00b  E..)..@.@..e....
        0x0010:  c0a8 c065 944a 0017 67be 9435 86e0 9c41  ...e.J..g..5...A
        0x0020:  5018 faa7 8be9 0000 7400 0000 0000       P.......t.....    <---- See the 't' here

### Switch sends back "t" to print to screen
21:45:31.401854 IP (tos 0xc0, ttl 254, id 3096, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x76a9 (correct), seq 74:75, ack 58, win 4071, length 1
        0x0000:  45c0 0029 0c18 0000 fe06 ae34 c0a8 c065  E..).......4...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c41 67be 9436  .......J...Ag..6
        0x0020:  5018 0fe7 76a9 0000 7400 80c0 a000       P...v...t.....

21:45:31.402246 IP (tos 0x10, ttl 64, id 6809, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xfff1 (correct), ack 75, win 64166, length 0
        0x0000:  4510 0028 1a99 4000 4006 1e65 c0a8 c00b  E..(..@.@..e....
        0x0010:  c0a8 c065 944a 0017 67be 9436 86e0 9c42  ...e.J..g..6...B
        0x0020:  5010 faa6 fff1 0000 0000 0000 0000       P.............

### Send and show "a"
21:45:31.548120 IP (tos 0x10, ttl 64, id 6810, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9ee8 (correct), seq 58:59, ack 75, win 64166, length 1
        0x0000:  4510 0029 1a9a 4000 4006 1e63 c0a8 c00b  E..)..@.@..c....
        0x0010:  c0a8 c065 944a 0017 67be 9436 86e0 9c42  ...e.J..g..6...B
        0x0020:  5018 faa6 9ee8 0000 6100 0000 0000       P.......a.....    <----- See the 'a' here

21:45:31.549584 IP (tos 0xc0, ttl 254, id 3097, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x89a8 (correct), seq 75:76, ack 59, win 4070, length 1
        0x0000:  45c0 0029 0c19 0000 fe06 ae33 c0a8 c065  E..).......3...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c42 67be 9437  .......J...Bg..7
        0x0020:  5018 0fe6 89a8 0000 6100 80c0 a000       P.......a.....
21:45:31.549933 IP (tos 0x10, ttl 64, id 6811, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xfff0 (correct), ack 76, win 64165, length 0
        0x0000:  4510 0028 1a9b 4000 4006 1e63 c0a8 c00b  E..(..@.@..c....
        0x0010:  c0a8 c065 944a 0017 67be 9437 86e0 9c43  ...e.J..g..7...C
        0x0020:  5010 faa5 fff0 0000 0000 0000 0000       P.............

### Send and show the rest of the username
21:45:31.706842 IP (tos 0x10, ttl 64, id 6812, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9ce7 (correct), seq 59:60, ack 76, win 64165, length 1
        0x0000:  4510 0029 1a9c 4000 4006 1e61 c0a8 c00b  E..)..@.@..a....
        0x0010:  c0a8 c065 944a 0017 67be 9437 86e0 9c43  ...e.J..g..7...C
        0x0020:  5018 faa5 9ce7 0000 6300 0000 0000       P.......c.....
21:45:31.708149 IP (tos 0xc0, ttl 254, id 3098, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x87a7 (correct), seq 76:77, ack 60, win 4069, length 1
        0x0000:  45c0 0029 0c1a 0000 fe06 ae32 c0a8 c065  E..).......2...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c43 67be 9438  .......J...Cg..8
        0x0020:  5018 0fe5 87a7 0000 6300 80c0 a000       P.......c.....
21:45:31.708532 IP (tos 0x10, ttl 64, id 6813, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffef (correct), ack 77, win 64164, length 0
        0x0000:  4510 0028 1a9d 4000 4006 1e61 c0a8 c00b  E..(..@.@..a....
        0x0010:  c0a8 c065 944a 0017 67be 9438 86e0 9c44  ...e.J..g..8...D
        0x0020:  5010 faa4 ffef 0000 0000 0000 0000       P.............
21:45:31.894636 IP (tos 0x10, ttl 64, id 6814, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9ee6 (correct), seq 60:61, ack 77, win 64164, length 1
        0x0000:  4510 0029 1a9e 4000 4006 1e5f c0a8 c00b  E..)..@.@.._....
        0x0010:  c0a8 c065 944a 0017 67be 9438 86e0 9c44  ...e.J..g..8...D
        0x0020:  5018 faa4 9ee6 0000 6100 0000 0000       P.......a.....
21:45:31.896141 IP (tos 0xc0, ttl 254, id 3099, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x89a6 (correct), seq 77:78, ack 61, win 4068, length 1
        0x0000:  45c0 0029 0c1b 0000 fe06 ae31 c0a8 c065  E..).......1...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c44 67be 9439  .......J...Dg..9
        0x0020:  5018 0fe4 89a6 0000 6100 80c0 a000       P.......a.....
21:45:31.896498 IP (tos 0x10, ttl 64, id 6815, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffee (correct), ack 78, win 64163, length 0
        0x0000:  4510 0028 1a9f 4000 4006 1e5f c0a8 c00b  E..(..@.@.._....
        0x0010:  c0a8 c065 944a 0017 67be 9439 86e0 9c45  ...e.J..g..9...E
        0x0020:  5010 faa3 ffee 0000 0000 0000 0000       P.............
21:45:32.031195 IP (tos 0x10, ttl 64, id 6816, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9ce5 (correct), seq 61:62, ack 78, win 64163, length 1
        0x0000:  4510 0029 1aa0 4000 4006 1e5d c0a8 c00b  E..)..@.@..]....
        0x0010:  c0a8 c065 944a 0017 67be 9439 86e0 9c45  ...e.J..g..9...E
        0x0020:  5018 faa3 9ce5 0000 6300 0000 0000       P.......c.....
21:45:32.033477 IP (tos 0xc0, ttl 254, id 3100, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x87a5 (correct), seq 78:79, ack 62, win 4067, length 1
        0x0000:  45c0 0029 0c1c 0000 fe06 ae30 c0a8 c065  E..).......0...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c45 67be 943a  .......J...Eg..:
        0x0020:  5018 0fe3 87a5 0000 6300 80c0 a000       P.......c.....
21:45:32.033831 IP (tos 0x10, ttl 64, id 6817, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffed (correct), ack 79, win 64162, length 0
        0x0000:  4510 0028 1aa1 4000 4006 1e5d c0a8 c00b  E..(..@.@..]....
        0x0010:  c0a8 c065 944a 0017 67be 943a 86e0 9c46  ...e.J..g..:...F
        0x0020:  5010 faa2 ffed 0000 0000 0000 0000       P.............
21:45:32.134829 IP (tos 0x10, ttl 64, id 6818, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x8ce4 (correct), seq 62:63, ack 79, win 64162, length 1
        0x0000:  4510 0029 1aa2 4000 4006 1e5b c0a8 c00b  E..)..@.@..[....
        0x0010:  c0a8 c065 944a 0017 67be 943a 86e0 9c46  ...e.J..g..:...F
        0x0020:  5018 faa2 8ce4 0000 7300 0000 0000       P.......s.....
21:45:32.136360 IP (tos 0xc0, ttl 254, id 3101, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x77a4 (correct), seq 79:80, ack 63, win 4066, length 1
        0x0000:  45c0 0029 0c1d 0000 fe06 ae2f c0a8 c065  E..)......./...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c46 67be 943b  .......J...Fg..;
        0x0020:  5018 0fe2 77a4 0000 7300 80c0 a000       P...w...s.....
21:45:32.136720 IP (tos 0x10, ttl 64, id 6819, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffec (correct), ack 80, win 64161, length 0
        0x0000:  4510 0028 1aa3 4000 4006 1e5b c0a8 c00b  E..(..@.@..[....
        0x0010:  c0a8 c065 944a 0017 67be 943b 86e0 9c47  ...e.J..g..;...G
        0x0020:  5010 faa1 ffec 0000 0000 0000 0000       P.............
21:45:32.729758 IP (tos 0x10, ttl 64, id 6820, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9ee3 (correct), seq 63:64, ack 80, win 64161, length 1
        0x0000:  4510 0029 1aa4 4000 4006 1e59 c0a8 c00b  E..)..@.@..Y....
        0x0010:  c0a8 c065 944a 0017 67be 943b 86e0 9c47  ...e.J..g..;...G
        0x0020:  5018 faa1 9ee3 0000 6100 0000 0000       P.......a.....
21:45:32.731192 IP (tos 0xc0, ttl 254, id 3102, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x89a3 (correct), seq 80:81, ack 64, win 4065, length 1
        0x0000:  45c0 0029 0c1e 0000 fe06 ae2e c0a8 c065  E..)...........e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c47 67be 943c  .......J...Gg..<
        0x0020:  5018 0fe1 89a3 0000 6100 80c0 a000       P.......a.....
21:45:32.731575 IP (tos 0x10, ttl 64, id 6821, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffeb (correct), ack 81, win 64160, length 0
        0x0000:  4510 0028 1aa5 4000 4006 1e59 c0a8 c00b  E..(..@.@..Y....
        0x0010:  c0a8 c065 944a 0017 67be 943c 86e0 9c48  ...e.J..g..<...H
        0x0020:  5010 faa0 ffeb 0000 0000 0000 0000       P.............
21:45:32.842825 IP (tos 0x10, ttl 64, id 6822, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9be2 (correct), seq 64:65, ack 81, win 64160, length 1
        0x0000:  4510 0029 1aa6 4000 4006 1e57 c0a8 c00b  E..)..@.@..W....
        0x0010:  c0a8 c065 944a 0017 67be 943c 86e0 9c48  ...e.J..g..<...H
        0x0020:  5018 faa0 9be2 0000 6400 0000 0000       P.......d.....
21:45:32.844133 IP (tos 0xc0, ttl 254, id 3103, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x86a2 (correct), seq 81:82, ack 65, win 4064, length 1
        0x0000:  45c0 0029 0c1f 0000 fe06 ae2d c0a8 c065  E..).......-...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c48 67be 943d  .......J...Hg..=
        0x0020:  5018 0fe0 86a2 0000 6400 80c0 a000       P.......d.....
21:45:32.844516 IP (tos 0x10, ttl 64, id 6823, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffea (correct), ack 82, win 64159, length 0
        0x0000:  4510 0028 1aa7 4000 4006 1e57 c0a8 c00b  E..(..@.@..W....
        0x0010:  c0a8 c065 944a 0017 67be 943d 86e0 9c49  ...e.J..g..=...I
        0x0020:  5010 fa9f ffea 0000 0000 0000 0000       P.............
21:45:32.907367 IP (tos 0x10, ttl 64, id 6824, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x92e1 (correct), seq 65:66, ack 82, win 64159, length 1
        0x0000:  4510 0029 1aa8 4000 4006 1e55 c0a8 c00b  E..)..@.@..U....
        0x0010:  c0a8 c065 944a 0017 67be 943d 86e0 9c49  ...e.J..g..=...I
        0x0020:  5018 fa9f 92e1 0000 6d00 0000 0000       P.......m.....
21:45:32.908601 IP (tos 0xc0, ttl 254, id 3104, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x7da1 (correct), seq 82:83, ack 66, win 4063, length 1
        0x0000:  45c0 0029 0c20 0000 fe06 ae2c c0a8 c065  E..).......,...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c49 67be 943e  .......J...Ig..>
        0x0020:  5018 0fdf 7da1 0000 6d00 80c0 a000       P...}...m.....
21:45:32.908939 IP (tos 0x10, ttl 64, id 6825, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffe9 (correct), ack 83, win 64158, length 0
        0x0000:  4510 0028 1aa9 4000 4006 1e55 c0a8 c00b  E..(..@.@..U....
        0x0010:  c0a8 c065 944a 0017 67be 943e 86e0 9c4a  ...e.J..g..>...J
        0x0020:  5010 fa9e ffe9 0000 0000 0000 0000       P.............
21:45:33.048498 IP (tos 0x10, ttl 64, id 6826, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x96e0 (correct), seq 66:67, ack 83, win 64158, length 1
        0x0000:  4510 0029 1aaa 4000 4006 1e53 c0a8 c00b  E..)..@.@..S....
        0x0010:  c0a8 c065 944a 0017 67be 943e 86e0 9c4a  ...e.J..g..>...J
        0x0020:  5018 fa9e 96e0 0000 6900 0000 0000       P.......i.....
21:45:33.050415 IP (tos 0xc0, ttl 254, id 3105, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x81a0 (correct), seq 83:84, ack 67, win 4062, length 1
        0x0000:  45c0 0029 0c21 0000 fe06 ae2b c0a8 c065  E..).!.....+...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c4a 67be 943f  .......J...Jg..?
        0x0020:  5018 0fde 81a0 0000 6900 80c0 a000       P.......i.....
21:45:33.050794 IP (tos 0x10, ttl 64, id 6827, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffe8 (correct), ack 84, win 64157, length 0
        0x0000:  4510 0028 1aab 4000 4006 1e53 c0a8 c00b  E..(..@.@..S....
        0x0010:  c0a8 c065 944a 0017 67be 943f 86e0 9c4b  ...e.J..g..?...K
        0x0020:  5010 fa9d ffe8 0000 0000 0000 0000       P.............
21:45:33.149953 IP (tos 0x10, ttl 64, id 6828, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x91df (correct), seq 67:68, ack 84, win 64157, length 1
        0x0000:  4510 0029 1aac 4000 4006 1e51 c0a8 c00b  E..)..@.@..Q....
        0x0010:  c0a8 c065 944a 0017 67be 943f 86e0 9c4b  ...e.J..g..?...K
        0x0020:  5018 fa9d 91df 0000 6e00 0000 0000       P.......n.....
21:45:33.151499 IP (tos 0xc0, ttl 254, id 3106, offset 0, flags [none], proto TCP (6), length 41)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x7c9f (correct), seq 84:85, ack 68, win 4061, length 1
        0x0000:  45c0 0029 0c22 0000 fe06 ae2a c0a8 c065  E..).".....*...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c4b 67be 9440  .......J...Kg..@
        0x0020:  5018 0fdd 7c9f 0000 6e00 80c0 a000       P...|...n.....
21:45:33.151885 IP (tos 0x10, ttl 64, id 6829, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffe7 (correct), ack 85, win 64156, length 0
        0x0000:  4510 0028 1aad 4000 4006 1e51 c0a8 c00b  E..(..@.@..Q....
        0x0010:  c0a8 c065 944a 0017 67be 9440 86e0 9c4c  ...e.J..g..@...L
        0x0020:  5010 fa9c ffe7 0000 0000 0000 0000       P.............
21:45:33.867814 IP (tos 0x10, ttl 64, id 6830, offset 0, flags [DF], proto TCP (6), length 42)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0xf2dd (correct), seq 68:70, ack 85, win 64156, length 2
        0x0000:  4510 002a 1aae 4000 4006 1e4e c0a8 c00b  E..*..@.@..N....
        0x0010:  c0a8 c065 944a 0017 67be 9440 86e0 9c4c  ...e.J..g..@...L
        0x0020:  5018 fa9c f2dd 0000 0d00 0000 0000       P.............

### Username sent.  Switch requests password

21:45:33.872030 IP (tos 0xc0, ttl 254, id 3107, offset 0, flags [none], proto TCP (6), length 52)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0xf5c0 (correct), seq 85:97, ack 70, win 4059, length 12
        0x0000:  45c0 0034 0c23 0000 fe06 ae1e c0a8 c065  E..4.#.........e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c4c 67be 9442  .......J...Lg..B
        0x0020:  5018 0fdb f5c0 0000 0d0a 5061 7373 776f  P.........Passwo
        0x0030:  7264 3a20                                rd:.
21:45:33.872420 IP (tos 0x10, ttl 64, id 6831, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffe5 (correct), ack 97, win 64144, length 0
        0x0000:  4510 0028 1aaf 4000 4006 1e4f c0a8 c00b  E..(..@.@..O....
        0x0010:  c0a8 c065 944a 0017 67be 9442 86e0 9c58  ...e.J..g..B...X
        0x0020:  5010 fa90 ffe5 0000 0000 0000 0000       P.............


### Begin typing the password which is "password"

### Send "p"
21:45:36.667308 IP (tos 0x10, ttl 64, id 6832, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x8fdc (correct), seq 70:71, ack 97, win 64144, length 1
        0x0000:  4510 0029 1ab0 4000 4006 1e4d c0a8 c00b  E..)..@.@..M....
        0x0010:  c0a8 c065 944a 0017 67be 9442 86e0 9c58  ...e.J..g..B...X
        0x0020:  5018 fa90 8fdc 0000 7000 0000 0000       P.......p.....
21:45:36.868429 IP (tos 0xc0, ttl 254, id 3108, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xea9b (correct), ack 71, win 4058, length 0
        0x0000:  45c0 0028 0c24 0000 fe06 ae29 c0a8 c065  E..(.$.....)...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 9443  .......J...Xg..C
        0x0020:  5010 0fda ea9b 0000 0000 80c0 0000       P.............

### Send "a"
21:45:36.868789 IP (tos 0x10, ttl 64, id 6833, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x9edb (correct), seq 71:72, ack 97, win 64144, length 1
        0x0000:  4510 0029 1ab1 4000 4006 1e4c c0a8 c00b  E..)..@.@..L....
        0x0010:  c0a8 c065 944a 0017 67be 9443 86e0 9c58  ...e.J..g..C...X
        0x0020:  5018 fa90 9edb 0000 6100 0000 0000       P.......a.....
21:45:37.070726 IP (tos 0xc0, ttl 254, id 3109, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xea9b (correct), ack 72, win 4057, length 0
        0x0000:  45c0 0028 0c25 0000 fe06 ae28 c0a8 c065  E..(.%.....(...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 9444  .......J...Xg..D
        0x0020:  5010 0fd9 ea9b 0000 0000 80c0 0000       P.............

### Send the rest of the password 
21:45:37.071089 IP (tos 0x10, ttl 64, id 6834, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x8cda (correct), seq 72:73, ack 97, win 64144, length 1
        0x0000:  4510 0029 1ab2 4000 4006 1e4b c0a8 c00b  E..)..@.@..K....
        0x0010:  c0a8 c065 944a 0017 67be 9444 86e0 9c58  ...e.J..g..D...X
        0x0020:  5018 fa90 8cda 0000 7300 0000 0000       P.......s.....
21:45:37.272820 IP (tos 0xc0, ttl 254, id 3110, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xea9b (correct), ack 73, win 4056, length 0
        0x0000:  45c0 0028 0c26 0000 fe06 ae27 c0a8 c065  E..(.&.....'...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 9445  .......J...Xg..E
        0x0020:  5010 0fd8 ea9b 0000 0000 80c0 0000       P.............
21:45:37.273166 IP (tos 0x10, ttl 64, id 6835, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x8cd9 (correct), seq 73:74, ack 97, win 64144, length 1
        0x0000:  4510 0029 1ab3 4000 4006 1e4a c0a8 c00b  E..)..@.@..J....
        0x0010:  c0a8 c065 944a 0017 67be 9445 86e0 9c58  ...e.J..g..E...X
        0x0020:  5018 fa90 8cd9 0000 7300 0000 0000       P.......s.....
21:45:37.474825 IP (tos 0xc0, ttl 254, id 3111, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xea9b (correct), ack 74, win 4055, length 0
        0x0000:  45c0 0028 0c27 0000 fe06 ae26 c0a8 c065  E..(.'.....&...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 9446  .......J...Xg..F
        0x0020:  5010 0fd7 ea9b 0000 0000 80c0 0000       P.............
21:45:37.475156 IP (tos 0x10, ttl 64, id 6836, offset 0, flags [DF], proto TCP (6), length 41)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x88d8 (correct), seq 74:75, ack 97, win 64144, length 1
        0x0000:  4510 0029 1ab4 4000 4006 1e49 c0a8 c00b  E..)..@.@..I....
        0x0010:  c0a8 c065 944a 0017 67be 9446 86e0 9c58  ...e.J..g..F...X
        0x0020:  5018 fa90 88d8 0000 7700 0000 0000       P.......w.....
21:45:37.676871 IP (tos 0xc0, ttl 254, id 3112, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xea9b (correct), ack 75, win 4054, length 0
        0x0000:  45c0 0028 0c28 0000 fe06 ae25 c0a8 c065  E..(.(.....%...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 9447  .......J...Xg..G
        0x0020:  5010 0fd6 ea9b 0000 0000 80c0 0000       P.............
21:45:37.677223 IP (tos 0x10, ttl 64, id 6837, offset 0, flags [DF], proto TCP (6), length 43)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0x2c63 (correct), seq 75:78, ack 97, win 64144, length 3
        0x0000:  4510 002b 1ab5 4000 4006 1e46 c0a8 c00b  E..+..@.@..F....
        0x0010:  c0a8 c065 944a 0017 67be 9447 86e0 9c58  ...e.J..g..G...X
        0x0020:  5018 fa90 2c63 0000 6f72 6400 0000       P...,c..ord...
21:45:37.878672 IP (tos 0xc0, ttl 254, id 3113, offset 0, flags [none], proto TCP (6), length 40)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [.], cksum 0xea9b (correct), ack 78, win 4051, length 0
        0x0000:  45c0 0028 0c29 0000 fe06 ae24 c0a8 c065  E..(.).....$...e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 944a  .......J...Xg..J
        0x0020:  5010 0fd3 ea9b 0000 0000 80c0 0000       P.............
21:45:39.058780 IP (tos 0x10, ttl 64, id 6838, offset 0, flags [DF], proto TCP (6), length 42)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [P.], cksum 0xf2d3 (correct), seq 78:80, ack 97, win 64144, length 2
        0x0000:  4510 002a 1ab6 4000 4006 1e46 c0a8 c00b  E..*..@.@..F....
        0x0010:  c0a8 c065 944a 0017 67be 944a 86e0 9c58  ...e.J..g..J...X
        0x0020:  5018 fa90 f2d3 0000 0d00 0000 0000       P.............

### Password submitted 

### Switch sends prompt

21:45:39.080470 IP (tos 0xc0, ttl 254, id 3114, offset 0, flags [none], proto TCP (6), length 55)
    192.168.192.101.telnet > 192.168.192.11.37962: Flags [P.], cksum 0x9e8d (correct), seq 97:112, ack 80, win 4049, length 15
        0x0000:  45c0 0037 0c2a 0000 fe06 ae14 c0a8 c065  E..7.*.........e
        0x0010:  c0a8 c00b 0017 944a 86e0 9c58 67be 944c  .......J...Xg..L
        0x0020:  5018 0fd1 9e8d 0000 0d0a 0d0a 6c61 622d  P...........lab-
        0x0030:  7377 6974 6368 23                        switch#
21:45:39.080904 IP (tos 0x10, ttl 64, id 6839, offset 0, flags [DF], proto TCP (6), length 40)
    192.168.192.11.37962 > 192.168.192.101.telnet: Flags [.], cksum 0xffdb (correct), ack 112, win 64129, length 0
        0x0000:  4510 0028 1ab7 4000 4006 1e47 c0a8 c00b  E..(..@.@..G....
        0x0010:  c0a8 c065 944a 0017 67be 944c 86e0 9c67  ...e.J..g..L...g
        0x0020:  5010 fa81 ffdb 0000 0000 0000 0000       P.............
```

</details>

## Capture Example 3: Creating a PCAP file 
Displaying packets as you capture them onscreen is great.  But often in troubleshooting you can't analyze them onscreen while they come in.  It is just too much data.  Rather you capture packets to a file to be processed later.  Maybe using a GUI tool like Wireshark installed on your primary workstation.  

In this example, we will once again capture the telnet traffic, but rather than display onscreen, we will create a capture file.  

```bash
sudo tcpdump -i eth0 -v -w telnet-capture.cap port 23
```

> `-w telnet-capture.cap` will "write" the capture to a file. 
>
> `-v` for a "write" capture will display the packet count to the screen.

***Output***

```
tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
76 packets captured
^C
76 packets received by filter
0 packets dropped by kernel

ls -l
total 8
-rw-r--r-- 1 tcpdump tcpdump 5909 Mar  3 22:06 telnet-capture.cap
```

With the file captured, you could "read" it in for processing with `tcpdump -r telnet-capture.cap`

***Partial Output***

```
reading from file telnet-capture.cap, link-type EN10MB (Ethernet), snapshot length 262144
22:06:33.957161 IP 192.168.192.11.37964 > 192.168.192.101.telnet: Flags [S], seq 2106082144, win 64240, options [mss 1460,sackOK,TS val 1751793461 ecr 0,nop,wscale 7], length 0
22:06:33.959364 IP 192.168.192.101.telnet > 192.168.192.11.37964: Flags [S.], seq 1143027395, ack 2106082145, win 4128, options [mss 1460], length 0
22:06:33.959801 IP 192.168.192.11.37964 > 192.168.192.101.telnet: Flags [.], ack 1, win 64240, length 0
22:06:33.960492 IP 192.168.192.11.37964 > 192.168.192.101.telnet: Flags [P.], seq 1:25, ack 1, win 64240, length 24 [telnet DO SUPPRESS GO AHEAD, WILL TERMINAL TYPE, WILL NAWS, WILL TSPEED, WILL LFLOW, WILL LINEMODE, WILL NEW-ENVIRON, DO STATUS]
22:06:33.962086 IP 192.168.192.101.telnet > 192.168.192.11.37964: Flags [.], ack 25, win 4104, length 0
22:06:33.962375 IP 192.168.192.101.telnet > 192.168.192.11.37964: Flags [P.], seq 1:13, ack 25, win 4104, length 12 [telnet WILL ECHO, WILL SUPPRESS GO AHEAD, DO TERMINAL TYPE, DO NAWS]
22:06:33.962691 IP 192.168.192.11.37964 > 192.168.192.101.telnet: Flags [.], ack 13, win 64228, length 0
.
.
.
```

But far more common is copying the file to another workstation for analysis.  

# Final Thoughts 
There you go, you are ready to explore packet capture and traffic analysis in your lab using your Raspberry Pi as a network sensor.  From here there is so much you can dive into next.  

* Explore all the different filters you can create 
* Look at the packets for protocols like CDP or Spanning-tree 
* Watch routing protocols exchange updates 
* Explore Wifi packet capture for wireless

## What about Wireshark? 

Wireshark is another very popular packet capture tool that can be installed and used on your Raspberry Pi.  You can do so with `sudo apt-get install wireshark`.  

I am a big fan of Wireshark, but it is a bit "heavy" for running on a small Raspberry Pi.  When I checked the installation needs on my device I got this: 

```
0 upgraded, 154 newly installed, 0 to remove and 0 not upgraded.
Need to get 97.0 MB of archives.
After this operation, 424 MB of additional disk space will be used.
```

My approach to Wireshark in the lab is to install it on my laptop/workstation and use `tcpdump` on the Raspberry Pi to create capture files.  I then copy the files to my workstation an analyze with Wireshark there.  