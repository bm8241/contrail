# API Guide - Configuration REST API

# 1 Overview

REST API is natively supported by configuration API server for CRUD (Create, Read, Update, Delete) operations to configuration resources.


## 1.1 Reference

REST API guide and reference is published on Juniper site for each Contrail release. Here is the link for 5.1 [https://www.juniper.net/documentation/en_US/contrail5.1/information-products/pathway-pages/api-guide/index.html](https://www.juniper.net/documentation/en_US/contrail5.1/information-products/pathway-pages/api-guide/index.html).

For REST API, here is a [JSON raw file](https://github.com/Juniper/contrail-vnc/raw/R4.1/contrail_openapi.json). Some online tool can be used to explore this documentation, like Swagger. Goto the online editor on http://swagger.io/swagger-editor/, import the URL of JSON file there. This JSON file is not packed into release yet.


## 1.2 Client

There is no prerequisites for client to use REST API.


# 2 Examples

All examples are showed with Bash shell script.


## 2.1 Authentication

When authentication is enabled on configuration API server, the client has to connect to Keystone service to get authenticated (get a token) before sending request out. Here is [Keystone API](https://developer.openstack.org/api-ref/identity/v3/).

```
api_url=http://10.6.11.1:8082
auth_url=http://10.6.11.2:5000/v3
username=admin
password=contrail123
tenant=admin

get_token()
{
    cat << __EOF__ > /tmp/body
{
    "auth": {
        "scope": {
            "project": {
                "domain": {"id": "default"},
                "name": "$tenant"
            }
        },
        "identity": {
            "methods": ["password"],
            "password": {
                "user": {
                    "name": "$username",
                    "domain": {"id": "default"},
                    "password": "$password"
                }
            }
        }
    }
}
__EOF__

    token=$(curl -s -D - -H "Content-Type: application/json" \
        -d @/tmp/body \
        $auth_url/auth/tokens | awk "/X-Subject-Token/"'{print $2}')
}
```


## 2.2 List resource

#### List all resource types.
```
list_all_types()
{
    curl -s -H "X-Auth-Token: $token" $api_url | \
        jq -r '."links"[] | if .link.rel == "collection"
            then .link.name else empty end'
}
```

#### List all resources of a specific type.
```
list_res()
{
    local type=virtual-networks

    curl -s -H "X-Auth-Token: $token" \
        $api_url/$type | \
        jq -r '."virtual-networks"[]."fq_name"'
}
```

#### List resources with queries.

API server provides some query supports for performance purpose. Here is the list of query keywords.
* `count`, output the number of resources.
* `detail`, list resources with properties and references.
* `fields`, list resources with `parent_type`, `parent_uuid` and specified properties. Without `fields` and `detail`, list resources with `href`, `fq_name` and `uuid` only.
* `filters`, list resources matching the filters.
* `shared`, list shared resources with the current project (the project of authenticated user) as well.
* `exclude_hrefs`, list resources without `href`. With `detail`, `href` is also excluded.

Note, `fields` or `filters` has to be the `Properties' defined in each resource class in `vnc_api/gen/resource_common.py`.

Count virtual networks.
```
list_res_with_query()
{
    local type=virtual-networks
    local query="count=true"

    curl -s -H "X-Auth-Token: $token" \
        $api_url/$type?$query | \
        jq -r .
}
```

Count virtual networks whoes network ID is 5.
```
    local query="filters=virtual_network_network_id==5&count=true"
```

Show details of virtual network whoes network ID is 5.
```
    local query="filters=virtual_network_network_id==5&detail=true"
```

List virtual networks with `perms2`.
```
    local query="fields=perms2"
```


## 2.3 Read resource

#### Read resource by UUID.
```
read_res_by_uuid()
{
    local type=virtual-network
    local uuid=d5478260-86b9-4629-b2ca-2aed1e1f37ac

    curl -s -H "X-Auth-Token: $token" \
        $api_url/$type/$uuid | \
        jq -r .
}
```

#### Read resource by FQ name, the FQ name has to be converted to UUID, then use UUID to read the resource.
```
fq_name_to_uuid()
{
    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d "{
            \"type\": \"$1\",
            \"fq_name\": $2
        }" \
        $api_url/fqname-to-id | jq -r .uuid
}

read_res_by_fq_name()
{
    local type=virtual-network
    local fq_name='["default-domain", "admin", "blue"]'
    local uuid

    uuid=$(fq_name_to_uuid "$type" "$fq_name")

    curl -s -H "X-Auth-Token: $token" \
        $api_url/$type/$uuid | \
        jq -r .
}
```

## 2.4 Create resource

#### Create virtual network
```
create_virtual_network()
{
    cat << __EOF__ > /tmp/body
{
    "virtual-network": {
        "parent_type": "project",
        "fq_name": [
            "default-domain",
            "admin",
            "green"
        ],
        "network_ipam_refs": [
            {
                "to": [
                    "default-domain",
                    "default-project",
                    "default-network-ipam"
                ],
                "attr": {
                    "ipam_subnets": [
                        {
                            "subnet": {
                                "ip_prefix": "192.168.30.0",
                                "ip_prefix_len": 24
                            },
                            "enable_dhcp": true
                        }
                    ]
                }
            }
        ]
    }
}
__EOF__

    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d @/tmp/body \
        $api_url/virtual-networks | \
        jq -r .
}
```

#### Create virtual network with specified gateway and service address
```
create_virtual_network()
{
    cat << __EOF__ > /tmp/body
{
    "virtual-network": {
        "parent_type": "project",
        "fq_name": [
            "default-domain",
            "admin",
            "green"
        ],
        "network_ipam_refs": [
            {
                "to": [
                    "default-domain",
                    "default-project",
                    "default-network-ipam"
                ],
                "attr": {
                    "ipam_subnets": [
                        {
                            "subnet": {
                                "ip_prefix": "192.168.30.0",
                                "ip_prefix_len": 24
                            },
                            "default_gateway": "192.168.30.1",
                            "dns_server_address": "192.168.30.2",
                            "enable_dhcp": true
                        }
                    ]
                }
            }
        ]
    }
}
__EOF__

    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d @/tmp/body \
        $api_url/virtual-networks | \
        jq -r .
}
```

#### Gateway address and service address

If gateway address is not specified, the default is the last address in the subnet.

If gateway address is set to "0.0.0.0", gateway is disabled, DHCP option `routers` has to be manually set poiting to some other host who will act as the gateway. It will also free the address.

If service address is not specified, the default is the second to last address in the subnet.

If service address is set to "0.0.0.0", gateway address is used as service address. If both service address and gateway address are set to "0.0.0.0", gateway is disabled, but the last address in subnet is allocated as service address, because service address is always allocated.

#### Create network policy
```
create_policy()
{
    cat << __EOF__ > /tmp/body
{
    "network-policy": {
        "parent_type": "project",
        "fq_name": [
            "default-domain",
            "admin",
            "red-blue"
        ],
        "network_policy_entries": {
            "policy_rule": [
                {
                    "direction": "<>",
                    "protocol": "any",
                    "src_addresses": [
                        {
                            "virtual_network": "default-domain:admin:red"
                        }
                    ],
                    "src_ports": [
                        {
                            "start_port": -1,
                            "end_port": -1
                        }
                    ],
                    "dst_addresses": [
                        {
                            "virtual_network": "default-domain:admin:blue"
                        }
                    ],
                    "dst_ports": [
                        {
                            "start_port": -1,
                            "end_port": -1
                        }
                    ],
                    "action_list": {
                        "simple_action": "pass"
                    }
                }
            ]
        }
    }
}
__EOF__

    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d @/tmp/body \
        $api_url/network-policys | \
        jq -r .
}
```

#### Create service instance
```
#!/bin/bash

vm_id=bfbed976-e0fd-4b73-89aa-ae4d2790ad81
service_instance=default-domain:admin:vsrx
port_tuple=default-domain:admin:vsrx:pt0
service_template=default-domain:firewall
vn_management=default-domain:admin:management
vn_left=default-domain:admin:vsrx-left
vn_right=default-domain:admin:vsrx-right
url=http://10.6.11.2:8082

fq_name_to_uuid()
{
    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d "{
            \"type\": \"$1\",
            \"fq_name\": $2
        }" \
        $api_url/fqname-to-id | jq -r .uuid
}

fqn_str_to_list()
{
    local fqn_str=$1
    local fqn_list="["

    for i in $(echo $fqn_str | tr ':' '\n'); do
        fqn_list+="\"$i\","
    done
    fqn_list=${fqn_list%"${fqn_list##*[!',']}"}
    fqn_list+="]"
    echo $fqn_list
}

create_st()
{
    local fqn=$(fqn_str_to_list $service_template)

    cat << __EOF__ > /tmp/body
{
    "service-template": {
        "parent_type": "domain",
        "fq_name": $fqn,
        "service_template_properties": {
            "version": 2,
            "service_mode": "in-network",
            "service_type": "firewall",
            "service_scaling": false,
            "ordered_interfaces": true,
            "interface_type": [
                {
                    "service_interface_type": "management"
                },
                {
                    "service_interface_type": "left"
                },
                {
                    "service_interface_type": "right"
                }
            ]
        }
    }
}
__EOF__

    curl -s -X POST \
        -H "Content-Type: application/json" \
        -d @/tmp/body \
        $url/service-templates
    echo ""
}

create_si()
{
    local st_fqn=$(fqn_str_to_list $service_template)
    local si_fqn=$(fqn_str_to_list $service_instance)

    st_id=$(fq_name_to_uuid "service-template" "$st_fqn")
    cat << __EOF__ > /tmp/body
{
    "service-instance": {
        "parent_type": "project",
        "fq_name": $si_fqn,
        "service_template_refs": [
            {
                "to": $st_fqn,
                "uuid": $st_id
            }
        ],
        "service_instance_properties": {
            "interface_list": [
                {
                    "virtual_network": "default-domain:admin:management"
                },
                {
                    "virtual_network": "default-domain:admin:left"
                },
                {
                    "virtual_network": "default-domain:admin:right"
                }
            ]
        }
    }
}
__EOF__

    curl -s -X POST \
      -H "Content-Type: application/json" \
      -d @/tmp/body \
      $url/service-instances
    echo ""
}

create_port_tuple()
{
    local pt_fqn=$(fqn_str_to_list $port_tule)

    curl -s -X POST \
      -H "Content-Type: application/json" \
      -d "{
          \"port-tuple\": {
              \"parent_type\": \"service-instance\",
              \"fq_name\": $pt_fqn
          }
      }" $url/port-tuples
    echo ""
}

update_vmi()
{
    local vmi_id=$1
    local type=$2
    local pt_id=$3
    local pt_fqn_list=$4

    cat << __EOF__ > /tmp/body
{
    "virtual-machine-interface": {
        "uuid": "$vmi_id",
        "virtual_machine_interface_properties": {
            "service_interface_type": "$type"
        }
    }
}
__EOF__
    curl -s -X PUT \
      -H "Content-Type: application/json" \
      -d @/tmp/body \
      $url/virtual-machine-interface/$vmi_id
    echo ""

    cat << __EOF__ > /tmp/body
{
    "type": "virtual-machine-interface",
    "uuid": "$vmi_id",
    "ref-type": "port-tuple",
    "ref-fq-name": $pt_fqn_list,
    "ref-uuid": "$pt_id",
    "operation": "ADD",
    "attr": null
}
__EOF__
    curl -s -X POST \
      -H "Content-Type: application/json" \
      -d @/tmp/body \
      $url/ref-update
    echo ""
}

attach_vmi_to_port_tuple()
{
    local type
    local pt_id
    local pt_fqn_list=$(fqn_str_to_list $port_tuple)
    local vmi_id
    local vmi_list

    vmi_list=$(curl -s -X GET $url/virtual-machine/$vm_id?fields=virtual_machine_interface_back_refs | \
      jq -r '."virtual-machine".virtual_machine_interface_back_refs[].to[2]')
 
    pt_id=$(fq_name_to_uuid "port-tuple" "$pt_fqn_list)

    for vmi_id in $vmi_list; do
        fqn_list=$(curl -s -X GET $url/virtual-machine-interface/$vmi_id | \
          jq -r '."virtual-machine-interface".virtual_network_refs[].to[]')
        fqn_str=$(echo $fqn_list | awk '{print $1":"$2":"$3}')
        if [ $fqn_str == $vn_management ]; then
            type=management
        elif [ $fqn_str == $vn_left ]; then
            type=left
        elif [ $fqn_str == $vn_right ]; then
            type=right
        else
            echo "ERROR: interface type."
            exit 0
        fi
        update_vmi $vmi_id $type $pt_id $pt_fqn_list
    done
}

create_st
create_si
create_port_tuple
attach_vmi_to_port_tuple

exit 0
```


## 2.5 Update resource

### 2.5.1 Update property

#### Add a subnet to existing virtual network
Read (GET) resource, make changes, then update (PUT) it.


### 2.5.2 Update reference

#### Attach network policy to virtual network
```
attach_policy()
{
    cat << __EOF__ > /tmp/body
{
    "type": "virtual-network",
    "uuid": "2ca21cc4-b0b6-4e7b-9071-f8acb435a8fa",
    "ref-type": "network-policy",
    "ref-uuid": "151f35c1-9ff6-4a13-a9af-6bfe72ad88ca",
    "ref-fq-name": [
        "default-domain",
        "admin",
        "red-blue"
    ],
    "operation": "ADD"
}
__EOF__

    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d @/tmp/body \
        $api_url/ref-update
}
```

#### Detach network policy from virtual network
```
detach_policy()
{
    cat << __EOF__ > /tmp/body
{
    "type": "virtual-network",
    "uuid": "2ca21cc4-b0b6-4e7b-9071-f8acb435a8fa",
    "ref-type": "network-policy",
    "ref-uuid": "151f35c1-9ff6-4a13-a9af-6bfe72ad88ca",
    "ref-fq-name": [
        "default-domain",
        "admin",
        "red-blue"
    ],
    "operation": "DELETE"
}
__EOF__

    curl -s -X POST -H "X-Auth-Token: $token" \
        -H "Content-Type: application/json" \
        -d @/tmp/body \
        $api_url/ref-update
}
```


## 2.6 Delete resource

UUID has to be specified to delete a resource. Given FQ name, it has to be converted to UUID prior to deleting.

#### Delete virtual network
```
delete_virtual_network_by_fq_name()
{
    local type=virtual-network
    local fq_name='["default-domain", "admin", "green"]'
    local uuid

    uuid=$(fq_name_to_uuid "$type" "$fq_name")
    curl -s -X DELETE -H "X-Auth-Token: $token" \
        $api_url/$type/$uuid
}
```


