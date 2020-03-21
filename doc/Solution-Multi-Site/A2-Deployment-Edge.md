
leaf-11
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.11/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.20.1/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.11/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.11
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65011
set groups underlay protocols bgp group underlay neighbor 10.6.20.0 peer-as 65021
set apply-groups underlay
set system host-name vqfx-leaf-11
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.11/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set forwarding-options storm-control-profiles default all
```

leaf-12
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.12/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.20.3/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.12/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.12
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65012
set groups underlay protocols bgp group underlay neighbor 10.6.20.2 peer-as 65021
set apply-groups underlay
set system host-name vqfx-leaf-12
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.12/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set forwarding-options storm-control-profiles default all
```

leaf-13
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.13/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.20.5/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.13/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.13
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65013
set groups underlay protocols bgp group underlay neighbor 10.6.20.4 peer-as 65021
set apply-groups underlay
set system host-name vqfx-leaf-13
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.13/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set forwarding-options storm-control-profiles default all
```

leaf-14
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.14/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.20.7/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.14/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.14
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65014
set groups underlay protocols bgp group underlay neighbor 10.6.20.6 peer-as 65021
set apply-groups underlay
set system host-name vqfx-leaf-14
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.14/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set forwarding-options storm-control-profiles default all
```

spine-21
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.21/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.30.0/31
set groups underlay interfaces xe-0/0/2 unit 0 family inet address 10.6.20.0/31
set groups underlay interfaces xe-0/0/3 unit 0 family inet address 10.6.20.2/31
set groups underlay interfaces xe-0/0/4 unit 0 family inet address 10.6.20.4/31
set groups underlay interfaces xe-0/0/5 unit 0 family inet address 10.6.20.6/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.21/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.21
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay cluster 10.6.0.21
set groups underlay protocols bgp group underlay local-as 65021
set groups underlay protocols bgp group underlay neighbor 10.6.30.1 peer-as 65031
set groups underlay protocols bgp group underlay neighbor 10.6.20.1 peer-as 65011
set groups underlay protocols bgp group underlay neighbor 10.6.20.3 peer-as 65012
set groups underlay protocols bgp group underlay neighbor 10.6.20.5 peer-as 65013
set groups underlay protocols bgp group underlay neighbor 10.6.20.7 peer-as 65014
set apply-groups underlay
set system host-name vqfx-spine-21
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.21/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set forwarding-options storm-control-profiles default all
```

leaf-111
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.111/32
set groups underlay interfaces xe-0/0/0 unit 0 family inet address 10.6.30.11/31
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.111/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set groups underlay routing-options route-distinguisher-id 10.6.0.111
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65111
set groups underlay protocols bgp group underlay neighbor 10.6.30.10 peer-as 65131
set apply-groups underlay
set system host-name vqfx-leaf-111
set system root-authentication encrypted-password "$6$l.zi0dZZ$EsvJo1Em2F0trWksE61MAAZAqgTx21xO0t0fam4rOgLqU8H6wb3O6yE.9eGkWEKN4hGPm2UYPdf4sTFf22afc1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system services ssh root-login allow
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces em0 unit 0 family inet address 10.6.8.111/24
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set forwarding-options storm-control-profiles default all
```

gw-31
```
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.31/32
set groups underlay interfaces ge-0/0/2 unit 0 family inet address 10.6.30.1/31
set groups underlay interfaces ge-0/0/0 unit 0 family inet address 172.16.0.1/24
set groups underlay interfaces ge-0/0/0 unit 0 family mpls
set groups underlay routing-options route-distinguisher-id 10.6.0.31
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65031
set groups underlay protocols bgp group underlay neighbor 10.6.30.0 peer-as 65021
set groups underlay protocols ospf area 0.0.0.0 interface lo0.0
set groups underlay protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set groups underlay protocols ldp interface ge-0/0/0.0
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.31/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set apply-groups underlay
set system root-authentication encrypted-password "$6$.eu1H0ZX$K3iXOzGi2WyIJbFaRxuVzjlK/W/3y.11o.3h8.rUbldqHi7akVrsQtj.HpOkqEbMIVQHiTpBzlX7/fCFJ27kJ1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system host-name vmx-31
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
set interfaces fxp0 unit 0 family inet address 10.6.8.31/24
```

gw-131
```
set groups underlay interfaces ge-0/0/0 unit 0 family inet address 172.16.0.3/24
set groups underlay interfaces ge-0/0/0 unit 0 family mpls
set groups underlay interfaces lo0 unit 0 family inet address 10.6.0.131/32
set groups underlay interfaces ge-0/0/2 unit 0 family inet address 10.6.30.10/31
set groups underlay interfaces ge-0/0/3 unit 0 family inet address 10.6.30.12/31
set groups underlay routing-options route-distinguisher-id 10.6.0.131
set groups underlay protocols bgp group underlay type external
set groups underlay protocols bgp group underlay family inet unicast
set groups underlay protocols bgp group underlay export underlay-export
set groups underlay protocols bgp group underlay local-as 65131
set groups underlay protocols bgp group underlay neighbor 10.6.30.11 peer-as 65111
set groups underlay protocols bgp group underlay neighbor 10.6.30.13 peer-as 65112
set groups underlay protocols ospf area 0.0.0.0 interface lo0.0
set groups underlay protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set groups underlay protocols ldp interface ge-0/0/0.0
set groups underlay policy-options policy-statement underlay-export term t1 from protocol direct
set groups underlay policy-options policy-statement underlay-export term t1 from route-filter 10.6.0.131/32 exact
set groups underlay policy-options policy-statement underlay-export term t1 then accept
set apply-groups underlay
set system root-authentication encrypted-password "$6$.eu1H0ZX$K3iXOzGi2WyIJbFaRxuVzjlK/W/3y.11o.3h8.rUbldqHi7akVrsQtj.HpOkqEbMIVQHiTpBzlX7/fCFJ27kJ1"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDlpjEdmQaKZBc7d6yYzQrMxwvOcU4rUy07S8/Ms4gq9v17QNjQ/+B9DEzPy7zuJSD7g0J3sP9u91tMDxLPa06Ia2nteTmw8yIncmH4gbLougY9ju1a2aWy9iZeez5qFP32Knw+8NW4AemGoi6ymAqwXyuZ8bnP+tO3bIcu1ycrq/HPAgo6/v7EL/DnjYlssxjt3uZ6CZioDX9+hQ9jAprY2B/b6kVPvOEc/xpV3GYiaK/Gj4W93dZ9a6z9M5m6xewwUUUcz6EyJ0kkF8BeiozbkY/x8E33uNXa99wroqQZnyOzf0i+4WY02IlrcyX0NGzw9IzcHfhega7TXt5TkYKV contrail-poc"
set system host-name vmx-131
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set chassis fpc 0 pic 0 tunnel-services bandwidth 1g
set interfaces fxp0 unit 0 family inet address 10.6.8.131/24
```

