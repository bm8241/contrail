## 1 Overview

This guidance shows how to deploy Contrail networking and OpenStack by Juju charms.

Sometimes, it's required to provisioning machine before deploying services on it. For example, enable NTP, add container hooks to support multiple interfaces in container, etc. There are two options for that, base charm and user provisioning.

With MAAS, machine is provided by MAAS, base charm is the only option.

In manual or local environment, machine is added by user,
* with juju-deployer v3 bundle configuration, due to the constraint of service placement, service can't be deployed on machine (in container), base charm is the only option, other services are deployed on base service,
* with juju-deployer v4 bundle configuration, both options are available.


## 2 Local environment

Local environment is for building the minimum setup for basic testing purpose.

[Configure KVM based local environment](https://jujucharms.com/docs/stable/config-KVM)


### 2.1 Setup local environment

* Install Ubuntu 14.04.2 (Trusty) on a physical server. It better to use physical server to avoid nested virtualiztion, because VMs will be launched in local environment.

* Upgrade kernel for the support of nested VMs, otherwise, when Nova launches VM on compute node VM, compute node VM hangs up.
```
sudo apt-get install \
    linux-headers-3.16.0-71 \
    linux-headers-3.16.0-71-generic \
    linux-image-3.16.0-71-generic \
    linux-image-extra-3.16.0-71-generic
```

* Update /etc/hosts with hostname and IP address for resolvable hostname.

* Ensure apt source is properly configured to access Ubuntu repositories.

* Install software-properties-common and python-yaml if they are not installed as part of Ubuntu.
```
sudo apt-get install software-properties-common python-yaml
```

* Add Juju repository and update apt sources.
```
sudo apt-add-repository ppa:juju/stable
sudo apt-get update
```

* Install packages for Juju local enironment.
```
sudo apt-get install juju-local qemu-kvm libvirt-bin bridge-utils \
    virt-manager qemu-system uvtool-libvirt uvtool
```

* Since all containers have static IP, reset DHCP allocation range to reserve space for containers.
```
virsh net-edit default
# set range to 192.168.122.2 - 192.168.122.19, save and quit editor.
virsh net-destroy default
virsh net-start default
service libvirt-bin restart
```

Check virbr0 status by ifconfig.
```
ifconfig virbr0
virbr0    Link encap:Ethernet  HWaddr 52:54:00:80:1d:62  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8068 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9102 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:495608 (495.6 KB)  TX bytes:73811715 (73.8 MB)
```

* Local environment can't be created by root user, a non-root user needs to be created.
```
sudo adduser juju
sudo usermod -G sudo,libvirtd juju
```


### 2.2 Bootstrap local environment

* Create environment configuration. This will create ~/.juju/environment.yaml.
```
juju generate-config
```

* Replace environment.yaml with the followings.
```
default: local

environments:
    local:
        type: local
        container: kvm
```

* Bootstrap local environment. This creates bootstrap server that is the machine "0" in the environment.
```
juju bootstrap
```


### 2.3 Add machines

For the minimum deployment, two machines will be added. One is the controller running OpenStack and Contrail controlling services. Another is the compute running OpenStack Nova compute and Contrail vrouter. The controller needs at least 40GB disk space, 4 CPU cores and 32GB memory. On the controller, container file systems of 14 services/containers and 1 template (/var/lib/lxc) take about 16GB disk space for installation.
```
juju add-machine --constraints "root-disk=40G cpu-cores=4 mem=32G" \
    --series trusty
juju add-machine --constraints "root-disk=10G cpu-cores=2 mem=8G" \
    --series trusty
```

VMs are created on bridge virbr0. DHCP server is enabled on virbr0. SNAT is configured by iptables between virbr0 and physical interface. All VMs and containers in VMs will get IP address from DHCP server and access external.

Configure container bridge on the controller. This is not required for compute.
* Login to the controller.
```
juju ssh 1
```

* Apply the followings to /etc/network/interfaces.d/eth0.cfg.
```
auto eth0
iface eth0 manual

auto lxcbr0
iface lxcbr0 inet dhcp
  bridge_ports eth0
```

* Restart the interface.
```
ifdown eth0 && ifup eth0 lxcbr0
```

NTP is required by Contrail services. It has to be installed on two machines.
```
sudo apt-get install ntp
```

Machine provisioning can be done by either user before adding machine, or base charm.


### 2.4 Deploy

Instead of manually deploy services and set relations, it's recommended to use the bundle deployment tool juju-deployer to deploy the setup based on the bundle configuration. Install juju-deployer if it's not there.
```
sudo apt-get install juju-deployer
```

Bundle configuration [contrail-2n-lxc.yaml](https://github.com/tonyliu0592/opencontrail-install/blob/master/juju/contrail-2n-lxc.yaml) is for deployment on two machines provisioned by user in local or manual environment.
```
juju-deployer -c contrail-2n-lxc.yaml
```

Service colocation (multiple services are deployed on one machine) is supported for Contrail. Bundle configuration [contrail-3n-group.yaml](https://github.com/tonyliu0592/opencontrail-install/blob/master/juju/contrail-3n-group.yaml) is an example of colocation deployment.


### 2.5 Access local setup

In local environment, VMs are on bridge virbr0 holding the private network. All services in container are also on the same private network. SNAT is configured on the host between virbr0 and external interface, so VMs and containers have external access.

To access the services in container, SSH port forwarding is required. The following tunnels are required.
* <host address>:80:<openstack-dashboard address>:80
* <host address>:443:<openstack-dashboard address>:443
* <host address>:8080:<contrail-webui address>:8080
* <host address>:8143:<contrail-webui address>:8143

```
sudo ssh -N -f \
    -L 10.87.64.140:80:192.168.122.64:80 \
    -L 10.87.64.140:443:192.168.122.64:443 \
    -L 10.87.64.140:8080:192.168.122.132:8080 \
    -L 10.87.64.140:8143:192.168.122.132:8143 \
    root@localhost
```


## 3 Manual environment

Manual environment is for building the setup where machines are manually added.


### 3.1 Setup manual environment

In the manual environment, Juju client, bootstrap server and target machines can be separated.

* Install Juju packages.
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:juju/stable
sudo apt-get install juju-core
```

* Configure Juju to manual mode.
```
juju generate-config
juju switch manual
```

* Update `bootstrap-host` in section `manual` in ~/.juju/environments.yaml. In this case, it's the server IP address where bootstrap runs.

Note, multi-interface is not supported by Juju. It can't be configured to use specific interface for deployment. Ensure single interface up before starting bootstrap of Juju server. Other interfaces can be re-enabled after bootstrap.

* Launch bootstrap on bootstrap host.
```
juju bootstrap
```


### 3.2 Target machine
The following steps are for preparing servers. In case of using MAAS, these are not required.

Ensure networking, NTP and resolvable hostname are all configured.

In manual mode, some additional steps are required on target machine.

* Install additional packages.
```
sudo apt-get install software-properties-common python-yaml
```

* Install LXC.
As stated in Co-location Support in [Provider Colocation Support](https://wiki.ubuntu.com/ServerTeam/OpenStackCharms/ProviderColocationSupport), it's a general rule to deploy charms in separate containers/machines.
```
sudo apt-get install lxc
```
Note, LXC is not required on compute node.

* Configure LXC bridge.
Update `/etc/network/interfaces`. Here is an example.
```
auto p1p1
iface p1p1 inet manual

auto lxcbr0
iface lxcbr0 inet static
    address 10.84.14.47
    netmask 255.255.255.0
    gateway 10.84.14.254
    dns-nameservers 10.84.5.100
    dns-search juniper.net
    bridge_ports p1p1
```


### 3.3 Add target machine
* Add machines.
```
juju add-machine ssh:10.84.14.47
juju add-machine ssh:10.84.14.48
```
If machine is added by hostname, the hostname has to be resovlable on DNS server.


### 3.4 Juju GUI
* Add Juju GUI onto Juju server.
```
juju deploy juju-gui --to 0
juju expose juju-gui
```
Wait a few minutes until the GUI server is up. User name and password are in ~/.juju/environments/manual.jenv ('username' and 'password').


## 3.5 Deploy
Here is a bundle configuration to deploy Contrail and OpenStack on 5 servers with HA. [contrail-5n-lxc.yaml](https://github.com/tonyliu0592/opencontrail-install/blob/master/juju/contrail-5n-lxc.yaml)
```
juju-deployer -c contrail-5n-lxc.yaml
```


## 4 High Aavailability and Load Balancing

Contrail requires L3 connectivity to underlay networking, no L2 dependency.

OpenStack services deployed by Juju have to use ha-cluster (Corosync and Pacemaker), because it's coded in OpenStack charms. Ha-cluster runs as subordinate service along with each OpenStack service to provide clustering service. Instances of the cluster can use multicast (on the same L2) or unicast (on separate L2) to commumicate between each other. Ha-cluster provides active-standby and active-active modes.

In case of active-standby mode, the VIP is present on one instance. All requests go to that instance. In case active instance goes down, the VIP will be present on one of other living instances. This mode provides redundancy, but no load balancing. It requires L2 to provide VIP.

In case of active-active mode, the MAC of VIP is multicast MAC, so the switch will send requests to all instances. On each instance, IP table rule is used to select which request is handled by this instance. For example, the source address of request is hashed. Each instance handles a range of hash value. This mode provides both redundancy and very limited load balancing. This [link](http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/_clone_the_ip_address.html) talks about active-active mode. It requires L2 to provide VIP.

OpenStack services are not clustering service where members of the cluster need to communicate between each other to exchange data, eg. in database cluster, members replicate data between each other. All required for HA and LB is to have multiple services providing redundancy and a pair of load-balancer in front of them to balance traffic. To achieve that, for now, external load-balancers (software or hardware) have to be deployed and manually configured. The VIP will be present on load-balancers. VRRP or equivalent technology can be used in case load-balancers are on the same L2. In case load-balancers are on separate L2, routing protocol service, like BIRD, can be deployed on each load-balancer to advertise VIP route to gateway. Then gateway will anycast the request to one of the next-hops of VIP route.


## Appendix A. Contrail repository

Note, this is only required for the deployment with Juniper Contrail package. In case of using Launchpad PPA, this is not needed.

* Install required packages.
```
sudo apt-get install dpkg-dev dpkg-sig rng-tools
```

* Unpack Contrail package and build a repository.
```
sudo dpkg -i contrail-install-packages_3.0.2.0-32~liberty_all.deb
mkdir -p /opt/contrail-3.0.2.0-32-liberty/repo
cd /opt/contrail-3.0.2.0-32-liberty
tar -C repo -xzf contrail_packages/contrail_debs.tgz
```

* Generate GPG key.
```
sudo rngd -r /dev/urandom
sudo cat /proc/sys/kernel/random/entropy_avail
gpg --gen-key
gpg --list-keys
```

* Export key into repo, sign packages, generate index and release files.
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

* Install HTTP server.
```
sudo apt-get install mini-httpd
```
Update /etc/default/mini-httpd to enable the start.
Update /etc/mini-httpd.conf to set host and root directory.

* Set apt source.
The following steps are done by charm, no need to do them manually. Here just shows how source and key are used. On target machine, download GPG key and update apt source list.
```
wget http://<server IP>/contrail/repo/key
apt-key add key
```
Update /etc/apt/sources.list with this line.
```
deb http://<host IP>/contrail/repo /
```

* Configure installation source and key.
Installation source and key have to be configured in configuration file. Here is an example in bundle configuration.
```
options:
  install-sources:
    "deb http://10.84.29.100/contrail-3.0.2.0-32-liberty/repo /"
  install-keys:
    "http://10.84.29.100/contrail-3.0.2.0-32-liberty/repo/key"
```


## Appendix B. Container configuration and hook

After installing lxc package, /usr/share/lxc and /var/lib/lxc are created. To add user defined hooks to provisioning container, create /usr/share/lxc/config/ubuntu-cloud.trusty.conf as the following.
```
lxc.hook.clone = /usr/share/lxc/hooks/user-hook
```

Then create /usr/share/lxc/hooks/user-hook.

A container template will be created before creating the first container, for example, /var/lib/lxc/juju-trusty-lxc-template. The configuration ubuntu-cloud.trusty.conf will show up in juju-trusty-lxc-template/config, like this.
```
lxc.include = /usr/share/lxc/config/ubuntu-cloud.trusty.conf
```

When creating the container, container template is cloned, then provisioning hooks will be invoked.

In case it's failed to bring up container with user hook, you can login to the target machine to manually clone a container to see the errors.
```
lxc-clone -o juju-trusty-lxc-template -n juju-local-machine-1-lxc-3
```

Here is an example of user hook to configure static IP address for container based on container ID, and make hostname resolvable.
```
prefix=192.168.122
base=20
netmask=255.255.255.0
gateway=192.168.122.1

idx=${LXC_NAME##j*-}
address=$prefix.$(($base + $idx))

cat << __EOF__ > "$LXC_ROOTFS_PATH/etc/network/interfaces"
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address $address
    netmask $netmask
    gateway $gateway

dns-nameservers 10.84.5.100
dns-search juniper.net
__EOF__

echo $address  $LXC_NAME >> "$LXC_ROOTFS_PATH/etc/hosts"
```


## Appendix C. Fetch charms
* Download required Juju charms on Juju server.
```
sudo apt-get install bzr
mkdir -p charms/trusty
bzr branch lp:~sdn-charmers/charms/trusty/contrail-analytics/trunk \
    charms/trusty/contrail-analytics
bzr branch lp:~sdn-charmers/charms/trusty/contrail-configuration/trunk \
    charms/trusty/contrail-configuration
bzr branch lp:~sdn-charmers/charms/trusty/contrail-control/trunk \
    charms/trusty/contrail-control
bzr branch lp:~sdn-charmers/charms/trusty/contrail-webui/trunk \
    charms/trusty/contrail-webui
bzr branch lp:~sdn-charmers/charms/trusty/neutron-contrail/trunk \
    charms/trusty/neutron-contrail
bzr branch lp:~sdn-charmers/charms/trusty/neutron-api-contrail/trunk \
    charms/trusty/neutron-api-contrail
export JUJU_REPOSITORY=charms
```

