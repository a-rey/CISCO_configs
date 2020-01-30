# CISCO Configuration Guide

[TOC]

<!-- ToC generated using https://imthenachoman.github.io/nGitHubTOC/ -->

## VLANs

```
Switch(config-if)#switchport access vlan <x>
```

_or_

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
Number of existing VLANs		: 3
Number of existing VTP VLANs		: 3
Number of existing extended VLANs	: 0
```

Defined VLANs in *vlan.dat*:

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

## Trunks

### Dynamic Trunking Protocol

```
Switch(config-if)#switchport mode <access|trunk|dynamic auto|dynamic desirable>
```

| Admin Mode          | Access      | Dynamic Auto | Trunk       | Dynamic Desirable |
| ------------------- | ----------- | ------------ | ----------- | ----------------- |
| `access`            | Access      | Access       | !!! BAD !!! | Access            |
| `dynamic auto`      | Access      | Access       | Trunk       | Trunk             |
| `trunk`             | !!! BAD !!! | Trunk        | Trunk       | Trunk             |
| `dynamic desirable` | Access      | Trunk        | Trunk       | Trunk             |

```
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk encapsulation <dot1q|isl|nonegotiate>
Switch(config-if)#switchport trunk allowed vlan <add|all|except|remove> <x>
Switch(config-if)#switchport trunk native vlan <x>
```

*NOTE:* Disable all auto negotiation (trunk protocol negotiation **and** operational mode):

```
Switch(config-if)#switchport nonegotiate
```

### Troubleshooting

List current trunks:

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

## Spanning Tree Protocol

Manually specifying root/secondary switch in a given VLAN **or** manual priority:

```
Switch(config)#spanning-tree vlan <x> root <primary|secondary>
Switch(config)#spanning-tree vlan <x> priority <y>
```

Manually specifying port (all VLANs) **or** per VLAN cost:

```
Switch(config)#interface <x>
Switch(config-if)#spanning-tree vlan <x> cost <y>
Switch(config-if)#spanning-tree cost <x>
```

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

Display per VLAN _local_ bridge ID, timers, and protocol:

```
Switch#show spanning-tree bridge
                                                   Hello  Max  Fwd
Vlan                         Bridge ID              Time  Age  Dly  Protocol
---------------- --------------------------------- -----  ---  ---  --------
VLAN0001         28673(28672,    1) 0019.e86a.1180    2    20   15  ieee
VLAN0022         32790(32768,   22) 0019.e86a.1180    2    20   15  ieee
VLAN0045         32813(32768,   45) 0019.e86a.1180    2    20   15  ieee
```

Display per VLAN root bridge ID, timers, and _local_ root port/root cost:

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

Display per VLAN interface costs, states, roles, types, and priority:

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

















