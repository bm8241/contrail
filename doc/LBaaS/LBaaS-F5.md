# 1 Architecture

## 1.1 With OpenStack

[F5 OpenStack Solutions](http://clouddocs.f5.com/cloud/openstack/v1/)

```
        Neutron API
            |
            |
    Neutron LBaaS plugin
            |
            |
      F5 LBaaS driver
            |
            |
      F5 LBaaS agent
            |
            |
       F5 device
```
[F5 Driver for OpenStack LBaaSv2](http://clouddocs.f5.com/products/openstack/lbaasv2-driver/master/)

[F5 Agent for OpenStack Neutron](http://clouddocs.f5.com/products/openstack/agent/master/)


## 1.2 With Contrail

```
            Neutron API
                |
                |
    Neutron Contrail LBaaS plugin
                |
                |
                |
    Contrail Configuration API
                |
                |
    Contrail service monitor
                |
                |
    Contrail F5 LBaaS driver
                |
                |
         F5 LBaaS agent
                |
                |
            F5 device
```
Contrail service monitor provides a plugin interface to LBaaS drivers to support LB from different vendors. While OpenStack can only support LBaaS driver from one provider, Contrail can support multiple providers. For example, user may create HAProxy LB, F5 LTM LB and A10 LB.

Contrail F5 LBaaS driver is a shim layer to convert data from service monitor to F5 LBaaS agent.


# 2 Installation

For now, F5 LBaaS driver and agent is not part of Contrail installation yet, so they have to be installed manually.

Copy Contrail F5 LBaaS driver.
```
/usr/lib/python2.7/dist-packages/svc_monitor/services/loadbalancer/drivers/f5/driver.py
```

Get F5 LBaaS agent and dependent packages.
```
wget https://github.com/F5Networks/f5-openstack-agent/releases/download/v9.2.0/python-f5-openstack-agent_9.2.0-1_1404_all.deb
wget https://github.com/F5Networks/f5-common-python/releases/download/v2.2.2/python-f5-sdk_2.2.2-1_1404_all.deb
wget https://github.com/F5Networks/f5-icontrol-rest-python/releases/download/v1.3.0/python-f5-icontrol-rest_1.3.0-1_1404_all.deb
```

Due to the conflict to contrail-f5 package, manually copy files.
```
mv /usr/lib/python2.7/dist-packages/f5 /usr/lib/python2.7/dist-packages/contrail_f5
dpkg --extract python-f5-icontrol-rest_1.3.0-1_1404_all.deb ./
dpkg --extract python-f5-sdk_2.2.2-1_1404_all.deb ./
dpkg --extract python-f5-openstack-agent_9.2.0-1_1404_all.deb ./
cp -r usr/lib/python2.7/dist-packages/* /usr/lib/python2.7/dist-packages/
```

Note, due to an issue to call socket functions, remove monkey patch from f5_openstack_agent/lbaasv2/drivers/bigip/__init__.py.


# 3 Use Case

## 3.1 Virtual LTM

![Figure vLTM](LBaaS/Figure-vLTM.png)

The vLTM is launched as a VM on 3 virtual networks, management, external and internal. Service monitor on underlay needs to access the management interface of vLTM to manage it. The gateway is required to support such overlay and underlay connection. Virtual network 'public' is exposed to underlay by gateway. A floating IP is allocated from 'public' virtual network and assigned to vLTM management interface. Now, service monitor can connect to and manage vLTM with that floating IP.

Once a LB is created with the VIP, the static route of that VIP needs to be added in Contrail.

For the client on virtual network 'external' to use LB service, they can reach vLTM external interface with VIP.

For the client on other virtual network to use LB service, network policy is required to allow the connection between 'external' and other virtual networks.

For the client on underlay to use LB service, route of VIP needs to be advertised to gateway for underlay to access.

## 3.2 Physical LTM

![Figure pLTM](LBaaS/Figure-pLTM.png)

For physical LTM, the management interface connects to IP fabric for service monitor to connect and manage. The external and internal interfaces need to connect to virtual network 'external' and 'internal' in Contrail. Gateway is required to make such underlay overlay connection. The external and internal interfaces connect to VRF RI 'internal' and 'external' respectively on gateway and those two RIs map to virtual networks in Contrail.

Once a LB is created with the VIP, the static route of VIP needs to be created in the RI 'external', and that route will be advertised to virutla network 'external' in Contrail.

For the client on virtual network 'external' to use LB service, they can see the VIP route and reach LTM external interface through gateway.

For the client on other virtual network to use LB service, network policy is required to allow the connection between 'external' and other virtual networks.

For the client on underlay to use LB service, route of VIP has to be leaked to IP fabric on gateway, then the client will be able to reach LTM external interface through gateway.


# 4 Workflow

## 4.1 Build topology

For virtual or physical LTM, build required Contrail configuration, setup gateway and make all physical links.

## 4.2 Service appliance set

This is Contrail configuration to specify provider name, driver and specific arguments. Here is an example.
```
config create sas f5 --driver svc_monitor.services.loadbalancer.drivers.f5.drive
r.OpencontrailF5LoadbalancerDriver --kv address=10.84.31.99 --kv user=admin --kv
 password=admin
```

## 4.3 LBaaS configuration

All LBaaS configuration in Contrail can be done either through Neutron LBaaS interface (in case of OpenStack integration), or through Contrail configuration directly (in case of integration with other orchestration, like Kubernetes).

See [LBaaS OpenStack](https://github.com/tonyliu0592/opencontrail/wiki/LBaaS-OpenStack) for how to configure LBaaS with Neutron CLI.


