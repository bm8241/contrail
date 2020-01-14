
# 1 VM

## 1.1 VM interface

To launch a VM, a VMI is created for the primary interface for untagged traffic.

[Example of VMI for VM primary interface](#a1-vm-primary-interface)


## 1.2 VM sub-interface

To support tagged traffic from VM, create a VMI for the sub-interface.

### 1.2.1 Contrail client

Create VMI for sub-interface with the followings.
* sub_interface_vlan_tag: 10
* reference to the primary VMI
* reference from the privmary VMI to the sub-VMI
* the same MAC address as the primary VMI
* reference to the virtual network
* allocate IP address from the vritual network

[Example of VMI for VM sub-interface](#a2-vm-sub-interface)


### 1.2.2 OpenStack client

#### 1. Create a trunk with the primary interface.
```
openstack network trunk create \
        --parent-port 69c5a8b4-632d-43a1-ade5-e00d7032f2c6 \
        vm-1-trunk
```
Contrail will create a VPG and reference primary VMI to VPG.
Note, VPG is supposed to be used for interface aggregation. Using VPG to represent the trunk may cause problems.

#### 2. Create the sub-interface.
The MAC adress of sub-interface has to be the same as primary interface.
```
openstack port create \
        --network red \
        --mac-address 02:69:c5:a8:b4:63 \
        vm-1-10
```
Contrail will create a VMI and allocate IP address.

#### 3. Add the sub-interface into the trunk.
```
openstack network trunk set \
        --subport port=dbf49a02-e07d-410d-8a51-90a7089d2d2d,segmentation-type=vlan,segmentation-id=10 \
        a3b5e49f-79e5-4ff9-a31a-1f3dfec4c6de
```
Contrail will set sub_interface_vlan_tag in VMI, add a reference to the VPG and a reference to primary VMI.

#### 4. Remove the sub-interface from the trunk.
```
openstack network trunk unset \
        --subport dbf49a02-e07d-410d-8a51-90a7089d2d2d \
        a3b5e49f-79e5-4ff9-a31a-1f3dfec4c6de
```
Contrail will remove the reference to VPG and the reference primary VMI, but not reset sub_interface_vlan_tag.

#### 5. Remove the trunk
```
openstack network trunk delete \
        a3b5e49f-79e5-4ff9-a31a-1f3dfec4c6de
```
Contrail will remove the reference to VPG and delete VPG.


# 2 BMS

## 2.1 BMS interface

### 2.1.1 Contrail client

To connect BMS, create a VMI for the primary interface for untagged traffic.

The following settings are required.
* Device and port that the BMS connects to
* Native VLAN ID for device to map untagged traffic to a VLAN
* Virtual network (VNI) for mapping traffic to overlay
* IP address allocated from the virtual network and MAC address for Contrail to provide DHCP service (This is optional. In case of static networking configuration on BMS, this is not required.)

Here are 3 options.
#### 1. Implicit VPG
For non-aggregated interface, VPG is not necessary to be explicitly created by user. In that case, Contrail will create an implicit VPG and set reference to physical-interface based on the info in key-value-pair. When VMI is deleted, that implicit VPG will be deleted as well.

[Example of VMI for BMS primary interface and the implicit VPG created by Contrail](#A31-implicit-vpg)

#### 2. Explicit VPG without reference
For consistency, it's recommended for user to explicitly create VPG. Nothing needs to be explicitly configured in VPG. Contrail will update VPG with reference to VMI and PI, based on the info in key-value-pair.

[Example of VMI for BMS primary interface and VPG without reference](#a32-explicit-vpg-without-reference)

#### Explicit VPG with reference
Create VPG with reference to PI. Create VMI and reference to VPG. VPG, device and port info is not required in key-value-pair in VMI.

[Example of VMI for BMS primary interface and VPG with reference](#a33-explicit-vpg-with-reference)

Note, with this option, VMI and VPG can't be deleted, because of the relationship between VMI and VPG is reversed. VPG references to VMI, so VMI can't be deleted while it's referenced by VPG. A specific check in API server doesn't allow VPG to be deleted while it references to VMI. This is a dead lock.



### 2.1.2 OpenStack client


## 2.2 BMS sub-interface

### 2.2.1 Contrail client

To support tagged traffic from BMS, create a VMI for the sub-interface.

Settings are the same as primary interface except for VLAN ID instead of native VLAN ID.

For BMS, unlike VM, the primary interface doesn't have to exist when adding sub-interface. There are also 3 options as the BMS primary interface.

[Example of VMI for BMS sub-interface and VPG without reference](#a41-explicit-vpg-without-reference)

To add a sub-interface to existing VPG, device and port still need to be specified in key-value-pair. Without that info, Contrail won't link VMI to VPG.

[Example of adding a sub-interface](#a42-add-sub-interface)


### 2.2.2 OpenStack client


## 2.3 BMS multi-homing

### 2.3.1 Contrail client

When multiple device ports are specified in key-value-pair in VMI, Contrail will add reference to multiple PIs in VPG. As the result, aggregated interface is configured on devices for those ports.

When adding primary interface or sub-interface to existing VPG, the same group of device ports have to be specified in key-value-pair.

[Example of BMS multi-homing](#a5-bms-multi-homing)


### 2.3.2 OpenStack client


# 3 SRIOV VM

## 3.1 SRIOV VM interface on access VF

## 3.2 SRIOV VM interface on trunk VF

## 3.3 SRIOV VM sub-interface

## 3.4 SRIOV VM multi-homing


# Appendix

## A.1 VM primary interface
```
- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - vm-1-while
  resource:
    display_name: vm-1-while
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vnic_type
        value: normal
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:69:c5:a8:b4:63
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
      attr: null

- resource_type: instance-ip
  parent_type: config-root
  resource_fq_name:
  - vm-1-white
  resource:
    display_name: vm-1-white
    instance_ip_family: v4
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - vm-1-white
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null
    subnet_uuid: b4103f8a-7033-4ec2-bcb8-740fb124acba
```


## A.2 VM sub-interface
```
- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - vm-1-red
  resource:
    display_name: vm-1-red
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vnic_type
        value: normal
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:69:c5:a8:b4:63
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 10
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - 69c5a8b4-632d-43a1-ade5-e00d7032f2c6
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
      attr: null

- resource_type: reference
  operation: ADD
  resource:
    src_type: virtual-machine-interface
    src_fqn:
      - default-domain
      - admin
      - vm-1-while
    dst:
    - dst_type: virtual-machine-interface
      dst_fqn:
        - default-domain
        - admin
        - vm-1-red
      attr: null

- resource_type: instance-ip
  parent_type: config-root
  resource_fq_name:
  - vm-1-red
  resource:
    display_name: vm-1-red
    instance_ip_family: v4
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - vm-1-red
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
      attr: null
    subnet_uuid: 0e41516a-c423-45d3-ae49-96ffe96ce052
```


## A.3 BMS primary interface

### A.3.1 Implicit VPG
```
- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - bms-11-white
  resource:
    display_name: bms-11-white
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: profile
        value: '{"local_link_information":[{"port_id":"xe-0/0/6","switch_id":"xe-0/0/6","switch_info":"vqfx-leaf-1","fabric":"poc"}]}'
      - key: vnic_type
        value: baremetal
      - key: tor_port_vlan_id
        value: '4000'
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:1d:19:18:2f:5e
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 0
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null

- resource_type: instance-ip
  parent_type: config-root
  resource_fq_name:
  - bms-11-white
  resource:
    display_name: bms-11-white
    instance_ip_family: v4
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - bms-11-white
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null
    subnet_uuid: b4103f8a-7033-4ec2-bcb8-740fb124acba
```
```
- resource_type: virtual-port-group
  parent_type: fabric
  resource_fq_name:
  - default-global-system-config
  - poc
  - vpg-internal-0
  resource:
    display_name: vpg-internal-0
    fq_name:
    - default-global-system-config
    - poc
    - vpg-internal-0
    parent_type: fabric
    parent_uuid: f23586a2-76ef-4a50-bfc4-772da8f17b94
    physical_interface_refs:
    - attr:
        ae_num: null
      to:
      - default-global-system-config
      - vqfx-leaf-1
      - xe-0/0/6
      uuid: cb9fd146-3f98-4568-b022-6627ac3ebc9b
    uuid: dbe9b81b-1ff0-4539-b37c-7a8a547e00be
    virtual_machine_interface_refs:
    - attr: null
      to:
      - default-domain
      - admin
      - bms-11-white
      uuid: d9fa2445-8c21-4b25-9d04-9dc52ae90eaa
    virtual_port_group_lacp_enabled: true
    virtual_port_group_user_created: false
```

### A.3.2 Explicit VPG without reference
```
- resource_type: virtual-port-group
  parent_type: fabric
  resource_fq_name:
  - default-global-system-config
  - poc
  - bms-11
  resource:
    display_name: bms-11

- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - bms-11-white
  resource:
    display_name: bms-11-white
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vpg
        value: bms-11
      - key: profile
        value: '{"local_link_information":[{"port_id":"xe-0/0/6","switch_id":"xe-0/0/6","switch_info":"vqfx-leaf-1","fabric":"poc"}]}'
      - key: vnic_type
        value: baremetal
      - key: tor_port_vlan_id
        value: '4000'
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:1d:19:18:2f:5e
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 0
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null

- resource_type: instance-ip
  parent_type: config-root
  resource_fq_name:
  - bms-11-white
  resource:
    display_name: bms-11-white
    instance_ip_family: v4
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - bms-11-white
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null
    subnet_uuid: b4103f8a-7033-4ec2-bcb8-740fb124acba
```

### A.3.3 Explicit VPG with reference
```
- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - bms-11-white
  resource:
    display_name: bms-11-white
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vnic_type
        value: baremetal
      - key: tor_port_vlan_id
        value: '4000'
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:1d:19:18:2f:5e
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 0
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null

- resource_type: virtual-port-group
  parent_type: fabric
  resource_fq_name:
  - default-global-system-config
  - poc
  - bms-11
  resource:
    display_name: bms-11
    physical_interface_refs:
    - to:
      - default-global-system-config
      - vqfx-leaf-1
      - xe-0/0/6
      attr:
        ae_num: null
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - bms-11-white
      attr: null

- resource_type: instance-ip
  parent_type: config-root
  resource_fq_name:
  - bms-11-white
  resource:
    display_name: bms-11-white
    instance_ip_family: v4
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - bms-11-white
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null
    subnet_uuid: b4103f8a-7033-4ec2-bcb8-740fb124acba
```


## A.4 BMS sub-interface

### A.4.1 Explicit VPG without reference
```
- resource_type: virtual-port-group
  parent_type: fabric
  resource_fq_name:
  - default-global-system-config
  - poc
  - bms-11
  resource:
    display_name: bms-11

- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - bms-11-red
  resource:
    display_name: bms-11-red
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vpg
        value: bms-11
      - key: profile
        value: '{"local_link_information":[{"port_id":"xe-0/0/6","switch_id":"xe-0/0/6","switch_info":"vqfx-leaf-1","fabric":"poc"}]}'
      - key: vnic_type
        value: baremetal
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:1d:19:18:2f:5e
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 10
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
      attr: null

- resource_type: instance-ip
  parent_type: config-root
  resource_fq_name:
  - bms-11-red
  resource:
    display_name: bms-11-red
    instance_ip_family: v4
    virtual_machine_interface_refs:
    - to:
      - default-domain
      - admin
      - bms-11-red
      attr: null
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
      attr: null
    subnet_uuid: 0e41516a-c423-45d3-ae49-96ffe96ce052
```

### A.4.2 Add sub-interface
```
- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - bms-11-red
  resource:
    display_name: bms-11-red
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vpg
        value: bms-11
      - key: profile
        value: '{"local_link_information":[{"port_id":"xe-0/0/6","switch_id":"xe-0/0/6","switch_info":"vqfx-leaf-1","fabric":"poc"}]}'
      - key: vnic_type
        value: baremetal
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:1d:19:18:2f:5e
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 10
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
      attr: null
```


## A.5 BMS multi-homing
```
- resource_type: virtual-port-group
  parent_type: fabric
  resource_fq_name:
  - default-global-system-config
  - poc
  - bms-11
  resource:
    display_name: bms-11

- resource_type: virtual-machine-interface
  parent_type: project
  resource_fq_name:
  - default-domain
  - admin
  - bms-11-white
  resource:
    display_name: bms-11-white
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vpg
        value: bms-11
      - key: profile
        value: '{"local_link_information":[{"port_id":"xe-0/0/6","switch_id":"xe-0/0/6","switch_info":"vqfx-leaf-1","fabric":"poc"}, {"port_id":"xe-0/0/6","switch_id":"xe-0/0/6","switch_info":"vqfx-leaf-2","fabric":"poc"}]}'
      - key: vnic_type
        value: baremetal
      - key: tor_port_vlan_id
        value: '4000'
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 02:1d:19:18:2f:5e
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 0
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - white
      attr: null
```

