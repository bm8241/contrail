# Solution Guide - Kubernetes

Integration
Namespace
Pod
Service
Security

# 1 Kubernets with Contrail

There are two connections between Kuernets and Contrail in the integration.
* contrail-kube-manager and kube-api-server
* Contrail CNI

![Figure Architecture](Kubernetes/Figure-Architecture.png)

## 1.1 contrail-kube-manager

This service connects to kube-api-server to receive updates. Then it connects to Contrail configuration API server to create necessary configurations (VM, VMI/port, IIP, etc.) to connect container onto overlay. It also sends updates to kube-api-server.

## 1.2 Contrail CNI

Kubelet on each node/minion runs with CNI parameters. When launching container, kubelet invokes CNI to setup networking. Contrail CNI connects to vrouter agent REST API to 1) get necessary configuration and 2) plug container network interface into vrouter.

## 1.3 Gateway

Contrail uses gateway to connect overlay and underlay to provide external access. Gateway is required to support exposing service and ingress features from Kubernetes.

A floating IP pool has to be created in Contrail. The FQ name of FIP pool is configured in /etc/contrailctl/kubemanager.conf for provisioning.
```
[KUBERNETES_VNC]
public_fip_pool = {'domain': 'default-domain', 'project': 'default', 'network': 'public', 'name': 'public-fip-pool'}
```

In the container contrail-kube-manager, the floating IP pool is configured in /etc/contrail/contrail-kubernetes.conf.
```
[VNC]
public_fip_pool = {'domain': 'default-domain', 'project': 'default', 'network': 'public', 'name': 'public-fip-pool'}
```

When exposing service or creating ingress in Kubernetes, a FIP is allocated from this pool as the external IP.


# 2 Namespace

When integrate with Contrail, a Kubernetes namespace can map to a project/tenant or a virtual network.

## 2.1 Single-tenant

When [KUBERNETES].cluster_project in /etc/contrail/contrail-kubernetes.conf is set, it's single-tenant where a Kubernetes namespace maps to a virtual network in Contrail. All unisolated namespaces map to the default virtual network "cluster-network". Each isolated namespace maps to a separated virtual network "&lt;NS name>-vn".

![Figure Single-Tenant](Kubernetes/Figure-Single-Tenant.png)

Here is an example to set [KUBERNETES].cluster_project in /etc/contrailctl/kubemanager.conf to enable single-tenant.
```
[KUBERNETES]
cluster_project = {'domain': 'default-domain', 'project': 'kubernetes'}
```

The followings are created by contrail-kube-manager during initialization.
* Flat IPAM &lt;cluster_project>:pod-ipam with subnet
* IPAM &lt;cluster_project>:service-ipam
* Security group k8s-default-default-default and k8s-default-default-sg for virtual network "cluster-network"
* Virtual network &lt;cluster_project>:cluster-network with pod-ipam and service-ipam

Appendix A.1


### 2.1.1 Unisolated namespace

Create an unisolated namespace.
```
apiVersion: v1
kind: Namespace
metadata:
 name: "dev-unisolated"
```

When Kubernetes creates an unisolated namespace, Contrail creates two SGs, k8s-default-&lt;NS name>-sg and k8s-default-&lt;NS name>-sg. No virtual network is created. Container in all unisolated NS will all be on cluster-network.

Launch a POD in unisolated namespace.
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-1
spec:
  containers:
  - name: nginx
    image: docker.io/nginx
    imagePullPolicy: IfNotPresent
```
```
kubectl create -f nginx-1.yaml -n &lt;namespace>
kubectl get pods -n &lt;namespace>
```

When a pod is launched in unisolated namespace, Contrail (contrail-kube-manager) does the followings.
* Create virtual machine &lt;NS name>-&lt;pod name>__&lt;VM UUID>.
* In virtual router where the pod is launched, add reference to VM.
* Create VMI &lt;cluster_project>:&lt;pod name>__&lt;VMI UUID> refering to the followings.
  * SG k8s-default-&lt;NS name>-default
  * SG k8s-default-&lt;NS name>-sg
  * VN &lt;cluster_project>:cluster-network
* Allocate IP address (cluster IP) from pod-ipam in VN cluster-network. Subnet UUID is specified to allocate from flat IPAM pod-ipam.

PODs in different unisolated namespaces can connect to each other, because they are on the same virtual network in Contrail.

### 2.1.2 Isolated namespace

Create an isolated namespace.
```
apiVersion: v1
kind: Namespace
metadata:
 name: "dev-isolated"
 annotations: {
   "opencontrail.org/isolation" : "true"
 }
```

When an isolated namespace is created in Kubernetes, Contrail creates the followings.
* Virtual network &lt;cluster_project>:&lt;namespace name>-vn
* Security group k8s-default-&lt;namespace name>-default and k8s-default-&lt;namespace name>-sg

When a pod is launched in isolated namespace, Contrail does the followings.
* Create virtual machine &lt;NS name>-&lt;pod name>__&lt;VM UUID>.
* In virtual router where the pod is launched, add reference to VM.
* Create VMI &lt;cluster_project>:&lt;pod name>__&lt;VMI UUID> refering to the followings.
  * SG k8s-default-&lt;NS name>-default
  * SG k8s-default-&lt;NS name>-sg
  * VN &lt;cluster_project>:&lt;NS name>-vn
* Allocate IP address (cluster IP) from pod-ipam in VN &lt;NS name>-vn. Subnet UUID is specified to allocate from flat IPAM pod-ipam.

PODS in different isolated namespaces can't connect to each other, because ports are on different virtual network.

## 2.2 Multi-tenant

When [KUBERNETES].cluster_project in /etc/contrail/contrail-kubernetes.conf is not set, it's multi-tenant where a Kubernetes namespace maps to a tenant/project in Contrail. Pods in unisolated namespaces are launched on the default virtual network "cluster-network". Each isolated namespace maps to a separated virtual network "&lt;NS name>-vn".

![Figure Multi-Tenant](Kubernetes/Figure-Multi-Tenant.png)

The followings are created by contrail-kube-manager during initialization.
* A project/tenant for each existing Kubenetes namespace, like default, kube-public and kube-system
* Flat IPAM default-domain:default:pod-ipam
* IPAM default-domain:default:service-ipam
* Security groups k8s-default-&lt;namespace>-sg and k8s-default-&lt;namespace>-sg for each namespace
* Virtual network default-domain:default:cluster-network with pod-ipam and service-ipam


### 2.2.1 Unisolated namespace

Create an unisolated namespace.
```
apiVersion: v1
kind: Namespace
metadata:
 name: "dev-unisolated"
```

Contrail-kube-manager creates the followings.
* Project default-domain:&lt;namespace>

When launch POD in unisolated namespace, contrail-kube-manager creates the port.
* In project default-domain:&lt;namespace>
* On virtual network default-domain:default:cluster-network
* Get address from IPAM default-domain:default:pod-ipam
* Attached with security groups k8s-default-default-default and k8s-default-default-sg

PODs in different unisolated namespaces can connect to each other, because they are on the same virtual network in Contrail.

### 2.2.2 Isolated namespace

Create an isolated namespace.
```
apiVersion: v1
kind: Namespace
metadata:
 name: "dev-isolated"
 annotations: {
   "opencontrail.org/isolation" : "true"
 }
```

Contrail-cube-manager creates the followings.
* Project default-domain:&lt;namespace>
* Virtual network default-domain:&lt;namespace>:&lt;namespace>-vn associated with default-domain:default:pod-ipam.
* Security groups default-domain:&lt;namespace>:k8s-default-&lt;namespace>-default and default-domain:&lt;namespace>:k8s-default-&lt;namespace>-sg

When launch POD in unisolated namespace, contrail-kube-manager creates the port.
* In project default-domain:&lt;namespace>
* On virtual network default-domain:&lt;namespace>:&lt;namespace>-vn
* Get address from IPAM default-domain:default:pod-ipam
* Attached with security groups default-domain:&lt;namespace>:k8s-default-&lt;namespace>-default and default-domain:&lt;namespace>:k8s-default-&lt;namespace>-sg

PODS in different isolated namespaces can't connect to each other, because ports are on different virtual network.

## 2.3 Customized namespace

Create a customized namespace.
```
apiVersion: v1
kind: Namespace
metadata:
 name: "dev-customized"
 annotations: {
   "opencontrail.org/network": '{"domain": "default-domain", "project": "demo", "name": "red"}'
 }
```

When launch POD in customized namespace, contrail-kube-manager creates the port.
* In project default-domain:default
* On the virtual network mapping to customized namespace
* Get address from IPAM assocated with that virtual network
* Security group?

## 2.4 POD on specified virtual network

Launch a POD on specified virtual network.
```
```

When launch POD on specified virtual network, contrail-kube-manager creates the port.
* In the project mapping to specified or default namespace
* On the specified virtual network
* Get address from IPAM assocated with specified virtual network
* Security group?

## 2.5 Kubernetes network policy

Kubernetes network policy will work as is. It's implemented by security group in Contrail. This will be released with 4.0.1.

## 2.6 POD SNAT

This is supported by Contrail. A router (configuration object) can be configured in Contrail to be the external gateway for a virtual network where containers are launched. This is the same as support to external gateway for OpenStack.


# 3 Service

Kubernetes service supports ClusterIP, NodePort, LoadBalancer and ExternalName. It also supports to specify IP with ExternalIPs. Contrail supports ClusterIP and LoadBalancer, and ExternalIPs.

When a service is created in Kubernetes, a loadbalancer is created in Contrail. The provider of loadbalancer is "native" and ECMP loadbalancing is implemented by vrouter. A floating IP is created as the VIP.

Create an application with multiple instances.
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: web-qa
spec:
  replicas: 2
  selector:
    app: web-qa
  template:
    metadata:
      name: web-qa
      labels:
        app: web-qa
    spec:
      containers:
      - name: web
        image: docker.io/nginx
        imagePullPolicy: IfNotPresent
```

## 3.1 ClusterIP

Create a service in front of those applications. ClusterIP is the default service type.
```
kind: Service
apiVersion: v1
metadata:
  name: web-qa
spec:
  selector:
    app: web-qa
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

When a service is created, contrail-kube-manager does the followings.
* Create LB VMI &lt;cluster_project>:&lt;service name>__&lt;VMI UUID> refering to the followings.
  * SG k8s-default-&lt;NS name>-default
  * SG k8s-default-&lt;NS name>-sg
  * VN &lt;cluster_project>:cluster-network
* Allocate LB IP address (service IP) from service-ipam in VN cluster-network. Subnet UUID is not required.
* Create loadbalancer &lt;cluster_project>:&lt;service name>__&lt;LB UUID>.
  * VIP is the LB IP address.
  * VMI is the LB VMI.
  * Provider is "native".
* Create floating IP as the child of LB IP with the same address, utilizing FIP to support port NAT.
* Create LB listener &lt;cluster_project>:&lt;service name>__&lt;LB UUID>-&lt;protocol>-&lt;port>-&lt;LB listener UUID>.
* Create LB pool &lt;cluster_project>:&lt;service name>__&lt;LB UUID>-&lt;protocol>-&lt;port>-&lt;LB listener UUID>.
* Create lB member &lt;cluster_project>:&lt;pool>:&lt;member UUID>.

When LB is created, "native" LB driver will do the followings.
* Set port mapping in FIP.
* Add VMI of all members into FIP.

With service type ClusterIP, the service can only be accessed within the cluster. A FIP is allocated from service FIP pool in cluster-network and mapped to all POD addresses. When accessing the service address within cluster, vrouter will balance the traffic among PODs.

## 3.2 Loadbalancer

Create a LoadBalancer type of service.
```
kind: Service
apiVersion: v1
metadata:
  name: web-qa
spec:
  selector:
    app: web-qa
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

With service type LoadBalancer, the service is exposed to external. A FIP is allocated from service FIP pool for the access within cluster. Also, a FIP is allocated from public FIP pool and mapped to all POD addresses. The FIP will be advertised to gateway who will do ECMP loadbalancing among PODs.


# 4 Ingress

## 4.1 Single service


## 4.2 Simple fanout


## 4.3 Name based


# Appendix A Single-tenant

## A.1 IPAM

&lt;cluster_project>:pod-ipam
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "pod-ipam"
    ],
    "uuid": "c9641741-c785-456e-845b-a14a253c3572",
    "ipam_subnet_method": "flat-subnet",
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "ipam_subnets": {
        "subnets": [
            {
                "subnet": {
                    "ip_prefix": "10.32.0.0",
                    "ip_prefix_len": 12
                },
                "dns_server_address": "10.47.255.253",
                "enable_dhcp": true,
                "created": null,
                "default_gateway": "10.47.255.254",
                "dns_nameservers": [],
                "dhcp_option_list": null,
                "subnet_uuid": null,
                "alloc_unit": 1,
                "last_modified": null,
                "host_routes": null,
                "addr_from_start": null,
                "subnet_name": null,
                "allocation_pools": []
            }
        ]
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-12-27T18:45:33.957901",
        "uuid": {
            "uuid_mslong": 14511749470582293870,
            "uuid_lslong": 9537393975711511922
        },
        "user_visible": true,
        "last_modified": "2017-12-27T18:45:33.957901",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "display_name": "pod-ipam"
}
```

&lt;cluster_project>:service-ipam
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "service-ipam"
    ],
    "uuid": "526f554a-0bf4-47c6-a8e4-768a3f98cef4",
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-12-27T18:45:34.000690",
        "uuid": {
            "uuid_mslong": 5940060210041472966,
            "uuid_lslong": 12169982429206466292
        },
        "user_visible": true,
        "last_modified": "2017-12-27T18:45:34.000690",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "display_name": "service-ipam"
}
```

## A.2 Security Group

k8s-default-dev-share-default
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "k8s-default-dev-share-default"
    ],
    "uuid": "ad29de07-5ef6-4f55-86bb-52c44827c09d",
    "parent_type": "project",
    "perms2": {
        "owner": "46c31b9b-d21c-4c27-9445-6c94db948b6d",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "security_group_id": 8000010,
    "id_perms": {
        "enable": true,
        "description": "Default security group",
        "creator": null,
        "created": "2018-01-12T09:02:15.110429",
        "uuid": {
            "uuid_mslong": 12477748365846007637,
            "uuid_lslong": 9708444424704868509
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:45:08.899388",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "security_group_entries": {
        "policy_rule": [
            {
                "direction": ">",
                "protocol": "any",
                "dst_addresses": [
                    {
                        "security_group": "local",
                        "subnet": null,
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "action_list": null,
                "created": null,
                "rule_uuid": "dc13bb48-e2a7-4c59-a0b8-740ecfcb9a2c",
                "dst_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ],
                "application": [],
                "last_modified": null,
                "ethertype": "IPv4",
                "src_addresses": [
                    {
                        "security_group": null,
                        "subnet": {
                            "ip_prefix": "0.0.0.0",
                            "ip_prefix_len": 0
                        },
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "rule_sequence": null,
                "src_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ]
            },
            {
                "direction": ">",
                "protocol": "any",
                "dst_addresses": [
                    {
                        "security_group": "local",
                        "subnet": null,
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "action_list": null,
                "created": null,
                "rule_uuid": "a84e2d98-2b8f-45ba-aa75-88494da73b11",
                "dst_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ],
                "application": [],
                "last_modified": null,
                "ethertype": "IPv6",
                "src_addresses": [
                    {
                        "security_group": null,
                        "subnet": {
                            "ip_prefix": "::",
                            "ip_prefix_len": 0
                        },
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "rule_sequence": null,
                "src_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ]
            },
            {
                "direction": ">",
                "protocol": "any",
                "dst_addresses": [
                    {
                        "security_group": null,
                        "subnet": {
                            "ip_prefix": "0.0.0.0",
                            "ip_prefix_len": 0
                        },
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "action_list": null,
                "created": null,
                "rule_uuid": "b7752ec1-6037-4c7f-97a9-291893fbed64",
                "dst_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ],
                "application": [],
                "last_modified": null,
                "ethertype": "IPv4",
                "src_addresses": [
                    {
                        "security_group": "local",
                        "subnet": null,
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "rule_sequence": null,
                "src_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ]
            },
            {
                "direction": ">",
                "protocol": "any",
                "dst_addresses": [
                    {
                        "security_group": null,
                        "subnet": {
                            "ip_prefix": "::",
                            "ip_prefix_len": 0
                        },
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "action_list": null,
                "created": null,
                "rule_uuid": "ea5cd2a8-2d47-47c4-a9ab-390de2317246",
                "dst_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ],
                "application": [],
                "last_modified": null,
                "ethertype": "IPv6",
                "src_addresses": [
                    {
                        "security_group": "local",
                        "subnet": null,
                        "virtual_network": null,
                        "subnet_list": [],
                        "network_policy": null
                    }
                ],
                "rule_sequence": null,
                "src_ports": [
                    {
                        "end_port": 65535,
                        "start_port": 0
                    }
                ]
            }
        ]
    },
    "annotations": {
        "key_value_pair": [
            {
                "key": "namespace",
                "value": "dev-share"
            },
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Namespace"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "k8s-default-dev-share-default"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    },
    "display_name": "k8s-default-dev-share-default"
}
```

k8s-default-dev-share-sg
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "k8s-default-dev-share-sg"
    ],
    "uuid": "791f1c7e-a66e-4c47-ba05-409f00ee2c8e",
    "parent_type": "project",
    "perms2": {
        "owner": "46c31b9b-d21c-4c27-9445-6c94db948b6d",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "security_group_id": 8000017,
    "id_perms": {
        "enable": true,
        "description": "Namespace security group",
        "creator": null,
        "created": "2018-01-12T09:02:15.236401",
        "uuid": {
            "uuid_mslong": 8727725933151013959,
            "uuid_lslong": 13404190917597736078
        },
        "user_visible": true,
        "last_modified": "2018-01-12T09:02:15.275407",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "display_name": "k8s-default-dev-share-sg",
    "annotations": {
        "key_value_pair": [
            {
                "key": "namespace",
                "value": "dev-share"
            },
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Namespace"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "k8s-default-dev-share-sg"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    }
}
```
 
## A.3 Virtual Network

&lt;cluster_project>:cluster-network
```
{
    "virtual_network_properties": {
        "forwarding_mode": "l3",
        "allow_transit": null,
        "network_id": null,
        "mirror_destination": false,
        "vxlan_network_identifier": null,
        "rpf": null
    },
    "fq_name": [
        "default-domain",
        "kubernetes",
        "cluster-network"
    ],
    "uuid": "1b9f7f74-17f0-493a-9108-729f91b43598",
    "address_allocation_mode": "user-defined-subnet-only",
    "mac_aging_time": 300,
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "display_name": "cluster-network",
    "pbb_evpn_enable": false,
    "mac_learning_enabled": false,
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2017-12-27T18:45:34.062865",
        "uuid": {
            "uuid_mslong": 1990449696915605818,
            "uuid_lslong": 10450728964983109016
        },
        "user_visible": true,
        "last_modified": "2017-12-29T10:29:20.685414",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "flood_unknown_unicast": false,
    "layer2_control_word": false,
    "port_security_enabled": true,
    "network_ipam_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "service-ipam"
            ],
            "href": "http://127.0.0.1:8082/network-ipam/526f554a-0bf4-47c6-a8e4-768a3f98cef4",
            "attr": {
                "ipam_subnets": [
                    {
                        "subnet": {
                            "ip_prefix": "10.167.0.0",
                            "ip_prefix_len": 16
                        },
                        "dns_server_address": "10.167.255.253",
                        "enable_dhcp": true,
                        "created": null,
                        "default_gateway": "10.167.255.254",
                        "dns_nameservers": [],
                        "dhcp_option_list": null,
                        "subnet_uuid": "10a8de65-9de8-419b-b14c-180bf2ab3dc9",
                        "alloc_unit": 1,
                        "last_modified": null,
                        "host_routes": null,
                        "addr_from_start": null,
                        "subnet_name": null,
                        "allocation_pools": []
                    }
                ],
                "host_routes": null
            },
            "uuid": "526f554a-0bf4-47c6-a8e4-768a3f98cef4"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "pod-ipam"
            ],
            "href": "http://127.0.0.1:8082/network-ipam/c9641741-c785-456e-845b-a14a253c3572",
            "attr": {
                "ipam_subnets": [
                    {
                        "subnet": null,
                        "dns_server_address": null,
                        "enable_dhcp": true,
                        "created": null,
                        "default_gateway": null,
                        "dns_nameservers": [],
                        "dhcp_option_list": null,
                        "subnet_uuid": "d2b090ce-cbcc-4b00-b50a-cc1ed5468b00",
                        "alloc_unit": 1,
                        "last_modified": null,
                        "host_routes": null,
                        "addr_from_start": null,
                        "subnet_name": null,
                        "allocation_pools": []
                    }
                ],
                "host_routes": null
            },
            "uuid": "c9641741-c785-456e-845b-a14a253c3572"
        }
    ],
    "pbb_etree_enable": false,
    "virtual_network_network_id": 5
}
```

&lt;cluster_project>:dev-vn
```
{
    "virtual_network_properties": {
        "forwarding_mode": "l3",
        "allow_transit": null,
        "network_id": null,
        "mirror_destination": false,
        "vxlan_network_identifier": null,
        "rpf": null
    },
    "fq_name": [
        "default-domain",
        "kubernetes",
        "dev-vn"
    ],
    "uuid": "ce01826b-e3e6-407f-8798-80612018e89c",
    "address_allocation_mode": "flat-subnet-only",
    "mac_aging_time": 300,
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "display_name": "dev-vn",
    "pbb_evpn_enable": false,
    "mac_learning_enabled": false,
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-09T11:40:06.196335",
        "uuid": {
            "uuid_mslong": 14844289246686494847,
            "uuid_lslong": 9770700546218977436
        },
        "user_visible": true,
        "last_modified": "2018-01-09T12:18:55.796399",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "flood_unknown_unicast": false,
    "layer2_control_word": false,
    "port_security_enabled": true,
    "network_ipam_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "pod-ipam"
            ],
            "href": "http://127.0.0.1:8082/network-ipam/c9641741-c785-456e-845b-a14a253c3572",
            "attr": {
                "ipam_subnets": [
                    {
                        "subnet": null,
                        "dns_server_address": null,
                        "enable_dhcp": true,
                        "created": null,
                        "default_gateway": null,
                        "dns_nameservers": [],
                        "dhcp_option_list": null,
                        "subnet_uuid": "48ed8235-efcd-44a1-998c-659e4f5840f4",
                        "alloc_unit": 1,
                        "last_modified": null,
                        "host_routes": null,
                        "addr_from_start": null,
                        "subnet_name": null,
                        "allocation_pools": []
                    }
                ],
                "host_routes": null
            },
            "uuid": "c9641741-c785-456e-845b-a14a253c3572"
        }
    ],
    "annotations": {
        "key_value_pair": [
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Namespace"
            },
            {
                "key": "namespace",
                "value": "dev"
            },
            {
                "key": "isolated",
                "value": "True"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "dev"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    },
    "pbb_etree_enable": false,
    "virtual_network_network_id": 11
}
```

## A.4 Virtual Machine Interface and Instance IP

Unisolated port.
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "dev-web-k528t__5a1fc03e-f7ab-11e7-8f66-52540065dced"
    ],
    "virtual_machine_interface_mac_addresses": {
        "mac_address": [
            "02:5a:1f:c0:3e:f7"
        ]
    },
    "display_name": "dev-share__dev-web-k528t",
    "security_group_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "k8s-default-dev-share-default"
            ],
            "href": "http://127.0.0.1:8082/security-group/ad29de07-5ef6-4f55-86bb-52c44827c09d",
            "attr": null,
            "uuid": "ad29de07-5ef6-4f55-86bb-52c44827c09d"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "k8s-default-dev-share-sg"
            ],
            "href": "http://127.0.0.1:8082/security-group/791f1c7e-a66e-4c47-ba05-409f00ee2c8e",
            "attr": null,
            "uuid": "791f1c7e-a66e-4c47-ba05-409f00ee2c8e"
        }
    ],
    "routing_instance_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "cluster-network",
                "cluster-network"
            ],
            "href": "http://127.0.0.1:8082/routing-instance/5ed7608a-28bb-4735-a8d8-2e9132b03d62",
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
            "uuid": "5ed7608a-28bb-4735-a8d8-2e9132b03d62"
        }
    ],
    "virtual_machine_interface_disable_policy": false,
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "cluster-network"
            ],
            "href": "http://127.0.0.1:8082/virtual-network/1b9f7f74-17f0-493a-9108-729f91b43598",
            "attr": null,
            "uuid": "1b9f7f74-17f0-493a-9108-729f91b43598"
        }
    ],
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:14:58.189964",
        "uuid": {
            "uuid_mslong": 6494120564367233511,
            "uuid_lslong": 10333036915785587949
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:14:58.253769",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "virtual_machine_refs": [
        {
            "to": [
                "dev-web-k528t__708154c6-f7ab-11e7-a9df-98f2b3a36be0"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine/708154c6-f7ab-11e7-a9df-98f2b3a36be0",
            "attr": null,
            "uuid": "708154c6-f7ab-11e7-a9df-98f2b3a36be0"
        }
    ],
    "vlan_tag_based_bridge_domain": false,
    "port_security_enabled": true,
    "annotations": {
        "key_value_pair": [
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Pod"
            },
            {
                "key": "namespace",
                "value": "dev-share"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "dev-web-k528t"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    },
    "uuid": "5a1fc03e-f7ab-11e7-8f66-52540065dced"
}
```

Instance IP
```
{
    "fq_name": [
        "dev-web-k528t__5a2f9cde-f7ab-11e7-8f66-52540065dced"
    ],
    "uuid": "5a2f9cde-f7ab-11e7-8f66-52540065dced",
    "service_health_check_ip": false,
    "instance_ip_address": "10.47.255.251",
    "perms2": {
        "owner": "cloud-admin",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "annotations": {
        "key_value_pair": [
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Pod"
            },
            {
                "key": "namespace",
                "value": "dev-share"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "dev-web-k528t"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    },
    "subnet_uuid": "d2b090ce-cbcc-4b00-b50a-cc1ed5468b00",
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:14:58.323069",
        "uuid": {
            "uuid_mslong": 6498585268770771431,
            "uuid_lslong": 10333036915785587949
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:14:58.363792",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "virtual_machine_interface_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-web-k528t__5a1fc03e-f7ab-11e7-8f66-52540065dced"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/5a1fc03e-f7ab-11e7-8f66-52540065dced",
            "attr": null,
            "uuid": "5a1fc03e-f7ab-11e7-8f66-52540065dced"
        }
    ],
    "service_instance_ip": false,
    "instance_ip_local_ip": false,
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "cluster-network"
            ],
            "href": "http://127.0.0.1:8082/virtual-network/1b9f7f74-17f0-493a-9108-729f91b43598",
            "attr": null,
            "uuid": "1b9f7f74-17f0-493a-9108-729f91b43598"
        }
    ],
    "instance_ip_secondary": false,
    "display_name": "dev-share__dev-web-k528t"
}
```

Isolated port.
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "dev-client__c64b3b12-f7b5-11e7-8f66-52540065dced"
    ],
    "virtual_machine_interface_mac_addresses": {
        "mac_address": [
            "02:c6:4b:3b:12:f7"
        ]
    },
    "display_name": "dev__dev-client",
    "security_group_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "k8s-default-dev-sg"
            ],
            "href": "http://127.0.0.1:8082/security-group/579019d5-038e-4901-b6ab-ed146022dd70",
            "attr": null,
            "uuid": "579019d5-038e-4901-b6ab-ed146022dd70"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "k8s-default-dev-default"
            ],
            "href": "http://127.0.0.1:8082/security-group/e43caf6e-6b35-40c3-b336-83c155078efe",
            "attr": null,
            "uuid": "e43caf6e-6b35-40c3-b336-83c155078efe"
        }
    ],
    "routing_instance_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-vn",
                "dev-vn"
            ],
            "href": "http://127.0.0.1:8082/routing-instance/45173786-a1b4-4c75-8ef0-590de67d2d05",
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
            "uuid": "45173786-a1b4-4c75-8ef0-590de67d2d05"
        }
    ],
    "virtual_machine_interface_disable_policy": false,
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-vn"
            ],
            "href": "http://127.0.0.1:8082/virtual-network/ce01826b-e3e6-407f-8798-80612018e89c",
            "attr": null,
            "uuid": "ce01826b-e3e6-407f-8798-80612018e89c"
        }
    ],
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T16:29:34.640295",
        "uuid": {
            "uuid_mslong": 14288579195414319591,
            "uuid_lslong": 10333036915785587949
        },
        "user_visible": true,
        "last_modified": "2018-01-12T16:29:34.708511",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "virtual_machine_refs": [
        {
            "to": [
                "dev-client__c64878a1-f7b5-11e7-9dbb-98f2b3a33b90"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine/c64878a1-f7b5-11e7-9dbb-98f2b3a33b90",
            "attr": null,
            "uuid": "c64878a1-f7b5-11e7-9dbb-98f2b3a33b90"
        }
    ],
    "vlan_tag_based_bridge_domain": false,
    "port_security_enabled": true,
    "annotations": {
        "key_value_pair": [
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Pod"
            },
            {
                "key": "namespace",
                "value": "dev"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "dev-client"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    },
    "uuid": "c64b3b12-f7b5-11e7-8f66-52540065dced"
}
```

```
{
    "fq_name": [
        "dev-client__c65c2a12-f7b5-11e7-8f66-52540065dced"
    ],
    "uuid": "c65c2a12-f7b5-11e7-8f66-52540065dced",
    "service_health_check_ip": false,
    "instance_ip_address": "10.47.255.250",
    "perms2": {
        "owner": "cloud-admin",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "annotations": {
        "key_value_pair": [
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Pod"
            },
            {
                "key": "namespace",
                "value": "dev"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "dev-client"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    },
    "subnet_uuid": "4b421367-165a-4555-80ab-2cff90cb9401",
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T16:29:34.763793",
        "uuid": {
            "uuid_mslong": 14293345578320728551,
            "uuid_lslong": 10333036915785587949
        },
        "user_visible": true,
        "last_modified": "2018-01-12T16:29:34.810063",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "virtual_machine_interface_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-client__c64b3b12-f7b5-11e7-8f66-52540065dced"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/c64b3b12-f7b5-11e7-8f66-52540065dced",
            "attr": null,
            "uuid": "c64b3b12-f7b5-11e7-8f66-52540065dced"
        }
    ],
    "service_instance_ip": false,
    "instance_ip_local_ip": false,
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-vn"
            ],
            "href": "http://127.0.0.1:8082/virtual-network/ce01826b-e3e6-407f-8798-80612018e89c",
            "attr": null,
            "uuid": "ce01826b-e3e6-407f-8798-80612018e89c"
        }
    ],
    "instance_ip_secondary": false,
    "display_name": "dev__dev-client"
}
```

# Appendix C Service

## C.1 LB VMI

```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "svc-dev-web__20c27603-2d0f-45f5-9647-defe4adaba9a"
    ],
    "virtual_machine_interface_mac_addresses": {
        "mac_address": [
            "02:20:c2:76:03:2d"
        ]
    },
    "display_name": "dev-share__svc-dev-web",
    "security_group_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "k8s-default-dev-share-sg"
            ],
            "href": "http://127.0.0.1:8082/security-group/791f1c7e-a66e-4c47-ba05-409f00ee2c8e",
            "attr": null,
            "uuid": "791f1c7e-a66e-4c47-ba05-409f00ee2c8e"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "k8s-default-dev-share-default"
            ],
            "href": "http://127.0.0.1:8082/security-group/ad29de07-5ef6-4f55-86bb-52c44827c09d",
            "attr": null,
            "uuid": "ad29de07-5ef6-4f55-86bb-52c44827c09d"
        }
    ],
    "routing_instance_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "cluster-network",
                "cluster-network"
            ],
            "href": "http://127.0.0.1:8082/routing-instance/5ed7608a-28bb-4735-a8d8-2e9132b03d62",
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
            "uuid": "5ed7608a-28bb-4735-a8d8-2e9132b03d62"
        }
    ],
    "virtual_machine_interface_disable_policy": false,
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "cluster-network"
            ],
            "href": "http://127.0.0.1:8082/virtual-network/1b9f7f74-17f0-493a-9108-729f91b43598",
            "attr": null,
            "uuid": "1b9f7f74-17f0-493a-9108-729f91b43598"
        }
    ],
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.324801",
        "uuid": {
            "uuid_mslong": 2360578910708516341,
            "uuid_lslong": 10828869012794555034
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:05.365345",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "vlan_tag_based_bridge_domain": false,
    "virtual_machine_interface_device_owner": "K8S:LOADBALANCER",
    "port_security_enabled": true,
    "uuid": "20c27603-2d0f-45f5-9647-defe4adaba9a"
}
```


## C.2 LB Instance IP and Floating IP

Instance IP
```
{
    "fq_name": [
        "svc-dev-web__ff9782ea-f79d-423e-af9e-cde45ef847f2"
    ],
    "uuid": "ff9782ea-f79d-423e-af9e-cde45ef847f2",
    "service_health_check_ip": false,
    "instance_ip_address": "10.167.87.84",
    "perms2": {
        "owner": "cloud-admin",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "virtual_network_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "cluster-network"
            ],
            "href": "http://127.0.0.1:8082/virtual-network/1b9f7f74-17f0-493a-9108-729f91b43598",
            "attr": null,
            "uuid": "1b9f7f74-17f0-493a-9108-729f91b43598"
        }
    ],
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.433006",
        "uuid": {
            "uuid_mslong": 18417333146843169342,
            "uuid_lslong": 12654778383687239666
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:05.433006",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "virtual_machine_interface_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "svc-dev-web__20c27603-2d0f-45f5-9647-defe4adaba9a"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/20c27603-2d0f-45f5-9647-defe4adaba9a",
            "attr": null,
            "uuid": "20c27603-2d0f-45f5-9647-defe4adaba9a"
        }
    ],
    "service_instance_ip": false,
    "instance_ip_local_ip": false,
    "instance_ip_secondary": false,
    "display_name": "svc-dev-web"
}
```

Floating IP
```
{
    "project_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes"
            ],
            "href": "http://127.0.0.1:8082/project/46c31b9b-d21c-4c27-9445-6c94db948b6d",
            "attr": null,
            "uuid": "46c31b9b-d21c-4c27-9445-6c94db948b6d"
        }
    ],
    "fq_name": [
        "svc-dev-web__ff9782ea-f79d-423e-af9e-cde45ef847f2",
        "dee62bd0-ed5a-4ac5-b7d7-dc6f329cdba7"
    ],
    "uuid": "dee62bd0-ed5a-4ac5-b7d7-dc6f329cdba7",
    "floating_ip_port_mappings": {
        "port_mappings": [
            {
                "protocol": "TCP",
                "src_port": 80,
                "dst_port": 80
            }
        ]
    },
    "parent_type": "instance-ip",
    "perms2": {
        "owner": "cloud-admin",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.562790",
        "uuid": {
            "uuid_mslong": 16061573297398762181,
            "uuid_lslong": 13247299199082224551
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:06.073466",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "floating_ip_address": "10.167.87.84",
    "virtual_machine_interface_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-web-669n0__59f3d2a8-f7ab-11e7-8f66-52540065dced"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/59f3d2a8-f7ab-11e7-8f66-52540065dced",
            "attr": null,
            "uuid": "59f3d2a8-f7ab-11e7-8f66-52540065dced"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-web-k528t__5a1fc03e-f7ab-11e7-8f66-52540065dced"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/5a1fc03e-f7ab-11e7-8f66-52540065dced",
            "attr": null,
            "uuid": "5a1fc03e-f7ab-11e7-8f66-52540065dced"
        }
    ],
    "floating_ip_port_mappings_enable": true,
    "display_name": "dee62bd0-ed5a-4ac5-b7d7-dc6f329cdba7",
    "floating_ip_traffic_direction": "ingress"
}
```


## C.3 LB

Loadbalancer
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90"
    ],
    "uuid": "34f826d8-f7ac-11e7-9dbb-98f2b3a33b90",
    "service_appliance_set_refs": [
        {
            "to": [
                "default-global-system-config",
                "native"
            ],
            "href": "http://127.0.0.1:8082/service-appliance-set/d5cf94dd-6556-40fc-b3dd-0020dacf7cfc",
            "attr": null,
            "uuid": "d5cf94dd-6556-40fc-b3dd-0020dacf7cfc"
        }
    ],
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "loadbalancer_properties": {
        "status": null,
        "provisioning_status": "ACTIVE",
        "admin_state": true,
        "vip_address": "10.167.87.84",
        "vip_subnet_id": null,
        "operating_status": "ONLINE"
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.486093",
        "uuid": {
            "uuid_mslong": 3816843397506535911,
            "uuid_lslong": 11365846252762905488
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:05.514920",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "virtual_machine_interface_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "svc-dev-web__20c27603-2d0f-45f5-9647-defe4adaba9a"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/20c27603-2d0f-45f5-9647-defe4adaba9a",
            "attr": null,
            "uuid": "20c27603-2d0f-45f5-9647-defe4adaba9a"
        }
    ],
    "display_name": "dev-share__svc-dev-web",
    "loadbalancer_provider": "native",
    "annotations": {
        "key_value_pair": [
            {
                "key": "cluster",
                "value": "k8s-default"
            },
            {
                "key": "kind",
                "value": "Service"
            },
            {
                "key": "namespace",
                "value": "dev-share"
            },
            {
                "key": "project",
                "value": "kubernetes"
            },
            {
                "key": "name",
                "value": "svc-dev-web"
            },
            {
                "key": "owner",
                "value": "k8s"
            }
        ]
    }
}
```

LB Listener
```
{
    "loadbalancer_listener_properties": {
        "default_tls_container": null,
        "protocol": "TCP",
        "connection_limit": null,
        "admin_state": true,
        "sni_containers": [],
        "protocol_port": 80
    },
    "fq_name": [
        "default-domain",
        "kubernetes",
        "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90-TCP-80-331d4fc1-7e80-47a7-a6a0-6cef54c37b6c"
    ],
    "uuid": "331d4fc1-7e80-47a7-a6a0-6cef54c37b6c",
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.564006",
        "uuid": {
            "uuid_mslong": 3683187762728552359,
            "uuid_lslong": 12006716381744823148
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:05.564006",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "loadbalancer_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90"
            ],
            "href": "http://127.0.0.1:8082/loadbalancer/34f826d8-f7ac-11e7-9dbb-98f2b3a33b90",
            "attr": null,
            "uuid": "34f826d8-f7ac-11e7-9dbb-98f2b3a33b90"
        }
    ],
    "display_name": "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90-TCP-80-331d4fc1-7e80-47a7-a6a0-6cef54c37b6c"
}
```

LB Pool
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90-TCP-80-331d4fc1-7e80-47a7-a6a0-6cef54c37b6c"
    ],
    "uuid": "3ed542dc-cbc5-4b47-aeb7-c35f8443a672",
    "parent_type": "project",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "loadbalancer_listener_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90-TCP-80-331d4fc1-7e80-47a7-a6a0-6cef54c37b6c"
            ],
            "href": "http://127.0.0.1:8082/loadbalancer-listener/331d4fc1-7e80-47a7-a6a0-6cef54c37b6c",
            "attr": null,
            "uuid": "331d4fc1-7e80-47a7-a6a0-6cef54c37b6c"
        }
    ],
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.646375",
        "uuid": {
            "uuid_mslong": 4527598516469844807,
            "uuid_lslong": 12589746098345846386
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:05.646375",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "loadbalancer_pool_properties": {
        "status": null,
        "protocol": "TCP",
        "subnet_id": null,
        "session_persistence": null,
        "admin_state": true,
        "persistence_cookie_name": null,
        "status_description": null,
        "loadbalancer_method": null
    },
    "display_name": "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90-TCP-80-331d4fc1-7e80-47a7-a6a0-6cef54c37b6c"
}
```

LB Member
```
{
    "fq_name": [
        "default-domain",
        "kubernetes",
        "svc-dev-web__34f826d8-f7ac-11e7-9dbb-98f2b3a33b90-TCP-80-331d4fc1-7e80-47a7-a6a0-6cef54c37b6c",
        "53d85c7f-6b13-482e-8706-92142bfa2543"
    ],
    "uuid": "53d85c7f-6b13-482e-8706-92142bfa2543",
    "parent_type": "loadbalancer-pool",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T15:21:05.811773",
        "uuid": {
            "uuid_mslong": 6041680602444548142,
            "uuid_lslong": 9729624660315350339
        },
        "user_visible": true,
        "last_modified": "2018-01-12T15:21:05.830431",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "display_name": "53d85c7f-6b13-482e-8706-92142bfa2543",
    "loadbalancer_member_properties": {
        "status": null,
        "status_description": null,
        "weight": 1,
        "admin_state": true,
        "address": null,
        "protocol_port": 80
    },
    "annotations": {
        "key_value_pair": [
            {
                "key": "vm",
                "value": "708154c6-f7ab-11e7-a9df-98f2b3a36be0"
            },
            {
                "key": "vmi",
                "value": "5a1fc03e-f7ab-11e7-8f66-52540065dced"
            }
        ]
    }
}
```

## C.4 External FIP

```
{
    "project_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes"
            ],
            "href": "http://127.0.0.1:8082/project/46c31b9b-d21c-4c27-9445-6c94db948b6d",
            "attr": null,
            "uuid": "46c31b9b-d21c-4c27-9445-6c94db948b6d"
        }
    ],
    "fq_name": [
        "default-domain",
        "kubernetes",
        "BGP",
        "BGP",
        "svc-dev-web__1526aa69-f7bf-11e7-9dbb-98f2b3a33b90120.136.134.67-externalIP"
    ],
    "uuid": "ac091da2-28d7-467f-bd49-10edb2885219",
    "floating_ip_port_mappings": {
        "port_mappings": [
            {
                "protocol": "TCP",
                "src_port": 80,
                "dst_port": 80
            }
        ]
    },
    "parent_type": "floating-ip-pool",
    "perms2": {
        "owner": "None",
        "owner_access": 7,
        "global_access": 0,
        "share": []
    },
    "id_perms": {
        "enable": true,
        "description": null,
        "creator": null,
        "created": "2018-01-12T17:36:13.280888",
        "uuid": {
            "uuid_mslong": 12396472031621105279,
            "uuid_lslong": 13639451559556829721
        },
        "user_visible": true,
        "last_modified": "2018-01-12T17:36:13.424379",
        "permissions": {
            "owner": "cloud-admin",
            "owner_access": 7,
            "other_access": 7,
            "group": "cloud-admin-group",
            "group_access": 7
        }
    },
    "floating_ip_address": "120.136.134.67",
    "virtual_machine_interface_refs": [
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-web-669n0__59f3d2a8-f7ab-11e7-8f66-52540065dced"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/59f3d2a8-f7ab-11e7-8f66-52540065dced",
            "attr": null,
            "uuid": "59f3d2a8-f7ab-11e7-8f66-52540065dced"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "svc-dev-web__78f5adca-cbfe-422a-810c-bb3be9c15589"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/78f5adca-cbfe-422a-810c-bb3be9c15589",
            "attr": null,
            "uuid": "78f5adca-cbfe-422a-810c-bb3be9c15589"
        },
        {
            "to": [
                "default-domain",
                "kubernetes",
                "dev-web-k528t__5a1fc03e-f7ab-11e7-8f66-52540065dced"
            ],
            "href": "http://127.0.0.1:8082/virtual-machine-interface/5a1fc03e-f7ab-11e7-8f66-52540065dced",
            "attr": null,
            "uuid": "5a1fc03e-f7ab-11e7-8f66-52540065dced"
        }
    ],
    "floating_ip_port_mappings_enable": true,
    "display_name": "svc-dev-web__1526aa69-f7bf-11e7-9dbb-98f2b3a33b90120.136.134.67-externalIP",
    "floating_ip_traffic_direction": "ingress"
}
```


