---
title: JNCIS Notes ISIS
permalink: /JNCIS_Notes/ISIS/
---

- [IS-IS General](#is-is-general)
  - [Addressing](#addressing)
  - [IS-IS PDU](#is-is-pdu)
- [IS-IS TLVs](#is-is-tlvs)
  - [TLVs used to setup/maintain neighboring relationship](#tlvs-used-to-setupmaintain-neighboring-relationship)
    - [Area Address (TLV 1)](#area-address-tlv-1)
    - [IS Neighbors (TLV 6)](#is-neighbors-tlv-6)
    - [Padding (TLV 8)](#padding-tlv-8)
    - [Authentication (TLV 10)](#authentication-tlv-10)
    - [Checksum (TLV 12)](#checksum-tlv-12)
    - [Protocols Supported (TLV 129)](#protocols-supported-tlv-129)
    - [IP Interface Address (TLV 132)](#ip-interface-address-tlv-132)
    - [Graceful Restart (TLV 211)](#graceful-restart-tlv-211)
    - [Point-to-Point Adjacency State (TLV 240)](#point-to-point-adjacency-state-tlv-240)
  - [TLVs used to populate Link-State Database and SPF calculation](#tlvs-used-to-populate-link-state-database-and-spf-calculation)
    - [IS Reachability (TLV 2)](#is-reachability-tlv-2)
    - [Extended IS Reachability (TLV 22)](#extended-is-reachability-tlv-22)
    - [IP Internal Reachability (TLV 128)](#ip-internal-reachability-tlv-128)
    - [IP External Reachability (TLV 130)](#ip-external-reachability-tlv-130)
    - [Extended IP Reachability (TLV 135)](#extended-ip-reachability-tlv-135)
    - [Traffic Engineering IP Router ID (TLV 134)](#traffic-engineering-ip-router-id-tlv-134)
    - [Dynamic Host Name (TLV 137)](#dynamic-host-name-tlv-137)
  - [TLVs used for Link-State Database Maintenance](#tlvs-used-for-link-state-database-maintenance)
    - [LSP Entry (TLV 9)](#lsp-entry-tlv-9)
- [IS-IS Areas and Levels](#is-is-areas-and-levels)
  - [Propagation of Routing Information (LSPs)](#propagation-of-routing-information-lsps)
  - [Route Leakage](#route-leakage)
- [Link-State Database](#link-state-database)
  - [Database Integrity](#database-integrity)
- [Configuration Options](#configuration-options)
  - [Interface Metrics](#interface-metrics)
  - [Wide Metrics](#wide-metrics)
  - [Mesh Groups](#mesh-groups)
  - [Graceful Restart](#graceful-restart)
  - [Authentication](#authentication)
  - [Overload Bit](#overload-bit)
- [Address Summarization](#address-summarization)
- [Summary](#summary)
  - [Check List](#check-list)

## IS-IS General

* http://www.networksorcery.com/enp/protocol/is-is.htm

### Addressing

ISIS uses ISO address, also called *Network Entity Title*.

Area: 1-13 bytes. e.g., 49.0001
System ID: 6 bytes. often encoding lo0's IP address, e.g., 1921.6800.1002 = 192.168.001.002
Selector: 1 byte. designed as similar to TCP port. JUNOS uses it to identify different links/circuits, called as *circuit ID*. Starting with 0.

### IS-IS PDU

-   Three type ISIS PDUs
    - http://www.rhyshaden.com/isis.htm
    - http://www.cisco.com/en/US/products/ps6599/products_white_paper09186a00800a3e6f.shtml
-   Each type can be level 1 and level 2.

```
PDU	Type
Hello	 
Level 1 LAN	    15
Level 2 LAN	    16
Point-to-point	    17

Link State	 
Level 1 LSP	    18
Level 2 LSP	    20

Sequence Numbers	 
Level 1 CSNP	  24
Level 2 CSNP	  25
Level 1 PSNP	  26
Level 2 PSNP	  27
```

-   **IIH**: IS-IS Hello packet. Used for setup/maintain adjacency.
    -   LAN IIH: used in broadcast segment
    -   point-to-point IIH: single type used for both levels.
-   **LSP**: Link State Packet. Core data structure forms inputs of SPF algorithm.
    -   SNP: Sequence Number Packet. Used to maintain link-state database
    -   Complete SNP (**CSNP**): contains headers of all LSPs in the database.
    -   Partial SNP (**PSNP**): contains headers of some LSPs in the
        database.
-   PDU is carrying TLVs, which contains routing information.

## IS-IS TLVs
-   T: 1 octet; L: 1 octet
-   TLV code points: http://www.iana.org/assignments/isis-tlv-codepoints

```
    Name                                  Value  IIH   LSP  SNP   Status/Reference
    ----------------------                -----  ---   ---  ---   ----------------
    Area Addresses                            1   y     y    n    ISO 10589
    IS Reach                                  2   n     y    n    ISO 10589
    IS Neighbor                               6   y     n    n    ISO 10589
    Padding                                   8   y     n    n    ISO 10589
    LSP Entries                               9   n     n    y    ISO 10589
    Authentication                           10   y     y    y    ISO 10589
    Opt. Checksum                            12   y     n    y    [RFC3358]
    The extended IS reachability TLV         22   n     y    n    [RFC3784]
    IP Int. Reach                           128   n     y    n    [RFC1195]
    Prot. Supported                         129   y     y    n    [RFC1195]
    IP Ext. Address                         130   n     y    n    [RFC1195]
    IP Intf. Address                        132   y     y    n    [RFC1195]
    The Traffic Engineering router ID TLV   134   n     y    n    [RFC3784]
    The extended IP reachability TLV        135   n     y    n    [RFC3784]
    Dynamic Name                            137   n     y    n    [RFC2763]
    Restart TLV                             211   y     n    n    [RFC3847]
    Multiple Topologies IS Reach            222   n     y    n    IETF-draft
    Multiple Topologies (routing instance)  229   y     y    n    IETF-draft
    IPv6 Intf. Addr.                        232   y     y    n    IETF-draft
    Multiple Topologies IP. Reach           235   n     y    n    IETF-draft
    IPv6 IP. Reach                          236   n     y    n    IETF-draft
    Multiple Topologies IPv6 IP. Reach      237   n     y    n    IETF-draft
    P2P 3-Way Adj. State                    240   y     n    n    [RFC3373]
```

-   An example:

```
    user@Shiraz> show isis database extensive | find TLV
    TLVs:
      Area address: 49.0001 (3)                                          <-- TLV 1   = Area address
      Speaks: IP                                                         <-- TLV 129 = Protocols supported
      Speaks: IPv6                                                       <-- TLV 129 = ..
      IP router id: 192.168.3.3                                          <-- TLV 134 = TE IP router ID
      IP address: 192.168.3.3                                            <-- TLV 132 = IP interface address
      Hostname: Shiraz                                                   <-- TLV 137 = Dynamic hostname resolution
      IS neighbor: Merlot.00, Internal, Metric: default 10               <-- TLV 2   = IS reachability
      IS extended neighbor: Merlot.00, Metric: default 10                <-- TLV 22  = Extended IP reachability
        IP address: 192.168.20.2                                         <-- TLV 22 subTLV 6
        Neighbor's IP address: 192.168.20.1                              <-- TLV 22 subTLV 8
      IP prefix: 192.168.3.3/32, Internal, Metric: default 0, Up         <-- TLV 128 = IP internal reachability
      IP prefix: 192.168.20.0/24, Internal, Metric: default 10, Up       <-- TLV 128 = ..
      IP extended prefix: 192.168.3.3/32 metric 0 up                     <-- TLV 135 = Extended IP reachability
      IP extended prefix: 192.168.20.0/24 metric 10 up                   <-- TLV 135 = ..
      IP external prefix: 172.16.3.0/24, Internal, Metric: default 0, Up <-- TLV 130 = IP external reachability
      Authentication data: 17 bytes                                      <-- TLV 10  = Authentication
```

### TLVs used to setup/maintain neighboring relationship

-   Carried in IS-IS IIH PDUs.

#### Area Address (TLV 1)

-   reporting which area(s) the local router is in. Max area number is 3. Including up to 3 (area length + area ID)
-   V=
    -   (1-octet) area length
    -   (1-13 octets) area ID

`Area address(es) TLV #1, length: 4`
`  Area address (length: 3): 49.0001`

#### IS Neighbors (TLV 6)

-   For broadcast segement, ie, LAN. Reporting MAC address of remote neighbor (6 octet)
-   V=
    -   (6-octets) Neighbor MAC address

`IS Neighbor(s) TLV #6, length: 6`
`  IS Neighbor: 0090.6967.4401`

#### Padding (TLV 8)

-   Padding is used to find out which MTU the neighbor is supporting. First padding the Hello packets to max MTU, then decreasing step by step until adjacency established. ISIS interface has to support 1492 MTU as a requirement.
-   V=
    -   (1-255 octets) 0x00

`Padding TLV #8, length: 255`
`Padding TLV #8, length: 255`

#### Authentication (TLV 10)

-   plain text or MD5(plain text)
-   V=
    -   (1-octet) Authentication Type. 1: plain text password; 54: MD5 is used.
    -   (variable) password.

`Authentication TLV #10, length: 21`
`  simple text password: this-is-the-password`

#### Checksum (TLV 12)

-   ISIS interface can be configured to do checksum against PDU/TLVs.
-   cannot combine with authentication since authentication is performed last which empty checksum field.
-   V=
    -   (2-octets) checksum.

`Checksum TLV #12, length: 2`
`  checksum: 0x7eb5 (correct)`

#### Protocols Supported (TLV 129)

-   which language (ip/ipv6) local system can speak
-   V=
    -   (1-octet) Network layer protocol ID. 0xCC: IPv4; 0x8E: IPv6.
    -   (1-octet) Network layer protocol ID. (junos support upto 2 protocols so far.)

#### IP Interface Address (TLV 132)

-   by default, this is lo0's IP address. In theory, this can be all IP address configured on the router.
-   V=
    -   (4-octets) IP address

#### Graceful Restart (TLV 211)

-   Used to report current state and communicate on restarting event (start/done)
-   V=
    -   (1-octet) Flag. 0: restart request; 1: restart ack.
    -   (2-octets) Remaining time: countdown time during which the restart should be completed.

`Restart Signaling TLV #211, length: 3`
`  Restart Request bit clear, Restart Acknowledgement bit clear`
`  Remaining holding time: 0s`

This above output says "I am steady".

#### Point-to-Point Adjacency State (TLV 240)

-   For point-to-point links, this TLV is used to report adjacency state. Only in IIH packets.
-   V=
    -   (1-octet) Ajacency State. 0: Up; 1: Initializing; 2: Down.
    -   (4-octets) Extended Local Circuit ID. JUNOS uses interface index (ifIndex) for this field.
    -   JUNOS set point-to-point interface's circuit ID as 0x01, but not useful to identify circuits, that's why using extended circuit ID.
    -   (6-octets) Neighbor System ID.
    -   (4-octets) Neighbor Extended Local Circuit ID.

`Point-to-point Adjacency State TLV #240, length: 15`
`  Adjacency State: Up`
`  Extended Local circuit ID: 0x00000042`
`  Neighbor SystemID: 1921.6800.2002`
`  Neighbor Extended Local circuit ID: 0x00000043`

### TLVs used to populate Link-State Database and SPF calculation

-   Carried in IS-IS LSP PDUs.
-   TLVs contains network topology information, i.e., links, subnets attached to nodes, etc. These TLVs will be flooded into the whole network.
-   Link information <u>(link=&lt;node,node&gt;,cost)</u> to SPF algorithm within the same level.
    -   IS Reachability (TLV 2)
    -   Extended IS Reachability (TLV 22)
-   Subnets information <u>(node,networks)</u> on reaching networks connecting to the nodes. If we can reach a node, we can also reach networks attached to the node.
    -   IP Internal Reachability (TLV 128)
    -   IP External Reachability (TLV 130)
    -   IP Internal Reachability (TLV 135)
-   Also, some miscellaneose TLVs
    -   Traffic Engineering IP Router ID (TLV 134)
    -   Dynamic Host Name (TLV 137)
-   And some common TLVs
    -   Area Address (TLV 1)
    -   Authentication (TLV 10)
    -   Protocols Supported (TLV 129)
    -   IP Interface Address (TLV 132)

#### IS Reachability (TLV 2)

-   V=
    -   (1-octet) virtual flag (const 0 for JUNOS)
    -   (1-octet) R (Reserved) bit, I/E bit, default metric. (I/E: Internal/External metric; metric ranges only from 0 to 63)
    -   (1-octet) S (Supported) bit, I/E bit, delay metric. (S=1: JUNOS does not support this. Rest bits are 0)
    -   (1-octet) S (Supported) bit, I/E bit, expense metric. (S=1: JUNOS does not support this. Rest bits are 0)
    -   (1-octet) S (Supported) bit, I/E bit, error metric. (S=1: JUNOS does not support this. Rest bits are 0)
    -   (7-octets) Neighbor ID

`IS Reachability TLV #2, length: 12`
`  IsNotVirtual`
`  IS Neighbor: 1921.6800.2002.00, Default Metric: 10, Internal`

-   internal vs external metric

#### Extended IS Reachability (TLV 22)

-   "wide" metric version of TLV 2, **3 octets**.
-   supports TE by using subTLVs, such as max link bandwidth, TE metrics, etc.
-   V=
    -   (7-octets) System ID + circuite ID/Selector
    -   (3-octets) Wide metric.
    -   (1-octet) sub-TLV length
    -   (variable) sub-TLVs

`IS extended neighbor: Shiraz.00, Metric: default 10`
`  IP address: 192.168.20.1`
`  Neighbor's IP address: 192.168.20.2`

`IS extended neighbor: Chianti.02, Metric: default 10`
`  IP address: 10.222.29.1`
`  Traffic engineering metric: 30            --+`
`  Current reservable bandwidth:               |`
`    Priority 0 : 1000Mbps                     |`
`    Priority 1 : 1000Mbps                     |`
`    Priority 2 : 1000Mbps                     |`
`    Priority 3 : 1000Mbps                     +--> TE Sub-TLVs`
`    Priority 4 : 1000Mbps                     |`
`    Priority 5 : 1000Mbps                     |`
`    Priority 6 : 1000Mbps                     |`
`    Priority 7 : 1000Mbps                     |`
`    Maximum reservable bandwidth: 1000Mbps    |`
`    Maximum bandwidth: 1000Mbps               |`
`  Administrative groups: 0 `<none>`           --+`

#### IP Internal Reachability (TLV 128)

-   By default, lo0 will be included into this TLV.
-   By default, level 1 internal routes will be advertised into level 2.
-   6 bits metric
-   V=
    -   (1-octet) U/D bit, I/E bit, default metric.
        -   1 U/D bit (up/down). 0: route be able to advertise to higher level; 1: not
        -   1 I/E bit (internal/external). 0: internal; 1: external
    -   (1-octet) S (Supported) bit, R (Reserved) bit, delay metric. (S=1: JUNOS does not support this. Rest bits are 0)
    -   (1-octet) S (Supported) bit, R (Reserved) bit, expense metric. (S=1: JUNOS does not support this. Rest bits are 0)
    -   (1-octet) S (Supported) bit, R (Reserved) bit, error metric. (S=1: JUNOS does not support this. Rest bits are 0)
    -   (4-octets) IP address
    -   (4-octets) Subnet mask

`IP prefix: 192.168.10.0/24, Internal, Metric: default 10, Up`

-   route is marked as *Internal*, i.e., I/E bit=0; and *Up* means the route can go to higher level, i.e., U/D=0.

#### IP External Reachability (TLV 130)

-   This TLV indicates the route is not native to ISIS, but learned from somewhere else.
-   **Note** that by default, level 1 external IP routes will *not* be advertised to level 2. However, by configuring "wide-metrics-only", external IP routes will be carried by TLV 135 and we will not differentiated internal and external any more. This way external IP routes will be advertised into level 2.
-   V=(same as TLV 128)

#### Extended IP Reachability (TLV 135)

-   "wide" metrics version of TVL 128 and 130. **4 octets**.
-   not differentiate internal or external routes anymore.
-   subTLVs are available for TE.
-   V=
    -   (4-octets) Wide metric
    -   (1-octet)
        -   1 U/D bit
        -   1 sub bit. 0: no subTLV present; 1: subTLV present
        -   6 bit prefix length (/0 to /64)
    -   (variable) Prefix
    -   (1-octet) Optional sub-TLV type
    -   (1-octet) Optional sub-TLV length
    -   (variable) Optional sub-TLV value

#### Traffic Engineering IP Router ID (TLV 134)

-   This router ID will be placed in other routers' both link-state database and TED.
-   V=
    -   (4-octets) Router ID

#### Dynamic Host Name (TLV 137)

-   mainly tell other router to use this name in 'show' command instead of use system ID.
-   V=
    -   (variable) Hostname

### TLVs used for Link-State Database Maintenance

-   Carried in IS-IS CSNP/PSNP PDUs. Summary information on local Link-State Database.
-   Contains three TLVs.
    -   LSP Entry (TLV 9)
    -   Authentication (TLV 10)
    -   Checksum (TLV 12)

#### LSP Entry (TLV 9)

-   contains summary information of local entries in the link-state database.
-   V=
    -   (2-octets) Remaining lifetime in seconds. By default, JUNOS
        assigns each LSP 1200s lifetime. After lifetime, LSP is
        considered dead.
    -   (8-octets) LSP ID = (system ID + circuit ID/selector + LSP
        number)
    -   (4-octets) Sequence Number. From 0x00000001 to 0xffffffff,
        incrementing by 1 each time originating router updates the LSP.
    -   (2-octets) Checksum of (LSP ID, Seq. Number).

`LSP entries TLV #9, length: 64`
`  lsp-id: 1921.6800.1001.00-00, seq: 0x000000d7,`
`    lifetime: 968s, chksum: 0x0960`
`  lsp-id: 1921.6800.2002.00-00, seq: 0x000000d8,`
`    lifetime: 1192s, chksum: 0x7961`

## IS-IS Areas and Levels

-   Area is identified by area ID in ISIS address. Area controls neighborhood formation.
    -   L1 router only neighbors with L1 routers configured with same area ID.
    -   L2 router neighbors with any routers.
-   Level is configured via level-1 or level-2 under protocol ISIS interfaces. Level controls flooding scope.
    -   L1 LSP is only flooded in its own L1 area.
    -   By default, L2 LSP is only flooded in its own L2 area.

### Propagation of Routing Information (LSPs)

-   Each router has its own LSP.

```
    user@Shiraz> show isis database Shiraz.00-00 detail
    IS-IS level 1 link-state database:

    Shiraz.00-00 Sequence: 0x56, Checksum: 0x52a2, Lifetime: 780 secs
      IS neighbor: Merlot.00 Metric: 10
      IS neighbor: Sangiovese.00 Metric: 10
      IP prefix: 192.168.16.1/32 Metric: 0 Internal Up   <<< Shiraz's lo0 address in its own routing table
      IP prefix: 192.168.17.0/24 Metric: 10 Internal Up
      IP prefix: 192.168.18.0/24 Metric: 10 Internal Up
```

-   The LSP-ID, Shiraz.00-00, can be broken down into three sections:
    -   Shiraz = system ID
    -   00 = non-zero value for the pseudonode.
    -   00 = fragment number. 
-   In this case, we only have fragment numbers of 00, which indicates that all the data fit into this LSP fragment, and there was no need to create more fragments. If there had been information that did not fit into the first LSP, IS-IS would have created more LSP fragments, such as 01, 02, and so on.
-   A Level 1 LSP is flooded within its own Level 1 area. Every router in same area does SPF calculation to reach every other router and every prefix attached to other routers.
-   Each L1/L2 border router announces its local Level 1 routes in its Level 2 LSP to the backbone. This allows all Level 2 routers to have explicit routing knowledge of all routes in the network.

```
    user@Merlot> show isis database Shiraz.00-00 detail
    IS-IS level 1 link-state database:

    Shiraz.00-00 Sequence: 0x57, Checksum: 0x50a3, Lifetime: 748 secs
      IS neighbor: Merlot.00 Metric: 10
      IS neighbor: Sangiovese.00 Metric: 10
      IP prefix: 192.168.16.1/32 Metric: 0 Internal Up  <<< Shiraz's lo0 address in Merlot's L1 database.
      IP prefix: 192.168.17.0/24 Metric: 10 Internal Up
      IP prefix: 192.168.18.0/24 Metric: 10 Internal Up
```

-   IS-IS level 2 link-state database:

```
    Merlot.00-00 Sequence: 0x59, Checksum: 0x111c, Lifetime: 829 secs
      IS neighbor: Merlot.02 Metric: 10
      IP prefix: 192.168.0.1/32 Metric: 0 Internal Up
      IP prefix: 192.168.0.2/32 Metric: 20 Internal Up
      IP prefix: 192.168.1.0/24 Metric: 10 Internal Up
      IP prefix: 192.168.16.1/32 Metric: 10 Internal Up  <<< Shiraz's lo0 address moves to Merlot's L2 database,
                                                             which will be flooded into L2 area.
      IP prefix: 192.168.17.0/24 Metric: 10 Internal Up
      IP prefix: 192.168.18.0/24 Metric: 20 Internal Up
```

-   Each Level 1 router uses L1/L2 border router as default router. L1/L2 is identified by Attached bit = 1. If multiple L1/L2 border router exist, level 1 router chose the closet one in terms of metric.

```
    user@Riesling> show isis database level 1
    IS-IS level 1 link-state database:
    LSP ID         Sequence Checksum Lifetime Attributes
    Riesling.00-00 0x55     0xb1f1   609      L1 L2 Attached <<< Attached bit is set.
    Cabernet.00-00 0x56     0xda17   809      L1 L2 Attached
```

-   Level 1 *external* routes will not be advertised into level 2 area. Using *wide-metrics-only* is a workaround since wide metrcs is carried by TLV 135, which doesn't differentiate *internal* routes and *external* routes. Or use policy similar to Route leakage below but in the opposite direction.

### Route Leakage

-   By default, level 2 routes are not injected into level 1 area.
-   To change the default behavior, using policies:

```
    term level-2-routes {
      from {
        protocol isis;
        level 2;
      }
      to level 1;
      then accept;
    }
```

## Link-State Database

### Database Integrity

-   The advertised LSPs in each level must be identical on each router.

```
    JTAC@LONCR4.re0> show isis database level 2 BAGPE2.00-00 extensive
    IS-IS level 2 link-state database:

      Packet: LSP ID: BAGPE2.00-00, Length: 95 bytes, Lifetime : 65522 secs
        Checksum: 0xb7da, Sequence: 0x692, Attributes: 0x3 <L1 L2>
        NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
        Packet type: 20, Packet version: 1, Max area: 0

      TLVs:
        Area address: 47.0023.0001.0000.0000.0003.0001 (13)
        Speaks: IP
        Hostname: BAGPE2
        IP address: 172.25.0.1
        IP extended prefix: 172.25.1.176/30 metric 3 up
        IS extended neighbor: BAGPE1.00, Metric: default 3
        IP extended prefix: 172.25.0.1/32 metric 0 up
      No queued transmissions

    JTAC@ATLCR1.re0> show isis database level 2 BAGPE2.00-00 extensive
    IS-IS level 2 link-state database:

      Packet: LSP ID: BAGPE2.00-00, Length: 95 bytes, Lifetime : 65524 secs
        Checksum: 0xb7da, Sequence: 0x692, Attributes: 0x3 <L1 L2>
        NLPID: 0x83, Fixed length: 27 bytes, Version: 1, Sysid length: 0 bytes
        Packet type: 20, Packet version: 1, Max area: 0

      TLVs:
        Area address: 47.0023.0001.0000.0000.0003.0001 (13)
        Speaks: IP
        Hostname: BAGPE2
        IP address: 172.25.0.1
        IP extended prefix: 172.25.1.176/30 metric 3 up
        IS extended neighbor: BAGPE1.00, Metric: default 3
        IP extended prefix: 172.25.0.1/32 metric 0 up
      No queued transmissions
```

## Configuration Options

### Interface Metrics

-   change interface metric under `protocol isis`

```
    interface so-0/1/0.0 {
      level 1 metric 25;
      level 2 metric 30;
    }
```

-   change reference bandwidth. Default is fast ethernet (100M) is metric 1. Formula is (reference_bandwidth/link BW), round to 1 is less than 1.

```
    user@Shiraz> show configuration protocols isis
    reference-bandwidth 1g;
```

### Wide Metrics

-   By default, small metrics and wide metrics are both advertised. But wide metrics is ignored.
-   *wide-metrics-only* knob makes advertising wide metrics only.

### Mesh Groups

-   By default, flooding works in a way that re-transit LSP to all adjacent neighbors but the sender.
-   Mesh group is configured at interface level. It specifies that LSPs received on an interface are not reflooded to any other interface on the router, which is also configured with the same mesh group value.

```
    user@Chianti> show configuration protocols isis
    level 1 disable;
    interface so-0/1/0.600 {
      mesh-group 101;
    }
    interface so-0/1/1.600 {
      mesh-group 101;
    }
    interface so-0/1/1.700 {
      mesh-group 101;
    }
```

### Graceful Restart

-   To avoid overhead when the IS-IS restart time is of a short duration. 
-   Operation is similar to OSPF graceful restart.
-   alerts neighbor restart event

```
        Restart Signaling TLV #211, length: 3
        Restart Request bit set, Restart Acknowledgement bit clear
        Remaining holding time: 0s
```

-   if neighbor is capable to help (aka in helper mode), neighbor acknowledges the request and maintains UP state during restart.

```
        Restart Signaling TLV #211, length: 3
        Restart Request bit clear, Restart Acknowledgement bit set
        Remaining holding time: 90s
```

-   if successfully restarted, ask neighbor's assistance to catch up all routing changes. Neighbor generates CSNPs to the restarting neighbor and responds to any received PSNPs with the appropriate LSP.
-   if restart takes longer than configured time (90 seconds by default
    but configurable), neighbor declares ISIS session down.

```
        [edit protocols isis]
        user@Riesling# set graceful-restart ?
          restart-duration  Maximum time for graceful restart to finish (seconds) <<< 90 sec is the default value
```

### Authentication

-   configured at `protocol isis` level or `protocol isis interface` level
-   simple

` authentication-key "$9$ewMKLNdVYoZjwYF/tOcSwYg4aU"; # SECRET-DATA`
` authentication-type simple; # SECRET-DATA`

-   md5

` authentication-key "$9$ewMKLNdVYoZjwYF/tOcSwYg4aU"; # SECRET-DATA`
` authentication-type md5; # SECRET-DATA`

-   or authenticate HELLO PDU only. So Authentication TLV won't be in LSP and SNP.

` hello-authentication-key "$9$6gr2/u1SyK8xdevs2g4jik.PQ6A0BEK"; # SECRET-DATA`
` hello-authentication-type md5; # SECRET-DATA`

### Overload Bit

-   When a router sets this bit to the value 1, other routers in the network remove the overloaded router from the *forwarding* topology. In essence it is no longer used to forward *transit* traffic through the network.
-   But the routes local to overloaded router still can take traffic.
-   often used for maintenance or leave router alone to reload BGP table after a restart.
-   set under `protocol isis`

```
    overload [timeout 60];
```

## Address Summarization

-   done by L1/L2 router when advertising level 1 to level 2, or verse visa.
-   done with policies.
    -   first check what group of prefixes can be summarized, `show isis database`.
    -   then write a policy to reject more specific.
    -   then write an aggregate to be advertised.

```
    [edit routing-options]
    user@Merlot# show
    aggregate {
      route 192.168.16.0/20;
    }
    [edit]
    user@Merlot# show policy-options
    policy-statement sum-int-L1-to-L2 {
      term suppress-specifics {
        from {
          route-filter 192.168.16.0/20 longer;
        }
        to level 2;
        then reject;
      }
      term send-aggregate-route {
        from protocol aggregate;
        to level 2;
        then accept;
      }
    }
```

## Summary

In this chapter, we examined the operation of the IS-IS routing protocol. We first discussed the various type, length, value (TLV) formats used to advertise information. We then explored the Shortest Path First (SPF) algorithm and saw how it calculates the path to each destination in the network. We then discussed some configuration options available within the JUNOS software for use with IS-IS. We first saw how graceful restart can help mitigate churn in a network. A look at interface metrics and authentication options followed. We then explored how a mesh group operates by reducing the flooding of LSPs. This was followed by the use of the overload bit in the network.

We concluded the chapter with an exploration of how IS-IS operates in a multilevel configuration. We saw how reachability was attained for each router in the network and the default flooding rules for routes from different levels. We then discussed methods for altering the default flooding rules using routing policies to leak routes between levels. Finally, we learned how to summarize routes in an IS-IS network using locally configured aggregate routes and routing policies.

### Check List

-   list ISIS PDUs and their functions
-   list ISIS TLVs and their use
-   describe multilevel ISIS network.
-   describe how to do address summarization
