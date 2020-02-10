# Integrate Contrail 2.0-22 with RHEL OSP 5

## 1 Overview

This guide is for integrating OpenContrail 2.0 with RHEL OSP 5 (OpenStack IceHouse on RHEL 7.0).

In this example, three VMs are required.

1. OpenStack controller

   See Appendix A for the list of installed services and their status.

2. OpenContrail controller

   It has OpenContrail configuration, analytics, database and control services.

3. Compute node

   It has OpenStack Nova compute and OpenContrail vRouter services.

## 2 Create RHEL 7.0 VMs

### 2.1 Build RHEL 7.0 VM image

* Download rhel-guest-image-7.0-20140930.0.x86_64.qcow2 from RedHat with subscription.

* Hardcode root password into the image, so it can be launched by libvirt.
```
# modprobe nbd
# qemu-nbd -c /dev/nbd0 <full QCOW2 image name>
# mkdir -p /mnt/image
# mount /dev/nbd0p1 /mnt/image

## Generate encrypted password.
# openssl passwd -1

## Edit /mnt/image/etc/shadow apply encrypted password to root.

# umount /mnt/image
# qemu-nbd -d /dev/nbd0
```

* Launch RHEL 7 VM. An example of XML file is in Appendix B.
```
# virsh create <XML file>
```

### 2.2 Provisioning RHEL 7 VM

* Login VM from console.
```
# virsh console <VM name>
```

* Configure the following items.
  * Network interface, IP address, gateway, DNS, etc.
  * Host name in `/etc/hostname`
  * Resovable host name (in `/etc/hosts` or DNS), ensure `ping $(hostname)` work. In case of separate management and control/data networks with different interfaces, a hostname on control/data network is required for control/data address.
  * NTP
  * Disable SELinux in `/etc/selinux/config`.
  * Enable `PermitRootLogin` in `/etc/ssh/sshd_config`.
  * Enable either `PasswordAuthentication` or `PubkeyAuthentication` in `/etc/ssh/sshd_config`.
  * Disable `iptables`, `ip6tables` and `firewalld`.

* Reboot.

* Login VM by SSH.

* Register subscription and repos.
```
# subscription-manager register --username <username> --password <password>
# subscription-manager list --available
## Get pool ID.
# subscription-manager attach --pool=<pool ID>
# subscription-manager repos --enable=rhel-7-server-extras-rpms
# subscription-manager repos --enable=rhel-7-server-optional-rpms
# subscription-manager repos --enable=rhel-7-server-rpms
# subscription-manager repos --enable=rhel-7-server-openstack-5.0-rpms
```

* Install `wget` package.
```
# yum install -y wget
```

* Register EPEL repo.
```
# wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
# rpm -ivh epel-release-7-5.noarch.rpm
```

## 3 OpenStack controller

### 3.1 Install OpenStack

* Install `openstack-packstack` and install OpenStack.
```
# yum install -y openstack-packstack
# packstack --allinone --mariadb-pw=juniper123 --use-epel=n --nagios-install=n
```

### 3.2 Provisioning OpenStack

* Set password for user `admin`.
```
# source keystonerc_admin
# keystone user-password-update --pass <password> admin
```

* Update `OS_PASSWORD` in `keystonerc_admin` with newly created password.

* Stop and disable Neutron services.
```
# systemctl stop neutron-server
# systemctl disable neutron-server
# systemctl stop neutron-dhcp-agent
# systemctl disable neutron-dhcp-agent
# systemctl stop neutron-l3-agent
# systemctl disable neutron-l3-agent
# systemctl stop neutron-metadata-agent
# systemctl disable neutron-metadata-agent
# systemctl stop neutron-openvswitch-agent
# systemctl disable neutron-openvswitch-agent
```

* Disable Nova compute service.
```
# nova service-disable <hostname> nova-compute
# systemctl stop openstack-nova-compute
# systemctl disable openstack-nova-compute
```

* Set Nova parameters.
```
# openstack-config --set /etc/nova/nova.conf DEFAULT network_api_class nova.network.neutronv2.api.API
# openstack-config --set /etc/nova/nova.conf DEFAULT neutron_url http://<OpenContrail controller IP>:9696
# openstack-config --set /etc/nova/nova.conf DEFAULT neutron_admin_auth_url http://<OpenStack controller IP>:35357/v2.0
# openstack-config --set /etc/nova/nova.conf DEFAULT  compute_driver nova.virt.libvirt.LibvirtDriver
# openstack-config --set /etc/nova/nova.conf DEFAULT  novncproxy_port 5999
```

* Restart Nova services.
```
# systemctl restart openstack-nova-api restart
# systemctl restart openstack-nova-conductor restart
# systemctl restart openstack-nova-scheduler restart
# systemctl restart openstack-nova-novncproxy restart
# systemctl restart openstack-nova-consoleauth restart
```

## 4 OpenContrail controller

Contrail installation and provisioning are done by Fab utility on the builder (one server in the cluster or a separate server). All `fab` command has to be executed in `/opt/contrail/utils` directory.

### 4.1 Install OpenContrail

* Copy `contrail-install-packages-2.0-22~icehouse.el7.noarch.rpm` onto VM and install it.
```
# rpm -ivh contrail-install-packages-2.0-22~icehouse.el7.noarch.rpm
```

* Patch `/opt/contrail/contrail_packages/setup.sh`, and run it.
```
# cd /opt/contrail/contrail_packages
# sed -i -e 's/pip-python/pip/g' setup.sh
# ./setup.sh
```

* Create `/opt/contrail/utils/fabfile/testbeds/testbed.py`. An example is in Appendix C.
In case that compute nodes are installed separately, don't put compute nodes in `testbed.py` for now.

* Install Contrail installation package.
This step copies Contrail installation package to all non-OpenStack nodes and install it.
```
# fab install_pkg_all_without_openstack:<package file>
```

* Patch `/opt/contrail/contrail_packages/setup.sh` on all other non-OpenStack nodes.

* Install Contrail service packages.
This step creates local repo of Contrail packages and install them on all non-OpenStack nodes.
```
# cd /opt/contrail/utils
# fab install_without_openstack
```

* Enable SSLv3 in Java.
Due to a recent Java security update, SSLv3 is disabled by default. But some Contrail services still use that to connect to the IF-MAP server. So this patch is required to enable SSLv3 again.
```
# cd /usr/lib/jvm/java-1.7.0-openjdk-1.7.0.75-2.5.4.2.el7_0.x86_64/jre/lib/security
# sed -i -e 's/jdk.tls.disabledAlgorithms=SSLv3/#jdk.tls.disabledAlgorithms=SSLv3/g' java.security
```

* Disable SELinux.
  * Edit `/etc/selinux/config` and set `SELINUX` to `disabled`. This change is permanent but requires reboot.
  * Run command `setenforce 0`. This change is transit but takes effect immediately.

* On compute nodes, copy vRouter kernel module to the right place.
This step is not required is compute nodes are installed separately.
```
# cp -r /lib/modules/3.10.0-123.el7.x86_64/extra/net /lib/modules/$(uname -r)/extra
```

### 4.2 Provisioning Contrail.

```
# cd /opt/contrail/utils
# fab setup_without_openstack
```


## Appendix A
```
== Nova services ==
openstack-nova-api:                     active
openstack-nova-cert:                    active
openstack-nova-compute:                 inactive
openstack-nova-network:                 inactive  (disabled on boot)
openstack-nova-scheduler:               active
openstack-nova-volume:                  inactive  (disabled on boot)
openstack-nova-conductor:               active
== Glance services ==
openstack-glance-api:                   active
openstack-glance-registry:              active
== Keystone service ==
openstack-keystone:                     active
== Horizon service ==
openstack-dashboard:                    active
== neutron services ==
neutron-server:                         inactive
neutron-dhcp-agent:                     active
neutron-l3-agent:                       active
neutron-metadata-agent:                 active
neutron-lbaas-agent:                    inactive  (disabled on boot)
neutron-openvswitch-agent:              active
neutron-linuxbridge-agent:              inactive  (disabled on boot)
neutron-ryu-agent:                      inactive  (disabled on boot)
neutron-nec-agent:                      inactive  (disabled on boot)
neutron-mlnx-agent:                     inactive  (disabled on boot)
neutron-metering-agent:                 inactive  (disabled on boot)
== Swift services ==
openstack-swift-proxy:                  active
openstack-swift-account:                active
openstack-swift-container:              active
openstack-swift-object:                 active
== Cinder services ==
openstack-cinder-api:                   active
openstack-cinder-scheduler:             active
openstack-cinder-volume:                active
openstack-cinder-backup:                active
== Ceilometer services ==
openstack-ceilometer-api:               active
openstack-ceilometer-central:           active
openstack-ceilometer-compute:           active
openstack-ceilometer-collector:         active
openstack-ceilometer-alarm-notifier:    active
openstack-ceilometer-alarm-evaluator:   active
openstack-ceilometer-notification:      active
== Support services ==
libvirtd:                               active
openvswitch:                            active
dbus:                                   active
tgtd:                                   inactive  (disabled on boot)
rabbitmq-server:                        active
memcached:                              active
```

## Appendix B
```
<domain type='kvm'>
  <name>vm135</name>
  <memory unit='KiB'>33554432</memory>
  <cpu mode='custom' match='exact'>
    <model fallback='allow'>Westmere</model>
    <vendor>Intel</vendor>
    <feature policy='require' name='vmx'/>
  </cpu>
  <currentMemory unit='KiB'>33554432</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <os>
    <type arch='x86_64'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <pae/>
  </features>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' cache='none'/>
      <source file='/var/cloud/libvirt/images/rhel-7-vm135.qcow2'/>
      <target dev='vda' bus='virtio'/>
    </disk>
    <interface type='bridge'>
      <mac address='52:54:00:36:97:e0'/>
      <source bridge='br-mgmt'/>
      <model type='virtio'/>
    </interface>
    <interface type='bridge'>
      <source bridge='br-data'/>
      <model type='virtio'/>
    </interface>
    <serial type='pty'>
      <target port='0'/>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0'>
      <listen type='address' address='0.0.0.0'/>
    </graphics>
    <video>
      <model type='cirrus' vram='9216' heads='1'/>
    </video>
    <memballoon model='virtio'>
    </memballoon>
  </devices>
</domain>
```

## Appendix C
```
from fabric.api import env

#Management ip addresses of hosts in the cluster
host1 = 'root@10.161.208.134'
host2 = 'root@10.161.208.135'

#External routers if any
#for eg. 
#ext_routers = [('mx1', '10.204.216.253')]
ext_routers = []

#Autonomous system number
router_asn = 64512

#Host from which the fab commands are triggered to install and provision
host_build = 'root@10.161.208.135'


#Role definition of the hosts.
env.roledefs = {
    'all': [host1, host2],
    'cfgm': [host2],
    'openstack': [host1],
    'control': [host2],
    'compute': [host2],
    'collector': [host2],
    'webui': [host2],
    'database': [host2],
    'build': [host_build],
    'storage-master': [host2],
    'storage-compute': [host2],
    # 'vgw': [host1], # Optional, Only to enable VGW. Only compute can support vgw
 #   'backup':[backup_node],  # only if the backup_node is defined
}

#Openstack admin password
env.openstack_admin_password = 'contrail123'

#Hostnames
env.hostnames = {
    'all': ['vm134', 'vm135']
}

env.password = 'c0ntrail123'
#Passwords of each host
env.passwords = {
    host1: 'c0ntrail123',
    host2: 'c0ntrail123',
  #  backup_node: 'secret',
    host_build: 'c0ntrail123',
}

#For reimage purpose
env.ostypes = {
    host1:'centos',
}


# INFORMATION FOR DB BACKUP/RESTORE ..
#=======================================================
#Optional,Backup Host configuration if it is not available then it will put in localhost
#backup_node = 'root@2.2.2.2'

# Optional, Local/Remote location of backup_data path 
# if it is not passed it will use default path 
#backup_db_path= ['/home/','/root/']
#cassandra backup can be defined either "full" or "custom"  
#full -> take complete snapshot of cassandra DB 
#custom -> take snapshot except defined in skip_keyspace 
#cassandra_backup='custom'  [ MUST OPTION] 
#skip_keyspace=["ContrailAnalytics"]  IF cassandra_backup is selected as custom
#service token need to define to do  restore of  backup data
#service_token = '53468cf7552bbdc3b94f'


#OPTIONAL ANALYTICS CONFIGURATION
#================================
# database_dir is the directory where cassandra data is stored
#
# If it is not passed, we will use cassandra's default
# /var/lib/cassandra/data
#
#database_dir = '<separate-partition>/cassandra'
#
# analytics_data_dir is the directory where cassandra data for analytics
# is stored. This is used to seperate cassandra's main data storage [internal
# use and config data] with analytics data. That way critical cassandra's 
# system data and config data are not overrun by analytis data
#
# If it is not passed, we will use cassandra's default
# /var/lib/cassandra/data
#
#analytics_data_dir = '<separate-partition>/analytics_data'
#
# ssd_data_dir is the directory where cassandra can store fast retrievable
# temporary files (commit_logs). Giving cassandra an ssd disk for this
# purpose improves cassandra performance
#
# If it is not passed, we will use cassandra's default
# /var/lib/cassandra/commit_logs
#
#ssd_data_dir = '<seperate-partition>/commit_logs_data'

#OPTIONAL BONDING CONFIGURATION
#==============================
#Inferface Bonding
#bond= {
#    host1 : { 'name': 'bond0', 'member': ['p2p0p0','p2p0p1','p2p0p2','p2p0p3'], 'mode': '802.3ad', 'xmit_hash_policy': 'layer3+4' },
#}

#OPTIONAL SEPARATION OF MANAGEMENT AND CONTROL + DATA and OPTIONAL VLAN INFORMATION
#==================================================================================
#control_data = {
#    host1 : { 'ip': '192.168.10.1/24', 'gw' : '192.168.10.254', 'device': 'bond0', 'vlan': '224' },
#}

#OPTIONAL STATIC ROUTE CONFIGURATION
#===================================
#static_route  = {
#    host1 : [{ 'ip': '10.1.1.0', 'netmask' : '255.255.255.0', 'gw':'192.168.10.254', 'intf': 'bond0' },
#             { 'ip': '10.1.2.0', 'netmask' : '255.255.255.0', 'gw':'192.168.10.254', 'intf': 'bond0' }],
#}

#storage compute disk config
#storage_node_config = {
#    host1 : { 'disks' : ['sdc', 'sdd'] },
#}

#live migration config
#live_migration = True


#To disable installing contrail interface rename package
env.interface_rename = False

#In environments where keystone is deployed outside of Contrail provisioning
#scripts , you can use the below options 
#
# Note : 
# "insecure" is applicable only when protocol is https
# The entries in env.keystone overrides the below options which used 
# to be supported earlier :
#  service_token
#  keystone_ip
#  keystone_admin_user
#  keystone_admin_password
#  region_name
#
env.keystone = {
    'keystone_ip'   : '10.161.208.134',
    'auth_protocol' : 'http',                  #Default is http
    'auth_port'     : '35357',                 #Default is 35357
    'admin_token'   : 'aa0f54bf01884671a758f146a9b2c5be',  #admin_token in keystone.conf
    'admin_user'    : 'admin',                 #Default is admin
    'admin_password': 'contrail123',           #Default is contrail123
    'service_tenant': 'service',               #Default is service
    'admin_tenant'  : 'admin',                 #Default is admin
    'region_name'   : 'RegionOne',             #Default is RegionOne
    'insecure'      : 'True',                  #Default = False
    'manage_neutron': 'no',                    #Default = 'yes' , Does configure neutron user/role in keystone required.
}


# In High Availability setups.
#env.ha = {
#    'internal_vip'   : '1.1.1.1',               #Internal Virtual IP of the HA setup.
#    'external_vip'   : '2.2.2.2',               #External Virtual IP of the HA setup.
#    'nfs_server'      : '3.3.3.3',               #IP address of the NFS Server which will be mounted to /var/lib/glance/images of openstack Node, Defaults to env.roledefs['compute'][0]
#    'nfs_glance_path' : '/var/tmp/images/',      #NFS Server path to save images, Defaults to /var/tmp/glance-images/
#}

# In environments where openstack services are deployed independently 
# from contrail, you can use the below options 
# service_token : Common service token for for all services like nova,
#                 neutron, glance, cinder etc
# amqp_host     : IP of AMQP Server to be used in openstack
# manage_amqp   : Default = 'no', if set to 'yes' provision's amqp in openstack nodes and
#                 openstack services uses the amqp in openstack nodes instead of config nodes.
#                 amqp_host is neglected if manage_amqp is set
#
env.openstack = {
    'service_token' : 'aa0f54bf01884671a758f146a9b2c5be', #Common service token for for all openstack services
    'amqp_host' : '10.161.208.134',            #IP of AMQP Server to be used in openstack
    'manage_amqp' : 'no',                    #Default no, Manage seperate AMQP for openstack services in openstack nodes.
}

# Neutron specific configuration 
#env.neutron = {
#   'protocol': 'http', # Default is http
#}

#To enable multi-tenancy feature
multi_tenancy = False

#To enable haproxy feature
#haproxy = True

#To Enable prallel execution of task in multiple nodes
#do_parallel = True

# To configure the encapsulation priority. Default: MPLSoGRE 
#env.encap_priority =  "'MPLSoUDP','MPLSoGRE','VXLAN'"

# Optional proxy settings.
# env.http_proxy = os.environ.get('http_proxy')

#To enable LBaaS feature
# Default Value: False
#env.enable_lbaas = True

#OPTIONAL REMOTE SYSLOG CONFIGURATION
#===================================
#For R1.10 this needs to be specified to enable rsyslog.
#For Later releases this would be enabled as part of provisioning,
#with following default values.
#
#port = 19876
#protocol = tcp
#collector = dynamic i.e. rsyslog clients will connect to servers in a round
#                         robin fasion. For static collector all clients will
#                         connect to a single collector. static - is a test
#                         only option.
#status = enable
#
#env.rsyslog_params = {'port':19876, 'proto':'tcp', 'collector':'dynamic', 'status':'enable'}

#OPTIONAL Virtual gateway CONFIGURATION
#=======================================

#Section vgw is only relevant when you want to use virtual gateway feature. 
#You can use one of your compute node as  gateway .

#Definition for the Key used
#-------------------------------------
#vn: Virtual Network fully qualified name. This particular VN will be used by VGW.
#ipam-subnets: Subnets used by vn. It can be single or multiple
#gateway-routes: If any route is present then only those routes will be published
#by VGW or Default route (0.0.0.0) will be published

#env.vgw = {host1: {'vgw1':{'vn':'default-domain:admin:public:public', 'ipam-subnets': ['10.204.220.128/29', '10.204.220.136/29', 'gateway-routes': ['8.8.8.0/24', '1.1.1.0/24']}]},
#                   'vgw2':{'vn':'default-domain:admin:public1:public1', 'ipam-subnets': ['10.204.220.144/29']}
#          }
```

