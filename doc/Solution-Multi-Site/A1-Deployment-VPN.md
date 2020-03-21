# A.1 Deployment VPN

## A.1.1 Underlay

### A.1.1.1 leaf-11
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.11/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.20.1/31
set groups underlay interfaces xe-0/0/3 unit 0 family ethernet-switching vlan members vlan-11
set groups underlay interfaces irb unit 11 family inet address 10.6.11.254/24
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.11/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.11.0/24 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.11
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65011
set groups underlay protocols bgp group underlay neighbor 10.6.20.0 peer-as 65021
set groups underlay vlans vlan-11 vlan-id 11
set groups underlay vlans vlan-11 l3-interface irb.11
set apply-groups underlay
set system host-name vqfx-leaf-11
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.11/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
```

#### A.1.1.2 spine-21
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.21/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.30.0/31
set groups underlay interfaces xe-0/0/2 unit 0 family inet address 10.6.20.0/31
set groups underlay interfaces xe-0/0/4 unit 0 family inet address 10.6.20.4/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.21/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.21
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay cluster 10.6.0.21
set groups underlay protocols bgp group underlay local-as 65021
set groups underlay protocols bgp group underlay neighbor 10.6.30.1 peer-as 65031
set groups underlay protocols bgp group underlay neighbor 10.6.20.1 peer-as 65011
set groups underlay protocols bgp group underlay neighbor 10.6.20.5 peer-as 65013
set apply-groups underlay
set system host-name vqfx-spine-21
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.21/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
```

### A.1.1.3 gw-31
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.31/32
set groups underlay interfaces ge-0/0/2 unit 0 family inet address 10.6.30.1/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.31/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.31
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65031
set groups underlay protocols bgp group underlay neighbor 10.6.30.0 peer-as 65021
set groups core interfaces ge-0/0/0 unit 0 family inet address 172.16.0.1/24
set groups core interfaces ge-0/0/0 unit 0 family mpls
set groups core policy-options policy-statement core-export term t1 then next-hop self
set groups core protocols ospf area 0.0.0.0 interface lo0.0
set groups core protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set groups core protocols bgp group core type internal
set groups core protocols bgp group core local-address 10.6.0.31
set groups core protocols bgp group core family inet unicast
set groups core protocols bgp group core family inet-vpn unicast
set groups core protocols bgp group core export core-export
set groups core protocols bgp group core local-as 64500
set groups core protocols bgp group core neighbor 10.6.0.33 peer-as 64500
set groups core protocols ldp interface ge-0/0/0.0
set apply-groups underlay
set apply-groups core
set system host-name vmx-31
set system root-authentication encrypted-password "$6$.eu1H0ZX$K3iXOzGi2WyIJbFaRxuVzjlK/W/3y.11o.3h8.rUbldqHi7akVrsQtj.HpOkqEbMIVQHiTpBzlX7/fCFJ27kJ1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
set interfaces fxp0 unit 0 family inet address 10.6.8.31/24
```

### A.1.1.4 gw-33
```
set groups core interfaces ge-0/0/0 unit 0 family inet address 172.16.0.3/24
set groups core interfaces ge-0/0/0 unit 0 family mpls
set groups core interfaces lo0 unit 0 family inet address 10.6.0.33/32
set groups core routing-options route-distinguisher-id 10.6.0.33
set groups core protocols bgp group core type internal
set groups core protocols bgp group core local-address 10.6.0.33
set groups core protocols bgp group core family inet unicast
set groups core protocols bgp group core family inet-vpn unicast
set groups core protocols bgp group core export core-export
set groups core protocols bgp group core local-as 64500
set groups core protocols bgp group core neighbor 10.6.0.31 peer-as 64500
set groups core protocols ospf area 0.0.0.0 interface lo0.0
set groups core protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set groups core protocols ldp interface ge-0/0/0.0
set groups core policy-options policy-statement core-export term t1 from protocol direct
set groups core policy-options policy-statement core-export term t1 from route-filter 10.6.0.33/32 exact
set groups core policy-options policy-statement core-export term t1 then next-hop self
set groups core policy-options policy-statement core-export term t1 then accept
set groups customer-a interfaces ge-0/0/2 unit 0 family inet address 10.20.1.254/24
set groups customer-a interfaces lo0 unit 101 family inet address 10.20.1.10/32
set groups customer-a policy-options policy-statement vrf-a-export term t1 from protocol bgp
set groups customer-a policy-options policy-statement vrf-a-export term t1 then accept
set groups customer-a routing-instances customer-a instance-type vrf
set groups customer-a routing-instances customer-a interface lo0.101
set groups customer-a routing-instances customer-a interface ge-0/0/2.0
set groups customer-a routing-instances customer-a route-distinguisher 10.6.0.33:60101
set groups customer-a routing-instances customer-a vrf-target target:60101:100
set groups customer-a routing-instances customer-a vrf-table-label
set groups customer-a routing-instances customer-a protocols ospf export vrf-a-export
set groups customer-a routing-instances customer-a protocols ospf area 1.1.1.1 interface ge-0/0/2.0
set groups customer-b interfaces ge-0/0/3 unit 0 family inet address 10.20.2.254/24
set groups customer-b interfaces lo0 unit 102 family inet address 10.20.2.10/32
set groups customer-b policy-options policy-statement vrf-b-export term t1 from protocol bgp
set groups customer-b policy-options policy-statement vrf-b-export term t1 then accept
set groups customer-b routing-instances customer-b instance-type vrf
set groups customer-b routing-instances customer-b interface lo0.102
set groups customer-b routing-instances customer-b interface ge-0/0/3.0
set groups customer-b routing-instances customer-b route-distinguisher 10.6.0.33:60102
set groups customer-b routing-instances customer-b vrf-target target:60102:100
set groups customer-b routing-instances customer-b vrf-table-label
set groups customer-b routing-instances customer-b protocols ospf export vrf-b-export
set groups customer-b routing-instances customer-b protocols ospf area 2.2.2.2 interface ge-0/0/3.0
set apply-groups core
set apply-groups customer-a
set apply-groups customer-b
set system root-authentication encrypted-password "$6$.eu1H0ZX$K3iXOzGi2WyIJbFaRxuVzjlK/W/3y.11o.3h8.rUbldqHi7akVrsQtj.HpOkqEbMIVQHiTpBzlX7/fCFJ27kJ1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system host-name vmx-33
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
set interfaces fxp0 unit 0 family inet address 10.6.8.33/24
```

### A.1.1.5 gw-101
```
set groups site interfaces xe-0/0/0 unit 0 family inet address 10.20.1.253/24
set groups site interfaces xe-0/0/2 unit 0 family ethernet-switching vlan members vlan-8
set groups site interfaces irb unit 8 family inet address 172.16.11.254/24
set groups site protocols ospf area 1.1.1.1 interface xe-0/0/0.0
set groups site protocols ospf area 1.1.1.1 interface irb.8
set groups site vlans vlan-8 vlan-id 8
set groups site vlans vlan-8 l3-interface irb.8
set apply-groups site
set system host-name gw-101
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.101/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
```

### A.1.1.6 gw-102
```
set groups site interfaces xe-0/0/0 unit 0 family inet address 10.20.2.253/24
set groups site interfaces xe-0/0/2 unit 0 family ethernet-switching vlan members vlan-8
set groups site interfaces irb unit 8 family inet address 172.16.12.254/24
set groups site protocols ospf area 2.2.2.2 interface xe-0/0/0.0
set groups site protocols ospf area 2.2.2.2 interface irb.8
set groups site vlans vlan-8 vlan-id 8
set groups site vlans vlan-8 l3-interface irb.8
set apply-groups site
set system host-name gw-102
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.102/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
```


## A.1.2 Overlay

### A.1.2.1 gw-31
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
set groups __contrail_overlay_bgp__ policy-options policy-statement mpls_over_udp term 1 then community add encap-udp
set groups __contrail_overlay_bgp__ policy-options policy-statement mpls_over_udp term 1 then accept
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn from family inet-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn from family inet6-vpn
set groups __contrail_overlay_bgp__ policy-options policy-statement _contrail_ibgp_export_policy term inet6-vpn then next-hop self
set groups __contrail_overlay_bgp__ policy-options community encap-udp members 0x030c:64512:13
set groups __contrail_overlay_bgp__ routing-options route-distinguisher-id 10.6.0.31
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_udp_tunnel source-address 10.6.0.31
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_udp_tunnel gre
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_udp_tunnel destination-networks 10.6.11.0/24
set groups __contrail_overlay_bgp__ routing-options dynamic-tunnels _contrail_udp_tunnel destination-networks 10.6.12.0/24
set groups __contrail_overlay_bgp__ routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
set groups __contrail_overlay_bgp__ routing-options router-id 10.6.0.31
set groups __contrail_overlay_bgp__ routing-options autonomous-system 64520
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 type internal
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 local-address 10.6.0.31
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 hold-time 90
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 family inet-vpn unicast
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 family inet6-vpn unicast
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 family evpn signaling
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 family route-target
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 export _contrail_ibgp_export_policy
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 cluster 10.6.0.31
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 local-as 64520
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 multipath
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 neighbor 10.6.11.1 peer-as 64520
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 neighbor 10.6.8.6 peer-as 64520
set groups __contrail_overlay_bgp__ protocols bgp group _contrail_asn-64520 vpn-apply-export
set groups __contrail_overlay_fip_snat__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-customer-a-l3-8 interface irb.8
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-customer-b-l3-9 interface irb.9
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_share-a-b-l3-10 interface irb.10
set groups __contrail_overlay_networking__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_networking__ interfaces irb unit 10 family inet address 192.168.50.3/24 virtual-gateway-address 192.168.50.1
set groups __contrail_overlay_networking__ interfaces irb unit 8 family inet address 192.168.10.3/24 virtual-gateway-address 192.168.10.1
set groups __contrail_overlay_networking__ interfaces irb unit 9 family inet address 192.168.10.3/24 virtual-gateway-address 192.168.10.1
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l2-8-import term t1 from community target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l2-8-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l2-8-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l2-8-export term t1 then community add target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l2-8-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l3-8-import term t1 from community target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l3-8-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l3-8-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l3-8-export term t1 then community add target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-a-l3-8-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l2-9-import term t1 from community target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l2-9-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l2-9-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l2-9-export term t1 then community add target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l2-9-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l3-9-import term t1 from community target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l3-9-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l3-9-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l3-9-export term t1 then community add target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-customer-b-l3-9-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-import term t1 from community target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-import term t1 from community target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-export term t1 then community add target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-export term t1 then community add target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l2-10-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-import term t1 from community target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-import term t1 from community target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-export term t1 then community add target_60102_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-export term t1 then community add target_60101_100
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_share-a-b-l3-10-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options community target_60101_100 members target:60101:100
set groups __contrail_overlay_networking__ policy-options community target_60102_100 members target:60102:100
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 bridge-domains bd-8 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 bridge-domains bd-8 routing-interface irb.8
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 bridge-domains bd-8 vxlan vni 8
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 route-distinguisher 10.6.0.31:8
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 vrf-import _contrail_red-customer-a-l2-8-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 vrf-export _contrail_red-customer-a-l2-8-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l2-8 vrf-target target:64520:8
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 routing-options static route 192.168.10.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 routing-options auto-export family inet unicast
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 interface irb.8
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 route-distinguisher 10.6.0.31:30008
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 vrf-import _contrail_red-customer-a-l3-8-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 vrf-export _contrail_red-customer-a-l3-8-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 vrf-target target:64520:30008
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-a-l3-8 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 bridge-domains bd-9 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 bridge-domains bd-9 routing-interface irb.9
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 bridge-domains bd-9 vxlan vni 9
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 route-distinguisher 10.6.0.31:9
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 vrf-import _contrail_red-customer-b-l2-9-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 vrf-export _contrail_red-customer-b-l2-9-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l2-9 vrf-target target:64520:9
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 routing-options static route 192.168.10.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 routing-options auto-export family inet unicast
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 interface irb.9
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 route-distinguisher 10.6.0.31:30009
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 vrf-import _contrail_red-customer-b-l3-9-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 vrf-export _contrail_red-customer-b-l3-9-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 vrf-target target:64520:30009
set groups __contrail_overlay_networking__ routing-instances _contrail_red-customer-b-l3-9 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 bridge-domains bd-10 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 bridge-domains bd-10 routing-interface irb.10
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 bridge-domains bd-10 vxlan vni 10
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 route-distinguisher 10.6.0.31:10
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 vrf-import _contrail_share-a-b-l2-10-import
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 vrf-export _contrail_share-a-b-l2-10-export
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l2-10 vrf-target target:64520:10
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 routing-options static route 192.168.50.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 routing-options auto-export family inet unicast
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 interface irb.10
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 route-distinguisher 10.6.0.31:30010
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 vrf-import _contrail_share-a-b-l3-10-import
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 vrf-export _contrail_share-a-b-l3-10-export
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 vrf-target target:64520:30010
set groups __contrail_overlay_networking__ routing-instances _contrail_share-a-b-l3-10 vrf-table-label
set apply-groups __contrail_basic__
set apply-groups __contrail_overlay_bgp__
set apply-groups __contrail_overlay_fip_snat__
set apply-groups __contrail_overlay_networking__
```

## A.1.3 Route table

### A.1.3.1 gw-31
```
root@vmx-31> show route table _contrail_red-customer-a-l3-8.inet.0 

_contrail_red-customer-a-l3-8.inet.0: 9 destinations, 10 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.20.1.0/24       *[BGP/170] 00:49:09, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 16
10.20.1.10/32      *[BGP/170] 00:49:09, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 16
172.16.11.0/24     *[BGP/170] 00:48:04, MED 2, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 16
192.168.10.0/24    *[Direct/0] 01:00:37
                    >  via irb.8
                    [Static/5] 01:20:33
                       Discard
192.168.10.3/32    *[Local/0] 01:00:37
                       Local via irb.8
192.168.10.4/32    *[BGP/170] 00:49:09, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 25
192.168.50.0/24    *[Direct/0] 01:00:37
                    >  via irb.10
192.168.50.3/32    *[Local/0] 01:00:37
                       Local via irb.10
192.168.50.4/32    *[BGP/170] 00:49:09, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 39

root@vmx-31> show route table _contrail_red-customer-b-l3-9.inet.0    

_contrail_red-customer-b-l3-9.inet.0: 9 destinations, 10 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.20.2.0/24       *[BGP/170] 00:49:20, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 17
10.20.2.10/32      *[BGP/170] 00:49:20, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 17
172.16.12.0/24     *[BGP/170] 00:49:20, MED 2, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 17
192.168.10.0/24    *[Direct/0] 01:00:48
                    >  via irb.9
                    [Static/5] 01:20:24
                       Discard
192.168.10.3/32    *[Local/0] 01:00:48
                       Local via irb.9
192.168.10.4/32    *[BGP/170] 00:49:20, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 32
192.168.50.0/24    *[Direct/0] 01:00:48
                    >  via irb.10
192.168.50.3/32    *[Local/0] 01:00:48
                       Local via irb.10
192.168.50.4/32    *[BGP/170] 00:49:20, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 39

root@vmx-31> show route table _contrail_share-a-b-l3-10.inet.0        

_contrail_share-a-b-l3-10.inet.0: 12 destinations, 16 routes (12 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.20.1.0/24       *[BGP/170] 00:49:34, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 16
10.20.1.10/32      *[BGP/170] 00:49:34, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 16
10.20.2.0/24       *[BGP/170] 00:49:34, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 17
10.20.2.10/32      *[BGP/170] 00:49:34, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 17
172.16.11.0/24     *[BGP/170] 00:48:29, MED 2, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 16
172.16.12.0/24     *[BGP/170] 00:49:34, MED 2, localpref 100, from 10.6.0.33
                      AS path: I, validation-state: unverified
                    >  to 172.16.0.3 via ge-0/0/0.0, Push 17
192.168.10.0/24    *[Direct/0] 01:01:02
                    >  via irb.8
                    [Direct/0] 01:01:02
                    >  via irb.9
192.168.10.3/32    *[Local/0] 01:01:02
                       Local via irb.8
                    [Local/0] 01:01:02
                       Local via irb.9
192.168.10.4/32    *[BGP/170] 00:49:34, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 32
                    [BGP/170] 00:49:34, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 25
192.168.50.0/24    *[Direct/0] 01:01:02
                    >  via irb.10
                    [Static/5] 01:21:28
                       Discard
192.168.50.3/32    *[Local/0] 01:01:02
                       Local via irb.10
192.168.50.4/32    *[BGP/170] 00:49:34, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    >  via gr-0/0/10.32770, Push 39
```

### A.1.3.2 gw-33
```
root@vmx-33> show route table customer-a.inet.0 

customer-a.inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.20.1.0/24       *[Direct/0] 01:30:02
                    > via ge-0/0/2.0
10.20.1.10/32      *[Direct/0] 01:30:02
                    > via lo0.101
10.20.1.254/32     *[Local/0] 01:30:02
                      Local via ge-0/0/2.0
172.16.11.0/24     *[OSPF/10] 00:49:16, metric 2
                    > to 10.20.1.253 via ge-0/0/2.0
192.168.10.0/24    *[BGP/170] 01:21:45, localpref 100, from 10.6.0.31
                      AS path: I, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 19
192.168.10.4/32    *[BGP/170] 00:50:20, MED 100, localpref 200, from 10.6.0.31
                      AS path: 64520 ?, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 300224
192.168.50.0/24    *[BGP/170] 01:22:15, localpref 100, from 10.6.0.31
                      AS path: I, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 18
192.168.50.4/32    *[BGP/170] 00:50:20, MED 100, localpref 200, from 10.6.0.31
                      AS path: 64520 ?, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 300240
224.0.0.5/32       *[OSPF/10] 01:30:02, metric 1
                      MultiRecv

root@vmx-33> show route table customer-b.inet.0    

customer-b.inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.20.2.0/24       *[Direct/0] 01:30:06
                    > via ge-0/0/3.0
10.20.2.10/32      *[Direct/0] 01:30:06
                    > via lo0.102
10.20.2.254/32     *[Local/0] 01:30:06
                      Local via ge-0/0/3.0
172.16.12.0/24     *[OSPF/10] 01:29:46, metric 2
                    > to 10.20.2.253 via ge-0/0/3.0
192.168.10.0/24    *[BGP/170] 01:21:29, localpref 100, from 10.6.0.31
                      AS path: I, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 20
192.168.10.4/32    *[BGP/170] 00:50:24, MED 100, localpref 200, from 10.6.0.31
                      AS path: 64520 ?, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 300272
192.168.50.0/24    *[BGP/170] 01:22:19, localpref 100, from 10.6.0.31
                      AS path: I, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 18
192.168.50.4/32    *[BGP/170] 00:50:24, MED 100, localpref 200, from 10.6.0.31
                      AS path: 64520 ?, validation-state: unverified
                    > to 172.16.0.1 via ge-0/0/0.0, Push 300240
224.0.0.5/32       *[OSPF/10] 01:30:06, metric 1
                      MultiRecv
```

### A.1.3.3 gw-101
```
root@gw-101> show route table inet.0 

inet.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.6.8.0/24        *[Direct/0] 01:38:55
                    >  via em0.0
10.6.8.101/32      *[Local/0] 01:38:55
                       Local via em0.0
10.20.1.0/24       *[Direct/0] 14:50:19
                    >  via xe-0/0/0.0
10.20.1.253/32     *[Local/0] 14:50:19
                       Local via xe-0/0/0.0
169.254.0.0/24     *[Direct/0] 1d 19:15:50
                    >  via em1.0
169.254.0.2/32     *[Local/0] 1d 19:15:50
                       Local via em1.0
172.16.11.0/24     *[Direct/0] 01:45:08
                    >  via irb.8
172.16.11.254/32   *[Local/0] 01:45:08
                       Local via irb.8
192.168.10.0/24    *[OSPF/150] 00:49:58, metric 0, tag 3489660928
                    >  to 10.20.1.254 via xe-0/0/0.0
192.168.10.4/32    *[OSPF/150] 00:49:58, metric 100, tag 3489660928
                    >  to 10.20.1.254 via xe-0/0/0.0
192.168.50.0/24    *[OSPF/150] 00:49:58, metric 0, tag 3489660928
                    >  to 10.20.1.254 via xe-0/0/0.0
192.168.50.4/32    *[OSPF/150] 00:49:58, metric 100, tag 3489660928
                    >  to 10.20.1.254 via xe-0/0/0.0
224.0.0.5/32       *[OSPF/10] 14:50:19, metric 1
                       MultiRecv
```

### A.1.3.4 gw-102
```
root@gw-102> show route table inet.0 

inet.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.6.8.0/24        *[Direct/0] 01:42:19
                    >  via em0.0
10.6.8.102/32      *[Local/0] 01:42:19
                       Local via em0.0
10.20.2.0/24       *[Direct/0] 01:36:02
                    >  via xe-0/0/0.0
10.20.2.253/32     *[Local/0] 01:36:02
                       Local via xe-0/0/0.0
169.254.0.0/24     *[Direct/0] 2d 22:04:17
                    >  via em1.0
169.254.0.2/32     *[Local/0] 2d 22:04:17
                       Local via em1.0
172.16.12.0/24     *[Direct/0] 01:36:02
                    >  via irb.8
172.16.12.254/32   *[Local/0] 01:36:02
                       Local via irb.8
192.168.10.0/24    *[OSPF/150] 01:22:35, metric 0, tag 3489660928
                    >  to 10.20.2.254 via xe-0/0/0.0
192.168.10.4/32    *[OSPF/150] 00:51:31, metric 100, tag 3489660928
                    >  to 10.20.2.254 via xe-0/0/0.0
192.168.50.0/24    *[OSPF/150] 01:23:25, metric 0, tag 3489660928
                    >  to 10.20.2.254 via xe-0/0/0.0
192.168.50.4/32    *[OSPF/150] 00:51:31, metric 100, tag 3489660928
                    >  to 10.20.2.254 via xe-0/0/0.0
224.0.0.5/32       *[OSPF/10] 01:36:03, metric 1
                       MultiRecv
```

