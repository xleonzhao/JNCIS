Routing policies control routing information transferred into and out of the routing table. It happens when

-   you do not want to import all learned routes into the routing table;
-   you do not want to advertise all your routes to neighboring routers;
-   you want one protocol to receive routes from another protocol;
-   you want to modify information associated with a route.

Routing Policy Processing
-------------------------

Evaluation `E()={if match then accept/reject and stop; else continue; fi}` go through

-   policy A
    -   E(term A)
    -   E(term B)
    -   E(term C)
    -   ...
-   policy B
    -   E(term A')
    -   E(term B')
    -   E(term C')
    -   ...
-   policy C
-   ...
-   Default policy yields either accept or reject

### Policy Chains

-   Reusable
-   Protocol default policy is always applied at the end of policy chain. BGP Default Policy is to advertise all BGP learned routes and reject anything else.

### Policy Subroutines

-   referred as a match criterion .
-   return only true or false, **not** accept/reject action. For modifying actions, yes it does modify.
-   **Note**: protocol default policy is evaluated within subroutine.
-   An [example](Policy_Subroutine_Example.md) show how subroutine can break things by combining above two.

### Prefix Lists

-   The way to match a prefix in prefix list is **exact match**.

### Policy Expressions

-   example: : A route has to be accepted in both policies in order to
    be exported in this example.
-   Alters the default way to process a policy chain.
-   `NOT (!) > AND (&&) > OR (||)`

Communities
-----------

-   Community is a 32 bits or 64 bits (extended community) value set between neighboring ASes, which is for coordinating policies / route preferences between neighbors.
-   valid values are from 1 to 65535.

### Regular Communities

-   community value 16 bit is AS number of the network originated the community, which makes sure community is globally unique.
-   define a community,

<!-- -->

    policy-options {
      community name members [community-ids];
    }

If multiple community ids are defined, they are logically **AND**.

- locate a route using a defined community name.

<!-- -->

    term find-Riesling-routes {
      from community out-via-Riesling;
      then {
        local-preference 200;
       }
    }
    community out-via-Riesling members 65010:1234;

Or from CLI:

` show route terse community 65010:4444`
` show route terse community-name both-comms`
` show route receive-protocol bgp 10.100.10.1 detail`

- apply or delete a community value

<!-- -->

    policy-statement delete-a-community {
      term delete-comm {
        from {
          route-filter 192.168.2.0/24 exact;
        }
        then {
          community add/delete/set comm-2;
        }
      }
    }
    community comm-2 members 65020:200;

### Extended Communities

-   mainly for VPN use, 8 octets
-   **Type**: 2 octets.
    -   high order octet=0x00: Administrator has 4 octets
    -   high order octet=0x01: Administrator has 2 octets
    -   lower order octe=0x02: route target or *target* as a JUNOS
        keyword,
    -   lower order octe=0x03: route origin or *origin*
-   **Administrator**: either 4 octets or 2 octets depending on high
    order octet of Type value.
    -   4 octets normally has router ID of originator
    -   2 octets normally has AS number of originator.
-   **Assigned Number**: either 2 octets or 4 octets

<!-- -->

    community origin-as members origin:65020:3;
    community origin-ip members origin:192.168.7.7:4;
    community target-as members target:65020:1;
    community target-ip members target:192.168.7.7:2;

### Regular Expressions

` show route community 65010:4...`

-   4?: has zero or one occurrence of 4;
-   4\*: has zero or more occurrence of 4;

<!-- -->

    user@Shiraz# show policy-options policy-statement test-regex
    term find-routes {
      from community complex-regex;
      then accept;
    }
    term reject-all-else {
      then reject;
    }

    community complex-regex members "^65020:[13].*$";

    user@Shiraz> test policy test-regex 0/0 <<< it will test all routes in the routing table.

Autonomous System Paths
-----------------------

### Regular Expressions

-   **.** represents 1 AS number
-   **?** represents 0 or 1 AS number
-   **.+** 1 AS number **repeats** itself
-   **". (65040\|65050)?"**: AS path length is at 1 or 2. If it is 2, second AS number must be either 65040 or 65050.

### Locating Routes

    show route terse aspath-regex ".* 65060"

    policy-statement reject-AS65030 {
      term find-routes {
        from as-path orig-in-65030;
      then reject;
      }
    }
    as-path orig-in-65030 ".* 65030";

-   The router combines each expression in the *as-path-group* together using a logical OR operation.

Summary
-------

In this chapter, you saw how the JUNOS software provides multiple methods for processing routing policies. We explored policy chains in depth and discovered how a policy subroutine works. We then looked at how to advertise a set of routes using a prefix list. Finally, we discussed the concept of a policy expression using logical Boolean operators. This complex system allows you the ultimate flexibility in constructing and advertising routes.

We concluded our chapter with a discussion of two BGP attributes, communities and AS Paths, and some methods of interacting with those attributes with routing policies. Both attributes are used as match criteria in a policy, and community values are altered as a policy action. Regular expressions are an integral part of locating routes, and we examined the construction of these expressions with respect to both communities and AS Paths.

Check List
----------

-   identify default processing of a policy chain
    -   left to right, ends with protocol default policy
-   policy subroutine is returning true/false, not accept/reject
-   logical evaluation of policy expression
-   prefix list is exact match
-   construct a community regular expression
-   construct an AS path regular expression
