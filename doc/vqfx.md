
Download RE and PFE image of vmdk or box from Juniper.

For box format, untar it and get vmdk image.
```
# tar -tf vqfx-re-virtualbox.box
Vagrantfile
box.ovf
metadata.json
packer-virtualbox-ovf-1524541301-disk001.vmdk
```

Convert vmdk to qcow2.
```
qemu-img convert -f vmdk -O qcow2 \
  packer-virtualbox-ovf-1524541301-disk001.vmdk \
  vqfx-re.qcow2
```

Create bridges for em0, em1, em2, em3 (xe-0/0/0), em4 (xe-0/0/1), etc.
```
for br in br-mgmt vqfx-em1 vqfx-em2 vqfx-xe0 vqfx-xe1 vqfx-xe2 vqfx-xe3; do
  brctl addbr $br
  ip link set $br up
done
```

Launch VM.
```
vqfx_re()
{
    virt-install \
      --connect qemu:///system \
      --virt-type kvm \
      --name vqfx-re \
      --vcpus 1 \
      --ram 1024 \
      --disk path=/var/tmp/vqfx-re.qcow2,format=qcow2 \
      --network bridge=br-mgmt,model=e1000 \
      --network bridge=vqfx-em1,model=e1000 \
      --network bridge=vqfx-em2,model=e1000 \
      --network bridge=vqfx-xe0,model=e1000 \
      --network bridge=vqfx-xe1,model=e1000 \
      --network bridge=vqfx-xe2,model=e1000 \
      --network bridge=vqfx-xe3,model=e1000 \
      --graphics vnc,listen=0.0.0.0 --noautoconsole \
      --boot hd
}

vqfx_pfe()
{
    virt-install \
      --connect qemu:///system \
      --virt-type kvm \
      --name vqfx-pfe \
      --vcpus 1 \
      --ram 2048 \
      --disk path=/var/tmp/vqfx-pfe.qcow2,format=qcow2 \
      --network bridge=br-mgmt,model=e1000 \
      --network bridge=vqfx-em1,model=e1000 \
      --graphics vnc,listen=0.0.0.0 --noautoconsole \
      --boot hd
}
```


```
#!/bin/bash

vqfx_re()
{
    virt-install \
      --connect qemu:///system \
      --virt-type kvm \
      --name vqfx-re \
      --vcpus 1 \
      --ram 1024 \
      --disk path=/var/tmp/vqfx-re.qcow2,format=qcow2 \
      --network bridge=vqfx-em0,model=e1000 \
      --network bridge=vqfx-em1,model=e1000 \
      --network bridge=vqfx-em2,model=e1000 \
      --network bridge=vqfx-xe0,model=e1000 \
      --network bridge=vqfx-xe1,model=e1000 \
      --network bridge=vqfx-xe2,model=e1000 \
      --network bridge=vqfx-xe3,model=e1000 \
      --graphics vnc,listen=0.0.0.0 --noautoconsole \
      --boot hd
}

vqfx_pfe()
{
    virt-install \
      --connect qemu:///system \
      --virt-type kvm \
      --name vqfx-pfe \
      --vcpus 1 \
      --ram 2048 \
      --disk path=/var/tmp/vqfx-pfe.qcow2,format=qcow2 \
      --network bridge=vqfx-em0,model=e1000 \
      --network bridge=vqfx-em1,model=e1000 \
      --graphics vnc,listen=0.0.0.0 --noautoconsole \
      --boot hd
}

launch()
{
    vqfx_re
    vqfx_pfe
}

destroy()
{
    virsh destroy vqfx-re
    virsh destroy vqfx-pfe
    virsh undefine vqfx-re
    virsh undefine vqfx-pfe
}

launch
#destroy
```

