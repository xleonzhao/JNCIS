Example Topology:

    +-----+ 172.16.0.0/16           +-----+
    | AS1 | 172.16.0.0/17 MED=50    | AS2 |
    |     | 172.16.128.0/17 MED=100 |     | 172.16.0.0/16
    | RT1-----------------------------RTA-----------------AS3
    |     |                         |     |
    | RT2-----------------------------RTB-----------------AS4
    |     | 172.16.0.0/16           |     | 172.16.0.0/16
    +-----+ 172.16.0.0/17 MED=100   +-----+
            172.16.128.0/17 MED=50

The RT1 and RT2 routers in above network are assigned the 172.16.0.0 /16 address space in AS1. The administrators of AS1 have assigned their address space so that the 172.16.0.0 /17 subnet is closer to RT1 while the 172.16.128.0 /17 subnet is closer to RT2. They would like to advertise these two subnets, as well as their larger aggregate route, to their peers in AS2. The two subnets should be used by AS2 to forward user traffic based on the assigned MED values. In addition, the 172.16.0.0 /16 aggregate route should be readvertised for reachability from the Internet (represented by AS3 and AS4). The Internet routers don’t have a need to receive the subnet routes since they provide no useful purpose outside the boundary of AS2. While the administrators of AS2 could filter these subnets before advertising them, it is a perfect scenario for using the No-Export wellknown community.

    [edit policy-options]
    user@RT2# show policy-statement adv-routes
    term aggregate {
      from {
        route-filter 172.16.0.0/16 exact;
      }
      then accept;
    }
    term subnets {
      from {
        route-filter 172.16.0.0/16 longer;
      }
      then {
        community add only-to-AS65020;
        accept;
      }
    }

Same principle can be applied to *no-advertise* for the following
topology:

    +-----+                         +-----+
    | AS1 | 172.16.0.0/16           | AS2 |
    |     | 172.16.0.0/17           |     |
    |    /---------------------------\    | 172.16.0.0/16
    | RT1 |                         | RTA-----------------AS3
    |    \---------------------------/    |
    |     | 172.16.0.0/16           |     |
    +-----+ 172.16.128.0/17         +-----+
