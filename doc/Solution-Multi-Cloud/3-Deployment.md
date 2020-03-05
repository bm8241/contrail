* [TOC](Multi-Cloud.md#toc)

# 3 Deployment

## 3.1 Deploy on Contrail-Kubernetes cluster

This deployment is based on [contrail-poc](https://github.com/tonyliu0592/contrail-poc) cluster `kubernetes-mc`. The deployment is done on Contrail Command.


### 3.1.1 Update server

#### Configure server data interface
![Figure 3.1 Server data interface](F3-1.png)

#### Note
Update data interface for all the following servers.
```
cmaster-1    eth1    10.6.11.61
csn-1        eth1    10.6.11.3
node-1       eth1    10.6.11.67
node-2       eth1    10.6.11.68
```


#### Create MC-GW server

![Figure 3.2 Create MC-GW server](F3-2.png)


### 3.1.2 Create public cloud

Public cloud is organized as this hierarchy.
```
- public cloud
  - provider
    - region
      - VPC
        - host
```


#### Create public cloud
This is to create the public cloud with initial hierarchy. After that, the public cloud can be updated on each layer.

![Figure 3.3 Create multi-cloud 1](F3-3.png)
![Figure 3.4 Create multi-cloud 2](F3-4.png)

Login Command VM and monitor the log.
```
ssh command
tail -f /var/log/contrail/cloud.log
```

This message indicates a successful creation.
```
msg="Terraform created all resources successfully"
```
AWS portal also shows all resources are create.


### 3.1.3 Create private cloud

Private cloud here is basically the MC-GW on premises.

#### Add sub-cluster
This is to create private cloud and connect to initial VPC. More VPCs can be added later by updating public cloud.

![Figure 3.5 Add sub-clusber 1](F3-5.png)
![Figure 3.6 Add sub-clusber 2](F3-6.png)
![Figure 3.7 Add sub-clusber 3](F3-7.png)

After click button "Create", Contrail Command will logout. To check the progress, login Command VM and monitor the log.
```
ssh command
tail -f /var/log/contrail/deploy.log
```

Once the process is finished, login Contrail Command. There will be `public-cloud` created.


### 3.1.4 Update public cloud

Click `public-cloud`, goto specific layer, update it and all below.
```
- public cloud
  - provider
    - region
      - VPC
        - host
```

#### Add VPC

![Figure 3.8 Add VPC 1](F3-8.png)
![Figure 3.9 Add VPC 2](F3-9.png)
![Figure 3.10 Add VPC 3](F3-10.png)

After creating the VPC, click button `provision` to apply the update. Resources on public cloud will be created as requested. Private cloud (MC-GW) will also be updated to add a new secured tunnel to the VPC.



## 3.2 Deploy MC-GW only

This is to deploy on-prem MC-GW and cloud MC-GW only. It can connect any workloads between local subnets and cloud VPCs. Some routing has to be done manually. The deployment is done with CLI.


### 3.2.1 Launch Contrail Command
Contrail Command container has all required utilities and playbooks. It doesn't matter if the Command has any clusters imported/created or no cluster. Since the deployment is done by CLI, not Command, the deployment is not visible on Contrail Command.


### 3.2.2 SSH key

#### Create SSH key pair.
```
ssh-keygen
```

#### Create SSH configuration `~/.ssh/config`.
```
Host *
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```

#### Copy SSH key into Command container.
```
docker exec contrail_command mkdir -p /root/.ssh
docker cp .ssh/id_rsa contrail_command:/root/.ssh/
docker cp .ssh/id_rsa.pub contrail_command:/root/.ssh/
docker cp .ssh/config contrail_command:/root/.ssh/
```


### 3.2.3 Deployment YAML

#### Build `topology.yaml`.
[topology.yaml](A4-Deployment-YAML.md#a421-topologyyaml)

#### Build `secret.yaml`.
[secret.yaml](A4-Deployment-YAML.md#a422-secretyaml)

#### Copy `topology.yaml` and `secret.yaml` into Command container.
```
docker cp topology.yaml contrail_command:/root
docker cp secret.yaml contrail_command:/root
```


### 3.2.4 Patch

Apply this [patch](A5-Patch.md#a5-patch) to enable `--skip_controller_validation`. Or use `--skip_validation` which will skip all validations.


### 3.2.5 Deploy

#### Create all required resources on public cloud.
```
docker exec contrail_command bash -c \
    "cd /root; deployer all topology \
        --topology topology.yaml \
        --secret secret.yaml \
        --skip_controller_validation \
        --retry 1"
```
The VPC subnet provided by `topology.yaml` is split (prefix length + 1) to two subnets, public and private. Cloud MC-GW is launched on those two subnets. An elastic IP is attached to public interface for public access. Multiple VPC subnets is not currently supported by deployment. It could be done manually.

The default route in VPC private subnet is MC-GW. The default route in VPC public subnet is the VPC internet gateway.

#### Create inventory for deployment.
```
docker exec contrail_command bash -c \
    "cd /root; deployer inventory create \
        --topology topology.yaml \
        --secret secret.yaml"
```

#### Deploy on-prem MC-GW and cloud MC-GW.
```
docker exec contrail_command bash -c \
    "cd /root; deployer multicloud build \
        --inventory inventories/inventory.yml"
```

#### Delete cloud resources.
```
docker exec contrail_command bash -c \
    "cd /root; deployer topology destroy \
        --secret secret.yaml"
```


### 3.2.6 VM on VPC

Launch a VM on the VPC private subnet with the same security group as MC-GW. Private VM can be accessed from MC-GW.


### 3.2.7 Overlay-underlay gateway

The virtual network where VM and BMS is launched has to be extended to the gateway for overlay-underlay connectivity. Gateway also needs to peer with BIRD service on MC-GW to populate virtual network.





