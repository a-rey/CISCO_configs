# CISCO CCNA Routing & Switching (ICND1 & 2) Configuration Guide

<!-- ToC generated using https://imthenachoman.github.io/nGitHubTOC/ -->

- [CISCO CCNA Routing & Switching (ICND1 & 2) Configuration Guide](#cisco-ccna-routing--switching-icnd1--2-configuration-guide)
  - [VLANs](#vlans)
    - [Voice VLAN](#voice-vlan)
    - [Troubleshooting](#troubleshooting)
  - [Trunks](#trunks)
    - [Dynamic Trunking Protocol](#dynamic-trunking-protocol)
    - [Troubleshooting](#troubleshooting-1)
  - [VLAN Trunking Protocol](#vlan-trunking-protocol)
    - [Troubleshooting](#troubleshooting-2)
  - [Spanning Tree Protocol](#spanning-tree-protocol)
    - [PortFast](#portfast)
    - [BPDU Guard](#bpdu-guard)
    - [Troubleshooting](#troubleshooting-3)
  - [EtherChannel (Layer 2)](#etherchannel-layer-2)
    - [PAgP](#pagp)
    - [LACP](#lacp)
    - [Troubleshooting](#troubleshooting-4)
  - [OSPF (v2)](#ospf-v2)
    - [Cost](#cost)
    - [IPv6 (OSPFv3)](#ipv6-ospfv3)
    - [Troubleshooting](#troubleshooting-5)
  - [EIGRP](#eigrp)
    - [Metric](#metric)
    - [EIGRP (IPv6)](#eigrp-ipv6)
    - [Troubleshooting](#troubleshooting-6)
  - [BGP](#bgp)
    - [Troubleshooting](#troubleshooting-7)
  - [HDLC](#hdlc)
    - [Troubleshooting](#troubleshooting-8)
  - [PPP](#ppp)
    - [PAP](#pap)
    - [CHAP](#chap)
    - [MLPPP](#mlppp)
    - [PPPoE](#pppoe)
    - [Troubleshooting](#troubleshooting-9)
  - [GRE Tunnels](#gre-tunnels)
    - [Troubleshooting](#troubleshooting-10)
  - [ACLs](#acls)
    - [Standard](#standard)
    - [Extended](#extended)
    - [IPv6](#ipv6)
    - [Troubleshooting](#troubleshooting-11)
  - [Inter-VLAN Routing](#inter-vlan-routing)
    - [Layer 3 Switch](#layer-3-switch)
    - [Router on a Stick](#router-on-a-stick)
    - [Troubleshooting](#troubleshooting-12)
  - [HSRP](#hsrp)
    - [Troubleshooting](#troubleshooting-13)
  - [SNMP](#snmp)
    - [Version 1](#version-1)
    - [Version 2c](#version-2c)
    - [Version 3](#version-3)
    - [Troubleshooting](#troubleshooting-14)
  - [IP SLA](#ip-sla)
    - [Troubleshooting](#troubleshooting-15)
  - [SPAN](#span)
    - [Local](#local)
    - [Remote](#remote)
    - [Troubleshooting](#troubleshooting-16)

## VLANs

Define on an interface:
- `Switch(config-if)#switchport access vlan <vlan>`
- _All_ interfaces are in VLAN1 (the default VLAN) by default

Define globally:
- `Switch(config)#vlan <vlan>`
- `Switch(config)#name <vlan-name>`
- `Switch(config-vlan)#exit`
  - VLAN not created until exiting VLAN configuration mode

### Voice VLAN

Define on an interface:
- `Switch(config-if)#switchport voice vlan <num|none|untagged|dot1p>`
  - Must have CDP enabled on port
  - `num` is the numerical VLAN ID

### Troubleshooting

Issues:
1. VLAN does not exist?
   1. Did you exit VLAN configuration mode?

Example troubleshooting output:

```
Switch#show vlan summary
Number of existing VLANs          : 3
Number of existing VTP VLANs      : 3
Number of existing extended VLANs : 0
```

```
Switch#show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    fa0/2, fa0/3, fa0/4, fa0/5
                                                fa0/6, fa0/7, fa0/8, fa0/9
                                                fa0/10, fa0/11, fa0/12, fa0/13
                                                fa0/14, fa0/15, fa0/16, fa0/17
                                                fa0/18, fa0/19, fa0/20, fa0/21
                                                fa0/22, fa0/23, fa0/24, gi0/1
                                                gi0/2
10   VLAN0010                         active    fa0/1
11   VLAN0011                         active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

```
Switch#show vlan id 11
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
11   VLAN0011                         active
VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
11   enet  100011     1500  -      -      -        -    -        0      0
Remote SPAN VLANs
------------------------------------------------------------------------------
Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
Switch#
```

```
Switch#show mac address-table dynamic
          Mac Address Table
-------------------------------------------
Vlan  Mac Address     Type     Ports
----  -----------     ----     -----
12    0200.1111.1111  dynamic  Fa0/11
12    0200.2222.2222  dynamic  Gi0/1
11    0200.3333.3333  dynamic  Gi0/1
11    0200.4444.4444  dynamic  Gi0/1
```

## Trunks

Define on an interface:
- `Switch(config-if)#switchport mode trunk`
- `Switch(config-if)#switchport trunk encapsulation <dot1q|isl|nonegotiate>`
- `Switch(config-if)#switchport trunk allowed vlan <add|all|except|remove> <vlan>`
- `Switch(config-if)#switchport trunk native vlan <vlan>`

### Dynamic Trunking Protocol

Define on an interface:
- `Switch(config-if)#switchport mode <dynamic auto|dynamic desirable>`

| Admin Mode          | Access      | Dynamic Auto | Trunk       | Dynamic Desirable |
| ------------------- | ----------- | ------------ | ----------- | ----------------- |
| `access`            | Access      | Access       | !!! BAD !!! | Access            |
| `dynamic auto`      | Access      | Access       | Trunk       | Trunk             |
| `trunk`             | !!! BAD !!! | Trunk        | Trunk       | Trunk             |
| `dynamic desirable` | Access      | Trunk        | Trunk       | Trunk             |

Disable all auto negotiation (trunk protocol negotiation **and** operational mode):
- `Switch(config-if)#switchport nonegotiate`

### Troubleshooting

Common Issues:
1. VLAN not allowed on trunk?
2. Bad DTP paramaters (eg `auto`&`auto` or `access`&`desirable`)?
3. Native VLANs don't match?
4. Incorrect encapsulation (ISL vs 802.1Q)?

Example troubleshooting output:

```
Switch#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/2       on           802.1q         not-trunking   1
Port        Vlans allowed on trunk
Fa0/2       1-4094
Port        Vlans allowed and active in management domain
Fa0/2       1,10-12
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/2       1,10-11
```

```
Switch#show interfaces fastEthernet 0/2 switchport
Name: FastEthernet0/2
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Negotiation of Trunking: true
Access Mode VLAN: none
Trunking Native Mode VLAN: 1 (default)
Trunking VLANs Enabled:
Trunking VLANs Active: active
Priority for untagged frames: 0
Override vlan tag priority: FALSE
Voice VLAN: none
Appliance trust: none
```

## VLAN Trunking Protocol

Set VTP domain/password:
- `Switch(config)#vtp domain <domain-name>`
- `Switch(config)#vtp password <password>`
- Domain and password must match (case sensitive) on **all** devices in VTP domain
- Both are unset by default

Set VTP mode:
- `Switch(config)#vtp mode <server|client|transparent|off>`
  - `server` mode is the default

| Function                                 | `server` | `client` | `transparent` | `off` |
| ---------------------------------------- | -------- | -------- | ------------- | ----- |
| Only sends VTP messages on trunks        | Y        | Y        | Y             | N     |
| Allows VLAN database changes             | Y        | N        | Y             | Y     |
| Can use standard range VLANs (1-1005)    | Y        | Y        | Y             | Y     |
| Can use extended range VLANs (1006-4095) | N        | N        | Y             | Y     |

Set VTP version:
- `Switch(config)#vtp version <1|2|3>`
  - Version 1 is the default

Enable VTP pruning:
- `Switch(config)#vtp pruning`
  - Disabled by default

### Troubleshooting

Common Issues:
1. Passwords don't match (check MD5 digest)?
2. VTP domain names don't match?
   1. These are case sensitive!
3. VTP operating modes don't match?
4. VLANs not syncing?
   1. Check revision number?
      1. Highest revision number wins!
   2. Check VTP pruning?
      1. Maybe the VLANs are not supposed to be there in the first place...

Example troubleshooting output:

```
Switch#show vtp status

VTP Version capable             : 1 to 3
VTP Version running             : 3
Configuration Revision          : 0
Maximum VLANs supported locally : 255
Number of existing VLANs        : 7
VTP Operating Mode              : Server
VTP Domain Name                 : test-domain-name
VTP Pruning Mode                : Enable
VTP V2 Mode                     : Disabled
VTP Traps Generation            : Disabled
MD5 digest                      : ddd4 ad64 4a9f a191 96a4 e053 b433
Configuration last modified by 0.0.0.0 at 1-30-2020 19:40:05
```

## Spanning Tree Protocol

Manually specifying root/secondary switch in a given VLAN **or** with a manual priority:
- `Switch(config)#spanning-tree vlan <vlan> root <primary|secondary>`
  - `root`: priority will be 24576 **or** the next lowest multiple of 4096 if 24576 is not low enough to become root _now_
  - `secondary`: priority will be 28672 
- `Switch(config)#spanning-tree vlan <vlan> priority <priority>`
  - Default base priority is 32768 (VLAN ID is added to this value)

Manually specifying port cost for all VLANs **or** per VLAN cost:
- `Switch(config-if)#spanning-tree cost <cost>`
- `Switch(config-if)#spanning-tree vlan <vlan> cost <cost>`

Default port costs:

| Speed    | IEEE Cost (pre 1998) | IEEE Cost (post 2004) |
| -------- | -------------------- | --------------------- |
| 10 Mbps  | 100                  | 2000000               |
| 100 Mbps | 19                   | 200000                |
| 1 Gbps   | 4                    | 20000                 |
| 10 Gbps  | 2                    | 2000                  |
| 100 Gbps | N/A                  | 200                   |
| 1 Tbps   | N/A                  | 20                    |

### PortFast

Enable globally on **all** interfaces:
- `Switch(config)#spanning-tree portfast default`

Enable **or** disable per interface:
- `Switch(config-if)#spanning-tree portfast [disable]`

### BPDU Guard

Enable globally:
- `Switch(config)#spanning-tree portfast bpduguard default`
- Only gets enabled on interfaces with PortFast already enabled

Enable **or** disable per interface:
- `Switch(config-if)#spanning-tree bpduguard <enable|disable>`

### Troubleshooting

Common Issues:
1. Root node in the wrong place?
   1. Check VLAN priorities (root node should have the most trunks connected to it)
2. Weird path to root node?
   1. Check for non-default costs along path
3. Trunk interfaces down?
   1. Check for BPDU guard and PortFast

Example troubleshooting output:

```
Switch#show spanning-tree vlan 1
VLAN0001
 Spanning tree enabled protocol ieee
   Root ID 	Priority    24577
  		Address     0019.e86a.2280
  		Cost        4
  		Port        25 (GigabitEthernet0/1)
  		Hello Time   2 sec Max Age 20 sec Forward Delay 15 sec
   Bridge ID 	Priority    28673  (priority 28672 sys-id-ext 1)
  		Address     0019.e86a.1180
  		Hello Time   2 sec Max Age 20 sec Forward Delay 15 sec
  		Aging Time 300
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/11           Desg FWD 19        128.11   P2p
Gi0/1            Root FWD 4         128.25   P2p
Gi0/2            Desg FWD 4         128.26   P2p
```

```
Switch#show spanning-tree bridge
                                                   Hello  Max  Fwd
Vlan                         Bridge ID              Time  Age  Dly  Protocol
---------------- --------------------------------- -----  ---  ---  --------
VLAN0001         28673(28672,    1) 0019.e86a.1180    2    20   15  ieee
VLAN0022         32790(32768,   22) 0019.e86a.1180    2    20   15  ieee
VLAN0045         32813(32768,   45) 0019.e86a.1180    2    20   15  ieee
```

```
Switch#show spanning-tree root
                                        Root    Hello Max Fwd
Vlan                   Root ID          Cost    Time  Age Dly  Root Port
---------------- -------------------- --------- ----- --- ---  ------------
VLAN0001         24577 0019.e86a.2280         4    2   20  15  Gi0/1
VLAN0022         32790 0019.e86a.1180         0    2   20  15
VLAN0045         32813 0019.e86a.1180         0    2   20  15
```

```
Switch#show spanning-tree interface FastEthernet 0/1 portFast
VLAN0001                                       disabled
VLAN0002                                       disabled
VLAN0045                                       disabled
```

```
Switch#show spanning-tree interface FastEthernet 0/1
Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0001            Desg FWD 19        128.1    P2p
VLAN0002            Desg FWD 19        128.1    P2p
VLAN0045            Desg FWD 19        128.1    P2p
```

```
Switch#show spanning-tree interface FastEthernet 0/11 detail
Port 11(FastEthernet0/11) of VLAN0001 is designated forwarding
  Port path cost 0, Port priority 128 ,Port Identifier 128.11
  Designated root has priority 32769, address 0019.e86a.1180
  Designated bridge has priority 32769, address 0019.e86a.1180
  Designated port id is 128.11 ,designated path cost 0
  Timers: message age 1,forward delay 0,hold 0
  Number of transition to forwarding state: 1
  The port is in the portfast mode
  Link type is point-to-point by default
  Bpdu guard is enabled by default
  BPDU: sent 12, received 5
```

## EtherChannel (Layer 2)

Define static channel:
- `Switch(config-if-range)#channel-group <number> mode on`
- `number` does _not_ need to match on both devices
- `number` must _match_ for all interfaces in local Etherchannel 

### PAgP

Define dynamic channel (PAgP - Cisco Proprietary):
- `Switch(config-if-range)#channel-group <number> mode <desirable|auto>`

|             | `on`        | `desirable` | `auto`      |
| ----------- | ----------- | ----------- | ----------- |
| `on`        | Y           | !!! BAD !!! | !!! BAD !!! |
| `desirable` | !!! BAD !!! | Y           | Y           |
| `auto`      | !!! BAD !!! | Y           | N           |

### LACP

Define dynamic channel (LACP - IEEE 802.3ad):
- `Switch(config-if-range)#channel-group <number> mode <passive|active>`

|           | `on`        | `active`    | `passive`   |
| --------- | ----------- | ----------- | ----------- |
| `on`      | Y           | !!! BAD !!! | !!! BAD !!! |
| `active`  | !!! BAD !!! | Y           | Y           |
| `passive` | !!! BAD !!! | Y           | N           |

### Troubleshooting

Common Issues:
1. Etherchannel interface up&down?
   1. Check for PAgP/LACP dynamic channel miss matches
   2. Other device missing Etherchannel configuration?

Example troubleshooting output:

```
Switch#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        u - unsuitable for bundling
        U - in use      f - failed to allocate aggregator
        d - default port
Number of channel-groups in use: 1
Number of aggregators:           1
Group  Port-channel  Protocol        Ports
------+-------------+-----------+----------------------------------------
  1     Po1(SU)          PagP        Fa0/1(P)      Fa0/2(P)
```

## OSPF (v2)

Enable with a process ID:
- `Router(config)#router ospf <process-id>`
  - `process-id` needs to be **locally** unique

Define max number of OSPF routes used for equal cost load balancing:
- `Router(config-router)#maximum-paths <max>`
  - Default `max` is 4
  - Set `max` to 1 to disable load balancing

Define a passive OSPF interface:
- `Router(config-router)#passive-interface <interface>`
- Can also enable globally:
  - `Router(config-router)#passive-interface default`
  - `Router(config-router)#no passive-interface <interface>`

Specify OSPF to advertise a default route:
- `Router(config-router)#default-information originate [always]`
  - `always` option means advertise a default route even if one does not exist

Specify interfaces to advertise/learn on:
- `Router(config-router)#network <network> <wildcard> area <area>`
  - If an interface matches 2 different `network` statements, the first one that was configured is used as the area and mask
- `Router(config-if)#ip ospf <process-id> area <area>`
  - Interface ospf area configuration is prefered over the `network` command if both are configured and match an interface

Manually specify Router ID (RID):
- `Router(config-router)#router-id <rid>`
  - RID selection priority ranking:
    1. `router-id` command value
    2. Highest Loopback interface IP (does not need to be OSPF enabled!)
    3. Highest interface IP (does not need to be OSPF enabled!)
  - Changing RID at runtime requires OSPF neighbor discovery to be restarted:
    - `Router# clear ip ospf process`
    - `Router# reload`

Adjusting timers:
- Hello timer: `Router(config-if)#ip ospf hello-interval <seconds>`
  - Default is 10 seconds for Ethernet interfaces
  - Default is 30 seconds for Serial interfaces
- Dead timer: `Router(config-if)#ip ospf dead-interval <seconds>`
  - Default is 4 * hello timer value
  - Changing the hello timer will **automatically** change the dead timer to 4x the value set

### Cost

Adjusting interface cost:
- Manually:
  
  - `Router(config-if)#ip ospf cost <cost>`
- By interface bandwidth:
  - `Router(config-if)#bandwidth <bandwidth in Kbps>`
    - Find interface default bandwidth in Kbps:
      - `Router#show interface <int>`
- By reference bandwidth:
  - `Router(config-router)#auto-cost reference-bandwidth <bandwidth in Mbps>`
    - Default is 100000 bps or 100 Mbps
- Cost equation: 
  
  - `cost = (reference bandwidth / interface bandwidth)`
- Default costs:

  | Link Type        | Default Bandwidth | Cost |
  | ---------------- | ----------------- | ---- |
  | Serial (56K)     | 56 Kbps           | 1785 |
  | Serial (64K)     | 64 Kbps           | 1562 |
  | Serial (T1)      | 1,549 Kbps        | 64   |
  | Ethernet         | 10,000 Kbps       | 10   |
  | Fast Ethernet    | 100,000 Kbps      | 1    |
  | Gigabit Ethernet | 1,000,000 Kbps    | 1    |
  | 10G Ethernet     | 10,000,000 Kbps   | 1    |
  | 100G Ethernet    | 100,000,000 Kbps  | 1    |

### IPv6 (OSPFv3)

Same as IPv4 with the following notes:
- OSPF neighbors **do not** need to be in the same subnet (with global or unique local addresses) since they use link-local addresses to communicate directly instead
  - When looking at the output of `show ipv6 ospf neighbor` or `show ipv6 ospf interfaces brief`, the IPv4 column for neighbor IP address has been replaced with an interface ID that is assigned locally
- OSPF RID is set **the same way** and selection priority is done **the same way** (_using IPv4 addresses_)
- OSPFv3 uses `ipv6` instead of the `ip` configuration command for all the same commands
- OSPFv3 uses the same OPSFv2 interface configuration assignment to enable OSPF on an interface
  - `Router(config-if)#ipv6 ospf <process-id> area <area>`
  - OSPFv3 **does not** allow the use of the OSPFv1 `network` command to assign OSPF enabled interfaces
- OPSFv2's `224.0.0.5` is `FF02::5` in OPSFv3 for neighbor relationship forming
- OPSFv2's `224.0.0.6` is `FF02::6` in OPSFv3 for DR & BDR communication

### Troubleshooting

Common Issues:
1. No OSPF neighbors...
   1. Authentication values incorrect?
   2. Local interfaces not in an up&up state?
   3. OSPF neighbor interfaces not in the same subnet?
   4. ACL blocking routing protocol packets to 224.0.0.5 and/or 224.0.0.6?
   5. Non-matched hello/dead timer values?
   6. Non-unique RIDs?
   7. Areas do not match?
   8. MTUs do not match?
2. Bad area design...
   1. Interfaces in the same subnet but also in different areas?
   2. Current area not touching an area border router (ABR)?
      1. Unless using a virtual link, all areas must have a connection to the backbone area (area 0)
3. Passive interfaces...
   1. `show ip ospf interface brief` shows even passive interfaces!
      - Use `show ip protocols` to deconflict

Example troubleshooting output:

```
Router#show ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "ospf 10"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 172.16.24.9
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    172.16.24.9 0.0.0.0 area 3
    172.16.20.0 0.0.3.255 area 0
  Routing Information Sources:
    Gateway			Distance		Last Update
    172.16.24.10			110			02:07:17
  Distance: (default is 110)
```

```
Router#show ip ospf interface brief
Interface  	PID  	Area  	IP Address/Mask  	Cost  	State  	Nbrs F/C
Gi0/0      	1    	0     	10.10.10.1/24    	1     	DR     	0/0
Se0/0/0    	1    	2     	10.0.1.1/30      	64    	P2P    	1/1
Se0/0/1    	1    	3     	10.0.1.5/30      	64    	P2P    	1/1
Se0/1/0    	1    	4     	10.0.1.9/30      	64    	P2P    	1/1
```

```
Router#show ip ospf interface Serial 0/0/0
Serial0/0/0 is up, line protocol is up
  Internet Address 172.16.24.9/30, Area 3
  Process ID 10, Router ID 172.16.24.9, Network Type POINT_TO_POINT, Cost: 64
  Transmit Delay is 1 sec, State POINT_TO_POINT,
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:2
  Supports Link-local Signaling (LLS)
  Index 1/2, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 172.16.24.10
  Suppress hello for 0 neighbor(s)
```

```
Router#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override
Gateway of last resort is not set
     10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
O       10.1.0.0/24 [110/65] via 10.51.0.1, 00:45:35, Serial0/0/0
O       10.2.0.0/24 [110/65] via 10.52.0.1, 00:45:35, Serial0/0/1
O       10.50.0.0/30 [110/128] via 10.51.0.1, 00:45:35, Serial0/0/0
                     [110/128] via 10.52.0.1, 00:04:38, Serial0/0/1
     192.168.1.0/24 is variably subnetted, 5 subnets, 2 masks
O       192.168.1.64/26 [110/2] via 192.168.3.5, 00:45:35, GigabitEthernet0/0
O       192.168.1.128/26 [110/2] via 192.168.3.4, 00:45:35, GigabitEthernet0/0
O IA    192.168.1.192/26 [110/2] via 192.168.3.3, 00:45:35, GigabitEthernet0/0
```

```
Router#show ip ospf neighbor
Neighbor ID  Pri  State         Dead Time  Address      Interface
1.1.1.1      1    Full/DR       00:00:31   192.168.1.1  GigabitEthernet0/0
2.2.2.2      1    Full/BDR      00:00:31   192.168.1.2  GigabitEthernet0/0
3.3.3.3      1    2Way/DROTHER  00:00:31   192.168.1.3  GigabitEthernet0/0
```

```
Router#show ip ospf
 Routing Process "ospf 10" with ID 10.51.0.1
 Start time: 19:09:43, Time elapsed: 00:01:01
 Supports only single TOS(TOS0) routes
 Supports opaque LSA
 Supports Link-local Signaling (LLS)
 Supports area transit capability
 Router is not originating router-LSAs with maximum metric
 Initial SPF schedule delay 5000 msecs
 Minimum hold time between two consecutive SPFs 10000 msecs
 Maximum wait time between two consecutive SPFs 10000 msecs
 Incremental-SPF disabled
 Minimum LSA interval 5 secs
 Minimum LSA arrival 1000 msecs
 LSA group pacing timer 240 secs
 Interface flood pacing timer 33 msecs
 Retransmission pacing timer 66 msecs
 Number of external LSA 0. Checksum Sum 0x000000
 Number of opaque AS LSA 0. Checksum Sum 0x000000
 Number of DCbitless external and opaque AS LSA 0
 Number of DoNotAge external and opaque AS LSA 0
 Number of areas in this router is  1. 1 normal 0 stub 0 nssa
 Number of areas transit capable is 0
 External flood list length 0
   Area  BACKBONE(0)
	     Number of interfaces in this area is 3
	     Area has no authentication
	     SPF algorithm last executed 19:09:43 ago
	     SPF algorithm executed 5 times
	     Area ranges are
	     Number of LSA 2. Checksum Sum 0x008AC0
	     Number of opaque link LSA 0. Checksum Sum 0x000000
	     Number of DCbitless LSA 0
	     Number of indication LSA 0
	     Number of DoNotAge LSA 0
	     Flood list length 0
```

```
Router#show ip ospf database
                   OSPF Router with ID(192.168.10.1)  (Process ID 50)
                     Router Link States Area(0)
LinkID        	ADV Router    	Age  	Seq#         	CheckSum  	Link count
192.168.30.1  	192.168.30.1  	90   	0x80000002C  	0x00EB29  	3
192.168.10.1  	192.168.10.1  	181  	0x80000002C  	0x00EB29  	7
192.168.20.1  	192.168.20.1  	91   	0x80000002C  	0x00EB29  	3
192.168.40.1  	192.168.40.1  	90   	0x80000002C  	0x00EB29  	3
```

```
Router#show ipv6 ospf neighbor
Neighbor ID  Pri  State         Dead Time  Interface Id  Interface
2.2.2.2      1    Full/BDR      00:00:33   4             GigabitEthernet0/0
3.3.3.3      1    Full/DROTHER  00:00:33   4             GigabitEthernet0/0
4.4.4.4      1    Full/DROTHER  00:00:33   4             GigabitEthernet0/0
```

```
Router#show ipv6 ospf interface brief
Interface  	PID  	Area  	Intf id  	Cost  	State  	Nbrs F/C
Gi0/0      	10   	0     	4        	1     	DR     	3/3
Gi0/1      	10   	0     	5        	1     	DR     	0/0
```

## EIGRP

Enable with an ASN:
- `Router(config)#router eigrp <asn>`
  - `asn` needs to be **globally** unique

Define max number of EIGRP routes used for _equal cost_ load balancing:
- `Router(config-router)#maximum-paths <max>`
  - Default `max` is 4
  - Set `max` to 1 to disable load ballancing

Enable _unequal cost_ load balancing:
- `Router(config-router)#variance <x>`
  - Applies to _all_ EIGRP routes with a sucessor (S) and feasible sucessor (FS) in the topology table
  - Allows for FS routes with a `FD(FS) < (variance * FD(S))` to be added to the routing table
    - FD is feasible distance

Define a passive EIGRP interface:
- `Router(config-router)#passive-interface <interface>`
- Can also enable globally:
  - `Router(config-router)#passive-interface default`
  - `Router(config-router)#no passive-interface <interface>`

Specify interfaces to advertise/learn on:
- `Router(config-router)#network <network> <wildcard>`
- Can also configure using classful network ID:
  - `Router(config-router)#network <classfull-network>`

Manually specify Router ID (RID):
- `Router(config-router)#eigrp router-id <rid>`
  - RID selection priority ranking:
    1. `eigrp router-id` command value
    2. Highest Loopback interface IP (does not need to be EIGRP enabled!)
    3. Highest interface IP (does not need to be EIGRP enabled!)

Enable auto-summarization:
- `Router(config-router)#auto-summary`
  - Not enabled by default

Define timers:
- Hello timer: `Router(config-if)#ip hello-interval eigrp <asn> <seconds>`
  - Default for Ethernet interfaces is 5 seconds
  - Default for Serial interfaces is 60 seconds
- Hold timer: `Router(config-if)#ip hold-time eigrp <asn> <seconds>`
  - Default is 3 * hello timer value
  - Value does _not_ change in sync when changing the hello timer directly

### Metric

![EIGRP Metric](https://www.practicalnetworking.net/wp-content/uploads/2016/04/eigrp-metric.png)

Metric equation with _default_ K values:
- `metric = 256 * (((10^7) / smallest_bandwidth) + cumulative_delay)`
- Default K values:
  - K1 (Bandwidth) = 1
  - K2 (Load) = 0
  - K3 (Delay) = 1
  - K4 (Reliability) = 0
  - K5 (MTU) = 0

Modify bandwidth:
- `Router(config-if)#bandwidth <bandwidth in Kbps>`
  - Default bandwidth can be seen with `Router#show int <int>`

Modify delay:
- `Router(config-if)#delay <delay in 10s of microseconds>`
  - Default delay can be seen with `Router#show int <int>`

### EIGRP (IPv6)

Same as IPv4 with the following notes:
- EIGRP neighbors **do not** need to be in the same subnet (with global or unique local addresses) since they use link-local addresses to communicate directly instead
  - When looking at the output of `show ipv6 eigrp neighbor` or `show ipv6 eigrp interfaces`, the IPv4 column for neighbor IP address has been replaced with an interface ID that is assigned locally
- EIGRP RID is set **the same way** and selection priority is done **the same way** (_using IPv4 addresses_)
  - If the device does not have an IPv4 address, you will need to set it
- EIGRP uses `ipv6` instead of the `ip` configuration command for all the same commands
- EIGRP uses interface configuration assignment to enable EIGRP on an interface
  - `Router(config-if)#ipv6 eigrp <asn>`
  - EIGRP **does not** allow the use of the `network` command to assign EIGRP enabled interfaces
- EIGRP for IPv4's `224.0.0.10` is `FF02::A` in EIGRP IPv6 for neighbor relationship forming
- EIGRP dual-stack has different values for the following traits for the IPv4 and IPv6 configuration:
  - Equal/unqual load ballancing (`variance` & `maximum-paths`)
  - Hello/hold timers
  - **Not** metric calculation. This is configured per interface with `delay` and `bandwidth` commands and applies for **both** IPv4 and IPv6 EIGRP.

### Troubleshooting

Common Issues:
1. No EIGRP neighbors...
   1. Authentication values incorrect?
   2. Local interfaces not in an up&up state?
   3. EIGRP neighbor interfaces not in the same subnet?
   4. ACL blocking routing protocol packets to 224.0.0.10?
   5. EIGRP neighbors not in same ASN?
   6. K-values do not match?
2. EIGRP neighbors can't stay up...
   1. Local hello timer value greater than neighbor's hold timer value?
3. Passive interfaces...
   1. `show ip eigrp interfaces` shows only active interfaces
      - Use `show ip protocols` to see passive interfaces
4. Auto summarization issues...
   1. Any discontiguous networks?

Example troubleshooting output:

```
Router#show ip eigrp interfaces
IP-EIGRP interfaces for process 50
                  Xmit Queue   Mean  Pacing Time  Multicast   Pending
Interface  Peers  Un/Reliable  SRTT  Un/Reliable  Flow Timer  Routes
Gi0/0      0      0/0          72    0/1          287         0
Se0/0/0    1      0/0          72    0/15         287         0
Se0/0/1    1      0/0          72    0/15         287         0
```

```
Router#show ip protocols
*** IP Routing is NSF aware ***
Routing Protocol is "eigrp 50"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 Protocol for AS(10)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    NSF-aware route hold timer is 240s
    Router-ID: 192.168.10.1
     Topology : 0(Base)
       Active Timer: 3 min
       Distance: internal 90 external 170
       Maximum Path: 4
       Maximum hopcount: 100
       Maximum metric variance: 1
  Automatic Summarization : disabled
  Maximum path: 4
  Routing for Networks:
    10.50.40.0/24
    192.0.2.0
    192.168.10.0
  Passive Interface(s):
    GigabitEthernet0/0
  Routing Information Sources:
    Gateway     Distance  Last Update
    192.0.2.2   90        00:12:13
    192.0.2.10  90        00:12:12
    192.0.2.6   90        00:12:15
  Distance: internal 90 external 170
```

```
Router#show ip eigrp neighbors
IP-EIGRP neighbors for process 50
H  Address          Interface  Hold  Uptime    SRTT  RTQ   Q   Seq
                               (sec)           (ms)       Cnt  Num
0  192.0.2.2        Se0/0/0     9    00:14:46  72    432   0    3
0  192.0.2.6        Se0/0/1     7    00:14:48  72    432   0    3
0  192.0.2.10       Se0/1/0    11    00:14:44  72    432   0    3
```

```
Router#show ip eigrp topology
IP-EIGRP Topology Table for AS(50)/ID(192.168.10.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status
P 172.16.34.0/29, 2 successors, FD is 2681856
        via 192.0.2.6 (2681856/2169856), Serial0/0/1
        via 192.0.2.10 (2681856/2169856), Serial0/1/0
P 192.0.2.8/30, 1 successors, FD is 2169856
        via Connected, Serial0/1/0
P 192.168.40.0/24, 1 successors, FD is 2172416
        via 192.0.2.10 (2172416/28160), Serial0/1/0
P 192.0.2.0/30, 1 successors, FD is 2169856
        via Connected, Serial0/0/0
P 192.168.10.0/24, 1 successors, FD is 28160
        via Connected, GigabitEthernet0/0
P 192.168.30.0/24, 1 successors, FD is 2172416
        via 192.0.2.6 (2172416/28160), Serial0/0/1
P 192.0.2.4/30, 1 successors, FD is 2169856
        via Connected, Serial0/0/1
P 192.168.20.0/24, 1 successors, FD is 2172416
        via 192.0.2.2 (2172416/28160), Serial0/0/0
```

```
Router#show ip route eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override
Gateway of last resort is not set
     172.16.0.0/29 is subnetted, 1 subnets
D       172.16.34.0 [90/2172416] via 192.0.2.6, 00:17:25, Serial0/0/1
                    [90/2681856] via 192.0.2.10, 00:01:14, Serial0/1/0
D    192.168.20.0/24 [90/2172416] via 192.0.2.2, 00:17:23, Serial0/0/0
D    192.168.30.0/24 [90/2172416] via 192.0.2.6, 00:17:25, Serial0/0/1
D    192.168.40.0/24 [90/2172416] via 192.0.2.10, 00:17:22, Serial0/1/0
```

## BGP

Enable with an ASN:
- `Router(config)#router bgp <asn>`
  - `asn` needs to be **globally** unique

Define a remote AS to connect with as a neighbor:
- `Router(config-router)#neighbor <neighbor-ip> remote-as <neighbor-asn>`

Manually specify Router ID (RID):
- `Router(config-router)#bgp router-id <rid>`
  - RID selection priority ranking:
    1. `bgp router-id` command value
    2. Highest Loopback interface IP
    3. Highest interface IP

Specify internal networks to advertise over eBGP:
- `Router(config-router)#network <network> mask <mask>`
- Can also configure using _classful_ network ID:
  - `Router(config-router)#network <classful-network>`
- A route to the network being advertised **must be in the local routing table** in order for it to be advertised over eBGP

### Troubleshooting

Common Issues:
1. No BGP neighbors...
   1. Is the local interface up&up?
   2. Any ACLs blocking TCP port 179?
   3. `remote-as` value in `neighbor` command wrong?
   4. Neighbor IP in `neighbor` command wrong?
   5. Neighbor IPs or local IP in the wrong subnet?
2. No BGP external routes...
   1. Bad subnet defined in `network` command?
   2. Advertised subnet not in local routing table?
      1. May need to use a discard route: `ip route <network> <mask> null0`

Example troubleshooting output:

```
Router#show ip bgp summary 
BGP Router identifier 200.200.200.4 , local AS number 400 
BGP table version is 7, main routing table version 7 
4 network entries using 592 bytes of memory. 
4 path entires using 256 bytes of memory. 
4/4 BGP path/bestpath attribute entries using 544 bytes of memory 
1 BGP AS-PATH entries using 24 bytes of memory 
0 BGP route map cache entries using 0 bytes of memory 
0 BGP filter-list cache entires using 0 bytes of memory 
BGP using 1416 total bytes of memory  
BGP activity 4/0 prefixes, 4/0 paths, scan interval 60 secs 

Neighbor       V  AS   MsgRcvd  MsgSent  TblVer  InQ  OutQ  Up/Down   State/PfxRcd  
200.200.200.1  4  100  26       26       3       0    0     00:21:54  1             
200.200.200.2  4  200  26       26       3       0    0     00:21:53  1             
200.200.200.3  4  300  26       26       3       0    0     00:21:53  1    
```

```
Router#show ip route

Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

     192.168.1.0/24 is variably subnetted, 5 subnets, 5 masks
B    192.168.1.0/25 [20/0] via 200.200.200.1, 00:14:54
B    192.168.1.128/26 [20/0] via 200.200.200.2, 00:14:27
B    192.168.1.192/27 [20/0] via 200.200.200.3, 00:12:47
C       192.168.1.224/28 is directly connected, GigabitEthernet0/1
L       192.168.1.225/32 is directly connected, GigabitEthernet0/1
     200.200.200.0/24 is variably subnetted, 2 subnets, 2 masks
C       200.200.200.0/29 is directly connected, GigabitEthernet0/0
L       200.200.200.4/32 is directly connected, GigabitEthernet0/0
```

```
Router#show ip bgp
BGP table version is 3, local router ID is 200.200.200.4 
Status Codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-Failure, S stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete 
RPKI validation codes: V valid, I invalid, N Not found 

    Network        Next Hop       Metric  LocPrf  Weight  Path   
*>  192.168.1.0    200.200.200.1  0               0       100 i  
*>  192.168.1.128  200.200.200.2  0               0       200 i  
*>  192.168.1.224  0.0.0.0        0               32768   i      
*>  192.168.1.192  200.200.200.3  0               0       300 i  
```

```
Router#show ip bgp neighbors 200.200.200.1
BGP neighbor is 200.200.200.1, remote AS 100, external link 
  BGP version 4, remote router ID 200.200.200.1 
  BGP state = ESTABLISHED, up for 00:22:04 
  Last read = 00:00:29, last write 00:00:29, hold time is 180, keepalive interval is 60 seconds 
  Neighbor sessions: 
    1 active, is not multisession capable  (disabled). 
  Neighbor capabilities: 
    Route refresh: advertised and received(new) 
    Four-octets ASN Capability: advertised and received 
    Address family IPv4 Unicast: advertised and received 
    Enhanced Refresh Capability: advertised and received 
    Multisession Capability: 
    Stateful switchover support enabled: NO for session 1 
  Message statistics: 
    InQ depth is 0 
    OutQ depth is 0 

                    Sent  Rcvd  
    Opens:          1     1     
    Notifications:  0     0     
    Updates:        2     2     
    Keepalives:     2     2     
    Route Refresh:  0     0     
    Total:          5     5     
  Default minimum time between advertisement runs is 30 seconds 

For address family: IPv4 Unicast 
  Session:200.200.200.1 
  BGP table version3, neighbor version 3/0 
  Output queue size : 0 
  Index 1, Advertise bit 0 
  1 update-group member 
  Slow-peer detection is disabled 
  Slow-peer split-update-group dynamic is disabled 
                                 Sent      Rcvd      
  Prefix activity:               ----      ----      
      Prefix Current:              1         1         
      Prefixes Total:              1         1         
      Implicit Withdraw:           0         0         
      Explicit Withdraw:           0         0         
      Used as bestpath:            0         0         
      Used as multipath:           0         0         
                                   Outbound  Inbound   
    Local Policy Denied Prefixes:  --------  --------  
      Bestpath from this peer:     1         0         
      Total:                       1         0         
  Number of NLRIs in the update sent: max 0, min 0 
  Last detected as dynamic slow peer: never 
  Dynamic slow peer recovered: never 
  Refresh Epoch: 1 
  Last Sent Refresh Start-of-rib: never 
  Last Sent Refresh End-of-rib: never 
  Last Received Refresh Start-of-rib: never 
  Last Received Refresh End-of-rib: never 
                                   Sent  Rcvd  
          Refresh Activity:        ----  ----  
              Refresh Start of RIB:  0     0     
              Refresh End of RIB:    0     0     
  
Address tracking is enabled, the RIB does have a route to 200.200.200.1 
  Connections established 1; dropped 0 
  Last reset never 
  Transport(tcp) path-mtu-discovery is enabled 
  Graceful-Restart is disabled 
Connection state is ESTAB, I/O status: 1, unread input bytes: 0 
Connection is ECN Disabled, Mininum incoming TTL 0, Outgoing TTL 1 
Local host: 192.168.1.225, Local port: 179 
Foreign host: 200.200.200.1, Foreign port: 58251 
Connection tableid (VRF): 0 
Maximum output segment queue size: 50 

Enqueued packets for retransmit: 0, input: 0  mis-ordered: 0 (0 bytes) 

Event Timers (current time is 0x4DC0841C): 
Timer      Starts  Wakeups  Next  
Retrans    3       0        0x0   
TimeWait   0       0        0x0   
AckHold    3       0        0x0   
SendWnd    0       0        0x0   
KeepAlive  0       0        0x0   
GiveUp     0       0        0x0   
PmtuAger   0       0        0x0   
DeadWait   0       0        0x0   
Linger     0       0        0x0   
ProcessQ   0       0        0x0   

iss: 4153197359  snduna: 4153197478  sndnxt: 4153197478 
irs: 3201954199  rcvnxt: 3201954318 

sndwnd:  16266  scale:      0  maxrcvwnd:  16384 
rcvwnd:  16266  scale:      0  delrcvwnd:    118 

SRTT: 330 ms, RTTO: 3159 ms, RTV: 2829 ms, KRTT: 0 ms 
minRTT: 0 ms, maxRTT: 1000 ms, ACK hold: 200 ms 
Status Flags: passive open, gen tcbs 
IP Precedence value : 6 

Datagrams (max data segment is 1460 bytes): 
Rcvd: 8 (out of order: 0), with data: 4, total data bytes: 118 
Sent: 7 (retransmit: 0, fastretransmit: 0, partialack: 0, Second Congestion: 
0), with data: 4, total data bytes: 118 

  Packets received in fast path: 0, fast processed: 0, slow path: 0 
  fast lock acquisition failures: 0, slow path: 0 
TCP Semaphore      0x30CD7404  FREE 
```

## HDLC

Enable HDLC on an interface:
- `Router(config-if)#encapsulation hdlc`
  - This is the _default_ encapuslation for serial interfaces

Disable keepalive messages:
- `Router(config-if)#no keepalive `
  - Keepalives are enabled by default
  - Sent every 10 seconds by default

Define interface clock rate:
- `Router(config-if)#clock rate <bps>`
  - In units of bits per second
  - Default is T1 speed (1544Kbps)

Define interface bandwidth:
- `Router(config-if)#bandwidth <Kbps>`
  - In units of Kilo bits per second
  - Default is T1 speed (1544Kbps)
  - Has **no** effect on actual line speed. Used for routing protocol metric calculations.

Common serial TDMA speeds:
- Overhead is 8Kbps

| Name        | Rate                            |
| ----------- | ------------------------------- |
| DS0         | 64 Kbps                         |
| T1 (DS1)    | 1544 Kbps (24 DS0s + overhead)  |
| T3 (DS3)    | 44736 Kbps (28 DS1s + overhead) |
| E1 (Europe) | 2048 Kbps (32 DS0s + overhead)  |
| E3 (Europe) | 32768 Kbps (16 E1s + overhead)  |

### Troubleshooting

Common Issues:
1. Interface is up&down?
   1. If other side is flipping between states, check for encapsulation miss match
   2. If other side _stays up_, check for keepalive messages disabled on one side but not the other
      1. Side showing up should be the one with keepalives disabled

Example troubleshooting output:

```
Router#show controllers serial 0/0/0
Interface Serial0/0/0
Hardware is SCC
DCE V.35, clock rate 256000
```

```
Router#show interfaces serial 0/0/0
Serial0/0/0 is up, line protocol is up
  Hardware is GT96K Serial
  Internet address is 10.100.0.1/12
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
    reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation HDLC
  Keepalive not set
  Last input never, output never, output hang never
  Last clearing of "show interface" counters 07:09:39
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: weighted fair
  Output queue: 0/1000/64/0 (size/max total/threshold/drops)
    Conversations  0/0/256 (active/max active/max total)
    Reserved Conversations 0/0 (allocated/max allocated)
    Available Bandwidth 1158 kilobits/sec
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     0 packets input, 0 bytes, 0 no buffer
     Received 0 broadcasts, 0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort
     0 packets output, 0 bytes, 0 underruns
     0 output errors, 0 collisions, 2 interface resets
     0 output buffer failures, 0 output buffers swapped out
     0 carrier transitions
     DCD=up  DSR=up  DTR=down  RTS=down  CTS=up
```

## PPP

Enable PPP on an interface:
- `Router(config-if)#encapsulation ppp`
  - The _default_ encapuslation for serial interfaces is HDLC

Define interface clock rate:
- `Router(config-if)#clock rate <bps>`
  - In units of bits per second
  - Default is T1 speed (1544Kbps)

Define interface bandwidth:
- `Router(config-if)#bandwidth <Kbps>`
  - In units of Kilo bits per second
  - Default is T1 speed (1544Kbps)
  - Has **no** effect on actual line speed. Used for routing protocol metric calculations.

### PAP

Enable PAP on the interface connecting to the neighbor:
- `Router(config-if)#ppp authentication pap `
  - PPP must be enabled on the interface first before this command

Define _local_ username and password:
- `Router(config-if)#ppp pap sent-username <username> password <password> `
  - PPP must be enabled on the interface first before this command
  - `username` is the username sent to _neighbor device_ 
  - `password` is the password sent to _neighbor device_

Define a _neighbor_ username and password expected from PAP:
- `Router(config)#username <username> password <password>`
  - `username` must match case-sensitive the username configured on the _neighbor_ device
  - `password` must match case-sensitive the password configured on the _neighbor_ device

### CHAP

Enable CHAP on the interface connecting to the neighbor device:
- `Router(config-if)#ppp authentication chap `
  - PPP must be enabled on the interface first before this command

Define username and password for neighbor device:
- `Router(config)#username <username> password <password>`
  - Must be done on _both_ devices
  - `username` must match case-sensitive the _hostname_ of the _neighbor_ device
  - `password` must match case-sensitive on _both devices_

### MLPPP

Create local multilink interface:
- `Router(config)#interface multilink <num>`
  - `num` must match multilink group locally and on neighbor router
- `Router(config-if)#encapsulation ppp`
- `Router(config-if)#ppp multilink`
- `Router(config-if)#ip address <ip> <mask>`
  - This is the IP of the multilink bundle logically seen by the neighbor router
- `Router(config-if)#ppp multilink group <num>`
  - `num` must match local group and on neighbor router

Add the multilink interface on all serial interfaces in the multilink:
-  `Router(config)#interface serial <int>`
-  `Router(config-if)#encapsulation ppp`
-  `Router(config-if)#ppp multilink`
-  `Router(config-if)#no ip address`
-  `Router(config-if)#ppp multilink group <num>`
  -  `num` must match multilink group locally and on neighbor router
- Add PAP/CHAP authentication on **each physical interface** in the multilink group if it is used

### PPPoE

Create _logical_ dialer interface:
- `Router(config)#interface dialer <num>`
  - `num` is only _locally_ unique
- `Router(config-if)#encapsulation ppp`
  - Layer 2: Add PAP/CHAP authentication to dialer interface if it is used
- `Router(config-if)#ip address negotiated`
  - Layer 3: Uses IPCP (a type of NCP) from PPP to learn IP from neighbor
- `Router(config-if)#mtu 1492`
  - Layer 3: Accounts for the 8 byte PPPoE header
- `Router(config-if)#dialer pool <pool>`
  - Layer 1: `pool` is only _locally_ unique

Define _physical_ Ethernet interface:
-  `Router(config)#interface <eth-int>`
-  `Router(config-if)#pppoe enable`
   - Layer 2: enables PPoE on interface
-  `Router(config-if)#pppoe-client dial-pool-number <pool>`
   - Layer 1: `pool` must match the pool defined in the corresponding dialer interface
-  `Router(config-if)#no ip address`
   - Layer 3: IP address tied to logical _not_ physical interface in PPPoE

### Troubleshooting

Common Issues:
1. PPP issues...
   1. Interface is up&down?
      1. If other side is flipping between states, check for encapsulation miss match
      2. If other side _stays down_, check for PAP/CHAP authentication failure
      3. Check LCP (Link Control Protocol) state?
         1. `REQsent` => likely an encapsulation miss match
         2. `LCPopen` => link is up
   2. `ping` to neighbor interface works but no routing?
      1. Check if interfaces in different subnets?
         1. PPP will add a host route to routing table by default (makes `ping` work)
   3. IPv4 works but not IPv6?
      1. Check NCP (Network Control Protocols) for IPv6CP in `Open` state?
   4. CDP is not working?
      1. Check NCP (Network Control Protocols) for CDPCP in `Open` state?
2. MLPPP issues...
   1. IP address assigned to physical interfaces instead of multilink interface?
   2. Interface multilink number _and_ group number don't match locally **and** with neighbor?
   3. Multilink interface will be up&up as long as one of the serial links in the multilink group is up&up
3. PPPoE issues...
   1. `show interface dialer <num>` shows the interface as `up (spoofing) & up (spoofing)`?
      1. If `show pppoe session` has no output, then check (**Layer 1**):
         1. Dial pool numbers matching in the Ethernet interface and dialer interface?
      2. If `show pppoe session` has no virtual access interface output, then check (**Layer 2**):
         1. CHAP/PAP authentication issues?
   2. `show interface dialer <num>` shows the interface as `up & up (spoofing)`?
      1. If dialer interface does not have an IP, then check (**Layer 3**):
         1. MTU equals 1492 on dialer interface?
         2. PPP NCP protocols allow for IPCP or IPv6CP to negotiate IP on dialer interface?

Example troubleshooting output:

```
Router#show controllers serial 0/0/1
Interface Serial0/0/1
Hardware is SCC
DTE V.35 RX clock detected.
```

```
Router#show ppp multilink
Multilink1
  Bundle name: R2
  Remote Endpoint Discriminator: [1] R2
  Local Endpoint Discriminator: [1] R1
  Bundle up for 00:06:00, total bandwidth 3088, load 1/255
  Receive buffer limit 24000 bytes, frag timeout 1000 ms
    0/0 fragments/bytes in reassembly list
    0 lost fragments, 3 reordered
    0/0 discarded fragments/bytes, 0 lost received
    0x26 received sequence, 0x2A sent sequence
  Member links: 2 active, 0 inactive (max 255, min not set)
    Se0/0/0, since 00:06:00
    Se0/1/0, since 00:05:53
No inactive multilink interfaces
```

```
Router#show pppoe session
1 client session
Uniq ID  PPPoE  RemMAC          Port                VT   VA         State
           SID  LocMAC                                   VA-st      Type
     N/A     1  30f7.0da3.1641 Gi0/1                Di2  Vi2        UP
                30f7.0da3.0da1                           UP
```

```
Router#show interfaces serial 0/0/1
Serial0/0/1 is up, line protocol is up
  Hardware is GT96K Serial
  Internet address is 10.100.0.2/12
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
    reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation PPP, LCP Open
   Open: IPCP, CDPCP, Loopback not set
  Keepalive set (10sec)
  Last input never, output never, output hang never
  Last clearing of "show interface" counters 07:09:39
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: weighted fair
  Output queue: 0/1000/64/0 (size/max total/threshold/drops)
    Conversations  0/0/256 (active/max active/max total)
    Reserved Conversations 0/0 (allocated/max allocated)
    Available Bandwidth 1158 kilobits/sec
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     0 packets input, 0 bytes, 0 no buffer
     Received 0 broadcasts, 0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort
     0 packets output, 0 bytes, 0 underruns
     0 output errors, 0 collisions, 2 interface resets
     0 output buffer failures, 0 output buffers swapped out
     0 carrier transitions
     DCD=up  DSR=up  DTR=down  RTS=down  CTS=up
```

```
Router#show interfaces dialer 2
Dialer2 is up, line protocol is up (spoofing)
  Hardware is Unknown
  Internet address is 10.1.3.1/32
  MTU 1492 bytes, BW 56 Kbit/sec, DLY 20000 usec,
    reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation PPP, LCP Closed, loopback not set
  Keepalive set (10 sec)
  DTR is pulsed for 1 seconds on reset
```

## GRE Tunnels

Define physical interface _public_ IP:
- `Router(config)#interface <int>`
- `Router(config-if)#ip address <public-ip> <mask>`
  - `public-ip` is the public IP used for the point-to-point connection across the WAN

Define the tunnel:
- `Router(config)#interface tunnel <num>`
  - `num` is only _locally_ unique
- `Router(config-if)#tunnel mode gre ip`
  - Sets GRE encapsulation for IPv4 _only_
- `Router(config-if)#tunnel source <src>`
  - `src` can be a local interface or IP on the WAN
  - This is the **local public source address** for the start of the tunnel
- `Router(config-if)#tunnel destination <dst>`
  - `dst` can be an IP or hostname for the tunnel endpoint across the WAN
  - This is the **remote public destination address** of the other end of the tunnel
- `Router(config-if)#ip address <private-ip> <mask>`
  - `private-ip` is used for the point-to-point **private** connection **inside** the tunnel

### Troubleshooting

Common Issues:
1. Is tunnel interface up&up?
   1. Tunnels are _stateless_
      1. Local end being up&up **does not** mean the remote end is up&up as well
2. Is tunnel interface up&down?
   1. By default, without any additional configuration, the tunnel will be up&down
   2. Is tunnel _destination_ IP route in the local routing table?
      1. If not, tunnel will be up&down
3. Is tunnel _source_ interface up&up?
4. Is an ACL blocking GRE (**transport** protocol number 47)
   1. Need a `permit ip ...` or `permit gre ...`
5. Is tunnel interface flapping states (up and down)?
   1. Likely a _recursive route_ due to a dynamic routing protocol being used through the tunnel to learn a better route to the tunnel's public interface through the tunnel itself

Example troubleshooting output:

```
Router#show interfaces tunnel 0
Tunnel0 is up, line protocol is up
  Hardware is Tunnel
  Internet address is 192.168.1.2/24
  MTU 1514 bytes, BW 8000000 Kbit/sec, DLY 5000 usec,
    reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation TUNNEL, Loopback not set
  keep alive not set
  Tunnel Source 10.10.10.2 (Serial0/0/1) , destination 10.10.10.1
   Tunnel Subblocks:
      src-tracks:
         Tunnel0 source tracking sub block associated with Serial0/0/1
          set of tunnel with sourceSerial0/0/1 , 1 member (includes iterators)
  , on interface<OK>
  Tunnel protocol/Transport GRE/IP
    keep disabled, sequencing disabled
  Tunnel 255, Fast tunnelling enabled
  Tunnel transport MTU 1476 bytes
  Tunnel transmit bandwidth 8000 (kbps)
  Tunnel receive bandwidth 8000 (kpbs)
  Last input never, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/0 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     0 packets input, 0 bytes, 0 no buffer
     Received 0 broadcasts, 0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort
     0 packets output, 0 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 output buffer failures, 0 output buffers swapped out
```

```
Router#show ip route connected
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override
Gateway of last resort is 172.16.1.2 to network 0.0.0.0
     1.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       1.1.1.0/24 is directly connected, GigabitEthernet0/0
L       1.1.1.1/32 is directly connected, GigabitEthernet0/0
     172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.16.1.0/24 is directly connected, Tunnel0
L       172.16.1.1/32 is directly connected, Tunnel0
     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/30 is directly connected, Serial0/0/0
L       192.168.1.1/32 is directly connected, Serial0/0/0
```

```
Router#show ip interface brief
Interface           IP-Address   OK? METHOD  Status                 Protocol
GigabitEthernet0/0  1.1.1.1      YES NVRAM   up                     up
GigabitEthernet0/1  unassigned   YES NVRAM   administratively down  down
Serial0/0/0         192.168.1.1  YES NVRAM   up                     up
Serial0/0/1         unassigned   YES NVRAM   administratively down  down
Tunnel0             172.16.1.1   YES NVRAM   up                     up
```

## ACLs

Adding notes to an ACL (named & numbered):
- `Router(config)#access-list <num> remark <msg>`
- `Router(config-std-nacl)#remark <msg>`
- `Router(config-ext-nacl)#remark <msg>`

Special IP and wildcard mask combinations:
- IPv4:
  - `host <ip>` = `<ip> 0.0.0.0`
  - `any` = `x.x.x.x 255.255.255.255`
- IPv6:
  - `host <ipv6>` = `<ipv6>/128`
  - `any` = `::/0`

### Standard

Numbered ACL definition:
- `Router(config)#access-list <num> <permit|deny> <src> <wildcard> [log]` 
  - `num` must be in ranges 1-99 or 1300-1999
  - `log` keyword enables notificational logging (level 6) for matching packets

Named ACL definition:
-  `Router(config)#ip access-list standard <num|name>`
  - `num` must be in ranges 1-99 or 1300-1999 _if used_
    - Numbered ACLs configured through named ACL configuration mode show up as numbered ACLs in the running configuration but are managed through named ACL configuration mode
- `Router(config-std-nacl)#<seq> <permit|deny> <src> <wildcard>`
  - `seq` is the sequence number for this rule in the list
    - Named ACLs enable editing/specifying ACL list order using sequence numbers _before_ each rule

### Extended

Numbered ACL definition:
- `Router(config)#access-list <num> <permit|deny> <proto> <src> <wc> <dst> <wc> [log]` 
  - `num` must be in ranges 100-199 or 2000-2699
  - `log` keyword enables notificational logging (level 6) for matching packets
  - `proto` is the _transport_ layer protocol keyword (`ip`, `tcp`, `udp`, `icmp`, `gre`, `ospf`, `eigrp`)
    - For TCP/UDP, following each IP & wildcard pair for source & destination, you can specify ports ( `eq`, `lt`, `ne`, `gt`, `range`)
    - Not specfying specific ports assumes _all_ ports will match rule

Named ACL definition:
-  `Router(config)#ip access-list extended <num|name>`
  - `num` must be in ranges 100-199 or 2000-2699 _if used_
    - Numbered ACLs configured through named ACL configuration mode show up as numbered ACLs in the running configuration but are managed through named ACL configuration mode
- `Router(config-ext-nacl)#<seq> <permit|deny> <proto> <src> <wildcard> <dst> <wildcard>`
  - `seq` is the sequence number for this rule in the list
    - Named ACLs enable editing/specifying ACL list order using sequence numbers _before_ each rule
  - `proto` is the _transport_ layer protocol keyword (`ip`, `tcp`, `udp`, `icmp`, `gre`, `ospf`, `eigrp`)
    - For TCP/UDP, following each IP & wildcard pair for source & destination, you can specify ports ( `eq`, `lt`, `ne`, `gt`, `range`)
    - Not specfying specific ports assumes _all_ ports will match rule

Common application ports to know:

| Port      | Protocol  | Application | Extended ACL keyword |
| --------- | --------- | ----------- | -------------------- |
| 20        | TCP       | FTP Data    | `ftp-data`           |
| 21        | TCP       | FTP Control | `ftp`                |
| 22        | TCP       | SSH         | -                    |
| 23        | TCP       | Telnet      | `telnet`             |
| 25        | TCP       | SMTP        | `smtp`               |
| 53        | UDP (TCP) | DNS         | `domain`             |
| 67        | UDP       | DHCP Server | `bootps`             |
| 68        | UDP       | DHCP Client | `bootpc`             |
| 69        | UDP       | TFTP        | `tftp`               |
| 80        | TCP       | HTTP        | `www`                |
| 110       | TCP       | POP3        | `pop3`               |
| 161 & 162 | UDP       | SNMP        | `snmp`               |
| 443       | TCP       | SSL         | -                    |
| 514       | UDP       | SYSLOG      | -                    |

### IPv6

Similar to IPv4 ACLs with the following notes:
- Only use _named_ ACLs and only match IPv6 traffic
  - `Router(config)#ipv6 access-list <name>`
- Can be used in conjunction with IPv4 ACLs (Dual Stack)
  - 1 IPv6 and/or IPv4 ACL per interface per direction 
- No concept of wildcard masks. Instead, IPv6 ACLs use prefix lengths.
- Standard IPv6 ACL:
  - `Router(config-ipv6-acl)#<permit|deny> <ipv6|icmp|tcp|...> <src> <dst>`
  - Contain a source _and_ destination **only**
- Extended IPv6 ACLs look like standard ones, but they match on ports/icmp types/etc
- Applying IPv6 ACL to an interface:
  - `Router(config-if)#ipv6 traffic-filter <name> <in|out>`
- Applying IPv6 ACL to a VTY line:
  - `Router(config-line)#ipv6 access-class <name> <in|out>`
- IPv6 ACLs also end in a default `deny any any` but also have the following **implicit permits**:
  - `permit icmp any any nd-na`
  - `permit icmp any any nd-ns`
  - These do _not_ include NDP router solicitations/advertisements:
    - `permit icmp any any router-advertisement`
    - `permit icmp any any router-solicitation`

### Troubleshooting

Common Issues:
1. Standard ACLs **close to source**?
2. Extended ACLs **close to destination**?
3. ACL rules ordered most to least specific?
   
   1. ACLs use _first match_ logic
4. ACL applied in wrong direction?
5. ACL has bad wildcard mask or swapped source and destination addresses?
6. Routers **ignore** outbound ACL for self generated packets
7. Router self-pings...
   1. To Serial interfaces will leave local interface and use inbound ACL if there is one
   2. To Ethernet interfaces will **not** leave local interface and instead test local TCP/IP stack
8. IPv6 ACL blocking required ICMP values? Example bad ACLs:
   1. `deny icmp any any`
   2. `deny ipv6 ff00::/8 any`
   3. `deny ipv6 any ff0::/8`
9. Router routing protocol overhead being blocked?
   1. | Protocol | Addresses (IPv4)      | Transport Protocol         | Addresses (IPv6) |
      | -------- | --------------------- | -------------------------- | ---------------- |
      | RIPv2    | 224.0.0.9             | UDP port 520               | FF02::9          |
      | OSPF     | 224.0.0.5 & 224.0.0.6 | OSPF (protocol number 89)  | FF02::5, FF02::6 |
      | EIGRP    | 224.0.0.10            | EIGRP (protocol number 88) | FF02::A          |

Example troubleshooting output:

```
Router#show ip access-lists
 Extended IP access list 100
	10  permit icmp any host 172.30.4.190
	20  deny icmp any 172.30.4.128 0.0.0.63
	30  permit tcp any host 172.30.4.190 eq 22
	40  permit tcp any host 172.30.4.129 eq 22
	50  deny tcp any host 172.30.4.129 eq telnet
	60  deny tcp any host 172.30.4.190 eq telnet
	70  permit ip any any
```

```
Router#show ipv6 access-lists
 Ipv6 access list advanceipv6
	 permit icmp host 3001::1 any sequence 10
	 permit icmp host 3000::1 any sequence 20
	 deny icmp 3000::/64 any sequence 30
	 permit tcp host 3000::1 eq 22 any sequence 40
	 permit tcp host 2750::2 eq 22 any sequence 50
	 deny tcp host 3000::1 eq telnet any sequence 60
	 deny tcp host 2750::2 eq telnet any sequence 70
	 permit ipv6 any any sequence 80
```

```
Router#show ip interface serial 0/0/0
Serial0/0/0 is up, line protocol is up
  Internet address is 172.30.4.230/30
  Broadcast address is 255.255.255.255
  Address determined by non-volatile memory
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Multicast reserved groups joined: 224.0.0.10
  Outgoing access list is not set
  Inbound  access list is 100
  Proxy ARP is enabled
  Local Proxy ARP is disabled
  Security level is default
  Split horizon is enabled
  ICMP redirects are always sent
  ICMP unreachables are always sent
  ICMP mask replies are never sent
  IP fast switching is enabled
  IP fast switching on the same interface is disabled
  IP Flow switching is disabled
  IP CEF switching is enabled
  IP CEF Fast switching turbo vector
  IP multicast fast switching is enabled
  IP multicast distributed fast switching is disabled
  IP route-cache flags are Fast, CEF
  Router Discovery is disabled
  IP output packet accounting is disabled
  IP access violation accounting is disabled
  TCP/IP header compression is disabled
  RTP/IP header compression is disabled
  Policy routing is disabled
  Network address translation is disabled
  BGP Policy Mapping is disabled
  Input features: MCI Check
  Output features: Post-Ingress-NetFlow
  IPv4 WCCP Redirect outbound is disabled
  IPv4 WCCP Redirect inbound is disabled
  IPv4 WCCP Redirect exclude is disabled
```

## Inter-VLAN Routing

### Layer 3 Switch

Enable IPv4/IPv6 routing:
- `Switch(config)#sdm prefer lanbase-routing`
  - Configures ASIC hardware to make room in memory for IP routing tables
  - _May_ require a `reload` before enabling IP routing with the next command
  - _May_ not be required on all switch models
- `Switch(config)#sdm prefer dual-ipv4-and-ipv6`
  - Used to enable IPv6 routing (does not support dynamic IPv6 routing)
- `Switch(config)#ip routing`

Define a Layer 3 switch SVI (logical):
- `Switch(config)#interface vlan <num>`
- `Switch(config-if)#ip address <ip> <mask>`
- Used between access and distribution switches with _multiple access_ ports connected to a VLAN

Define a Layer 3 Switch Routed Port (physical):
- `Switch(config-if)#no switchport`
- `Switch(config-if)#ip address <ip> <mask>`
- Used between distribution and core switches with _one_ link between each other for point-to-point routing 

Define a Layer 3 Switch EtherChannel (Layer 3):
1. Used between distribution and core switches with _multiple_ links between each other for redundant/load balanced routing 
2. Define associated _physical_ interfaces:
   - `Switch(config)#interface <int>`
   - `Switch(config-if)#no switchport`
     - This command will remove **all L2 configuration commands on the interface**
   - `Switch(config-if)#no ip address`
   - `Switch(config-if)#channel-group <num> mode <on|active|passive|desirable|auto>`
     - `num` must match _locally_ defined EtherChannel interface number
     - Mode supports LACP/PAgP negotiation or `on` for always enabled
3. Define layer 3 _logical_ EtherChannel:
   - `Switch(config)#interface port-channel <num>`
     - `num` must match _locally_ on all physical interfaces in EtherChannel
   - `Switch(config-if)#no switchport`
   - `Switch(config-if)#ip address <ip> <mask>`

### Router on a Stick

Defining a subinterface for per-VLAN traffic:

- `Router(config)#interface gigabitEthernet 0/<subint>`
  - `subint` is a number to represent the VLAN interface on the trunk (eg `0.10` for VLAN 10)
  - `subint` _does not_ need to match the VLAN ID on the trunk
- `Router(config-subif)#encapsulation <dot1q|isl> <vlan> [native]`
  - `vlan` is the encapsulated VLAN ID on the trunk matching this subnet. It _must_ match the VLAN ID configured on the switch.
  - `native` keyword is used to specify the native VLAN on the trunk that the switch sends untagged
- `Router(config-subif)#ip address <ip> <mask>`
  - Encapsulation must be configured first before this command

Defining native VLAN on _physical_ interface (for untagged traffic):

- `Router(config)#interface <int>`
- `Router(config-if)#ip address <ip> <mask>`
  - Router expects untagged native VLAN traffic to be in this subnet _without_ having to define a matching native VLAN ID as done in a subinterface

### Troubleshooting

Common Issues:

1. SVI issues...
   1. Missing `sdm prefer` or `ip routing`?
   2. VLAN interface is shutdown?
   3. VLAN is shutdown?
   4. VLAN interface created but not assigned to an interface?
      1. VLAN interface will be up&down until VLAN assigned to an access port
2. Router on a stick issues...
   1. Wrong encapsulation number on subinterface to match VLAN ID?
   2. Bad IP/subnet mask for VLAN subinterface?
   3. Wrong native VLAN ID on subinterface?
      1. `native` keyword on wrong subinterface?
      2. IP/subnet mask wrong on physical interface for native VLAN?
   4. DTP in use on neighbor switch?
   5. Physical interface up&up? 
      1. Logical subinterfaces will not come up if physical interface is down
3. Layer 3 Etherchannel issues...
   1. Non-matching channel numbers on interfaces and channel definition?
   2. `switchport` (L2) not disabled on all interfaces _and_ channel?
   3. IP address assigned to physical interface and not channel interface?
   4. LACP/PAgP protocol miss match?

Example troubleshooting output:

```
Router#show vlan
Virtual LAN ID:    1 IEEE 802.1Q Encapsulation

  vLAN Trunk Interfaces:   GigabitEthernet0/0
This is configured as native vlan for the following interface(s):
GigabitEthernet0/0             Native-vlan Tx-type:Untagged

  Protocols Configured:  	Address:        	Received:  	Transmitted:
   IP                    	100.100.100.65  	0          	0
   other                 	                	0          	0
  
  0 packets, 0 bytes input
  0 packets, 0 bytes output
  
Virtual LAN ID:    10 IEEE 802.1Q Encapsulation

  vLAN Trunk Interfaces:   GigabitEthernet0/0.100
  Protocols Configured:	Address:	Received:	Transmitted:
  
  0 packets, 0 bytes input
  0 packets, 0 bytes output
  
Virtual LAN ID:    20 IEEE 802.1Q Encapsulation

  vLAN Trunk Interfaces:   GigabitEthernet0/0.150
  Protocols Configured:  	Address:         	Received:  	Transmitted:
   IP                    	100.100.100.129  	0          	0
   other                 	                 	0          	0
   
  0 packets, 0 bytes input
  0 packets, 0 bytes output
```

```
Router#show ip interface brief
Interface               IP-Address       OK? METHOD  Status                 Protocol
GigabitEthernet0/0      100.100.100.65   YES NVRAM   up                     up
GigabitEthernet0/0.150  100.100.100.129  YES NVRAM   up                     up
GigabitEthernet0/0.100  unassigned       YES NVRAM   up                     up
GigabitEthernet0/1      unassigned       YES NVRAM   administratively down  down
Serial0/0/0             192.168.1.1      YES NVRAM   up                     up
Serial0/0/1             unassigned       YES NVRAM   administratively down  down
```

```
Switch#show ip interface brief
Interface           IP-Address  OK? METHOD  Status  Protocol
FastEthernet0/1     unassigned  YES NVRAM   up      up
FastEthernet0/2     unassigned  YES NVRAM   down    down
FastEthernet0/3     unassigned  YES NVRAM   down    down
FastEthernet0/4     unassigned  YES NVRAM   down    down
FastEthernet0/5     unassigned  YES NVRAM   down    down
FastEthernet0/6     unassigned  YES NVRAM   down    down
FastEthernet0/7     unassigned  YES NVRAM   down    down
FastEthernet0/8     unassigned  YES NVRAM   down    down
FastEthernet0/9     unassigned  YES NVRAM   down    down
FastEthernet0/10    unassigned  YES NVRAM   up      up
FastEthernet0/11    unassigned  YES NVRAM   up      up
FastEthernet0/12    unassigned  YES NVRAM   down    down
FastEthernet0/13    unassigned  YES NVRAM   down    down
FastEthernet0/14    unassigned  YES NVRAM   down    down
FastEthernet0/15    unassigned  YES NVRAM   down    down
FastEthernet0/16    unassigned  YES NVRAM   down    down
FastEthernet0/17    unassigned  YES NVRAM   down    down
FastEthernet0/18    unassigned  YES NVRAM   down    down
FastEthernet0/19    unassigned  YES NVRAM   down    down
FastEthernet0/20    unassigned  YES NVRAM   down    down
FastEthernet0/21    unassigned  YES NVRAM   down    down
FastEthernet0/22    unassigned  YES NVRAM   down    down
FastEthernet0/23    unassigned  YES NVRAM   down    down
FastEthernet0/24    unassigned  YES NVRAM   down    down
GigabitEthernet0/1  unassigned  YES NVRAM   up      up
GigabitEthernet0/2  unassigned  YES NVRAM   up      up
Vlan1               unassigned  YES NVRAM   up      up
Vlan69              10.10.69.1  YES NVRAM   up      up
Vlan70              10.10.70.1  YES NVRAM   up      down
```

## HSRP

Enable on an interface:

- `Router(config-if)#standby <group> ip <virtual-ip>`
- `group` must be **globally** unique across all routers in this HSRP group

Define group priority:

- `Router(config-if)#standby <group> priority <priority>`
- Setting this higher than the current active router's priority will _not_ make the local router active unless preemption is enabled
- The default priority is **100**

Enable preemption:

- `Router(config-if)#standby <group> preempt`
- This is **disabled by default** 

Define version:

- `Router(config-if)#standby version <1|2>`
- The **default is version 1**
- Version must match _globally_ with all routers in the HSRP group

Define timers:

- `Router(config-if)#standby <group> timers <hello> <hold>`
- `hello` and `hold` are in seconds
- _Only the active router can set and immediately activate changes in these timer values_
  - If the local router is not active, the timer values will activate when the local router becomes active

Define interface priority tracking:

- `Router(config)#track <track-num> <interface> line-protocol`
- `Router(config-if)#standby <group> track <track-num> decrement <priority-dec>`
  - `priority-dec` will be decrimented from the HSRP group number `group`'s priority if interface `interface`'s line protocol changes state

HSRP/VRRP version differences:

| Differences                            | HSRP Version 1   | HSRP Version 2   | VRRP             |
| -------------------------------------- | ---------------- | ---------------- | ---------------- |
| IPv6 Support                           | No               | Yes              | v2 No & v3 Yes   |
| Smallest Hello/Hold Timer Unit         | seconds          | milliseconds     | milliseconds     |
| Group Number Range                     | 0 - 255          | 0 - 4095         | 0 - 255          |
| Virtual MACs Used (`x` = group number) | `0000.0C07.ACxx` | `0000.0C9F.Fxxx` | `0000.5E00.01xx` |
| IPv4 Multicast Addresses Used          | 224.0.0.2        | 224.0.0.102      | 224.0.0.18       |
| Require a Unique Router ID?            | No               | Yes              | Yes              |
| Preemption Enabled By Default?         | No               | No               | Yes              |

### Troubleshooting

Common Issues:

1. Multiple active routers?
   1. Likely a group number miss match
2. Default gateway only works sometimes?
   1. Routers in group with miss matching virtual IPs?
   2. Virtual IP not in in the same subnet as all interfaces?
   3. Virtual IP in use by some other device in the subnet?
3. Router with highest priority not active?
   1. HSRP routers will listen for a certain amount of time on boot and default to the active router in the HSRP group if no other router with a better priority or IP speaks during that time
      1. This methodology favors the router who booted first if _preemption is not enabled_
4. Multiple active routers in a given group?
   1. Version miss match?
   2. ACL blocking UDP port 1985 and/or 224.0.0.2/224.0.0.102?

Example troubleshooting output:

```
Router#show standby
GigabitEthernet0/0 - Group 2
	State is Standby
		6 State changes last change is 00:00:25
	Virtual IP address is 10.10.10.15
	Active virtual MAC address is 0000.0c07.ac02
		Local virtual MAC address is 0000.0c07.ac02 (v1 default)
	Hello time 3 sec, hold time 10 sec
		Next hello sent in 0.752 secs
	Preemption enabled
	Active router is 10.10.10.1, priority 100 (expires in 8.080 sec)
	Standby router is local
	Priority 90 (configured 140)
		Track object 1 state Down decrement 50
	Group name is "hsrp-g0/0-2" (default)
GigabitEthernet0/0 - Group 1
	State is Standby
		4 State changes last change is 00:00:25
	Virtual IP address is 10.10.10.10
	Active virtual MAC address is 0000.0c07.ac01
		Local virtual MAC address is 0000.0c07.ac01 (v1 default)
	Hello time 3 sec, hold time 10 sec
		Next hello sent in 0.752 secs
	Preemption enabled
	Active router is 10.10.10.1, priority 140 (expires in 8.080 sec)
	Standby router is local
	Priority 50 (default 100)
		Track object 1 state Down decrement 50
	Group name is "hsrp-g0/0-1" (default)
```

## SNMP

_NOTE:_ SNMP servers are valid on Routers _and_ Switches. The below configuration examples may also be run on a Switch.

Terminology:
- OID = Variable (eg `1.3.6.12.19`)
- MIB = Database of OIDs
- NMS = Manager = Client
  -  This is the device getting/setting MIB on different devices
- Agent = Server
  - This is the device with the MIB responding to NMS queries/requests

Define server location & contact:
- `Router(config)#snmp-server location <msg>`
- `Router(config)#snmp-server contact <msg>`

Enable SNMP traps:
- `Router(config)#snmp-server enable traps`

### Version 1

 Define a community string:
- `Router(config)#snmp-server community <password> <ro|rw> [ipv6 <acl>] [<acl>]`
- `password` must match on both SNMP NMS and agent 
  - Sent in **clear text**
- `ro` defines a read only password that only allows for SNMP Get Requests from NMS to agent 
- `rw` defines a read & write password that allows for SNMP Get & Set Requests from NMS to agent
- `acl` can be IPv6 or IPv4 and is applied to **incoming** traffic to the agent

Define an NMS to receive traps from agent:
- `Router(config)#snmp-server host <address> traps version 1 <password>`
- `address` is the IP or hostname of the NMS
- `password` is the community string shared with the NMS
  - Sent in **clear text**

### Version 2c

Define a community string:
- Same as SNMP version 1. See above.

Define an NMS to receive traps from agent:
- `Router(config)#snmp-server host <address> <traps|informs> version 2c <password>`
- Same as SNMP version 1 with the addition of an `inform` option
  - Same as `trap` but requires ACK from NMS for _error recovery_

### Version 3

Define a group:
- `Router(config)#snmp-server group <name> v3 <noauth|auth|priv> [write v1default] [access [ipv6] <acl>]`
- `name` is the name of the group
- `v1default` is the _default_ MIB view that gives access to the majority of the MIB
- `acl` is the same as SNMP versions 1 & 2c
- `noauth|auth|priv` differences:
  
  - | Keyword  | Integrity | Authentication | Encryption |
    | -------- | --------- | -------------- | ---------- |
    | `noauth` | Yes       | No             | No         |
    | `auth`   | Yes       | Yes            | No         |
    | `priv`   | Yes       | Yes            | Yes        |

Define a user:
- `Router(config)#snmp-server user <name> <group> v3 [auth <options>] [priv <options>]`
- `name` is the username
- `group` is the group for this user
- `auth` option is required only if user's group has `auth` **or** `priv` enabled
- `priv` oprion is required onyl if user's group has `priv` enabled

Define an NMS to receive traps from agent:
- `Router(config)#snmp-server host <address> version 3 <noauth|auth|priv> <username>`
- `address` is the IP or hostname of the NMS
- `username` is the username allows to receive traps
- `noauth|auth|priv` option must match the group option set for the `username`

### Troubleshooting

Common Issues:
1. NMS cannot set MIB on server?
   1. SNMP v1/v2c: Is the community string being used read-only?
   2. SNMPv3: `snmp-server user` missing `auth` and/or `priv` to match the group's setting?
2. NMS cannot connect to server?
   1. ACL blocking UDP port 161/162?

Example troubleshooting output:

```
Router#show snmp chassis
69696969
```

```
Router#show snmp contact
Dude Bro (420) 420-6969
```

```
Router#show snmp location
The sun dude...
```

```
Router#show snmp community

Community name: ILMI
Community Index: cisco0
Community Securityname: ILMI
storage-type:  read only      active

Community name: keep-it-safe
Community Index: cisco1
Community Securityname: keep-it-safe
storage-type:  read write      active

Community name: keep-it-safe@1
Community Index: cisco2
Community Securityname: keep-it-safe@1
storage-type:  read write      active

Community name: keep-it-secret
Community Index: cisco2
Community Securityname: keep-it-secret
storage-type:  read only      active

Community name: keep-it-secret@1
Community Index: cisco3
Community Securityname: keep-it-secret@1
storage-type:  read only      active
```

```
Router#show snmp host
Notification host: 1.1.1.1    udp-port: 162    type: trap
user: this-is-old-school    security model: v1

Notification host: 192.168.69.69    udp-port: 162    type: trap
user: this-is-secret-i-think    security model: v2c

Notification host: 10.10.10.100    udp-port: 162    type: trap
user: dude    security model: v3 priv
```

## IP SLA

Define an SLA:
- `Router(config)#ip sla <num>`

Enable an SLA forever starting now:
- `Router(config)#ip sla schedule <num> start-time now life forever`
- `num` is the SLA number configured locally

Define an ICMP Echo SLA:
- `Router(config-ip-sla)#icmp-echo <ip>`
- `ip` is the target you wish to marry this SLA to
- Define the frequency in between ICMP Echo requests:
  - `Router(config-ip-sla-echo)#frequency <seconds>`
  - Default is 60 seconds

### Troubleshooting

Common Issues:
1. No statistics?
   1. Is SLA responder enabled on the other device?
      1. Not needed from some SLAs (eg ICMP Echo SLA)
2. Statistics show failures but history show success?
   1. Increase the number of buckets?
   2. Increase frequency?

Example troubleshooting output:

```
Router#show ip sla summary
IPSLAs Latest Operation Summary
Codes : * active, ^ inactive, ~ pending

ID     	Type       	Destination  	Stats  	Return Code  	Last Run
*6969  	icmp-echo  	10.10.10.10  	-      	timeout      	4 seconds ago
```

```
Router#show ip sla statistics
IPSLAs Latest Operation Statistic

IPSLA operation id:6969
		  Latest RTT: NoConnection/Busy/Timeout
Latest operation start time: Sat Feb 15 17:11:26 PST 2020
Latest operation return code: timeout
Number of successes: 0
Number of failures: 1
Operation time to live: forever
```

```
Router#show ip sla configuration
IP SLAs Infrastructure Engine- III
Entry Number: 6969
Owner:
Tag:
Operation timeout(milliseconds): 5000
Type of operation to perform: icmp-echo
Target address/Source address: 10.10.10.10/0.0.0.0
Type of Service parameter: 0x0
Request Size (ARR data portion): 28
verify data: No
vrf Name:
Schedule:
	 Operation frequency (seconds): 69 (not considered if randomly schedule)
	 Next Scehduled Start Time: Start time already passed
	 Group Scheduled : FALSE
	 Randomly Scheduled : FALSE
	 Life (seconds) : forever
	 Entry Ageout (seconds): Never
	 Recurring (Starting everyday) : FALSE
	 Status of entry (SNMP rowstatus): Active
Threshold (milliseconds): 5000
destribution Statistics:
	 Number of statistic hours kept: 2
	 Number of statistic distribution buckets kept: 1
	 Statistic distribution interval (milliseconds): 20
Enhanced history:
History Statistics:
	 Number of history Lives kept: 0
	 Number of history Buckets kept: 15
	 History Filter Type: None
```

## SPAN

### Local

Define sources:
- `Switch(config)#monitor session <num> source <source> <rx|tx|both>`
- `num` must match on _all_ local source ports in this SPAN
- `source` can be a VLAN or physical interface (which can also be a trunk)
- `both` is the default SPAN direction

Define destinations:
- `Switch(config)#monitor session <num> destination interface <dest>`
- `num` must match on _all_ local destination ports and must match _all_ source ports in SPAN
- `dest` must be a local interface number unless using remote SPAN
  - Destination port is no longer considered in the switch's CAM table for unicast frames and has no source MAC tied to it in the CAM table 

### Remote
- Must define local VLAN as the destination for SPAN
- The _same_ VLAN ID must be defined on _all_ switchs in between local source and remote destination port

### Troubleshooting

Common Issues:
1. Local SPAN...
   1. Destination...
      1. Port can be used in only one SPAN at a time
      2. Port cannot be a source SPAN port
   2. Source...
      1. Can have mutliple ports **or** VLANs
         2. Cannot have a mix of VLANs and ports (one or the other) 
      3. Can be an EtherChannel or trunk port
      4. Can have a mix of Rx/Tx/Both as long as they are applied to _different_ SPAN sources

Example troubleshooting output:

```
Switch#show monitor

Session 1
----------
Type               : Local Session
Source Ports       :
    Boths          : Fa0/1-2
    rx Only        : Gi0/1-2
    tx Only        : Fa0/11
Destination Ports  : Fa0/9-10
Encapsulation      : Native
Ingress            : Disabled
```

