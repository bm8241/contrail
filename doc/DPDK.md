
CentOS 7.5

# Hugepage

[Hugepage support by Linux kernel](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt)


## Transparent hugepage

[Transparent hugepage support by Linux kernel](https://www.kernel.org/doc/Documentation/vm/transhuge.txt)

By default, THP is enabled.
```
# cat /proc/meminfo | grep Huge
AnonHugePages:     10240 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

To disable THP, update `GRUB_CMDLINE_LINUX` in `/etc/default/grub2`.
```
transparent_hugepage=never
```

Apply gurb update.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```


## Page size

For X86-64, default hugepage size is 2M. Size 1G is supported by processors with flag `pdpe1gb`.
```
cat /proc/cpuinfo | grep pdpe1gb.
```


## Create pool

It's recommended to create hugepage pool when system boots up.

Check boot line for hugepage pool.
```
# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-957.1.3.el7.x86_64 root=/dev/mapper/vg1-root ro crashkernel=auto rd.lvm.lv=vg1/root rd.lvm.lv=vg1/swap rhgb quiet LANG=en_US.UTF-8
```

To create a pool of 2M hugepage, update `GRUB_CMDLINE_LINUX` in `/etc/default/grub2`.
```
default_hugepagesz=2M hugepagesz=2M hugepages=16000
```

To create a pool of 1G hugepage as default.
```
default_hugepagesz=1G hugepagesz=1G hugepages=48
```

To create a pool of 2M hugepage and a pool of 1G hugepage with default size 2M.
```
default_hugepagesz=2M hugepagesz=2M hugepages=16000 hugepagesz=1G hugepages=48"
```

To create a pool of 2M hugepage and a pool of 1G hugepage with default size 1G.
```
default_hugepagesz=1G hugepagesz=1G hugepages=48 hugepagesz=2M hugepages=16000
```

Apply grub update.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```


## Mount

A systemd mount unit comes with the system for default hugepage size.
```
# systemctl | grep hugepage
dev-hugepages.mount    loaded active mounted   Huge Pages File System
```

If both 2M and 1G hugpage pools are created, it's recommended to create mount for each size explicitly in `/etc/fstab`.
```
hugetlbfs /dev/hugepages2M hugetlbfs pagesize=2M 0 0
hugetlbfs /dev/hugepages1G hugetlbfs pagesize=1G 0 0
```

Systemd mount unit will be added for each mount.
```
# systemctl | grep hugepage
  dev-hugepages.mount        loaded active mounted   Huge Pages File System
  dev-hugepages1G.mount      loaded active mounted   /dev/hugepages1G
  dev-hugepages2M.mount      loaded active mounted   /dev/hugepages2M
```

Check mount.
```
# cat /proc/mounts | grep hugepage
hugetlbfs /dev/hugepages1G hugetlbfs rw,seclabel,relatime,pagesize=1G 0 0
hugetlbfs /dev/hugepages2M hugetlbfs rw,seclabel,relatime,pagesize=2M 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,seclabel,relatime 0 0
```

Note, the order of mounts in `/proc/mounts` is not determined by the order in `/etc/fstab`. It's actually not determinable.


## Check pool and usage

Check `/proc/meminfo` for hugepage pool of default size.
```
# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:   16000
HugePages_Free:    16000
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```
The above shows 1) THP is disabled, 2) default hugepage size is 2M, 3) 16000 2M hugepages total.

Check sysctl for the number of hugepages of default size.
```
# cat /proc/sys/vm/nr_hugepages
16000
```

Check sysfs for the number of hugepages of each size.
```
# cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
16000
# cat /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages    
48
```

Check sysfs for the number of hugepages allocated to each NUMA node.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
8000
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/nr_hugepages
24
# cat /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
8000
# cat /sys/devices/system/node/node1/hugepages/hugepages-1048576kB/nr_hugepages
24
```


# Libvirt

## Hugepage

[Hugepage support by libvirt](https://libvirt.org/formatdomain.html#elementsMemoryBacking)


### THP

Start a VM with hugepage as backend memory.
```
  <memoryBacking>
    <hugepages\>
  </memoryBacking>
```

THP is allocated for VM, if it's enabled and there is no other hugepage pools.
```
# cat /proc/vmstat | grep hugepage
nr_anon_transparent_hugepages 159
```


### 2M hugepage

Start a VM using 2M hugepage.
```
  <memoryBacking>
    <hugepages>
      <page size='2048' unit='KiB'/>
    </hugepages>
  </memoryBacking>
```

Libvirt will find the right hugepage mount and pass to qemu-kvm.
```
# ps ax | grep mem-path
 2490 ?        Sl     0:13 /usr/libexec/qemu-kvm -name host1 -S -machine pc-i440fx-rhel7.0.0,accel=kvm,usb=off,dump-guest-core=off -cpu Haswell,-hle,-rtm -m 1024 -mem-prealloc -mem-path /dev/hugepages/libvirt/qemu/1-host1 ......
```
When default hugepage is 2M, `/dev/hugepages` is for the default. If default hugepage is 1G and 2M hugepage is mounted on `/dev/hugepages2M`, libvirt will take the right 2M hugepage mount.

520 2M hugepages are used for VM (1GB memory).
```
# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:   16000
HugePages_Free:    15480
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

They are allocated from node 1.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/free_hugepages  
8000
# cat /sys/devices/system/node/node1/hugepages/hugepages-2048kB/free_hugepages 
7480
```


### 1G hugepage

Start a VM to use 1G hugepage.
```
  <memoryBacking>
    <hugepages>
      <page size='1048576' unit='KiB'/>
    </hugepages>
  </memoryBacking>
```

Libvirt will find the right mount and pass to qemu-kvm.
```
# ps ax | grep mem-path
 2841 ?        Sl     0:13 /usr/libexec/qemu-kvm -name host1 -S -machine pc-i440fx-rhel7.0.0,accel=kvm,usb=off,dump-guest-core=off -cpu Haswell,-hle,-rtm -m 4096 -mem-prealloc -mem-path /dev/hugepages1G/libvirt/qemu/1-host1 ......
```
When default hugepage is 2M, `/dev/hugepages1G` is for 1G hugepage. If default hugepage is 1G, libvirt will take the right 1G hugepage mount `/dev/hugepages`.

4 1G hugepages are allocated from node 0 for VM (4G memory).
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/free_hugepages
20
# cat /sys/devices/system/node/node1/hugepages/hugepages-1048576kB/free_hugepages
24
```

Check proc to find out how many hugepages used by the process.
```
#cat /proc/<pid>/numa_maps | grep hugepage
2aaac0000000 default file=/dev/hugepages1G/libvirt/qemu/1-host1/qemu_back_mem.pc.ram.2wVJuq\040(deleted) huge anon=4 dirty=4 N0=4 kernelpagesize_kB=1048576
```


# Vrouter-DPDK

## Hugepage

Vrouter-DPDK allocates hugpage for bridge table (and 20% overflow) and flow table (and 20% overflow). The default bridge table size is 256K. The default flow table size is 512K.

Vrouter-DPDK goes through `/proc/mounts` to get all hugepage mounts. Then it will allocate hugepages from the first whoever has sufficient space.

Note, when both 2M and 1G hugepage are reserved and mounted, the order in `/proc/mounts` is not determinable. It means vrouter could allocate hugepages from ether mount points. This needs to be fixed.

With `--socket-mem 1024 1024`, 2G hugepages are allocated for DPDK memory pool (1G from each NUMA node). Here is an example with 1G hugepage. 
```
7f4ec0000000 default file=/dev/hugepages1G/rtemap_0 huge dirty=1 N0=1 kernelpagesize_kB=1048576
7f4f00000000 default file=/dev/hugepages1G/rtemap_1 huge dirty=1 N1=1 kernelpagesize_kB=1048576
```


# OpenStack

## Hugepage

When launch a VM with hugepage as the back memory, for example, property `hw:mem_page_size='1GB'` can be specified in flavor to use 1G hugepage.

Note, the VM memory has to be multiple of pagesize specified in flavor.
```
    def can_fit_hugepages(self, pagesize, memory):
        """Returns whether memory can fit into hugepages size

        :param pagesize: a page size in KibB
        :param memory: a memory size asked to fit in KiB

        :returns: whether memory can fit in hugepages
        :raises: MemoryPageSizeNotSupported if page size not supported
        """
        for pages in self.mempages:
            if pages.size_kb == pagesize:
                return (memory <= pages.free_kb and
->                      (memory % pages.size_kb) == 0)
        raise exception.MemoryPageSizeNotSupported(pagesize=pagesize)
```









## Default hugepage size 2M

Check boot line, no specfic hugepage reservation.
```
# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-957.1.3.el7.x86_64 root=/dev/mapper/vg1-root ro crashkernel=auto rd.lvm.lv=vg1/root rd.lvm.lv=vg1/swap rhgb quiet LANG=en_US.UTF-8
```

A systemd mount unit comes with the host for the default hugepage.
```
# systemctl status dev-hugepages.mount
? dev-hugepages.mount - Huge Pages File System
   Loaded: loaded (/usr/lib/systemd/system/dev-hugepages.mount; static; vendor preset: disabled)
   Active: active (mounted) since Fri 2018-10-05 16:19:49 PDT; 2 months 15 days ago
    Where: /dev/hugepages
     What: hugetlbfs
     Docs: https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt
           http://www.freedesktop.org/wiki/Software/systemd/APIFileSystems
    Tasks: 0
```

Default hugepage size is 2M. No hugepages are reserved.
```
# cat /proc/meminfo | grep Huge
AnonHugePages:     10240 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```
`AnonHugePages` is anonymous memory regions such as heap and stack space that can be mapped by THP (Transparent HugePage).

```
# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
# cat /sys/kernel/mm/transparent_hugepage/defrag
[always] madvise never
```

[How to use, monitor, and disable transparent hugepages in Red Hat Enterprise Linux 6 and 7?](https://access.redhat.com/solutions/46111)

Check sysctl for the number of hugepages.
```
# cat /proc/sys/vm/nr_hugepages
0
```

Check sysfs for the number of hugepages of each size.
```
# cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
0
# cat /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages    
0
```

Check sysfs for the number of hugepages allocated to each NUMA node.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages 
0
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/nr_hugepages       
0
# cat /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages 
0
# cat /sys/devices/system/node/node1/hugepages/hugepages-1048576kB/nr_hugepages       
0
```

Start a VM with hugepage as backend memory.
```
  <memoryBacking>
    <hugepages\>
  </memoryBacking>
```

THP is allocated for VM.
```
# cat /proc/vmstat | grep hugepage
nr_anon_transparent_hugepages 159
```


## Reserve default 2M hugepage

Update `GRUB_CMDLINE_LINUX` in /etc/default/grub2 to disable THP and reserve default 2M hugepage.
```
transparent_hugepage=never default_hugepagesz=2M hugepagesz=2M hugepages=16000
```

Apply update.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

After reboot, THP is disabled, 16000 default 2M hugepages are reserved.
```
# cat /proc/sys/vm/nr_hugepages
16000

# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:   16000
HugePages_Free:    16000
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

Check sysfs for the number of hugepages of each size.
```
# cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
16000
# cat /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages    
0
```

Check sysfs for the number of hugepages allocated to each NUMA node.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
8000
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/nr_hugepages
0
# cat /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
8000
# cat /sys/devices/system/node/node1/hugepages/hugepages-1048576kB/nr_hugepages
0
```

One default hugepage mount for the default size.
```
# systemctl | grep hugepage
dev-hugepages.mount    loaded active mounted   Huge Pages File System
```

Start VM using 2M hugepage.
```
  <memoryBacking>
    <hugepages>
      <page size='2048' unit='KiB'/>
    </hugepages>
  </memoryBacking>
```

Libvirt will find hugepage mount and pass to qemu-kvm.
```
# ps ax | grep mem-path
 2490 ?        Sl     0:13 /usr/libexec/qemu-kvm -name host1 -S -machine pc-i440fx-rhel7.0.0,accel=kvm,usb=off,dump-guest-core=off -cpu Haswell,-hle,-rtm -m 1024 -mem-prealloc -mem-path /dev/hugepages/libvirt/qemu/1-host1 ......
```

520 2M hugepages are used for VM (1GB memory).
```
# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:   16000
HugePages_Free:    15480
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

They are allocated from node 1.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/free_hugepages  
8000
# cat /sys/devices/system/node/node1/hugepages/hugepages-2048kB/free_hugepages 
7480
```


## Reserve 1G hugepage

Update grub to reserve 1G hugepage, default hugepage size is still 2M.
```
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=vg1/root rd.lvm.lv=vg1/swap rhgb quiet transparent_hugepage=never default_hugepagesz=2M hugepagesz=2M hugepages=16000 hugepagesz=1G hugepages=48"
```

Apply update.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

After reboot, meminfo shows default hugepages.
```
# cat /proc/sys/vm/nr_hugepages
16000

# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:   16000
HugePages_Free:    16000
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

Check sysfs for the number of hugepages of each size.
```
# cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
16000
# cat /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages    
48
```

Check sysfs for the number of hugepages allocated to each NUMA node.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
8000
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/nr_hugepages
24
```

There is only default hugepage mount for default size 2M.
```
# systemctl | grep huge
dev-hugepages.mount    loaded active mounted   Huge Pages File System
```

To use 1G hugepage, need to create a mount.
```
mkdir /dev/hugepages1G
mount -t hugetlbfs hugetlbfs /dev/hugepages1G -o pagesize=1G
```
```
# mount | grep hugepage
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages1G type hugetlbfs (rw,relatime,seclabel,pagesize=1G)
```

To make hugepage mount persistent, update `/etc/fstab`.
```
hugetlbfs /dev/hugepages1G hugetlbfs pagesize=1G 0 0
```

Need to restart libvirtd to recoganize the new hugepage mount.
```
systemctl restart libvirtd
```
`/dev/hugepages1G/libvirt` will be created by libvirted. No need to set `hugetlbfs_mount` in `/etc/libvirt/qemu.conf`.

Update VM to use 1G hugepage and restart it.
```
  <memoryBacking>
    <hugepages>
      <page size='1048576' unit='KiB'/>
    </hugepages>
  </memoryBacking>
```

Libvirt will find the right mount.
```
# ps ax | grep mem-path
 2841 ?        Sl     0:13 /usr/libexec/qemu-kvm -name host1 -S -machine pc-i440fx-rhel7.0.0,accel=kvm,usb=off,dump-guest-core=off -cpu Haswell,-hle,-rtm -m 4096 -mem-prealloc -mem-path /dev/hugepages1G/libvirt/qemu/1-host1 ......
```

4 1G hugepages are allocated from node 0 for VM (4G memory).
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/free_hugepages
20
# cat /sys/devices/system/node/node1/hugepages/hugepages-1048576kB/free_hugepages
24
```

Check proc to find out how many hugepages used by the process.
```
#cat /proc/<pid>/numa_maps | grep hugepage
2aaac0000000 default file=/dev/hugepages1G/libvirt/qemu/1-host1/qemu_back_mem.pc.ram.2wVJuq\040(deleted) huge anon=4 dirty=4 N0=4 kernelpagesize_kB=1048576
```


## Reserve default 1G hugepage

```
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=vg1/root rd.lvm.lv=vg1/swap rhgb quiet transparent_hugepage=never default_hugepagesz=1G hugepagesz=2M hugepages=16000 hugepagesz=1G hugepages=48"
```

Apply update.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

After reboot, meminfo shows default hugepages.
```
# cat /proc/sys/vm/nr_hugepages
48
# cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:      48
HugePages_Free:       48
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:    1048576 kB
```

Check sysfs for the number of hugepages of each size.
```
# cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
16000
# cat /sys/kernel/mm/hugepages/hugepages-1048576kB/nr_hugepages    
48
```

Check sysfs for the number of hugepages allocated to each NUMA node.
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
8000
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/nr_hugepages
24
```

There is only default hugepage mount for default size 1G.
```
# systemctl | grep huge
dev-hugepages.mount    loaded active mounted   Huge Pages File System
```

To use 2M hugepage, need to create a mount.
```
mkdir /dev/hugepages2M
mount -t hugetlbfs hugetlbfs /dev/hugepages2M -o pagesize=2M
```
```
# mount | grep hugepage
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
hugetlbfs on /dev/hugepages2M type hugetlbfs (rw,relatime,seclabel,pagesize=2M)
```

To make hugepage mount persistent, update `/etc/fstab`.
```
hugetlbfs /dev/hugepages2M hugetlbfs pagesize=2M 0 0
```

Need to restart libvirtd to recoganize the new hugepage mount.
```
systemctl restart libvirtd
```
`/dev/hugepages2M/libvirt` will be created by libvirted. No need to set `hugetlbfs_mount` in `/etc/libvirt/qemu.conf`.

Update VM to use 2M hugepage and retart it.
```
  <memoryBacking>
    <hugepages>
      <page size='2048' unit='KiB'/>
    </hugepages>
  </memoryBacking>
```

Libvirt will find the right mount.
```
# ps ax | grep mem-path
 2709 ?        Sl     0:09 /usr/libexec/qemu-kvm -name host1 -S -machine pc-i440fx-rhel7.0.0,accel=kvm,usb=off,dump-guest-core=off -cpu Haswell,-hle,-rtm -m 4096 -mem-prealloc -mem-path /dev/hugepages2M/libvirt/qemu/2-host1 ......
```

2056 2M hugepages are allocated from node 0 for VM (4G memory).
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-2048kB/free_hugepages
5944
# cat /sys/devices/system/node/node1/hugepages/hugepages-2048kB/free_hugepages
8000
```

Update VM to use 1G hugepage and retart it.
```
  <memoryBacking>
    <hugepages>
      <page size='1048576' unit='KiB'/>
    </hugepages>
  </memoryBacking>
```

Libvirt will find the right mount.
```
# ps ax | grep mem-path
 2798 ?        Sl     0:19 /usr/libexec/qemu-kvm -name host1 -S -machine pc-i440fx-rhel7.0.0,accel=kvm,usb=off,dump-guest-core=off -cpu Haswell,-hle,-rtm -m 4096 -mem-prealloc -mem-path /dev/hugepages/libvirt/qemu/2-host1 ......
```

4 1G hugepages are allocated from node 0 for VM (4G memory).
```
# cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/free_hugepages 
20
# cat /sys/devices/system/node/node1/hugepages/hugepages-1048576kB/free_hugepages  
24
```

Check proc to find out how many hugepages used by the process.
```
# cat /proc/2798/numa_maps | grep hugepage
2aaac0000000 default file=/dev/hugepages/libvirt/qemu/2-host1/qemu_back_mem.pc.ram.2wVJuq\040(deleted) huge anon=4 dirty=4 N0=4 kernelpagesize_kB=1048576
```


# Vrouter-DPDK

## Hugepage

Vrouter allocate hugpage for bridge table (and 20% overflow) and flow table (and 20% overflow). The default bridge table size is 256K. The default flow table size is 512K.

Vrouter goes through `/proc/mounts` to get all hugepage mounts. Then it will allocate hugepage from whoever the first having sufficient space.

Note, when both 2M and 1G hugepage are reserved and mounted, the order in `/proc/mounts` is non-deterministic. It means vrouter could allocate hugepages from ether mount points. This needs to be fixed.


## Thread and CPU affinity

Vrouter-DPDK is invoked by taskset with CPU mask, which is to specify CPU affinity for forwarding threads.

Vrouter-DPDK has the following threads, with CPU mask 0x0f (4 forwarding threads) for example.
* contrail-vroute: the main thread on all system CPUs
* vfio-sync: on CPU 0-3
* eal-intr-thread: on all system CPUs
* lcore-slave-1: timer thread on all system CPUs
* lcore-slave-2: user-vhost thread on all system CPUs
* lcore-slave-8: packet thread on all system CPUs
* lcore-slave-9: netlink thread on all system CPUs
* lcore-slave-9: netlink thread on all system CPUs
* lcore-slave-10: forwarding thread on CPU 0 (0x00)
* lcore-slave-11: forwarding thread on CPU 1 (0x01)
* lcore-slave-12: forwarding thread on CPU 2 (0x02)
* lcore-slave-13: forwarding thread on CPU 3 (0x04)

rte_lcore_count() returns 9, the number of lcores.


Check thread and CPU affinity from `/proc/<pid>/task/<tid>/status`.

Check affinity with `top` command. First, launch `top` with `-p` option. Then press `f` key, and add "Last used CPU" column to the display. The currently/lastly used CPU core will appear under "P" (or "PSR") column.


# OpenStack DPDK

## Hugepage

When launch a VM with hugepage as the back memory, for example, property `hw:mem_page_size='1GB'` can be specified in flavor to use 1G hugepage.

Note, the VM memory has to be multiple of pagesize specified in flavor.
```
146  ->     def can_fit_hugepages(self, pagesize, memory):
147             """Returns whether memory can fit into hugepages size
148  
149             :param pagesize: a page size in KibB
150             :param memory: a memory size asked to fit in KiB
151  
152             :returns: whether memory can fit in hugepages
153             :raises: MemoryPageSizeNotSupported if page size not supported
154             """
155             for pages in self.mempages:
156                 if pages.size_kb == pagesize:
157                     return (memory <= pages.free_kb and
158                             (memory % pages.size_kb) == 0)
159             raise exception.MemoryPageSizeNotSupported(pagesize=pagesize)
```


