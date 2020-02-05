

# CISCO Configuration Guide

[TOC]

<!-- ToC generated using https://imthenachoman.github.io/nGitHubTOC/ -->

## VLANs

Define on an interface:

```
Switch(config-if)#switchport access vlan <x>
```

Define globally:

```
Switch(config)#vlan <x>
Switch(config-vlan)#exit 
```

### Voice VLAN

*NOTE:* must have CDP enabled on port

```
Switch(config-if)#switchport voice vlan <x|none|untagged|dot1p>
```

### Troubleshooting

Overview:

```
Switch#show vlan summary
Number of existing VLANs          : 3
Number of existing VTP VLANs      : 3
Number of existing extended VLANs	: 0
```

Defined VLANs:

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

Specific VLAN details:

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

MAC addresses per VLAN per port:

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

```
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk encapsulation <dot1q|isl|nonegotiate>
Switch(config-if)#switchport trunk allowed vlan <add|all|except|remove> <x>
Switch(config-if)#switchport trunk native vlan <x>
```

### Dynamic Trunking Protocol

```
Switch(config-if)#switchport mode <dynamic auto|dynamic desirable>
```

| Admin Mode          | Access      | Dynamic Auto | Trunk       | Dynamic Desirable |
| ------------------- | ----------- | ------------ | ----------- | ----------------- |
| `access`            | Access      | Access       | !!! BAD !!! | Access            |
| `dynamic auto`      | Access      | Access       | Trunk       | Trunk             |
| `trunk`             | !!! BAD !!! | Trunk        | Trunk       | Trunk             |
| `dynamic desirable` | Access      | Trunk        | Trunk       | Trunk             |

*NOTE:* Disable all auto negotiation (trunk protocol negotiation **and** operational mode):

```
Switch(config-if)#switchport nonegotiate
```

### Troubleshooting

List current trunks and allowed VLANs:

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

Check interface VLAN modes:

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

Set VTP domain/password (both unset by default):

_NOTE:_ Domain and password must match (case sensitive) on all devices in VTP domain

```
Switch(config)#vtp domain <x>
Switch(config)#vtp password <x>
```

Set VTP mode (`server` mode is the default):

| Function                                 | `server` | `client` | `transparent` | `off` |
| ---------------------------------------- | -------- | -------- | ------------- | ----- |
| Only sends VTP messages on trunks        | Y        | Y        | Y             | N     |
| Allows VLAN database changes             | Y        | N        | Y             | Y     |
| Can use standard range VLANs (1-1005)    | Y        | Y        | Y             | Y     |
| Can use extended range VLANs (1006-4095) | N        | N        | Y             | Y     |

```
Switch(config)#vtp mode <server|client|transparent|off>
```

Set VTP version (version 1 is the default):

```
Switch(config)#vtp version <1|2|3>
```

Enable VTP pruning (disabled by default):

```
Switch(config)#vtp pruning
```

### Troubleshooting

Show current VTP status:

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

- `root`: priority will be 24576 **or** the next lowest multiple of 4096 if 24576 is not low enough to become root _now_
- `secondary`: priority will be 28672 
- _NOTE:_ default base priority is 32768 (VLAN ID is added to this value)

```
Switch(config)#spanning-tree vlan <x> root <primary|secondary>
Switch(config)#spanning-tree vlan <x> priority <y>
```

Manually specifying port cost for all VLANs **or** per VLAN cost:

```
Switch(config)#interface <x>
Switch(config-if)#spanning-tree vlan <x> cost <y>
Switch(config-if)#spanning-tree cost <x>
```

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

```
Switch(config)#spanning-tree portfast default
```

Enable **or** disable per interface:

```
Switch(config)#interface <x>
Switch(config-if)#spanning-tree portfast <disable>
```

### BPDU Guard

Enable globally:

_NOTE:_ only gets enabled on interfaces with PortFast already enabled

```
Switch(config)#spanning-tree portfast bpduguard default
```

Enable **or** disable per interface:

```
Switch(config)#interface <x>
Switch(config-if)#spanning-tree bpduguard <enable|disable>
```

### Troubleshooting

Display VLAN STP overview:

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

Display per VLAN _local_ bridge ID settings:

```
Switch#show spanning-tree bridge
                                                   Hello  Max  Fwd
Vlan                         Bridge ID              Time  Age  Dly  Protocol
---------------- --------------------------------- -----  ---  ---  --------
VLAN0001         28673(28672,    1) 0019.e86a.1180    2    20   15  ieee
VLAN0022         32790(32768,   22) 0019.e86a.1180    2    20   15  ieee
VLAN0045         32813(32768,   45) 0019.e86a.1180    2    20   15  ieee
```

Display per VLAN root bridge ID details:

```
Switch#show spanning-tree root
                                        Root    Hello Max Fwd
Vlan                   Root ID          Cost    Time  Age Dly  Root Port
---------------- -------------------- --------- ----- --- ---  ------------
VLAN0001         24577 0019.e86a.2280         4    2   20  15  Gi0/1
VLAN0022         32790 0019.e86a.1180         0    2   20  15
VLAN0045         32813 0019.e86a.1180         0    2   20  15
```

Display per VLAN interface portFast setting:

```
Switch#show spanning-tree interface FastEthernet 0/1 portFast
VLAN0001                                       disabled
VLAN0002                                       disabled
VLAN0045                                       disabled
```

Display per VLAN interface STP settings:

```
Switch#show spanning-tree interface FastEthernet 0/1
Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0001            Desg FWD 19        128.1    P2p
VLAN0002            Desg FWD 19        128.1    P2p
VLAN0045            Desg FWD 19        128.1    P2p
```

Check for BPDU Guard status:

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

- ```
  Switch(config)#interface range <x>
  Switch(config-if-range)#channel-group <x> mode on
  ```

Define dynamic channel (PAgP - Cisco Proprietary):

|             | `on`        | `desirable` | `auto`      |
| ----------- | ----------- | ----------- | ----------- |
| `on`        | Y           | !!! BAD !!! | !!! BAD !!! |
| `desirable` | !!! BAD !!! | Y           | Y           |
| `auto`      | !!! BAD !!! | Y           | N           |

- ```
  Switch(config)#interface range <x>
  Switch(config-if-range)#channel-group <x> mode <desirable|auto>
  ```

Define dynamic channel (LACP - IEEE 802.3ad):

|           | `on`        | `active`    | `passive`   |
| --------- | ----------- | ----------- | ----------- |
| `on`      | Y           | !!! BAD !!! | !!! BAD !!! |
| `active`  | !!! BAD !!! | Y           | Y           |
| `passive` | !!! BAD !!! | Y           | N           |

- ```
  Switch(config)#interface range <x>
  Switch(config-if-range)#channel-group <x> mode <passive|active>
  ```

### Troubleshooting

Display status:

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

## OSPF

Enable with a process ID:
- `Router(config)#router ospf <process-id>`
  - _NOTE:_ `process-id` needs to be **locally** unique

Define max number of OSPF routes used for equal cost load balancing:
- `Router(config-router)#maximum-paths <max>`
  - _NOTE:_ default `max` is 4
  - _NOTE:_ set `max` to 1 to disable load balancing

Define a passive OSPF interface:
- `Router(config-router)#passive-interface <interface>`
- _NOTE:_ can also enable globally:
  - `Router(config-router)#passive-interface default`
  - `Router(config-router)#no passive-interface <interface>`

Specify OSPF to advertise a default route:
- `Router(config-router)#default-information originate [always]`
  - _NOTE:_ `always` option means advertise a default route even if one does not exist

Specify interfaces to advertise/learn on:
- `Router(config-router)#network <network> <wildcard> area <area>`
  - _NOTE:_ if an interface matches 2 different `network` statements, the first one that was configured is used as the area and mask
- `Router(config-if)#ip ospf <process-id> area <area>`
  - _NOTE:_ interface ospf area configuration is prefered over the `network` command if both are configured and match an interface

Manually specify Router ID (RID):
- `Router(config-router)#router-id <rid>`
  - _NOTE:_ RID selection priority ranking:
    1. `router-id` command value
    2. Highest Loopback interface IP (does not need to be OSPF enabled!)
    3. Highest interface IP (does not need to be OSPF enabled!)
  - _NOTE:_ changing RID at runtime requires OSPF neighbor discovery to be restarted:
    - `Router# clear ip ospf process`
    - `Router# reload`

Adjusting timers:
- Hello timer: `Router(config-if)#ip ospf hello-interval <seconds>`
  - _NOTE:_ default is 10 seconds for Ethernet interfaces
  - _NOTE:_ default is 30 seconds for Serial interfaces
- Dead timer: `Router(config-if)#ip ospf dead-interval <seconds>`
  - _NOTE:_ default is 4 * hello timer value

### Cost

Adjusting interface cost:
- Manually:
  
  - `Router(config-if)#ip ospf cost <cost>`
- By interface bandwidth:
  - `Router(config-if)#bandwidth <bandwidth in Kbps>`
    - _NOTE:_ find interface default bandwidth in Kbps:
      - `Router#show interface <int>`
- By reference bandwidth:
  - `Router(config-router)#auto-cost reference-bandwidth <bandwidth in Mbps>`
    - _NOTE:_ default is 100000 bps or 100 Mbps
- _NOTE:_ cost equation: 
  
  - `cost = (reference bandwidth / interface bandwidth)`
- _NOTE:_ default costs:

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

### Troubleshooting

OSPF issues:

1. No OSPF neighbors
   1. Authentication values incorrect?
   2. Local interfaces not in an up&up state?
   3. OSPF neighbor interfaces not in the same subnet?
   4. ACL blocking routing protocol packets to 224.0.0.5 and/or 224.0.0.6?
   5. Non-matched hello/dead timer values?
   6. Non-unique RIDs?
   7. Areas do not match?
   8. MTUs do not match?
2. Bad area design
   1. Interfaces in the same subnet but also in different areas?
   2. Current area not touching an area border router (ABR) to have an interface in the backbone area (area 0)?
3. Passive interfaces
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

## EIGRP

Enable with an ASN:
- `Router(config)#router eigrp <asn>`
  - _NOTE:_ `asn` needs to be **globally** unique

Define max number of EIGRP routes used for _equal cost_ load balancing:
- `Router(config-router)#maximum-paths <max>`
  - _NOTE:_ default `max` is 4
  - _NOTE:_ set `max` to 1 to disable load ballancing

Enable _unequal cost_ load balancing:
- `Router(config-router)#variance <x>`
  - _NOTE:_ applies to _all_ EIGRP routes with a sucessor (S) and feasible sucessor (FS) in the topology table
  - _NOTE:_ allows for FS routes with a `FD(FS) < (variance * FD(S))` to be added to the routing table
  - _NOTE:_ does **not** add other non-FS routes into the routing table even if they meet the criteria

Define a passive EIGRP interface:
- `Router(config-router)#passive-interface <interface>`
- _NOTE:_ can also enable globally:
  - `Router(config-router)#passive-interface default`
  - `Router(config-router)#no passive-interface <interface>`

Specify interfaces to advertise/learn on:
- `Router(config-router)#network <network> <wildcard>`
- _NOTE:_ Can also configure using classfull network ID:
  - `Router(config-router)#network <classfull-network>`

Manually specify Router ID (RID):
- `Router(config-router)#eigrp router-id <rid>`
  - _NOTE:_ RID selection priority ranking:
    1. `router-id` command value
    2. Highest Loopback interface IP (does not need to be EIGRP enabled!)
    3. Highest interface IP (does not need to be EIGRP enabled!)

Enable auto-summarization:
- `Router(config-router)#auto-summary`
  - _NOTE:_ not enabled by default

Define timers:
- Hello timer: `Router(config-if)#ip hello-interval eigrp <asn> <seconds>`
  - _NOTE:_ default for Ethernet interfaces is 5 seconds
  - _NOTE:_ default for Serial interfaces is 60 seconds
- Hold timer: `Router(config-if)#ip hold-time eigrp <asn> <seconds>`
  - _NOTE:_ default is 3 * hello timer value
  - _NOTE:_ value does _not_ change in sync when changing the hello timer directly

### Metric

Metric equation with default K values:
- `metric = 256 * (((10^7) / smallest_bandwidth) + cumulative_delay)`
- Default K values:
  - K1 (Bandwidth) = 1
  - K2 (Load) = 0
  - K3 (Delay) = 1
  - K4 (Reliability) = 0
  - K5 (MTU) = 0

Modify bandwidth:
- `Router(config-if)#bandwidth <bandwidth in Kbps>`
  - _NOTE:_ default bandwidth can be seen with `Router#show int <int>`
  - _NOTE:_ is enabled with default K values

Modify delay:
- `Router(config-if)#delay <delay in 10s of microseconds>`
  - _NOTE:_ default delay can be seen with `Router#show int <int>`
  - _NOTE:_ is enabled with default K values

### Troubleshooting

EIGRP issues:

1. No EIGRP neighbors
   1. Authentication values incorrect?
   2. Local interfaces not in an up&up state?
   3. EIGRP neighbor interfaces not in the same subnet?
   4. ACL blocking routing protocol packets to 224.0.0.10?
   5. EIGRP neighbors not in same ASN?
   6. K-values do not match?
2. EIGRP neighbors can't stay up
   1. Hello timer values > neighbor hold timers?
3. Passive interfaces
   1. `show ip eigrp interfaces` shows only active interfaces
      - Use `show ip protocols` to see passive interfaces
4. Auto summarization issues
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





















