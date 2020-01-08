* [TOC](SRIOV.md#toc)

# 4 VM interface on trunk VF

* A VF is allocated to be a VM interface.
* A MAC address and VLAN ID 0 is set on VF.
* Both tagged and untagged traffic are accepted from VM.
* Traffic from VM is untouched when going through VF.
* VLAN interface is supported inside VM.


## 4.1 Workflow to launch VM

Here are the steps to launch a VM with trunk VF on a tenant VN.

#### 1. Create a trunk VN with VLAN ID 0.
A separate VN with VLAN ID 0 is required. The subnet is some dummy address space. It can be used for all trunk port.
1. SRIOV physical network configured in nova.conf for Nova compute
2. VLAN ID 0 that will be assigned to VF

This step can be done either by OpenStack CLI or via Contrail API.

Here is an example of OpenStack CLI.
```
openstack network set \
        --provider-physical-network sriov \
        --provider-segment 0
        sriov-0
```

#### 2. Create a VM port on the trunk VN with VNIC type `direct`.
A dummy IP address will be allocated.

This step can be done either by OpenStack CLI or via Contrail API.

Here is an example of OpenStack CLI.
```
openstackport create \
        --network sriov0 \
        --vnic-type direct \
        $port_name
```

#### 3. Launch a VM on the port.
Nova compute will allocate VF from physical network (PF), assign VLAN ID 0 and set MAC address on VF.

This step has to be done by OpenStack. Here is an example.
```
openstack server create \
        --image $image \
        --flavor $flavor \
        --port $port_id \
        $vm_name"
```

#### 4. Find out device and port.
Run a script on the compute node where VM is launched to find out the device and port that PF connects to.

Given the port MAC address, which is set on VF, the script finds out PF. Device and port can retrieved from LLDP neighbor of PF.

#### 5-1. Untagged traffic from VM.
Create a VM port on tenant VN to configure device. This port contains device info for Contrail to configure device. It also has native-vlan-id for untagged traffic. Only one trunk port is allowed on each PF.

#### 5-2. Tagged traffic from VM.
Create a VM port on tenant VN to configure device. This port contains device info for Contrail to configure device.

This step has to be done via Contrail API, because some property settings are not supported by OpenStack CLI.

#### 6. Configure device.
Once the port is created, device manager service in Contrail will start a workflow to configure device.

* [Configuration example](A2-Untagged.md#a2-untagged)


## 4.2 libvirt

An interface of `hostdev` type is created for VM.
```
    <interface type='hostdev' managed='yes'>
      <mac address='02:4a:eb:b0:e0:08'/>
      <driver name='vfio'/>
      <source>
        <address type='pci' domain='0x0000' bus='0x83' slot='0x13' function='0x3'/>
      </source>
      <vlan>
        <tag id='0'/>
      </vlan>
      <alias name='hostdev0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </interface>
```

It sets MAC and no VLAN ID on VF `83:13.3`.
```
# lspci | grep "83:13.3"
83:13.3 Ethernet controller: Intel Corporation 82599 Ethernet Controller Virtual Function (rev 01)

# ip link | grep "02:4a:eb:b0:e0:08"
    vf 13 MAC 02:4a:eb:b0:e0:08, spoof checking on, link-state auto, trust on, query_rss off
```


