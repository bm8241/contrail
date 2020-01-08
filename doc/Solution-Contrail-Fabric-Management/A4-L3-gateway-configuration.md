* [TOC](ToC-Contrail-Fabric-Management)

## A.4 L3 gateway configuration

### A.4.1 L3 gateway on spine-1
```
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 7 vrf-target target:64512:8000006
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 6 vrf-target target:64512:8000005
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import term t1 from community target_64512_8000008
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-export term t1 then community add target_64512_8000008
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000008 members target:64512:8000008
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000006 members target:64512:8000006
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000005 members target:64512:8000005
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.21:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_blue-l2-7-import
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_red-l2-6-import
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_gateway__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 proxy-macip-advertisement
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 virtual-gateway-accept-data
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 family inet address 192.168.10.7/24 preferred
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 family inet address 192.168.10.7/24 virtual-gateway-address 192.168.10.1
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 virtual-gateway-v4-mac 00:00:5e:01:00:01
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 proxy-macip-advertisement
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 virtual-gateway-accept-data
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 family inet address 192.168.20.6/24 preferred
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 family inet address 192.168.20.6/24 virtual-gateway-address 192.168.20.1
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 virtual-gateway-v4-mac 00:00:5e:01:00:01
set groups __contrail_overlay_evpn_gateway__ vlans bd-6 vlan-id none
set groups __contrail_overlay_evpn_gateway__ vlans bd-6 l3-interface irb.6
set groups __contrail_overlay_evpn_gateway__ vlans bd-6 vxlan vni 6
set groups __contrail_overlay_evpn_gateway__ vlans bd-7 vlan-id none
set groups __contrail_overlay_evpn_gateway__ vlans bd-7 l3-interface irb.7
set groups __contrail_overlay_evpn_gateway__ vlans bd-7 vxlan vni 7
set groups __contrail_overlay_evpn_type5__ interfaces lo0 unit 1008 family inet address 127.0.0.1/32
set groups __contrail_overlay_evpn_type5__ routing-options forwarding-table chained-composite-next-hop ingress evpn
set groups __contrail_overlay_evpn_type5__ protocols evpn default-gateway no-gateway-community
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 from protocol direct
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 then accept
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 from protocol static
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 then accept
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 instance-type vrf
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 interface lo0.1008
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 interface irb.6
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 interface irb.7
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 vrf-import _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 vrf-export _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-export
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 routing-options rib _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8.inet6.0 multipath
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 routing-options static route 172.16.0.3/32 discard
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 routing-options multipath
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes advertise direct-nexthop
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes encapsulation vxlan
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes vni 8
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes export dummy_type5
```

```
root@vqfx-spine-1> show route table default-switch.evpn.0 

default-switch.evpn.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:10.6.0.11:7999::6::52:54:00:8b:38:40/304 MAC/IP        
                   *[BGP/170] 00:12:58, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd/304 MAC/IP        
                   *[BGP/170] 00:10:59, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.20.3 via xe-0/0/3.0
2:10.6.0.21:7999::6::00:00:5e:01:00:01/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.0.21:7999::6::02:05:86:71:2a:00/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.0.21:7999::7::00:00:5e:01:00:01/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.0.21:7999::7::02:05:86:71:2a:00/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20/304 MAC/IP        
                   *[BGP/170] 00:12:58, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
2:10.6.0.11:7999::6::52:54:00:8b:38:40::192.168.10.4/304 MAC/IP        
                   *[EVPN/170] 00:10:14
                      Indirect
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd::192.168.20.3/304 MAC/IP        
                   *[EVPN/170] 00:10:51
                      Indirect
2:10.6.0.21:7999::6::02:05:86:71:2a:00::192.168.10.1/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.0.21:7999::6::02:05:86:71:2a:00::192.168.10.7/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.0.21:7999::7::02:05:86:71:2a:00::192.168.20.1/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.0.21:7999::7::02:05:86:71:2a:00::192.168.20.6/304 MAC/IP        
                   *[EVPN/170] 00:12:26
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20::192.168.10.3/304 MAC/IP        
                   *[BGP/170] 00:12:58, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
3:10.6.0.11:7999::6::10.6.0.11/248 IM            
                   *[BGP/170] 00:12:58, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
3:10.6.0.12:7999::7::10.6.0.12/248 IM            
                   *[BGP/170] 00:12:58, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.20.3 via xe-0/0/3.0
3:10.6.0.21:7999::6::10.6.0.21/248 IM            
                   *[EVPN/170] 00:12:56
                      Indirect
3:10.6.0.21:7999::7::10.6.0.21/248 IM            
                   *[EVPN/170] 00:12:56
                      Indirect
3:10.6.11.3:2::6::10.6.11.3/248 IM            
                   *[BGP/170] 00:12:58, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
3:10.6.11.3:3::7::10.6.11.3/248 IM            
                   *[BGP/170] 00:12:58, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
3:10.6.11.4:2::6::10.6.11.4/248 IM            
                   *[BGP/170] 00:12:58, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
3:10.6.11.4:3::7::10.6.11.4/248 IM            
                   *[BGP/170] 00:12:58, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
5:10.6.11.4:4::8::192.168.10.3::32/248               
                   *[BGP/170] 00:09:34, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.1 via xe-0/0/2.0
```

### A.4.2 L3 gateway on spine-2
```
set groups __contrail_overlay_evpn__ routing-options forwarding-table export EVPN-LB
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 7 vrf-target target:64512:8000006
set groups __contrail_overlay_evpn__ protocols evpn vni-options vni 6 vrf-target target:64512:8000005
set groups __contrail_overlay_evpn__ protocols evpn encapsulation vxlan
set groups __contrail_overlay_evpn__ protocols evpn extended-vni-list all
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 from protocol evpn
set groups __contrail_overlay_evpn__ policy-options policy-statement EVPN-LB term term1 then load-balance per-packet
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import term t1 from community target_64512_8000008
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-export term t1 then community add target_64512_8000008
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-import term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_evpn__ policy-options policy-statement _contrail_red-l2-6-export term t1 then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in from community community-esi-in
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term esi-in then accept
set groups __contrail_overlay_evpn__ policy-options policy-statement import-evpn term default-term then reject
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000008 members target:64512:8000008
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000006 members target:64512:8000006
set groups __contrail_overlay_evpn__ policy-options community target_64512_8000005 members target:64512:8000005
set groups __contrail_overlay_evpn__ policy-options community community-esi-in members target:64512:7999999
set groups __contrail_overlay_evpn__ switch-options vtep-source-interface lo0.0
set groups __contrail_overlay_evpn__ switch-options route-distinguisher 10.6.0.22:7999
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_blue-l2-7-import
set groups __contrail_overlay_evpn__ switch-options vrf-import _contrail_red-l2-6-import
set groups __contrail_overlay_evpn__ switch-options vrf-import import-evpn
set groups __contrail_overlay_evpn__ switch-options vrf-target target:64512:7999999
set groups __contrail_overlay_evpn_gateway__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 proxy-macip-advertisement
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 virtual-gateway-accept-data
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 family inet address 192.168.10.8/24 preferred
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 family inet address 192.168.10.8/24 virtual-gateway-address 192.168.10.1
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 6 virtual-gateway-v4-mac 00:00:5e:01:00:01
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 proxy-macip-advertisement
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 virtual-gateway-accept-data
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 family inet address 192.168.20.5/24 preferred
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 family inet address 192.168.20.5/24 virtual-gateway-address 192.168.20.1
set groups __contrail_overlay_evpn_gateway__ interfaces irb unit 7 virtual-gateway-v4-mac 00:00:5e:01:00:01
set groups __contrail_overlay_evpn_gateway__ vlans bd-6 vlan-id none
set groups __contrail_overlay_evpn_gateway__ vlans bd-6 l3-interface irb.6
set groups __contrail_overlay_evpn_gateway__ vlans bd-6 vxlan vni 6
set groups __contrail_overlay_evpn_gateway__ vlans bd-7 vlan-id none
set groups __contrail_overlay_evpn_gateway__ vlans bd-7 l3-interface irb.7
set groups __contrail_overlay_evpn_gateway__ vlans bd-7 vxlan vni 7
set groups __contrail_overlay_evpn_type5__ interfaces lo0 unit 1008 family inet address 127.0.0.1/32
set groups __contrail_overlay_evpn_type5__ routing-options forwarding-table chained-composite-next-hop ingress evpn
set groups __contrail_overlay_evpn_type5__ protocols evpn default-gateway no-gateway-community
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 from protocol direct
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 1 then accept
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 from protocol static
set groups __contrail_overlay_evpn_type5__ policy-options policy-statement dummy_type5 term 2 then accept
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 instance-type vrf
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 interface lo0.1008
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 interface irb.6
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 interface irb.7
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 vrf-import _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-import
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 vrf-export _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8-export
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 routing-options rib _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8.inet6.0 multipath
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 routing-options static route 172.16.0.4/32 discard
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 routing-options multipath
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes advertise direct-nexthop
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes encapsulation vxlan
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes vni 8
set groups __contrail_overlay_evpn_type5__ routing-instances _contrail___contrail_lr_internal_vn_d4ec94b9-7242-4718-a563-a6d71e204118__-l3-8 protocols evpn ip-prefix-routes export dummy_type5
```

```
root@vqfx-spine-2> show route table default-switch.evpn.0 

default-switch.evpn.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:10.6.0.11:7999::6::52:54:00:8b:38:40/304 MAC/IP        
                   *[BGP/170] 00:13:38, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd/304 MAC/IP        
                   *[BGP/170] 00:11:39, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.20.7 via xe-0/0/3.0
2:10.6.0.22:7999::6::00:00:5e:01:00:01/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.0.22:7999::6::02:05:86:71:05:00/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.0.22:7999::7::00:00:5e:01:00:01/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.0.22:7999::7::02:05:86:71:05:00/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20/304 MAC/IP        
                   *[BGP/170] 00:13:39, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
2:10.6.0.11:7999::6::52:54:00:8b:38:40::192.168.10.4/304 MAC/IP        
                   *[EVPN/170] 00:10:53
                      Indirect
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd::192.168.20.3/304 MAC/IP        
                   *[EVPN/170] 00:11:31
                      Indirect
2:10.6.0.22:7999::6::02:05:86:71:05:00::192.168.10.1/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.0.22:7999::6::02:05:86:71:05:00::192.168.10.8/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.0.22:7999::7::02:05:86:71:05:00::192.168.20.1/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.0.22:7999::7::02:05:86:71:05:00::192.168.20.5/304 MAC/IP        
                   *[EVPN/170] 00:13:06
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20::192.168.10.3/304 MAC/IP        
                   *[BGP/170] 00:13:39, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
3:10.6.0.11:7999::6::10.6.0.11/248 IM            
                   *[BGP/170] 00:13:39, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
3:10.6.0.12:7999::7::10.6.0.12/248 IM            
                   *[BGP/170] 00:13:39, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.20.7 via xe-0/0/3.0
3:10.6.0.22:7999::6::10.6.0.22/248 IM            
                   *[EVPN/170] 00:13:35
                      Indirect
3:10.6.0.22:7999::7::10.6.0.22/248 IM            
                   *[EVPN/170] 00:13:35
                      Indirect
3:10.6.11.3:2::6::10.6.11.3/248 IM            
                   *[BGP/170] 00:13:39, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
3:10.6.11.3:3::7::10.6.11.3/248 IM            
                   *[BGP/170] 00:13:39, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
3:10.6.11.4:2::6::10.6.11.4/248 IM            
                   *[BGP/170] 00:13:39, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
3:10.6.11.4:3::7::10.6.11.4/248 IM            
                   *[BGP/170] 00:13:39, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
5:10.6.11.4:4::8::192.168.10.3::32/248               
                   *[BGP/170] 00:10:14, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > to 10.6.20.5 via xe-0/0/2.0
```

### A.4.3 Virtual network on MX-1
```
set groups __contrail_overlay_fip_snat__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_blue-l3-7 interface irb.7
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6 interface irb.6
set groups __contrail_overlay_networking__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_networking__ interfaces irb unit 6 family inet address 192.168.10.6/24 virtual-gateway-address 192.168.10.1
set groups __contrail_overlay_networking__ interfaces irb unit 7 family inet address 192.168.20.4/24 virtual-gateway-address 192.168.20.1
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options community target_64512_8000006 members target:64512:8000006
set groups __contrail_overlay_networking__ policy-options community target_64512_8000005 members target:64512:8000005
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 route-distinguisher 10.6.0.31:7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-import _contrail_blue-l2-7-import
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-export _contrail_blue-l2-7-export
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-target target:64512:7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 bridge-domains bd-7 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 bridge-domains bd-7 routing-interface irb.7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 bridge-domains bd-7 vxlan vni 7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 interface irb.7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 route-distinguisher 10.6.0.31:30007
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-import _contrail_blue-l3-7-import
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-export _contrail_blue-l3-7-export
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-target target:64512:30007
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 routing-options static route 192.168.20.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 routing-options auto-export family inet unicast
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 route-distinguisher 10.6.0.31:6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-import _contrail_red-l2-6-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-export _contrail_red-l2-6-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-target target:64512:6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 bridge-domains bd-6 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 bridge-domains bd-6 routing-interface irb.6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 bridge-domains bd-6 vxlan vni 6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 interface irb.6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 route-distinguisher 10.6.0.31:30006
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-import _contrail_red-l3-6-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-export _contrail_red-l3-6-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-target target:64512:30006
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 routing-options static route 192.168.10.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 routing-options auto-export family inet unicast
```

### A.4.4 Virtual network on MX-2
```
set groups __contrail_overlay_fip_snat__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_blue-l3-7 interface irb.7
set groups __contrail_overlay_fip_snat__ routing-instances _contrail_red-l3-6 interface irb.6
set groups __contrail_overlay_networking__ interfaces irb gratuitous-arp-reply
set groups __contrail_overlay_networking__ interfaces irb unit 6 family inet address 192.168.10.7/24 virtual-gateway-address 192.168.10.1
set groups __contrail_overlay_networking__ interfaces irb unit 7 family inet address 192.168.20.5/24 virtual-gateway-address 192.168.20.1
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import term t1 from community target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-export term t1 then community add target_64512_8000006
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import term t1 from community target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import term t1 then accept
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import then reject
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-export term t1 then community add target_64512_8000005
set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-export term t1 then accept
set groups __contrail_overlay_networking__ policy-options community target_64512_8000006 members target:64512:8000006
set groups __contrail_overlay_networking__ policy-options community target_64512_8000005 members target:64512:8000005
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 route-distinguisher 10.6.0.32:7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-import _contrail_blue-l2-7-import
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-export _contrail_blue-l2-7-export
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 vrf-target target:64512:7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 bridge-domains bd-7 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 bridge-domains bd-7 routing-interface irb.7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l2-7 bridge-domains bd-7 vxlan vni 7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 interface irb.7
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 route-distinguisher 10.6.0.32:30007
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-import _contrail_blue-l3-7-import
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-export _contrail_blue-l3-7-export
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-target target:64512:30007
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 routing-options static route 192.168.20.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_blue-l3-7 routing-options auto-export family inet unicast
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vtep-source-interface lo0.0
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 instance-type virtual-switch
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 route-distinguisher 10.6.0.32:6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-import _contrail_red-l2-6-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-export _contrail_red-l2-6-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 vrf-target target:64512:6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 protocols evpn encapsulation vxlan
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 protocols evpn extended-vni-list all
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 bridge-domains bd-6 vlan-id none
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 bridge-domains bd-6 routing-interface irb.6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l2-6 bridge-domains bd-6 vxlan vni 6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 instance-type vrf
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 interface irb.6
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 route-distinguisher 10.6.0.32:30006
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-import REJECT-MAINTENANCE-MODE
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-import _contrail_red-l3-6-import
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-export _contrail_red-l3-6-export
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-target target:64512:30006
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 vrf-table-label
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 routing-options static route 192.168.10.0/24 discard
set groups __contrail_overlay_networking__ routing-instances _contrail_red-l3-6 routing-options auto-export family inet unicast
```

### A.4.5 Network policy on MX-1
```
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import term t1 from community target_64512_8000005
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import term t1 from community target_64512_8000005
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000006
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import term t1 from community target_64512_8000006
```

```
root@vmx-1> show route table _contrail_red-l3-6.inet.0 

_contrail_red-l3-6.inet.0: 5 destinations, 6 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.10.0/24    *[Direct/0] 00:42:49
                    > via irb.6
                    [Static/5] 00:42:49
                      Discard
192.168.10.3/32    *[BGP/170] 00:07:13, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite, Push 23
192.168.10.6/32    *[Local/0] 00:42:49
                      Local via irb.6
192.168.20.0/24    *[Direct/0] 00:07:18
                    > via irb.7
192.168.20.4/32    *[Local/0] 00:07:18
                      Local via irb.7

root@vmx-1> show route table _contrail_red-l2-6.evpn.0    

_contrail_red-l2-6.evpn.0: 14 destinations, 14 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:10.6.0.11:7999::6::52:54:00:8b:38:40/304 MAC/IP        
                   *[BGP/170] 00:42:58, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd/304 MAC/IP        
                   *[BGP/170] 00:07:27, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
2:10.6.0.31:6::6::00:00:5e:00:01:01/304 MAC/IP        
                   *[EVPN/170] 00:42:57
                      Indirect
2:10.6.0.31:6::6::2c:6b:f5:e1:fc:f0/304 MAC/IP        
                   *[EVPN/170] 00:42:57
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20/304 MAC/IP        
                   *[BGP/170] 00:42:58, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
2:10.6.0.31:6::6::00:00:5e:00:01:01::192.168.10.1/304 MAC/IP        
                   *[EVPN/170] 00:42:57
                      Indirect
2:10.6.0.31:6::6::2c:6b:f5:e1:fc:f0::192.168.10.6/304 MAC/IP        
                   *[EVPN/170] 00:42:57
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20::192.168.10.3/304 MAC/IP        
                   *[BGP/170] 00:42:58, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.0.11:7999::6::10.6.0.11/248 IM            
                   *[BGP/170] 00:42:58, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
3:10.6.0.12:7999::7::10.6.0.12/248 IM            
                   *[BGP/170] 00:07:27, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
3:10.6.0.31:6::6::10.6.0.31/248 IM            
                   *[EVPN/170] 00:42:57
                      Indirect
3:10.6.11.3:2::6::10.6.11.3/248 IM            
                   *[BGP/170] 00:42:58, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.3:3::7::10.6.11.3/248 IM            
                   *[BGP/170] 00:07:27, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.4:2::6::10.6.11.4/248 IM            
                   *[BGP/170] 00:42:58, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite

root@vmx-1> show route table _contrail_blue-l3-7.inet.0   

_contrail_blue-l3-7.inet.0: 5 destinations, 6 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.10.0/24    *[Direct/0] 00:07:38
                    > via irb.6
192.168.10.3/32    *[BGP/170] 00:07:33, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite, Push 23
192.168.10.6/32    *[Local/0] 00:07:38
                      Local via irb.6
192.168.20.0/24    *[Direct/0] 00:32:07
                    > via irb.7
                    [Static/5] 00:32:07
                      Discard
192.168.20.4/32    *[Local/0] 00:32:07
                      Local via irb.7

root@vmx-1> show route table _contrail_blue-l2-7.evpn.0    

_contrail_blue-l2-7.evpn.0: 14 destinations, 14 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:10.6.0.11:7999::6::52:54:00:8b:38:40/304 MAC/IP        
                   *[BGP/170] 00:07:49, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd/304 MAC/IP        
                   *[BGP/170] 00:32:18, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
2:10.6.0.31:7::7::00:00:5e:00:01:01/304 MAC/IP        
                   *[EVPN/170] 00:32:17
                      Indirect
2:10.6.0.31:7::7::2c:6b:f5:e1:fc:f0/304 MAC/IP        
                   *[EVPN/170] 00:32:17
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20/304 MAC/IP        
                   *[BGP/170] 00:07:49, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
2:10.6.0.31:7::7::00:00:5e:00:01:01::192.168.20.1/304 MAC/IP        
                   *[EVPN/170] 00:32:17
                      Indirect
2:10.6.0.31:7::7::2c:6b:f5:e1:fc:f0::192.168.20.4/304 MAC/IP        
                   *[EVPN/170] 00:32:17
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20::192.168.10.3/304 MAC/IP        
                   *[BGP/170] 00:07:49, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.0.11:7999::6::10.6.0.11/248 IM            
                   *[BGP/170] 00:07:49, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
3:10.6.0.12:7999::7::10.6.0.12/248 IM            
                   *[BGP/170] 00:32:18, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.0 via ge-0/0/2.0
3:10.6.0.31:7::7::10.6.0.31/248 IM            
                   *[EVPN/170] 00:32:16
                      Indirect
3:10.6.11.3:2::6::10.6.11.3/248 IM            
                   *[BGP/170] 00:07:49, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.3:3::7::10.6.11.3/248 IM            
                   *[BGP/170] 00:32:18, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.4:2::6::10.6.11.4/248 IM            
                   *[BGP/170] 00:07:49, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
```

### A.4.6 Network policy on MX-2
```
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l2-7-import term t1 from community target_64512_8000005
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_blue-l3-7-import term t1 from community target_64512_8000005
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l2-6-import term t1 from community target_64512_8000006
+set groups __contrail_overlay_networking__ policy-options policy-statement _contrail_red-l3-6-import term t1 from community target_64512_8000006
```

```
root@vmx-2> show route table _contrail_red-l3-6.inet.0 

_contrail_red-l3-6.inet.0: 5 destinations, 6 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.10.0/24    *[Direct/0] 00:40:19
                    > via irb.6
                    [Static/5] 00:40:20
                      Discard
192.168.10.3/32    *[BGP/170] 00:04:44, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite, Push 23
192.168.10.7/32    *[Local/0] 00:40:19
                      Local via irb.6
192.168.20.0/24    *[Direct/0] 00:04:49
                    > via irb.7
192.168.20.5/32    *[Local/0] 00:04:49
                      Local via irb.7

root@vmx-2> show route table _contrail_red-l2-6.evpn.0    

_contrail_red-l2-6.evpn.0: 14 destinations, 14 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:10.6.0.11:7999::6::52:54:00:8b:38:40/304 MAC/IP        
                   *[BGP/170] 00:40:44, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd/304 MAC/IP        
                   *[BGP/170] 00:05:13, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
2:10.6.0.32:6::6::00:00:5e:00:01:01/304 MAC/IP        
                   *[EVPN/170] 00:40:43
                      Indirect
2:10.6.0.32:6::6::2c:6b:f5:e1:fc:f0/304 MAC/IP        
                   *[EVPN/170] 00:40:43
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20/304 MAC/IP        
                   *[BGP/170] 00:40:44, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
2:10.6.0.32:6::6::00:00:5e:00:01:01::192.168.10.1/304 MAC/IP        
                   *[EVPN/170] 00:40:43
                      Indirect
2:10.6.0.32:6::6::2c:6b:f5:e1:fc:f0::192.168.10.7/304 MAC/IP        
                   *[EVPN/170] 00:40:43
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20::192.168.10.3/304 MAC/IP        
                   *[BGP/170] 00:40:44, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.0.11:7999::6::10.6.0.11/248 IM            
                   *[BGP/170] 00:40:44, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
3:10.6.0.12:7999::7::10.6.0.12/248 IM            
                   *[BGP/170] 00:05:13, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
3:10.6.0.32:6::6::10.6.0.32/248 IM            
                   *[EVPN/170] 00:40:42
                      Indirect
3:10.6.11.3:2::6::10.6.11.3/248 IM            
                   *[BGP/170] 00:40:44, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.3:3::7::10.6.11.3/248 IM            
                   *[BGP/170] 00:05:13, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.4:2::6::10.6.11.4/248 IM            
                   *[BGP/170] 00:40:44, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite

root@vmx-2> show route table _contrail_blue-l3-7.inet.0   

_contrail_blue-l3-7.inet.0: 5 destinations, 6 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

192.168.10.0/24    *[Direct/0] 00:05:27
                    > via irb.6
192.168.10.3/32    *[BGP/170] 00:05:22, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite, Push 23
192.168.10.7/32    *[Local/0] 00:05:27
                      Local via irb.6
192.168.20.0/24    *[Direct/0] 00:29:55
                    > via irb.7
                    [Static/5] 00:29:56
                      Discard
192.168.20.5/32    *[Local/0] 00:29:55
                      Local via irb.7

root@vmx-2> show route table _contrail_blue-l2-7.evpn.0    

_contrail_blue-l2-7.evpn.0: 14 destinations, 14 routes (14 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2:10.6.0.11:7999::6::52:54:00:8b:38:40/304 MAC/IP        
                   *[BGP/170] 00:05:42, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
2:10.6.0.12:7999::7::52:54:00:7d:b4:bd/304 MAC/IP        
                   *[BGP/170] 00:30:11, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
2:10.6.0.32:7::7::00:00:5e:00:01:01/304 MAC/IP        
                   *[EVPN/170] 00:30:10
                      Indirect
2:10.6.0.32:7::7::2c:6b:f5:e1:fc:f0/304 MAC/IP        
                   *[EVPN/170] 00:30:10
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20/304 MAC/IP        
                   *[BGP/170] 00:05:42, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
2:10.6.0.32:7::7::00:00:5e:00:01:01::192.168.20.1/304 MAC/IP        
                   *[EVPN/170] 00:30:10
                      Indirect
2:10.6.0.32:7::7::2c:6b:f5:e1:fc:f0::192.168.20.5/304 MAC/IP        
                   *[EVPN/170] 00:30:10
                      Indirect
2:10.6.11.4:2::6::02:16:10:87:1a:20::192.168.10.3/304 MAC/IP        
                   *[BGP/170] 00:05:42, MED 100, localpref 200, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.0.11:7999::6::10.6.0.11/248 IM            
                   *[BGP/170] 00:05:42, localpref 100, from 10.6.0.11
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
3:10.6.0.12:7999::7::10.6.0.12/248 IM            
                   *[BGP/170] 00:30:11, localpref 100, from 10.6.0.12
                      AS path: I, validation-state: unverified
                    > to 10.6.30.2 via ge-0/0/2.0
3:10.6.0.32:7::7::10.6.0.32/248 IM            
                   *[EVPN/170] 00:30:09
                      Indirect
3:10.6.11.3:2::6::10.6.11.3/248 IM            
                   *[BGP/170] 00:05:42, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.3:3::7::10.6.11.3/248 IM            
                   *[BGP/170] 00:30:11, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
3:10.6.11.4:2::6::10.6.11.4/248 IM            
                   *[BGP/170] 00:05:42, MED 200, localpref 100, from 10.6.11.1
                      AS path: ?, validation-state: unverified
                    > via Tunnel Composite
```

