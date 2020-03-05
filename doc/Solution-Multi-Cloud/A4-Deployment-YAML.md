* [TOC](Multi-Cloud.md#toc)

## A.4 Deployment YAML

### A.4.1 Kubernetes cluster
Cloud topology.yaml
```
- provider: aws
  organization:
  project:
  prebuild: r1912
  regions:
    - name: us-west-1
      vpc:
        - name: vpc-1-09019c77-0b22-47a9-bc6d-d0b3d11f2d9c
          cidr_block: 10.11.0.0/16
          subnets:
            - name: mc-gw-30a8e2a3-f435-42c4-9bff-3c7dcc3ca50f
              cidr_block: 10.11.0.0/23
              availability_zone: b
          security_groups:
              - name: egress-87adffeb-51c3-4772-b3fd-58655a940968
                egress:
                  from_port: 0
                  to_port: 0
                  protocol: -1
                  cidr_blocks:
                  - 0.0.0.0/0
              - name: ingress-df1e12ae-56af-4aa4-aaaf-e46232b45abe
                ingress:
                  from_port: 0
                  to_port: 0
                  protocol: -1
                  cidr_blocks:
                  - 0.0.0.0/0
          instances:
            - name: mc-gw-vpc-1
              roles:
                - gateway
              provision: true
              username: ec2-user
              os: rhel7
              instance_type: t2.xlarge
              subnets: mc-gw-30a8e2a3-f435-42c4-9bff-3c7dcc3ca50f
              availability_zone: b
              protocols_mode:
                - ssl_server
              security_groups:
                - egress-87adffeb-51c3-4772-b3fd-58655a940968
                - ingress-df1e12ae-56af-4aa4-aaaf-e46232b45abe
            - name: node-1-vpc-1
              roles:
                - compute_node
              provision: true
              username: ec2-user
              os: rhel7
              instance_type: t2.xlarge
              subnets: mc-gw-30a8e2a3-f435-42c4-9bff-3c7dcc3ca50f
              availability_zone: b
              security_groups:
                - egress-87adffeb-51c3-4772-b3fd-58655a940968
                - ingress-df1e12ae-56af-4aa4-aaaf-e46232b45abe
```

On-prem topology.yaml
```
- provider: onprem
  organization:
  project:
  instances:
    - name: node-1
      public_ip: 10.6.8.67
      private_ip: 10.6.11.67
      interface: eth1
      provision: true
      username: root
      password: c0ntrail123
      roles:
        - compute_node
      gateway: 10.6.11.254
    - name: cmaster-1
      public_ip: 10.6.8.61
      private_ip: 10.6.11.61
      interface: eth1
      provision: false
      username: root
      password: c0ntrail123
      roles:
        - controller
        - k8s_master
    - name: node-2
      public_ip: 10.6.8.68
      private_ip: 10.6.11.68
      interface: eth1
      provision: true
      username: root
      password: c0ntrail123
      roles:
        - compute_node
      gateway: 10.6.11.254
    - name: mc-gw
      public_ip: 10.6.8.81
      private_ip: 10.6.12.81
      interface: eth1
      provision: true
      username: root
      password: c0ntrail123
      services:
        - bgp_rr
      roles:
        - gateway
      private_subnet:
        - 10.6.12.0/24
        - 10.6.11.0/24
      protocols_mode:
        - ssl_client
      gateway: 10.6.12.254
    - name: vqfx-leaf-2-7cec4b18-d8f4-4e3e-ba18-2ade773c91be
      public_ip: 10.6.8.12
      private_ip: 10.6.12.254
      private_subnet:
        - 10.6.12.0/24
      roles:
        - tor
      provision: true
      username: root
      password: Juniper
      interface:
        - irb.12
      AS: 65012
      protocols_mode:
        - bgp
```

### A.4.2 MC-GW

#### A.4.2.1 topology.yaml
```
- provider: onprem
  organization:
  project:
  instances:
    - name: mc-gw
      public_ip: 10.6.8.81
      private_ip: 10.6.12.81
      interface: eth1
      provision: true
      username: root
      password: c0ntrail123
      roles:
        - gateway
      private_subnet:
        - 10.6.12.0/24
        - 10.6.11.0/24
      protocols_mode:
        - ssl_client
      gateway: 10.6.12.254

- provider: aws
  organization:
  project:
  prebuild: r1912
  regions:
    - name: us-west-1
      vpc:
        - name: vpc-1-aws
          cidr_block: 10.11.0.0/16
          subnets:
            - name: mc-gw
              cidr_block: 10.11.0.0/23
              availability_zone: b
          security_groups:
              - name: egress
                egress:
                  from_port: 0
                  to_port: 0
                  protocol: -1
                  cidr_blocks:
                  - 0.0.0.0/0
              - name: ingress
                ingress:
                  from_port: 0
                  to_port: 0
                  protocol: -1
                  cidr_blocks:
                  - 0.0.0.0/0
          instances:
            - name: mc-gw-vpc-1
              roles:
                - gateway
              provision: true
              username: ec2-user
              os: rhel7
              instance_type: t2.xlarge
              subnets: mc-gw
              availability_zone: b
              protocols_mode:
                - ssl_server
              security_groups:
                - egress
                - ingress
```

#### A4.2.2 secret.yaml
```
public_key:
  name: contrail-poc
  value: __public_key__
aws_access_key: __aws_access_key__
aws_secret_key: __aws_secret_key__
authorized_registries:
  - registry: "hub.juniper.net/contrail"
    tag: "1912.32"
    username: __registry_username__
    password: __registry_password__
```

