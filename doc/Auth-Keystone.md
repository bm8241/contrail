
# 1 Overview

Contrail provides configuration API and analytics API for user to configure objects and query operational status. Authention for API request is supported by the integration with Keystone (OpenStack service).


## 1.1 Workflow

```
  Client                                          Keystone
    |                                                |
    |  username and password                         |
    |----------------------------------------------->|
    |  token                                         |
    |<-----------------------------------------------|
    |                                                |
    |                   API Server                   |
    |                       |                        |
    |  request and token    |                        |
    |---------------------->|                        |
    |                       |  token                 |
    |                       |----------------------->|
    |                       |  response with role    |
    |                       |<-----------------------|
    |  response             |                        |
    |<----------------------|
    |                       |
```


## 1.2 Configuration API

Configuration API supports 'no-auth', 'cloud-admin' and 'rbac'. It's configured in /etc/contrail/contrail-api.conf. Here is an example.
```
aaa_mode = cloud-admin
```

*   no-auth

    Authentication is not required for any requests.

*   cloud-admin

    Authentication is required, the user has to have role 'admin' in authentication tenant/project.

*   rbac

    Authentication is required, also RBAC (role based access control) is enabled. Each user may have different privilege in certian scope based on RBAC rules.


## 1.3 Analytics API

Analytics API supports 'no-auth' and 'cloud-admin'.


# 2 Authentication

For Contrail services to access API, the authentication configuration is in /etc/contrail/contrail-keystone-auth.conf. For Neutron Contrail plugin, the authentication configuration is in /etc/neutron/plugins/opencontrail/ContrailPlugin.ini on Neutron API server node.


## 2.1 Keystone v2

Here is an example of /etc/neutron/plugins/opencontrail/ContrailPlugin.ini.
```
[APISERVER]
api_server_ip =172.16.0.150
api_server_port =8082
multi_tenancy =True
contrail_extensions = ipam:neutron_plugin_contrail.plugins.opencontrail.contrail_plugin_ipam.NeutronPluginContrailIpam,policy:neutron_plugin_contrail.plugins.opencontrail.contrail_plugin_policy.NeutronPluginContrailPolicy,route-table:neutron_plugin_contrail.plugins.opencontrail.contrail_plugin_vpc.NeutronPluginContrailVpc,contrail:None


[COLLECTOR]
analytics_api_ip =172.16.0.150
analytics_api_port =8081

[KEYSTONE]
auth_url=http://172.16.0.240:5000
admin_user=admin
admin_password=contrail123
auth_user=admin
admin_tenant_name=admin
```

Neutron Contrail plugin supports Keystone v2 only, not v3. In case of HA deployment, 'api_server_ip' and 'analytics_api_ip' point to the VIP. The list of server addresses is not supported. That means a LB is required in front of Contrail configuration and analytics API services to serve Neutron Contrail plugin.


## 2.2 Keystone v3

Here is an example of /etc/contrail/contrail-keystone-auth.conf.
```
[KEYSTONE]
auth_url=http://172.16.0.240:35357/v3
auth_host=172.16.0.240
auth_protocol=http
auth_port=35357
admin_user=admin
admin_password=contrail123
admin_tenant_name=admin
memcache_servers=127.0.0.1:11911
auth_type = password
user_domain_name = Default
domain_id = default
;insecure=False
;certfile=/etc/contrail/ssl/certs/keystone.pem
;keyfile=/etc/contrail/ssl/certs/keystone.pem
;cafile=/etc/contrail/ssl/certs/keystone_ca.pem
```


# 3 RBAC


# 4 Project synchronization

When a project is created in OpenStack, Contrail won't create the same project (same name and UUID) until there is request to this project in Contrail. Here is a script to trigger Contrail to synchronize project from OpenStack.

```
#!/bin/bash

get_project()
{
    uuid=${id:0:8}'-'${id:8:4}'-'${id:12:4}'-'${id:16:4}'-'${id:20}
    echo $uuid
    curl -s -H "X-Auth-Token: $token" \
        http://10.87.68.151:8082/project/$uuid
    echo ""
}

token=$(openstack token issue | awk "/ id /"'{print $4}')

id=$(openstack project list | awk "/$1/"'{print $2}')

get_project
```

Project creation in Contrail is triggered by request only, so only required projects will be created. (keystone_sync_on_demand is True.)

When a project is deleted in OpenStack, Contrail will try to delete the same project triggered by the re-sync timer, which is 60s by default and specified by [DEFAULT].keystone_resync_interval_secs.

Note, OpenStack doesn't check if a project is empty when deleting it. Contrail won't delete a project until it's empty. So, when a non-empty project is deleted from OpenStack, it will stay in Contrail. If a new project with the same name but different UUID is created in OpenStack, Contrail will create a new project with FQ name &lt;domain>:&lt;project name>-&lt;UUID> to differentiate the stale project &lt;domain>:&lt;project name>.


# 5 Secured access


# 6 Troubleshoot


