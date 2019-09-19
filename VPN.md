---
title: JNCIS Notes VPN
permalink: /JNCIS_Notes/VPN/
---

VPN Basics
----------

-   Routing table is seperate for each customer, so different customers
    can use same prefixes, such as 192.168.x.x.
    -   In order to do that, we need establish a tuple of (customer
        distinguisher/ID, customer routes, vpn label) and propagate this
        tuple to other portion of the network (e.g. via BGP).
    -   this tuple is relevant at three different places:
        -   configuration; (only first two elements in tuple need to be
            configured, vpn label is automatically assigned)
        -   bgp message format; (see
            [L3VPN](#VPN_Network_Layer_Reachability_Information))
        -   routing lookup, it corresponds to vrf table: one vrf table
            =&gt; one customer.
    -   cisco is one vpn label per (customer distinguisher/ID, customer
        routes), juniper is one vpn label per customer distinguisher/ID.
-   MPLS/LSP between PEs is a pre-request for traffic forwarding which
    uses two labels, one label (RSVP/LDP label) is used to forward to
    correct PE, the other label (vpn label) is used to find correct vrf
    table inside the PE.

Customer Edge router (CE): physically located at customer site. Connecting to PE via layer 2 (FR, ATM, Ethernet, etc).
Provider Edge router (PE): physically located at provider site. Connecting to CE via layer 2, and connecting to other PEs via MPLS LSP.
Provider router (P): doing nothing but forwarding MPLS traffic.
VPN Routing and Forwarding table (VRF): for each customer, PE has one VRF storing routing information specific to this customer.
VPN Forwarding Table (VFT): for layer 2 VPN
VPN Connection Table (VCT): for layer 2 VPN, a subset of VFT.

-   L3VPN: provider involves in customer routing domain. Customer
    advertises routing information to the provider, then the provider
    advertise the routing information to other customer's sites.
-   L2VPN: customer does routing for themselves.

Layer 3 VPNs
------------

<a href="#VPN_Network_Layer_Reachability_Information"></a>
### VPN Network Layer Reachability Information

-   this is new NLRI to represent customer routes.

<!-- -->

         8        8        8        8
    +--------+--------+--------+--------+
    |Mask    |MPLS Label                |
    +--------+--------+--------+--------+
    |Route Distinguisher                |
    +--------+--------+--------+--------+
    |Route Distinguisher (continued)    |
    +--------+--------+--------+--------+
    |IPv4 Prefix                        |
    +--------+--------+--------+--------+

Route Distinguisher: unique to each customer VPN. It makes possible that two customers can share same IP prefixes, like 192.168/16, without any confusion of BGP. It has 3 fields: Type, Administrator, Assigned Number fields.

-   Type=0: Administrator 2 octets, Assigned Number 4 octets. Type=1:
    Administrator 4 octets, Assigned Number 2 octets.
-   Administrator: 2 octets places ASN; 4 octets places PE IP.
-   Assigned Number: arbitrary, up to operator.

### Basic Operational Concepts

#### Control plane (routes advertisement)

-   PEs form a full mesh BGP connections. It is a normal BGP session but
    with VPN NLRI and extended community to differentiate VPN NLRIs.

`user@Chianti> show configuration protocols bgp`
`group Internal-Peers {`
`  type internal;`
`  local-address 192.168.20.1;`
`  family inet-vpn {`
`    unicast;`
`  }`
`  neighbor 192.168.32.1;`
`  neighbor 192.168.48.1;`
`}`

-   The Chardonnay router configures a VRF table corresponding to that
    particular customer to store customer routes. In our sample network,
    these are direct and static routes (but may also include routes
    received from a routing protocol operating between the PE and CE
    routers, later).

`user@Chardonnay> show configuration routing-instances`
`VPN-A {`
`  instance-type vrf;`
`  interface so-0/1/0.0;`
`  vrf-target target:65432:1111;`
`  routing-options {`
`    static {`
`      route 172.16.2.0/24 next-hop 10.222.44.1;`
`    }`
`  }`
`}`

:\* The instance-type command informs the router what type of VPN
service to operate for the customer

:\* customer distinguisher:

:\*\* route-distinguisher: "Each routing instance that you configure on
a PE router must have a unique route distinguisher associated with it.
VPN routing instances need a route distinguisher to help the Border
Gateway Protocol (BGP) to distinguish between potentially identical
network layer reachability information (NLRI) messages received from
different VPNs. We recommend that you use a unique route distinguisher
for each routing instance that you configure. Although you can use the
same route distinguisher on all PE routers in the same VPN, if you use a
unique route distinguisher, you can determine the PE router from which a
route originated.
[1](http://www.juniper.net/techpubs/software/junos/junos83/swconfig83-vpns/id-10110155.html#id-10110155)"

:\*\*\* route-distinguisher can be configured globally (set
routing-options route-distinguisher-id 192.168.20.1) or per vrf (?)

:\* customer routes:

:\*\* Statically configured customer routes using static routes.

:\*\* Dynamically leared customer routes via routing protocols such as
BGP, OSPF, and RIP.

:\* The interface command places the customer’s interface into the VRF
table, the interface address will be put into vrf table as part of
customer routes.

`user@Chardonnay> show route table VPN-A.inet.0`
`VPN-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`
`10.222.44.0/24 *[Direct/0] 2d 00:49:16`
`               > via so-0/1/0.0`
`10.222.44.2/32 *[Local/0] 2d 00:49:18`
`               Local via so-0/1/0.0`
`172.16.2.0/24 *[Static/5] 1d 20:00:29`
`              > to 10.222.44.1 via so-0/1/0.0`

-   Chardonnay then consults the export policy applied to this VRF table
    to determine which routes should be advertised to the other PE
    routers in the network. Two way to do so:
    -   *vrf-target* automatically advertises all active routes that
        belong to this particular VRF table.
    -   *vrf-import* and *vrf-export* can do fine granularity control
        over advertising routes.

`user@Chardonnay> show route advertising-protocol bgp 192.168.20.1 detail`
`VPN-A.inet.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)`
`* 10.222.44.0/24 (1 entry, 1 announced)`
` BGP group int type Internal`
`   Route Distinguisher: 192.168.32.1:3`
`   VPN Label: 100128`
`   Nexthop: Self`
`   Localpref: 100`
`   AS path: I`
`   `<u>`Communities: target:65432:1111`</u>

` * 172.16.2.0/24 (1 entry, 1 announced)`
`  BGP group int type Internal`
`    Route Distinguisher: 192.168.32.1:3`
`    VPN Label: 100144`
`    Nexthop: Self`
`    Localpref: 100`
`    AS path: I`
`    `<u>`Communities: target:65432:1111`</u>

NEED SOME PACKET CAPTURE HERE.

-   The receiving PE router, Chianti in our example, examines the
    incoming routes and compares them to the import policy applied to
    the VRF table. This policy matches on all routes that contain the
    route target community configured to the customer’s VRF table.

`user@Chianti> show configuration routing-instances`
`VPN-A {`
`  instance-type vrf;`
`  interface so-0/1/2.0;`
`  `<u>`vrf-target target:65432:1111;`</u>
`  ...`
`}`

-   The receiving PE router then examines the received routes to
    determine the address listed in the BGP Next Hop attribute. For the
    routes advertised by Chardonnay, this value is the local BGP peering
    address of 192.168.32.1 (Self). Chianti, the receiving router, then
    determines if the BGP Next Hop is available inet.3 routing table.
    This ensures that an MPLS LSP is established from the local router
    to the remote PE router for forwarding user traffic within the VPN.

`user@Chianti> show route table inet.3`
`inet.3: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`

`192.168.32.1/32 *[RSVP/7] 2d 23:15:46, metric 20`
`                 > via so-0/1/0.0, label-switched-path Chianti-to-Char`

-   The MPLS label advertised with the route, the VPN Label, is also
    associated with the received routes in the bgp.l3vpn.0 table. It is
    placed in a label stack along with the label used to reach the
    remote PE router:

`user@Chianti> show route table bgp.l3vpn.0 detail`
`bgp.l3vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)`

`192.168.32.1:3:172.16.2.0/24 (1 entry, 0 announced)`
`        *BGP Preference: 170/-101`
`             Route Distinguisher: 192.168.32.1:3`
`             Source: 192.168.32.1`
`             Next hop: via so-0/1/0.0 weight 1, selected`
`             Label-switched-path Chianti-to-Char`
`             `<u>`Label operation: Push 100144, Push 100448(top)`</u>
`             Protocol next hop: 192.168.32.1`
`             Push 100128`
`             Local AS: 65432 Peer AS: 65432`
`             AS path: I`
`             Communities: target:65432:1111`
`             VPN Label: 100144`
`             Localpref: 100`
`             Router ID: 192.168.32.1`
`             Secondary Tables: VPN-A.inet.0 <<< also put in vrf.`

-   Alternatively, one can use vrf-import/vrf-export for finer policy
    control through bgp community attributes. An example is below:

`VPN {`
`   instance-type vrf;`
`   interface t1-7/3/0:1:1`
`   route-distinguisher 65000:1234`
`   vrf-import set_vrf_comm_in;`
`   vrf-export set_vrf_comm_out;`
`   vrf-table-label;`
`   routing-options {`
`       static {`
`           route customer_routes next-hop customer_intf_ip`
`       }`
`       generate {`
`           route 0.0.0.0/0 { # better to have a default discard routes for safety, i.e., not`
`                             # to generate ICMP unreachable messages, which is using q2, same`
`                             # queue is used by ppp keepalives. We police q2 per vrf, but adding`
`                             # add vrf together will render policer useless and cause ppp ka lost.`
`               preference 200;`
`               discard;`
`           }`
`       }`
`   }`
`}`

`policy-statement set_vrf_comm_in {`
`   term 10 {`
`       from community cust_vpn_comm;`
`       then accept;`
`   }`
`   term 20 {`
`       then reject;`
`   }`
`}`
`policy-statement set_vrf_comm_out {`
`   from protocol direct;`
`   then {`
`       community add cust_vpn_comm;`
`       accept;`
`   }`
`}`
`community cust_vpn_comm members target:65000:1234`

#### Forwarding plane (packet forwarding)

-   The CE router of Sangiovese performs a route table lookup to locate
    a route to 172.16.2.1. It finds a locally configured static route
    for that destination with a next-hop value of 10.222.30.2. It then
    forwards the packet to Chianti.
-   Chianti receives the packet on the interface configured within the
    VRF table associated with the CE router. It performs a route table
    lookup within that VRF table to locate a route to 172.16.2.1.
-   Chianti actually performs a double push operation on the forwarded
    packet. The VPN label of 100,144 is placed on the bottom of the
    stack, and the 100,448 label is placed on the top of the stack.
-   The Merlot router, a P router, receives an MPLS packet from Chianti.
    It examines the top label value of 100,448 and performs a switching
    table lookup for that label value. The result tells Merlot to pop
    the top MPLS label and forward the remaining data to Chardonnay.
-   Chardonnay now receives an MPLS packet with a top label of 100,144,
    which it originally allocated from its label space for routes
    belonging to the VRF table. The table lookup tells Chardonnay to pop
    the top label and forward the remaining data, an IP packet in this
    case, to Shiraz:

`user@Chardonnay> show route table mpls.0 label 100144`
`mpls.0: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`

`100144 *[VPN/170] 1d 21:00:48`
`        > to 10.222.44.1 via so-0/1/0.0, Pop`

### Using BGP for PE-CE Route Advertisements

-   Once iBGP session between two PE established, prefixes inside of one
    vrf table at PE A will be automatically advertised to PE B via MBGP.
-   as-override is needed on receiving PE to make sure its connecting CE
    will accepts the route instead of thinking there is a loop.

<!-- -->

-   Sangiovese (CE1) BGP configuration

`user@Sangiovese> show configuration protocols bgp`
`group VPN-Connectivity {`
`  type external;`
`  export send-statics;`
`  peer-as 65432;`
`  neighbor 10.222.30.2;`
`}`

-   Chianti (PE1) received BGP routes from CE1

`user@Chianti> show route table VPN-A 172.16.1/24`
`VPN-A.inet.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`

`172.16.1.0/24 *[BGP/170] 00:10:08, MED 0, localpref 100`
`                 AS path: 65000 I`
`              > to 10.222.30.1 via so-0/1/2.0`

-   Chardonnay (PE2) received BGP routes from PE1

`user@Chianti> show route advertising-protocol bgp 192.168.32.1 detail`
`VPN-A.inet.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)`

`* 172.16.1.0/24 (1 entry, 1 announced)`
` BGP group int type Internal`
`     Route Distinguisher: 192.168.20.1:3`
`     VPN Label: 100416`
`     Nexthop: Self`
`     MED: 0`
`     Localpref: 100`
`     AS path: 65000 I`
`     Communities: target:65432:1111`

-   Chardonnay (PE2) BGP configuration to Shiraz (CE2)

`user@Chardonnay> show configuration routing-instances protocols bgp`
`group Shiraz-CE {`
`  type external;`
`  peer-as 65000;`
`  `**`as-override;`**
`  neighbor 10.222.44.1;`
`}`

### Using OSPF for PE-CE Route Advertisements

-   A additional policy is needed to export BGP routes learned from PE
    to OSPF domain of CE.

<!-- -->

-   Chaniti (PE1) OSPF configuration

`user@Chianti> show configuration routing-instances VPN-B`
`instance-type vrf;`
`interface ge-0/2/0.0;`
`route-distinguisher 65432:2222;`
`vrf-target target:65432:2222;`
`protocols {`
`  ospf {`
`    area 0.0.0.0 {`
`      interface ge-0/2/0.0;`
`    }`
`  }`
`}`

-   Chainti (PE1) received OSPF routes

`user@Chianti> show route protocol ospf table VPN-B.inet.0`
`VPN-B.inet.0: 6 destinations, 6 routes (6 active, 0 holddown, 0 hidden)`
`+ = Active Route, - = Last Active, * = Both`

`192.168.16.1/32 *[OSPF/10] 00:21:35, metric 1`
`                 > to 10.222.29.1 via ge-0/2/0.0`

-   Cabernet (PE2) OSPF configuration. Cabernet need to advertise routes
    to Chablis via OSPF, but the route is currently installed within the
    VRF table on Cabernet as a BGP route. So we need do route
    redistribution via policy.

`user@Cabernet> show configuration policy-options`
`policy-statement bgp-to-ospf {`
`  term advertise-remote-routes {`
`    from protocol bgp;`
`    then accept;`
`  }`
`}`

`user@Cabernet> show configuration routing-instances VPN-B protocols`
`ospf {`
`  export bgp-to-ospf;`
`  area 0.0.0.0 {`
`    interface so-0/1/2.0;`
`  }`
`}`

Domain ID: by default, Cabernet will advertise OSPF route to CE as a type 5 AS external routes. If CEs are still connected via their own WAN infrastructure, this type 5 routes will be less preferred than type 3. To workaround this, use domain ID.
Route Tag: The JUNOS software also uses the VPN route tag to prevent routing loops. When a PE router receives a Type 5 LSA from a CE router, it examines both the LSA ID and the tag value. If the LSA ID is not the local router but the tag values match, the local PE router assumes that a possible loop is forming. To avoid this, the PE doesn’t include that LSA in its SPF calculation.

### Internet Access for VPN Customers

#### Independent Internet Access

-   Option 1: Customer maintains a different circuit to the Internet.
    Customer routes VPN traffic to PE, and Internet traffic to the
    different circuit.
-   Option 2: Customer uses two logical circuit to connect to PE. One is
    for VPN traffic, the other is for internet traffic as a layer 2 link
    to pass through PE and ended as another router which can get traffic
    to the Internet.

#### Distributed Internet Access

-   **Option 1**: two logical circuits between Sangiovese and Chianti.

`user@Sangiovese> show interfaces terse so-0/2/0`
`Interface  Admin Link Proto Local Remote`
`so-0/2/0   up    up`
`so-0/2/0.0 up    up   inet  10.222.30.1/24`
`so-0/2/0.1 up    up   inet  10.222.100.1/24`

so-0/2/0.0 is connecting to Chaniti's VRF table. so-0/2/0.1 is a normal
interface to Chaniti.

`0.0.0.0/0 *[Static/5] 00:02:29`
`           > to 10.222.100.2 via so-0/2/0.1`

Chaniti needs to install a static route to Sangiovese's prefix and
distribute it to the Internet via BGP, so people knows how to return
traffic back to Sangiovese.

-   **Option 2**: two logical interfaces between Sangiovese and Chaniti.
    S sends both VPN and Internet traffic via one interface, but C will
    send the returning internet traffic to S via another traffic. All
    work is done at C. When C receives VPN traffic, it matches entries
    in VRF table, so it is fine. For rest traffic, C needs inet.0 to do
    lookup.

`user@Chianti> show configuration routing-instances VPN-A routing-options`
`static {`
`  route 0.0.0.0/0 next-table inet.0;`
`}`

Same config for return traffic as option 1.

-   **Option 3**: it is different from option 2 is how to deal with
    returning traffic from the Internet. Instead of using static routes,
    it copies S's prefix in the inet.0.

`user@Chianti> show configuration routing-options`
`rib-groups {`
`  vpn-a-into-inet-0 {`
`    import-rib [ VPN-A.inet.0 inet.0 ];`
`  }`
`}`
`route-distinguisher-id 192.168.20.1;`
`autonomous-system 65432;`

`user@Chianti> show configuration routing-instances VPN-A`
`instance-type vrf;`
`interface so-0/1/2.0;`
`vrf-target target:65432:1111;`
`routing-options {`
`  static {`
`    route 0.0.0.0/0 next-table inet.0;`
`  }`
`}`
`protocols {`
`bgp {`
`  group Sangiovese-CE {`
`    type external;`
`    family inet {`
`      unicast {`
`        rib-group vpn-a-into-inet-0;`
`      }`
`    }`
`    peer-as 65000;`
`    as-override;`
`    neighbor 10.222.30.1;`
`  }`
` }`
`}`

#### Centralized Internet Access

-   The main difference between the two models is that just one CE-PE
    router pair is configured for Internet access. This central CE
    router advertises a 0.0.0.0/0 default route to the other CE routers
    within the VPN. This route advertisement attracts Internet bound
    traffic to the central CE router, which then forwards it to the
    Internet.

Transporting Layer 2 Frames across a Provider Network
-----------------------------------------------------

-   The ability to transport all Layer 2 technologies across a common IP
    network using MPLS labels.
-   From forwarding perspective, it is just put L2 frame as a MPLS
    payload. MPLS packets have two labels, outer one is for P routers to
    go to remote PE, inner one is for remote PE to know which customer
    circuits this L2 frame should be forwarded on.
-   From signaling perspective, we need someone to propagate inner
    labels between PEs. Two main varieties: L2VPN (Kireeti Kompella) and
    L2 circuits (Luca Martini)
    -   L2VPN uses the BGP as the mechanism for PE routers to
        communicate with each other about their customer connections.
    -   L2 circuit uses the LDP between PE routers. Every router
        establishes a unique connection for each customer using the VPN.

### Layer 2 VPN

#### MBGP l2vpn NLRI

-   <http://www.ietf.org/rfc/rfc4761.txt>
    -   "BGP was chosen as the means for exchanging L2 VPN information
        for two reasons: it offers mechanisms for both auto-discovery
        and signaling, and allows for operational convergence. A bonus
        for using BGP is a robust inter-AS solution for L2VPNs."
        [2](http://tools.ietf.org/html/draft-kompella-l2vpn-l2vpn-02)

`     +------------------------------------+`
`     |  Length (2 octets)                 |`
`     +------------------------------------+`
`     |  Route Distinguisher  (8 octets)   |`
`     +------------------------------------+`
`     |  VE ID (2 octets)                  |`
`     +------------------------------------+`
`     |  VE Block Offset (2 octets)        |`
`     +------------------------------------+`
`     |  Label Base (3 octets)             |`
`     +------------------------------------+`

VE: VPLS-aware PE.
VE ID: local customer edge ID. To distinguish different customer on same PE.

-   The Encaps Type of customer circuit is carried over extended
    community attribute

`     +------------------------------------+`
`     | Extended community type (2 octets) |`
`     +------------------------------------+`
`     |  Encaps Type (1 octet)             |`
`     +------------------------------------+`
`     |  Control Flags (1 octet)           |`
`     +------------------------------------+`
`     |  Layer-2 MTU (2 octet)             |`
`     +------------------------------------+`
`     |  Reserved (2 octets)               |`
`     +------------------------------------+`

-   Encaps Type:
    -   0—Reserved
    -   1—Frame Relay
    -   2—ATM AAL5 virtual circuit connection transport
    -   3—ATM transparent cell transport
    -   4—Ethernet virtual LAN
    -   5—Ethernet
    -   6—Cisco high-level data link control
    -   7—Point-to-Point Protocol
    -   8—Circuit emulation
    -   9—ATM virtual circuit connection cell transport
    -   10—ATM virtual path connection cell transport
    -   11—IP Interworking
    -   19—Virtual private label switching

====L2=Frame Relay====

-   CE Shiraz configuration

`user@Shiraz> show configuration interfaces so-0/1/0`
`encapsulation frame-relay;`
`unit 600 {`
`  dlci 600;`
`  family inet {`
`    address 10.222.100.1/24;`
`  }`
`}`

-   PE Chardonnay configuration

`user@Chardonnay> show configuration interfaces so-0/1/0`
`dce;`
`encapsulation frame-relay-ccc;`
`unit 600 {`
`  encapsulation frame-relay-ccc;`
`  dlci 600;`
`}`

-   PE Chablis configuration

`user@Chablis> show configuration interfaces so-0/1/0`
`dce;`
`encapsulation frame-relay-ccc;`
`unit 600 {`
`  encapsulation frame-relay-ccc;`
`  dlci 600;`
`}`

-   CE Cabernet configuration

`user@Cabernet> show configuration interfaces so-0/1/2`
`encapsulation frame-relay;`
`unit 600 {`
`  dlci 600;`
`  family inet {`
`    address 10.222.100.2/24;`
`  }`
`}`

-   Now interfaces between PE-CE should up.
    -   For simplicity we’ve used the same DLCI value of 600 on both
        sides of the VPN, but this is not required. Only the PE and its
        connected CE need to agree on the DLCI used.
    -   When you’re using the frame-relay-ccc encapsulation, the DLCI
        values must be greater than or equal to 512.

<!-- -->

-   PE Chardonnay routing instance to configure site-identifier. Shiraz
    router is assigned a site ID of 1, whereas the Cabernet router is
    assigned a site ID of 4.

`user@Chardonnay> show configuration routing-instances`
`FR-Customer {`
`  instance-type l2vpn;`
`  interface so-0/1/0.600;`
`  vrf-target target:65432:1111;`
`  protocols {`
`    l2vpn {`
`      encapsulation-type frame-relay;`
`      site Shiraz-CE {`
`        site-identifier 1;`
`        interface so-0/1/0.600 {`
`          remote-site-id 4;`
`        }`
`      }`
`    }`
`  }`
`}`

-   PE now will advertise label base + range information for its PE-CE
    connection to Shiraz

`user@Chardonnay> show route advertising-protocol bgp 192.168.52.1 detail`
`FR-Customer.l2vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)`
`* 192.168.32.1:2:`**`1:3`**`/96 (1 entry, 1 announced) <<< local site ID=1, label offset=3`
` BGP group internal-peers type Internal`
`     Route Distinguisher: 192.168.32.1:2`
`     Label-base: 800000, range: 2, status-vector: 0x80`
`     Nexthop: Self`
`     Localpref: 100`
`     AS path: I`
`     Communities: target:65432:1111`
`     Layer2-info: encaps:FRAME RELAY, control flags:2, mtu: 0`

-   To compute the exact label to use, Chablis uses the formula:

`VPN label = label-base-remote + local-site-id – label-offset-remote = 800,000 + 4 – 3 = 800,001.`

`user@Chablis> show l2vpn connections`
`Instance: FR-Customer`
`Local site: Cabernet-CE (4)`
`    connection-site Type St Time last up         # Up trans`
`    1               rmt  Up Nov 6 12:50:09 2001  1`
`      Local interface: so-0/1/0.600, Status: Up, Encapsulation: FRAME RELAY`
`      Remote PE: 192.168.32.1, Negotiated control-word: Yes (Null)`
`      Incoming label: 800000, Outgoing label: 800001 <<< here.`

-   Silimarly, Chablis configuration is:
    -   remote-site-id is ignored and JUNOS will automatically start
        from 1 based on the order of the interface config.

`user@Chablis> show configuration routing-instances FR-Customer protocols`
`l2vpn {`
`  encapsulation-type frame-relay;`
`  site Cabernet-CE {`
`  site-identifier 4;`
`    interface so-0/1/0.600;`
`  }`
`}`

`user@Chablis> show route advertising-protocol bgp 192.168.32.1 detail`
`FR-Customer.l2vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)`
`* 192.168.52.1:2:4:1/96 (1 entry, 1 announced)`
` BGP group internal-peers type Internal`
`     Route Distinguisher: 192.168.52.1:2`
`     Label-base: 800000, range: 1, status-vector: 0x0`
`     Nexthop: Self`
`     Localpref: 100`
`     AS path: I`
`     Communities: target:65432:1111`
`     Layer2-info: encaps:FRAME RELAY, control flags:2, mtu: 0`

====L2=ATM====

-   PE Sangiovese configuration

`user@Sangiovese> show configuration interfaces at-0/1/0`
`atm-options {`
`  vpi 0 {`
`    maximum-vcs 1000;`
`  }`
`}`
`unit 800 {`
`  `<u>`encapsulation atm-ccc-vc-mux;`</u>
`  vci 0.800;`
`}`

`user@Sangiovese> show configuration routing-instances ATM-Customer {`
`instance-type l2vpn;`
`interface at-0/1/0.800;`
`vrf-target target:65432:2222;`
`protocols {`
`  l2vpn {`
`    `<u>`encapsulation-type atm-aal5`</u>`;`
`    site Sherry-CE {`
`      site-identifier 1;`
`      interface at-0/1/0.800;`
`    }`
`  }`
`}`

-   PE will receive ATM cells from CE, then assembles to a AAL5 data,
    sends it over LSP. Remote PE then slices it to ATM cells and sends
    it to remote CE.

<!-- -->

-   PE BGP advertisement

`user@Sangiovese> show route advertising-protocol bgp 192.168.52.1 detail`
`ATM-Customer.l2vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)`
`* 192.168.24.1:3:1:1/96 (1 entry, 1 announced)`
` BGP group internal-peers type Internal`
`     Route Distinguisher: 192.168.24.1:3`
`     Label-base: 800000, range: 2, status-vector: 0x80`
`     Nexthop: Self`
`     Localpref: 100`
`     AS path: I`
`     Communities: target:65432:2222`
`                  Layer2-info: encaps:ATM AAL5, control flags:2, mtu: 0`

-   L2VPN is operational

`user@Sangiovese> show l2vpn connections`
`Instance: ATM-Customer`
`Local site: Sherry-CE (1)`
`    connection-site Type St Time last up          # Up trans`
`    2               rmt  Up Jul 16 14:32:58 2003  1`
`      Local interface: at-0/1/0.800, Status: Up, Encapsulation: ATM AAL5`
`      Remote PE: 192.168.52.1, Negotiated control-word: Yes (Null)`
`      Incoming label: 800001, Outgoing label: 800002`

====L2=Ethernet VLAN====

-   PE configuration
    -   The VLAN ID must be greater than or equal to 512 for the Layer 2
        VPN connections. In addition, the values must match on both
        sides of the provider network. This is a difference from what we
        saw with Frame Relay and ATM, where the Layer 2 identifier could
        be different on the CE routers.

`user@Sangiovese> show configuration interfaces fe-0/0/3`
`vlan-tagging;`
`encapsulation vlan-ccc;`
`unit 900 {`
`  encapsulation vlan-ccc;`
`  vlan-id 900;`
`}`

`user@Chardonnay> show configuration routing-instances VLAN-Customer protocols`
`l2vpn {`
`  encapsulation-type ethernet-vlan;`
`  site Shiraz-CE {`
`    site-identifier 2;`
`    interface fe-0/0/0.900;`
`  }`
`}`

`user@Chianti> show l2vpn connections`
`Instance: VLAN-Customer`
`Local site: Sherry-CE (1)`
`    connection-site Type St Time last up          # Up trans`
`    2               rmt  Up Jul 17 09:44:15 2003  2`
`      Local interface: fe-0/3/0.900, Status: Up, Encapsulation: VLAN`
`      Remote PE: 192.168.32.1, Negotiated control-word: Yes (Null)`
`      Incoming label: 800001, Outgoing label: 800002`

====L2=FR-to-ATM====

-   Using "interworking" as encapsulation type

`user@Chardonnay> show configuration routing-instances IP-Interworking protocols l2vpn`
`encapsulation-type interworking;`
`site Shiraz-CE-Frame-Relay {`
`  site-identifier 1;`
`  interface so-0/1/0.0;`
`}`

### Layer 2 Circuit

#### LDP extension

-   Two PEs use LDP targeted hello message to form a session.
-   A new TLV is used to map a label to a circuit.

`+----------------------------------+`
`| FEC Element Type      (1 octet)  |`
`+----------------------------------+`
`| C Bit and VC Type     (2 octets) |`
`+----------------------------------+`
`| VC Information Length (1 octet)  |`
`+----------------------------------+`
`| Group ID              (4 octets) |`
`+----------------------------------+`
`| Virtual Circuit ID    (4 octets) |`
`+----------------------------------+`
`| Interface Parameters  (variable) |`
`+----------------------------------+`

FEC type: 128
C bit: bit 15, 0
VC type:

-   0x0001—Frame Relay DLCI
-   0x0002—ATM AAL5 virtual circuit connection transport
-   0x0003—ATM transparent cell transport
-   0x0004—Ethernet virtual LAN
-   0x0005—Ethernet
-   0x0006—Cisco high-level data link control
-   0x0007—Point-to-Point Protocol
-   0x8008—Circuit emulation
-   0x0009—ATM virtual circuit connection cell transport
-   0x000a—ATM virtual path connection cell transport

VC info length: now constant 4. No interface parameters.
Group ID: now constant 0.
Virtual Circuit ID: uniquely identifies, when combined with the VC type, a particular circuit in the network. The two PE routers must agree on the virtual circuit ID value to establish the Layer 2 Circuit in the network.

-   Example LDP packet with extension

`Jul 28 17:24:18 LDP sent TCP PDU 192.168.32.1 -> 192.168.24.1 (none)`
`Jul 28 17:24:18 ver 1, pkt len 214, PDU len 210, ID 192.168.32.1:0`
`Jul 28 17:24:18 Msg LabelMap (0x400), len 32, ID 9512`
`Jul 28 17:24:18   TLV FEC (0x100), len 16`
`Jul 28 17:24:18     L2CKT, VC Type 32772 VC Id 800 Group 0`
`Jul 28 17:24:18   TLV Label (0x200), len 4`
`Jul 28 17:24:18     Label 100080`

====L2=Frame Relay====

-   PE Chardonnay to PE Chablis

`user@Chardonnay> show configuration protocols l2circuit`
`neighbor 192.168.52.1 {`
`  interface so-0/1/0.600 {`
`    virtual-circuit-id 600;`
`  }`
`}`

`user@Chardonnay> show l2circuit connections`
`Neighbor: 192.168.52.1`
`  Interface             Type St Time last up          # Up trans`
`  so-0/1/0.600 (vc 600) rmt  Up Jul 26 13:16:18 2003  1`
`    Local interface: so-0/1/0.600, Status: Up, Encapsulation: FRAME RELAY`
`    Remote PE: 192.168.52.1, Negotiated control-word: Yes (Null)`
`    Incoming label: 100096, Outgoing label: 100096`

Check List
----------

-   Be able to describe the basic concept of a VPN. A virtual private
    network (VPN) is the operation of a common physical infrastructure
    where each customer remains segregated within its own virtual
    connections. Some common examples of “traditional” VPNs include
    leased lines, Frame Relay, and ATM.

<!-- -->

-   Be able to describe the functions of the CE, PE, and P routers. The
    CE router advertises routes from the customer site to its connected
    PE router using a Layer 3 routing protocol. The PE receives these
    routes and assigns both an RD and a route target to the routes. They
    are then advertised to remote PE routers over MBGP sessions. The
    routes are then readvertised to the CE routers. The P routers, on
    the other hand, are solely responsible for forwarding MPLS packets
    between the PE routers.

<!-- -->

-   Be able to identify the format of a VPN-IPv4 NLRI. The routes
    advertised between PE routers over their MBGP sessions use a
    specialized format. This format includes an MPLS label allocated by
    the originating PE router, an RD, and the actual customer route.
    This combination of information allows the PE routers to keep the
    customer VPNs separate as well as forward traffic appropriately
    within each VPN.

<!-- -->

-   Be able to describe the use of a route target. VPN route targets are
    BGP extended communities that the PE routers use to logically build
    the VPN for each customer. The advertising PE router assigns a route
    target to the customer routes before advertising them to the remote
    PE routers. Each receiving PE router uses its locally configured
    route targets to determine which inbound MBGP routes to accept. The
    route targets further determine which specific VRF table the
    received routes are placed in.

<!-- -->

-   Be able to describe the advertisement of Layer 2 VPN information.
    Once the interface configurations and physical encapsulations are
    configured correctly, knowledge of this connection is advertised
    between PE routers. This advertisement uses MBGP to send virtual
    circuit data between the PE routers.

<!-- -->

-   Be able to describe how a connection is established for a Layer 2
    Circuit. The two PE routers on either end of the Layer 2 Circuit
    form an LDP neighbor relationship between themselves using targeted
    Hello messages. Once they further establish an LDP session, each PE
    router advertises the customer circuit information in a Label
    Mapping message.
