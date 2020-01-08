* [TOC](Solution-Guide-SRIOV)

## A.1 Tagged

```
set groups __contrail_basic__ snmp community public authorization read-only
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
set groups __contrail_basic__ protocols l2-learning global-mac-table-aging-time 1800
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 192.190.0.138
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ routing-options router-id 192.190.0.138
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 192.190.0.138
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 192.179.0.36 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 192.179.0.37 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-8-import term t1 from community target_64512_8000007
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-8-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-8-export term t1 then community add target_64512_8000007
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-8-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-7-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-7-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000007 members target:64512:8000007
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000006 members target:64512:8000006
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 8 vrf-target target:64512:8000007
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 7 vrf-target target:64512:8000006
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 192.190.0.138:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_blue-l2-8-import
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_red-l2-7-import
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ interfaces ae2 description "Virtual Port Group : t1"
set groups __contrail_overlay_evpn_access__ interfaces ae2 mtu 9192
set groups __contrail_overlay_evpn_access__ interfaces ae2 unit 0 family ethernet-switching interface-mode trunk
set groups __contrail_overlay_evpn_access__ interfaces ae2 unit 0 family ethernet-switching vlan members bd-7
set groups __contrail_overlay_evpn_access__ interfaces ae2 unit 0 family ethernet-switching vlan members bd-8
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ protocols igmp-snooping vlan all proxy
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_evpn_access__ vlans bd-7 description "Virtual Network - red"
set groups __contrail_overlay_evpn_access__ vlans bd-7 vlan-id 12
set groups __contrail_overlay_evpn_access__ vlans bd-7 vxlan vni 7
set groups __contrail_overlay_evpn_access__ vlans bd-8 description "Virtual Network - blue"
set groups __contrail_overlay_evpn_access__ vlans bd-8 vlan-id 14
set groups __contrail_overlay_evpn_access__ vlans bd-8 vxlan vni 8
set groups __contrail_overlay_security_group_qfx5110-48s-4c__
set groups __contrail_overlay_lag__ chassis aggregated-devices ethernet device-count 128
set groups __contrail_overlay_lag__ interfaces xe-0/0/3 description "Virtual Port Group : t1"
set groups __contrail_overlay_lag__ interfaces xe-0/0/3 gigether-options 802.3ad ae2
set groups __contrail_overlay_lag__ interfaces ae2 description "Virtual Port Group : t1"
set groups __contrail_overlay_lag__ interfaces ae2 aggregated-ether-options lacp active
set groups __contrail_overlay_lag__ interfaces ae2 aggregated-ether-options lacp periodic fast
set groups __contrail_overlay_multi_homing__ chassis aggregated-devices ethernet device-count 128
set groups __contrail_overlay_multi_homing__ interfaces ae2 description "Virtual Port Group : t1"
set groups __contrail_overlay_multi_homing__ interfaces ae2 esi 00:98:80:2c:a8:b7:12:3d:55:00
set groups __contrail_overlay_multi_homing__ interfaces ae2 esi all-active
set groups __contrail_overlay_multi_homing__ interfaces ae2 aggregated-ether-options lacp active
set groups __contrail_overlay_multi_homing__ interfaces ae2 aggregated-ether-options lacp system-priority 100
set groups __contrail_overlay_multi_homing__ interfaces ae2 aggregated-ether-options lacp system-id 00:55:d3:21:7b:8a
set groups __contrail_overlay_multi_homing__ interfaces ae2 aggregated-ether-options lacp admin-key 1
set groups __contrail_overlay_multi_homing__ interfaces xe-0/0/3 description "Virtual Port Group : t1"
set groups __contrail_overlay_multi_homing__ interfaces xe-0/0/3 gigether-options 802.3ad ae2
set groups __contrail_overlay_storm_control__
```

