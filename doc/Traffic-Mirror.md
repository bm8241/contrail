* 1 Overview
* 2 Netowrk Mirror
  * 2.1 Address Based Anayzer
  * 2.2 Service Instance Based Analyzer
  * 2.3 Packet Capture on Web UI
  * 2.4 Internals
* 3 Port Mirror
  * 3.1 Dynamic NH
    * 3.1.1 With Juniper Header
    * 3.1.2 Without Juniper Header
  * 3.2 Static NH


## 1 Overview

Contrail supports to mirror/duplicate traffic and send it to an analyzer. Traffic can be mirrored on either network (network mirror) or interface (port mirror).

ACL is supported by network mirror to duplicate specified traffic by 5-tuple.

When traffic is mirrored, a Juniper header is added for steering mirrored traffic to the analyzer on L3. Given analyzer address or FQ name of analyzer service instance, mirror vrouter will be able to find the NH for mirrored traffic by looking up L3 table in VRF. Mirrored traffic with Juniper header is encapsulated in MPLSoUDP/MPLSoGRE in which the MPLS lable is used to identify/map analyzer interface. Analyzer vrouter takes the L3 packet (Juniper header + mirrored L2 frame), identifies analyzer interface, adds a L2 header (src MAC: vrouter, dst MAC: analyzer), and sends it to the analyzer.
```
+-----------+-----------+---------------+----------+----------+
| MPLSoUDP/ | Analyzer  | UDP Header    | Metadata | Mirrored |
| MPLSoGRE  | IP Header | Analyser Port |          | L2 Frame |
+-----------+-----------+---------------+----------+----------+
             \__________________  ________________/
                                \/
                          Juniper Header
```

For network mirror, Juniper header is always added. For port mirror, there is option to disable Juniper header.
```
+-------------------+---------+------------------------------------------+
|                   |         |                Port Mirror               |
|                   | Network +--------------------+---------------------+
|                   | Mirror  |     Dynamic NH     |      Static NH      |
|                   |         | (Virtual Analyzer) | (Physical Analyzer) |
+-------------------+---------+--------------------+---------------------+
| ACL               |    X    |                    |                     |
+-------------------+---------+--------------------+---------------------+
| Juniper Header    |    X    |         X          |                     |
+-------------------+---------+--------------------+---------------------+
| No Juniper Header |         |         X          |          X          |
+-------------------+---------+--------------------+---------------------+
```
With dynamic NH, the NH is found by vrouter by looking up VRF given L2/L3 address or FQ name. This mode is for supporting virtual analyzer who connects to vrouter.

With static NH, the NH is statically configured. This mode is for supporting physical analyzer who connects to VTEP (eg. ToR).


#### Note
As the best practice, analyzer should be on an exclusive virtual network and on a separate vrouter from mirror network or port.

All tests are done with Contrail 3.2.5.0-51.

The issue, that source address is not set properly for network mirror, will be fixed in 4.1.


## 2 Network Mirror

Network mirror is enabled by network mirror policy. ACL can be configured to mirror specific traffic. Mirrored traffic always has Juniper header.


### 2.1 Address Based Analyzer

With this option, analyzer address has to be configured. Network policy is also required to advertise analyzer address to mirror network.

Create a policy with mirror rule with analyzer address. For example, any traffic will be mirrored and analyzer address is 192.168.100.3.
```
config create policy mirror --rule \
    action=mirror,analyzer-name=my-analyzer,analyzer-address=192.168.100.3
```

Attach this policy to the mirror network as dynamic policy.
```
config add network red --policy mirror --dynamic
```

Due to an issue, disable policy on analyzer port. See 2.5 Issues.
```
config add port 465e53c8-4df7-40f9-b81a-dd93bb8825e5 --disable-policy
```

Now, all traffic in and out virtual network "red" is mirrored to analyzer.


### 2.2 Service Instance Based Analyzer

With this option, service instance (v2, port tuple based) of analyzer is configured. Service instance in the policy will connect mirror network and analyzer network. No network policy is required to advertise analyzer address to mirror network.

Create a service template for transparent analyzer service with left interface only, if it doesn't exist.
```
config create st analyzer-v2 --mode transparent --type analyzer \
    --interface type=left
```

Create a service instance and map analyzer interface to the left interface.
```
config create si analyzer-si --template analyzer-v2 \
    --network left=default-domain:demo:management \
    --vm 0c1e36ea-4cec-4f27-a0c8-14d6123b6085
```
VM ID is for finding interface and building port tuple. Nova CLI can be used to get VM ID by VM name.
```
nova --os-tenant-name demo list | awk "/analyzer/"'{print $2}'
```

Create a policy with a mirror rule with service instance FQ name.
```
config create policy mirror --rule \
    action=mirror,analyzer-name=default-domain:demo:analyzer-si
```

Attach this policy to the target network as dynamic policy.
```
config add network red --policy mirror --dynamic
```

Due to an issue, disable policy on analyzer port. See 2.5 Issues.
```
config add port 465e53c8-4df7-40f9-b81a-dd93bb8825e5 --disable-policy
```

Now, all traffic in and out virtual network "red" is mirrored to analyzer.


### 2.3 Packet Capture on Web UI

With this option, user creates an analyzer on web UI Monitor -> Debug -> Packet Capture, choosing analyzer network, target network and mirror rule. Then web UI will create service instance (version 1), mirror policy and attach mirror policy to the target network.

The analyzer will be launched by service monitor based on service instance via Nova API. The analyzer is launched with VM image with name "analyzer" and the flavor with name "m1.medium".


### 2.4 Internals

The mirror index is set on the original flow.
```
# flow -l

   335988<=>414252       192.168.20.3:36609                                  1 (2)
                         192.168.10.3:0    
(Gen: 2, K(nh):15, Action:F, Flags:, QOS:-1, S(nh):15,  Stats:43/4214, 
 Mirror Index : 0 SPort 49581, TTL 0, Sinfo 4.0.0.0)

   414252<=>335988       192.168.10.3:36609                                  1 (1)
                         192.168.20.3:0    
(Gen: 71, K(nh):23, Action:F, Flags:, QOS:-1, S(nh):23,  Stats:43/4214, 
 Mirror Index : 0 SPort 60576, TTL 0, Sinfo 3.0.0.0)
```

One mirror entry is for each analyzer.
```
# mirror --get 0

Index    NextHop    Flags    VNI
------------------------------------------------
    0         27       D          0
```

The NH is for building tunnel to the analyzer vrouter.
```
# nh --get 27

Id:27         Type:Tunnel         Fmly: AF_INET  Rid:0  Ref_cnt:2          Vrf:-1
              Flags:Valid, Udp, Copy SIP, 
              Oif:0 Len:14 Flags Valid, Udp, Copy SIP,  Data:00 00 00 00 00 00 00 25 90 c4 82 60 08 00 
              Vrf:-1  Sip:10.84.32.12  Dip:192.168.100.4
              Sport:8097 Dport:8099
```

Vrouter looks up destination/analyzer address in orginal flow VRF.
```
# rt --dump 1 | grep 100.5/32
192.168.100.5/32       32           LP         17             36        -
```

NH index is 36 and MPLS label is 17.
```
# nh --get 36
Id:36         Type:Tunnel         Fmly: AF_INET  Rid:0  Ref_cnt:7          Vrf:0
              Flags:Valid, MPLSoUDP, 
              Oif:0 Len:14 Flags Valid, MPLSoUDP,  Data:00 25 90 c4 82 72 00 25 90 c4 82 60 08 00 
              Vrf:0  Sip:10.84.32.12  Dip:10.84.32.13
```

There is no entry in flow table for mirror flow on the mirror vrouter, because flow is created by vrouter agent only when there is policy (ACL) applies to that flow, and there is no policy applies to mirror traffic on the mirror vrouter. Flow for mirror flow is created on analyzer vrouter because of the policy on analyzer port. If the policy is disabled on analyzer port, then there will be no flow for mirror flow on analyzer vrouter.


#### Issues

For network mirror, there is a bug that the source address of Juniper header is not properly set (The source is the mirror vrouter underlay address.), mirror traffic is dropped by analyzer vrouter due to policy check. The workaround is to disable the policy on analyzer port, or use port mirror. Also the ingress traffic is not mirrored properly.


## 3 Port Mirror

Port mirror is enabled by mirror configuration on the VM interface. No ACL can be configured. All traffic will be mirrored. The only control bit is direction. Port mirror supports to mirror traffic without Juniper header.


### 3.1 Dynamic NH

For virtual analyzer (connecting to vrouter), dynamic NH is used to support both with and without Juniper header.


#### 3.1.1 With Juniper Header

For address based analyzer, enable the mirror on the port and set analyzer address. Analyzer name is mandatory, but it can be any name. Network policy is reuqired to advertise analyzer address to mirror network (where mirror port is).
```
config add port 2a479330-2917-4ebb-b33b-2f495bac3aa6 \
    --mirror name=my-analyzer,address=192.168.100.3
```

For service instance based analyzer, enable mirror on the port and set service instance FQ name as analyzer name. Analyzer address is not required. Don't use mirror network policy, which will apply to all traffic on the network. Use regular network policy to advertise analyzer address to mirror network.
```
config add port 2a479330-2917-4ebb-b33b-2f495bac3aa6 \
    --mirror name=default-domain:demo:analyzer-si
```

Mirror is enabled on the port with flag and mirror index.
```
# vif --list

vif0/3      OS: tap2a479330-29
            Type:Virtual HWaddr:00:00:5e:00:01:00 IPaddr:192.168.10.3
            Vrf:1 Flags:PMrMtL3L2D QOS:-1 Ref:5 Mirror index 0
            RX packets:246070  bytes:10986030 errors:0
            TX packets:436134  bytes:82689844 errors:0
            Ingress Mirror Metadata: 3 17 64 65 66 61 75 6c 74 2d 64 
                                     6f 6d 61 69 6e 3a 64 65 6d 6f 3a 
                                     72 65 64 ff 0 
            Egress Mirror Metadata: 4 17 64 65 66 61 75 6c 74 2d 64 6f 
                                    6d 61 69 6e 3a 64 65 6d 6f 3a 72 
                                    65 64 ff 0 
            Drops:3713
```
Flag 'Mr' means mirror Rx/ingress and flag 'Mt' means mirror Tx/egress. Metadata is virtual network info added to Juniper header.

With either option, mirror vrouter finds the NH based on analyzer address or service instance FQ name.

The rest works the same as network mirror.

The Juniper (UDP) header is built based on mirror entry and mirror metadata on the port. The source is port address. The destination is analytics address.

On the mirror vrouter, in case of address based analyzer, since network policy is required to advertise analyzer address to mirror network, flow entry is created for mirror flow. In case of service instance based analyzer, no policy applies to mirror flow, so no flow entry created for mirror flow.


#### 3.1.2 Without Juniper Header

With dynamic NH for analyzer VM, to support no Juniper header, mirrored L2 frame is encapsulated in MPLSoUDP.
```
+-----+-------+----------+
| UDP | MPLS  | Mirrored |
|     |       | L2 Frame |
+-----+-------+----------+
```

Analyzer MAC, IP address and routing instance are required. Mirror vrouter checks if analyzer RI exists. If yes, vrouter does L2 lookup in analayzer VRF, get the NH (to analyzer) and use it as the NH of mirrored flow. Otherwise, vrouter will create the analyzer RI, then control node will push all info down to vrouter, then vrouter does L2 lookup and get the NH. MPLSoUDP encapsulation is built based on this NH.

When analyzer vrouter receives and de-capsulates MPLSoUDP packet, it expects L3 packet and check the source IP address. Since it's L2 frame, to avoid the check, the policy has to be disabled on analyzer interface. Then based on MPLS label, analyzer vrouter will send the L2 frame to analyzer.

```
config add port 2a479330-2917-4ebb-b33b-2f495bac3aa6 \
    --mirror juniper-header=false,nh-mode=dynamic,network=default-domain:demo:management,mac=4e:70:33:df:96:a8,name=my-analyzer,address=1.2.3.4
```

The policy has to be disabled on the analyzer port.
```
config add port 465e53c8-4df7-40f9-b81a-dd93bb8825e5 --disable-policy
```

Mirror vrouter finds the NH based on analyzer MAC address and routing instance FQ name (the script takes network name and build routing instance name).

```
vif --list

vif0/3      OS: tap2a479330-29
            Type:Virtual HWaddr:00:00:5e:00:01:00 IPaddr:192.168.10.3
            Vrf:1 Flags:PMrMtL3L2D QOS:-1 Ref:5 Mirror index 0
            RX packets:330240  bytes:14870386 errors:0
            TX packets:520830  bytes:86746711 errors:0
            Ingress Mirror Metadata: 3 17 64 65 66 61 75 6c 74 2d 64 
                                     6f 6d 61 69 6e 3a 64 65 6d 6f 3a 
                                     72 65 64 ff 0 
            Egress Mirror Metadata: 4 17 64 65 66 61 75 6c 74 2d 64 6f 
                                    6d 61 69 6e 3a 64 65 6d 6f 3a 72 
                                    65 64 ff 0 
            Drops:3735
```

On mirror vrouter, no entry for mirror flow in flow table. On analyzer vrouter, no entry for mirror flow either, because policy is disabled on analyzer port.


#### Issues

Analyzer IP address is not required to find NH, but there is some validation against it, so a dummy address has to be configured to work around this issue.

Eventually, the NH should be found based on either analyzer IP address or analyzer service instance FQ name, just like how other options work.


### 3.2 Static NH

To support physical analyzer connecting to VTEP (ToR), static NH has to be configured and no Juniper header added. Mirrored L2 frame is encapsulated in VXLAN and is sent to VTEP who will flood mirror traffic within that L2 domain. The mirror flag has to be set for analyzer network to enable such flood. VNI of analyzer network and VTEP address of analyzer are required to configure static NH. VXLAN header is built based on those info.

```
+-------+----------+
| VXLAN | Mirrored |
|       | L2 Frame |
+-------+----------+
```


## Appendix A Object in JSON Format

### Appendix A.1 Without Service Instance

#### Policy "mirror"
```
{
    "fq_name": [
        "default-domain",
        "demo",
        "mirror"
    ],
    "uuid": "efdd9a96-fb12-4c52-bbe8-27583e49aa31",
    "parent_type": "project",
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-09-24T02:55:14.002096",
        "uuid": {
            "uuid_mslong": 17284140918165883986,
            "uuid_lslong": 13540115539645016625
        },
        "user_visible": true,
        "last_modified": "2017-09-24T02:55:14.002096",
        "permissions": {
            "owner": "admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "KeystoneAdmin",
            "group_access": 7
        }
    },
    "network_policy_entries": {
        "policy_rule": [
            {
                "direction": "<>",
                "protocol": "any",
                "dst_addresses": [
                    {
                        "security_group": null,
                        "subnet": null,
                        "virtual_network": "any",
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "action_list": {
                    "gateway_name": null,
                    "log": false,
                    "alert": false,
                    "qos_action": null,
                    "assign_routing_instance": null,
                    "mirror_to": {
                        "analyzer_name": "any-name",
                        "analyzer_mac_address": null,
                        "juniper_header": true,
                        "udp_port": null,
                        "analyzer_ip_address": "192.168.100.3",
                        "static_nh_header": null,
                        "routing_instance": null,
                        "encapsulation": null,
                        "nh_mode": null
                    },
                    "simple_action": null,
                    "apply_service": []
                },
                "created": null,
                "rule_uuid": "c096debb-b05c-4d0a-9d3f-b57d2c54fc29",
                "dst_ports": [
                    {
                        "end_port": -1,
                        "start_port": -1
                    }
                ],
                "application": [],
                "last_modified": null,
                "ethertype": null,
                "src_addresses": [
                    {
                        "security_group": null,
                        "subnet": null,
                        "virtual_network": "any",
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "rule_sequence": null,
                "src_ports": [
                    {
                        "end_port": -1,
                        "start_port": -1
                    }
                ]
            }
        ]
    },
    "display_name": "mirror"
}
```

#### Virtual network "red"
```
{
    "virtual_network_properties": {
        "forwarding_mode": null,
        "allow_transit": false,
        "network_id": null,
        "mirror_destination": false,
        "vxlan_network_identifier": null,
        "rpf": "enable"
    },
    "is_shared": false,
    "ecmp_hashing_include_fields": {
        "destination_ip": true,
        "ip_protocol": true,
        "source_ip": true,
        "hashing_configured": false,
        "source_port": true,
        "destination_port": true
    },
    "fq_name": [
        "default-domain",
        "demo",
        "red"
    ],
    "uuid": "a983108d-5617-4c16-b3c8-8ab956cfe436",
    "display_name": "red",
    "network_policy_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "blue-red"
            ],
            "href": "http://10.87.68.167:8082/network-policy/033cd8ec-4065-4e16-a799-c3fca5f3a1a3",
            "attr": {
                "timer": null,
                "sequence": {
                    "major": 0,
                    "minor": 0
                }
            },
            "uuid": "033cd8ec-4065-4e16-a799-c3fca5f3a1a3"
        },
        {
            "to": [
                "default-domain",
                "demo",
                "mirror"
            ],
            "href": "http://10.87.68.167:8082/network-policy/efdd9a96-fb12-4c52-bbe8-27583e49aa31",
            "attr": {
                "timer": {
                    "start_time": null,
                    "off_interval": null,
                    "on_interval": null,
                    "end_time": null
                },
                "sequence": {
                    "major": 0,
                    "minor": 0
                }
            },
            "uuid": "efdd9a96-fb12-4c52-bbe8-27583e49aa31"
        },
        {
            "to": [
                "default-domain",
                "demo",
                "management"
            ],
            "href": "http://10.87.68.167:8082/network-policy/ce31ffb9-d80e-4d35-ace2-75b3db9c0ad2",
            "attr": {
                "timer": null,
                "sequence": {
                    "major": 1,
                    "minor": 0
                }
            },
            "uuid": "ce31ffb9-d80e-4d35-ace2-75b3db9c0ad2"
        }
    ],
    "router_external": false,
    "network_ipam_refs": [
        {
            "to": [
                "default-domain",
                "default-project",
                "default-network-ipam"
            ],
            "href": "http://10.87.68.167:8082/network-ipam/e8e63c76-b883-4fbd-815d-073669650413",
            "attr": {
                "ipam_subnets": [
                    {
                        "subnet": {
                            "ip_prefix": "192.168.10.0",
                            "ip_prefix_len": 24
                        },
                        "dns_server_address": "192.168.10.2",
                        "enable_dhcp": true,
                        "created": "2017-09-21T07:01:18.161641",
                        "default_gateway": "192.168.10.1",
                        "dns_nameservers": [],
                        "dhcp_option_list": null,
                        "subnet_uuid": "49e3347f-44f5-41ab-8919-d9bbf193e6ad",
                        "alloc_unit": 1,
                        "last_modified": "2017-09-21T07:01:18.161641",
                        "host_routes": null,
                        "addr_from_start": true,
                        "subnet_name": "",
                        "allocation_pools": []
                    }
                ],
                "host_routes": null
            },
            "uuid": "e8e63c76-b883-4fbd-815d-073669650413"
        }
    ],
    "parent_type": "project",
    "import_route_target_list": {
        "route_target": []
    },
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "route_target_list": {
        "route_target": []
    },
    "export_route_target_list": {
        "route_target": []
    },
    "flood_unknown_unicast": false,
    "id_perms": {
        "enable": true,
        "description": "",
        "creator": null,
        "created": "2017-09-21T07:01:17.561096",
        "uuid": {
            "uuid_mslong": 12214624813579717654,
            "uuid_lslong": 12954756856761279542
        },
        "user_visible": true,
        "last_modified": "2017-09-24T02:56:49.123450",
        "permissions": {
            "owner": "neutron",
            "owner_access": 7,
            "other_access": 7,
            "group": "_member_",
            "group_access": 7
        }
    },
    "port_security_enabled": true,
    "multi_policy_service_chains_enabled": false,
    "virtual_network_network_id": 5
}
```

### Appendix A.2 With Service Instance

#### Service template "analyzer-v2"
```
{
    "fq_name": [
        "default-domain",
        "analyzer-v2"
    ],
    "uuid": "0a357fe1-bccf-460a-ae54-eac61b213c1b",
    "parent_type": "domain",
    "perms2": {
        "owner": "cloud-admin",
        "owner_access": 7,
        "global_access": 0,
        "share": [
            {
                "tenant_access": 5,
                "tenant": "domain:5f0eb78f-3aab-4af0-8427-6c88a8e9bb5d"
            }
        ]
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-09-24T04:14:24.400829",
        "uuid": {
            "uuid_mslong": 735634721657013770,
            "uuid_lslong": 12561923397222743067
        },
        "user_visible": true,
        "last_modified": "2017-09-24T04:14:24.400829",
        "permissions": {
            "owner": "admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "KeystoneAdmin",
            "group_access": 7
        }
    },
    "service_template_properties": {
        "instance_data": null,
        "service_mode": "transparent",
        "availability_zone_enable": false,
        "service_virtualization_type": null,
        "interface_type": [
            {
                "static_route_enable": false,
                "shared_ip": false,
                "service_interface_type": "left"
            }
        ],
        "image_name": null,
        "ordered_interfaces": true,
        "version": 2,
        "service_type": "analyzer",
        "flavor": null,
        "service_scaling": false,
        "vrouter_instance_type": null
    },
    "display_name": "analyzer-v2"
}
```

#### Service instance "analyzer-si"
```
{
    "fq_name": [
        "default-domain",
        "demo",
        "analyzer-si"
    ],
    "uuid": "bea15a63-b9d4-40fd-b994-c0800fc16cda",
    "service_instance_properties": {
        "right_virtual_network": null,
        "left_ip_address": null,
        "availability_zone": null,
        "management_virtual_network": null,
        "auto_policy": false,
        "ha_mode": null,
        "virtual_router_id": null,
        "scale_out": null,
        "right_ip_address": null,
        "left_virtual_network": null,
        "interface_list": [
            {
                "virtual_network": "default-domain:demo:management",
                "ip_address": null,
                "static_routes": null,
                "allowed_address_pairs": null
            }
        ]
    },
    "parent_type": "project",
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "instance_ip_refs": [
        {
            "to": [
                "bea15a63-b9d4-40fd-b994-c0800fc16cda-left-v4"
            ],
            "href": "http://10.87.68.167:8082/instance-ip/83475260-67cd-4a27-82dd-9ee2ec99b9da",
            "attr": {
                "interface_type": "left"
            },
            "uuid": "83475260-67cd-4a27-82dd-9ee2ec99b9da"
        }
    ],
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-09-24T04:21:19.399823",
        "uuid": {
            "uuid_mslong": 13736359722822680829,
            "uuid_lslong": 13372524849822526682
        },
        "user_visible": true,
        "last_modified": "2017-09-24T04:21:19.628233",
        "permissions": {
            "owner": "admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "KeystoneAdmin",
            "group_access": 7
        }
    },
    "display_name": "analyzer-si",
    "service_template_refs": [
        {
            "to": [
                "default-domain",
                "analyzer-v2"
            ],
            "href": "http://10.87.68.167:8082/service-template/0a357fe1-bccf-460a-ae54-eac61b213c1b",
            "attr": null,
            "uuid": "0a357fe1-bccf-460a-ae54-eac61b213c1b"
        }
    ]
}
```

#### Policy "mirror"
```
{
    "fq_name": [
        "default-domain",
        "demo",
        "mirror"
    ],
    "uuid": "12c63907-1f03-48ea-84ef-4963463d02eb",
    "parent_type": "project",
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-09-24T04:28:22.638528",
        "uuid": {
            "uuid_mslong": 1352831440819276010,
            "uuid_lslong": 9578955623169327851
        },
        "user_visible": true,
        "last_modified": "2017-09-24T04:28:22.638528",
        "permissions": {
            "owner": "admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "KeystoneAdmin",
            "group_access": 7
        }
    },
    "network_policy_entries": {
        "policy_rule": [
            {
                "direction": "<>",
                "protocol": "any",
                "dst_addresses": [
                    {
                        "security_group": null,
                        "subnet": null,
                        "virtual_network": "any",
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "action_list": {
                    "gateway_name": null,
                    "log": false,
                    "alert": false,
                    "qos_action": null,
                    "assign_routing_instance": null,
                    "mirror_to": {
                        "analyzer_name": "default-domain:demo:analyzer-si",
                        "analyzer_mac_address": null,
                        "juniper_header": true,
                        "udp_port": null,
                        "analyzer_ip_address": null,
                        "static_nh_header": null,
                        "routing_instance": null,
                        "encapsulation": null,
                        "nh_mode": null
                    },
                    "simple_action": null,
                    "apply_service": []
                },
                "created": null,
                "rule_uuid": "2beb024e-c98c-4359-aa63-e898cc3bce06",
                "dst_ports": [
                    {
                        "end_port": -1,
                        "start_port": -1
                    }
                ],
                "application": [],
                "last_modified": null,
                "ethertype": null,
                "src_addresses": [
                    {
                        "security_group": null,
                        "subnet": null,
                        "virtual_network": "any",
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "rule_sequence": null,
                "src_ports": [
                    {
                        "end_port": -1,
                        "start_port": -1
                    }
                ]
            }
        ]
    },
    "display_name": "mirror"
}
```

#### Virtual network "red"
```
{
    "virtual_network_properties": {
        "forwarding_mode": null,
        "allow_transit": false,
        "network_id": null,
        "mirror_destination": false,
        "vxlan_network_identifier": null,
        "rpf": "enable"
    },
    "is_shared": false,
    "ecmp_hashing_include_fields": {
        "destination_ip": true,
        "ip_protocol": true,
        "source_ip": true,
        "hashing_configured": false,
        "source_port": true,
        "destination_port": true
    },
    "fq_name": [
        "default-domain",
        "demo",
        "red"
    ],
    "uuid": "a983108d-5617-4c16-b3c8-8ab956cfe436",
    "display_name": "red",
    "network_policy_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "blue-red"
            ],
            "href": "http://10.87.68.167:8082/network-policy/033cd8ec-4065-4e16-a799-c3fca5f3a1a3",
            "attr": {
                "timer": null,
                "sequence": {
                    "major": 0,
                    "minor": 0
                }
            },
            "uuid": "033cd8ec-4065-4e16-a799-c3fca5f3a1a3"
        },
        {
            "to": [
                "default-domain",
                "demo",
                "mirror"
            ],
            "href": "http://10.87.68.167:8082/network-policy/12c63907-1f03-48ea-84ef-4963463d02eb",
            "attr": {
                "timer": {
                    "start_time": null,
                    "off_interval": null,
                    "on_interval": null,
                    "end_time": null
                },
                "sequence": {
                    "major": 0,
                    "minor": 0
                }
            },
            "uuid": "12c63907-1f03-48ea-84ef-4963463d02eb"
        }
    ],
    "router_external": false,
    "network_ipam_refs": [
        {
            "to": [
                "default-domain",
                "default-project",
                "default-network-ipam"
            ],
            "href": "http://10.87.68.167:8082/network-ipam/e8e63c76-b883-4fbd-815d-073669650413",
            "attr": {
                "ipam_subnets": [
                    {
                        "subnet": {
                            "ip_prefix": "192.168.10.0",
                            "ip_prefix_len": 24
                        },
                        "dns_server_address": "192.168.10.2",
                        "enable_dhcp": true,
                        "created": "2017-09-21T07:01:18.161641",
                        "default_gateway": "192.168.10.1",
                        "dns_nameservers": [],
                        "dhcp_option_list": null,
                        "subnet_uuid": "49e3347f-44f5-41ab-8919-d9bbf193e6ad",
                        "alloc_unit": 1,
                        "last_modified": "2017-09-21T07:01:18.161641",
                        "host_routes": null,
                        "addr_from_start": true,
                        "subnet_name": "",
                        "allocation_pools": []
                    }
                ],
                "host_routes": null
            },
            "uuid": "e8e63c76-b883-4fbd-815d-073669650413"
        }
    ],
    "parent_type": "project",
    "import_route_target_list": {
        "route_target": []
    },
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "route_target_list": {
        "route_target": []
    },
    "export_route_target_list": {
        "route_target": []
    },
    "flood_unknown_unicast": false,
    "id_perms": {
        "enable": true,
        "description": "",
        "creator": null,
        "created": "2017-09-21T07:01:17.561096",
        "uuid": {
            "uuid_mslong": 12214624813579717654,
            "uuid_lslong": 12954756856761279542
        },
        "user_visible": true,
        "last_modified": "2017-09-24T04:28:28.874384",
        "permissions": {
            "owner": "neutron",
            "owner_access": 7,
            "other_access": 7,
            "group": "_member_",
            "group_access": 7
        }
    },
    "port_security_enabled": true,
    "multi_policy_service_chains_enabled": false,
    "virtual_network_network_id": 5
}
```

### Appendix A.3 Port Mirror with Juniper Header
#### Port
```
{
    "fq_name": [
        "default-domain",
        "demo",
        "2a479330-2917-4ebb-b33b-2f495bac3aa6"
    ],
    "virtual_machine_interface_mac_addresses": {
        "mac_address": [
            "02:2a:47:93:30:29"
        ]
    },
    "virtual_machine_interface_bindings": {
        "key_value_pair": [
            {
                "key": "vnic_type",
                "value": "normal"
            },
            {
                "key": "vif_type",
                "value": "vrouter"
            },
            {
                "key": "host_id",
                "value": "a5s167"
            }
        ]
    },
    "security_group_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "default"
            ],
            "href": "http://10.87.68.167:8082/security-group/58c0c9f0-32dc-4539-9ebf-260729d6c54e",
            "attr": null,
            "uuid": "58c0c9f0-32dc-4539-9ebf-260729d6c54e"
        }
    ],
    "routing_instance_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "red",
                "red"
            ],
            "href": "http://10.87.68.167:8082/routing-instance/3f3f1c7d-4581-4f4f-9edf-e33301927725",
            "attr": {
                "direction": "both",
                "protocol": null,
                "ipv6_service_chain_address": null,
                "dst_mac": null,
                "mpls_label": null,
                "vlan_tag": null,
                "src_mac": null,
                "service_chain_address": null
            },
            "uuid": "3f3f1c7d-4581-4f4f-9edf-e33301927725"
        }
    ],
    "virtual_machine_interface_disable_policy": false,
    "parent_type": "project",
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "red"
            ],
            "href": "http://10.87.68.167:8082/virtual-network/a983108d-5617-4c16-b3c8-8ab956cfe436",
            "attr": null,
            "uuid": "a983108d-5617-4c16-b3c8-8ab956cfe436"
        }
    ],
    "display_name": "2a479330-2917-4ebb-b33b-2f495bac3aa6",
    "id_perms": {
        "enable": true,
        "description": "",
        "creator": null,
        "created": "2017-09-21T07:02:05.807862",
        "uuid": {
            "uuid_mslong": 3046565507996536507,
            "uuid_lslong": 12914968348532161190
        },
        "user_visible": true,
        "last_modified": "2017-09-26T16:40:21.060682",
        "permissions": {
            "owner": "neutron",
            "owner_access": 7,
            "other_access": 7,
            "group": "_member_",
            "group_access": 7
        }
    },
    "virtual_machine_refs": [
        {
            "to": [
                "7001a0ab-b76b-448b-a6b0-c1efd4ffbb36"
            ],
            "href": "http://10.87.68.167:8082/virtual-machine/7001a0ab-b76b-448b-a6b0-c1efd4ffbb36",
            "attr": null,
            "uuid": "7001a0ab-b76b-448b-a6b0-c1efd4ffbb36"
        }
    ],
    "virtual_machine_interface_device_owner": "compute:nova",
    "virtual_machine_interface_properties": {
        "sub_interface_vlan_tag": null,
        "local_preference": null,
        "interface_mirror": {
            "traffic_direction": "both",
            "mirror_to": {
                "analyzer_name": "my-analyzer",
                "analyzer_mac_address": null,
                "juniper_header": true,
                "udp_port": 8099,
                "analyzer_ip_address": "192.168.100.3",
                "static_nh_header": null,
                "routing_instance": null,
                "encapsulation": null,
                "nh_mode": null
            }
        },
        "service_interface_type": null
    },
    "port_security_enabled": true,
    "uuid": "2a479330-2917-4ebb-b33b-2f495bac3aa6"
}
```

### Appendix A.4 Port Mirror without Juniper Header
#### Port
```
{
    "ecmp_hashing_include_fields": {
        "destination_ip": true,
        "ip_protocol": true,
        "source_ip": true,
        "hashing_configured": false,
        "source_port": true,
        "destination_port": true
    },
    "fq_name": [
        "default-domain",
        "demo",
        "2a479330-2917-4ebb-b33b-2f495bac3aa6"
    ],
    "virtual_machine_interface_mac_addresses": {
        "mac_address": [
            "02:2a:47:93:30:29"
        ]
    },
    "virtual_machine_interface_dhcp_option_list": {
        "dhcp_option": []
    },
    "virtual_machine_interface_bindings": {
        "key_value_pair": [
            {
                "key": "vnic_type",
                "value": "normal"
            },
            {
                "key": "host_id",
                "value": "a5s167"
            },
            {
                "key": "vif_type",
                "value": "vrouter"
            }
        ]
    },
    "security_group_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "default"
            ],
            "href": "http://10.87.68.167:8082/security-group/58c0c9f0-32dc-4539-9ebf-260729d6c54e",
            "attr": null,
            "uuid": "58c0c9f0-32dc-4539-9ebf-260729d6c54e"
        }
    ],
    "routing_instance_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "red",
                "red"
            ],
            "href": "http://10.87.68.167:8082/routing-instance/3f3f1c7d-4581-4f4f-9edf-e33301927725",
            "attr": {
                "direction": "both",
                "protocol": null,
                "ipv6_service_chain_address": null,
                "dst_mac": null,
                "mpls_label": null,
                "vlan_tag": null,
                "src_mac": null,
                "service_chain_address": null
            },
            "uuid": "3f3f1c7d-4581-4f4f-9edf-e33301927725"
        }
    ],
    "virtual_machine_interface_disable_policy": false,
    "parent_type": "project",
    "virtual_machine_interface_allowed_address_pairs": {
        "allowed_address_pair": []
    },
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "demo",
                "red"
            ],
            "href": "http://10.87.68.167:8082/virtual-network/a983108d-5617-4c16-b3c8-8ab956cfe436",
            "attr": null,
            "uuid": "a983108d-5617-4c16-b3c8-8ab956cfe436"
        }
    ],
    "display_name": "2a479330-2917-4ebb-b33b-2f495bac3aa6",
    "perms2": {
        "owner": "717e091a693341d18aa784f8344ab87f",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": "",
        "creator": null,
        "created": "2017-09-21T07:02:05.807862",
        "uuid": {
            "uuid_mslong": 3046565507996536507,
            "uuid_lslong": 12914968348532161190
        },
        "user_visible": true,
        "last_modified": "2017-09-27T08:09:22.787056",
        "permissions": {
            "owner": "neutron",
            "owner_access": 7,
            "other_access": 7,
            "group": "_member_",
            "group_access": 7
        }
    },
    "virtual_machine_refs": [
        {
            "to": [
                "7001a0ab-b76b-448b-a6b0-c1efd4ffbb36"
            ],
            "href": "http://10.87.68.167:8082/virtual-machine/7001a0ab-b76b-448b-a6b0-c1efd4ffbb36",
            "attr": null,
            "uuid": "7001a0ab-b76b-448b-a6b0-c1efd4ffbb36"
        }
    ],
    "virtual_machine_interface_device_owner": "compute:nova",
    "virtual_machine_interface_properties": {
        "sub_interface_vlan_tag": null,
        "local_preference": null,
        "interface_mirror": {
            "traffic_direction": "both",
            "mirror_to": {
                "analyzer_name": "my-analyzer",
                "analyzer_mac_address": "02:ab:34:e0:97:57",
                "juniper_header": false,
                "udp_port": null,
                "analyzer_ip_address": null,
                "static_nh_header": null,
                "routing_instance": "default-domain:demo:management:management",
                "encapsulation": null,
                "nh_mode": "dynamic"
            }
        },
        "service_interface_type": null
    },
    "port_security_enabled": true,
    "uuid": "2a479330-2917-4ebb-b33b-2f495bac3aa6"
}
```

