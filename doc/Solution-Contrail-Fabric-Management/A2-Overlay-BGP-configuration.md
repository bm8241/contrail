* [TOC](ToC-Contrail-Fabric-Management)

## A.2 Overlay BGP configuration

### A.2.1 Spine-1 with RR
```
set version 18.1R1.9
set groups __contrail_basic__ snmp community public authorization read-only
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then community add COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then accept
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term100 then accept
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from community COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 then reject
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then as-path-prepend "9999 9999 9999"
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then accept
set groups __contrail_basic__ policy-options community COM-MAINTENANCE members 9999:9999
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.21
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.21
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.21
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 cluster 0.0.0.100
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.11 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.12 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.11.1 peer-as 64512
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.21:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_type5__ routing-options forwarding-table chained-composite-next-hop ingress evpn
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 from protocol direct
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 then accept
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 from protocol static
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 then accept
```

```
root@vqfx-spine-1> show bgp summary 
Groups: 2 Peers: 7 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.rtarget.0        
                       6          6          0          0          0          0
inet.0               
                      35         10          0          0          0          0
bgp.evpn.0           
                       3          3          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.6.0.11             64512       1516       1503       0       0    11:26:25 Establ
  bgp.rtarget.0: 1/1/1/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.12             64512       1516       1503       0       0    11:26:26 Establ
  bgp.rtarget.0: 1/1/1/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.11.1             64512       1378       1501       0       0    11:26:17 Establ
  bgp.rtarget.0: 4/4/4/0
  bgp.evpn.0: 3/3/3/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.20.1             64011       1752       1736       0       0    13:11:02 Establ
  inet.0: 4/10/10/0
10.6.20.3             64012       1750       1741       0       0    13:10:58 Establ
  inet.0: 2/9/9/0
10.6.30.1             64031       1760       1741       0       0    13:10:56 Establ
  inet.0: 2/8/8/0
10.6.30.3             64032       1760       1741       0       0    13:10:54 Establ
  inet.0: 2/8/8/0
```

### A.2.2 Spine-2 with RR
```
set version 18.1R1.9
set groups __contrail_basic__ snmp community public authorization read-only
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then community add COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then accept
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term100 then accept
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from community COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 then reject
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then as-path-prepend "9999 9999 9999"
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then accept
set groups __contrail_basic__ policy-options community COM-MAINTENANCE members 9999:9999
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.22
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.22
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.22
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 cluster 0.0.0.100
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.11 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.12 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.11.1 peer-as 64512
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.22:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_type5__ routing-options forwarding-table chained-composite-next-hop ingress evpn
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 from protocol direct
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 then accept
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 from protocol static
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 then accept
```

```
root@vqfx-spine-2> show bgp summary 
Groups: 2 Peers: 7 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.rtarget.0        
                       6          6          0          0          0          0
inet.0               
                      52         10          0          0          0          0
bgp.evpn.0           
                       3          3          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.6.0.11             64512       1523       1511       0       0    11:30:25 Establ
  bgp.rtarget.0: 1/1/1/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.12             64512       1524       1511       0       0    11:30:23 Establ
  bgp.rtarget.0: 1/1/1/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.11.1             64512       1386       1509       0       0    11:30:17 Establ
  bgp.rtarget.0: 4/4/4/0
  bgp.evpn.0: 3/3/3/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.20.5             64011       1766       1746       0       0    13:14:59 Establ
  inet.0: 4/13/13/0
10.6.20.7             64012       1768       1753       0       0    13:14:55 Establ
  inet.0: 2/13/13/0
10.6.30.5             64031       1773       1752       0       0    13:14:53 Establ
  inet.0: 2/13/13/0
10.6.30.7             64032       1773       1751       0       0    13:14:51 Establ
  inet.0: 2/13/13/0
```

### A.2.3 MX-1 with RR
```
set version 18.3R1.9
set groups __contrail_basic__ snmp community public authorization read-only
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then community add COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then accept
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term100 then accept
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from community COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 then reject
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then as-path-prepend "9999 9999 9999"
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then accept
set groups __contrail_basic__ policy-options community COM-MAINTENANCE members 9999:9999
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.31
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.31
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_asn-64512 source-address 10.6.0.31
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_asn-64512 udp
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_asn-64512 destination-networks 10.6.11.0/24
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.31
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family inet-vpn unicast
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family inet6-vpn unicast
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export mpls_over_udp
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 cluster 0.0.0.100
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.11 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.12 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.21 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.22 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.11.1 peer-as 64512
set groups __contrail_overlay_bgp__ policy-options policy-statement mpls_over_udp term 1 then community add encap-udp
set groups __contrail_overlay_bgp__ policy-options policy-statement mpls_over_udp term 1 then accept
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options community encap-udp members 0x030c:64512:13
```

```
root@vmx-1> show bgp summary 
Groups: 2 Peers: 7 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.rtarget.0        
                      18         18          0          0          0          0
inet.0               
                      28         13          0          0          0          0
bgp.l3vpn.0          
                       2          2          0          0          0          0
bgp.l3vpn-inet6.0    
                       0          0          0          0          0          0
bgp.evpn.0           
                      13         13          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.6.0.11             64512        352        362       0       0     2:37:04 Establ
  bgp.rtarget.0: 4/4/4/0
  bgp.evpn.0: 2/2/2/0
  _contrail_red-l2-6.evpn.0: 2/2/2/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.12             64512        352        365       0       0     2:37:03 Establ
  bgp.rtarget.0: 6/6/6/0
  bgp.evpn.0: 2/2/2/0
  _contrail_red-l2-6.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.21             64512        347        350       0       0     2:37:05 Establ
  bgp.rtarget.0: 0/0/0/0
  bgp.evpn.0: 0/0/0/0
  _contrail_red-l2-6.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.22             64512        347        350       0       0     2:37:16 Establ
  bgp.rtarget.0: 0/0/0/0
  bgp.evpn.0: 0/0/0/0
  _contrail_red-l2-6.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.11.1             64512        345        362       0       0     2:37:12 Establ
  bgp.rtarget.0: 8/8/8/0
  bgp.l3vpn.0: 2/2/2/0
  bgp.l3vpn-inet6.0: 0/0/0/0
  bgp.evpn.0: 9/9/9/0
  _contrail_red-l3-6.inet.0: 1/1/1/0
  _contrail_red-l2-6.evpn.0: 4/4/4/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.30.0             64021      18376      18610       0       0 5d 19:59:00 Establ
  inet.0: 9/14/14/0
10.6.30.4             64022      18379      18626       0       0 5d 19:58:56 Establ
  inet.0: 4/14/14/0
```

### A.2.4 MX-2 with RR
```
set version 18.3R1.9
set groups __contrail_basic__ snmp community public authorization read-only
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then community add COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then accept
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term100 then accept
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from community COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 then reject
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then as-path-prepend "9999 9999 9999"
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then accept
set groups __contrail_basic__ policy-options community COM-MAINTENANCE members 9999:9999
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.32
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.32
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_asn-64512 source-address 10.6.0.32
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_asn-64512 udp
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_asn-64512 destination-networks 10.6.11.0/24
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.32
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family inet-vpn unicast
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family inet6-vpn unicast
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export mpls_over_udp
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 cluster 0.0.0.100
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.11 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.12 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.21 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.22 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.11.1 peer-as 64512
set groups __contrail_overlay_bgp__ policy-options policy-statement mpls_over_udp term 1 then community add encap-udp
set groups __contrail_overlay_bgp__ policy-options policy-statement mpls_over_udp term 1 then accept
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options community encap-udp members 0x030c:64512:13
```

```
root@vmx-2> show bgp summary 
Groups: 2 Peers: 7 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.rtarget.0        
                      18         18          0          0          0          0
inet.0               
                      30         13          0          0          0          0
bgp.l3vpn.0          
                       2          2          0          0          0          0
bgp.l3vpn-inet6.0    
                       0          0          0          0          0          0
bgp.evpn.0           
                      13         13          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.6.0.11             64512        352        363       0       0     2:36:18 Establ
  bgp.rtarget.0: 4/4/4/0
  bgp.evpn.0: 2/2/2/0
  _contrail_red-l2-6.evpn.0: 2/2/2/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.12             64512        350        369       0       0     2:36:14 Establ
  bgp.rtarget.0: 6/6/6/0
  bgp.evpn.0: 2/2/2/0
  _contrail_red-l2-6.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.21             64512        345        345       0       0     2:35:51 Establ
  bgp.rtarget.0: 0/0/0/0
  bgp.evpn.0: 0/0/0/0
  _contrail_red-l2-6.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.22             64512        344        346       0       0     2:36:06 Establ
  bgp.rtarget.0: 0/0/0/0
  bgp.evpn.0: 0/0/0/0
  _contrail_red-l2-6.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.11.1             64512        343        359       0       0     2:36:02 Establ
  bgp.rtarget.0: 8/8/8/0
  bgp.l3vpn.0: 2/2/2/0
  bgp.l3vpn-inet6.0: 0/0/0/0
  bgp.evpn.0: 9/9/9/0
  _contrail_red-l3-6.inet.0: 1/1/1/0
  _contrail_red-l2-6.evpn.0: 4/4/4/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.30.2             64021      18375      18608       0       0 5d 19:57:51 Establ
  inet.0: 9/15/15/0
10.6.30.6             64022      18375      18625       0       0 5d 19:57:47 Establ
  inet.0: 4/15/15/0
```

### A.2.5 Leaf-1
```
set version 18.1R1.9
set groups __contrail_basic__ snmp community public authorization read-only
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then community add COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then accept
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term100 then accept
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from community COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 then reject
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then as-path-prepend "9999 9999 9999"
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then accept
set groups __contrail_basic__ policy-options community COM-MAINTENANCE members 9999:9999
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.11
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.11
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.11
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.21 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.22 peer-as 64512
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.11:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
```

```
root@vqfx-leaf-1> show bgp summary 
Groups: 2 Peers: 4 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.rtarget.0        
                      14         12          0          0          0          0
inet.0               
                      26         11          0          0          0          0
bgp.evpn.0           
                       0          0          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.6.0.21             64512       1515       1526       0       0    11:32:08 Establ
  bgp.rtarget.0: 6/7/7/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.22             64512       1516       1526       0       0    11:32:07 Establ
  bgp.rtarget.0: 6/7/7/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.20.0             64021       1749       1763       0       0    13:16:46 Establ
  inet.0: 7/13/13/0
10.6.20.4             64022       1750       1768       0       0    13:16:41 Establ
  inet.0: 4/13/13/0
```

### A.2.6 Leaf-2
```
set version 18.1R1.9
set groups __contrail_basic__ snmp community public authorization read-only
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then community add COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term1 then accept
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE term term100 then accept
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from family evpn
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from community COM-MAINTENANCE
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 2
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 1
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 3
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 4
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 5
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 from nlri-route-type 6
set groups __contrail_basic__ policy-options policy-statement REJECT-MAINTENANCE-MODE term term1 then reject
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then as-path-prepend "9999 9999 9999"
set groups __contrail_basic__ policy-options policy-statement MAINTENANCE-MODE-underlay then accept
set groups __contrail_basic__ policy-options community COM-MAINTENANCE members 9999:9999
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.12
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.12
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.12
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.21 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.22 peer-as 64512
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.12:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
```

```
root@vqfx-leaf-2> show bgp summary 
Groups: 2 Peers: 4 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
bgp.rtarget.0        
                      14         12          0          0          0          0
inet.0               
                      30         12          0          0          0          0
bgp.evpn.0           
                       0          0          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.6.0.21             64512       1518       1529       0       0    11:33:10 Establ
  bgp.rtarget.0: 6/7/7/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.0.22             64512       1517       1528       0       0    11:33:06 Establ
  bgp.rtarget.0: 6/7/7/0
  bgp.evpn.0: 0/0/0/0
  default-switch.evpn.0: 0/0/0/0
  __default_evpn__.evpn.0: 0/0/0/0
10.6.20.2             64021       1756       1763       0       0    13:17:41 Establ
  inet.0: 8/15/15/0
10.6.20.6             64022       1759       1772       0       0    13:17:37 Establ
  inet.0: 4/15/15/0
```

