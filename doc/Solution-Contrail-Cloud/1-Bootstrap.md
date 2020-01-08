* [TOC](Contrail-Cloud.md#toc)

# 1 Bootstrap

Bootstrap is a Contrail Networking cluster deployed on single host (physical server or virtual machine) without OpenStack (not needed) and Contrail Command (failed to import such cluster due to some issues).

In this guide, bootstrap is deployed with Contrail 1912.32 GA on a VM. The VM is based on `CentOS-7-x86_64-GenericCloud-1805.qcow2` cloud image, with 4 vCPU, 32GB memory and 60GB disk.


## 1.1 Deploy

Here are the prerequisites to deploy bootstrap.
* L2 access to all devices by a network interface connecting to management/OOB network where management inteface of all devices are connected to.
* Internet access.
* NTP server.
* Hostname.
* `/etc/hosts`

Deployement is done by the script [bootstrap](https://github.com/tonyliu0592/contrail-poc/blob/master/cluster/bootstrap/bootstrap).

Copy the script and run it.
```
./bootstrap
```

When it's done, Contrail Networking is deployed.


## 1.2 Fabric underlay

The fabric is discovered with ZTP as a greenfield. Fabric underlay will be built when fabric is created.

Here are steps to build a fabric with 1 spine and 4 leaves (2 per rack).
#### Zeroize devices.
```
request system zeroize
```

#### Get configuration tool.
```
yum install git
git clone https://github.com/tonyliu0592/contrail-toolbox.git
cd contrail-toolbox/config
./copy-vnc-lib
# Update parameters in admin.env.
vi admin.env
source admin.env
# Validate.
./config show virtual-network
```

#### Check devices.
* All are up and running in zero mode.
* LLDP is enabled on all interfaces and all neighbors are discovered.


#### Create fabric.
Create resource file [job-create-fabric.yaml](A1-Resource-file.md#a11-job-create-fabricyaml).

Run the job.
```
./config set --file job-create-fabric.yaml
```

This job discovers all devices from DHCP leasing. It also discovers underlay links by importing LLDP neighbor info from each device. Corresponding resource is created in Contrail, like fabric, physical-router, etc. Day-0 configuration is pushed onto each device, no configuration pushed by Contrail yet.

#### Assign role.
Create resource file [job-assign-role.yaml](A1-Resource-file.md#a12-job-assign-roleyaml).

Run the job.
```
./config set --file job-create-fabric.yaml
```

This job assigns role for each device, updates configuration in Contrail, creates bgp-router and pushes configuration to each device.


