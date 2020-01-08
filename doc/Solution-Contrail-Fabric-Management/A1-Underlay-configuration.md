* [TOC](Contrail-Fabric-Management.md#toc)

## A.1 Underlay configuration

### A.1.1 MX-1
```
set version 18.3R1.9
set system host-name vmx1
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
set interfaces ge-0/0/2 unit 0 family inet address 10.6.30.1/30
set interfaces ge-0/0/3 unit 0 family inet address 10.6.30.5/30
set interfaces fxp0 unit 0 family inet address 10.6.8.31/24
set interfaces lo0 unit 0 family inet address 10.6.0.31/32
set routing-options route-distinguisher-id 10.6.0.31
set routing-options autonomous-system 64031
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.30.2 peer-as 64021
set protocols bgp group fabric neighbor 10.6.30.6 peer-as 64022
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
```

### A.1.2 MX-2
```
set version 18.3R1.9
set system host-name vmx2
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
set interfaces ge-0/0/2 unit 0 family inet address 10.6.30.9/30
set interfaces ge-0/0/3 unit 0 family inet address 10.6.30.13/30
set interfaces fxp0 unit 0 family inet address 10.6.8.32/24
set interfaces lo0 unit 0 family inet address 10.6.0.32/32
set routing-options route-distinguisher-id 10.6.0.32
set routing-options autonomous-system 64032
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.30.14 peer-as 64022
set protocols bgp group fabric neighbor 10.6.30.10 peer-as 64021
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
```

### A.1.3 Spine-1
```
set version 18.1R1.9
set system host-name vqfx-s1
set interfaces xe-0/0/0 unit 0 family inet address 10.6.30.2/30
set interfaces xe-0/0/1 unit 0 family inet address 10.6.30.10/30
set interfaces xe-0/0/2 unit 0 family inet address 10.6.20.1/30
set interfaces xe-0/0/3 unit 0 family inet address 10.6.20.5/30
set interfaces em0 unit 0 family inet address 10.6.8.21/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.6.0.21/32
set forwarding-options storm-control-profiles default all
set routing-options route-distinguisher-id 10.6.0.21
set routing-options autonomous-system 64021
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.30.1 peer-as 64031
set protocols bgp group fabric neighbor 10.6.30.9 peer-as 64032
set protocols bgp group fabric neighbor 10.6.20.2 peer-as 64011
set protocols bgp group fabric neighbor 10.6.20.6 peer-as 64012
set protocols igmp-snooping vlan default
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
set vlans default vlan-id 1
```

### A.1.4 Spine-2
```
set version 18.1R1.9
set system host-name vqfx-s2
set interfaces xe-0/0/0 unit 0 family inet address 10.6.30.6/30
set interfaces xe-0/0/1 unit 0 family inet address 10.6.30.14/30
set interfaces xe-0/0/2 unit 0 family inet address 10.6.20.9/30
set interfaces xe-0/0/3 unit 0 family inet address 10.6.20.13/30
set interfaces em0 unit 0 family inet address 10.6.8.22/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.6.0.22/32
set forwarding-options storm-control-profiles default all
set routing-options route-distinguisher-id 10.6.0.22
set routing-options autonomous-system 64022
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.30.5 peer-as 64031
set protocols bgp group fabric neighbor 10.6.30.13 peer-as 64032
set protocols bgp group fabric neighbor 10.6.20.14 peer-as 64012
set protocols bgp group fabric neighbor 10.6.20.10 peer-as 64011
set protocols igmp-snooping vlan default
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
set vlans default vlan-id 1
```

### A.1.5 Leaf-1
```
set version 18.1R1.9
set system host-name vqfx-l1
set interfaces xe-0/0/0 unit 0 family inet address 10.6.20.2/30
set interfaces xe-0/0/1 unit 0 family inet address 10.6.20.10/30
set interfaces xe-0/0/4 unit 0 family ethernet-switching vlan members vlan-11
set interfaces em0 unit 0 family inet address 10.6.8.11/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces irb unit 11 family inet address 10.6.11.254/24
set interfaces lo0 unit 0 family inet address 10.6.0.11/32
set forwarding-options storm-control-profiles default all
set routing-options route-distinguisher-id 10.6.0.11
set routing-options autonomous-system 64011
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.20.1 peer-as 64021
set protocols bgp group fabric neighbor 10.6.20.9 peer-as 64022
set protocols igmp-snooping vlan default
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
set vlans default vlan-id 1
set vlans vlan-11 vlan-id 11
set vlans vlan-11 l3-interface irb.11
```

### A.1.6 Leaf-2
```
set version 18.1R1.9
set system host-name vqfx-l2
set interfaces xe-0/0/0 unit 0 family inet address 10.6.20.6/30
set interfaces xe-0/0/1 unit 0 family inet address 10.6.20.14/30
set interfaces em0 unit 0 family inet address 10.6.8.12/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.6.0.12/32
set forwarding-options storm-control-profiles default all
set routing-options route-distinguisher-id 10.6.0.12
set routing-options autonomous-system 64012
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.20.5 peer-as 64021
set protocols bgp group fabric neighbor 10.6.20.13 peer-as 64022
set protocols igmp-snooping vlan default
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
set vlans default vlan-id 1
```

