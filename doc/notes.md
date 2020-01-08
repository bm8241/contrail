curl -H "X-Auth-Token:$(keystone token-get | awk '/ id / {print $4}')" http://127.0.0.1:8082/virtual-networks | python -m json.tool


strace -p 19826 -e trace=network -s 512


curl -s -H "X-Auth-Token: $(keystone token-get | awk '/ id / {print $4}')" local
host:8082/global-vrouter-configs


$ qemu-img convert -f qcow2 -O vmdk trusty-server-cloudimg-amd64-disk1.img trust
y-server-cloudimg-amd64-disk1.vmdk

$ head -20 trusty-server-cloudimg-amd64-disk1.vmdk

$ glance image-create --name esxi/ubuntu-trusty --disk-format vmdk --container-f
ormat bare --property vmware_disktype="sparse" --property vmware_adaptertype="id
e" --is-public True --file trusty-server-cloudimg-amd64-disk1.vmdk

docker run -it --entrypoint "/bin/bash" <image ID>


* offload

  * In VM, when checksum offload is enabled, packet with incompleted CS is sent out from tap interface. Otherwise, packet with completed CS is sent out from tap interface.

  * In VM, when segmentation offload is enabled, packet without segmentation is sent out. Otherwise, packet is segmented based on VM interface MTU.

  * Vrouter always does software segmentation if it's required. Segmentation offload setting on vrouter underlay interface only affects host traffic, not overlay (encapsulated) traffic.

  * Vrouter checks NIC capability of checksum offload (NETIF_F_HW_CSUM). This flag comes from hardware. It can't be changed by software, like ethtool. If NIC has such capability, vrouter will let NIC to calculate checksum. Otherwise, vrouter will do software checksum. Again, software setting, like ethtool, doesn't change vrouter behavior.

  * Issues
    * NIC cards that has NETIF_F_HW_CSUM set, but do not really support checksum and generate an incorrect checksum (e.g., Broadcom Corporation NetXtreme BCM5719 Gigabit Ethernet PCIe NIC cards).

    * Nested VMs, there is a bug in vrouter when calculate checksum for packet with MPLSoGRE encapsulation (talk to gateway).

    * VMs running in ESXi Hypervisor

  * Workaround
    To avoid offload issues, the workaround is to disable offload in VM, so vrouter always gets completed checksum and no need to do fragmentation.


Data Plane

  Underlay
    IP

  Overlay
    Layer 2

    Layer 3
      UDP tunnel, MPLS over UDP
      GRE tunnel, MPLS over GRE

L2 Frame -> vRouter
  L2 mode, MAC VRF
  L2 and L3 mode, if it's IP, IP VRF, otherwise MAC VRF


Control Plane


