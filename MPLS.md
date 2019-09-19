---
title: JNCIS Notes MPLS
permalink: /JNCIS_Notes/MPLS/
---

MPLS Intro.
-----------

-   Basic idea is to use label switching to forward traffic.
    -   Label is allocated locally by a local router and only has meaning to that local router. On the contray, IP is allocated globally.
    -   Label table normally small and can be fast lookup.
-   Singalling protocols to exchange label information
    -   [The Resource Reservation Protocol](RSVP.md)
    -   [The Label Distribution Protocol](LDP.md)
-   MPLS header

<!-- -->

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ Label
    |                Label                  | Exp |S|       TTL     | Stack
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ Entry

      Label:  Label Value, 20 bits
      Exp:    Experimental Use, 3 bits
      S:      Bottom of Stack, 1 bit. S=1: this is last label. S=0: next 32 bits are another label.
      TTL:    Time to Live, 8 bits

RSVP vs LDP
-----------

-   RSVP
    -   pros: support traffic engineering
    -   cons: configuration is complex
-   LDP
    -   pros: configuration is simpler
    -   cons: doesn't support traffic engineering

### LDP Tunneling

RSVP LSP will logically function as a single link, LDP is running on top of it. Now, we have two labels in the middle of LSP.

`user@Zinfandel> show route table mpls.0 detail | find 108480  <<< Zinfandel is LSP ingress router`
`108480 (1 entry, 1 announced)                                 <<< 108480 is the incoming label (learned via LDP)`
`        *LDP  Preference: 9`
`              Next hop: via so-0/1/1.0 weight 1, selected`
`              Label-switched-path from-Zinfandel`
`              Label operation: Swap 100352, Push 100304(top)  <<< swap 108480 with 100352, then push 100304 to enter LSP`
`              State: `<Active Int>
`              Age: 2 Metric: 1`
`              Task: LDP`
`              Announcement bits (1): 0-KRT`
`              AS path: I`
`              Prefixes bound to route: 192.168.16.1/32`

[Configuration
Example](//ldp-tunneling_configuration_example "wikilink")

Constrained Shortest Path First
-------------------------------

-   CSPF: SPF with constraints: bandwidth (max bw, available bw for each priority), Administrative Groups/Link Color (administrative use or not use of a particular link), user speicified ERO.
-   CSPF is based on Traffic Engineering Database (TED)
-   Check the TED for bandwidth and link color information.

`user@Sherry> show ted database Sherry.00 extensive | find Chianti`
`  To: Chianti.02, Local: 10.222.29.1, Remote: 0.0.0.0`
`    Color: 0 `<none>
`    Metric: 10`
`    Static BW: 1000Mbps`
`    Reservable BW: 1000Mbps`
`    Available BW [priority] bps:`
`      [0] 1000Mbps [1] 1000Mbps [2] 1000Mbps [3] 1000Mbps`
`      [4] 1000Mbps [5] 1000Mbps [6] 1000Mbps [7] 1000Mbps`
`    Interface Switching Capability Descriptor(1):`
`    Switching type: Packet`
`    Encoding type: Packet`
`    Maximum LSP BW [priority] bps:`
`      [0] 1000Mbps [1] 1000Mbps [2] 1000Mbps [3] 1000Mbps`
`      [4] 1000Mbps [5] 1000Mbps [6] 1000Mbps [7] 1000Mbps`

### Populate information into TED

-   OSPF: Type 10 LSA including Node and Link information, such as max bandwidth, unreserved bandwidth, color etc. Details see [OSPF Type 10 LSA](/JNCIS_Notes/OSPF#Type_10:_MPLS_Traffic_Engineering_LSA "wikilink").
-   ISIS: Type 22 Extended IS Reachability TLV. Details see [ISIS Type 22 TLV](/JNCIS_Notes/ISIS#Extended_IS_Reachability_.28TLV_22.29 "wikilink").
-   By default, the JUNOS software advertises TED information after a change of 10 percent of the interface’s available bandwidth. You can alter this percentage between 1 and 20 percent with the update-threshold command. This is applied to individual interfaces within the `edit protocols rsvp` configuration hierarchy.

### CSPF Algorithm Steps

1. When a bandwidth reservation is requested for the LSP, the algorithm removes all links from the database that don’t have enough unreserved bandwidth available at the setup priority level of the LSP.

2. When the LSP requires a network path that includes links belonging to an administrative group, the algorithm removes all links that don’t contain the requested group value.

3. When the LSP requires a network path that excludes links belonging to an administrative group, the algorithm removes all links that currently contain the requested group value.

4. The algorithm calculates the shortest path from the ingress to the egress router using the remaining information in the database. When an explicit route is requested by the user, separate shortest-path calculations are performed. The first is run from the ingress router to the first hop in the route. The next is calculated from the first hop of the route to the next hop in the route, if one is defined. This step-by-step process is performed for each node defined in the route until the egress router is encountered.

5. When multiple equal-cost paths exist from the ingress to egress router, the path whose lasthop interface address matches the egress router address is selected. This algorithm step is helpful when the egress router address is not a loopback address.

6. If multiple equal-cost paths still exist, the algorithm selects the path with the fewest number of physical hops.

7. In the event that multiple equal-cost paths continue to exist, the algorithm selects one of the paths based on the load-balancing configuration of the LSP: random, most-fill, or least-fill.

8. The algorithm generates an ERO containing the physical interface address of each node along the path from ingress to egress router. This ERO is used by the RSVP process to signal and establish the LSP.

### Administrative Groups

-   Administrative Groups is a bit vector. A group value of 5 sets bit number 5 to a value of 1 in the vector.
-   Define Administrative Group

`user@Sherry> show configuration protocols mpls`
`  admin-groups {`
`    Gold 2;`
`    Silver 15;`
`    Bronze 28;`
`  }`
`  interface all;`
`  interface at-0/1/0.0 {`
`    admin-group Silver;`
`  }`
`  interface ge-0/2/0.0 {`
`    admin-group [ Gold Bronze ];`
`  }`

-   Use Administrative Group

`user@Sherry# show protocols mpls label-switched-path Sherry-to-Char`
`  to 192.168.32.1;`
`  admin-group {`
`    include Silver;`
`    exclude Gold;`
`  }`

### LSP priority and preemption

Setup Priority: used to determine if enough bandwidth is available at that priority level (between 0 and 7) to establish the LSP. 0: best; 7: worst.
Hold Priority: used by an established LSP to retain its bandwidth reservations in the network. When a new LSP with higher/smaller setup priority than existing LSP's hold priority, existing LSP will be preempted. 0: strongest; 7: weakest.

-   Default setup priority is **7**, hold priority is **0**.
-   The setup priority must be equal to or less than the hold priority.
    -   **LSP1**: *setup*=2, *hold*=5. **LSP2**: *setup*=2, *hold*=5.
    -   **LSP1** is established.
    -   **LSP2** is in process of estabalishment, but no enough
        bandwidth for both **LSP1** and **LSP2**.
    -   **LSP2**.*setup* &lt; **LSP1**.*hold*, **LSP2** is established.
        **LSP1** is preempted.
    -   **LSP1** tries to re-establish.
    -   **LSP1**.*setup* &lt; **LSP2**.*hold*, the process will loop for
        ever.

`label-switched-path high-pri-LSP {`
`  to 192.168.32.1;`
`  bandwidth 90m;`
`  priority 3 3;`
`  primary via-Sangiovese;`
`}`

LSP Traffic Protection
----------------------

### Primary LSP Paths

-   User can specify user-defined ERO, bandwidth, admin group, priority, etc.
-   One LSP can have 0 or 1 primary path.

retry-timer: default=30 seconds.

1.  controls the length of time for ingress router to re-establish primary path after it failed;
2.  after primary path comes up again, ingress router will wait sometime between retry-timer and 2\*retry-timer to actually use primary path: to avoid instability.

retry-limit: default=0. If not 0, ingress will stop attempting to reestablish primary path.

`user@Sherry# show label-switched-path Sherry-to-Char`
`  to 192.168.32.1;`
`  primary via-Chablis {`
`    bandwidth 20m;`
`  }`

`  path via-Chablis {`
`    192.168.3.1;`
`  }`

### Secondary LSP Paths

`  secondary via-Chianti {`
`    bandwidth 10m;`
`    standby;`
`  }`
`  path via-Chianti {`
`    192.168.4.1;`
`  }`

-   When ingress router receives ResvTear message from its neighbor, it
    clears primary path then ran CSPF using constraints from secondary
    path configuration. The resulting ERO will passed to RSVP process
    and secondary path is signaled.
-   Between primary path down and secondary path up, there will be no
    LSP available and traffic will be forwarded using IPv4 nexthop.

#### Standby secondary path

-   If using "standby", secondary path will be signaled when primary
    path is up.
-   Once primary path becomes up again, it will not be used immediately.
    Instead, sometime upto 2\*retry-time will be waited before primary
    path is used. Similarly, secondary path is not immediately cleared,
    it will wait sometime before really tearing down (when standby is
    not used)
-   One LSP can have multiple secondary path without primary path.
    Because primary path is "revertive" (taking traffic when it came
    back up), it causes another network traffic shift. If no primary
    path, the working secondary path fails then comes up, it will not be
    used for forwarding traffic.

#### Adaptive Mode

-   The default bandwidth reservation style in the JUNOS software is
    fixed filter (FF). This style allocates a unique bandwidth
    reservation for each sender (LSP ID) and receiver (Tunnel ID) pair.
    The opposite reservation style is known as shared explicit (SE),
    which allocates a single reservation for each unique RSVP session.
    This reservation can be shared among multiple sending ID values from
    the ingress router. The SE reservation style is enabled by
    configuring the adaptive keyword within your LSP.
-   When primary and secondary path use same link, without adaptive
    mode, primary and secondary will request bandwidth seperately. With
    adaptive mode, primary and secondary path can share the same
    bandwidth.
-   It also allows make-before-break operation when moving traffic from
    one path to another path.

`user@Sherry# show label-switched-path Sherry-to-Char`
`to 192.168.32.1;`
`bandwidth 75m;`
`adaptive;`
`primary via-Sangiovese;`
`secondary via-Chianti {`
`  standby;`
`}`

### Fast Reroute

-   Problem to solve: even with primary path and standby secondary path,
    a failover still may cause packets drop especially for big pipes
    (like OC192 trunk). Ingress router only switches to secondary once
    it receives ResvTear message from the failure point, which may be
    several hops away. Between the time when failure happens and when
    ingress router is informed, ingress router still pushes label, which
    is invalid at the failure point, to the traffic, which results
    packet loss.

#### Node Protection

-   Using [Fast Reroute
    Object](/JNCIS_Notes/MPLS/RSVP#Fast_Reroute_Object "wikilink") in
    RSVP session to inform each router along the path to automatically
    build a detour path which avoids to use the next router.
-   By default, detour route inherits the constraints from main route.
    In addition, user can limit the bandwidth and number of hops.
-   During the failure, the upstream router next to the failure node
    will use detour route to label/pass the traffic to complete bypass
    the failure node.
-   It is one detour path to one RSVP LSP, not very scalable.

`user@Sherry# show label-switched-path Sherry-to-Char`
`  to 192.168.32.1;`
`  fast-reroute;`
`  primary via-Chablis;`

`user@Sherry> show rsvp session ingress detail`
`Ingress RSVP: 1 sessions`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 0`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  Suggested label received: -, Suggested label sent: -`
`  Recovery label received: -, Recovery label sent: 100256`
`  Resv style: 1 FF, Label in: -, Label out: 100256`
`  Time left: -, Since: Sat Jun 7 11:26:43 2003`
`  Tspec: rate 0bps size 0bps peak Infbps m 20 M 1500`
`  Port number: sender 1 receiver 11185 protocol 0`
`  `<u>`FastReroute desired`</u>
`  PATH rcvfrom: localclient`
`  PATH sentto: 10.222.5.2 (at-0/1/1.0) 56 pkts`
`  RESV rcvfrom: 10.222.5.2 (at-0/1/1.0) 58 pkts`
`  Explct route: 10.222.5.2 10.222.60.2 10.222.6.2`
`  Record route: `<self>` 10.222.5.2 10.222.60.2 10.222.6.2`
`    Detour is Up`
`    Detour PATH sentto: 10.222.28.2 (at-0/1/0.0) 55 pkts`
`    Detour RESV rcvfrom: 10.222.28.2 (at-0/1/0.0) 55 pkts`
`    `<u>`Detour Explct route: 10.222.28.2 10.222.4.2 10.222.44.2`</u>
`    Detour Record route: `<self>` 10.222.28.2 10.222.4.2 10.222.44.2`
`    Detour Label out: 100240`

`user@Chablis> show rsvp session transit detail`
`Transit RSVP: 1 sessions`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 1`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  FastReroute desired`
`  PATH rcvfrom: 10.222.5.1 (at-0/2/0.0) 237 pkts`
`  PATH sentto: 10.222.60.2 (so-0/1/0.0) 237 pkts`
`  RESV rcvfrom: 10.222.60.2 (so-0/1/0.0) 238 pkts`
`  Explct route: 10.222.60.2 10.222.6.2`
`  Record route: 10.222.5.1 `<self>` 10.222.60.2 10.222.6.2`
`  Detour is Up`
`    Detour PATH sentto: 10.222.62.2 (so-0/1/1.0) 236 pkts`
`    Detour RESV rcvfrom: 10.222.62.2 (so-0/1/1.0) 236 pkts`
`    Detour Explct route: 10.222.62.2 10.222.3.2 10.222.45.2`
`    Detour Record route: 10.222.5.1 `<self>` 10.222.62.2 10.222.3.2 10.222.45.2`
`    Detour Label out: 100176`

`user@Cabernet> show rsvp session transit detail`
`Transit RSVP: 1 sessions`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 1`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  FastReroute desired`
`  PATH rcvfrom: 10.222.60.1 (so-0/1/2.0) 243 pkts`
`  PATH sentto: 10.222.6.2 (so-0/1/1.0) 243 pkts`
`  RESV rcvfrom: 10.222.6.2 (so-0/1/1.0) 243 pkts`
`  Explct route: 10.222.6.2`
`  Record route: 10.222.5.1 10.222.60.1 `<self>` 10.222.6.2`
`  Detour is Up`
`    Detour PATH sentto: 10.222.61.1 (so-0/1/0.0) 242 pkts`
`    Detour RESV rcvfrom: 10.222.61.1 (so-0/1/0.0) 242 pkts`
`    Detour Explct route: 10.222.61.1 10.222.3.2 10.222.45.2`
`    Detour Record route: 10.222.5.1 10.222.60.1 `<self>` 10.222.61.1 10.222.3.2 10.222.45.2`
`    Detour Label out: 100160`

-   Since both Chablis and Cabernet use Zinfandel as detour, Zinfandel
    has chance to merge two into one. All in all, these two detour
    belongs to same main LSP and goes to same destination.

`user@Zinfandel> show rsvp session transit detail`
`Transit RSVP: 1 sessions, 1 detours`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 1`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  Detour branch from 10.222.62.1, to skip 192.168.48.1, Up`
`    PATH rcvfrom: 10.222.62.1 (so-0/1/3.0) 247 pkts`
`    PATH sentto: 10.222.3.2 (so-0/1/1.0) 247 pkts`
`    RESV rcvfrom: 10.222.3.2 (so-0/1/1.0) 248 pkts`
`    Explct route: 10.222.3.2 10.222.45.2`
`    Record route: 10.222.5.1 10.222.62.1 `<self>` 10.222.3.2 10.222.45.2`
`    Label in: 100176, Label out: `**`100224`**
`  Detour branch from 10.222.61.2, to skip 192.168.32.1, Up`
`    PATH rcvfrom: 10.222.61.2 (so-0/1/2.0) 247 pkts`
`    PATH sentto: 10.222.3.2 (so-0/1/1.0) 0 pkts`
`    RESV rcvfrom: 10.222.3.2 (so-0/1/1.0) 0 pkts`
`    Explct route: 10.222.3.2 10.222.45.2`
`    Record route: 10.222.5.1 10.222.60.1 10.222.61.2 `<self>` 10.222.3.2 10.222.45.2`
`    Label in: 100160, Label out: `**`100224`**

`user@Merlot> show rsvp session transit detail`
`Transit RSVP: 1 sessions`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 1`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  Detour branch from 10.222.62.1, to skip 192.168.48.1, Up`
`  Detour branch from 10.222.61.2, to skip 192.168.32.1, Up`
`    PATH rcvfrom: 10.222.3.1 (so-0/1/1.0) 257 pkts`
`    PATH sentto: 10.222.45.2 (so-0/1/2.0) 259 pkts`
`    RESV rcvfrom: 10.222.45.2 (so-0/1/2.0) 257 pkts`
`    Explct route: 10.222.45.2`
`    Record route: 10.222.5.1 10.222.62.1 10.222.3.1 `<self>` 10.222.45.2`
`    Label in: `**`100224`**`, Label out: 3`

`user@Chardonnay> show rsvp session egress detail`
`Egress RSVP: 1 sessions, 2 detours`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 0`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  Resv style: 1 FF, Label in: 3, Label out: -`
`  `<u>`FastReroute desired`</u>
`  PATH rcvfrom: 10.222.6.1 (so-0/1/2.0) 263 pkts`
`  PATH sentto: localclient`
`  RESV rcvfrom: localclient`
`  Record route: 10.222.5.1 10.222.60.1 10.222.6.1 `<self>
`  Detour branch from 10.222.28.1, to skip 192.168.52.1, Up`
`    PATH rcvfrom: 10.222.44.1 (so-0/1/0.0) 262 pkts`
`    PATH sentto: localclient`
`    RESV rcvfrom: localclient`
`    Record route: 10.222.28.1 10.222.4.1 10.222.44.1 `<self>
`    Label in: 3, Label out: -`
`  Detour branch from 10.222.62.1, to skip 192.168.48.1, Up`
`  Detour branch from 10.222.61.2, to skip 192.168.32.1, Up`
`    PATH rcvfrom: 10.222.45.1 (so-0/1/1.0) 264 pkts`
`    PATH sentto: localclient`
`    RESV rcvfrom: localclient`
`    Record route: 10.222.5.1 10.222.62.1 10.222.3.1 10.222.45.1 `<self>
`    Label in: 3, Label out: -`

#### Link Protection

-   Each node build a LSP to its neighbor to bypass the interconnecting
    link.
-   When a failure occurs, the point of local repair (PLR) performs a
    label swap operation to place the label advertised by the downstream
    node in the bottom MPLS header. The PLR then performs a label push
    operation to add the label corresponding to the bypass LSP and
    forwards the packet along the bypass. The penultimate router along
    the bypass pops the top label value and forwards the remaining data
    to the router that is downstream of the point of local repair. The
    received label value is known to this router since it originally
    allocated it for the main LSP. As such, it performs the appropriate
    label function and forwards the traffic along the path of the main
    LSP. Through this process, <u>a bypass LSP can officially support
    repair capabilities for multiple LSPs in a many-to-one fashion.</u>

`user@Cabernet> show configuration protocols rsvp`
`  interface all {`
`    link-protection;`
`  }`

`user@Cabernet> show rsvp session ingress detail`
`Ingress RSVP: 3 sessions`

`192.168.32.1`
`  From: 192.168.48.1, LSPstate: Up, ActiveRoute: 0`
`  LSPname: Bypass_to_10.222.6.2`
`  Type: Bypass LSP`
`  PATH rcvfrom: localclient`
`  PATH sentto: 10.222.61.1 (so-0/1/0.0) 4 pkts`
`  RESV rcvfrom: 10.222.61.1 (so-0/1/0.0) 4 pkts`
`  Explct route: 10.222.61.1 10.222.3.2 10.222.45.2`
`  Record route: `<self>` 10.222.61.1 10.222.3.2 10.222.45.2`

`192.168.52.1`
`  From: 192.168.48.1, LSPstate: Up, ActiveRoute: 0`
`  LSPname: Bypass_to_10.222.60.1`
`  Type: Bypass LSP`
`  PATH rcvfrom: localclient`
`  PATH sentto: 10.222.61.1 (so-0/1/0.0) 4 pkts`
`  RESV rcvfrom: 10.222.61.1 (so-0/1/0.0) 4 pkts`
`  Explct route: 10.222.61.1 10.222.62.1`
`  Record route: `<self>` 10.222.61.1 10.222.62.1`

`192.168.56.1`
`  From: 192.168.48.1, LSPstate: Up, ActiveRoute: 0`
`  LSPname: Bypass_to_10.222.61.1`
`  Type: Bypass LSP`
`  PATH rcvfrom: localclient`
`  PATH sentto: 10.222.60.1 (so-0/1/2.0) 4 pkts`
`  RESV rcvfrom: 10.222.60.1 (so-0/1/2.0) 4 pkts`
`  Explct route: 10.222.60.1 10.222.62.2`
`  Record route: `<self>` 10.222.60.1 10.222.62.2`

-   To use link protection bypasses, LSP configures *link-protection*
    command. All interfaces along this LSP has to be locally configured
    to do link protection (shown above). Otherwise, even the LSP desires
    to be link protected, but actually it won't get it.

`user@Sherry# show label-switched-path Sherry-to-Char`
`  to 192.168.32.1;`
`  link-protection;`
`  primary via-Chablis;`

`user@Cabernet> show rsvp session detail`
`Ingress RSVP: 3 sessions`
`... (bypass LSPs shown above)`

`Transit RSVP: 1 sessions`

`192.168.32.1`
`  From: 192.168.16.1, LSPstate: Up, ActiveRoute: 0`
`  LSPname: Sherry-to-Char, LSPpath: Primary`
`  `<u>`Link protection desired`</u>
`  `<u>`Type: Link protected LSP`</u>
`  PATH rcvfrom: 10.222.60.1 (so-0/1/2.0) 3 pkts`
`  PATH sentto: 10.222.6.2 (so-0/1/1.0) 3 pkts`
`  RESV rcvfrom: 10.222.6.2 (so-0/1/1.0) 4 pkts`
`  Explct route: 10.222.6.2`
`  Record route: 10.222.5.1 10.222.60.1 `<self>` 10.222.6.2`

Controlling LSP Behavior
------------------------

### Explicit Null Advertisements

-   Egress router advertises a label 0 to penultimate router to let it
    forward the label instead pop it out, which is useful when egress
    router would like to use label experiment bits (for CoS use).

### Controlling Time-to-Live

-   The default operation w.r.t. TTL is
    1.  ingress router decrements IP TTL by 1.
    2.  <u>ingress router copies IP TTL to MPLS TTL. </u>
    3.  routers along LSP decrements MPLS TTL by 1.
    4.  penultimate router decrements MPLS TTL by 1, and pops up MPLS
        label.
    5.  <u>penultimate router copies MPLS TTL back to IP TTL.</u>
    6.  egress router decrement IP TTL by 1.
-   With "*no-decrement-ttl*" or "*no-propagate-ttl*" option under
    "protocol mpls",
    1.  ingress router decrements IP TTL by 1.
    2.  <u>ingress router put MPLS TTL as 255.</u>
    3.  routers along LSP decrements MPLS TTL by 1.
    4.  penultimate router decrements MPLS TTL by 1, and pops up MPLS
        label.
    5.  <u>penultimate router does not copy MPLS TTL back to IP TTL.</u>
    6.  egress router decrement IP TTL by 1.

    -   *no-decrement-ttl* needs a special Label Request Object to
        signal this intention, while *no-propagate-ttl* doesn't (for
        better interoperability).

### LSP and Routing Protocol Interactions

-   When a LSP is setup, egress router address will be put into inet.3.
-   LSP is by default used by BGP when looking up nexthop, because BGP
    will consult inet.3 first, then inet.0.
-   LSP can be used by IGP by putting LSP into ISIS as a virtual link.
    **LSP has to be bi-directional,** and both ends of the LSP need to
    put the LSP into ISIS.

`user@Sangiovese> show configuration protocols isis`
`level 1 disable;`
`level 2 wide-metrics-only;`
`interface all;`
`label-switched-path Sangio-to-Char {`
`  level 2 metric 5;`
`}`

-   "*traffic-engineering bgp-igp*" **moves** inet.3 routes to inet.0.
-   "*traffic-engineering bgp-igp-both-ribs*" **copies** inet.3 routes
    to inet.0.
    -   RSVP routes have lower protocol preference so be more
        preferrable. IGP routes will not be preferrable thus will be
        inactive. It may break some policies because policies only apply
        to active routes.
-   "*traffic-engineering mpls-forwarding*" **copies** inet.3 routes to
    inet.0, but still keep IGP routes active.

`user@Sherry> show configuration protocols mpls`
`traffic-engineering mpls-forwarding;`
`label-switched-path Sherry-to-Char {`
`  to 192.168.32.1;`
`  primary via-Sangiovese;`
`}`
`path via-Sangiovese {`
`  192.168.24.1 loose;`
`}`
`interface all;`

`user@Sherry> show route 192.168.32.1`
`inet.0: 29 destinations, 30 routes (25 active, 0 holddown, 4 hidden)`
`@ = Routing Use Only, `**`#`` ``=`` ``Forwarding`` ``Use`` ``Only`**
`+ = Active Route, - = Last Active, * = Both`

`192.168.32.1/32 @[IS-IS/18] 00:00:08, metric 30`
`                   to 10.222.29.2 via ge-0/2/0.0`
`                 > to 10.222.28.2 via at-0/1/0.0`
`                #[RSVP/7] 00:00:04, metric 30`
`                 > via at-0/1/0.0, label-switched-path Sherry-to-Char`

`inet.3: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)`
`@ = Routing Use Only, # = Forwarding Use Only`
`+ = Active Route, - = Last Active, * = Both`

`192.168.32.1/32 *[RSVP/7] 00:00:04, metric 30`
`                 > via at-0/1/0.0, label-switched-path Sherry-to-Char`

`traceroute to 192.168.32.1 (192.168.32.1), 30 hops max, 40 byte packets`
`1 10.222.28.2 (10.222.28.2) 1.531 ms 1.345 ms 1.109 ms`
`   MPLS Label=100464 CoS=0 TTL=1 S=1`
`2 10.222.4.2 (10.222.4.2) 1.369 ms 0.895 ms 1.152 ms`
`   MPLS Label=100464 CoS=0 TTL=1 S=1`
`3 192.168.32.1 (192.168.32.1) 0.901 ms 0.915 ms 1.160 ms`

Check List
----------

-   Be able to explain how an RSVP-based path is established in an MPLS
    network. An MPLS network using RSVP for signaling label-switched
    paths uses Path and Resv messages for the establishment of the LSP.
    Path messages are transmitted downstream, whereas Resv messages are
    transmitted upstream. Each message enables routers to allocate and
    maintain protocol state associated with the path.

<!-- -->

-   Know the RSVP message types used in an MPLS network for removing
    protocol state. RSVP routers remove protocol state from the network
    by transmitting PathTear and ResvTear messages. These messages
    traverse the LSP path in the same direction of their counterparts.
    PathTear messages are transmitted downstream, whereas ResvTear
    messages are sent upstream.

<!-- -->

-   Be able to describe the RSVP objects used to route an LSP through
    the network and allocate label information. An RSVP-based LSP uses
    the Explicit Route object to formulate a path through an MPLS
    network. This allows the LSP to utilize links other than that
    specified by the IGP shortest path. The actual path used by the LSP
    is detailed in the Record Route object, which is also used for loop
    prevention during the LSP establishment phase. Finally, the MPLS
    labels used to forward traffic through the LSP are assigned using
    the Label Request and Label objects.

<!-- -->

-   Be able to describe how RSVP sessions are identified in an MPLS
    network. Each established label-switched path belongs to a specific
    RSVP session in the network. This session is uniquely identified by
    information contained in the Sender-Template, Session, and Session
    Attribute objects. Specifically, the address of the egress router,
    the tunnel ID, and the LSP ID are used to identify the session.

<!-- -->

-   Be able to describe how LDP forms a session between two peers. Two
    neighboring LDP routers first begin exchanging Hello messages with
    each other. These messages contain the label space advertised by the
    local router as well as the transport address to be used for the
    establishment of the session. Once each router determines the
    neighbor on its interface, the peer with the higher transport
    address becomes the active node. This active node sends
    initialization messages to the passive node to begin the session
    setup phase. Once the passive peer returns its own initialization
    message, the session becomes fully established.

<!-- -->

-   Understand how address and label information is propagated in an LDP
    network. After two LDP routers form a session between themselves,
    they begin advertising their local interface addresses to the peer
    with address messages. This allows the receiving peer to associate a
    physical next hop with the established session. Once this exchange
    is completed, the peers advertise reachable prefixes and labels in a
    label mapping message. The prefixes advertised by each LDP router
    compose a FEC, which is readvertised throughout the LDP network.

<!-- -->

-   Be able to describe how information is placed into the traffic
    engineering database. Both OSPF and IS-IS have been extended to
    carry traffic engineering specific information within their protocol
    updates. The Type 10 Opaque LSA and the Extended IS Reachability TLV
    contain bandwidth, administrative group, and addressing information.
    These advertisements are then placed into the TED on each IGP
    router.

<!-- -->

-   Know the steps used by the CSPF algorithm to locate a path for the
    LSP. The algorithm first removes all links from the TED that don’t
    meet the bandwidth requirements of the LSP. Then all links that do
    not contain any included administrative group information are
    removed. If the LSP is configured to exclude any administrative
    group information, those links are then removed by the algorithm.
    Using the remaining links, the algorithm locates the shortest path
    through the network from the ingress to the egress router.

<!-- -->

-   Be familiar with methods available to the ingress router for
    protecting MPLS traffic flows. An individual LSP may be configured
    for both a primary and a secondary path through the network. When
    the primary path is available, it is used for traffic forwarding. In
    a failure mode, the ingress router establishes the secondary path
    and begins forwarding traffic using MPLS labels once again. The
    failover time on the ingress router can be shortened by allowing the
    secondary path to be established in the network prior to the primary
    path failure. This is accomplished through the use of the standby
    command.

<!-- -->

-   Be able to describe how user traffic within an LSP is protected from
    failures. Once the ingress router has begun forwarding traffic along
    the LSP, a network failure in the path can cause those packets to be
    dropped from the network. To alleviate this problem, the LSP can be
    configured for either node or link protection fast reroute. These
    methods allow all routers in the LSP to preestablish paths in the
    network to avoid the next downstream link or node. During a failure
    mode, the device that notices the failure immediately begins using
    the temporary paths to continue forwarding traffic within the LSP.

<!-- -->

-   Be able to describe how multiple LSP paths share bandwidth
    reservations. By default, all newly established paths in the network
    reserve unique and distinct bandwidth reservations using the fixed
    filter style. Within a single RSVP session, multiple paths can share
    a reservation when the shared explicit style is used during the path
    establishment. Within the JUNOS software, the adaptive command
    accomplishes this goal. In addition, adaptive mode allows the
    ingress router to use a make-before-break system for rerouting an
    LSP in the network. The new path is established and traffic is moved
    before the old path is torn down.

<!-- -->

-   Understand methods used to provide LSP visibility to non-BGP learned
    routes. Advertising the LSP to the IGP link-state database is one
    effective method for accomplishing this goal. Each router in the
    network then sees the LSP as a point-to-point link with some metric
    value attached. This information is used in the SPF calculation on
    the router, which may result in the LSP being used for traffic
    forwarding.
