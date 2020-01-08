* [TOC](Contrail-Fabric-Management.md)

# 2 Import fabric

Import existing fabric where underlay is already configured.
![Figure 2.1 Existing fabric](F2-1.png)

Other than subnet, "Management subnets" can also be a list of individual devices with `/32` address.
![Figure 2.2 Create fabric](F2-2.png)

Given the "Management subnets", devices are discovered.
![Figure 2.3 Device discovery](F2-3.png)

Each device is assigned a node profile during discovery. User needs to assign role to each device. This is the case where GW is on spine.
![Figure 2.4 Spine is GW](F2-4.png)

This is the case where GW is on MX.
![Figure 2.5 MX is GW](F2-5.png)

Given role assignment, configuration is pushed to each device.
![Figure 2.6 Auto configure](F2-6.png)

The fabric is created.
![Figure 2.7 Imported fabric](F2-7.png)

Update leaf to associate with CSN.
![Figure 2.8 Associate leaf with CSN](F2-8.png)

