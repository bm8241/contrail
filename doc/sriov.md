
# 1. Introduction

There are two targets.
1. Improve some implementations to be consistent with others.
2. Support new features.

The following cases are covered.
* BMS interface for untagged traffic
* BMS sub-interface for tagged traffic
* BMS multi-homing
* SRIOV VM interface on trunk VF for untagged traffic
* SRIOV VM sub-interface on trunk VF for tagged traffic
* SRIOV VM interface on access VF for untagged traffic
* SRIOV VM multi-homing
* VLAN interface
* VLAN multi-homing

Both OpenStack and Contrail interfaces are supported to do above cases.


# 2. Problem statement

## 2.1 VMI

Here is an example of VMI for BMS untagged interface.
```
- resource_type: virtual-machine-interface
  resource_fq_name:
  - default-domain
  - admin
  - bms-22-1-untagged
  parent_type: project
  resource:
    display_name: bms-22-1-untagged
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vnic_type
        value: baremetal
      - key: vif_type
        value: vrouter
      - key: profile
        value: "{\"local_link_information\":[{\"port_id\":\"xe-0/0/2\",\"switch_id\":\"xe-0/0/2\",\"switch_info\":\"vqfx-leaf-2\",\"fabric\":\"poc\"}]}"
      - key: tor_port_vlan_id
        value: "1"
      - key: vpg
        value: bms-22
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 52:54:00:03:28:f0
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 0
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
```
1. The reference from VMI to VPG is represented by key-value-pair. In Contrail, the relationship/link between resources is specified by reference.

2. The relationship between VMI and VPG is reversed. It should be VMI referencing to VGP, not another way around.

3. Because of #2, there is a specific check to ensure VPG is not deleted when there is referenced from VMI. In general, a resource can't be deleted if it's referenced by any other resources. If the relationship is correct, that generic rule will ensure VPG not being deleted if it's referenced by any VMI. Specific check is not required.

4. When VMI is deleted, VPG still has reference to it.

5. The physical-interface is specified by key-value-pair. It should be a reference.

6. Physical-router is specified by key-value-pair. Once physical-interface is referenenced, the parent of physical-interface is physical-router. No need to specify it.

7. Fabric is specified by key-value-pair. Once physical-router is discovered, there is reference leading to fabric. No need to specify it.

8. `tor_port_vlan_id` is specified by key-value-pair. It should be `native_vlan_id` to be consistent with Junos configuration. It should be added into schema as a new property.

9. VPG is used to represent trunk in OpenStack. One resource can't be used for different purpose. VPG is for interface aggregation, not interface group for trunk. This will cause problems.


## 2.2 Feature request

1. Support to do all use cases by OpenStack interface.

2. SRIOV automation, automatically discover device and port by VF.


# 3. Proposed solution

The requests from Contrail client create VMI and VPG with proper relationship, no key-value-pair is required. See "3.3.1 Contrail client" for details.

The request from OpenStack client creates port with key-value-pair `binding:profile` to specify all required info. API server or schema transformer translates `binding:profile` to resource (eg. VPG) and relationship (eg. to physical-interface). See "3.3.2 OpenStack client" for details.

In case of SRIOV automation, Contrail will need to discover the device and port by VF provided by OpenStack.


## 3.1 Alternatives considered

None


## 3.2 API schema changes

The reference between VMI and VPG needs to be fixed. VMI should reference to VPG, not another way around. This will eliminate the specific check on VPG. It will also fix the reference issue.

A new VMI property native_vlan_id will be added.


## 3.3 User workflow impact

### 3.3.1 Contrail client

#### Untagged interface
* Create VPG, reference to single PI for non-aggregated interface or multiple PIs for aggregated interface.
* Create VMI, set MAC if it's required, set native_vlan_id to map untagged traffic to VLAN, reference to virtual network, reference to VPG. Once DM gets this VMI update, it will configure PI(s).
* Optionally, to provide DHCP service, create IIP and reference to VMI.

#### Tagged interface
The same as untagged interface, except for setting sub_interface_vlan_tag instead of native_vlan_id.


### 3.3.2 OpenStack client

#### 3.3.2.1 BMS
In case of BMS, user will need to create a port on a virtual network and provide required info in `profile` key-value-pair. Contrail will translate those info to resource and relationship.

Here are examples of POST body to create a port to Neutron API.

#### Untagged interface
```
{
    "port": {
        "name": "test-port",
        "network_id": "b960dafe-b6a7-49a3-bd2d-f1fa0f29fef9",
        "binding:profile": {
            "device_port_group": {
                "name": "bms-11",
                "list": [
                    {"device_name": "qfx-5110-1", "port_name": "xe-0/0/2"}
                ]
            },
            "native_vlan_id": "4095"
        },
        "admin_state_up": true
    }
}
```

#### Tagged interface
```
{
    "port": {
        "name": "test-port",
        "network_id": "b960dafe-b6a7-49a3-bd2d-f1fa0f29fef9",
        "binding:profile": {
            "device_port_group": {
                "name": "bms-11",
                "list": [
                    {"device_name": "qfx-5110-1", "port_name": "xe-0/0/2"}
                ]
            },
            "vlan_id": "10",
        },
        "admin_state_up": true
    }
}
```

#### Multi-homing
```
{
    "port": {
        "name": "test-port",
        "network_id": "b960dafe-b6a7-49a3-bd2d-f1fa0f29fef9",
        "binding:profile": {
            "device_port_group": {
                "name": "bms-11",
                "list": [
                    {"device_name": "qfx-5110-1", "port_name": "xe-0/0/2"},
                    {"device_name": "qfx-5110-2", "port_name": "xe-0/0/2"}
                ]
            },
            "vlan_id": "10",
        },
        "admin_state_up": true
    }
}
```


#### 3.3.2.2 SRIOV
In case of SRIOV, the process is automated. The port will be updated by nova-compute with compute node, VF and VLAN info. Here is an example.
```
{
    "key": "profile",
    "value": "{\"pci_slot\": \"0000:83:12.7\", \"physical_network\": \"sriov\", \"pci_vendor_info\": \"8086:10ed\"}"},
{
    "key": "host_id",
    "value": "dc-poclab-compute2.poc-nl.jnpr.net"},
{
    "key": "vnic_type",
    "value": "direct"},
{
    "key": "vif_details",
    "value": "{\"port_filter\": true, \"vlan\": \"7\"}"},
{
    "key": "vif_type",
    "value": "hw_veb"}
```

Note, need to confirm nova-compute doesn't override user settings in `binding:profile`.

#### Access VF
* Create a dummy access virtual network with VLAN ID that will be assigned to VF, if it doesn't exist.
* Create a port on the dummy virtual network with the actual virtual network and device port group name in `binding:profile`.
* Launch SRIOV VM on the port. Nova-compute will update `binding:profile` with VF and VF VLAN ID.
* Contrail will discover the device and port by VF, and configure the port with the actual virtual network and VLAN ID.

#### Untagged interface on trunk VF
* Create a dummy trunk virtual network with VLAN ID 0, if it doesn't exist.
* Create a port on trunk virtual network with the actual virtual network, native VLAN ID and device port group name in `binding:profile`.
* Launch SRIOV VM on the port. Nova-compute will update `binding:profile` with VF and VF VLAN ID.
* Contrail will discover the device and port by VF, and configure the port with the actual virtual network and native VLAN ID.

#### Tagged interface on trunk VF
* Create a port on a virtual network with copied `binding:profile` from the primary interface and the specific VLAN ID.

#### Multi-homing
* Create one dummy virtual network for each SRIOV NIC.
* Create multiple ports, one on each dummy virtual network. All ports have to specify the same device port group name.
* Launch SRIOV VM on multiple ports.
* Contrail will take care of the rest.


## 3.4 UI changes


## 3.5 Notification impact

None.


# 4. Implementation

## 4.1 Relationship
```
+-----+   +-----+   +----+
| VMI |-->|     |-->| PI |
+-----+   | VPG |   +----+
+-----+   |     |   +----+
| VMI |-->|     |-->| PI |
+-----+   +-----+   +----+
```
VMI reference to VPG. VPG reference to PI (physical-interface).

VPG can't be deleted when it's referenced by any VMI. This will remove that specific check.


## 4.2 Workflow

Device manager listens to the update of VMI and VPG.

* First of all, VPG has to be created.
* When VPG add or delete reference to PI(s), DM checks VMI back-reference in VPG. If there is VMI referencing to VPG, DM will update configuration on the PI(s). If there is no VMI referencing to VPG, DM does nothing.
* To create an interface for untagged traffic (BMS primary interface, SRIOV VM primary interface on trunk VF), create VMI and add reference to VPG. When DM gets this VMI update, it will update configuration on PI(s).
* Only one VMI is allowed to have native VLAN ID per VPG. This specific check can be done by API server.
* A new property native_vlan_id will be added in VMI.
* To create an interface for tagged traffic (BMS sub-interface, SRIOV VM primary interface on access VF, SRIOV VM sub-interface on trunk VF), create VMI and add reference to VPG. When DM gets this VMI update, it will update configuration on PI(s).
* Multiple VMIs with VLAN ID can be added. VLAN ID could be duplicated, not necessarily unique. For new VMI with duplicated VLAN ID, DM does nothing. For new VMI with new VLAN ID, DM will update configuration on PI(s).
* When VPG adds reference to more PI, DM will gets the VPG update and configure additional PI for interface aggregation.
* When VPG removes reference to single PI, DM will update PI from aggregated interface to non-aggregated interface.


## 4.3 OpenStack support

Contrail client is able to manage all resource. For OpenStack client, all required info is carried by `bind:profile`. Contrail API server or schema transformer will need to translate those info to proper resource and relationship. For example, create VPG and reference to PI.


## 4.4 Discover device and port

#### VF to PF

For SRIOV VM, nova-compute updates port `binding:profile` with VF info. There are two options to find PF by VF.
1. Connect to the compute node, go through all PFs to find out which PF the VF belongs to.
2. User provides a mapping between VF and PF after compute node deployment.

#### PF to device port
Once get PF, there are two options to get device and port that PF connects to.
1. Enable LLDP on compute node. Connect to compute node to get device and port by LLDP neighbor info.
2. User provides a topology mapping between PF on compute node and device port.


# 5. Performance and scaling impact

N/A


# 6. Upgrade

N/A


# 7. Deprecations

N/A


# 8. Dependencies

N/A


# 9. Testing


# 10. Documentation Impact


# 11. References

