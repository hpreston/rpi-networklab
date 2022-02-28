# Dynamic DNS - Maintaining accurate A and PTR records for DHCP leases
* [Creating a secure key for authenticating updates](#creating-a-secure-key-for-authenticating-updates)
* [Updating the ISC DHCP Server Configuration](#updating-the-isc-dhcp-server-configuration)
* [Updating the Bind Configuration](#updating-the-bind-configuration)
* [Applying changes](#applying-changes)
* [Monitoring for Dynamic DNS Updates](#monitoring-for-dynamic-dns-updates)
* [Final Thoughts](#final-thoughts)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


In [Using a Raspberry Pi as a DHCP Server](dhcp-server.md) we learned how to setup the ISC DHCP Server on the RPi to issue IP addresses dynamically.  And in [Using a Raspberry Pi as a DNS Server](dns-server.md) we learned how to setup Bind9 on RPi to provide DNS resolution.  But what about DNS resolution for dynamically assigned addresses?  This is where Dynamic DNS, sometimes called DDNS, comes in.  

What needs to happen is that the DHCP server must send updates to the DNS server for each lease that is issued, and again when they expire.  In this document we'll look at how we can update the configurations for both ISC DHCP Server and Bind9 to allow DDNS. 

> Note: This document is going to cover a basic setup and configuration.  For full details on DDNS setup consult the documentation for ISC DHCP server and Bind9.

## Creating a secure key for authenticating updates
It wouldn't be very good security to allow just ANYONE to update the DNS servers records.  It is important that updates are sent from trusted and approved clients.  Authentication of DDNS is done with a "key" that is known by both the DHCP server and the DNS server. 

1. We will use the `ddns-confgen` utility that was installed with bind/bind-utils to generate the key file contents. 
    > The argument `-k ddns-key` allows us to configure the "key-name" that we want to use.  If we don't provide one a default key-name is used. 

    ```bash
    ddns-confgen -k ddns-key

    # Output

    # To activate this key, place the following in named.conf, and
    # in a separate keyfile on the system or systems from which nsupdate
    # will be run:
    key "ddns-key" {
            algorithm hmac-sha256;
            secret "rxL46YrXMAiSOeUkLP4DugwoyuOohhn9VeNzunlYxWs=";
    };

    # Then, in the "zone" statement for each zone you wish to dynamically
    # update, place an "update-policy" statement granting update permission
    # to this key.  For example, the following statement grants this key
    # permission to update any name within the zone:
    update-policy {
            grant ddns-key zonesub ANY;
    };

    # After the keyfile has been placed, the following command will
    # execute nsupdate using this key:
    nsupdate -k <keyfile>
    ```

    * The output of the command provides three things: 
        1. The contents of the keyfile that will be `included` by both the dns and dhcp servers
        1. An example configuration to include in the bind zone file that allows the updating of `ANY` data 
        1. An example `nsupdate` command to perform dynamic DNS updating using the file.  This could be done from a client rather than the DHCP server.
    
1. Using the output from `ddns-confgen` create the key file.  It could be placed anywhere, but I'm placing it in the bind configuration directory. 

    ```bash
    cat /etc/bind/ddns-keyfile.key

    # Output 
    key "ddns-key" {
            algorithm hmac-sha256;
            secret "rxL46YrXMAiSOeUkLP4DugwoyuOohhn9VeNzunlYxWs=";
    };

    ```

## Updating the ISC DHCP Server Configuration
With the keyfile created, we can now update the configuration of our DHCP server to send dynamic updates to the DNS server. 

1. Add the following configuration to the `/etc/dhcp/dhcpd.conf` file

    ```bash
    # Include the Dynamic DNS Keyfile 
    include "/etc/bind/ddns-keyfile.key";

    # Dynamic DNS Update Configuration
    ddns-update-style standard;
    ddns-rev-domainname "in-addr.arpa.";
    deny client-updates;
    do-forward-updates on;
    update-optimization off;        # Turn this off to always send DDNS updates. Good for lab, bad for prod
    update-conflict-detection off;  # I turn this off in the lab to always update DNS even if a host changes IPs/interfaces

    # Include the Dynamic DNS Keyfile 
    include "/etc/bind/ddns-keyfile.key";

    zone lab.example. {             # name of your forward DNS zone
            primary 192.168.192.11; # DNS server IP address here
            key ddns-key;           # Must reference the name of the "key" in the keyfile
    }

    zone 192.168.192.in-addr.arpa. {   # name of your reverse DNS zone
            primary 192.168.192.11;    # DNS server IP address here
            key ddns-key;              # Must reference the name of the "key" in the keyfile
    }
    ```

## Updating the Bind Configuration
In order to accept the dynamic updates from the DHCP server, Bind must be configured to allow them. 

1. Update the configuration in the file `/etc/bind/named.conf.local` to: 
    * `include` the key file
    * set the `update-policy` for both the forward and reverse zones

    ```php
    //
    // Do any local configuration here
    //

    // Consider adding the 1918 zones here, if they are not used in your
    // organization
    //include "/etc/bind/zones.rfc1918";

    // Include the Dynamic DNS Keyfile 
    include "/etc/bind/ddns-keyfile.key";

    // Forward lookup zone for lab.example
    zone "lab.example" IN {
            type master;
            file "/etc/bind/db.lab.example";

            // Dynamic DNS Update Configuration
            update-policy {
                grant ddns-key zonesub A TXT DHCID;
            };
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

            // Dynamic DNS Update Configuration
            update-policy {
                grant ddns-key zonesub PTR TXT DHCID;
            };
        };
    ```

1. As DDNS updates come into the DNS server, bind creates a journal file to track the updates and create the dynamic DNS entries.  In order for this to succeed, the `bind` user needs write permissions to the `/etc/bind` directory.  The default installation only provides read permissions by setting the `group` to `bind`, but the `owner` to `root`. Change the owner of the `/etc/bind` directory to the `bind` user to allow the creation and management of the journal files. 

    ```bash
    sudo chown bind:bind /etc/bind
    ```

## Applying changes
To apply the changes, simply restart both the dns and dhcp server services.

```bash
sudo systemctl restart bind 
sudo systemctl restart isc-dhcp-server 
```

> Note: if any errors in restarting the services are noted, check the configuration files and look for details on the error in the system journal (`journalctl -xe`)

## Monitoring for Dynamic DNS Updates
The easiest way to see if the configuration is working correctly, you can monitor the system journal on the RPi server.  

```bash
journalctl -xef -u named -u isc-dhcp-server
```

You can wait for a DHCP client to send a renew message, or you can restart the DHCP client service on the `lab-client` to force an update `sudo systemctl restart dhcpcd`.  

When an update happens, you will see messages like this in the journal.  I have added comments within to identify the steps involved.

```bash
# The DHCP Client attempts to renew its ip address by sending a DHCPREQUEST. The server replies with a DHCPACK
Feb 27 11:18:15 lab-server dhcpd[77080]: DHCPREQUEST for 192.168.192.102 from b8:27:eb:d3:22:e7 (lab-client) via eth0
Feb 27 11:18:15 lab-server dhcpd[77080]: DHCPACK on 192.168.192.102 to b8:27:eb:d3:22:e7 (lab-client) via eth0

# Bind checks if a DNS entry already exists, there isn't one
Feb 27 11:18:15 lab-server named[75396]: client @0x7f6804de28 192.168.192.11#47183/key ddns-key: updating zone 'lab.example/IN': update unsuccessful: lab-client.lab.example: 'name not in use' prerequisite not satisfied (YXDOMAIN)
# Bind deletes and readds the DHCID (DHCP Client ID info)
Feb 27 11:18:15 lab-server named[75396]: client @0x7f797ba8a8 192.168.192.11#37505/key ddns-key: updating zone 'lab.example/IN': deleting rrset at 'lab-client.lab.example' DHCID
Feb 27 11:18:15 lab-server named[75396]: client @0x7f797ba8a8 192.168.192.11#37505/key ddns-key: updating zone 'lab.example/IN': adding an RR at 'lab-client.lab.example' DHCID AAEBN40H7hrfLp1E6YojFr2meGSWZs75Bv3n0GSKLK9TFzg=
# Bind deletes and readds the A record for the clients DHCP address 
Feb 27 11:18:15 lab-server named[75396]: client @0x7f797ba8a8 192.168.192.11#37505/key ddns-key: updating zone 'lab.example/IN': deleting rrset at 'lab-client.lab.example' A
Feb 27 11:18:15 lab-server named[75396]: client @0x7f797ba8a8 192.168.192.11#37505/key ddns-key: updating zone 'lab.example/IN': adding an RR at 'lab-client.lab.example' A 192.168.192.102

# The DHCP server logs teh successful creation of the forward mapping
Feb 27 11:18:15 lab-server dhcpd[77080]: Added new forward map from lab-client.lab.example to 192.168.192.102

# Bind deletes and readds the PTR record
Feb 27 11:18:15 lab-server named[75396]: client @0x7f797ba8a8 192.168.192.11#40925/key ddns-key: updating zone '192.168.192.in-addr.arpa/IN': deleting rrset at '102.192.168.192.in-addr.arpa' PTR
Feb 27 11:18:15 lab-server named[75396]: client @0x7f797ba8a8 192.168.192.11#40925/key ddns-key: updating zone '192.168.192.in-addr.arpa/IN': adding an RR at '102.192.168.192.in-addr.arpa' PTR lab-client.lab.example.

# The DHCP server logs the successful creation of the reverse mapping
Feb 27 11:18:15 lab-server dhcpd[77080]: Added reverse map from 102.192.168.192.in-addr.arpa. to lab-client.lab.example
```

> Note: If you have host reservations configured for lab devices in your `dhcpd.conf` file you will ***NOT*** get dynamic dns updates created.  This took me by a bit of surprise, and after some looking into it this seems to be by design of the ISC DHCP Server (and might happen with other DHCP servers as well). The reason is that by default `host` reservations are NOT recorded as `leases` by the server, and therefore do NOT follow the same process as true dynamic address assignment.  There are some configuration options that can be done to get DDNS for reserved IPs, but the general advice is to manually create the DNS entries for reserved addresses.  Afterall, DDNS is meant for names/ips that are changing, and reservations don't change. 
> 
> To make this a bit easier, you can configure the `fixed-address` in a host configuration to use a DNS name rather than IP address.  For example 
> ```bash
> host lab-client {
>  hardware ethernet b8:27:eb:d3:22:e7;
>  fixed-address lab-client.lab.example;
> }
> ```

## Final Thoughts 
Alright, with dynamic dns configured our lab dns and dhcp are setup in a robust fashion that will support nearly any type of scenario we might need to explore with these protocols.  Furthermore, it is setup in a way that mimics how many enterprise networks are configured.  