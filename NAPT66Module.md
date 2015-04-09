The NAPT66 kernel module based on the Netfilter

# Introduction #

The NAPT66 kernel module based on the Netfilter


# Details #

> The NAPT66 kernel module based on the Netfilter
> NAPT66 is based on the netfilter framework on the Linux core architecture, version 2.6.30.
  1. Netfilter architecture analysis

> Netfilter is an important part of the network core of Linux, which provides a new mechanism for operation of network packets. In Netfilter , each protocol defines several hooks ( IPv6 hooks that are defined below), the packets of each protocol will throughout these hooks according to some certain rules, each hook can mount some handler functions. The kernel module can register handlers on the hook, to manipulate packets.
> At each hook, when the packet is processed, if you find that you need to drop packets, the packet will be released immediately instead of into the subsequent process; if the return value is accepted, the packet will go to the next hook.

> 2.NAPT process flow

> 2.1  connecting track section
> System creates a connection entry that records the information about each network connection, including the source address and port or destination address and port etc. All packets belonging to the connection will simply match the connection record, instead of generating a new record.  NAPT66 System uses a connection track table to keep the tracks of all connections. In order to improve the productivity of the connection track table, we use a hash algorithm, the specific method is using Elf algorithm: protocol, addresses and ports are composed of a string, after the operation, we will get the hash value.
> When the packet accesses the netfilter framework, it will through the various hooks. First, find the connection track table, if there is no match, create a new connection record, record connection-related information.

> 2.2 NAPT section
> After the connection track module records the information of a packet, then access NAPT process. According to NAPT rules, we need to complete two jobs: modification of the packet address and port (according to the flow of packets to decide to modify the source address or destination address); modification of the upper-layer protocols.

> The first part, address translation.
> first, when the packet arrives NF\_INET\_POST\_ROUTING hook from the internal network to the external network, find the connection track tables, if the lookup fails, a new entry of the  connection track table will be created, then do SNAT (that is, modify the packet's source address and port), the packet’s address changes to the Internet IPv6 address of the NAPT66 device. On the other hand, when the packet arrives NF\_INET\_PRE\_ROUTING hook from the external network to the internal network, still find connection track table firstly, if the lookup fails, NAPT66 is no longer processing the packet, then do reverse SNAT (that is, modify the packet's destination address and port), in order to make the packet can correctly arrive internal network.


> The second part, the handling mechanism of upper-layer protocols, includes packet checksum recalculation ,active FTP ,ICMPv6 。

  1. The calculation of the checksum

> For a large number of packets in the network, if we recalculate each complete packet’s checksum, then the operation is very large. In order to improve the efficiency of the system, our calculation of the checksum does not have the traditional method, but to fine-tune the checksum.

> Because, for most of the packets, we only need modify the packet’s (the source / destination) address and port fields, thanks to the outstanding design of the IPv6 Protocol, the fields are fixed length and alignment, so after NAPT, checksum’s difference is caused by the difference of the corresponding address and port fields. For every entry of the connection track table, you need to calculate the difference when creating an entry for the first time, when processing packets that belong to the connection. Simply on the basis of the difference to fine-tune the checksum can significantly improve the operational efficiency of the system.

> 2. Active FTP support

> FTP-ALG for the active FTP support was provided.

> 3. ICMPv6 error messages

> ICMPv6 error message which is used for reporting IPv6 packet’s errors, including destination unreachable, bag is too long, a timeout, parameters and so on.
> NAPT66 system analysis the type of ICMPv6 message and payload, modifies the packet, properly handles the error message. The internal users can transparently receive these error messages, which will improve the network experience to users.

