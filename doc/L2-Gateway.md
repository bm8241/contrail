# 1 Overview

L2 gateway provides L2 connectivity between bare-metal server (BMS) and overlay. It requires the support to BGP EVPN and VXLAN. In case of CLOS underlay, ToR is normally the L2 gateway. QFX5100 is an example.


# 2 Control Plane

## 2.1 BGP peering

Contrail CNs (control node) have to peer with the ToR who has BMS connected. Create configuration BGP router and physical router, and allow Contrail to manage it, then Contrail service, DM (device manager), will push configuration to the ToR to create BGP peering.

Here is an example.
```
config create bgp-router qfx1 \
    --address 172.16.0.201 \
    --asn 201 \
    --vendor Juniper \
    --peer controller

config create physical-router qfx1 \
    --vendor Juniper \
    --model qfx5100 \
    --management-address 10.84.29.251 \
    --loopback-address 172.16.0.201 \
    --managed \
    --username root \
    --password Juniper123 \
    --virtual-router tsn \
    --bgp-router qfx1
```

Here is the configuration pushed to QFX by DM.
[ edit groups __contrail__ ]
```
interfaces {
    lo0 {
        unit 0 {
            family inet {
                address 172.16.0.201/32 {
                    primary;
                    preferred;
                }
            }
        }
    }
}
routing-options {
    router-id 172.16.0.201;
    route-distinguisher-id 172.16.0.201;
    autonomous-system 201;
    resolution {
        rib bgp.rtarget.0 {
            resolution-ribs inet.0;
        }
    }
}
protocols {
    bgp {
        group _contrail_asn-201 {
            type internal;
            local-address 172.16.0.201;
            hold-time 90;
            family evpn {
                signaling;
            }
            family route-target;
        }
        group _contrail_asn-201-external {
            type external;
            multihop;
            local-address 172.16.0.201;
            hold-time 90;
            family evpn {               
                signaling;
            }
            family route-target;
            neighbor 172.16.201.3 {
                peer-as 64512;
            }
        }
    }
}
policy-options {
    community _contrail_switch_policy_ members target:201:1;
}
switch-options {
    vtep-source-interface lo0.0;
}
```

Check BGP peering on QFX.
```
qfx1> show bgp group _contrail_asn-201-external 
Group Type: External                               Local AS: 201
  Name: _contrail_asn-201-external Index: 2      Flags: <Export Eval>
  Options: <Multihop>
  Holdtime: 0
  Total peers: 1        Established: 1
  172.16.201.3+58455
  bgp.rtarget.0: 4/4/4/0
  bgp.evpn.0: 5/5/5/0
  default-switch.evpn.0: 5/5/5/0
  __default_evpn__.evpn.0: 0/0/0/0
```

## 2.2 BMS

To provide connectivity between BMS and overly, the physical interface and logical interface that BMS connects to have to be configured.

Here is an example.
```
config create physical-interface xe-0/0/1 \
    --physical-router qfx1

config create logical-interface \
    --physical-router qfx1 \
    --type server \
    --network red \
    --mac f2:f9:d9:0e:80:3c \
    xe-0/0/1.100
```

After creating logical interface, DM will push configuration to QFX.
```
interfaces {
    xe-0/0/1 {
        flexible-vlan-tagging;
        encapsulation extended-vlan-bridge;
        unit 100 {
            vlan-id 100;
        }
    }
}
protocols {
    evpn {
        vni-options {
            vni 5 {
                vrf-target target:64512:8000003;
            }
        }
        encapsulation vxlan;
        multicast-mode ingress-replication;
        extended-vni-list all;
    }
}
policy-options {
    policy-statement _contrail_red-l2-5-import {
        term _contrail_switch_policy_ {
            from community _contrail_switch_policy_;
            then accept;
        }
        term t1 {
            from community _contrail_target_64512_8000003;
            then accept;
        }
    }
    policy-statement _contrail_switch_export_policy_ {
        term t1 {
            then {
                community add _contrail_switch_export_community_;
            }
        }
    }
    community _contrail_switch_export_community_ members target:64512:8000003;
    community _contrail_target_64512_8000003 members target:64512:8000003;
    community _contrail_switch_policy_ members target:201:1;
}
switch-options {
    vtep-source-interface lo0.0;
    route-distinguisher 172.16.0.201:1;
    vrf-import _contrail_red-l2-5-import;
    vrf-export _contrail_switch_export_policy_;
    vrf-target {                        
        target:201:1;
        auto;
    }
}
vlans {
    contrail_red-l2-5 {
        interface xe-0/0/1.100;
        vxlan {
            vni 5;
        }
    }
}
```


## 2.3 BUM traffic

Once logical interface is created on a VN, that VN will be pulled by the TSN associated to the physical router where tha logical interface is located. Then vrouter on TSN will create a L2 route for BUM traffic.

This EVPN type 3 BUM route is advertised from vrouter to CN who in turn advertises it to QFX.
```
qfx1> show route table bgp.evpn.0               
......
3:192.168.2.131:2::5::192.168.2.131/248 IM            
                   *[BGP/170] 00:01:05, MED 200, localpref 100, from 192.168.2.130
                      AS path: 64512 ?, validation-state: unverified
                    > to 192.168.2.131 via irb.300
3:192.168.2.131:3::7::192.168.2.131/248 IM            
                   *[BGP/170] 00:01:05, MED 200, localpref 100, from 192.168.2.130
                      AS path: 64512 ?, validation-state: unverified
                    > to 192.168.2.131 via irb.300
......
```

When BMS sends DHCP request to QFX, the request is forwarded to TSN who will respond based on port configuration (specified MAC and allocated IP address).


## 2.4 Subnet

Other than BMS, Contrail also provides connectivity between existing underlay subnet/VLAN and overlay. In this case, address of underlay server is not managed by Contrail. TSN is not required to provide DHCP or DNS service. MAC address is not required either.

Create physical router without assiciating with TSN.
```
config create physical-router qfx1 \
    --vendor Juniper \
    --model qfx5100 \
    --management-address 10.84.29.251 \
    --loopback-address 172.16.0.201 \
    --managed \
    --username root \
    --password Juniper123 \
    --bgp-router qfx1
```
In case the same QFX is used to connect both BMS and subnet, TSN is required.

Create a VN with the same subnet address range as underlay. To avoid address collision, a separated allocation pool from underlay can be specified.

Create logical interface for subnet.
```
config create logical-interface \
    --physical-router qfx1 \
    --type subnet \
    --subnet 10.1.1.0/24 \
    --network red \
    xe-0/0/1.100
```

Now, create a VM and it shall be able to connect to servers on underlay subnet.

```
qfx1> show route table bgp.evpn.0 

bgp.evpn.0: 4 destinations, 4 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:192.168.2.18:2::5::02:8b:c3:b3:ae:91/304 MAC/IP        
                   *[BGP/170] 00:00:53, MED 100, localpref 100, from 192.168.2.130
                      AS path: 64512 ?, validation-state: unverified
                    > to 172.16.1.46 via et-0/0/49.0
2:192.168.2.18:2::5::02:8b:c3:b3:ae:91::10.1.1.249/304 MAC/IP        
                   *[BGP/170] 00:00:53, MED 100, localpref 100, from 192.168.2.130
                      AS path: 64512 ?, validation-state: unverified
                    > to 172.16.1.46 via et-0/0/49.0
3:192.168.2.18:2::5::192.168.2.18/248 IM            
                   *[BGP/170] 00:00:53, MED 200, localpref 100, from 192.168.2.130
                      AS path: 64512 ?, validation-state: unverified
                    > to 172.16.1.46 via et-0/0/49.0
```


