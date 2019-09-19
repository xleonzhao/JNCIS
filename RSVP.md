---
title: JNCIS Notes MPLS RSVP
permalink: /JNCIS_Notes/MPLS/RSVP/
---

RSVP Overview
-------------

-   <http://www.ietf.org/rfc/rfc2205.txt>
-   <http://www.networksorcery.com/enp/protocol/rsvp.htm>

Originally designed to provide end-user hosts with the ability to reserve network resources for data traffic flows. Now the concept is extended to take routers as end points which transport traffic flows.

RSVP creates independent sessions to handle each data flow. A session is uniquely identified by (Session, Sender Template, SessionAttribute) objects in a Path message, and (Session, Filter Spec) objects in a Resv message.

After protocol finishes, each router along the path should have the
following information for packet forwarding
|                |                    |     |                |                    |
| -------------- | ------------------ | --- | -------------- | ------------------ |
| incoming label | incoming interface |     | outgoing label | outgoing interface |

### RSVP Common Header

                    0             1              2             3
             +-------------+-------------+-------------+-------------+
             | Vers | Flags|  Msg Type   |       RSVP Checksum       |
             +-------------+-------------+-------------+-------------+
             |  Send_TTL   | (Reserved)  |        RSVP Length        |
             +-------------+-------------+-------------+-------------+
             |                       Objects                         |
             +-------------+-------------+-------------+-------------+

             Msg Type: 8 bits
                  1 = Path
                  2 = Resv
                  3 = PathErr
                  4 = ResvErr
                  5 = PathTear
                  6 = ResvTear
                  7 = ResvConf

             Send_TTL: is IP_TTL, used to detect non-RSVP capable routers.

### RSVP message and objects

A RSVP message is a vechile which contains several Objects.

| RSVP1                                                       | Path | Resv | PathTear | ResvTear | PathErr | ResvErr | ResvConf |
| ----------------------------------------------------------- | ---- | ---- | -------- | -------- | ------- | ------- | -------- |
| Session                                                     | M    | M    | M        | M        | M       | M       | M        |
| Style                                                       |      | M    |          | M        |         | M       | M        |
| FlowSpec                                                    |      | O    |          | O        |         | O       | M        |
| FilterSpec                                                  |      | O    |          | M        |         | O       | M        |
| Sender Template                                             | M    |      | M        |          | M       |         |          |
| Sender Tspec                                                | M    |      | O        |          | M       |         |          |
| Adspec                                                      | O    |      | O        |          | O       |         |          |
| Error Spec                                                  |      |      |          |          | M       | M       | M        |
| Policy Data                                                 | O    | O    |          |          | O       | O       |          |
| Integrity                                                   | O    |      | O        | O        | O       | O       | O        |
| Scope                                                       |      | O    |          | O        |         | O       |          |
| ResvConf                                                    |      | O    |          |          |         |         | ?        |
| Time Values                                                 | M    | M    |          |          |         |         |          |
| RSVP Hop                                                    | M    | M    | M        | M        |         | M       |          |
| TE extension [RFC3209](http://www.ietf.org/rfc/rfc3209.txt) |      |      |          |          |         |         |          |
| Label Request                                               | M    |      |          |          |         |         |          |
| Explicit Route                                              | O    |      |          |          |         |         |          |
| Record Route                                                | O    | O    |          |          |         |         |          |
| Session Attribute                                           | O    |      |          |          |         |         |          |
| COS Flowspec                                                |      |      |          |          |         |         |          |
| Label                                                       |      | M    |          |          |         |         |          |

**M**: Mandatory; **O**: Optional.

### RSVP LSP

There are four classes of routers along a RSVP Label Switching Path (LSP), in an order from upstream to downstream:

* Ingress: the beginning of LSP.
* Transit: the intermediate router.
* Penultimate: the very next router to egress.
* Egress: the end of LSP.

#### LSP establishment process

1.  Ingress router (upstream) sends a *Path* message addressed to egress router (downstream).
    -   The *Path* message contains following TE extension objects:
        -   Session (mandatory)
        -   Explicit Route Object (optional)
        -   Record Route Object (optional)
        -   Session Attribute Object (optional)
    -   The Path message set IP Router-Alert option is set in IP header, which triggers each router along the path to examine the Path message.
2.  A transit or penultimate router examines the Path message. If everything is OK,
    1.  creates a new Path State Block (PSB), which contains
        -   <u>Session</u> (key field)
        -   <u>Sender Template</u> (key field)
        -   Sender Tspec
        -   Previous hop IP address from RSVP Hop object
        -   others (such as remaining IP TTL, Policy data and/or Adspec objects (optional), Non-RSVP flag (set if IP TTL is not equal to RSVP TTL)
    2.  replaces RSVP Hop object with its own outgoing interface address
    3.  adds its outgoing interface address into Record Route Object
    4.  forwards *Path* message further to next router downstream.

    -   If there is a problem in processing *Path* message, a *PathErr* is sent back hop by hop (addressed to previous hop's address) to ingress router.

3.  Egress router receives *Path* message. If everything OK,
    1.  allocates a label (and/or resources)
    2.  generates a *Resv* message addressed to previous hop (=RSVP Hop object in Path message), which contains the following TE objects:
        -   Session (mandatory)
        -   Label (mandatory)
        -   Style (mandatory)
        -   Record Route Object (optional)
    3.  sends Resv message hop by hop to ingress router.

    -   If there is a problem, it sends *ResvErr* hop by hop to ingress
        router.

4.  A transit or penultimate router receives the *Resv* message. If everything is OK,

    1.  creates a new Reservation State Block (RSB), which contains
        -   <u>Session</u> (key field)
        -   <u>RSVP Hop</u> (key field) (**=outgoing interface**)
        -   <u>Filter Spec List</u> (same content as Sender Template) (key field)
        -   Label object (**=outgoing label**)
        -   interface on which the reservation to be made (obtained by lookup PSB using Session + Filter Spec=Sender Template) (**=incoming interface**)
        -   Style
        -   others like Flowspec, Scope, Resv Confirm.
    2.  allocates a new label (and/or resources) and puts it into Label object (**=incoming label**)
    3.  replaces RSVP Hop object with its outgoing interface
    4.  adds its outgoing interface into Record Route Object
    5.  forwards *Resv* upstream to next router.

    -   Now it has forwarding table entry (**incoming label, incoming interface, outgoing label, outgoing interface**).
    -   If there is a problem in processing Resv message, a *ResvErr* is sent hop by hop towards egress router.

5.  Once ingress router receives *Resv* message, a LSP is established.

-   LSP is maintained via periodically sending *Path/Resv* message.

<!-- -->

-   LSP is tore down to free up resources
    -   A *PathTear* is generated by ingress or any router with timed out path state. PathTear will destroy any related data structure along the path and travel hop by hop to egress. Or,
    -   A *ResvTear* is generated by egress or any router with timed out reservation state. ResvTear will destroy any related data structure along the path and travel hop by hop to ingress.

RSVP Messages
-------------

### Path message

An intermediate router receivs a Path message:

`Apr 24 10:56:24 RSVP recv Path 192.168.16.1->192.168.32.1 Len=228 so-0/1/0.0`
<u>`Apr 24 10:56:24 Session7 Len 16 192.168.32.1(port/`**`tunnel`` ``ID`**` 44214) Proto 0`</u>
`Apr 24 10:56:24 Hop Len 12 10.222.1.1/0x084ec198`
`Apr 24 10:56:24 Time Len 8 30000 ms`
`Apr 24 10:56:24 SessionAttribute Len 28 Prio (7,0) flag 0x0 "Sherry-to-Char"`
<u>`Apr 24 10:56:24 Sender7 Len 12 192.168.16.1(port/`**`lsp`` ``ID`**` 1)`</u>
`Apr 24 10:56:24 Tspec Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`Apr 24 10:56:24 ADspec Len 48`
`Apr 24 10:56:24 SrcRoute Len 20 10.222.1.2 S 10.222.45.2 S`
`Apr 24 10:56:24 LabelRequest Len 8 EtherType 0x800`
`Apr 24 10:56:24 Properties Len 12 Primary path`
`Apr 24 10:56:24 RecRoute Len 20 10.222.1.1 10.222.29.1`

`* `**`LSP`` ``ID`**` and `**`Tunnel`` ``ID`**` uniquely defines a LSP.`

It forwards Path message further down:

`Apr 24 10:56:24 RSVP send Path 192.168.16.1->192.168.32.1 Len=228 so-0/1/2.0`
`Apr 24 10:56:24 Session7 Len 16 192.168.32.1(port/tunnel ID 44214) Proto 0`
<u>`Apr 24 10:56:24 Hop Len 12 10.222.45.1/0x08550264`</u>
`Apr 24 10:56:24 Time Len 8 30000 ms`
`Apr 24 10:56:24 SessionAttribute Len 28 Prio (7,0) flag 0x0 "Sherry-to-Char"`
`Apr 24 10:56:24 Sender7 Len 12 192.168.16.1(port/lsp ID 1)`
`Apr 24 10:56:24 Tspec Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`Apr 24 10:56:24 ADspec Len 48`
`Apr 24 10:56:24 SrcRoute Len 12 10.222.45.2 S`
`Apr 24 10:56:24 LabelRequest Len 8 EtherType 0x800`
`Apr 24 10:56:24 Properties Len 12 Primary path`
<u>`Apr 24 10:56:24 RecRoute Len 28 10.222.45.1 10.222.1.1 10.222.29.1`</u>

*Session7* indicates it is a Session object with C-Type=7.

### Resv message

An intermediate router receivs a Resv message:

`Apr 24 10:56:24 RSVP recv Resv 10.222.45.2->10.222.45.1 Len=120 so-0/1/2.0`
<u>`Apr 24 10:56:24 Session7 Len 16 192.168.32.1(port/tunnel ID 44214) Proto 0`</u>
`Apr 24 10:56:24 Hop Len 12 10.222.45.2/0x08550264`
`Apr 24 10:56:24 Time Len 8 30000 ms`
`Apr 24 10:56:24 Style Len 8 FF`
`Apr 24 10:56:24 Flow Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
<u>`Apr 24 10:56:24 Filter7 Len 12 192.168.16.1(port/lsp ID 1)`</u>
<u>`Apr 24 10:56:24 Label Len 8 3`</u>
`Apr 24 10:56:24 RecRoute Len 12 10.222.45.2`

Then it generates a new Resv message:

`Apr 24 10:56:24 RSVP send Resv 10.222.1.2->10.222.1.1 Len=128 so-0/1/0.0`
`Apr 24 10:56:24 Session7 Len 16 192.168.32.1(port/tunnel ID 44214) Proto 0`
<u>`Apr 24 10:56:24 Hop Len 12 10.222.1.2/0x084ec198`</u>
`Apr 24 10:56:24 Time Len 8 30000 ms`
`Apr 24 10:56:24 Style Len 8 FF`
`Apr 24 10:56:24 Flow Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`Apr 24 10:56:24 Filter7 Len 12 192.168.16.1(port/lsp ID 1)`
<u>`Apr 24 10:56:24 Label Len 8 100001`</u>
<u>`Apr 24 10:56:24 RecRoute Len 20 10.222.1.2 10.222.45.2`</u>

### PathErr message

A PathErr message is sent to ingress router hop by hop to notify network
problems, addressed to the interface address of previous hop upstream.
The PathErr message will not destroy any established soft-state data
structures along the path.

`Apr 25 07:06:00 RSVP send PathErr 10.222.1.2->10.222.1.1 Len=84 so-0/1/0.0`
`Apr 25 07:06:00 Session7 Len 16 192.168.32.1(port/tunnel ID 44222) Proto 0`
<u>`Apr 25 07:06:00 Error Len 12 code 24 value 2 flag 0 by 10.222.1.2`</u>
`Apr 25 07:06:00 Sender7 Len 12 192.168.16.1(port/lsp ID 1)`
`Apr 25 07:06:00 Tspec Len 36 rate 15Mbps size 15Mbps peak Infbps m 20 M 1500`

### ResvErr message

A ResvErr message is sent to egress router downstream to notify a
problem, addressed to the interface address of next hop downstream. It
will not destroy any established soft states along the path.

`Apr 25 10:35:37 RSVP recv ResvErr 10.222.29.1->10.222.29.100 Len=104 ge-0/2/0.0`
`Apr 25 10:35:37 Session7 Len 16 192.168.32.1(port/tunnel ID 44234) Proto 0`
`Apr 25 10:35:37 Hop Len 12 10.222.29.1/0x084fb0cc`
<u>`Apr 25 10:35:37 Error Len 12 code 4 value 0 flag 0 by 10.222.29.1`</u>
`Apr 25 10:35:37 Style Len 8 FF`
`Apr 25 10:35:37 Flow Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`Apr 25 10:35:37 Filter7 Len 12 192.168.16.1(port/lsp ID 1)`

### PathTear message

PathTear message destroys soft states, PSB and RSB, along the path. It
is addressed to egress router and forwarded to downstream.

`Apr 25 09:56:54 RSVP send PathTear 192.168.16.1->192.168.32.1 Len=84 so-0/1/2.0`
<u>`Apr 25 09:56:54 Session7 Len 16 192.168.32.1(port/tunnel ID 44230) Proto 0`</u>
`Apr 25 09:56:54 Hop Len 12 10.222.45.1/0x08550264`
<u>`Apr 25 09:56:54 Sender7 Len 12 192.168.16.1(port/lsp ID 1)`</u>
`Apr 25 09:56:54 Tspec Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`

### ResvTear message

ResvTear message destroys soft states, RSB , along the path. It is
addressed to previous hop upstream and forwarded to ingress router.

`Apr 25 09:54:57 RSVP send ResvTear 10.222.29.2->10.222.29.1 Len=56 ge-0/2/0.0`
<u>`Apr 25 09:54:57 Session7 Len 16 192.168.32.1(port/tunnel ID 44230) Proto 0`</u>
`Apr 25 09:54:57 Hop Len 12 10.222.29.2/0x084fb0cc`
`Apr 25 09:54:57 Style Len 8 FF`
`Apr 25 09:54:57 Filter7 Len 12 192.168.16.1(port/lsp ID 1)`

RSVP Objects
------------

                    0             1              2             3
             +-------------+-------------+-------------+-------------+
             |       Length (bytes)      |  Class-Num  |   C-Type    |
             +-------------+-------------+-------------+-------------+
             |                                                       |
             //                  (Object contents)                   //
             |                                                       |
             +-------------+-------------+-------------+-------------+

Class Number: The values assigned to the RSVP classes are encoded using one of three bit patterns. These patterns are used when the received class value is not recognized, or unknown. The bit patterns (where b is a 0 or 1) are:

:\* 0bbbbbbb: The receipt of an unknown class value using this bit
pattern causes the local router to reject the message and return an
error to the originating node.

:\* 10bbbbbb: The receipt of an unknown class value using this bit
pattern causes the local router to ignore the object. It is not
forwarded to any RSVP neighbors, and no error messages are generated.

:\* 11bbbbbb: The receipt of an unknown class value using this bit
pattern causes the local router to also ignore the object. In this case,
however, the router forwards the object to its neighbors without
examining or modifying it in any way.

### LSP-Tunnel-IPv4 Session Object

Again, session is identified by (destination address, destination
port/LSP tunnel ID, protocol)

Class = 1, C-Type = 7

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                   IPv4 tunnel end point address               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |  MUST be zero                 |      Tunnel ID                |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                       Extended Tunnel ID                      |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

IPv4 Tunnel Endpoint Address: This field displays the IPv4 address of the LSP egress router.

<!-- -->

Tunnel ID: This field contains a unique value generated by the ingress router of the LSP. This helps to distinguish the particular LSP from other paths originating at the same ingress router. The selected value remains constant throughout the life of the LSP.

<!-- -->

Extended Tunnel ID: An ingress router places its IPv4 address in this field to further identify the session. This allows the LSP to be uniquely identified by the ingress-egress router addresses. Although the use of this field is not required, the JUNOS software places the ingress address here by default.

### IPv4 RSVP-Hop Object

IP interface address of the neighboring RSVP router.

Class = 3, C-Type = 1

               +-------------+-------------+-------------+-------------+
               |             IPv4 Next/Previous Hop Address            |
               +-------------+-------------+-------------+-------------+
               |                 Logical Interface Handle              |
               +-------------+-------------+-------------+-------------+

Logical Interface Handle: Each interface that transmits and receives RSVP messages is assigned a unique 32-bit value known as the logical interface handle. The sending router, *R1*, populates this field, which contains the unique ID value. The receiving router, *R2*, stores this value in its PSB, and later to place this value into its Resv message which allow *R1* to associate the received Resv message with correct interface.

### Time-Values Object

This object defines a time period within which the receiving router
should expect to receive the same type message from sender. If not,
receiving router will timeout the soft state information. JUNOS by
default uses 30,000 (30 seconds).

Class = 5, C-Type = 1

               +-------------+-------------+-------------+-------------+
               |                   Refresh Period R                    |
               +-------------+-------------+-------------+-------------+

### Error Spec Object

Whoever first detects an error in RSVP processing will send this object,
so other routers know where and what the failure is. Complete error code
is listed in [1](http://www.ietf.org/rfc/rfc2205.txt).

Class = 6, C-Type = 1

               +-------------+-------------+-------------+-------------+
               |            IPv4 Error Node Address (4 bytes)          |
               +-------------+-------------+-------------+-------------+
               |    Flags    |  Error Code |        Error Value        |
               +-------------+-------------+-------------+-------------+

### Style Object

There are three styles:

Fixed Filter (FF): The resource is reserved for a particular (Session, Sender-Template) pair, not shareable with others.
Shared Explicit (SE): Multiple Sender-Template objects that share a common Session object are allowed to use the same resource.
Wildcard Filter (WF): A set of resources can be shared among all possible senders. JUNOS doesn't support it.

Class = 8, C-Type = 1

               +-------------+-------------+-------------+-------------+
               |   Flags     |              Option Vector              |
               +-------------+-------------+-------------+-------------+

Flags: no flag is defined so far.
Option Vector: first 19 bits are reserved, all 0s.

:\* 01010—Fixed filter (FF)

:\* 10001—Wildcard filter (WF)

:\* 10010—Shared explicit (SE)

### LSP-Tunnel-IPv4 Filter-Spec Object

In a Resv message to uniquely identify the sender of the LSP, useful for
SE style.

Class = 10, C-Type = 7 (same format as LSP-Tunnel-IPv4 Sender Template
Object)

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                   IPv4 tunnel sender address                  |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |  MUST be zero                 |            LSP ID             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

LSP ID: This field contains a unique value generated by the ingress router of the LSP. This helps to distinguish the particular sender and its resources from other paths originating at the same ingress router for the same RSVP session.

### LSP-Tunnel-IPv4 Sender Template Object

In a Path message to uniquely identify the sender.

Class = 11, C-Type = 7 (same format as LSP-Tunnel-IPv4 Filter Spec
Object)

### Integrated Services Sender-Tspec Object

Like the IntServ Flowspec object, the format and use of the object
fields was designed for a controlled load environment where average and
peak data rates could be defined. Defined in [RFC
2210](http://www.networksorcery.com/enp/rfc/rfc2210.txt).

            31           24 23           16 15            8 7             0
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       1   | Ver=0 |    Reserved           |   Tspec Length=7              |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       2   | Service #=1   |0| Reserved    |   Data Length=6               |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       3   | Para. ID=127  | Para. Flag=0  |   Parameter Length=5          |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       4   |  Token Bucket Rate [r] (32-bit IEEE floating point number)    |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       5   |  Token Bucket Size [b] (32-bit IEEE floating point number)    |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       6   |  Peak Data Rate [p] (32-bit IEEE floating point number)       |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       7   |  Minimum Policed Unit [m] (32-bit integer)                    |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       8   |  Maximum Packet Size [M]  (32-bit integer)                    |
           +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Token Bucket Rate: This field displays the bandwidth reservation request for the LSP, if one was configured. A value of 0x00000000 in this field represents a reservation with no bandwidth requirement.
Token Bucket Size: This field also displays the bandwidth reservation request for the LSP. Again, a value of 0x00000000 means that no bandwidth was requested.
Peak Data Rate: This field was originally designed to display the maximum load of data able to be sent through the reservation. This field is not used by the JUNOS software, and a constant value of 0x7f800000 is placed in this field, which represents an infinite peak bandwidth limit.
Minimum Policed Unit: This field displays the smallest packet size supported through the LSP. The router treats all packets smaller than 20 bytes as if they were 20 bytes in length.
Maximum Packet Size: This field displays the largest packet size supported through the LSP. The router treats all packets larger than 1500 bytes as if they were 1500 bytes in length.

### Label Object

In Resv messages and advertises an MPLS label value upstream to the
neighboring router.

class = 16, C_Type = 1

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           Label Value                         |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

### Label Request Object

Class = 19, C_Type = 1

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |           Reserved            |             L3PID             |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Layer 3 Protocol ID (L3PID): This field contains information about the Layer 3 protocol carried in the LSP. The standard EtherType value of 0x0800 is used in this field to represent IPv4.

### Explicit Route Object

In ERO in a Path mesage, ingress router can specify a explicit route to
egress. An intermediate router checks if the first address in ERO is its
own local address. If yes, it removes the first address from ERO and
continues forward the Path message. If not and the first address is
strict, a PathErr will be generated. JUNOS only support IPv4 prefix, but
RFC supports IPv6 prefix and AS path.

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |L|    Type     |     Length    | IPv4 address (4 bytes)        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | IPv4 address (continued)      | Prefix Length |      Resvd    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

L: L=1: loose; L=0: strict.
Type: 0x01 - IPv4 prefix
Prefix Length: by default, it is 32.

### Record Route Object

The Record Route object (RRO) may be contained in either a Path or a
Resv message. The RRO contains subobjects that describe the nodes
through which the message has passed.

Class = 21, C_Type = 1

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |      Type     |     Length    | IPv4 address (4 bytes)        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | IPv4 address (continued)      | Prefix Length |      Flags    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Flags: This field contains flags that alert other routers in the network about the capabilities or conditions of the local node. Currently, four flag values are defined:

:\* Bit 0 - Local protection available. This flag bit, 0x01, is set to
indicate that the next downstream link from the router is protected by a
local repair mechanism, such as fast reroute link protection. The flag
is set only when the Session Attribute object requests that link
protection be enabled for the LSP.

:\* Bit 1 - Local protection in use. This flag bit, 0x02, is set to
indicate that the local router is actively using a repair mechanism to
maintain the LSP due to some outage condition.

:\* Bit 2 This flag bit, 0x04, is set to indicate that the local router
has a backup path enabled that provides the same bandwidth reservation
guarantees as established for an LSP protected through a local
protection scheme.

:\* Bit 3 This flag bit, 0x08, is set to indicate that the next
downstream node as well as the downstream link is protected by a local
repair mechanism, such as fast reroute node protection. This flag is set
only when the next downstream node is protected against failure and the
Session Attribute object requests that node protection be enabled for
the LSP.

### Detour Object

The Detour object is contained in a Path message, which establishes a
fast reroute detour in the network. This Path message inherits the
Session, Sender-Template, and Session Attribute objects from the
established LSP. This allows the detour path to be associated with the
main RSVP session on all possible routers. The Detour object itself
lists the originating node of the Path message, the point of local
repair, and the address of the downstream node being protected against
failures. These ID values may be repeated in a single Detour object when
an RSVP router merges multiple detour paths as they head toward the
egress router.

Class-Num = 63, C-Type = 7

                0             1              2             3
           +-------------+-------------+-------------+-------------+
           |              Local Repair Node ID 1                   |
           +-------------+-------------+-------------+-------------+
           |                    Avoid_Node_ID  1                   |
           +-------------+-------------+-------------+-------------+
          //                        ....                          //
           +-------------+-------------+-------------+-------------+
           |              Local Repair Node ID n                   |
           +-------------+-------------+-------------+-------------+
           |                    Avoid_Node_ID  n                   |
           +-------------+-------------+-------------+-------------+

Local Repair Node ID: This field encodes the address of the router creating the detour path through the network. While any possible address on the local router may be used, the JUNOS software places the address of the outgoing interface in this field.
Avoid Node ID: This field encodes the router ID of the downstream node that is being protected against failures.

Ingress router sent out two Path messages.

`May 12 17:41:01 RSVP send Path 192.168.16.1->192.168.32.1 Len=244 ge-0/2/0.0`
<u>`May 12 17:41:01 Session7 Len 16 192.168.32.1(port/tunnel ID 24027) Proto 0`</u>
`May 12 17:41:01 Hop Len 12 10.222.29.1/0x08528264`
`May 12 17:41:01 Time Len 8 30000 ms`
<u>`May 12 17:41:01 SessionAttribute Len 24 Prio (7,0) flag 0x0 "Sherry-to-Char"`</u>
<u>`May 12 17:41:01 Sender7 Len 12 192.168.16.1(port/lsp ID 1)`</u>
`May 12 17:41:01 Tspec Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`May 12 17:41:01 ADspec Len 48`
<u>`May 12 17:41:01 SrcRoute Len 28 10.222.29.2 S 10.222.1.2 S 10.222.45.2 S`</u>
`May 12 17:41:01 LabelRequest Len 8 EtherType 0x800`
`May 12 17:41:01 Properties Len 12 Primary path`
`May 12 17:41:01 RecRoute Len 12 10.222.29.1`
`May 12 17:41:01 FastReroute Len 20 Prio(7,0) Hop 6 BW 0bps`
`May 12 17:41:01   Include 0x00000000 Exclude 0x00000000`

The second Path message contain detour routes in case the router next to
ingress router failed. This Path message inherited Session,
SessionAttribute and Sender Template objects from the first Path
message.

`May 12 17:41:04 RSVP send Path 192.168.16.1->192.168.32.1 Len=236 at-0/1/0.0`
<u>`May 12 17:41:04 Session7 Len 16 192.168.32.1(port/tunnel ID 24027) Proto 0`</u>
`May 12 17:41:04 Hop Len 12 10.222.28.1/0x085280cc`
`May 12 17:41:04 Time Len 8 30000 ms`
<u>`May 12 17:41:04 SessionAttribute Len 24 Prio (7,0) flag 0x0 "Sherry-to-Char"`</u>
<u>`May 12 17:41:04 Sender7 Len 12 192.168.16.1(port/lsp ID 1)`</u>
`May 12 17:41:04 Tspec Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`May 12 17:41:04 ADspec Len 48`
<u>`May 12 17:41:04 SrcRoute Len 28 10.222.28.2 S 10.222.4.2 S 10.222.44.2 S`</u>
`May 12 17:41:04 LabelRequest Len 8 EtherType 0x800`
`May 12 17:41:04 Properties Len 12 Primary path`
`May 12 17:41:04 RecRoute Len 12 10.222.28.1`
<u>`May 12 17:41:04 Detour Len 12 Branch from 10.222.28.1 to avoid 192.168.20.1`</u>

Some router in detour route can merge those detour objects.

`May 12 17:41:01 RSVP send Path 192.168.16.1->192.168.32.1 Len=260 so-0/1/0.0`
<u>`May 12 17:41:01 Session7 Len 16 192.168.32.1(port/tunnel ID 24027) Proto 0`</u>
`May 12 17:41:01 Hop Len 12 10.222.44.1/0x085a7198`
`May 12 17:41:01 Time Len 8 30000 ms`
<u>`May 12 17:41:01 SessionAttribute Len 24 Prio (7,0) flag 0x0 "Sherry-to-Char"`</u>
<u>`May 12 17:41:01 Sender7 Len 12 192.168.16.1(port/lsp ID 1)`</u>
`May 12 17:41:01 Tspec Len 36 rate 0bps size 0bps peak Infbps m 20 M 1500`
`May 12 17:41:01 ADspec Len 48`
`May 12 17:41:01 SrcRoute Len 12 10.222.44.2 S`
`May 12 17:41:01 LabelRequest Len 8 EtherType 0x800`
`May 12 17:41:01 Properties Len 12 Primary path`
`May 12 17:41:01 RecRoute Len 36 10.222.44.1 10.222.46.2 10.222.1.1 10.222.29.1`
<u>`May 12 17:41:01 Detour Len 28 Branch from 10.222.46.2 to avoid 192.168.32.1`
`May 12 17:41:01   Branch from 10.222.28.1 to avoid 192.168.20.1`
`May 12 17:41:01   Branch from 10.222.30.2 to avoid 192.168.40.1`</u>

### Fast Reroute Object

The Fast Reroute object is contained in Path messages sent along the
path of an established LSP. It alerts all downstream routers that the
ingress router desires protection along the LSP’s path. Each router
along the LSP, with the exception of the egress, then creates a detour
path around the next downstream node using the Detour object.
Information within the Fast Reroute object allows each of the routers to
consult a local traffic engineering database for calculating a path to
the egress router. This information includes a bandwidth reservation, a
hop count, LSP priority values, and administrative group knowledge.

Class-Num = 205, C-Type = 1

                 0             1             2             3
          +-------------+-------------+-------------+-------------+
          |       Length (bytes)      |  Class-Num  |   C-Type    |
          +-------------+-------------+-------------+-------------+
          | Setup Prio  | Hold Prio   | Hop-limit   |    Flags    |
          +-------------+-------------+-------------+-------------+
          |                  Bandwidth                            |
          +-------------+-------------+-------------+-------------+
          |                  Include-any                          |
          +-------------+-------------+-------------+-------------+
          |                  Exclude-any                          |
          +-------------+-------------+-------------+-------------+
          |                  Include-all                          |
          +-------------+-------------+-------------+-------------+

Setup Priority: This field contains the priority of the LSP used for assigning resources during its establishment in the network. Possible values range from 0 through 7, with 0 representing the strongest priority value and 7 the weakest priority value. The JUNOS software uses a setup priority value of 7 by default.
Hold Priority: This field contains the priority of the LSP used for maintaining resources after becoming established in the network. Possible values range from 0 through 7, with 0 representing the strongest priority value and 7 the weakest priority value. The JUNOS software uses a hold priority value of 0 by default.
Hop Limit: This field displays the total number of transit hops a detour path may take through the network, excluding the local repair node and any router performing a detour merge operation. For example, a hop limit value of 2 means that the detour can leave the local repair node and transit two other routers before being merged or reaching the egress router.
Bandwidth: When populated, this field displays the bandwidth reservation (in bytes per second) that should be performed for all detour paths. By default, the JUNOS software places a value of 0 in this field. You may alter this default by configuring a bandwidth within the fast-reroute definition of the LSP.
Include Any: This field contains information pertaining to network links that are assigned a particular administrative group. When a group value is placed in this field, each network link along the detour path must be assigned to that group. A value of 0 in this field means that no group values are required and that all network links may be used.
Exclude Any: This field contains information pertaining to network links that are assigned a particular administrative group. When a group value is placed in this field, each network link along the detour path must not be assigned to that group. A value of 0 in this field means that any network link may be used.

### LSP-Tunnel Session Attribute Object

class = 207, C-Type = 7

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |   Setup Prio  | Holding Prio  |     Flags     |  Name Length  |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                                                               |
       //          Session Name      (NULL padded display string)      //
       |                                                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Setup Priority: This field contains the priority of the LSP used for assigning resources during its establishment in the network. Possible values range from 0 through 7, with 0 representing the strongest priority value and 7 the weakest priority value. The JUNOS software uses a setup priority value of 7 by default.
Hold Priority: This field contains the priority of the LSP used for maintaining resources after becoming established in the network. Possible values range from 0 through 7, with 0 representing the strongest priority value and 7 the weakest priority value. The JUNOS software uses a hold priority value of 0 by default.
Flags: This field contains flags that alert other routers in the network about the capabilities of the LSP and its resource reservations. Currently, five flag values are defined:

:\* Bit 0 This flag bit, 0x01, is set to permit downstream routers to
use a local repair mechanism, such as fast reroute link protection,
allowing transit routers to alter the explicit route of the LSP.

:\* Bit 1 This flag bit, 0x02, is set to request that label recording be
performed along the LSP path. This means that each downstream node
should place its assigned label into the Record Route object in the Resv
message.

:\* Bit 2 This flag bit, 0x04, is set to indicate that the egress router
should use the Shared Explicit reservation style for the LSP. This
allows the ingress router to reroute the primary path of the LSP without
first releasing the LSP’s resources.

:\* Bit 3 This flag bit, 0x08, is set to indicate that each router along
the LSP should reserve bandwidth for its fast reroute detour path. The
detour paths in the network use this bit value only when the Fast
Reroute object is omitted from the Path message created by the ingress
router.

:\* Bit 4 This flag bit, 0x10, is set to permit downstream routers to
use a node repair mechanism, such as fast reroute node protection. Each
router should then calculate a detour path that protects the LSP from a
failure of the next downstream node.

RSVP Hello Extension
--------------------

For the purpose of fast detection of immediate neighboring node failure.
The method is to advertise a (random) number to the neighbor and to
expect same number will be echoed back. If not, something bad may have
happened. Two objects are used for this purpose.

### Hello Request Object

Class = HELLO Class, C_Type = 1

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                         Src_Instance                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                         Dst_Instance                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Src_Instance: a 32 bit value that represents the sender's instance. The advertiser maintains a per neighbor representation/value. This value MUST change when the sender is reset, when the node reboots, or when communication is lost to the neighboring node and otherwise remains the same. This field MUST NOT be set to zero.
Dst_Instance: The most recently received Src_Instance value received from the neighbor. This field MUST be set to zero (0) when no value has ever been seen from the neighbor.

### Hello Ack Object

Class = HELLO Class, C_Type = 2. Same format as Hello Request Object

`Received RSVP Hello Message:`
`       RSVPv1 Hello Message (20), Flags: [none], length: 32, ttl: 1, checksum: 0x60df`
`         Hello Object (22) Flags: [reject if unknown], Class-Type: Hello Request (1), length: 12`
`           Source Instance: `**`0xa43ae452`**`, Destination Instance: `**`0x31a05043`**
`           0x0000: a43a e452 31a0 5043 `
`         Restart Capability Object (131) Flags: [ignore silently if unknown], Class-Type: IPv4 (1), length: 12`
`           Restart  Time: 60000ms, Recovery Time: 0ms`
`           0x0000: 0000 ea60 0000 0000 `
`Echoed back RSVP Hello Message`
`       RSVPv1 Hello Message (20), Flags: [none], length: 32, ttl: 1, checksum: 0x4b3f`
`         Hello Object (22) Flags: [reject if unknown], Class-Type: Hello Ack (2), length: 12`
`           Source Instance: `**`0x31a05043`**`, Destination Instance: `**`0xa43ae452`**
`           0x0000: 31a0 5043 a43a e452 `
`         Restart Capability Object (131) Flags: [ignore silently if unknown], Class-Type: IPv4 (1), length: 12`
`           Restart  Time: 0ms, Recovery Time: 0ms`
`           0x0000: 0000 0000 0000 0000`
