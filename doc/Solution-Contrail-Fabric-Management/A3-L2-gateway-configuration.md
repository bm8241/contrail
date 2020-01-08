* [TOC](ToC-Contrail-Fabric-Management)

## A.3 L2 gateway configuration

### A.3.1 BMS L2 on L2GW
```
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 6 vrf-target target:64512:8000005
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000005 members target:64512:8000005
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.11:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_red-l2-6-import
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ interfaces xe-0/0/2 flexible-vlan-tagging
set groups __contrail_overlay_evpn_access__ interfaces xe-0/0/2 native-vlan-id 4094
set groups __contrail_overlay_evpn_access__ interfaces xe-0/0/2 mtu 9192
set groups __contrail_overlay_evpn_access__ interfaces xe-0/0/2 encapsulation extended-vlan-bridge
set groups __contrail_overlay_evpn_access__ interfaces xe-0/0/2 unit 0 vlan-id 4094
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_evpn_access__ vlans bd-6 interface xe-0/0/2.0
set groups __contrail_overlay_evpn_access__ vlans bd-6 vxlan vni 6
```

### A.3.2 Multi-homing BMS on leaf-1
```
set groups __contrail_overlay_evpn_access__ interfaces ae0 flexible-vlan-tagging
set groups __contrail_overlay_evpn_access__ interfaces ae0 native-vlan-id 4094
set groups __contrail_overlay_evpn_access__ interfaces ae0 mtu 9192
set groups __contrail_overlay_evpn_access__ interfaces ae0 encapsulation extended-vlan-bridge
set groups __contrail_overlay_evpn_access__ interfaces ae0 unit 0 vlan-id 4094
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_evpn_access__ vlans bd-7 interface ae0.0
set groups __contrail_overlay_evpn_access__ vlans bd-7 vxlan vni 7
set groups __contrail_overlay_lag__ chassis aggregated-devices ethernet device-count 128
set groups __contrail_overlay_lag__ interfaces xe-0/0/3 gigether-options 802.3ad ae0
set groups __contrail_overlay_lag__ interfaces ae0 aggregated-ether-options lacp active
set groups __contrail_overlay_lag__ interfaces ae0 aggregated-ether-options lacp periodic fast
set groups __contrail_overlay_multi_homing__ chassis aggregated-devices ethernet device-count 128
set groups __contrail_overlay_multi_homing__ interfaces ae0 esi 00:19:fe:11:b4:22:d8:59:f3:00
set groups __contrail_overlay_multi_homing__ interfaces ae0 esi all-active
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp active
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp system-priority 100
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp system-id 00:3f:95:8d:22:4b
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp admin-key 1
set groups __contrail_overlay_multi_homing__ interfaces xe-0/0/3 gigether-options 802.3ad ae0
```

### A.3.3 Multi-homing BMS on leaf-2
```
set groups __contrail_overlay_evpn_access__ interfaces ae0 flexible-vlan-tagging
set groups __contrail_overlay_evpn_access__ interfaces ae0 native-vlan-id 4094
set groups __contrail_overlay_evpn_access__ interfaces ae0 mtu 9192
set groups __contrail_overlay_evpn_access__ interfaces ae0 encapsulation extended-vlan-bridge
set groups __contrail_overlay_evpn_access__ interfaces ae0 unit 0 vlan-id 4094
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_evpn_access__ vlans bd-7 interface ae0.0
set groups __contrail_overlay_evpn_access__ vlans bd-7 vxlan vni 7
set groups __contrail_overlay_lag__ chassis aggregated-devices ethernet device-count 128
set groups __contrail_overlay_lag__ interfaces xe-0/0/3 gigether-options 802.3ad ae0
set groups __contrail_overlay_lag__ interfaces ae0 aggregated-ether-options lacp active
set groups __contrail_overlay_lag__ interfaces ae0 aggregated-ether-options lacp periodic fast
set groups __contrail_overlay_multi_homing__ chassis aggregated-devices ethernet device-count 128
set groups __contrail_overlay_multi_homing__ interfaces ae0 esi 00:19:fe:11:b4:22:d8:59:f3:00
set groups __contrail_overlay_multi_homing__ interfaces ae0 esi all-active
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp active
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp system-priority 100
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp system-id 00:3f:95:8d:22:4b
set groups __contrail_overlay_multi_homing__ interfaces ae0 aggregated-ether-options lacp admin-key 1
set groups __contrail_overlay_multi_homing__ interfaces xe-0/0/3 gigether-options 802.3ad ae0
```

