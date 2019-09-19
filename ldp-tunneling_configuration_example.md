![](img/ldp-tunneling-example.png)

* http://www.fr-fr.juniper.net/techpubs/software/junos/junos85/swconfig85-vpns/id-11145789.html

The following list explains how, for packets being sent from Router CE2
to Router CE1, the LDP, RSVP, and VPN labels are announced by the
various routers. These steps include examples of label values
(illustrated in Figure 26).

-   LDP labels
    -   Router PE1 announces LDP label 3 for itself to Router P1.
    -   Router P1 announces LDP label 100,001 for Router PE1 to Router
        P3.
    -   Router P3 announces LDP label 100,002 for Router PE1 to Router
        PE2.
-   RSVP labels
    -   Router P1 announces RSVP label 3 to Router P2.
    -   Router P2 announces RSVP label 100,003 to Router P3.
-   VPN label
    -   Router PE1 announces VPN label 100,004 to Router PE2 for the
        route from Router CE1 to Router CE2.

![](img/ldp-tunneling-example-2.png)

For a packet sent from Host B in Figure 26 to Host A, the packet headers
and labels change as the packet travels to its destination:

1. The packet that originates from Host B has a source address of B and
a destination address of A in its header. 2. Router CE2 adds to the
packet a next-hop of interface so-1/0/0. 3. Router PE2 swaps out the
next-hop of interface so-1/0/0 and replaces it with a next-hop of PE1.
It also adds two labels for reaching Router PE1, first the VPN label
(100,004), then the LDP label (100,002). The VPN label is thus the inner
(bottom) label on the stack, and the LDP label is the outer label. 4.
Router P3 swaps out the LDP label added by Router PE2 (100,002) and
replaces it with its LDP label for reaching Router PE1 (100,001). It
also adds the RSVP label for reaching Router P2 (100,003). 5. Router P2
removes the RSVP label (100,003) because it is the penultimate hop in
the MPLS LSP. 6. Router P1 removes the LDP label (100,001) because it is
the penultimate LDP router. It also swaps out the next-hop of PE1 and
replaces it with the next-hop interface, so-1/0/0. 7. Router PE1 removes
the VPN label (100,004). It also swaps out the next-hop interface of
so-1/0/0 and replaces it with its next-hop interface, ge-1/1/0. 8.
Router CE1 removes the next-hop interface of ge-1/1/0, and the packet
header now contains just a source address of B and a destination address
of A.

Configurations
--------------

### Router PE1

Routing Instance for VPN-A

`routing-instance {`
`   VPN-A {`
`       instance-type vrf;`
`       interface ge-1/0/0.0;`
`       route-distinguisher 65535:0;`
`       vrf-import VPN-A-import;`
`       vrf-export VPN-A-export;`
`   }`
`}`

Instance Routing Protocol

`protocols {`
`   rip {`
`       group PE1-to-CE1 {`
`           neighbor ge-1/0/0.0;`
`       }`
`   }`
`}`

Interfaces

`interfaces {`
`   so-1/0/0 {`
`       unit 0 {`
`           family mpls;`
`       }`
`   }`
`   ge-1/0/0 {`
`       unit 0;`
`   }`
`}`

Master Protocol Instance

`protocols {`
`}`

Enable LDP

`ldp {`
`   interface so-1/0/0.0;`
`}`

Enable MPLS

`mpls {`
`   interface so-1/0/0.0;`
`   interface ge-1/0/0.0;`
`}`

Configure IBGP

`bgp {`
`   group PE1-to-PE2 {`
`       type internal;`
`       local-address 10.255.1.1; `
`       family inet-vpn {`
`           unicast;`
`       }`
`       neighbor 10.255.100.1;`
`   }`
`}`

Configure VPN Policy

`policy-options {`
`   policy-statement VPN-A-import {`
`       term a {`
`           from {`
`               protocol bgp;`
`               community VPN-A;`
`           }`
`           then accept;`
`       }`
`       term b {`
`           then reject;`
`       }`
`   }`
`   policy-statement VPN-A-export {`
`       term a {`
`           from protocol rip;`
`           then {`
`               community add VPN-A;`
`               accept;`
`           }`
`       }`
`       term b {`
`           then reject;`
`       }`
`   }`
`   community VPN-A members target:65535:00;`
`}`

### Router P1

Master Protocol Instance

`protocols {`
`}`

Enable RSVP

`rsvp {`
`   interface so-1/0/1.0;`
`}`

Enable LDP

`ldp {`
`   interface so-1/0/0.0;`
`   interface lo0.0;`
`}`

Enable MPLS

`mpls {`
`   label-switched-path P1-to-P3 {`
`       to 10.255.100.1;`
`       ldp-tunneling;`
`   }`
`   interface so-1/0/0.0;`
`   interface so-1/0/1.0;`
`}`

Configure OSPF for Traffic Engineering Support

`ospf {`
`   traffic-engineering;`
`   area 0.0.0.0 {`
`       interface so-1/0/0.0;`
`       interface so-1/0/1.0;`
`   }`
`}`

### Router P2

Master Protocol Instance

`protocols {`
`}`

Enable RSVP

`rsvp {`
`   interface so-1/1/0.0;`
`   interface at-2/0/0.0;`
`}`

Enable MPLS

`mpls {`
`   interface so-1/1/0.0;`
`   interface at-2/0/0.0;`
`}`

### Router P3

Master Protocol Instance

`protocols {`
`}`

Enable RSVP

`rsvp {`
`   interface at-2/0/1.0;`
`}`

Enable LDP

`ldp {`
`   interface so-0/0/0.0;`
`   interface lo0.0;`
`}`

Enable MPLS

`mpls {`
`   label-switched-path P3-to-P1 {`
`       to 10.255.2.2;`
`       ldp-tunneling;`
`   }`
`   interface at-2/0/1.0;`
`   interface so-0/0/0.0;`
`}`

Configure OSPF for Traffic Engineering Support

`ospf {`
`   traffic-engineering;`
`   area 0.0.0.0 {`
`       interface at-2/0/1.0;`
`       interface at-2/0/1.0;`
`   }`
`}`

### Router PE2

Master Protocol Instance

`protocols {`
`}`

Routing Instance for VPN-A

`routing-instance {`
`   VPN-A {`
`       instance-type vrf;`
`       interface so-1/2/0.0;`
`       route-distinguisher 65535:1;`
`       vrf-import VPN-A-import;`
`       vrf-export VPN-A-export;`
`   }`
`}`

Instance Routing Protocol

`protocols {`
`   ospf {`
`       area 0.0.0.0 {`
`           interface so-1/2/0.0;`
`       }`
`   }`
`}`

Interfaces

`interfaces {`
`   so-0/0/0 {`
`       unit 0 {`
`           family mpls;`
`       }`
`   }`
`   so-1/2/0 {`
`       unit 0;`
`   }`
`}`

Enable LDP

`ldp {`
`   interface so-0/0/0.0;`
`}`

Enable MPLS

`mpls {`
`   interface so-0/0/0.0;`
`   interface so-1/2/0.0;`
`}`

Configure IBGP

`bgp {`
`   group PE2-to-PE1 {`
`       type internal;`
`       local-address 10.255.200.2; `
`       family inet-vpn {`
`           unicast;`
`       }`
`       neighbor 10.255.1.1;`
`   }`
`}`

Configure VPN Policy

`policy-options {`
`   policy-statement VPN-A-import {`
`       term a {`
`           from {`
`               protocol bgp;`
`               community VPN-A;`
`           }`
`           then accept;`
`       }`
`       term b {`
`           then reject;`
`       }`
`   }`
`   policy-statement VPN-A-export {`
`       term a {`
`           from protocol ospf;`
`           then {`
`               community add VPN-A;`
`               accept;`
`           }`
`       }`
`       term b {`
`           then reject;`
`       }`
`   }`
`   community VPN-A members target:65535:01;`
`}`
