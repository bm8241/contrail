* [TOC](ToC-Contrail-Fabric-Management)

## A.7 Underlay configuration

### A.7.1 vqfx-leaf-1
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
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.251/32 primary
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.251/32 preferred
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 unit 0 family inet address 10.6.20.1/30
set groups __contrail_underlay_ip_clos__ routing-options router-id 10.6.0.251
set groups __contrail_underlay_ip_clos__ routing-options route-distinguisher-id 10.6.0.251
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table export PFE-LB
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table ecmp-fast-reroute
set groups __contrail_underlay_ip_clos__ protocols bgp log-updown
set groups __contrail_underlay_ip_clos__ protocols bgp graceful-restart
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP type external
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP mtu-discovery
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP import IPCLOS_BGP_IMP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP export IPCLOS_BGP_EXP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP vpn-apply-export
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP local-as 64501
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP multipath multiple-as
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection minimum-interval 350
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection multiplier 3
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection session-mode automatic
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.2 description vqfx-spine-1
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.2 peer-as 64500
set groups __contrail_underlay_ip_clos__ policy-options policy-statement PFE-LB then load-balance per-packet
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term default then reject
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term default then reject
set groups __contrail_underlay_infra_bms_access__
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.251
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.251
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64512
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.251
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.252 peer-as 64512
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
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.251:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ protocols igmp-snooping vlan all proxy
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_security_group_vqfx-10000__
set groups __contrail_overlay_lag__
set groups __contrail_overlay_multi_homing__
set groups __contrail_overlay_storm_control__
set groups __contrail_overlay_telemetry__
set apply-groups __contrail_basic__
set apply-groups __contrail_underlay_ip_clos__
set apply-groups __contrail_underlay_infra_bms_access__
set apply-groups __contrail_overlay_bgp__
set apply-groups __contrail_overlay_evpn__
set apply-groups __contrail_overlay_evpn_access__
set apply-groups __contrail_overlay_security_group_vqfx-10000__
set apply-groups __contrail_overlay_lag__
set apply-groups __contrail_overlay_multi_homing__
set apply-groups __contrail_overlay_storm_control__
set apply-groups __contrail_overlay_telemetry__
set system host-name vqfx-leaf-1
set system root-authentication encrypted-password "$6$WtgFqfMu$pm2RMagIYtOs.hHf1ORCxszF1bXruPq9l5/qedJCGCH/n9ng8HjHFCtCDzR9r1sWMmnD1GsmelokV.11INzi61"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system services telnet
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces xe-0/0/0 unit 0 family inet
set interfaces em0 unit 0 family inet address 10.6.8.11/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet
set routing-options static route 10.6.8.0/24 next-hop 10.6.8.254
set routing-options static route 0.0.0.0/0 next-hop 10.6.8.254
set protocols lldp interface all
```

### A.7.2 vqfx-leaf-2
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
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.250/32 primary
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.250/32 preferred
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 unit 0 family inet address 10.6.20.5/30
set groups __contrail_underlay_ip_clos__ routing-options router-id 10.6.0.250
set groups __contrail_underlay_ip_clos__ routing-options route-distinguisher-id 10.6.0.250
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table export PFE-LB
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table ecmp-fast-reroute
set groups __contrail_underlay_ip_clos__ protocols bgp log-updown
set groups __contrail_underlay_ip_clos__ protocols bgp graceful-restart
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP type external
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP mtu-discovery
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP import IPCLOS_BGP_IMP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP export IPCLOS_BGP_EXP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP vpn-apply-export
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP local-as 64502
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP multipath multiple-as
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection minimum-interval 350
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection multiplier 3
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection session-mode automatic
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.6 description vqfx-spine-1
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.6 peer-as 64500
set groups __contrail_underlay_ip_clos__ policy-options policy-statement PFE-LB then load-balance per-packet
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term default then reject
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term default then reject
set groups __contrail_underlay_infra_bms_access__
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.250
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.250
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64512
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.250
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.252 peer-as 64512
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
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.250:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ protocols igmp-snooping vlan all proxy
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_security_group_vqfx-10000__
set groups __contrail_overlay_lag__
set groups __contrail_overlay_multi_homing__
set groups __contrail_overlay_storm_control__
set groups __contrail_overlay_telemetry__
set apply-groups __contrail_basic__
set apply-groups __contrail_underlay_ip_clos__
set apply-groups __contrail_underlay_infra_bms_access__
set apply-groups __contrail_overlay_bgp__
set apply-groups __contrail_overlay_evpn__
set apply-groups __contrail_overlay_evpn_access__
set apply-groups __contrail_overlay_security_group_vqfx-10000__
set apply-groups __contrail_overlay_lag__
set apply-groups __contrail_overlay_multi_homing__
set apply-groups __contrail_overlay_storm_control__
set apply-groups __contrail_overlay_telemetry__
set system host-name vqfx-leaf-2
set system root-authentication encrypted-password "$6$HIvGoukA$8vH0SmYF2GJXyBACEANCSK5pj.20b1eMn0ckDwgiHBofMeuwmJrdQMeQHQ05GnCpKIoeqTFT4TuI7wQlJdJv//"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system services telnet
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces xe-0/0/0 unit 0 family inet
set interfaces em0 unit 0 family inet address 10.6.8.12/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet
set routing-options static route 10.6.8.0/24 next-hop 10.6.8.254
set routing-options static route 0.0.0.0/0 next-hop 10.6.8.254
set protocols lldp interface all
```

### A.7.3 vqfx-leaf-3
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
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.249/32 primary
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.249/32 preferred
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 unit 0 family inet address 10.6.20.9/30
set groups __contrail_underlay_ip_clos__ routing-options router-id 10.6.0.249
set groups __contrail_underlay_ip_clos__ routing-options route-distinguisher-id 10.6.0.249
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table export PFE-LB
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table ecmp-fast-reroute
set groups __contrail_underlay_ip_clos__ protocols bgp log-updown
set groups __contrail_underlay_ip_clos__ protocols bgp graceful-restart
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP type external
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP mtu-discovery
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP import IPCLOS_BGP_IMP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP export IPCLOS_BGP_EXP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP vpn-apply-export
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP local-as 64503
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP multipath multiple-as
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection minimum-interval 350
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection multiplier 3
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection session-mode automatic
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.10 description vqfx-spine-1
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.10 peer-as 64500
set groups __contrail_underlay_ip_clos__ policy-options policy-statement PFE-LB then load-balance per-packet
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term default then reject
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term default then reject
set groups __contrail_underlay_infra_bms_access__
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.249
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.249
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64512
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.249
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.252 peer-as 64512
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
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.249:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ protocols igmp-snooping vlan all proxy
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_security_group_vqfx-10000__
set groups __contrail_overlay_lag__
set groups __contrail_overlay_multi_homing__
set groups __contrail_overlay_storm_control__
set groups __contrail_overlay_telemetry__
set apply-groups __contrail_basic__
set apply-groups __contrail_underlay_ip_clos__
set apply-groups __contrail_underlay_infra_bms_access__
set apply-groups __contrail_overlay_bgp__
set apply-groups __contrail_overlay_evpn__
set apply-groups __contrail_overlay_evpn_access__
set apply-groups __contrail_overlay_security_group_vqfx-10000__
set apply-groups __contrail_overlay_lag__
set apply-groups __contrail_overlay_multi_homing__
set apply-groups __contrail_overlay_storm_control__
set apply-groups __contrail_overlay_telemetry__
set system host-name vqfx-leaf-3
set system root-authentication encrypted-password "$6$kCDReWsg$tFUJZ3DddPyXJ9DI8k/3fA4X0JqW5WJ9s3Ih3RcfFA8Z4szpWhX8msYhYjcXl.uBLuTv.QwoJ3s3TuDjea4mw/"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system services telnet
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces xe-0/0/0 unit 0 family inet
set interfaces em0 unit 0 family inet address 10.6.8.13/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet
set routing-options static route 10.6.8.0/24 next-hop 10.6.8.254
set routing-options static route 0.0.0.0/0 next-hop 10.6.8.254
set protocols lldp interface all
```

### A.7.4 vqfx-leaf-4
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
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.248/32 primary
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.248/32 preferred
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/0 unit 0 family inet address 10.6.20.13/30
set groups __contrail_underlay_ip_clos__ routing-options router-id 10.6.0.248
set groups __contrail_underlay_ip_clos__ routing-options route-distinguisher-id 10.6.0.248
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table export PFE-LB
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table ecmp-fast-reroute
set groups __contrail_underlay_ip_clos__ protocols bgp log-updown
set groups __contrail_underlay_ip_clos__ protocols bgp graceful-restart
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP type external
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP mtu-discovery
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP import IPCLOS_BGP_IMP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP export IPCLOS_BGP_EXP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP vpn-apply-export
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP local-as 64504
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP multipath multiple-as
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection minimum-interval 350
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection multiplier 3
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection session-mode automatic
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.14 description vqfx-spine-1
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.14 peer-as 64500
set groups __contrail_underlay_ip_clos__ policy-options policy-statement PFE-LB then load-balance per-packet
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term default then reject
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term default then reject
set groups __contrail_underlay_infra_bms_access__
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.248
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.248
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64512
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.248
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.252 peer-as 64512
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
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.248:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_access__ protocols evpn multicast-mode ingress-replication
set groups __contrail_overlay_evpn_access__ protocols igmp-snooping vlan all proxy
set groups __contrail_overlay_evpn_access__ switch-options vrf-target auto
set groups __contrail_overlay_security_group_vqfx-10000__
set groups __contrail_overlay_lag__
set groups __contrail_overlay_multi_homing__
set groups __contrail_overlay_storm_control__
set groups __contrail_overlay_telemetry__
set apply-groups __contrail_basic__
set apply-groups __contrail_underlay_ip_clos__
set apply-groups __contrail_underlay_infra_bms_access__
set apply-groups __contrail_overlay_bgp__
set apply-groups __contrail_overlay_evpn__
set apply-groups __contrail_overlay_evpn_access__
set apply-groups __contrail_overlay_security_group_vqfx-10000__
set apply-groups __contrail_overlay_lag__
set apply-groups __contrail_overlay_multi_homing__
set apply-groups __contrail_overlay_storm_control__
set apply-groups __contrail_overlay_telemetry__
set system host-name vqfx-leaf-4
set system root-authentication encrypted-password "$6$n5wU5Bxa$J6ARRQr8JAnrmUw.XbNGb0XtlAEqRMv4Ga726S6kZOLlqQPyOWjeao945dcLImRBKD3UxPF3q35bnwMcrGWpk1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system services telnet
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces xe-0/0/0 unit 0 family inet
set interfaces em0 unit 0 family inet address 10.6.8.14/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet
set routing-options static route 10.6.8.0/24 next-hop 10.6.8.254
set routing-options static route 0.0.0.0/0 next-hop 10.6.8.254
set protocols lldp interface all
```

### A.7.5 vqfx-spine-1
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
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.252/32 primary
set groups __contrail_underlay_ip_clos__ interfaces lo0 unit 0 family inet address 10.6.0.252/32 preferred
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/2 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/2 unit 0 family inet address 10.6.20.2/30
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/3 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/3 unit 0 family inet address 10.6.20.6/30
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/4 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/4 unit 0 family inet address 10.6.20.10/30
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/5 mtu 9192
set groups __contrail_underlay_ip_clos__ interfaces xe-0/0/5 unit 0 family inet address 10.6.20.14/30
set groups __contrail_underlay_ip_clos__ routing-options router-id 10.6.0.252
set groups __contrail_underlay_ip_clos__ routing-options route-distinguisher-id 10.6.0.252
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table export PFE-LB
set groups __contrail_underlay_ip_clos__ routing-options forwarding-table ecmp-fast-reroute
set groups __contrail_underlay_ip_clos__ protocols bgp log-updown
set groups __contrail_underlay_ip_clos__ protocols bgp graceful-restart
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP type external
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP mtu-discovery
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP import IPCLOS_BGP_IMP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP export IPCLOS_BGP_EXP
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP vpn-apply-export
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP local-as 64500
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP multipath multiple-as
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection minimum-interval 350
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection multiplier 3
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP bfd-liveness-detection session-mode automatic
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.1 description vqfx-leaf-1
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.1 peer-as 64501
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.5 description vqfx-leaf-2
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.5 peer-as 64502
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.9 description vqfx-leaf-3
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.9 peer-as 64503
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.13 description vqfx-leaf-4
set groups __contrail_underlay_ip_clos__ protocols bgp group IPCLOS_eBGP neighbor 10.6.20.13 peer-as 64504
set groups __contrail_underlay_ip_clos__ policy-options policy-statement PFE-LB then load-balance per-packet
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_EXP term default then reject
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol bgp
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback from protocol direct
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term loopback then accept
set groups __contrail_underlay_ip_clos__ policy-options policy-statement IPCLOS_BGP_IMP term default then reject
set groups __contrail_underlay_infra_bms_access__
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.252
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.252
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64512
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-address 10.6.0.252
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 vpn-apply-export
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 cluster 10.6.0.252
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 local-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.248 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.249 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.250 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.0.251 peer-as 64512
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64512 neighbor 10.6.8.95 peer-as 64512
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
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.252:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_gateway__
set groups __contrail_overlay_evpn_type5__ routing-options forwarding-table chained-composite-next-hop ingress evpn
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 1 from protocol direct
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 1 then accept
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 2 from protocol static
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 2 then accept
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 3 from protocol evpn
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 3 from route-filter 0.0.0.0/0 prefix-length-range /32-/32
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement type5_policy term 3 then accept
set groups __contrail_overlay_dhcp_relay__
set groups __contrail_overlay_telemetry__
set apply-groups __contrail_basic__
set apply-groups __contrail_underlay_ip_clos__
set apply-groups __contrail_underlay_infra_bms_access__
set apply-groups __contrail_overlay_bgp__
set apply-groups __contrail_overlay_evpn__
set apply-groups __contrail_overlay_evpn_gateway__
set apply-groups __contrail_overlay_evpn_type5__
set apply-groups __contrail_overlay_dhcp_relay__
set apply-groups __contrail_overlay_telemetry__
set system host-name vqfx-spine-1
set system root-authentication encrypted-password "$6$mSfSFMk7$h0qbYTXt5zXIFzLtPxMeXkUukj40niWNvtZkHJ78hyP7hZqkMipHcNhYGBX0HDAVv2rcA5fL/b0OkrcyhAnWw."
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system services telnet
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces xe-0/0/2 unit 0 family inet
set interfaces xe-0/0/3 unit 0 family inet
set interfaces xe-0/0/4 unit 0 family inet
set interfaces xe-0/0/5 unit 0 family inet
set interfaces em0 unit 0 family inet address 10.6.8.21/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet
set routing-options static route 10.6.8.0/24 next-hop 10.6.8.254
set routing-options static route 0.0.0.0/0 next-hop 10.6.8.254
set protocols lldp interface all
```

