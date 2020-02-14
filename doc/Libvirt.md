## Introduction
This guide shows how to install Contrail networking and libvirt/KVM, and how Contrail connects KVM based VMs to virtual network.


## Setup Servers
The setup consists of two servers.
* Controller running all controlling nodes (configuration, analytics, database, Web UI, control)
* Hypervisor running Contrail vrouter and libvirt/KVM, and VMs

OS is Ubuntu 14.04.2. Contrail is 3.0.0.0-2723.

Here are prerequisites to each server before start installing Contrail.
* Networking configuration
* Resolvable hostname
* NTP service


## Install Contrail
* Copy Contrail installation package, eg. contrail-install-packages_3.0.0.0-2723~ubuntu-14-04kilo_all.deb, to the controller.

* Install the package.
```
# dpkg -i <package>
# cd /opt/contrail/contrail_packages
# ./setup.sh
```

* Build /opt/contrail/utils/fabfile/testbeds/testbed.py. Here is an [example](https://github.com/tonyliu0592/opencontrail-install/blob/master/fabric/testbed-contrail-only.py).

* Apply the [fabfile patch](https://github.com/tonyliu0592/opencontrail-install/blob/master/fabric/fabfile-3.0-2723.diff).
```
# cd /opt/contrail/utils/fabfile
# patch -p1 < /path/to/fabfile-3.0-2723.diff
```

* Run fab commands.
```
# cd /opt/contrail/utils
# fab install_pkg_all_without_openstack:</path/to/package>
# fab install_without_openstack:manage_nova_compute='no'
# fab setup_without_openstack:manage_nova_compute='no',config_nova='no'
```

## Web UI local authentication
Update /etc/contrail/config.global.js.
```
config.staticAuth = [];
config.staticAuth[0] = {};
config.staticAuth[0].username = 'admin';
config.staticAuth[0].password = 'password';
config.staticAuth[0].roles = ['superAdmin'];
```

Restart Web UI services.
```
# service supervisor-webui restart
```

## Libvirt/KVM
* Install libvirt on the hypervisor node.
```
# apt-get install qemu-system-x86=2.0.0+dfsg-2ubuntu1.22 qemu-kvm libvirt-bin virtinst
```

* Enable the following settings in /etc/libvirt/qemu.conf to allow libvirt use tap interface.
```
clear_emulator_capabilities = 0
user = "root"
group = "root"
cgroup_device_acl = [
    "/dev/null", "/dev/full", "/dev/zero",
    "/dev/random", "/dev/urandom",
    "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
    "/dev/rtc", "/dev/hpet",
    "/dev/net/tun"
]
```

* Restart libvirt.
```
# service libvirt-bin restart
```


## Connect VM

* Get utility and update 'agent_default' in agent.
```
# wget https://github.com/tonyliu0592/opencontrail-config/raw/master/agent
```

* Create tenant 'admin'.
```
# ./agent tenant-add admin
```

* On Contrail Web UI, http://<controller>:8080, create virtual network and subnet in tenant 'admin', eg. network 'red' with subnet 192.168.10.0/24.

* Create virtual machine in Contrail. This is only a logical object in Contrail representing the real virtual machine launched later.
```
# ./agent vm-add host1-red --network red 
      <mac address="02:e4:ac:47:db:b6"/>
      <target dev="tape4ac47db-b6"/>
```
Other than virtual machine object, it also created a tap interface and VM interface with a MAC address allocated.

* Create VM image.
```
# wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
# cp cirros-0.3.4-x86_64-disk.img host1.img
```

* Build VM description file, eg. host1.xml. Appendix A has an example. Replace __mac__ and __tap__ with the value above. Set __name__ and __image_file__(full name with path).

* Launch VM.
```
# virsh create host1.xml
```

* Access VM can be done by VNC. "virsh vncdisplay <VM>" shows the display number, then VNC to <hypervisor>:<display>.

For Cirros cloud image, it will take a few minutes to boots up, because it looks for metadata. Once it boots up, login and check networking, interface should be configured by DHCP.


## Appendix A
```
<domain type="kvm">
  <name>__name__</name>
  <memory unit='Mb'>2048</memory>
  <vcpu>1</vcpu>
  <os>
    <type arch='x86_64'>hvm</type>
    <boot dev="hd"/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <clock offset="utc"/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' cache='none'/>
      <source file='__image_file__'/>
      <target dev='vda' bus='virtio'/>
    </disk>
    <interface type="ethernet">
      <mac address="__mac__"/>
      <target dev="__tap__"/>
      <model type="virtio"/>
      <script path=""/>
    </interface>
    <serial type='pty'>
      <target port='0'/>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0'/>
    <video>
      <model type="cirrus"/>
    </video>
  </devices>
</domain>
```

