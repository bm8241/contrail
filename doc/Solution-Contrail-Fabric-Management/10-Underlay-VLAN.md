* [TOC](Contrail-Fabric-Management.md#toc)

# 10 Underlay VLAN

The existing underlay VLAN can be connected to overlay and mapped to virtual network. For the host on underlay VLAN, networking settings (IP address, gateway, DNS, etc.) are statically configured or supported by some other service on underlay.
![Figure 10.1 Topology](F10-1.png)

Create virtual port group on switch-1.
![Figure 10.2 Virtual port group on switch-1](F10-2.png)

Create virtual port group on switch-2.
![Figure 10.3 Virtual port group on switch-2](F10-3.png)

Create virtual port group on multiple devices to provide multi-homing.
![Figure 10.4 Virtual port group multi-homing](F10-4.png)



