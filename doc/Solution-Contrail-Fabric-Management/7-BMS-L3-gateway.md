* [TOC](Contrail-Fabric-Management.md)

# 7 BMS L3 gateway

Create two virtual networks.
![Figure 7.1 Virtual network](F7-1.png)

Launch a BMS instance on each virtual network.
![Figure 7.2 BMS instance](F7-2.png)

VXLAN routing has to be enabled for the tenant/project.
![Figure 7.3 Enable VXLAN routing](F7-3.png)


## 7.1 L3 gateway on spine

L3 gateway is assigned on spine during importing fabric.
![Figure 7.4 L3 gateway on spine](F7-4.png)

Create logical router and connect virtual networks onto it.
![Figure 7.5 Create logical router](F7-5.png)

* [Logical router configuration on spine-1](A4-L3-gateway-configuration.md#a41-l3-gateway-on-spine-1)
* [Logical router configuration on spine-2](A4-L3-gateway-configuration.md#a42-l3-gateway-on-spine-2)

This is the workflow of L3 routing
![Figure 7.6 Workflow](F7-6.png)


## 7.2 L3 Gateway on MX

L3 gateway is assigned on MX during importing fabric.
![Figure 7.7 L3 gateway on MX](F7-7.png)

Extend virtual networks to MX.
![Figure 7.8 Extend virtual network to MX](F7-8.png)

* [Virtual network configuration on MX-1](A4-L3-gateway-configuration.md#a43-virtual-network-on-mx-1)
* [Virtual network configuration on MX-2](A4-L3-gateway-configuration.md#a44-virtual-network-on-mx-2)

Create a network policy to connect virtual networks.
![Figure 7.9 Create network policy](F7-9.png)

Attach the network policy to virtual networks.
![Figure 7.10 Attach network policy](F7-10.png)

* [Network policy configuration on MX-1](A4-L3-gateway-configuration.md#a45-network-policy-on-mx-1)
* [Network policy configuration on MX-2](A4-L3-gateway-configuration.md#a46-network-policy-on-mx-2)

Note, BMS can ping IRB address on L3GW. The gateway address on L3GW doesn't respond ICMP request because it's a virtual address.


## 7.3 L3 Gateway on leaf



