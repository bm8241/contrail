* [1 Introduction](#1-introduction)
* [2 Prepare for deployment](#2-prepare-for-deployment)
  * [2.1 POC Environment](#21-poc-environment)
  * [2.2 Networking](#22-networking)
  * [2.3 DNS and NTP](#23-dns-and-ntp)
  * [2.4 Proxy Server](#24-proxy-server)
  * [2.5 Configure Physical Server for Running VM](#25-configure-physical-server-for-running-vm)
  * [2.6 Build VM as Controller](#26-build-vm-as-controller)
  * [2.7 Add VM into MAAS](#27-add-vm-into-maas)
  * [2.8 Compute Node](#28-compute-node)
  * [2.9 Build Contrail Repository](#29-build-contrail-repository)
* [3 Deploy](#3-deploy)
  * [3.1 Create MAAS cloud in Juju](#31-create-maas-cloud-in-juju)
  * [3.2 Bootstrap](#32-bootstrap)
  * [3.3 Deploy](#33-deploy)
  * [3.4 Re-deploy](#34-re-deploy)
* [4 Credit](#4-credit)
* [Appendix A Tagged and Untagged Subnet](#appendix-a-tagged-and-untagged-subnet)
  * [A.1 Primary interface only](#a1-primary-interface-only)
  * [A.2 Sub-interface on physical server only](#a2-sub-interface-on-physical-server-only)
  * [A.3 Sub-interface on both physical server and VM](#a3-sub-interface-on-both-physical-server-and-vm)
* [Appendix B Proxy](#appendix-b-proxy)
* [Appendix C Underlay Networking](#appendix-c-underlay-networking)
* [Appendix D Contrail repository](#appendix-d-contrail-repository)
* [Appendix E Network Interface Configuration](#appendix-e-network-interface-configuration)
  * [E.1 Physical Server for Controller VM](#e1-physical-server-for-controller-vm)
  * [E.2 Controller VM](#e2-controller-vm)
* [Appendix F Bundle Configuration](#appendix-f-bundle-configuration)


# 1 Introduction

This is the POC deployment for Contrail 3.1.2 and OpenStack Mitaka with Juju 2.1.0 and MAAS 2.1.3.

MN: Management Network
DN: Data Network

MA: Management Address
DA: Data Address

# 2 Prepare for deployment

## 2.1 POC Environment

```
  +-----------------+  +-----------------+  +------------+  +-------+
  |    server1      |  |    server2      |  |  server3   |  | MAAS  |
  | +-------------+ |  | +-------------+ |  |            |  |       |
  | | Juju        | |  | | controller2 | |  |  compute1  |  | Juju  |
  | | controller  | |  | +-------------+ |  +------+-----+  +---+---+
  | +-------------+ |  |                 |         |            |
  | +-------------+ |  | +-------------+ |         |            |
  | | controller1 | |  | | controller3 | |         |            |
  | +-------------+ |  | +-------------+ |         |            |
  +--------+--------+  +--------+--------+         |            |
           |                    |                  |            |
           +--------------------+------------------+------------+
               Untagged Data Network
               Tagged VLAN Management Network
```
* MAAS and Juju are already installed.
* 3 physical servers are added into MAAS.
* One physical link on each server connects to untagged DN and tagged VLAN MN.
* Subnet of DN and MN are configured in MAAS.

## 2.2 Networking
For a POC deployment, one single network/subnet is used for all purpose, management, deployment, data and control, API and UI. See details in Appendix C.

The untagged DN subnet and tagged MN subnet are already configured and in space 'space-0'. Create space 'space-mgmt' and put tagged MN subnet in it.

In MN subnet, reserve address block for VIPs, MX gateways and any manually allocated addresses.

Ensure the gateway is configured for MN subnet, and no gateway is configured for DN subnet, because DN subnet is a closed and isolated network.

Check /etc/maas/rackd.conf to ensure the right address of MAAS is used. In case of any updates to configuration, service maas-rackd has to be restarted.

## 2.3 DNS and NTP
Upstream DNS and NTP services are provided and configured in MAAS. Then MAAS provides DNS service to the POC, so MAAS domain is handled by MAAS DNS and others will be handled by upstream DNS.

Verify DNS on MAAS.
```
host www.google.com
```
DNSSEC needs to be disabled in MAAS, in case the upstream DNS server doesn't support it.

Update /etc/resolv.conf on the host where Juju client runs, set DNS server to MAAS, so Juju can also identify MAAS hostnames.

## 2.4 Proxy Server
An upstream proxy is provided for internet access. Due to a problem in Kafka charm, an intermediate proxy is required. See Appendix B for details.

After enabled the Apache proxy server, set it in MAAS as "Proxy for APT and HTTP/HTTPS".

## 2.5 Configure Physical Server for Running VM
Physical servers are already added in MAAS, with bond0 and bond0.35. To support running VM, the following configuration changes are required.
* Remove bond0.35.
* Remove untagged DN subnet from bond0.
* Create bridge br0 on bond0.
* Add untagged DN subnet on br0 and statically assign an address.
* Add sub-interface br0.35 on br0.
* Add tagged MN subnet on br0.35 and statically assign an address.

Appendix E.1 shows /etc/network/interfaces on physical server after the above updates.

Physical server has the whole disk sda configured and used. Configure disk sdb as LVM volume vg0 to provide volume storage for VM.

Deploy Ubuntu Xenial 16.04 on physical server.

After deployment, login server with SSH key and check the followings.
* Networking, interface, bridge, default route, connections, etc.
* DNS.
* NTP.

## 2.6 Build VM as Controller
Build VM on physical server.
* server1: juju-controller, controller1
* server2: controller2, controller3

1. Install packages.
```
apt-get install lvm2 qemu libvirt-bin virtinst
```

2. Build volume pool.
```
virsh pool-define-as --name lvm --type logical --source-dev /dev/sdb
virsh pool-build lvm
virsh pool-start lvm
virsh pool-autostart lvm
```

3. Create volume for VM.

   On physical server1:
```
virsh vol-create-as lvm juju-controller 200G
virsh vol-create-as lvm controller1 400G
```

   On physical server2:
```
virsh vol-create-as lvm controller2 400G
virsh vol-create-as lvm controller3 400G
```

4. Create VM.

   Create juju-controller and controller1 on server1.
```
virt-install --connect qemu:///system --virt-type kvm \
    --name juju-controller --vcpus 8 --ram 32768 \
    --disk bus=virtio,vol=lvm/juju-controller,cache=none \
    -w bridge=br0,model=virtio \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --pxe --boot network

virt-install --connect qemu:///system --virt-type kvm \
    --name controller1 --vcpus 20 --ram 98304 \
    --disk bus=virtio,vol=lvm/controller1,cache=none \
    -w bridge=br0,model=virtio \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --pxe --boot network
```

   Create controller2 and controller3 on server1.
```
virt-install --connect qemu:///system --virt-type kvm \
    --name controller2 --vcpus 20 --ram 98304 \
    --disk bus=virtio,vol=lvm/controller2,cache=none \
    -w bridge=br0,model=virtio \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --pxe --boot network

virt-install --connect qemu:///system --virt-type kvm \
    --name controller3 --vcpus 20 --ram 98304 \
    --disk bus=virtio,vol=lvm/controller3,cache=none \
    -w bridge=br0,model=virtio \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --pxe --boot network
```

   Check VM.
```
virsh list
```

   VM can also be accessed by VNC.

## 2.7 Add VM into MAAS
Add VMs into MAAS for Juju to use.

1. SSH key

   Get user 'maas' public key /var/lib/maas/.ssh/id_rsa.pub on MAAS server, append it to /home/ubuntu/.ssh/authorized_keys on physical servers.

2. Virsh remote access.

   To verify virsh remote access from MAAS:
```
sudo -i
su maas
virsh -c qemu+ssh://ubuntu@<server>/system list
```
   Ensure /var/lib/maas/.ssh/known_hosts is updated, otherwise it will cause failure.

3. Create machine on MAAS web UI to add VM.

   * Machine name: poc-vm-<virsh domain name>
   * MAC address: <MAC> (virsh dumpxml <name> | grep mac)
   * Power type: Virsh (virtual systems)
   * Power address: qemu+ssh://ubuntu@<server address>/system
   * Power ID: <libvirt instance/domain name>

   After machine is added, it will be in commissioning state. A VNC console can be opened to watch what's happening on VM.

4. Create a tag as the same as server/node name and associate with the server.

5. Configure interface on MAAS web UI.

   * Add untagged DN subnet on the primary interface ens3 and statically assign an address.
   * Create a sub-interface ens3.35 on the primary interface.
   * Add tagged MN subnet on the sub-interface and statically assign an address.

Appendix E.2 shows /etc/network/interfaces on VM after the above configurations.

## 2.8 Compute Node
Physical server server3 is used as compute node. It's already added in MAAS. No need to change any configurations.


## 2.9 Build Contrail Repository

See [Appendix D Contrail repository](#appendix-d-contrail-repository).


# 3 Deploy

## 3.1 Create MAAS cloud in Juju
1. Create this maas-clouds.yaml.

```
clouds:
    maas:
        type: maas
        auth-types: [oauth1]
        endpoint: http://<MAAS MA>/MAAS
```
   MAAS is on both MN and DN. Use the management address here so deployment will be done on MN.

2. Add the defined MAAS cloud.

```
juju add-cloud maas maas-clouds.yaml
```

3. Add credential.

```
juju add-credential maas
```
   When prompted for "maas-oauth", you should paste your MAAS API key. Your API key can be found in the Account/User Preferences page in the MAAS web.

## 3.2 Bootstrap
This is to create the Juju controller for the cloud.

1. Create this config.yaml.

```
no-proxy: localhost,127.0.0.1
apt-http-proxy: http://<MAAS MA>:3129
apt-https-proxy: http://<MAAS MA>:3129
apt-ftp-proxy: http://<MAAS MA>:3129
http-proxy: http://<MAAS MA>:3129
https-proxy: http://<MAAS MA>:3129
ftp-proxy: http://<MAAS MA>:3129
```

2. Bootstrap

```
juju bootstrap --to poc-vm-juju-controller --config config.yaml --model-default config.yaml maas controller
```
   This creates the Juju controller 'controller' for cloud 'maas'.

## 3.3 Deploy
```
juju deploy <bundle configuration>
```
The configuration is in Appendix F Bundle Configuration.

The deployment for such setup takes about one hour to do the followings.
* Deploy Ubuntu Xenial on all controller VMs.
* Deploy Ubuntu Trusty on compute node (physical server).
* Provisioning VMs for deploying containers.
* Allocate container for each service/application.
* Deploy application in container.
* Create relationships.
* Provisioning application.

Note, local Contrail charms are used here, because of a local update to support download repository key by HTTP, so no need to embed the key into bundle configuration.

Here is the patch to <contrail charm>/hooks/charmhelpers/fetch/ubuntu.py.
```
                 key_file.flush()
                 key_file.seek(0)
                 subprocess.check_call(['apt-key', 'add', '-'], stdin=key_file)
+        elif 'http://' in key:
+            with NamedTemporaryFile('w+') as key_file:
+                subprocess.check_call(['wget', key, '-O-'], stdout=key_file)
+                subprocess.check_call(['apt-key', 'add', key_file.name])
         else:
             # Note that hkp: is in no way a secure protocol. Using a
             # GPG key id is pointless from a security POV unless you
```

## 3.4 Re-deploy
```
juju destroy-model default
juju add-model default
juju deploy <bundle configuration>
```

# 4 Credit
Thanks to Robert Ayres at Canonical for helping with this deployment guide!


# Appendix A Tagged and untagged subnet

## A.1 Primary interface only
Use primary interface on the physical server to connect to each subnet.
```
  +---------------------------+
  |     Physical Server       |
  | +-----------------------+ |
  | |          VM           | |
  | | +-------------------+ | |
  | | |    containter     | | |
  | | | eth0         eth1 | | |
  | | +--+------------+---+ | |
  | |    |            |     | |
  | | lxd-br0      lxd-br1  | |
  | |    |            |     | |
  | |   eth0         eth1   | |
  | +----+------------+-----+ |
  |      |            |       |
  |     br0          br1      |
  |      |            |       |
  |     p1s0         p1s1     |
  +------+------------+-------+
         |            |
      Management     Data
       Network      Network
```
As this example, on physical server p1s0 connects to MN and p1s1 connects to DN. Bridge br0 is on eth0 and bridge br1 is on eth1. VM is launched on both bridges. In VM, eth0 connects to MN and eth1 connects to DN. When Juju creates containers in VM, it will create LXD bridges on both primary interfaces.


## A.2 Sub-interface on physical server only
In case only one physical link is provided by physical server, sub-interface is used for server to connect to multiple networks.
```
  +---------------------------+
  |     Physical Server       |
  | +-----------------------+ |
  | |          VM           | |
  | | +-------------------+ | |
  | | |    containter     | | |
  | | | eth0         eth1 | | |
  | | +--+------------+---+ | |
  | |    |            |     | |
  | | lxd-br0      lxd-br1  | |
  | |    |            |     | |
  | |   eth0         eth1   | |
  | +----+------------+-----+ |
  |      |            |       |
  |     br0          br1      |
  |      |            |       |
  |     p1s0         p1s0.5   |
  +------+------------+-------+
         |            |
      Management     Data
       Network      Network
      (untagged)    (VLAN5)
```
As this example, physical server has primary interface p1s0 connet to untagged MN, p1s0.5 connect to VLAN5 DN. Bridge br0 is on p1s0 and bridge br1 is on p1s0.5. VM and container are the same as A.1.

With this option, the DN subnet is VLAN for phsyical server, but untagged subnet for VM. This is not supported by MAAS who manages all subnets for physical server and VM, because MAAS doesn't allow the same subnet to be both untagged and VLAN. It's not allowed to have an untagged subnet and a VLAN subnet to have the same address either.


## A.3 Sub-interface on both physical server and VM

```
  +---------------------------+
  |     Physical Server       |
  | +-----------------------+ |
  | |          VM           | |
  | | +-------------------+ | |
  | | |    containter     | | |
  | | | eth0         eth1 | | |
  | | +--+------------+---+ | |
  | |    |            |     | |
  | | lxd-br0      lxd-br1  | |
  | |    |            |     | |
  | |   eth0         eth0.5 | |
  | +----+------------------+ |
  |      |                    |
  |     br0          br0.5    |
  |      |                    |
  |     p1s0                  |
  +------+--------------------+
         |
         +------------+
         |            |
      Management     Data
       Network      Network
      (untagged)    (VLAN5)
```
As this example, br0 is on p1s0 and br0.5 is a sub-interface of br0. Tagged traffic will be broadcast to both VM interface and sub-interface. This is how tagged traffic can get to VM. In VM, eth0 is for untagged traffic and sub-interface eth0.5 is for tagged traffic. This is supported by MAAS, because DN is tagged VLAN for both physical server and VM. But, MN and DN have to be on separate space in MAAS, so Juju can use constraint space to create LXD bridges on both primary and sub-interface. Without using contraints space, Juju will create bridge on primary interface only.


# Appendix B Proxy
Servers in this environment don't have direct access to public/internet. An upstream proxy is provided for internet access. Normally, this upstream proxy is used directly for deployment.

The challenge here is from Kafka charm (cs:~sdn-charmers/trusty/apache-kafka). During installation, this charm reads index from https://pypi.python.org/simple/ for installing pathlib and jujubigdata packages by pip. It uses urllib that doesn't understand proxy CONNECT commands. so it will issue 'HTTP GET https://<url>' requests at the proxy instead. This requires your proxy (expecting CONNECT requests for https resources generally) to fetch a https url on behalf of the client. This is not supported by the upstream proxy. So an intermediate proxy supporting this requirement between client and upstream proxy needs to be built

Apache web service is already running to support MAAS web UI. Just need to add another virtual site to Apache as the proxy.

1. Create /etc/apache2/sites-available/proxy.conf.
```
<VirtualHost *:3129>
    ServerAdmin webmaster@localhost

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    ProxyRequests On
    SSLProxyEngine On
    ProxyPreserveHost Off
    ProxyRemote * <upstream proxy with port>
    AllowCONNECT 1-65535
    NoProxy <subnet> <subnet> ...
</VirtualHost>
```
   Note, to filter subnet where proxy is not required, Apache needs DNS to be configured properly to resolve name in the request.

2. Enable the new site.
```
sudo a2ensite proxy
```

3. Update /etc/apache2/ports.conf to add a 'Listen 3129' entry.

4. Enable other Apache modules.
```
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_connect
sudo a2enmod ssl
```

5. Restart Apache service.
```
sudo service apache2 restart
```

6. Verify the proxy on MAAS server.
```
# Use CONNECT request.
http_proxy=http://localhost:3129 https_proxy=http://localhost:3129 curl https://www.google.com

# Use HTTP GET https.
http_proxy=http://localhost:3129 https_proxy=http://localhost:3129 python -c 'import urllib; print urllib.urlopen("https://www.google.com").read()'
```
The first command is to verify proxy. The second command is to verify that special requirement is supported.


# Appendix C Underlay Networking
Here are 4 types of traffic.
1. Management

   This is the traffic for management and administration purpose. It includes SSH, SNMP, configuration/server management, deployment, etc.

2. Data

   This is the tunneling traffic, like MPLSoGRE, MPLSoUDP and VXLAN.

3. Control

   This is the traffic from all controlling applications. It includes BGP, Cassandra/MySQL, RabbitMQ, Zookeeper, XMPP, Sandesh, etc.

4. API and UI

   This is for access service API (for example, Keystone, Contrail configuration, etc.) and web UI (OpenStack dashboard and Contrail web UI).

For POC deployment, all traffic can be on a single network.

For Contrail production deployment, it's recommended to run both control and data traffic on one network (one or multiple routable subnets). Management traffic will be on a separate network (not routable with data network).


# Appendix D. Contrail repository

1. Install required packages.
```
sudo apt-get install dpkg-dev dpkg-sig rng-tools
```

2. Unpack Contrail package and build a repository.

   Get package contrail-installer-packages_3.1.2.0-62-mitaka.tgz.
```
mkdir -p /opt/contrail-3.1.2.0-62-mitaka/repo
cd /opt/contrail-3.1.2.0-62-mitaka/repo
tar -xzf contrail-installer-packages_3.1.2.0-62-mitaka.tgz
```

3. Generate GPG key.
```
sudo rngd -r /dev/urandom
sudo cat /proc/sys/kernel/random/entropy_avail
gpg --gen-key
gpg --list-keys
```

4. Export key into repo, sign packages, generate index and release files.
```
cd repo
dpkg-sig --sign builder *.deb

apt-ftparchive packages . > Packages
sed -i 's/Filename: .\//Filename: /g' Packages 
gzip -c Packages > Packages.gz

apt-ftparchive release . > Release
gpg --clearsign -o InRelease Release
gpg -abs -o Release.gpg Release

gpg --output key --armor --export <key ID>
```
<key ID> is from the output of "gpg --list-keys".

5. Add link for the repository.

   Since Apache2 is already running to provide web service, just need to add a link for the repository.
```
cd /var/www/html
ln -s /opt/contrail-3.1.2.0-62-mitaka
```


# Appendix E Network Interface Configuration

## E.1 Physical Server for Controller VM
/etc/network/interfaces
```
auto lo
iface lo inet loopback
    dns-search maas
    dns-nameservers 172.21.2.20

auto eno1
iface eno1 inet manual
    bond-lacp_rate slow
    bond-master bond0
    mtu 1500
    bond-miimon 100
    bond-xmit_hash_policy layer2
    bond-mode active-backup

auto eno2
iface eno2 inet manual
    bond-lacp_rate slow
    bond-master bond0
    mtu 1500
    bond-miimon 100
    bond-xmit_hash_policy layer2
    bond-mode active-backup

auto bond0
iface bond0 inet manual
    bond-lacp_rate slow
    mtu 1500
    hwaddress ether 00:8c:fa:ff:f9:cc
    bond-slaves none
    bond-miimon 100
    bond-xmit_hash_policy layer2
    bond-mode active-backup

auto br0
iface br0 inet static
    address 172.21.3.64/23
    mtu 1500
    hwaddress ether 00:8c:fa:ff:f9:cc
    bridge_fd 15
    bridge_ports bond0

auto br0.35
iface br0.35 inet static
    address 10.75.47.64/25
    gateway 10.75.47.1
    vlan_id 35
    mtu 1500
    vlan-raw-device br0
```

## E.2 Controller VM
/etc/network/interfaces
```
auto lo
iface lo inet loopback
    dns-nameservers 172.21.2.20
    dns-search maas

auto ens3
iface ens3 inet manual
    mtu 1500

auto br-ens3
iface br-ens3 inet static
    address 172.21.3.31/23
    bridge_ports ens3

auto ens3.35
iface ens3.35 inet manual
    vlan-raw-device ens3
    mtu 1500
    vlan_id 35

auto br-ens3.35
iface br-ens3.35 inet static
    address 10.75.47.31/25
    gateway 10.75.47.1
    bridge_ports ens3.35

source /etc/network/interfaces.d/*.cfg
```


# Appendix F Bundle Configuration
```
#
# This bundle configuration deploys services on 3 VM controllers and 2 physical
# compute nodes.
# Controller VM(LXC):  3
# Compute:             1
#
# Each controlling service is deployed in a separate container.
#

machines:
  "0":
    series: trusty
    constraints: tags=poc-vm-controller1
  "1":
    series: trusty
    constraints: tags=poc-vm-controller2
  "2":
    series: trusty
    constraints: tags=poc-vm-controller3
  "3":
    series: trusty
    constraints: tags=server3

series: trusty

services:
  ubuntu:
    charm: cs:trusty/ubuntu
    num_units: 3
    to: [ "0", "1", "2" ]
  ntp:
    charm: cs:trusty/ntp
    options:
      source: 10.75.47.125
    num_units: 0

  # OpenStack
  rabbitmq:
    charm: cs:trusty/rabbitmq-server
    constraints: "spaces=space-0,space-mgmt"
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  mysql:
    charm: cs:trusty/percona-cluster
    constraints: "spaces=space-0,space-mgmt"
    options:
      dataset-size: 15%
      max-connections: 10000
      root-password: password
      sst-password: password
      vip: 10.75.47.110
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  mysql-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0
  keystone:
    charm: cs:~sdn-charmers/trusty/keystone
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      admin-role: admin
      admin-password: admin123
      vip: 10.75.47.111
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  keystone-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0
  glance:
    charm: cs:trusty/glance
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      vip: 10.75.47.112
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  glance-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0
  openstack-dashboard:
    charm: cs:trusty/openstack-dashboard
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      vip: 10.75.47.113
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  openstack-dashboard-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0
  nova-controller:
    charm: cs:trusty/nova-cloud-controller
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      console-access-protocol: novnc
      network-manager: Neutron
      vip: 10.75.47.114
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  nova-controller-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0
  neutron-api:
    charm: cs:trusty/neutron-api
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      manage-neutron-plugin-legacy-mode: false
      vip: 10.75.47.115
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  neutron-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0
  heat:
    charm: cs:trusty/heat
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      vip: 10.75.47.116
      vip_cidr: 25
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  heat-hacluster:
    charm: cs:trusty/hacluster
    options:
      cluster_count: 3
    num_units: 0

  nova-compute:
    charm: cs:trusty/nova-compute
    options:
      openstack-origin: cloud:trusty-mitaka
    num_units: 1
    to: [ "3" ]

  # Contrail
  cassandra:
    charm: cs:trusty/cassandra
    constraints: "spaces=space-0,space-mgmt"
    options:
      authenticator: AllowAllAuthenticator
      heap_newsize: 300M
      install_sources: |
        - deb http://www.apache.org/dist/cassandra/debian 22x main
        - ppa:openjdk-r/ppa
        - ppa:stub/cassandra
      max_heap_size: 3G
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  zookeeper:
    charm: cs:~charmers/trusty/zookeeper
    constraints: "spaces=space-0,space-mgmt"
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  kafka:
    charm: cs:~sdn-charmers/trusty/apache-kafka
    #charm: cs:trusty/kafka
    constraints: "spaces=space-0,space-mgmt"
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  contrail-configuration:
    series: trusty
    charm: ./trusty/contrail-configuration
    constraints: "spaces=space-0,space-mgmt"
    options:
      openstack-origin: cloud:trusty-mitaka
      cassandra-units: 3
      install-sources:
        "deb http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo /"
      install-keys:
        "http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo/key"
      vip: 10.75.47.120
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  contrail-control:
    series: trusty
    charm: ./trusty/contrail-control
    constraints: "spaces=space-0,space-mgmt"
    options:
      install-sources:
        "deb http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo /"
      install-keys:
        "http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo/key"
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  contrail-analytics:
    series: trusty
    charm: ./trusty/contrail-analytics
    constraints: "spaces=space-0,space-mgmt"
    options:
      kafka-units: 3
      cassandra-units: 3
      install-sources:
        "deb http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo /"
      install-keys:
        "http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo/key"
      vip: 10.75.47.120
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  contrail-webui:
    series: trusty
    charm: ./trusty/contrail-webui
    constraints: "spaces=space-0,space-mgmt"
    options:
      install-sources:
        "deb http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo /"
      install-keys:
        "http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo/key"
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  haproxy:
    charm: cs:trusty/haproxy
    constraints: "spaces=space-0,space-mgmt"
    options:
      default_options: dontlognull
      peering_mode: active-active
    num_units: 3
    to: [ "lxd:0", "lxd:1", "lxd:2" ]
  keepalived:
    charm: cs:~sdn-charmers/trusty/keepalived
    options:
      router-id: 1
      virtual-ip: 10.75.47.120
  neutron-contrail-plugin:
    series: trusty
    charm: ./trusty/neutron-api-contrail
    options:
      install-sources:
        "deb http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo /"
      install-keys:
        "http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo/key"
  contrail-vrouter:
    series: trusty
    charm: ./trusty/neutron-contrail
    options:
      install-sources:
        "deb http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo /"
      install-keys:
        "http://10.75.47.125/contrail-3.1.2.0-62-mitaka/repo/key"
  
relations:
  # OpenStack
  - [ ubuntu, ntp ]
  - [ mysql, mysql-hacluster ]
  - [ keystone, keystone-hacluster ]
  - [ glance, glance-hacluster ]
  - [ nova-controller, nova-controller-hacluster ]
  - [ neutron-api, neutron-hacluster ]
  - [ openstack-dashboard, openstack-dashboard-hacluster ]
  - [ keystone, mysql ]
  - [ glance, mysql ]
  - [ glance, keystone ]
  - [ openstack-dashboard, keystone ]
  - [ nova-controller, mysql ]
  - [ nova-controller, rabbitmq ]
  - [ nova-controller, keystone ]
  - [ nova-controller, glance ]
  - [ neutron-api, mysql ]
  - [ neutron-api, rabbitmq ]
  - [ neutron-api, nova-controller ]
  - [ neutron-api, keystone ]
  - [ neutron-api, neutron-contrail-plugin ]
  - [ heat, mysql ]
  - [ heat, keystone ]
  - [ heat, rabbitmq ]
  - [ heat, heat-hacluster ]
  - [ "nova-compute:shared-db", "mysql:shared-db" ]
  - [ "nova-compute:amqp", "rabbitmq:amqp" ]
  - [ nova-compute, ntp ]
  - [ nova-compute, glance ]
  - [ nova-compute, nova-controller ]
  - [ nova-compute, contrail-vrouter ]

  # Contrail
  - [ kafka, zookeeper ]
  - [ "contrail-configuration:cassandra", "cassandra:database" ]
  - [ "contrail-configuration:contrail-analytics-api", "contrail-analytics:contrail-analytics-api" ]
  - [ contrail-configuration, zookeeper ]
  - [ contrail-configuration, rabbitmq ]
  - [ "contrail-configuration:identity-admin", "keystone:identity-admin" ]
  - [ "contrail-configuration:identity-service", "keystone:identity-service" ]
  - [ contrail-configuration, haproxy ]
  - [ "contrail-analytics:cassandra", "cassandra:database" ]
  - [ "contrail-analytics:contrail-api", "contrail-configuration:contrail-api" ]
  - [ "contrail-analytics:contrail-discovery", "contrail-configuration:contrail-discovery" ]
  - [ contrail-analytics, kafka ]
  - [ contrail-analytics, zookeeper ]
  - [ "contrail-analytics:identity-admin", "keystone:identity-admin" ]
  - [ "contrail-analytics:identity-service", "keystone:identity-service" ]
  - [ contrail-analytics, haproxy ]
  - [ "contrail-control:contrail-discovery", "contrail-configuration:contrail-discovery" ]
  - [ "contrail-control:contrail-ifmap", "contrail-configuration:contrail-ifmap" ]
  - [ "contrail-control:contrail-api", "contrail-configuration:contrail-api" ]
  - [ contrail-control, keystone ]
  - [ neutron-contrail-plugin, contrail-configuration ]
  - [ neutron-contrail-plugin, keystone ]
  - [ contrail-webui, keystone ]
  - [ "contrail-webui:contrail_api", "contrail-configuration:contrail-api" ]
  - [ "contrail-webui:contrail_discovery", "contrail-configuration:contrail-discovery" ]
  - [ "contrail-webui:cassandra", "cassandra:database" ]
  - [ contrail-webui, haproxy ]
  - [ contrail-vrouter, keystone ]
  - [ "contrail-vrouter:contrail-discovery", "contrail-configuration:contrail-discovery" ]
  - [ "contrail-vrouter:contrail-api", "contrail-configuration:contrail-api" ]
  - [ haproxy, keepalived ]
```

