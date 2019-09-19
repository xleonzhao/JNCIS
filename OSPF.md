---
title: JNCIS Notes OSPF
permalink: /JNCIS_Notes/OSPF/
---

OSPF: Open Shortest Path First

-   <http://www.networksorcery.com/enp/rfc/rfc2328.txt>
-   <http://www.networksorcery.com/enp/protocol/ospf.htm>

Link-State Advertisements
-------------------------

-   LSA is carried inside an OSPF Link State Update Packet

```
show ospf database summary
show ospf database router area 0 extensive
show ospf database network area 0 extensive
show ospf database netsummary area 3 extensive
show ospf database area 3 lsa-id 192.168.16.2 extensive
show ospf database area 3 lsa-id 192.168.16.2 extensive advertising-router 192.168.0.3
show ospf database extern
show ospf database nssa
```

These commands can be further detailed by using "extensive" and specifying "area" or router via "lsa-id".

### The Common LSA Header

-   20 octets

<!-- -->

            0           1           2           3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |        LS age         |    Options    |    LS type    |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |            Link State ID                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             Advertising Router                |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             LS sequence number                |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |     LS checksum           |         length        |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

* LS age: MaxAge is 3600 seconds by default. LSA originator will reflood its own LSA at 3000 seconds. Transit router will increment prior to reflooding.
* Options:
    -   bit 7: DN bit, if set, a router should not forward this LSA.
    -   bit 6: O bit, Sender set it if it supports Opaque LSA.
    -   bit 3: N/P bit. set if support not-so-stubby LSA
    -   bit 1: E bit, set if support type 5 external LSA.

Type:

-   1: **Router LSA** (node/link info)
-   2: Network LSA
-   3: **Newtork summary LSA** (node/link info)
-   4: ASBR summary LSA
-   5: **AS external LSA** (node-&gt;network info)
-   6: Group membership LSA (not supported)
-   7: NSSA external LSA
-   8: External attributes LSA (not supported)
-   9: Opaque LSA (link-local scope)
-   10: Opaque LSA (area-local scope)
-   11: Opaque LSA (AS-wide scope) (not supported)

LS Sequence Number: starting from 0x80000001, incremented to (be overflowed to) 0x7fffffff.

### Type 1: The Router LSA

-   Area scope
-   forms the basic inputs to SPF algorithm. router (node) and link info
    are here.

<!-- -->

            0           1           2           3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |    0    |V|E|B|    0      |        # links        |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |              Link ID                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             Link Data                 |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |     Type      |     # TOS     |        metric         |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                  ...                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |      TOS      |    0      |      TOS  metric          |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |              Link ID                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             Link Data                 |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                  ...                  |

00000VEB ("web"):

-   V (virtual): virtual link is established
-   E (extern): this is an ASBR router
-   B (border): this is an ABR router

Type:

-   Point-to-Point: sonet etc. *Link ID*=adjacent peer IP address; *Link
    Data*=local interface address
-   Transit: Ethernet. *Link ID*=DR's interface address; *Link
    Data*=local interface address
-   Stub: Loopback address, or interface in passive mode. *Link
    ID*=address; *Link Data*=mask.
-   Virtual Link: logical connection between two ABRs. *Link ID*=peer
    address; *Link Data*=local interface to remote ABR.

metric: cost of the link, one of the most important piece of information.

<!-- -->

    Type   ID          Adv Rtr     Seq        Age Opt Cksum  Len
    Router 192.168.0.1 192.168.0.1 0x80000002 422 0x2 0x1c94 60
    bits 0x3, link count 3
    id 192.168.0.2, data 192.168.1.1, type PointToPoint (1)
    TOS count 0, TOS 0 metric 1
    id 192.168.1.0, data 255.255.255.0, type Stub (3)
    TOS count 0, TOS 0 metric 1

### Type 2: The Network LSA

-   Area scope
-   Ethernet DR originates this LSA to list all OSPF routers attached to
    the same LAN segment.

<!-- -->

        0           1           2           3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             Network Mask                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |            Attached Router                |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                  ...                  |

    Type    ID          Adv Rtr     Seq        Age Opt Cksum  Len
    Network 192.168.2.2 192.168.0.3 0x80000002 369 0x2 0x7a2d 32
    mask 255.255.255.0
    attached router 192.168.0.3
    attached router 192.168.0.2

### Type 3: The Network Summary LSA

-   Area scope. If crossing area boundary, LSA will be re-generated
    instead of flooding.
-   Link State ID in LSA common header is the network being
    summaried/advertised. It says that a network (identified by *"Link
    State ID"*) can be reached via *"Advertising Router"* by the cost of
    *"metric"*.

<!-- -->

            0           1           2           3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |             Network Mask                  |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |      0        |          metric               |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |     TOS       |        TOS  metric            | <<< not used by JUNOS
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                  ...                  |

metric: default metric for almost all interface type is 1. For aggregated routes, the metric is the largest among contributing routes.

<!-- -->

    Type     ID           Adv Rtr     Seq        Age Opt Cksum  Len
    Summary *192.168.49.0 192.168.0.3 0x80000001 439 0x2 0x88cc 28
    mask 255.255.255.0
    TOS 0x0, metric 1

### Type 4: The ASBR Summary LSA

-   Area scope, just tell who is ASBR.
-   a router generates and relays a type 4 LSA from one area to another
    area when it
    -   received a type 4 LSA from backbone area, which means there
        is/are ASBR in other non-backbone areas
    -   received a type 1 LSA / router LSA from its own area with E bit
        set
-   Link State ID will be ASBR address

same packet format as Type 3, but with

-   Network Mask is constant 0
-   only metric has a meaning (distance to ASBR), others are not
    supported.

<!-- -->

    Type     ID           Adv Rtr     Seq        Age Opt Cksum  Len
    ASBRSum *192.168.48.1 192.168.0.3 0x80000001 18  0x2 0x7bd8 28
    mask 0.0.0.0
    TOS 0x0, metric 1

### Type 5: The AS External LSA

-   Domain scope. Flooding without re-generating, everyone has same
    copy.
-   Link State ID: network being advertised in this LSA

<!-- -->

            0                   1                   2                   3
            0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                         Network Mask                          |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |E|     0       |                  metric                       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      Forwarding address                       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      External Route Tag                       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |E|    TOS      |                TOS  metric                    |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      Forwarding address                       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                      External Route Tag                       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
           |                              ...                              |

-   E bit followed by 7 0s
    -   if E=1 (default): use *metric* as total cost (type 2)
    -   if E=0: use *metric* plus cost to ASBR as total cost (type 1)
-   Forwarding address: address toward which packets should be forwarded
    (like *[Protocol Next Hop](/Protocol_Next_Hop "wikilink")* for BGP
    routes). Default of 0.0.0.0 means ASBR itself.
-   External Route Tag: OSPF is not using it, other routing protocol may
    use it. default is 0.0.0.0
-   TOS / TOS metric: optional

<!-- -->

    Type   ID         Adv Rtr     Seq        Age  Opt Cksum  Len
    Extern 172.16.1.0 192.168.0.1 0x80000002 1583 0x2 0x3e6b 36
    mask 255.255.255.0
    Type 2, TOS 0x0, metric 0, fwd addr 0.0.0.0, tag 0.0.0.0

ASBR 192.168.0.1 advertised an external route 172.16.1.0/24 with total
cost = 0

### Stub Area and its Variants

-   The purpose is to reduce the database size.
-   Commonly combined with a default route.
-   Inform ABR to stop generating/injecting certain type LSA into this
    area.

Stub Area: no AS external LSA (type 5) and ASBR summary LSA (type 4) in this area, i.e., only type 1,2,3 allowed in this area.
Totally Stub Area: No AS summary LSA (type 3) in this area, i.e., only type 1,2 allowed in this area.
Not-So-Stubby Area (NSSA): allow AS external LSA (in type 7 format which then be translated to type 5) be sent out of this area, but still not into this area (still a stub), i.e., only type 1,2,7 allowed in this area.

**Configuration**

-   Stub area

For *all* routers in the stub area

    [edit protocol ospf]
    area 0.0.0.5 {
      stub

For ABR which connects stub area and backbone area, it needs to provide
default route (0/0)

    [edit protocol ospf]
    area 0.0.0.5 {
      stub default-metric 20 <<< metric for default route

-   Furthermore, to configure a totally stub area, add *no-summaries*
    after default-metric on ABR
-   To configure a NSSA, just change *stub* to *nssa*. But on ABR, to
    configure default route, syntax is:

<!-- -->

    area 0.0.0.3 {
    nssa {
      default-lsa default-metric 25; <<< this will be a type-7 LSA into NSSA
    }

-   Furthermore, you can have a totally stubby NSSA by using
    *no-summaries*. 0/0 route will become a type-3 LSA. To opt it to a
    type-7 LSA, you can use *type-7* under *default-lsa*.

### Type 7: The NSSA External LSA

-   Area scope
-   A type 7 LSA will be translated to Type 5 at area boundary
-   Same format as Type 5
-   Link State ID is the network being advertised
-   Forwarding address by default is router ID of ASBR.

<!-- -->

    Type ID         Adv Rtr      Seq        Age Opt Cksum  Len
    NSSA 172.16.6.0 192.168.48.1 0x80000006 931 0x8 0xe7e5 36
    mask 255.255.255.0
    Type 2, TOS 0x0, metric 0, fwd addr 192.168.48.1, tag 0.0.0.0

-   Option=0x8 indicates this ABR supports NSSA and is capable to
    translate type 7 into type 5.

After translation:

    Type   ID          Adv Rtr     Seq        Age Opt Cksum  Len
    Extern *172.16.6.0 192.168.0.3 0x80000005 919 0x2 0xa55f 36
    mask 255.255.255.0
    Type 2, TOS 0x0, metric 0, fwd addr 192.168.48.1, tag 0.0.0.0

-   Adv Rtr is changed to be ABR's address
-   Forwarding address is still ASBR's address

### Type 9: Graceful Restart LSA

-   Opaque LSA
-   Link-local scope
-   Link-State ID: 8 bits opaque type and 24 bits opaque ID.
-   Details see [Graceful Restart](/#Graceful_Restart "wikilink")

### Type 10: MPLS Traffic Engineering LSA

-   Opaque LSA
-   Area-local scope
-   Link-State ID: 8 bits opaque type (constant 1) and 8 bits reserved
    and 16 bits Instance. The Instance field is used by the local router
    to support multiple, separate LSAs. By default, the JUNOS software
    generates one LSA for the router itself as well as a separate LSA
    for each operational interface. From
    [1](http://www.ietf.org/rfc/rfc3630.txt),

`  The Instance field is an arbitrary value used to maintain multiple`
`  Traffic Engineering LSAs.  A maximum of 16777216 Traffic Engineering`
`  LSAs may be sourced by a single system.  The LSA ID has no`
`  topological significance.`

-   An LSA contains one top-level TLV, either a router address TLV, or a
    link TLV. And link TLV includes multiple sub-TLVs, which are defined
    as:
    -   Type 1 - Link type (1 octet). V=1: Point-to-point, V=2:
        Multi-access
    -   Type 2 - Link ID (4 octets)
    -   Type 3 - Local interface IP address (4 octets)
    -   Type 4 - Remote interface IP address (4 octets)
    -   Type 5 - Traffic engineering metric (4 octets)
    -   Type 6 - Maximum bandwidth (4 octets)
    -   Type 7 - Maximum reservable bandwidth (4 octets)
    -   Type 8 - Unreserved bandwidth (32 octets). Starting from
        priority 1 to priority 7. Each priority takes 4 octets, in unit
        of bytes/sec.
    -   Type 9 - Administrative group (4 octets)

<!-- -->

    Dec 14 20:30:53.400416   id 1.0.3.128, type OpaqArea (0xa), age 0x2de
    Dec 14 20:30:53.400440   options 0x0
    Dec 14 20:30:53.400466   adv rtr 2.1.1.227, seq 0x80000069, cksum 0xbe38, len 116
    Dec 14 20:30:53.400490     Area-opaque TE LSA
    Dec 14 20:30:53.400517     Link (2), length 92:
    Dec 14 20:30:53.400546       Linktype (1), length 1:  2
    Dec 14 20:30:53.400575       LinkID (2), length 4:  3.2.193.1
    Dec 14 20:30:53.400605       LocIfAdr (3), length 4:  3.2.193.1
    Dec 14 20:30:53.400633       TEMetric (5), length 4:  1
    Dec 14 20:30:53.400664       MaxBW (6), length 4:  100Mbps
    Dec 14 20:30:53.400695       MaxRsvBW (7), length 4:  0bps
    Dec 14 20:30:53.400768       UnRsvBW (8), length 32:
                                   Priority 0, 100Mbps
                                   Priority 1, 100Mbps
                                   Priority 2, 100Mbps
                                   Priority 3, 100Mbps
                                   Priority 4, 100Mbps
                                   Priority 5, 100Mbps
                                   Priority 6, 100Mbps
                                   Priority 7, 100Mbps
    Dec 14 20:30:53.400898       Color (9), length 4:  0

The Link-State Database
-----------------------

### Database Integrity

-   all routers in the same area should have exact same OSPF database.
    (So Feiyi is right in theory and practice, for same area.)

` show ospf database`

### The Shortest Path First Algorithm

In essence, it is [Dijkstra's
algorithm](/:w:Dijkstra's_algorithm "wikilink") (see
[animation](http://www.cs.sunysb.edu/~skiena/combinatorica/animations/dijkstra.html)).
We only need get inputs right. In implementation, there are three
conceptual database:

-   Link-state database: contains inputs (router ID, neighbor ID, cost)
-   Cadidate database: intermediate results
-   Tree database: final results - SPF tree.

SPF run is expensive in terms of CPU cycles. To avoid SPF takes too much
CPU time, two timers are implemented.

-   After **Three** consecutive rapid SPF runs, a mandtaory (**5
    second**) hold down timer is fired.
-   A **200 millisecond** delay is pre-configured between back-to-back
    SPF runs.
    -   can be altered by *spf-delay* knob (range from 50 ms to 1 sec.
        Prior to 6.x, default value is 1 second).
    -   best practive is this value is slightly larger than worst
        propagation delay in the network.

Given above, SPF runs is shaped to (200ms 200ms 200ms 5s) in time.

Configuration Options
---------------------

### Interface Metrics

-   By default, fast ethernet (100Mbps) has a metric 1, any higher BW
    will be less than 1. Formula is 100M/BW, more general form is
    reference-bandwidth/BW.

`show ospf interface detail`

-   Can alter an interface metric manually

<!-- -->

    [edit protocols ospf]
    area 0.0.0.1 {
      interface fe-0/0/0.0 {
        metric 15;
      }
    }

-   or change *reference-bandwidth*

<!-- -->

    [edit protocols ospf]
    reference-bandwidth 1g;

### Authentication

**Three modes**

-   none (default)
-   simple: plain text shared password is transmitted in OSPF packets
-   MD5: md5(plain text shared password) is transmitted in OSPF packets

**Configuration**

-   under protocol ospf, config type (routers in same area have to use
    same type)
-   under OSPF interface, config key

<!-- -->

    [edit protocols ospf]
    user@Chablis# show
      area 0.0.0.2 {
      authentication-type simple; # SECRET-DATA
      interface so-0/1/0.0 {
        authentication-key "$9$9SwLCORrlMXNbvWaZ"; # SECRET-DATA
      }
    }

### Graceful Restart

-   Some network changes are transient, i.e., they will return back to
    its original states in very short time. We are trying to avoid
    unnecessary SPF for those transient changes.
-   Requires **all** neighbors support, i.e., it is either all or zero
    situation. If one neighbor doesn't support this, it will break the
    whole thing: declares adjacency down and propagate LSAs. Those LSAs
    will be accepted by other routers and triggers the whole network to
    converge.

**Three modes**

-   Restart Candidate: the router which knows it will restart (via cli
    *restart routing*). It will do:
    -   store current forwarding table/adjacency/config in memory;
    -   ask **each** neighbors for assistance via sending a grace LSA
        (type 9);
    -   restart
-   Possible Helper: default mode for all restart-capable routers
-   Helper: when a router receives a notification, it will transit to
    Helper mode and do:
    -   maintain adjacency until certain time passed
    -   after being notified the restarting router is back, it sends all
        its database back to restarting router.

**Grace LSA**

-   Link-State ID: 8b type=3 + 24b ID=0
-   3 TLVs
    -   T=1 (grace period), L=4B, V=X seconds. Within X seconds,
        neighbor should not declare adjacency down.
    -   T=2 (restart reason), L=1B, V=unknown (0), software restart (1),
        software upgrade or reload (2), or a switch to a redundant
        control processor (3).
    -   T=3 (IP address), L=4B, V=restarting router interface IP when it
        connects to a shared segment (LAN).

<!-- -->

    Type    ID      Adv Rtr      Seq        Age Opt Cksum  Len
    OpaqLoc 3.0.0.0 192.168.32.2 0x80000001 46  0x2 0xf16a 36
    Grace 90
    Reason 1

**Configuration**

-   *restart-duration*: from restarting event to expected adjacency
    re-establishment, default is 60 seconds.
-   *notify-duration*: from end of restart-duration to declare negihbor
    is down, default is 30 seconds.

### Virtual Links

-   used for an area not directly conntecting to area 0.
    -   Regular network summary will have area ID 0 because they are
        assumingly advertised into area 0.
    -   If one network summary have area ID not 0, this summary will not
        be used by other ABR's SPF calculation, nor to be
        re-summarized/populated to avoid loop.

On one backbone router and one ABR connecting two non-backbone areas,
setup a virutal link to "bring" that ABR into area 0 as if it were
directly connected to backbone.

    user@Cabernet# show  <<< backbone router
    area 0.0.0.0 {
      virtual-link neighbor-id 192.168.16.2 transit-area 0.0.0.1;

    user@Riesling# show <<< ABR connecting two non-backbone areas
    area 0.0.0.4 {
      interface fe-0/0/0.0;
    }
    area 0.0.0.1 {
      interface fe-0/0/1.0;
    }
    area 0.0.0.0 {
      virtual-link neighbor-id 192.168.0.1 transit-area 0.0.0.1;
    }

Address Summarization
---------------------

-   Stub/NSSA is to reduce the routing table for none-backbone routers,
    while address summarization is to reduce the routing table for
    backbone routers.
-   done by ABR.

### Area Route Summarization

-   ABR is configured with *area-range*.

<!-- -->

    area 0.0.0.2 {
      area-range 192.168.32.0/20;

-   ABR has all routing information

<!-- -->

    user@Chardonnay> show route protocol ospf 192.168.32/20
    inet.0: 31 destinations, 32 routes (31 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both
    192.168.32.0/20 *[OSPF/10] 00:05:21, metric 16777215
                       Discard  <<< to ensure no one can reach this unless it's to one of the following
    192.168.32.1/32 *[OSPF/10] 00:05:21, metric 6
                       > via at-0/1/0.0
    192.168.32.2/32 *[OSPF/10] 00:05:21, metric 7
                       > via at-0/1/0.0
    192.168.33.0/24 [OSPF/10] 00:05:21, metric 6
                       > via at-0/1/0.0
    192.168.34.0/24 *[OSPF/10] 00:05:21, metric 7
                       > via at-0/1/0.0

-   ABR sends a network summary LSA to area 0, while the metric for
    summarized route is the highest metric of contributing routes.

<!-- -->

    Type    ID            Adv Rtr     Seq        Age Opt Cksum  Len
    Summary *192.168.32.0 192.168.0.2 0x80000001 276 0x2 0x3b35 28
    mask 255.255.240.0
    TOS 0x0, metric 7

### NSSA Route Summarization

Same configuration to summarize NSSA external routes.

    area 0.0.0.3 {
    nssa {
      default-lsa {
        default-metric 25;
        type-7;
      }
      no-summaries;
      area-range 172.16.4.0/22; <<< placing here is to summarize external routes
    }
    area-range 192.168.48.0/20; <<< placing here is to summarize intra-area routes

Summary
-------

In this chapter, we took a very detailed look at the operation of OSPF.
We discussed each of the link-state advertisement types, including
packet formats, and showed an example of their use in a sample network.
We then explored the shortest path first (SPF) algorithm and how it
calculates the path to each destination in the network. We performed an
example of the calculation on a small network sample.

We then examined the configuration options within the protocol. We
looked at the router’s ability to use graceful restart to avoid network
outages. We then discussed authentication in the network and altering
the metric values advertised in the router LSAs. Finally, we described
the uses of virtual links and saw an example of their configuration.

We concluded the chapter with configuration examples of a stub and a
not-so-stubby area. We explored the effect of these area types on the
link-state database as well as methods for maintaining reachability in
the network. Finally, we looked at summarizing internal area and NSSA
routes on the ABRs before advertising those routes into the OSPF
backbone.

Check List
----------

-   tell format and function of each type of LSA
-   preferences for OSPF routes
    -   intra-area routes learned within its own area &gt; learned from
        other area &gt; type 1 extern routes &gt; type 2 extern routes
-   three data structures used by SPF calculation
-   use and config of a virtual link
-   understand stub and NSSA area
-   config area summarization
