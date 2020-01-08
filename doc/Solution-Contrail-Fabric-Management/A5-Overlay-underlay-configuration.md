* [TOC](ToC-Contrail-Fabric-Management)

## A.5 Overlay-underlay configuration

### A.5.1 Public network on MX-1
```
set groups __contrail_overlay_fip_snat__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term term-_contrail_public-l3-8 from destination-address 172.16.10.0/24
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term term-_contrail_public-l3-8 then routing-instance _contrail_public-l3-8
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term default-term then accept
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_public-l3-8 interface irb.8
set groups __contrail_overlay_networking__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_networking__ interfaces irb unit 8 family inet address 172.16.10.4/24 virtual-gateway-address 172.16.10.1
set groups __contrail_overlay_networking__ forwarding-options family inet filter input redirect_to_public_vrf_filter
set groups __contrail_overlay_networking__ routing-options static route 172.16.10.0/24 discard
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-import term t1 from community target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-export term t1 then community add target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-import term t1 from community target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-export term t1 then community add target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options community target_64512_8000007 members target:64512:8000007
set groups __contrail_overlay_networking__ firewall family inet filter redirect_to_public_vrf_filter term term-8 from destination-address 172.16.10.0/24
set groups __contrail_overlay_networking__ firewall family inet filter redirect_to_public_vrf_filter term term-8 then routing-instance _contrail_public-l3-8
set groups __contrail_overlay_networking__ firewall family inet filter redirect_to_public_vrf_filter term default-term then accept
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 route-distinguisher 10.6.0.31:8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-import _contrail_public-l2-8-import
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-export _contrail_public-l2-8-export
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-target target:64512:8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 bridge-domains bd-8 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 bridge-domains bd-8 routing-interface irb.8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 bridge-domains bd-8 vxlan vni 8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 interface irb.8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 route-distinguisher 10.6.0.31:30008
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-import _contrail_public-l3-8-import
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-export _contrail_public-l3-8-export
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-target target:64512:30008
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 routing-options static route 172.16.10.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 routing-options static route 0.0.0.0/0 next-table inet.0
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 routing-options auto-export family inet unicast
```

### A.5.2 Public network on MX-2
```
set groups __contrail_overlay_fip_snat__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term term-_contrail_public-l3-8 from destination-address 172.16.10.0/24
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term term-_contrail_public-l3-8 then routing-instance _contrail_public-l3-8
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term default-term then accept
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_public-l3-8 interface irb.8
set groups __contrail_overlay_networking__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_networking__ interfaces irb unit 8 family inet address 172.16.10.3/24 virtual-gateway-address 172.16.10.1
set groups __contrail_overlay_networking__ forwarding-options family inet filter input redirect_to_public_vrf_filter
set groups __contrail_overlay_networking__ routing-options static route 172.16.10.0/24 discard
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-import term t1 from community target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-export term t1 then community add target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l2-8-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-import term t1 from community target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-export term t1 then community add target_64512_8000007
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_public-l3-8-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options community target_64512_8000007 members target:64512:8000007
set groups __contrail_overlay_networking__ firewall family inet filter redirect_to_public_vrf_filter term term-8 from destination-address 172.16.10.0/24
set groups __contrail_overlay_networking__ firewall family inet filter redirect_to_public_vrf_filter term term-8 then routing-instance _contrail_public-l3-8
set groups __contrail_overlay_networking__ firewall family inet filter redirect_to_public_vrf_filter term default-term then accept
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 route-distinguisher 10.6.0.32:8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-import _contrail_public-l2-8-import
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-export _contrail_public-l2-8-export
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 vrf-target target:64512:8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 bridge-domains bd-8 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 bridge-domains bd-8 routing-interface irb.8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l2-8 bridge-domains bd-8 vxlan vni 8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 interface irb.8
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 route-distinguisher 10.6.0.32:30008
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-import _contrail_public-l3-8-import
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-export _contrail_public-l3-8-export
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-target target:64512:30008
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 routing-options static route 172.16.10.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 routing-options static route 0.0.0.0/0 next-table inet.0
set groups __contrail_overlay_networking__ routing-instances _contrail_public-l3-8 routing-options auto-export family inet unicast
```

### A.5.3 FIP on MX-1
```
set groups __contrail_overlay_fip_snat__ services service-set sv-_contrail_red-l3-6-n nat-rules sv-_contrail_red-l3-6-n-sn-rule
set groups __contrail_overlay_fip_snat__ services service-set sv-_contrail_red-l3-6-n nat-rules sv-_contrail_red-l3-6-n-dn-rule
set groups __contrail_overlay_fip_snat__ services service-set sv-_contrail_red-l3-6-n next-hop-service inside-service-interface si-0/0/0.11
set groups __contrail_overlay_fip_snat__ services service-set sv-_contrail_red-l3-6-n next-hop-service outside-service-interface si-0/0/0.12
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-sn-rule match-direction input
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-sn-rule term term_192_168_10_4 from source-address 192.168.10.4/32
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-sn-rule term term_192_168_10_4 then translated source-prefix 172.16.10.5/32
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-sn-rule term term_192_168_10_4 then translated translation-type basic-nat44
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-dn-rule match-direction output
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-dn-rule term term_172_16_10_5 from destination-address 172.16.10.5/32
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-dn-rule term term_172_16_10_5 then translated destination-prefix 192.168.10.4/32
set groups __contrail_overlay_fip_snat__ services nat rule sv-_contrail_red-l3-6-n-dn-rule term term_172_16_10_5 then translated translation-type dnat-44
set groups __contrail_overlay_fip_snat__ interfaces si-0/0/0 unit 11 family inet
set groups __contrail_overlay_fip_snat__ interfaces si-0/0/0 unit 11 service-domain inside
set groups __contrail_overlay_fip_snat__ interfaces si-0/0/0 unit 12 family inet
set groups __contrail_overlay_fip_snat__ interfaces si-0/0/0 unit 12 service-domain outside
set groups __contrail_overlay_fip_snat__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_fip_snat__ interfaces irb unit 6 family inet filter input redirect-to-_contrail_red-l3-6-nat-vrf
set groups __contrail_overlay_fip_snat__ policy-options policy-statement _contrail_red-l3-6-nat-import term t1 from community target_64512_8000005
set groups __contrail_overlay_fip_snat__ policy-options policy-statement _contrail_red-l3-6-nat-import term t1 from community target_64512_8000006
set groups __contrail_overlay_fip_snat__ policy-options policy-statement _contrail_red-l3-6-nat-export term t1 then reject
set groups __contrail_overlay_fip_snat__ policy-options community target_64512_8000005 members target:64512:8000005
set groups __contrail_overlay_fip_snat__ policy-options community target_64512_8000006 members target:64512:8000006
set groups __contrail_overlay_fip_snat__ firewall family inet filter redirect-to-_contrail_red-l3-6-nat-vrf term term-_contrail_red-l3-6-nat from source-address 192.168.10.4/32
set groups __contrail_overlay_fip_snat__ firewall family inet filter redirect-to-_contrail_red-l3-6-nat-vrf term term-_contrail_red-l3-6-nat then routing-instance _contrail_red-l3-6-nat
set groups __contrail_overlay_fip_snat__ firewall family inet filter redirect-to-_contrail_red-l3-6-nat-vrf term default-term then accept
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term term-_contrail_public-l3-8 from destination-address 172.16.10.0/24
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term term-_contrail_public-l3-8 then routing-instance _contrail_public-l3-8
set groups __contrail_overlay_fip_snat__ firewall family inet filter _contrail_redirect-to-public-vrfs-inet4 term default-term then accept
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_blue-l3-7 interface irb.7
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_public-l3-8 interface irb.8
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6 interface irb.6
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat instance-type vrf
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat interface si-0/0/0.11
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat vrf-import _contrail_red-l3-6-nat-import
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat vrf-export _contrail_red-l3-6-nat-export
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat vrf-table-label
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat routing-options static route 0.0.0.0/0 next-hop si-0/0/0.11
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6-nat routing-options auto-export family inet unicast
```

