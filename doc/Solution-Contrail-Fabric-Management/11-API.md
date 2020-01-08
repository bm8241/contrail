* [TOC](Contrail-Fabric-Management.md)

# 11 API

Fabric management can be done via REST API provided by configuration API server.

The tool [config](https://github.com/tonyliu0592/contrail-toolbox/blob/master/config/config) is used in this guide.

#### Create fabric from zero

Here is an example.
```
config set --file job-create-fabric.yaml
```
[job-create-fabric.yaml](A6-Resource-file.md#a61-job-create-fabricyaml)


#### Import existing fabric

Here is an example.
```
config set --file job-import-fabric.yaml
```
[job-import-fabric.yaml](A6-Resource-file.md#a62-job-import-fabricyaml)


#### Assign role

Here is an example.
```
config set --file job-assign-role.yaml
```
[job-assign-role.yaml](A6-Resource-file#a63-job-assign-roleyaml)

[vqfx-leaf-1 configuration](A7-Underlay-configuration.md#a71-vqfx-leaf-1)
[vqfx-leaf-2 configuration](A7-Underlay-configuration.md#a72-vqfx-leaf-2)
[vqfx-leaf-3 configuration](A7-Underlay-configuration.md#a73-vqfx-leaf-3)
[vqfx-leaf-4 configuration](A7-Underlay-configuration.md#a74-vqfx-leaf-4)
[vqfx-spine-1 configuration](A7-Underlay-configuration.md#a75-vqfx-spine-1)

#### Link L2 GW to CSN

Here is an example.
```
config set --file ref-leaf-csn.yaml
```
[ref-leaf-csn.yaml](A6-Resource-file.md#a64-ref-leaf-csnyaml)


#### Create VPG

Here is an example.
```
config set --file vpg-bms-22.yaml
```
[vpg-bms-22.yaml](A6-Resource-file.md#a65-vpg-bms-22yaml)

