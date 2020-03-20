* [TOC](Multi-Site.md#toc)

# 5 Deployment example 1

* For service provider, each site gateway is both CE and PE.
* Address space for each site is pre-allocated without overlapping.
* No EVPN-VXLAN based multi-homing to avoid troubles may caused by VXLANoVXLAN for VM on vrouter. MC-LAG is an option.
* vQFX (19.4R1.10) is as leaf and spine, and vMX (18.3R1.9) is as gateway.


## 5.1 Underlay

![Figure 5.1 Underlay](F5-1.png)

* In each site, every leaf and gateway connects (eBGP peer) to all spines to form the CLOS fabric underlay. For small site without spine, every leaf connects to all gateways. Each device has an unique underlay ASN.

* In the service provider core, IGP (OSPF in this case) is used to connect all routers to provider underlay connectivities between site gateways.

* Based on the underlay connectivity in the core, iBGP peering is established on all gateways via route reflectors in the core. Connectivity between sites is provided by this iBGP.

Eventually, this underlay provides connectivity between the controller on primary site and compute on other sites.


## 5.2 Overlay

![Figure 5.2 Overlay](F5-2.png)

* Seperated control nodes, one for each site. Control node in each site peers with local gateway. Compute in each site connects to local control node.


### 5.2.1 Underlay connectivity

Workload (VM or BMS) on overlay of one site connects to workload on underlay of another site, eg. physical servers on legacy site.


### 5.2.2 Overlay connectivity

Workload (VM or BMS) on overlay of one site connects to workload on overlay of another site, eg. remote compute.



