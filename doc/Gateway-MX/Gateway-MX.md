# Solution Guide - Gateway MX

# 1 Overview

This guide is about how to use MX as the gateway to provide external or underlay connectivity for the overlay managed by Contrail.

Here is a typical topology. Depends on performance requirments, gateway can connect to either spine or leaf.

![Figure Overview](Figure-Overview.png)


# 2 Underlay/INET

## 2.1 eBGP

A typical IP fabric uses eBGP on all leaves, spines and gateways to build underlay connectivity.


## 2.2 iBGP

In case of iBGP, RR (Route Reflector) is recommended to avoid fully meshed peering between all BGP nodes.


# 3 Overlay/VPN

## 3.1 Loopback Address

A loopback address is allocated and assigned on each MX. It's used for BGP peering with control node and tunneling with vrouter. The connectvity between Contrail and loopback address is provided by underlay.

In case separate interfaces are used for control and data planes, the address of control interface will be used as the next-hop, when MX advertises route. To resolve this issue, a loopback interface should be used for both control and data planes.

```
set interfaces lo0 unit 0 family inet address 10.6.0.31/32
```


## 3.2 BGP

### 3.2.1 AS

Typically, the gateway has a single global unique ASN.
```
set routing-options autonomous-system 64031
```


### 3.2.2 eBGP and iBGP

eBGP is used when Contrail and gateway are in different AS.
```
set protocols bgp group vpn-contrail type external
set protocols bgp group vpn-contrail multihop
set protocols bgp group vpn-contrail local-address 10.6.0.31
set protocols bgp group vpn-contrail keep all
set protocols bgp group vpn-contrail family inet-vpn unicast
set protocols bgp group vpn-contrail family evpn signaling
set protocols bgp group vpn-contrail family route-target
set protocols bgp group vpn-contrail neighbor 10.6.11.1 peer-as 64512
```

iBGP is used when Contrail and gateway are in the same AS.
```
set protocols bgp group vpn-contrail type internal
set protocols bgp group vpn-contrail local-address 10.6.0.31
set protocols bgp group vpn-contrail keep all
set protocols bgp group vpn-contrail family inet-vpn unicast
set protocols bgp group vpn-contrail family evpn signaling
set protocols bgp group vpn-contrail family route-target
set protocols bgp group vpn-contrail neighbor 10.6.11.1
```

`local-as` can be used to enable iBGP when gateway global ASN is different from Contrail ASN.
```
set protocols bgp group vpn-contrail type internal
set protocols bgp group vpn-contrail local-address 10.6.0.31
set protocols bgp group vpn-contrail local-as 64512
set protocols bgp group vpn-contrail keep all
set protocols bgp group vpn-contrail family inet-vpn unicast
set protocols bgp group vpn-contrail family evpn signaling
set protocols bgp group vpn-contrail family route-target
set protocols bgp group vpn-contrail neighbor 10.6.11.1 peer-as 64512
```


## 3.3 BGP Family

### 3.3.1 L3VPN
```
set protocols bgp group vpn-contrail family inet-vpn unicast
```


### 3.3.2 EVPN
```
set protocols bgp group vpn-contrail family evpn signaling
```


### 3.3.3 Route Target

```
set protocols bgp group vpn-contrail family route-target
```

Family "route-target" is for optimization purpose. When it's configured on MX, MX will advertise route-target route when there is VRF import policy. MX also checks route-target route table before advertising VPN-IPv4 route to a neighbor. If the route target in the route is not advertised by the neighbor, MX won't advertise the route.

In case separate interfaces on control and data planes, MX receives route-target route from Contrail control node. The next-hop of RT route is control node address (on control plane). MX tries to resolve the next-hop in MPLS table (inet.3) that is on data plane, and fails. So that RT route is not applied and hidden. That results MX doesn't advertise routes. A static route can be added into inet.3 to make the next-hop of control interface resolvable. Then MX applies the RT route and advertise routes. OpenContrail doesn't have such issue, because it doesn't try to resolve the next-hop.


## 3.4 Tunnel

Tunnel service has to be enabled. Here is an example.
```
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
```

### 3.4.1 MPLSoGRE Tunnel

For L3VPN, after BGP receives an INET-VPN route and puts it in table `bgp.l3vpn.0`, it looks for a MPLS path for that route. BGP tries to resolve the route in table `inet.3`. If it's successful, a GRE tunnle will be created and a MPLS route will be added in `inet.3`. Otherwise, the route will be hidden in `bgp.l3vpn.0`.

When enable the tunnel, routes of `destination-networks` are added into `inet.3`. Here is an example.
```
set routing-options dynamic-tunnels contrail source-address 10.6.0.31
set routing-options dynamic-tunnels contrail gre
set routing-options dynamic-tunnels contrail destination-networks 10.6.11.0/24
```
`source-address` is the loopback address.

Here is an example of GRE tunnel route in table `inet.3`.
```
10.6.11.4/32 (1 entry, 1 announced)
        *Tunnel Preference: 300
                Next hop type: Router, Next hop index: 0
                Address: 0xd7a9210
                Next-hop reference count: 3
                Next hop: via gr-0/0/0.32769, selected
                Session Id: 0x0
                State: <Active>
                Local AS: 64031 
                Age: 10 
                Validation State: unverified 
                Task: DYN_TUNNEL
                Announcement bits (2): 0-Resolve tree 1 1-Resolve_IGP_FRR task 
                AS path: I 
```

Here is dynamic tunnel database.
```
> show dynamic-tunnels database    
*- Signal Tunnels #- PFE-down
Table: inet.3       

Destination-network: 10.6.11.0/24
Tunnel to: 10.6.11.1/32 State: Up (expires in 00:06:58 seconds)
  Reference count: 0
  Next-hop type: gre
    Source address: 10.6.0.31
    Next hop: gr-0/0/10.32769
      State: Up
Tunnel to: 10.6.11.7/32 State: Up
  Reference count: 2
  Next-hop type: gre
    Source address: 10.6.0.31
    Next hop: gr-0/0/10.32770
      State: Up
```


### 3.4.2 MPLSoUDP Tunnel

UDP tunnel is better for loadbalancing.

```
set routing-options dynamic-tunnels contrail source-address 10.6.0.31
set routing-options dynamic-tunnels contrail udp
set routing-options dynamic-tunnels contrail destination-networks 10.6.11.0/24
```

Here is an example of UDP tunnel route in table `inet.3`.
```
10.6.11.4/32 (1 entry, 1 announced)
        *Tunnel Preference: 300
                Next hop type: Tunnel Composite, Next hop index: 0
                Address: 0xd7a87f0
                Next-hop reference count: 2
                Tunnel type: UDP, Reference count: 5, nhid: 0
                Destination address: 10.6.11.4, Source address: 10.6.0.31
                State: <Active>
                Local AS: 64031 
                Age: 24:46 
                Validation State: unverified 
                Task: DYN_TUNNEL
                Announcement bits (2): 0-Resolve tree 1 1-Resolve_IGP_FRR task 
                AS path: I 
```

Policy is required to append encapsulation community when export routes from VRF to Contrail.
```
set policy-options policy-statement vrf-export-provider-1 term t1 then community add provider-1
set policy-options policy-statement vrf-export-provider-1 term t1 then community add encap-udp
set policy-options policy-statement vrf-export-provider-1 term t1 then accept
set policy-options community provider-1 members target:64512:101
set policy-options community encap-udp members encapsulation:64512:13
```


## 3.5 Routing Instance

### 3.5.1 VRF

`vrf` type of RI is for holding L3 routes.
```
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 interface lo0.11
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-target target:64512:101;
set routing-instances provider-1 vrf-table-label
```

### 3.5.2 Virtual Switch


# 4 Route Import/Export

## 4.1 Workflow

### 4.1.1 Import

* First of all, BGP establishes peering with Contrail. Without any VRF RI and import policy, table `bgp.l3vpn.0` is not created and BGP doesn't receive any INET-VPN route.

* When VRF RI is created (`vrf-table-label` has to be configured.), either implicit policy or explicit policy can be used.

  *  Configure `vrf-target` will enable implicit policy that import route with specific RT community and export route with appending specific RT community.

  * Configure 'vrf-import' and 'vrf-export' to specify explicit policies, in case any additional manipulations are required.

* With any VRF RI and import policy, table `bgp.l3vpn.0` is created.

* Based on import policy, a RIB group `vpn-unicast` is created for each RT.
```
vpn-unicast target:64512:101, Address: 0xd7a8e40
  Address Family: l3vpn, Flags: 0x4, References: 0
  Export RIB: l3vpn.0
  Import RIB: bgp.l3vpn.0
  Secondary Import RIB: provider-1.inet.0
```

* BGP tries to resolve the route in table `inet.3`. If it's successful, a GRE tunnel is allocated. Otherwise, the route is hidden.

* INET-VPN routes matching import policy (route-target community) are received by BGP and placed in table `bgp.l3vpn.0`. Route are also converted to INET route and placed in VRF table who is the secondary import RIB in RIB group. Otherwise, routes are discarded.

Here is an example of INET-VPN route in table `bgp.l3vpn.0`. It is advertised from Contrail by BGP. Route distinguisher `10.6.11.4:2` is composed of vrouter IP address and an ID allocated by vrouter. It's advertised from Contrail control node 10.6.11.1. The next-hop is via dynamic GRE tunnel interface gr-0/0/0.32769. MPLS label is 25.
```
10.6.11.4:2:172.16.11.3/32                
                   *[BGP/170] 00:03:11, MED 100, localpref 100, from 10.6.11.1
                      AS path: 64512 ?, validation-state: unverified
                    > via gr-0/0/0.32769, Push 25
```

The route is converted to INET route and placed in VRF.
```
172.16.11.3/32     *[BGP/170] 02:35:37, MED 100, localpref 100, from 10.6.11.1
                      AS path: 64512 ?, validation-state: unverified
                    > via gr-0/0/0.32769, Push 25
```


### 4.1.2 Export

* To export route from VRF, based on export policy, the route will be converted from INET to INET-VPN, put into table `bgp.l3vpn.0` and  exported by BGP. A MPLS label will be allocated in table `mpls.0` for the INET-VPN route.
 
This is a loopback interface in the VRF showing in table `bgp.l3vpn.0`.
```
64512:101:172.16.11.250/32                
                   *[Direct/0] 00:43:14
                    > via lo0.11
```

The route is advertised with MPLS label 300624 showing by "show route advertising-protocol bgp 10.6.11.1 detail".
```
* 64512:101:172.16.11.250/32 (1 entry, 1 announced)
 BGP group vpn-contrail type External
     Route Distinguisher: 64512:101
     VPN Label: 300624
     Nexthop: Self
     Flags: Nexthop Change
     AS path: [64031] I
```

The MPLS label is allocated in table `mpls.0`.
```
300624             *[VPN/170] 00:55:34
                      receive table provider-1.inet.0, Pop
```


## 4.2 Implicit VRF Import/Export Policy

With `vrf-target`, implicit import and export policies are created.
```
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 vrf-table-label
set routing-instances provider-1 vrf-target target:64512:101;
```

Implicit import policy imports route who has community 'target:64540:100'. As the result, routes advertised from Contrail virtual network with 'target:64540:100' are imported into this RI.
```
> show policy __vrf-import-5b4s37-166-internal__ 
Policy __vrf-import-5b4s37-166-internal__:
    Term unnamed:
        from community __vrf-community-5b4s37-166-common-internal__ [target:64540:100 ]
        then accept
    Term unnamed:
        then reject
```

Implicit export policy exports route with community 'target:64540:100'. As the result, routes are advertised to Contrail and imported into virtual network who has 'target:64540:100'.
```
> show policy __vrf-export-5b4s37-166-internal__ 
Policy __vrf-export-5b4s37-166-internal__:
    Term unnamed:
        then community + __vrf-community-5b4s37-166-common-internal__ [target:64540:100 ] accept
```


## 4.3 Explicit VRF Import/Export Policy

Policies can be explicitly defined to import and export routes. In this example, routes advertised from Contrail virtual network with 'target:64540:91' and 'target:64540:92' are imported into the RI. Routes in RI are advertised with 'target:64540:91' and 'target:64540:92' and imported into both virtual networks.
```
set policy-options policy-statement provider-1-export term t1 then community add provider-1
set policy-options policy-statement provider-1-export term t1 then accept
set policy-options policy-statement provider-1-import term t1 from community provider-1
set policy-options policy-statement provider-1-import term t1 from community ext-host
set policy-options policy-statement provider-1-import term t1 then accept
set policy-options community ext-host members target:64510:101
set policy-options community provider-1 members target:64512:101
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 interface lo0.11
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-table-label
set routing-instances provider-1 vrf-import provider-1-import
set routing-instances provider-1 vrf-export provider-1-export
```


# 5 External/Underlay Connectivity

Here is the idea.
* Having routes in master RI to steer ingress traffic (from external/underlay to overlay) to VRF RI.
* Having routes in VRF RI to steer egress traffic (from overlay to external/underlay) to master RI.
* Routes can be either leaked to static.

There are two working options.
1. Logical tunnel
2. RIB group and static route with next-table

The following sections are the details.


## 5.1 Logical Tunnel

Logical tunnel is for connection master routing instances and VRF routing instance. This is optional depending on use case. Due to the bandwidth limitation, have to check requirement and tunnel bandwidth on specific hardware to make decision.


### 5.1.1 Static

Here is an example to use static route on logical tunnel.
```
set chassis fpc 0 pic 0 tunnel-services
set interfaces lt-0/0/0 unit 100 encapsulation frame-relay
set interfaces lt-0/0/0 unit 100 dlci 10
set interfaces lt-0/0/0 unit 100 peer-unit 200
set interfaces lt-0/0/0 unit 100 family inet
set interfaces lt-0/0/0 unit 200 encapsulation frame-relay
set interfaces lt-0/0/0 unit 200 dlci 10
set interfaces lt-0/0/0 unit 200 peer-unit 100
set interfaces lt-0/0/0 unit 200 family inet
set routing-options static route 172.16.11.0/24 next-hop lt-0/0/0.100
set routing-instances provider-1 interface lt-0/0/0.200
set routing-instances provider-1 routing-options static route 0.0.0.0/0 next-hop lt-0/0/0.200
```


### 5.1.2 Dynamic

Here is an example to configure BGP peering between VRF and master with aggregate route.
```
set chassis fpc 0 pic 0 tunnel-services
set interfaces lt-0/0/0 unit 100 encapsulation frame-relay
set interfaces lt-0/0/0 unit 100 dlci 10
set interfaces lt-0/0/0 unit 100 peer-unit 200
set interfaces lt-0/0/0 unit 100 family inet address 192.168.200.0/31
set interfaces lt-0/0/0 unit 200 encapsulation frame-relay
set interfaces lt-0/0/0 unit 200 dlci 10
set interfaces lt-0/0/0 unit 200 peer-unit 100
set interfaces lt-0/0/0 unit 200 family inet address 192.168.200.1/31
set protocols bgp group vrf type internal
set protocols bgp group vrf local-address 192.168.200.0
set protocols bgp group vrf keep all
set protocols bgp group vrf family inet unicast
set protocols bgp group vrf export provider-1-export
set protocols bgp group vrf neighbor 192.168.200.1
set policy-options policy-statement provider-1-export term t1 then community add provider-1
set policy-options policy-statement provider-1-export term t1 then accept
set policy-options policy-statement provider-1-aggregate-export term 1 from protocol aggregate
set policy-options policy-statement provider-1-aggregate-export term 1 from route-filter 172.16.11.0/24 exact
set policy-options policy-statement provider-1-aggregate-export term 1 then next-hop self
set policy-options policy-statement provider-1-aggregate-export term 1 then accept
set policy-options community provider-1 members target:64512:101
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 interface lt-0/0/0.200
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-import provider-1-import
set routing-instances provider-1 vrf-export provider-1-export
set routing-instances provider-1 routing-options aggregate route 172.16.11.0/24
set routing-instances provider-1 protocols bgp group master type internal
set routing-instances provider-1 protocols bgp group master local-address 192.168.200.1
set routing-instances provider-1 protocols bgp group master keep all
set routing-instances provider-1 protocols bgp group master family inet unicast
set routing-instances provider-1 protocols bgp group master export provider-1-aggregate-export
set routing-instances provider-1 protocols bgp group master neighbor 192.168.200.0
```


## 5.2 Next-table

A route table can be specified as the route next-hop. Conceptually, traffic can be steered between `inet.0` and `vrf.inet.0` like this example.
```
+--------------------------------------+   +-----------------------------+
| inet.0                               |   | vrf.inet.0                  |
| 172.16.11.0/24 next-table vrf.inet.0 |-->|                             |
|                                      |<--| 0.0.0.0/0 next-table inet.0 |
+--------------------------------------+   +-----------------------------+
```

The problem of this solution is that it will cause routing loop. For example, traffic heading 172.16.11.9 is steered to `vrf.inet.0`. If it's not resolved by any specific route, it will be returned back to `inet.0` by the default route. To avoid such routing loop, such configuration is not allowed by Junos.

Having the third table is also not allowed by Junos.


## 5.3 RIB Group

RIB group is typically for leaking route between routing tables. Conceptually, a RIB group can be created to import INET route from `vrf.inet.0` to `inet.0`, and another RIB group to import INET route from `inet.0` to `vrf.inet.0`.

```
set routing-options rib-groups provider-1-master import-rib provider-1.inet.0
set routing-options rib-groups provider-1-master import-rib inet.0
set routing-options rib-groups master-provider-1 import-rib inet.0
set routing-options rib-groups master-provider-1 import-rib provider-1.inet.0
set protocols bgp group corp type external
set protocols bgp group corp family inet unicast rib-group master-provider-1
set protocols bgp group corp export direct
set protocols bgp group corp neighbor 10.6.30.1 peer-as 64041
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-import provider-1-import
set routing-instances provider-1 vrf-export provider-1-export
set routing-instances provider-1 vrf-table-label
set routing-instances provider-1 routing-options auto-export family inet unicast rib-group provider-1-master
```
This configuration gets routes leaked from `inet.0` to `vpn.inet.0`. But on another way around, the routes received from Contrail are not leaked from `vpn.inet.0` to `inet.0`. This is because of Junos design. Those routes are already leaked from `bgp.l3vpn.0`, so `vpn.inet.0` is the secondary RIB for those routes. The routes in the secondary RIB can't be leaked again.


## 5.4 RIB Group and Next-table

### 5.4.1 Ingress

For ingress traffic, because Junos doesn't leak overlay /32 routes from VRF to master, here are two options.

1. Add generate (aggregate) route in VRF and use RIB group to leak aggregate route from `vrf.inet.0` to `inet.0`.
```
set routing-options rib-groups provider-1-master import-rib provider-1.inet.0
set routing-options rib-groups provider-1-master import-rib inet.0
set routing-options rib-groups provider-1-master import-policy provider-1-master-import
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-target target:64512:101
set routing-instances provider-1 vrf-table-label
set routing-instances provider-1 routing-options static route 0.0.0.0/0 next-table inet.0
set routing-instances provider-1 routing-options generate route 172.16.11.0/24 next-table provider-1.inet.0
set routing-instances provider-1 routing-options auto-export family inet unicast rib-group provider-1-master
```

2. Add static route with next-table to `vrf.inet.0` in master.
```
set routing-options static route 172.16.11.0/24 next-table provider-1.inet.0
```

Option 2 is recommended.

Note, export policy needs to be updated for routing protocol to advertise such static route.


### 5.4.2 Egress

For egress traffic, here are two options.

1. Add static route with next-table to `inet.0` in VRF.
```
set routing-instances provider-1 routing-options static route 0.0.0.0/0 next-table inet.0
```

The issue here is that, if it's default route like above, it will cause routing loop. For example, ingress traffic to 172.16.11.5/32 that doesn't exist in `vrf.int.0` will be looping between master and VRF. Using specific route can avoid routing loop, but that's not dynamic and doesn't scale.

2. Leak routes received by routing protocol in master to VRF.
```
set protocols bgp group corp type external
set protocols bgp group corp family inet unicast rib-group bgp-corp-provider-1
set protocols bgp group corp export direct
set protocols bgp group corp neighbor 10.6.30.1 peer-as 64041
set routing-options rib-groups bgp-corp-provider-1 import-rib inet.0
set routing-options rib-groups bgp-corp-provider-1 import-rib provider-1.inet.0
```

Again, due to Junos constraint, the route leaked into VRF (secondary RIB) can't be advertised to Contrail. The solution is to add a default reject route.
```
set routing-instances provider-1 routing-options static route 0.0.0.0/0 reject
```


### 5.4.3 Solution

As a conslusion, here is the solution.
* Leak route from master to VRF for egress traffic.
* Add static route in master for ingress traffic.

Appendex A.1 is the complete configuration.

Note, this doesn't work with MPLSoUDP.


## 5.5 Forwarding Filter and Next-table

This solution is to use forwarding filter to steer ingress traffic to VRF RI, and use static route with next-table to steer egress traffic to master RI.

There are two issues with this solution.
1. It doesn't work with MPLSoUDP, due to some issue in Junos.
2. To advertise the route to external, a route pointing to gateway itself has to be added. The ingress traffic will hit the filter first, so that static route is only for advertising purpose, it has no effect on traffic.


## 5.6 VRF to VRF

Appendix A.2 is an example configuration.

Note, due to family route-target, in Contrail, the remote VRF RT has to be configured as import RT in the exposed VN. Otherwise, gateway won't advertise INET-VPN routes from remote VRF.


## 5.7 Community

The route from Contrail has the following communities.
* route target
* encapsulation
* mac-mobility
* 0x8004 (security group)
* 0x8071 (origin VN)

Depend on use case, like the route going to external or another Contrail cluster, those communities may or may not need to be cleaned up.

Configuration in A.2 is an example of cleaning up communities.


# 6 Mulitple Clusters
One gateway can support multiple clusters who should have different ASN.

* One ASN for the gateway.
* Clusters have different private ASNs.
* iBGP among control nodes within each cluster.
* eBGP between gateway and control nodes of each cluster.
* Multiple BGP groups can share the same interface connecting to different set of neighbors.
* One dynamic tunnel groups for each cluster if they are in separate networks.
* Each cluster should have separate public address space. Since no address collision, one VRF routing instance can be shared by multiple clusters. And the public virtual network in all clusters have to have the same routing target. As a result, public route from one cluster will be leaked to another cluster.


# Appendix

## A.1 RIB Group and Next-table

```
set version 18.3R1.9
set chassis fpc 0 pic 0 tunnel-services
set interfaces ge-0/0/0 mac 52:54:00:8c:f9:2b
set interfaces ge-0/0/0 unit 0 family inet address 10.6.30.2/30
set interfaces ge-0/0/1 mac 52:54:00:c4:ee:41
set interfaces ge-0/0/1 unit 0 family inet address 10.6.20.1/30
set interfaces fxp0 unit 0 family inet address 10.6.8.31/24
set interfaces lo0 unit 0 family inet address 10.6.0.31/32
set interfaces lo0 unit 11 family inet address 172.16.11.250/32
set interfaces lo0 unit 12 family inet address 172.16.12.250/32
set routing-options interface-routes rib-group inet master-direct-vrf
set routing-options static route 172.16.11.0/24 next-table provider-1.inet.0
set routing-options static route 172.16.12.0/24 next-table provider-2.inet.0
set routing-options rib-groups bgp-corp-vrf import-rib inet.0
set routing-options rib-groups bgp-corp-vrf import-rib provider-1.inet.0
set routing-options rib-groups bgp-corp-vrf import-rib provider-2.inet.0
set routing-options rib-groups master-direct-vrf import-rib inet.0
set routing-options rib-groups master-direct-vrf import-rib provider-1.inet.0
set routing-options rib-groups master-direct-vrf import-rib provider-2.inet.0
set routing-options rib-groups master-direct-vrf import-policy rib-import-master-vrf
set routing-options route-distinguisher-id 10.6.0.31
set routing-options autonomous-system 64031
set routing-options dynamic-tunnels contrail source-address 10.6.0.31
set routing-options dynamic-tunnels contrail gre
set routing-options dynamic-tunnels contrail destination-networks 10.6.11.0/24
set protocols bgp group corp type external
set protocols bgp group corp family inet unicast rib-group bgp-corp-vrf
set protocols bgp group corp export direct
set protocols bgp group corp neighbor 10.6.30.1 peer-as 64041
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.20.2 peer-as 64011
set protocols bgp group vpn-contrail type external
set protocols bgp group vpn-contrail multihop
set protocols bgp group vpn-contrail local-address 10.6.0.31
set protocols bgp group vpn-contrail keep all
set protocols bgp group vpn-contrail family inet-vpn unicast
set protocols bgp group vpn-contrail family route-target
set protocols bgp group vpn-contrail neighbor 10.6.11.1 peer-as 64512
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 from protocol aggregate
set policy-options policy-statement direct term t1 then accept
set policy-options policy-statement direct term t2 from protocol static
set policy-options policy-statement direct term t2 from route-filter 172.16.11.0/24 exact
set policy-options policy-statement direct term t2 then accept
set policy-options policy-statement direct term t3 from protocol static
set policy-options policy-statement direct term t3 from route-filter 172.16.12.0/24 exact
set policy-options policy-statement direct term t3 then accept
set policy-options policy-statement rib-import-master-vrf term t2 from protocol direct
set policy-options policy-statement rib-import-master-vrf term t2 then accept
set policy-options policy-statement rib-import-master-vrf term end then reject
set policy-options policy-statement vrf-export-provider-1 term t1 then community add provider-1
set policy-options policy-statement vrf-export-provider-1 term t1 then accept
set policy-options policy-statement vrf-export-provider-1 term end then reject
set policy-options policy-statement vrf-export-provider-2 term t1 then community add provider-2
set policy-options policy-statement vrf-export-provider-2 term t1 then accept
set policy-options policy-statement vrf-export-provider-2 term end then reject
set policy-options policy-statement vrf-import-provider-1 term t1 from community provider-1
set policy-options policy-statement vrf-import-provider-1 term t1 from community ext-host
set policy-options policy-statement vrf-import-provider-1 term t1 then accept
set policy-options policy-statement vrf-import-provider-1 term end then reject
set policy-options policy-statement vrf-import-provider-2 term t1 from community provider-2
set policy-options policy-statement vrf-import-provider-2 term t1 from community ext-host
set policy-options policy-statement vrf-import-provider-2 term t1 then accept
set policy-options policy-statement vrf-import-provider-2 term end then reject
set policy-options community all-encaps members encapsulation:*:*
set policy-options community all-origin-vns members 0x8071:*:*
set policy-options community all-security-groups members 0x8004:*:*
set policy-options community encap-udp members encapsulation:64512:13
set policy-options community ext-host members target:64510:101
set policy-options community provider-1 members target:64512:101
set policy-options community provider-2 members target:64512:102
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 interface lo0.11
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-import vrf-import-provider-1
set routing-instances provider-1 vrf-export vrf-export-provider-1
set routing-instances provider-1 vrf-table-label
set routing-instances provider-1 routing-options static route 0.0.0.0/0 reject
set routing-instances provider-2 instance-type vrf
set routing-instances provider-2 interface lo0.12
set routing-instances provider-2 route-distinguisher 64512:102
set routing-instances provider-2 vrf-import vrf-import-provider-2
set routing-instances provider-2 vrf-export vrf-export-provider-2
set routing-instances provider-2 vrf-table-label
set routing-instances provider-2 routing-options static route 0.0.0.0/0 reject
```

## A.2 VRF to VRF

```
set version 18.3R1.9
set chassis fpc 0 pic 0 tunnel-services
set interfaces ge-0/0/0 mac 52:54:00:8c:f9:2b
set interfaces ge-0/0/0 unit 0 family inet address 10.6.30.2/30
set interfaces ge-0/0/1 mac 52:54:00:c4:ee:41
set interfaces ge-0/0/1 unit 0 family inet address 10.6.20.1/30
set interfaces fxp0 unit 0 family inet address 10.6.8.31/24
set interfaces lo0 unit 0 family inet address 10.6.0.31/32
set routing-options route-distinguisher-id 10.6.0.31
set routing-options autonomous-system 64031
set routing-options dynamic-tunnels contrail source-address 10.6.0.31
set routing-options dynamic-tunnels contrail gre
set routing-options dynamic-tunnels contrail destination-networks 10.6.11.0/24
set routing-options dynamic-tunnels contrail destination-networks 10.6.0.0/16
set protocols bgp group corp type external
set protocols bgp group corp family inet unicast
set protocols bgp group corp export direct
set protocols bgp group corp neighbor 10.6.30.1 peer-as 64041
set protocols bgp group fabric type external
set protocols bgp group fabric family inet unicast
set protocols bgp group fabric export direct
set protocols bgp group fabric neighbor 10.6.20.2 peer-as 64011
set protocols bgp group vpn-contrail type external
set protocols bgp group vpn-contrail multihop
set protocols bgp group vpn-contrail local-address 10.6.0.31
set protocols bgp group vpn-contrail keep all
set protocols bgp group vpn-contrail family inet-vpn unicast
set protocols bgp group vpn-contrail family route-target
set protocols bgp group vpn-contrail neighbor 10.6.11.1 peer-as 64512
set protocols bgp group vpn-external type external
set protocols bgp group vpn-external multihop
set protocols bgp group vpn-external local-address 10.6.0.31
set protocols bgp group vpn-external keep all
set protocols bgp group vpn-external family inet-vpn unicast
set protocols bgp group vpn-external family route-target
set protocols bgp group vpn-external export vpn-external-export
set protocols bgp group vpn-external neighbor 10.6.0.41 peer-as 64041
set policy-options policy-statement direct term t1 from protocol direct
set policy-options policy-statement direct term t1 then accept
set policy-options policy-statement provider-1-export term t1 then accept
set policy-options policy-statement provider-1-import term t1 from community provider-1
set policy-options policy-statement provider-1-import term t1 from community ext-host
set policy-options policy-statement provider-1-import term t1 then accept
set policy-options policy-statement vpn-external-export term t1 from community provider-1
set policy-options policy-statement vpn-external-export term t1 then community add ext-host
set policy-options policy-statement vpn-external-export term t1 then community delete all-encaps
set policy-options policy-statement vpn-external-export term t1 then community delete all-security-groups
set policy-options policy-statement vpn-external-export term t1 then community delete all-origin-vns
set policy-options policy-statement vpn-external-export term t1 then accept
set policy-options community all-encaps members encapsulation:*:*
set policy-options community all-origin-vns members 0x8071:*:*
set policy-options community all-security-groups members 0x8004:*:*
set policy-options community ext-host members target:64510:101
set policy-options community provider-1 members target:64512:101
set firewall family inet filter to-vrf term 1 from destination-address 172.16.11.0/24
set firewall family inet filter to-vrf term 1 then routing-instance provider-1
set firewall family inet filter to-vrf term default then accept
set routing-instances provider-1 instance-type vrf
set routing-instances provider-1 route-distinguisher 64512:101
set routing-instances provider-1 vrf-import provider-1-import
set routing-instances provider-1 vrf-export provider-1-export
```

