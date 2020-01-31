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

```
Switch(config)#interface range <x>
Switch(config-if-range)#channel-group <x> mode on
```

Define dynamic channel (PAgP):

|             | `on`        | `desirable` | `auto`      |
| ----------- | ----------- | ----------- | ----------- |
| `on`        | Y           | !!! BAD !!! | !!! BAD !!! |
| `desirable` | !!! BAD !!! | Y           | Y           |
| `auto`      | !!! BAD !!! | Y           | N           |

```
Switch(config)#interface range <x>
Switch(config-if-range)#channel-group <x> mode <desirable|auto>
```

Define dynamic channel (LACP):

|           | `on`        | `active`    | `passive`   |
| --------- | ----------- | ----------- | ----------- |
| `on`      | Y           | !!! BAD !!! | !!! BAD !!! |
| `active`  | !!! BAD !!! | Y           | Y           |
| `passive` | !!! BAD !!! | Y           | N           |

```
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

















