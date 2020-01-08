* [TOC](SRIOV.md)

# 2 Hypervisor

## 2.1 IOMMU

Enable IOMMU in BIOS.

Enable IOMMU in kernel. Update `/etc/default/grub` with IOMMU setting.
```
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet intel_iommu=on"
```

Apply grub settings.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

In case of EFI.
```
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```

Reboot.

Check kernel settings.
```
# cat /proc/cmdline 
BOOT_IMAGE=/vmlinuz-3.10.0-957.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet intel_iommu=on
```

Check with libvirt.
```
virt-host-validate | grep IOMMU
```


## 2.2 VF

Enable VF.
```
echo 16 > /sys/class/net/ens1f1/device/sriov_numvfs
```

Persistently.
```
echo "echo 16 > /sys/class/net/ens1f1/device/sriov_numvfs" >> /etc/rc.local
```

Ensure all VF links are up.


## 2.3 OpenStack

Update nova.conf for nova-compute.
```
[pci]
passthrough_whitelist = {"devname": "ens1f1", "physical_network": "sriov"}
```

The resource view on compute node should have device pools for both PF and VFs.
Otherwise, ensure all VFs are up and restart nova-compute.
```
Final resource view: name=dc-poclab-compute1.poc-nl.jnpr.net phys_ram=327569MB used_ram=1536MB phys_disk=889GB used_disk=2GB total_vcpus=64 used_vcpus=2 pci_stats=[PciDevicePool(count=1,numa_node=1,product_id='10fb',tags={dev_type='type-PF',physical_network='sriov'},vendor_id='8086'), PciDevicePool(count=16,numa_node=1,product_id='10ed',tags={dev_type='type-VF',physical_network='sriov'},vendor_id='8086')]
```


