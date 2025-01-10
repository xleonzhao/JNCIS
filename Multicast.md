---
title: JNCIS Notes Multicast
permalink: /JNCIS_Notes/Multicast/
---

- [Multicast Overview](#multicast-overview)
  - [Multicast Address](#multicast-address)
  - [Multicast protocols](#multicast-protocols)
  - [Reverse Path Forwarding (RPF)](#reverse-path-forwarding-rpf)
  - [PIM Rendezvous Points (RP)](#pim-rendezvous-points-rp)
  - [Multicast Operation Theory](#multicast-operation-theory)
- [PIM RP](#pim-rp)
  - [Static Configuration](#static-configuration)
  - [Auto-RP](#auto-rp)
  - [Bootstrap Routing](#bootstrap-routing)
- [The Multicast Source Discovery Protocol](#the-multicast-source-discovery-protocol)
  - [Operational Theory](#operational-theory)
  - [Mesh Groups](#mesh-groups)
  - [Anycast RP](#anycast-rp)
- [Reverse Path Forwarding](#reverse-path-forwarding)
  - [Creating a New RPF Table](#creating-a-new-rpf-table)
  - [Install local routes](#install-local-routes)
  - [Install static routes](#install-static-routes)
  - [Install OSPF routes](#install-ospf-routes)
  - [Install ISIS routes](#install-isis-routes)
  - [Install BGP routes](#install-bgp-routes)
  - [Using an Alternate RPF Table](#using-an-alternate-rpf-table)
  - [IGMP](#igmp)
- [Summary](#summary)
- [Check List](#check-list)

Multicast Overview
------------------

-   <http://www.cisco.com/univercd/cc/td/doc/cisintwk/ito_doc/ipmulti.htm>
-   [IP Multicast](/:w:IP_Multicast "wikilink")
-   [Multicast Overview (PDF)](/Media:Multicast_Overview.pdf "wikilink")
-   [Multicast Security (PDF)](/Media:Multicast_Security.pdf "wikilink")
-   [Multicast Tutorial (PPT)](/Media:Multicast_Tutorial.ppt "wikilink")

### Multicast Address

-   224/4 (224.0.0.0 - 239.255.255.255)
    [1](http://www.iana.org/assignments/multicast-addresses)
    -   224.0.0.0/24: Reserved link-local address
        -   224.0.0.1: All systems on this subnet
        -   224.0.0.2: All routers on this subnet
        -   224.0.0.5: OSPF routers
        -   224.0.0.6: OSPF designated routers
        -   224.0.0.13 All PIM Routers
        -   224.0.0.22 IGMP
    -   224.0.1.0/24: Internetwork Control Block
    -   232.0.0.0/8: Source Specific Multicast address block
    -   233.0.0.0/8 GLOP Block [2](http://www.ietf.org/rfc/rfc3180.txt)
        -   For example, the AS 62010 is written in hex as F23A.
            Separating out the two octets F2 and 3A, we get 242 and 58
            in decimal. This would give us a subnet of 233.242.58.0 that
            would be globally reserved for AS 62010 to use.
    -   239.0.0.0/8: Administratively Scoped
        [3](http://www.ietf.org/rfc/rfc2365.txt). Private multicast
        address.
    -   Ethernet multicast address: 01-00-5e-00-00-00 to
        01-00-5e-7f-ff-ff. Only the 23 least significant bits of the IP
        multicast group are placed in the frame. "If two hosts on the
        same subnet each subscribe to a different multicast group whose
        address differs only in the first 5 bits, Ethernet packets for
        both multicast groups will be delivered to both hosts, requiring
        the network software in the hosts to discard the unrequired
        packets."

### Multicast protocols

-   Protocol Independent Multicast (PIM)
    [4](http://www.networksorcery.com/enp/protocol/pim.htm) to multicast
    is similar to IGP for unicast
    -   PIM-DM (Dense Mode): assume every one is interested in a
        multicast group. Sends multicast traffic to every one until
        someone prunes the forwarding tree.
    -   PIM-SM (Sparse Mode): assume no one is interested in a multicast
        group unless one explicitly asks. Multicast traffic only
        forwards to someone interested to receive it.
    -   Auto-RP
    -   Bootstrap RP
-   Multicast Source Discovery Protocol (MSDP)
    [5](http://www.networksorcery.com/enp/protocol/msdp.htm) to
    multicast is similar to EGP for unicast
    -   Anycast
-   Internet Group Management Protocol (IGMP)
    [6](http://www.networksorcery.com/enp/protocol/igmp.htm)

### Reverse Path Forwarding (RPF)

-   Unicast we care where the destination is going to, multicast we care
    where the source is coming from.
-   RPF check is to check an incoming packet's source IP address against
    its RPF table (default is inet.0, or inet.2 if MBGP is used). Only
    if the packet comes into the interface via which the router would
    unicast to reach source IP, then check succeeds.
-   RPF check is to prevent loop because if unicast has no loop, RPF
    will have no loop either.
-   RPF check is kicked in when one m-router received multicast traffic
    from different neighbors. Traffic fails RPF check will be discarded.
-   *RPF neighbor* for a particular IP address is the unicast nexthop to
    reach the IP address.
-   Why called RPF? "The JOIN messages are forwarded in the reverse
    direction of the path from the RP to receiver. The inbound interface
    of a JOIN message is added to the outbound interface list for
    forwarding multicast data packets."

### PIM Rendezvous Points (RP)

-   Basic fact of multicast: sender don't know priorly who/where
    receivers will be; receiver don't know priorly who/where the sender
    will be.
-   To meet sender and receiver, let's meet in a public place first: RP.

### Multicast Operation Theory

(writing convention: *who*, **action**, PROTOCOL MESSAGE)

-   Stage 0: Every m-router need to know who/where RP is as a prior
    knowledge.
    -   via static config, Auto-RP, or Bootstrap RP.

<!-- -->

-   Stage 1: Receiver arrives RP
    1.  Receiver sends out IGMP REPORT message to indicate its interest
        in a particular group, say, *G1*;
    2.  The nearby m-router, *R1*, creates (\*, *G1*) state and **turn
        on** that interface towards Receiver for *G1*; (**turn on**
        means to add the message's incoming interface to outbound
        interface list for (\*,G) or (S,G). Later multicast traffic will
        be multiplied and forwarded to all outbound interfaces.)
    3.  *R1* sends PIM JOIN (\*,*G1*) to RP; (PIM JOIN is sent to the
        outgoing interface which connects its RPF neighbor of RP. PIM
        JOIN message destination address is 224.0.0.13:
        ALL-PIM-ROUTERS.)
    4.  m-routers along the path between *R1* and RP creates (\*, *G1*)
        and **turn on** the (\*, *G1*) interface towards *R1*;
    5.  RP receives PIM JOIN message creates (\*, *G1*) state and **turn
        on** the (\*, *G1*) interface towards to *R1*. Now, a Rendezvous
        Points Tree (RPT) rooted at RP is established to reach *R1*.

    -   An example of PIM JOIN (\*,G) message:

<!-- -->

    Apr 27 08:39:15 PIM fe-0/0/0.0 RECV 10.222.61.1 -> 224.0.0.13 V2
    Apr 27 08:39:15   JoinPrune to 10.222.61.2 holdtime 210 groups 1 sum 0xb354 len 34
    Apr 27 08:39:15   group 224.6.6.6 joins 1 prunes 0
    Apr 27 08:39:15     join list:
    Apr 27 08:39:15       source 192.168.48.1 flags sparse,rptree,wildcard <<< source listed as RP, but it means *

-   Stage 2: Source arrives RP
    1.  Source, *S1*, starting sending multicast traffic to group *G1*;
    2.  The nearby m-router, say *R2*, creates (*S1*, *G1*) state;
    3.  For each multicast packet, *R2* encapsulates it into a PIM
        REGISTER message then unicast it to RP;
    4.  RP receives PIM REGISTER message, decapsulates it and forwards
        it down to RPT;
    5.  After couple of PIM REGISTER message, RP sends PIM REGISTER-STOP
        and a PIM JOIN (*S1*, *G1*) to *S1*;
    6.  m-routers along the path between (RP, *S1*) creates (*S1*, *G1*)
        state and **turn on** the (*S1*, *G1*) interface towards to RP;
    7.  *R2* stops encapsulation multicast traffic and forwards native
        multicast data to RP.

    -   Message example:

<!-- -->

    PIM REGISTER
    Apr 27 12:29:34 PIM fe-0/0/2.0 RECV 10.250.0.123 -> 192.168.48.1 V1
    Apr 27 12:29:34   Register Source 1.1.1.1 Group 224.6.6.6 sum 0xdbfe len 292

    PIM JOIN (S,G)
    Apr 27 12:29:45 PIM fe-0/0/2.0 SENT 10.222.6.1 -> 224.0.0.13 V2
    Apr 27 12:29:45   JoinPrune to 10.222.6.2 holdtime 210 groups 1 sum 0xdbfc len 34
    Apr 27 12:29:45   group 224.6.6.6 joins 1 prunes 0
    Apr 27 12:29:45     join list:
    Apr 27 12:29:45       source 1.1.1.1 flags sparse

    PIM REGISTER-STOP
    Apr 27 12:29:46 PIM SENT 192.168.48.1 -> 10.250.0.123 V1
    Apr 27 12:29:46   RegisterStop Source 1.1.1.1 Group 224.6.6.6 sum 0xf3ee len 16

-   Stage 3: Switch - source meets receivers
    1.  *R1* receives multicast traffic;
    2.  *R1* sends PIM JOIN (*S1*,*G1*) towards *R2*;
    3.  m-routers along the path between *R1* and *R2* creates (*S1*,
        *G1*) state if it doesn't have one and **turn on** the (*S1*,
        *G1*) interface towards *R1*;
    4.  *R2* receives PIM JOIN message, **turn on** the (*S1*, *G1*)
        interface towards to *R1* if needed. Now, a Shortest Path Tree
        (SPT) is established rooted from *R2* to *R1* and others.

<!-- -->

-   Stage 4: Receiver leaves RP
    1.  Now *R1* receives two identical copies of multicast data, one
        from RP, one from *S1*;
    2.  *R1* sends PIM PRUNE to RP with RP bit set
    3.  m-routers along the path between *R1* and RP forward PIM PRUNE
        message to RP due to RP bit set;
    4.  RP **turn off** the (*S1*, *G1*) interface towards to *R1* if
        *R1* is the only receiver on that interface (turn off means to
        remove the message's incoming interface from outbound interface
        list for (\*,G) or (S,G).)

<!-- -->

-   Stage 5: Source leaves RP
    1.  if RP ountbound interface list for (*S1*, *G1*) is empty, RP
        sends PIM PRUNE message to *R2*;
    2.  m-router along the path between RP and *R2* **turn off** (*S1*,
        *G1*) interface towards RP.
    3.  Now multicast traffic only flow on SPT.

    -   Message example:

<!-- -->

    Apr 27 12:29:49 PIM fe-0/0/0.0 SENT 10.222.61.1 -> 224.0.0.13 V2
    Apr 27 12:29:49   JoinPrune to 10.222.61.2 holdtime 210 groups 1 sum 0xa3fc len 34
    Apr 27 12:29:49   group 224.6.6.6 joins 0 prunes 1
    Apr 27 12:29:49     prune list:
    Apr 27 12:29:49     source 1.1.1.1 flags sparse,rptree <<< rptree=RP bit set

-   Stage 6: Maintain the relationship
    -   *R1* periodically sends PIM JOIN to *S1*
    -   *R1* periodically sends PIM JOIN+PRUNE to RP to tell RP it is
        still interested in *G1* but not from *S1*
    -   Message example

<!-- -->

    Apr 27 12:32:42 PIM so-0/1/3.0 SENT 10.222.3.1 -> 224.0.0.13 V2
    Apr 27 12:32:42   JoinPrune to 10.222.3.2 holdtime 210 groups 1 sum 0xb3fd len 34
    Apr 27 12:32:42   group 224.6.6.6 joins 1 prunes 0
    Apr 27 12:32:42     join list:
    Apr 27 12:32:42       source 1.1.1.1 flags sparse

    Apr 27 12:31:22 PIM fe-0/0/0.0 SENT 10.222.61.1 -> 224.0.0.13 V2
    Apr 27 12:31:22   JoinPrune to 10.222.61.2 holdtime 210 groups 1 sum 0xab31 len 42
    Apr 27 12:31:22   group 224.6.6.6 joins 1 prunes 1
    Apr 27 12:31:22     join list:
    Apr 27 12:31:22       source 192.168.48.1 flags sparse,rptree,wildcard
    Apr 27 12:31:22     prune list:
    Apr 27 12:31:22       source 1.1.1.1 flags sparse,rptree

PIM RP
------

-   this is to answer a simple question: where is RP?
    -   answer is static config, Auto-RP, or Bootstrap Routing.
    -   if multiple methods are used for RP mapping, preference is given
        BSR &gt; AutoRP &gt; static.
-   PIM packet format [7](http://www.ietf.org/rfc/rfc2362.txt) (after IP
    header)

<!-- -->

         0                   1                   2                   3
         0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        |PIM Ver| Type  | Reserved      |           Checksum            |
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

            PIM Ver
                  PIM Version number is 2.

            Type  Types for specific PIM messages.  PIM Types are:

               0 = Hello
               1 = Register
               2 = Register-Stop
               3 = Join/Prune
               4 = Bootstrap
               5 = Assert
               6 = Graft (used in PIM-DM only)
               7 = Graft-Ack (used in PIM-DM only)
               8 = Candidate-RP-Advertisement

### Static Configuration

    > show configuration protocols pim rp
    local {
      address 192.168.48.1;
    }

### Auto-RP

-   better than static because it is dynamic, providing redundancy and
    fail-over features.
-   cisco proprietary, junos supports
-   each router is one of the following: candidate RP, mapping agent,
    and regular one.

candidate RP: potentially be RP for a particular group. It sends out Cisco-RP-Announce packet in PIM dense mode to 224.0.1.39 /32.

<!-- -->

    user@Sangiovese> show configuration protocols pim rp
    local {
      address 192.168.24.1;
    }
    auto-rp announce;

    Apr 27 14:54:45 PIM SENT 192.168.24.1 -> 224.0.1.39+496 AutoRP v1
    Apr 27 14:54:45   announce hold 150 rpcount 1 len 20 rp 192.168.24.1
    Apr 27 14:54:45   version 2 groups 1 prefixes 224.0.0.0/4

mapping agent: select who will be RP for a group then sends out result in Cisco-RP-Discovery in PIM dense mode to 224.0.1.40 /32.

-   selecting RP is based on 1) candidate RP who is announcing more
    specific wins; 2) candidate RP with highest IP address wins.

<!-- -->

    user@Chianti> show configuration protocols pim rp
    auto-rp mapping;

    Apr 27 14:55:10 PIM SENT 192.168.20.1 -> 224.0.1.40+496 AutoRP v1
    Apr 27 14:55:10   mapping hold 150 rpcount 1 len 20 rp 192.168.24.1
    Apr 27 14:55:10   version 2 groups 1 prefixes 224.0.0.0/4

regular router: listen Cisco-RP-Discovery message and install RP-group mapping.

<!-- -->

    user@Zinfandel> show configuration protocols pim rp
    auto-rp discovery;

### Bootstrap Routing

-   standard method using PIMv2.
-   candidate BSR (BootStrap Router) sends out *bootstrap message*. BSR
    with highest priority wins. If tie, highest IP address wins.
-   BSR message format:

```
`    0                   1                   2                   3`
`    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |PIM Ver| Type=4| Reserved      |           Checksum            |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |         Fragment Tag          | Hash Mask len | BSR-priority  |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                 Encoded-Unicast-BSR-Address                   |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                         Encoded-Group Address-1               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   | RP-Count-1    | Frag RP-Cnt-1 |         Reserved              |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+----+`
`   |                 Encoded-Unicast-RP-Address-1                  |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    |`
`   |          RP1-Holdtime         | RP1-Priority  |   Reserved    |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    |`
`   |                 Encoded-Unicast-RP-Address-2                  |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    |`
`   |          RP2-Holdtime         | RP2-Priority  |   Reserved    |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    |`
`   |                               .                               |    +---> total m RPs for Encoded-Group Address-1`
`   |                               .                               |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    |`
`   |                 Encoded-Unicast-RP-Address-m                  |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+    |`
`   |          RPm-Holdtime         | RPm-Priority  |   Reserved    |    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+----+`
`   |                         Encoded-Group Address-2               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                               .                               |`
`   |                               .                               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                         Encoded-Group Address-n               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   | RP-Count-n    | Frag RP-Cnt-n |          Reserved             |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                 Encoded-Unicast-RP-Address-1                  |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |          RP1-Holdtime         | RP1-Priority  |   Reserved    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                 Encoded-Unicast-RP-Address-2                  |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |          RP2-Holdtime         | RP2-Priority  |   Reserved    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                               .                               |`
`   |                               .                               |`
`   |                               .                               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                 Encoded-Unicast-RP-Address-m                  |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |          RPm-Holdtime         | RPm-Priority  |   Reserved    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
```

Fragment Tag: Bootstrap message is divided up into \`semantic fragments', if the original message exceeds the maximum packet size boundaries. Fragment Tag is a randomly generated number, acts to distinguish the fragments belonging to different Bootstrap messages; fragments belonging to same Bootstrap message carry the same \`Fragment Tag'.
Hash Mask len: The length (in bits) of the mask to use in the hash function. For IPv4 we recommend a value of 30. For IPv6 we recommend a value of 126.
RP count: Total number of RPs for this group.
Fragment RP Count: When a bootstrap message is fragmented, this field displays the total of RP addresses present in this fragment for the advertised group address range.
Encoded Unicast Address: there is an **error** in above format since it takes 6 octets. The format is:

```
` 0                   1                   2                   3`
` 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1`
`+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`| Addr Family   | Encoding Type |     Unicast Address           |`
`+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`| Unicast Address (cont'd)      |`
`+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
```

-   Addr Family: 1=IP
-   Encoding Type: 0=native encoding (default value).

Encoded Group Address; it takes 8 octets. The format is:

```
` 0                   1                   2                   3`
` 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1`
`+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`| Addr Family   | Encoding Type |   Reserved    |  Mask Len     |`
`+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`|                Group multicast Address                        |`
`+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
```

-   each router with a local RP configuration sends a Candidate-RP
    Advertisement (C-RP-Adv) message unicast to BSR.
    -   It is currently a best practice to make each C-RP a C-BSR as
        well. This aids in network troubleshooting as well as the timely
        advertisement of the RP information from the BSR in the domain.
    -   It is possible that more than one RP is configured for the same
        group. JUNOS randomly pick one RP for that group.
    -   C-RP-Adv message format

```
`    0                   1                   2                   3`
`    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |PIM Ver| Type=8| Reserved      |           Checksum            |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   | Prefix-Cnt    |   Priority    |             Holdtime          |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                 Encoded-Unicast-RP-Address                    |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                         Encoded-Group Address-1               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                               .                               |`
`   |                               .                               |`
`   |                               .                               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
`   |                         Encoded-Group Address-n               |`
`   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
```

Priority: Lowest wins
Holdtime: The amount of time the advertisement is valid. This field allows advertisements to be aged out. C-RP will periodically resend its C-RP-Adv message before Holdtime expires.
Encoded Unicast RP Address: 6 octets
Encoded Group Address: 8 octets.

-   BSR collects all C-RP-Adv then sends out a single *bootstrap
    message* with RP mapping information (which RP is responsible for
    which group).

<!-- -->

    user@Merlot> show configuration protocols pim rp
    bootstrap-priority 100;

    user@Sangiovese> show configuration protocols pim rp
    bootstrap-priority 200;

    show pim bootstrap

    Aug 21 14:24:29 PIM at-0/1/0.801 RECV 209.16.195.169 -> 224.0.0.13 V2 Bootstrap sum 0x9ed7 len 1444
    Aug 21 14:24:29    tag 27912 masklen 30 priority 255 bootstrap router 209.16.195.1
    Aug 21 14:24:29       group 224.0.1.0 count 1 fragcount 1
    Aug 21 14:24:29          rp address 209.16.195.17 holdtime 150 priority 10
    Aug 21 14:24:29       group 224.0.2.0 count 1 fragcount 1
    Aug 21 14:24:29          rp address 209.16.195.19 holdtime 150 priority 1
    Aug 21 14:24:29       group 224.0.5.0 count 1 fragcount 1
    Aug 21 14:24:29          rp address 209.16.195.19 holdtime 150 priority 1
    Aug 21 14:24:29       group 224.0.17.0 count 1 fragcount 1
    Aug 21 14:24:29          rp address 209.16.195.17 holdtime 150 priority 10

The Multicast Source Discovery Protocol
---------------------------------------

-   RP in one domain tells the "x.x.x.x is current *active sources* and
    y.y.y.y is its RP, and optioanlly this is PIM register message
    x.x.x.x sent to y.y.y.y" to another RP (in same or different
    domain), who then can PIM JOIN the SPT.
-   Using TCP, port is 639.
-   If MSDP used to connect two ASs, those ASs are normally neighboring
    ASs.

### Operational Theory

-   Session is kept via keepalive message. 75 seconds hold time.
-   The originating RP floods the SA message to its MSDP peers, which
    further flood the message to their peers.
-   Based on peer-RPF flooding rules, which is trying to locate the
    cloest peer to RP in SA to avoid endless flooding, each MSDP peer
    accepts SA only one of following is true:
    -   the advertising peer belongs to a configured mesh group
    -   the advertising peer is configured as default peer
    -   the advertising peer is RP in SA
    -   does a route loop on RP in SA:
        -   returns a BGP route and protocol next hop is the advertising
            peer
        -   returns a BGP route and AS path includes the AS where the
            peer resides
        -   returns an IGP route and the physical nexthop to RP and to
            peer is same
-   If any of the receiving MSDP routers have a local (\*,G) state for
    the advertised group address, they generate a PIM Join message
    addressed to the multicast source.
    -   "More precisely, the (S,G) Join message from the last-hop router
        travels only as far as the first router with an existing (S,G)
        state, which adds an interface to its outgoing interface list.
        In practice, the (S,G) Join rarely leaves the AS in which it was
        created."
-   The MSDP router then forwards the native multicast traffic down its
    local RPT towards the interested listeners. As the traffic reaches
    the last-hop router, a Join message is created and a separate SPT is
    formed between the local router and the first-hop router.
-   SA message format

<!-- -->

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |       1       |           length=x+y          |  Entry Count  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          RP Address                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           Reserved            |  Sprefix Len  | \
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  \
    |                         Group Address                         |   ) repeat Entry Count times
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  /
    |                         Source Address                        | /
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

length: x=(8+12\*"Entry Count"), 8 is the first 8 octets of SA message, 12 is the length for each Entry. If y&gt;0, the *last* Entry has an encapsulated IP packet (normally PIM register message), and y=total_length field in encapulsated IP header.

### Mesh Groups

-   when receiving SA from a peer in peer group, not forward it to other
    peers in same group.

<!-- -->

    user@Zinfandel> show configuration protocols msdp
    group set-as-mesh {
      mode mesh-group;
      local-address 192.168.56.1;
      peer 192.168.40.1;
      peer 192.168.20.1;
      peer 192.168.24.1;
    }
    group remaining-peers {
      local-address 192.168.56.1;
      peer 192.168.52.1;
    }

### Anycast RP

-   Anycast address is configured under lo0.0 which will be advertised
    via IGP by default.

<!-- -->

    user@Sangiovese> show configuration interfaces lo0
    unit 0 {
      family inet {
        address 192.168.24.1/32 {
          primary;
        }
        address 192.168.200.1/32;
      }
    }

-   Several RPs share same IP address.

<!-- -->

    user@Zinfandel> show configuration protocols pim rp
    static {
      address 192.168.200.1;
    }

-   m-router picks one which is the closest in terms of IGP cost.
-   Reason to use anycast:
    -   AutoRP and BSR make RP redundant and failover, but failover
        takes long time: the time to detect current RP fails, the time
        for backup RP to take over.
    -   Anycast RP will make failover fast, which is equal to IGP
        failover time.
-   Anycast need MSDP, because RP closest to source is not necessarily
    the same RP to receiver, if that's the case, source arrives at
    McDonald while receiver waits at KFC.
-   Once RP receives a SA message which contains PIM REGISTER message,
    it decapsulates the PIM REGISTER message and forward down to RPT,
    which will prompt a PIM JOIN (S,G) sent to source by one m-router
    down RPT.

Reverse Path Forwarding
-----------------------

RPF check is by default done in inet.0. But we can use other routing
table such as inet.2, or others. Problem is inet.2 has to have enough
unicast routing information. How?

### Creating a New RPF Table

Answer is to use *rib-group*. *Rib-group* means a routing protocol can
install a route into a group of routing tables, and the first one is
always the primary. *Import-rib* is the statement to control which set
of routing tables a routing protocol can install, hence *import*, a
route.

After defining a rib-group, apply it to a routing protocol.

Rib-group installs identical routing information into multiple tables,
but we need to make inet.0 and inet.2 different (that's why we use
inet.2). How? ISIS multicast TLVs and MBGP have sth to do that.

### Install local routes

```
`user@Chianti> show configuration routing-options`
`interface-routes {`
`  rib-group inet populate-inet2;`
`}`
`rib-groups {`
`   populate-inet2 {`
`    import-rib [ inet.0 inet.2 ];`
`  }`
`}`
```

### Install static routes

```
`user@Zinfandel> show configuration routing-options`
`static {`
`  rib-group populate-inet2;`
`  route 192.168.57.0/24 next-hop 10.222.62.3;`
`}`
`rib-groups {`
`  populate-inet2 {`
`    import-rib [ inet.0 inet.2 ];`
`  }`
`}`
```

### Install OSPF routes

```
`user@Chardonnay> show configuration protocols ospf`
`rib-group populate-inet2;`
```

### Install ISIS routes

Two methods, one is use rib-group same as OSPF, the other is to use
multi-topology TLVs, and this one we can alter multicast forwarding
path.

```
`user@Merlot> show configuration protocols isis`
`multicast-topology;`
`level 1 disable;`
`interface at-0/1/0.0;`
`interface ge-0/2/0.0 {`
`  level 2 ipv4-multicast-metric 5;`
`}`

`user@Sherry> show isis database extensive Sherry.00-00 | find TLV`
`TLVs:`
`  ...`
`  IS neighbor: Sherry.02, Internal, Metric: default 10`
`  ...`
`  Multicast IS neighbor: Sherry.02, Metric: default 5`
`  ...`
`  Multicast IP prefix: 192.168.16.1/32 metric 0 up`
`  Multicast IP prefix: 10.222.29.0/24 metric 5 up`
`  Multicast IP prefix: 10.222.28.0/24 metric 10 up`
```

### Install BGP routes

```
`user@Chablis> show configuration policy-options | find alter-inet2-localpref`
`policy-statement alter-inet2-localpref {`
`  term set-to-200 {`
`    to rib inet.2;`
`    then {`
`      local-preference 200;`
`    }`
`  }`
`}`

`user@Sherry> show configuration protocols bgp`
`group internal {`
`  type internal;`
`  import alter-inet2-localpref; `
`  local-address 192.168.16.1;`
`  family inet {`
`    any;`
`  }`

`user@Cabernet> show route 1.1.1/24`
`inet.0: 23 destinations, 32 routes (23 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`

`1.1.1.0/24   *[BGP/170] 00:31:15, MED 0, localpref 100, from 192.168.32.1`
`                AS path: 65000 I`
`              > via so-0/1/1.0`

`inet.2: 20 destinations, 23 routes (20 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`
`1.1.1.0/24   *[BGP/170] 00:07:58, MED 0, localpref 200, from 192.168.52.1`
`                AS path: 65000 I`
`              > via so-0/1/2.0`
```

### Using an Alternate RPF Table

Inform PIM/MSDP to use inet.2

```
`user@Cabernet> show configuration routing-options use-inet2-for-rpf `
`import-rib inet.2;`

`user@Cabernet> show configuration protocols msdp`
`rib-group use-inet2-for-rpf;`

`user@Cabernet> show configuration protocols pim`
`rib-group inet use-inet2-for-rpf;`

`user@Cabernet> show multicast rpf inet summary`
`Multicast RPF table: inet.2, 20 entries`
```

### IGMP

Summary
-------

In this chapter, we saw how the JUNOS software uses each of the three RP
election mechanisms to construct a rendezvous point tree (RPT) and a
shortest path tree (SPT) in a PIM sparse-mode network. The simplest RP
election mechanism to configure was static assignment of the RP address.
However, this method doesn’t provide capabilities for dynamic failover
and redundancy, so we explored both Auto-RP and Bootstrap routing. We
discussed the various configuration options and operation of Auto-RP and
BSR within the context of a single PIM domain. Each RP election method
displayed the ability to assign roles to certain routers in the network
and dynamically select and learn the address of the RP for each
multicast group address range.

We concluded the chapter with a discussion of the Multicast Source
Discovery Protocol (MSDP) and explained how it exchanges knowledge of
active multicast sources between peer routers. We explored the use of
MSDP within a single PIM domain for creating a virtual RP. This use of
MSDP is commonly referred to as anycast RP since multiple physical
routers share a common RP address. All non-RP routers use this anycast
address as the RP address for all possible multicast groups and transmit
their protocol packets to the metrically closest anycast router. This is
not the only use for MSDP, however, since it was originally designed for
communications between RP routers in different PIM domains. Each
individual domain elects an RP using one of the three defined methods,
and these routers are then peered together using MSDP. Routers create an
RPT between themselves and the RP in their respective domain. Active
multicast sources are transmitted between peers using SA messages. When
a peer router receives a message for a group that belongs to an
established RPT, that peer creates an SPT between itself and the source
of the traffic. In this method, data traffic flows across multiple PIM
domains.

Check List
----------

-   Be able to describe the operation of Auto-RP.
-   Be familiar with the process of electing a bootstrap router and
    selecting an RP in a BSR domain.
-   Be able to describe the operation of MSDP within a single PIM
    domain.
-   Be able to describe the operation of a MSDP in a multidomain
    environment.
-   Understand and be able to identify methods for populating a new RPF
    table.
-   Be able to describe methods available for advertising
    multicast-specific information.
