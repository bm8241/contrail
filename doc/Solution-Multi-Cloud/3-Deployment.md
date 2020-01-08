* [TOC](Multi-Cloud.md#toc)

# 3 Deployment

This deployment is based on [contrail-poc](https://github.com/tonyliu0592/contrail-poc) cluster `kubernetes-mc`.


## 3.1 Update server

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


## 3.2 Create public cloud

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


## 3.3 Create private cloud

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


## 3.4 Update public cloud

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



