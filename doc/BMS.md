Manage Bare Metal Server with Contrail 5

# 1 Overview

Contrail provides connectivity for virtual machine and BMS on overlay, and connectivity between overlay and underlay.

This paper shows how to use Contrail Command to manage BMS.


# 2 Contrail Fabric

## 2.1 Fabric topology

The underlay fabric is the typical IP CLOS fabric. Here is an example.


## 2.2 Import fabric

Go to page `INFRASTRUCTURE > Fabrics`, click `Create`.

There are two options, 1) creating a new fabric, 2) importing an existing fabric. This example is to import existing fabric. Select `Existing Fabric` and click `Provision`.

On step 1 `Create Fabric`, do the followings and click `Next`.
* Set fabric name.
* Set device credential.
* Set CIDR of management subnet.

On step 2 `Device discovery`, Contrail walks through each address in management subnet, discovers supported devices. When this process is done, click `Next`.

On step 3 `Assign the roles`, click `Assign` icon at the right end of each device line. For leaf, assign `leaf` as the `Physical Role` and `CRB-Access` as the the `Routing Bridgeing Roles`. For spine, assign `spine` and `DC-Gateway` respectively. In this example, the spine is used as DC-GW. Click `Autoconfigure`.

On step 4 `Autoconfigure`, Contrail configures each device based on the role. When this process is done, click `Finish`.

On the page `INFRASTRUCTURE > Fabrics > <fabric name>`, click `Edit` icon on the right end of device line to modify leaf devices.

On the device page, expand `Associated Service Nodes`, select `tor-service-node` as the `Virtual Router Type`, and select CSN as `Existing TSN`. Click `Save`.

Now, the fabric is imported.


# 3 BMS management

## 3.1 BMS with LCM

### 3.1.1 BMS flavor

### 3.1.2 BMS image

### 3.1.3 Provision network

### 3.1.3 Provision logical router

## 3.2 BMS without LCM

# 4 BMS inventory

# 5 BMS launch

# 6 BMS l2

# 7 BMS l3

# 8 BMS overlay-underlay



# 3 BMS image and flavor

Contrail supports BMS LCM (Life Cycle Management) including creating, enrolling, imaging, launching and deleting BMS.


## 3.1 BMS image

A kernel image and a ramdisk image are required to enroll BMS. Inspection is not supported by 5.0.2.

A host image, and specific kernel and ramdisk images are required to image BMS.

Go to page `WORKLOADS > Images`, click `Import`.

On the page `WORKLOADS > Images > Import Image`, do the followings and click `Import.
* Set image name.
* Select `Baremetal Server` as `Server Type`.
* Set upload file.
* Select container format and disk format.
* Select kernel image and ramdisk image for the host image.


## 3.2 BMS flavor

Flavor is used by Nova scheduler to choose BMS based on BMS profile.

Go to page `WORKLOADS > Flavors`, click `Create`.


# 4 BMS cases

## 4.1 BMS Inventory

Contrail Command is able to manage BMS inventory.

Go to page `INFRASTRUCTURE > Servers`, click `Create`.

On the page `INFRASTRUCTURE > Servers > Create Server`, set the followings and click `Create`.
* Select `Detailed`.
* Select `Baremetal`.
* Set hostname.
* For new BMS, select enroll kernel image and enroll ramdisk image as the `Deploy Kernel` and `Deploy Ramdisk`. For existing BMS, skip them.
* Set network interface name, MAC address and leaf interface. For new BMS, an interface with PXE enabled is required.
* Set `Port Groups` for BMS multi-homing.
* Set `IPMI Info` for new BMS.
* Set `Baremetal Properites` for new BMS. Once BMS inspection is supported, these properties will be set automatically.


## 4.2 Provisioing network


## 4.3 Enroll new BMS

After new BMS is created in inventory, it has to be enrolled (imported into Ironic) into the cluster.

Go to page `INFRASTRUCTURE > Cluster`,  tab `Cluster Nodes`, tab `Baremetal Servers`, click `Add`.

On the pop-up, select the new BMS from inventory, click `Enroll Server`. This adds BMS into Ironic. "openstack baremetal node list" will show it.

Click `change provision state` icon on the right end of BMS line to activate the BMS. The provision state will change to `manageable`, then `available`.


## 4.5 Launch new BMS


## 4.6 Launch existing BMS



