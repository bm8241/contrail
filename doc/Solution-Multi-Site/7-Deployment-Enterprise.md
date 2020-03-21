* [TOC](Multi-Site.md#toc)

# 7 Deployment enterprise multi-site

* All sites are connected by L3VPN provided by service provider.
* The gateway of eash site is CE, connets to PE.
* This deployment is in a virtual environemnt with vQFX (19.4R1.10) as leaf and spine, and vMX (19.4R1.10) as gateway.


## 7.1 Underlay

![Figure 7.1 Underlay](F7-1.png)

* In each site, every leaf and gateway connects (eBGP peer) to all spines to form the CLOS fabric underlay. For small site without spine, every leaf connects to all gateways. Each device has an unique underlay ASN.

* Based on the L3VPN connectivity provided by service provider, iBGP peering is established on all gateways.


## 7.2 Overlay

![Figure 7.2 Overlay](F7-2.png)

* Seperated control nodes, one for each site. Control node in each site peers with local gateway. Compute in each site connects to local control node.

