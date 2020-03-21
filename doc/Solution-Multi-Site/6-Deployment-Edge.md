* [TOC](Multi-Site.md#toc)

# 6 Deployment service provider edge

* All sites are owned by service provider.
* The gateway in each site is as both CE and PE.
* Address space for each site is pre-allocated without overlapping.
* This deployment is in a virtual environemnt with vQFX (19.4R1.10) as leaf and spine, and vMX (19.4R1.10) as gateway.


## 6.1 Underlay

![Figure 6.1 Underlay](F6-1.png)

* In each site, every leaf and gateway connects (eBGP peer) to all spines to form the CLOS fabric underlay. For small site without spine, every leaf connects to all gateways. Each device has an unique underlay ASN.

* In the service provider core, IGP (OSPF in this case) is used to connect all routers to provider underlay connectivities between site gateways.

* Based on the underlay connectivity in the core, iBGP peering is established on all gateways via route reflectors in the core. Connectivity between sites is provided by this iBGP.

Eventually, this underlay provides connectivity between the controller on primary site and compute on other sites.


## 6.2 Overlay

![Figure 6.2 Overlay](F6-2.png)

* Seperated control nodes, one for each site. Control node in each site peers with local gateway. Compute in each site connects to local control node.



