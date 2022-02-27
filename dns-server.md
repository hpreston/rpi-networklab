# Using a Raspberry Pi as a DNS Server
* [What DNS Server Software to Use? dnsmasq vs Bind](#what-dns-server-software-to-use--dnsmasq-vs-bind)
* [Installing Bind9 on Raspberry Pi](#installing-bind9-on-raspberry-pi)
* [Testing DNS Resolution](#testing-dns-resolution)
* [Configuring the `lab-server` to use itself for DNS lookups](#configuring-the--lab-server--to-use-itself-for-dns-lookups)
* [Configuring Bind9 on Raspberry Pi](#configuring-bind9-on-raspberry-pi)
* [Verifying DNS Lookups on Lab Clients](#verifying-dns-lookups-on-lab-clients)
+ [Testing on an RPi `lab-client`](#testing-on-an-rpi--lab-client-)
+ [Testing from a Network Device](#testing-from-a-network-device)
* [Advanced Configuration: Dynamic DNS Updates from ISC DHCP Server -> Bind](#advanced-configuration--dynamic-dns-updates-from-isc-dhcp-server----bind)
* [Final Thoughts](#final-thoughts)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


Right up there with DHCP, DNS is a fundamental part of any network.  Few people other than network engineers actually LIKE to remember and refer to servers, applications, etc by their IP address.  Nope... people like to use names.  And that's where DNS comes in. Also like DHCP, DNS shows up on the [CCNA certification](https://learningnetwork.cisco.com/s/ccna-exam-topics) as well as popping up in many other professional, specialist, and expert certifications. 

There are options for hosting a DNS server in your network lab, the most common I've seen include: 

* Some network devices can act as a DNS server
* Add a server to your lab and run an enterprise DNS server such as Microsoft DNS Server or Bind on Linux

The option of using [configuring a network router](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_dns/configuration/15-sy/dns-15-sy-book/Configuring-DNS.html#GUID-B35BAE29-08A7-4ACE-94FE-7950A4422202) to act as a DNS server is something I have done in many labs.  And while it works, it's really not a good solution.  For one, it is something you'll likely never see actually used in a real network.  And secondly, the DNS server in most network devices is very limited and doesn't support many of the DNS features needed in modern networks. 

So the best option is the second, and that's where the Raspberry Pi comes in as a great option.  As a low cost computer, RPis are inexpensive to add to a lab.  And because it runs Linux, you can install and use the same DNS server that you might run across in the office.  

## What DNS Server Software to Use? dnsmasq vs Bind
If you do some searching for "DNS Server for Raspberry Pi", you'll find lots of documentation, blog 
posts, and forum discussions.  I know because I did it.  Most of them seem to suggest using [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html).  This was a bit of a surprise to me, as I've long used [Bind](https://www.isc.org/bind/) as the DNS server on Linux systems.  

> Note: Today "Bind" is really "Bind9".

I asked myself, "Why all the love for dnsmasq?"  I'd heard of dnsmasq before, but I hadn't really used it.  Not wanting to just go with what I've always done, I did some digging to see if I could learn more. What I found broke down to three main reasons: 

1. dnsmasq provides both DHCP and DNS services in a single package 
1. dnsmasq is simpler to configure (according to opinions I read, not sure of accuracy here)
1. dnsmasq has a lower cpu and memory footprint than the ISC options of DHCP Server and Bind9 (DNS)

So which do we use for our network lab?  

In the end, it's up to you.  The reasons above are compelling, however I decided to stick with Bind9 for my lab.  The main reason for this choice was that I'm much more likely to use Bind9 in work environments than dnsmasq, and I'd rather lab what I'm going to use in work.  

> Note: There are other DNS servers for Linux that could be used. And some are used regularly in enterprise environment.  Bind9 continues to be popular, and is the one I'm most familar with, so that's what I used ;-) 

## Installing Bind9 on Raspberry Pi 
Bind9 is available in the standard packages for Raspberry Pi, so installation is like any other Linux server. 

```bash
sudo apt-get install bind9 bind9utils dnsutils
```

> Note: In addition to bind9, I'm also installing very useful utilities for DNS.  The `dnsutils` is a great package to also include on your `lab-client` devices to allow lookups using tools like `dig`.

<details><summary>Output: Installation Command</summary>

```
# Output
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  bind9-dnsutils bind9-utils dns-root-data python3-ply
Suggested packages:
  bind-doc ufw python-ply-doc
The following NEW packages will be installed:
  bind9 bind9-dnsutils bind9-utils bind9utils dns-root-data dnsutils python3-ply
0 upgraded, 7 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,873 kB of archives.
After this operation, 3,494 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://deb.debian.org/debian bullseye/main arm64 python3-ply all 3.11-4 [65.5 kB]
Get:2 http://deb.debian.org/debian bullseye/main arm64 bind9-utils arm64 1:9.16.22-1~deb11u1 [421 kB]
Get:3 http://deb.debian.org/debian bullseye/main arm64 dns-root-data all 2021011101 [5,524 B]
Get:4 http://deb.debian.org/debian bullseye/main arm64 bind9 arm64 1:9.16.22-1~deb11u1 [472 kB]
Get:5 http://deb.debian.org/debian bullseye/main arm64 bind9-dnsutils arm64 1:9.16.22-1~deb11u1 [390 kB]
Get:6 http://deb.debian.org/debian bullseye/main arm64 bind9utils all 1:9.16.22-1~deb11u1 [260 kB]
Get:7 http://deb.debian.org/debian bullseye/main arm64 dnsutils all 1:9.16.22-1~deb11u1 [260 kB]
Fetched 1,873 kB in 1s (1,603 kB/s)
Selecting previously unselected package python3-ply.
(Reading database ... 35899 files and directories currently installed.)
Preparing to unpack .../0-python3-ply_3.11-4_all.deb ...
Unpacking python3-ply (3.11-4) ...
Selecting previously unselected package bind9-utils.
Preparing to unpack .../1-bind9-utils_1%3a9.16.22-1~deb11u1_arm64.deb ...
Unpacking bind9-utils (1:9.16.22-1~deb11u1) ...
Selecting previously unselected package dns-root-data.
Preparing to unpack .../2-dns-root-data_2021011101_all.deb ...
Unpacking dns-root-data (2021011101) ...
Selecting previously unselected package bind9.
Preparing to unpack .../3-bind9_1%3a9.16.22-1~deb11u1_arm64.deb ...
Unpacking bind9 (1:9.16.22-1~deb11u1) ...
Selecting previously unselected package bind9-dnsutils.
Preparing to unpack .../4-bind9-dnsutils_1%3a9.16.22-1~deb11u1_arm64.deb ...
Unpacking bind9-dnsutils (1:9.16.22-1~deb11u1) ...
Selecting previously unselected package bind9utils.
Preparing to unpack .../5-bind9utils_1%3a9.16.22-1~deb11u1_all.deb ...
Unpacking bind9utils (1:9.16.22-1~deb11u1) ...
Selecting previously unselected package dnsutils.
Preparing to unpack .../6-dnsutils_1%3a9.16.22-1~deb11u1_all.deb ...
Unpacking dnsutils (1:9.16.22-1~deb11u1) ...
Setting up python3-ply (3.11-4) ...
Setting up dns-root-data (2021011101) ...
Setting up bind9-utils (1:9.16.22-1~deb11u1) ...
Setting up bind9 (1:9.16.22-1~deb11u1) ...
Adding group `bind' (GID 114) ...
Done.
Adding system user `bind' (UID 109) ...
Adding new user `bind' (UID 109) with group `bind' ...
Not creating home directory `/var/cache/bind'.
wrote key file "/etc/bind/rndc.key"
named-resolvconf.service is a disabled or a static unit, not starting it.
Created symlink /etc/systemd/system/bind9.service → /lib/systemd/system/named.service.
Created symlink /etc/systemd/system/multi-user.target.wants/named.service → /lib/systemd/system/named.service.
Setting up bind9utils (1:9.16.22-1~deb11u1) ...
Setting up bind9-dnsutils (1:9.16.22-1~deb11u1) ...
Setting up dnsutils (1:9.16.22-1~deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
```

</details>

## Testing DNS Resolution
By default the Bind9 server will startup and support DNS lookups, including recursive lookups for public domain names.  We can test that it working from the server with the `dig` command.  

```bash
# Lookup the address for the Cisco Learning Network using the local server
dig learningnetwork.cisco.com @localhost
```

> If you do NOT include @localhost in the command, dig will use the configured DNS servers for the RPi.  This is likely whatever the DNS servers in your home network and were provided by the DHCP server to the wlan0 network.  We'll update the RPi `lab-server` to use itself for DNS resolution shortly.

<details><summary>Output: dig command</summary>

```
# Output
; <<>> DiG 9.16.22-Debian <<>> learningnetwork.cisco.com @localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10418
;; flags: qr rd ra; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 77bdbc1b157f3b4401000000621aa0fce0f01d6c29267348 (good)
;; QUESTION SECTION:
;learningnetwork.cisco.com.     IN      A

;; ANSWER SECTION:
learningnetwork.cisco.com. 60   IN      CNAME   learningnetwork.cisco.com.00d3i000000uddneai.live.siteforce.com.
learningnetwork.cisco.com.00d3i000000uddneai.live.siteforce.com. 300 IN CNAME n.edge2.salesforce.com.
n.edge2.salesforce.com. 60      IN      CNAME   iad-dfw.edge2.salesforce.com.
iad-dfw.edge2.salesforce.com. 60 IN     CNAME   dfw.edge2.salesforce.com.
dfw.edge2.salesforce.com. 60    IN      A       13.110.28.14
dfw.edge2.salesforce.com. 60    IN      A       13.110.28.9
dfw.edge2.salesforce.com. 60    IN      A       13.110.28.13

;; Query time: 163 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sat Feb 26 16:51:56 EST 2022
;; MSG SIZE  rcvd: 283
```

</details>

*In the output you can see that a successful reply for the lookup was provided.  A `CNAME` and then the related `A` records all are shown.*

## Configuring the `lab-server` to use itself for DNS lookups
Update the static IP configuration for the `eth0` lab interface to include a static DNS server.  This is done in `/etc/dhcpcd.conf`. 

> More details on network configuration for a RPi is provided in [network-interface-config.md](network-interface-config.md)

```bash
# Home Network Lab Setup
interface eth0
static ip_address=192.168.192.11/24
static domain_name_servers=192.168.192.11
```

If you make changes to the `dhcpcd.conf` file, you need to restart the service to apply changes.  

```bash
sudo systemctl restart dhcpcd
```

And then you can check the contents of `/etc/resolv.conf` to verify the server has been added. 

```bash
# Generated by resolvconf
nameserver 192.168.192.11
nameserver 208.67.222.222
nameserver 208.67.220.220
```

If you re-do the `dig` command, leaving off `@localhost` this time, you can verify lookups are first trying the local server.

```bash
dig learningnetwork.cisco.com
```

<details><summary>Output: dig command</summary>

```
; <<>> DiG 9.16.22-Debian <<>> learningnetwork.cisco.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64206
;; flags: qr rd ra; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3db5e61f030092be01000000621aa313c684b16ffb476d46 (good)
;; QUESTION SECTION:
;learningnetwork.cisco.com.     IN      A

;; ANSWER SECTION:
learningnetwork.cisco.com. 60   IN      CNAME   learningnetwork.cisco.com.00d3i000000uddneai.live.siteforce.com.
learningnetwork.cisco.com.00d3i000000uddneai.live.siteforce.com. 300 IN CNAME n.edge2.salesforce.com.
n.edge2.salesforce.com. 60      IN      CNAME   iad-dfw.edge2.salesforce.com.
iad-dfw.edge2.salesforce.com. 60 IN     CNAME   iad.edge2.salesforce.com.
iad.edge2.salesforce.com. 60    IN      A       13.110.24.13
iad.edge2.salesforce.com. 60    IN      A       13.110.24.10
iad.edge2.salesforce.com. 60    IN      A       13.110.24.11

;; Query time: 499 msec
;; SERVER: 192.168.192.11#53(192.168.192.11)
;; WHEN: Sat Feb 26 17:00:51 EST 2022
;; MSG SIZE  rcvd: 283
```

</details>

*Notice the new local address listed in the line `;; SERVER: 192.168.192.11#53(192.168.192.11)`*

## Configuring Bind9 on Raspberry Pi 
Being able to do public internet lookups using our own DNS server is nice, but the real reason to add a DNS server to your lab is to create DNS entries for your lab gear.  Here we will setup a basic configuration for the domain `lab.example` including a few `A` and `CNAME` records for our lab devices and `PTR` records for IP addresses. 

> For the full configuration guide for Bind9 go [here](https://bind9.readthedocs.io/en/latest/). What follows is a basic configuration for a network lab.

1. Begin by defining the forward and reverse zones for your lab. This is done by adding the configuration to the file `/etc/bind/named.conf.local`.
    > A "forward zone" translates names -> ip addresses.  A "reverse zone" translates ip addresses -> names

    ```
    // Forward lookup zone for lab.example
    zone "lab.example" IN {
            type master;
            file "/etc/bind/db.lab.example";
        };

    // Reverse lookup zone for the network 192.168.192.0
    //   Note: Reverse lookup zones are named with the first 
    //         3 octets of an IPv4 address written in reverse. 
    //         Our example network has the same first and third
    //         octet, so it may look like the zone name is 
    //         ordered 1st, 2nd, 3rd octets. It is actually 
    //         3rd, 2nd, 1st.
    zone "192.168.192.in-addr.arpa" {
            type master;
            file "/etc/bind/db.rev.192.168.192.in-addr.arpa";
        };
    ```

1. Next we will configure the forward lookup zone for `lab.example`.  This is done in the "database" file listed in the zone configuration. Specifically `/etc/bind/db.lab.example`. 

    ```
    $TTL   1H

    ; Start of Authority (SOA) for the domain with short timers
    ;  Note: The value of "serial" must be incremented with each update
    lab.example. IN SOA ns1.lab.example. lab-server.lab.example. (
        11 ; serial
        2H ; refresh
        1H ; retry
        1W ; expire
        1D ; minimum
    )

    ; NS or name server records for the DNS server itself 
                        IN   NS           ns1.lab.example.

    ; Name Server A Records
    ns1                 IN   A            192.168.192.11

    ; A Record for the lab-server
    lab-server          IN   A            192.168.192.11

    ; A CNAME for files.lab.example -> lab-server.lab.example
    files               IN   CNAME        lab-server.lab.example.
    ```

1. And now the reverse lookup zone. This will be in the database file `/etc/bind/db.rev.192.168.192.in-addr.arpa`

    ```
    $TTL   1H

    @ IN SOA ns1.lab.example. lab-server.lab.example. (
        11 ; serial
        2H ; refresh
        1H ; retry
        1W ; expire
        1D ; minimum
    )
    ; NS record
                IN     NS      ns1.lab.example.

    ; PTR record for the lab-server
    11          IN     PTR     lab-server.lab.example.
    ```

1. Creating a bind configuration can be challenging.  The format is particular, and placement of `.`'s in the right places can be troublesome.  Luckly we can use the following commands to check our files for errors. 

    ```bash
    # Check the forward lookup zone
    sudo named-checkzone lab.example db.lab.example

    # Output 
    zone lab.example/IN: loaded serial 11
    OK

    # Check the reverse lookup zone
    sudo named-checkzone 192.168.192.in-addr.arpa db.rev.192.168.192.in-addr.arpa

    # Output 
    zone 192.168.192.in-addr.arpa/IN: loaded serial 11
    OK
    ```

    > Note: Don't feel too bad if you have errors the first time you run the command.  I had several that I needed to fix before I got the correct configs shown above ;-) 

1. Now restart bind9 to apply the configuration.  

    ```bash
    sudo systemctl restart bind9
    ```

    > If you get any errors, look for messages poiting to the possible problem with commands like `systemctl status bind9`, `journalctl -xe`, `journalctl -u named`

## Verifying DNS Lookups on Lab Clients
Here we will verify that we can use the `lab-server` dns on the `lab-client` RPi and configure DNS on a network switch to use our new server.  

### Testing on an RPi `lab-client`
1. First, make sure your `lab-client` is configured to use the `lab-server` as a DNS server.  This could be done with a [static-ip configuration](network-interface-config.md), or through a [DHCP provided configuration](dhcp-server.md).
1. If you haven't already, install `apt-get install dnsutils` on your `lab-client`. 
1. Try a lookup of the `files.lab.example` CNAME that was configured. 

    ```bash
    dig files.lab.example
    ```

    <details><summary>Output: dig lookup</summary>

    ```
    ; <<>> DiG 9.16.22-Debian <<>> files.lab.example
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39862
    ;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: 26963ac0e9e70cdf01000000621abda0c8d5501dda5953fa (good)
    ;; QUESTION SECTION:
    ;files.lab.example.             IN      A

    ;; ANSWER SECTION:
    files.lab.example.      3600    IN      CNAME   lab-server.lab.example.
    lab-server.lab.example. 3600    IN      A       192.168.192.11

    ;; Query time: 3 msec
    ;; SERVER: 192.168.192.11#53(192.168.192.11)
    ;; WHEN: Sat Feb 26 18:54:08 EST 2022
    ;; MSG SIZE  rcvd: 115
    ```

    </details>

    * The key part is the `ANSWER SECTION`.  You can see we got the replies we'd expect.

    ```
    ;; ANSWER SECTION:
    files.lab.example.      3600    IN      CNAME   lab-server.lab.example.
    lab-server.lab.example. 3600    IN      A       192.168.192.11
    ```

### Testing from a Network Device 
This is a networking lab discussion, let's configure our lab network device to receive use our new DNS server.  

> For this example I am using a Cisco Catalyst 3650 switch (WS-C3650-24TS-S).  

1. Configure the `name-server` on the switch.

    ```
    lab-switch#conf t
    lab-switch(config)#ip name-server 192.168.192.11
    lab-switch(config)#end
    ```

1. Attempt to ping the `lab-server` by name. 

    ```
    lab-switch#ping files.lab.example
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 192.168.192.11, timeout is 2 seconds:
    !!!!!
    ```

## Advanced Configuration: Dynamic DNS Updates from ISC DHCP Server -> Bind 
If you are truly going to get the benefits of DHCP and DNS in your lab, you'll likely want DNS to work for clients who receive IP addresses via DHCP as well.  

Checkout the [Dynamic DNS - dhcpd to bind](dynamic-dns.md) guide for details on that setup.

## Final Thoughts 
Now we have a full featured DNS server running in our lab.  This doc just scratched the surface of all the possible DNS configurations you might want to explore and use in your setup.  A few things you could explore more include: 

* IPv6 DNS records 
* DNSSEC for secure DNS 
* DNS configuration for collaboration applications and email 
* DNS lookup/response traffic flow 
* DNS over TCP