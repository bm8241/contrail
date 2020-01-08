* [TOC](Contrail-Fabric-Management.md#toc)

## A.6 Resource file

### A.6.1 job-create-fabric.yaml
```
- resource_type: job
  resource:
    job_template_fq_name:
    - default-global-system-config
    - fabric_onboard_template
    input:
      fabric_fq_name:
      - default-global-system-config
      - poc
      fabric_display_name: poc
      device_auth:
        root_password: Juniper
      overlay_ibgp_asn: 64512
      loopback_subnets:
      - 10.6.0.0/24
      management_subnets:
      - cidr: 10.6.8.0/24
        gateway: 10.6.8.254
      fabric_subnets:
      - 10.6.20.0/24
      fabric_asn_pool:
      - asn_min: 64500
        asn_max: 64599
      node_profiles:
      - node_profile_name: juniper-mx
      - node_profile_name: juniper-qfx10k
      - node_profile_name: juniper-qfx10k-lean
      - node_profile_name: juniper-qfx5120
      - node_profile_name: juniper-qfx5k
      - node_profile_name: juniper-qfx5k-lean
      - node_profile_name: juniper-srx
      import_configured: false
      enterprise_style: true
      supplemental_day_0_cfg:
      - name: cfg
        cfg: |
          set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
          set interfaces em1 unit 0 family inet address 169.254.0.2/24
          set routing-options static route 0.0.0.0/0 next-hop 10.6.8.254
          set protocols lldp interface all
      device_to_ztp:

      - serial_number: "65186286083"
        supplemental_day_0_cfg: cfg
        hostname: vqfx-leaf-1
      - serial_number: "41043147134"
        supplemental_day_0_cfg: cfg
        hostname: vqfx-leaf-2
      - serial_number: "84383385230"
        supplemental_day_0_cfg: cfg
        hostname: vqfx-leaf-3
      - serial_number: "74583202585"
        supplemental_day_0_cfg: cfg
        hostname: vqfx-leaf-4
      - serial_number: "54613660016"
        supplemental_day_0_cfg: cfg
        hostname: vqfx-spine-1
```

### A.6.2 job-import-fabric.yaml
```
- resource_type: job
  resource:
    job_template_fq_name:
    - default-global-system-config
    - existing_fabric_onboard_template
    input:
      fabric_fq_name:
      - default-global-system-config
      - poc
      device_auth:
      - username: root
        password: Juniper
      overlay_ibgp_asn: 64512
      loopback_subnets:
      - 10.6.0.0/24
      management_subnets:
      - cidr: 10.6.8.11/32
      node_profiles:
      - node_profile_name: juniper-mx
      - node_profile_name: juniper-qfx10k
      - node_profile_name: juniper-qfx10k-lean
      - node_profile_name: juniper-qfx5120
      - node_profile_name: juniper-qfx5k
      - node_profile_name: juniper-qfx5k-lean
      - node_profile_name: juniper-srx
      import_configured: false
      enterprise_style: true
```

### A.6.3 job-assign-role.yaml
```
- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
    - default-global-system-config
    - vqfx-leaf-1
    dst:
    - dst_type: overlay-role
      dst_fqn:
      - default-global-system-config
      - crb-access
      attr: null
    - dst_type: physical-role
      dst_fqn:
      - default-global-system-config
      - leaf
      attr: null

- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
    - default-global-system-config
    - vqfx-leaf-2
    dst:
    - dst_type: overlay-role
      dst_fqn:
      - default-global-system-config
      - crb-access
      attr: null
    - dst_type: physical-role
      dst_fqn:
      - default-global-system-config
      - leaf
      attr: null

- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
    - default-global-system-config
    - vqfx-leaf-3
    dst:
    - dst_type: overlay-role
      dst_fqn:
      - default-global-system-config
      - crb-access
      attr: null
    - dst_type: physical-role
      dst_fqn:
      - default-global-system-config
      - leaf
      attr: null

- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
    - default-global-system-config
    - vqfx-leaf-4
    dst:
    - dst_type: overlay-role
      dst_fqn:
      - default-global-system-config
      - crb-access
      attr: null
    - dst_type: physical-role
      dst_fqn:
      - default-global-system-config
      - leaf
      attr: null

- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
    - default-global-system-config
    - vqfx-spine-1
    dst:
    - dst_type: overlay-role
      dst_fqn:
      - default-global-system-config
      - dc-gateway
      attr: null
    - dst_type: physical-role
      dst_fqn:
      - default-global-system-config
      - spine
      attr: null

- resource_type: job
  resource:
    job_template_fq_name:
    - default-global-system-config
    - role_assignment_template
    input:
      fabric_fq_name:
      - default-global-system-config
      - poc
      role_assignments:
      - device_fq_name:
        - default-global-system-config
        - vqfx-spine-1
        physical_role: spine
        routing_bridging_roles:
        - DC-Gateway
        - Route-Reflector
      - device_fq_name:
        - default-global-system-config
        - vqfx-leaf-1
        physical_role: leaf
        routing_bridging_roles:
        - CRB-Access
      - device_fq_name:
        - default-global-system-config
        - vqfx-leaf-2
        physical_role: leaf
        routing_bridging_roles:
        - CRB-Access
      - device_fq_name:
        - default-global-system-config
        - vqfx-leaf-3
        physical_role: leaf
        routing_bridging_roles:
        - CRB-Access
      - device_fq_name:
        - default-global-system-config
        - vqfx-leaf-4
        physical_role: leaf
        routing_bridging_roles:
        - CRB-Access
```

### A.6.4 ref-leaf-csn.yaml
```
- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
      - default-global-system-config
      - vqfx-leaf-1
    dst:
    - dst_type: virtual-router
      dst_fqn:
        - default-global-system-config
        - csn-1
      attr: null

- resource_type: reference
  operation: ADD
  resource:
    src_type: physical-router
    src_fqn:
      - default-global-system-config
      - vqfx-leaf-2
    dst:
    - dst_type: virtual-router
      dst_fqn:
        - default-global-system-config
        - csn-1
      attr: null
```

### A.6.5 vpg-bms-22.yaml
```
- resource_type: virtual-port-group
  resource_fq_name:
  - default-global-system-config
  - poc
  - bms-22
  parent_type: fabric
  resource:
    display_name: bms-22

- resource_type: virtual-machine-interface
  resource_fq_name:
  - default-domain
  - admin
  - bms-22-1-untagged
  parent_type: project
  resource:
    display_name: bms-22-1-untagged
    virtual_machine_interface_bindings:
      key_value_pair:
      - key: vnic_type
        value: baremetal
      - key: vif_type
        value: vrouter
      - key: profile
        value: "{\"local_link_information\":[{\"port_id\":\"xe-0/0/2\",\"switch_id\":\"xe-0/0/2\",\"switch_info\":\"vqfx-leaf-2\",\"fabric\":\"poc\"}]}"
      - key: tor_port_vlan_id
        value: "1"
      - key: vpg
        value: bms-22
    virtual_machine_interface_mac_addresses:
      mac_address:
      - 52:54:00:03:28:f0
    virtual_machine_interface_properties:
      sub_interface_vlan_tag: 0
    virtual_network_refs:
    - to:
      - default-domain
      - admin
      - red
```

