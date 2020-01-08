* [TOC](SRIOV.md#toc)

# 3 VM interface on access VF

* A VF is allocated to be a VM interface.
* A MAC address and a VLAN ID is set on VF.
* Only untagged traffic is accepted from VM.
* Traffic from VM is tagged with VLAN ID by VF.
* VLAN interface is not supported inside VM.


## 3.1 Workflow to launch VM

Here are the steps to launch a VM with access VF on a tenant VN.

#### 1. Update tenant VN with two properties.
1. SRIOV physical network configured in nova.conf for Nova compute
2. VLAN ID that will be assigned to VF
To simplify the workflow, the VN ID assigned to tenant VN is used as VLAN ID.

This step can be done either by OpenStack CLI or via Contrail API.

Here is an example of OpenStack CLI.
```
openstack network set \
        --provider-physical-network sriov \
        --provider-segment $vlan_id
        $network_name
```

#### 2. Create a VM port on tenant VN with VNIC type `direct`.
An IP address will be allocated. Even with static address in VM, that address should be allocated and assigned to the port, to avoid conflict with other ports.

This step can be done either by OpenStack CLI or via Contrail API.

Here is an example of OpenStack CLI.
```
openstackport create \
        --network $network_name \
        --vnic-type direct \
        $port_name
```

#### 3. Launch a VM on the port.
Nova compute will allocate VF from physical network (PF), assign VLAN ID and set MAC address on VF.

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

#### 5. Create a VM port on tenant VN to configure device.
This port contains device info for Contrail to configure device. No need to allocate IP address for this port.

This step has to be done via Contrail API, because some property settings are not supported by OpenStack CLI.

#### 6. Configure device.
Once the port is created, device manager service in Contrail will start a workflow to configure device.

* [Configuration example](A1-Tagged.md#a1-tagged)

## 3.2 libvirt

An interface of `hostdev` type is created for VM.
```
    <interface type='hostdev' managed='yes'>
      <mac address='02:5a:0d:c4:39:77'/>
      <driver name='vfio'/>
      <source>
        <address type='pci' domain='0x0000' bus='0x83' slot='0x13' function='0x3'/>
      </source>
      <vlan>
        <tag id='15'/>
      </vlan>
      <alias name='hostdev0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </interface>
```

It sets MAC and VLAN ID on VF `83:13.3`.
```
# lspci | grep "83:13.3"
83:13.3 Ethernet controller: Intel Corporation 82599 Ethernet Controller Virtual Function (rev 01)

# ip link | grep "02:5a:0d:c4:39:77"
    vf 13 MAC 02:5a:0d:c4:39:77, vlan 15, spoof checking on, link-state auto, trust on, query_rss off
```


