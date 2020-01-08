* [TOC](ToC-Contrail-Fabric-Management)

# 6 BMS Multi-homing

## 6.1 Create BMS inventory

Create bonding interface when create BMS inventory.
![Figure 6.1 BMS inventory](Solution-CFM/F6-1.png)


## 6.2 Launch BMS

Launch BMS instance with bonding interface.
![Figure 6.2 Create BMS instance](Solution-CFM/F6-2.png)

Virtual port group is created for BMS instance.
![Figure 6.3 VPG for BMS instance](Solution-CFM/F6-3.png)

* [Multi-homing BMS configuration on leaf-1](A3-L2-gateway-configuration#a32-multi-homing-bms-on-leaf-1)
* [Multi-homing BMS configuration on leaf-2](A3-L2-gateway-configuration#a33-multi-homing-bms-on-leaf-2)


## 6.3 Bonding configuration on BMS

Here is an example of bonding configuration on CentOS 7 BMS.
```
[root@bms-dh ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens3
DEVICE=ens3
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no

[root@bms-dh ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens4
DEVICE=ens4
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no

[root@bms-dh ~]# cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
TYPE=Bond
ONBOOT=yes
BOOTPROTO=dhcp
BONDING_MASTER=yes
BONDING_OPTS="mode=4 miimon=100"
NM_CONTROLLED=no
```


## 6.4 Connectivity

LACP state on leaf-1.
```
root@vqfx-leaf-1> show lacp interfaces               
Aggregated interface: ae0
    LACP state:       Role   Exp   Def  Dist  Col  Syn  Aggr  Timeout  Activity
      xe-0/0/3       Actor    No    No   Yes  Yes  Yes   Yes     Fast    Active
      xe-0/0/3     Partner    No    No   Yes  Yes  Yes   Yes     Slow    Active
    LACP protocol:        Receive State  Transmit State          Mux State 
      xe-0/0/3                  Current   Slow periodic Collecting distributing

{master:0}
root@vqfx-leaf-1> show lacp statistics interfaces 
Aggregated interface: ae0
    LACP Statistics:       LACP Rx     LACP Tx   Unknown Rx   Illegal Rx 
      xe-0/0/3                 107         704            0            0

{master:0}
```

LACP state on leaf-2.
```
root@vqfx-leaf-2> show lacp interfaces 
Aggregated interface: ae0
    LACP state:       Role   Exp   Def  Dist  Col  Syn  Aggr  Timeout  Activity
      xe-0/0/3       Actor    No    No   Yes  Yes  Yes   Yes     Fast    Active
      xe-0/0/3     Partner    No    No   Yes  Yes  Yes   Yes     Slow    Active
    LACP protocol:        Receive State  Transmit State          Mux State 
      xe-0/0/3                  Current   Slow periodic Collecting distributing

{master:0}
root@vqfx-leaf-2> show lacp statistics interfaces 
Aggregated interface: ae0
    LACP Statistics:       LACP Rx     LACP Tx   Unknown Rx   Illegal Rx 
      xe-0/0/3                 212         708            0            0

{master:0}
```

```
root@vqfx-leaf-1> show ethernet-switching table 

Ethernet switching table : 5 entries, 5 learned
Routing instance : default-switch
   Vlan                MAC                 MAC      Logical                Active
   name                address             flags    interface              source
   bd-7                02:e8:f6:db:57:48   D        vtep.32769             10.6.11.7
   bd-7                52:54:00:5b:50:a5   DLR      ae0.0
   vlan-11             52:54:00:59:de:06   D        xe-0/0/4.0
   vlan-11             52:54:00:73:33:ea   D        xe-0/0/4.0
   vlan-11             52:54:00:d6:74:56   D        xe-0/0/4.0
```

```
root@vqfx-leaf-1> show route table default-switch.evpn.0 
......
2:10.6.0.11:7999::7::52:54:00:5b:50:a5/304 MAC/IP        
                   *[EVPN/170] 00:09:29
                      Indirect
2:10.6.0.12:7999::7::52:54:00:5b:50:a5/304 MAC/IP        
                   *[BGP/170] 00:09:27, localpref 100, from 10.6.0.31
                      AS path: I, validation-state: unverified
                    > to 10.6.20.0 via xe-0/0/0.0
                    [BGP/170] 00:09:28, localpref 100, from 10.6.0.32
                      AS path: I, validation-state: unverified
                    > to 10.6.20.0 via xe-0/0/0.0
......
```

The ECMP route on vrouter for multi-homing BMS.
![Figure 6.4 Vrouter ECMP route](Solution-CFM/F6-4.png)

