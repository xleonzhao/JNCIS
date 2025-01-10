---
title: JNCIS Notes System
permalink: /JNCIS_Notes/System/
---

- [Misc](#misc)
- [Login](#login)
- [Service](#service)
- [Syslog](#syslog)
- [Chassis](#chassis)

Misc
----

no-panic-on-nmi: no kernel crash at RE memory error
default-address-selection: use lo0 as source address for locally generated packets
dump-on-panic: create a core file when system crash
backup-router: When the router is booting, the routing protocol process (rpd) is not running; therefore, the router has no static or default routes. To allow the router to boot and to ensure that the router is reachable over the network if the routing protocol process fails to start properly, you configure a backup router (running IP version 4 \[IPv4\] or IP version 6 \[IPv6\]), which is a router that is directly connected to the local router (that is, on the same subnet).

Login
-----

allow-commands: specify the (additional) operational mode commands that members of a login class can use (than whatever current class permits).
deny-commands: specify the operational mode commands that the user is denied permission to issue, even though the permissions set with the permissions statement would allow it.
Permission table:

<table>
<tbody>
<tr class="odd">
<td><p>Permission Flag</p></td>
<td><p>Description</p></td>
</tr>
<tr class="even">
<td><p>access</p></td>
<td><p>Can view the access configuration in configuration mode using the show configuration operational mode command.</p></td>
</tr>
<tr class="odd">
<td><p>access-control</p></td>
<td><p>Can view and configure access information at the [edit access] hierarchy level.</p></td>
</tr>
<tr class="even">
<td><p>admin</p></td>
<td><p>Can view user account information in configuration mode and with the show configuration command.</p></td>
</tr>
<tr class="odd">
<td><p>admin-control</p></td>
<td><p>Can view user accounts and configure them at the [edit system login] hierarchy level.</p></td>
</tr>
<tr class="even">
<td><p>all</p></td>
<td><p>Has all permissions.</p></td>
</tr>
<tr class="odd">
<td><p>clear</p></td>
<td><p>Can clear (delete) information learned from the network that is stored in various network databases using the clear commands.</p></td>
</tr>
<tr class="even">
<td><p>configure</p></td>
<td><p>Can enter configuration mode using the configure command.</p></td>
</tr>
<tr class="odd">
<td><p>control</p></td>
<td><p>Can perform all control-level operations—all operations configured with the -control permission flags.</p></td>
</tr>
<tr class="even">
<td><p>field</p></td>
<td><p>Reserved for field (debugging) support.</p></td>
</tr>
<tr class="odd">
<td><p>firewall</p></td>
<td><p>Can view the firewall filter configuration in configuration mode.</p></td>
</tr>
<tr class="even">
<td><p>firewall-control</p></td>
<td><p>Can view and configure firewall filter information at the [edit firewall] hierarchy level.</p></td>
</tr>
<tr class="odd">
<td><p>floppy</p></td>
<td><p>Can read from and write to the removable media.</p></td>
</tr>
<tr class="even">
<td><p>flow-tap</p></td>
<td><p>Can view the flow-tap configuration in configuration mode.</p></td>
</tr>
<tr class="odd">
<td><p>flow-tap control</p></td>
<td><p>Can view the flow-tap configuration in configuration mode and can configure flow-tap configuration information at the [edit services flow-tap] hierarchy level.</p></td>
</tr>
<tr class="even">
<td><p>flow-tap-operation</p></td>
<td><p>Can make flow-tap requests to the router. For example, a Dynamic Tasking Control Protocol (DTCP) client must authenticate itself to JUNOS as an administrative user. That account must have flow-tap-operation permission.</p>
<p>'''Note: '''flow-tap operation is not included in the all permission.</p></td>
</tr>
<tr class="odd">
<td><p>interface</p></td>
<td><p>Can view the interface configuration in configuration mode and with the show configuration operational mode command.</p></td>
</tr>
<tr class="even">
<td><p>interface-control</p></td>
<td><p>Can view the interface configuration in configuration mode and with the show configuration operational mode command.</p></td>
</tr>
<tr class="odd">
<td><p>maintenance</p></td>
<td><p>Can perform system maintenance, including starting a local shell on the router and becoming the superuser in the shell using the su root command, and can halt and reboot the router using the request system commands.</p></td>
</tr>
<tr class="even">
<td><p>network</p></td>
<td><p>Can access the network by entering the ping, SSH, telnet, and traceroute commands.</p></td>
</tr>
<tr class="odd">
<td><p>reset</p></td>
<td><p>Can restart software processes using the restart command and can configure whether software processes are enabled or disabled at the [edit system processes] hierarchy level.</p></td>
</tr>
<tr class="even">
<td><p>rollback</p></td>
<td><p>Can use the rollback command to return to a previously committed configuration other than the most recently committed one.</p></td>
</tr>
<tr class="odd">
<td><p>routing</p></td>
<td><p>Can view general routing, routing protocol, and routing policy configuration information in configuration and operational modes.</p></td>
</tr>
<tr class="even">
<td><p>routing-control</p></td>
<td><p>Can view general routing, routing protocol, and routing policy configuration information and configure general routing at the [edit routing-options] hierarchy level, routing protocols at the [edit protocols] hierarchy level, and routing policy at the [edit policy-options] hierarchy level.</p></td>
</tr>
<tr class="odd">
<td><p>secret</p></td>
<td><p>Can view passwords and other authentication keys in the configuration.</p></td>
</tr>
<tr class="even">
<td><p>secret-control</p></td>
<td><p>Can view passwords and other authentication keys in the configuration and can modify them in configuration mode.</p></td>
</tr>
<tr class="odd">
<td><p>security</p></td>
<td><p>Can view security configuration in configuration mode and with the show configuration operational mode command.</p></td>
</tr>
<tr class="even">
<td><p>security-control</p></td>
<td><p>Can view and configure security information at the [edit security] hierarchy level.</p></td>
</tr>
<tr class="odd">
<td><p>shell</p></td>
<td><p>Can start a local shell on the router by entering the start shell command.</p></td>
</tr>
<tr class="even">
<td><p>snmp</p></td>
<td><p>Can view Simple Network Management Protocol (SNMP) configuration information in configuration and operational modes.</p></td>
</tr>
<tr class="odd">
<td><p>snmp-control</p></td>
<td><p>Can view SNMP configuration information and modify SNMP configuration at the [edit snmp] hierarchy level.</p></td>
</tr>
<tr class="even">
<td><p>system</p></td>
<td><p>Can view system-level information in configuration and operational modes.</p></td>
</tr>
<tr class="odd">
<td><p>system-control</p></td>
<td><p>Can view system-level configuration information and configure it at the [edit system] hierarchy level.</p></td>
</tr>
<tr class="even">
<td><p>trace</p></td>
<td><p>Can view trace file settings in configuration and operational modes.</p></td>
</tr>
<tr class="odd">
<td><p>trace-control</p></td>
<td><p>Can view trace file settings and configure trace file properties.</p></td>
</tr>
<tr class="even">
<td><p>view</p></td>
<td><p>Can use various commands to display current systemwide, routing table, and protocol-specific values and statistics. Cannot view secret configuration.</p></td>
</tr>
</tbody>
</table>

Service
-------

connection-limit: limit—Maximum number of simultaneous connections (a value from 1 through 250). The default is 75.
rate-limit: limit—Maximum number of connection attempts accepted per minute (a value from 1 through 250). The default is 150.

Syslog
------

System Logging Facilities:

|                      |                                                                                                                                    |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Facility             | Type of Event or Error                                                                                                             |
| any                  | All (messages from all facilities)                                                                                                 |
| authorization        | Authentication and authorization attempts                                                                                          |
| change-log           | Changes to the JUNOS configuration                                                                                                 |
| conflict-log         | Specified configuration is invalid on the routing platform type                                                                    |
| daemon               | Actions performed or errors encountered by system processes                                                                        |
| dfc                  | Events related to dynamic flow capture                                                                                             |
| firewall             | Packet filtering actions performed by a firewall filter                                                                            |
| ftp                  | Actions performed or errors encountered by the FTP process                                                                         |
| interactive-commands | Commands issued at the JUNOS command-line interface (CLI) prompt or by a client application such as a JUNOScript or NETCONF client |
| kernel               | Actions performed or errors encountered by the JUNOS kernel                                                                        |
| pfe                  | Actions performed or errors encountered by the Packet Forwarding Engine                                                            |
| user                 | Actions performed or errors encountered by user-space processes                                                                    |

System Log Message Severity Levels: Messages from the facility that are rated at that level or higher are logged to the destination.

|                |                                                                                                                         |
|----------------|-------------------------------------------------------------------------------------------------------------------------|
| Severity Level | Description                                                                                                             |
| any            | Includes all severity levels                                                                                            |
| none           | Disables logging of the associated facility to a destination                                                            |
| emergency      | System panic or other condition that causes the routing platform to stop functioning                                    |
| alert          | Conditions that require immediate correction, such as a corrupted system database                                       |
| critical       | Critical conditions, such as hard drive errors                                                                          |
| error          | Error conditions that generally have less serious consequences than errors in the emergency, alert, and critical levels |
| warning        | Conditions that warrant monitoring                                                                                      |
| notice         | Conditions that are not errors but might warrant special handling                                                       |
| info           | Events or nonerror conditions of interest                                                                               |

Chassis
-------

no-source-route: Discard IP traffic that has loose or strict source-route constraints. Use this option when you want the router to use only the IP destination address on transit traffic for forwarding decisions.