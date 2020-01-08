* [TOC](Multi-Cloud.md#toc)

## A.1 ToR

### A.1.1 ToR configuration
```
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ type external
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ multihop
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ local-address 10.6.12.254
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ hold-time 90
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ keep all
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ family inet unicast
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ authentication-key "$9$QcRzn6AleW-VY8XUHq.zFuO1IclwY4ZDkWLNbY2UD.P5T6A"
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ export contrail_multi_cloud_export_BGP
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ peer-as 65000
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ local-as 64512
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ bfd-liveness-detection version automatic
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ bfd-liveness-detection minimum-interval 500
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ bfd-liveness-detection multiplier 5
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ bfd-liveness-detection session-mode multihop
set groups __contrail_multi_cloud_BGP__ protocols bgp group __contrail_multi_cloud__ neighbor 10.6.12.81
set groups __contrail_multi_cloud_BGP__ policy-options policy-statement contrail_multi_cloud_export_BGP term t1 from route-filter 10.6.12.0/24 exact
set groups __contrail_multi_cloud_BGP__ policy-options policy-statement contrail_multi_cloud_export_BGP term t1 then accept
```

### A.1.2 ToR route table
```
root@vqfx-leaf-2> show route table inet.0 

inet.0: 20 destinations, 24 routes (19 active, 0 holddown, 2 hidden)
+ = Active Route, - = Last Active, * = Both

10.6.0.11/32       *[BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 64011 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.0.12/32       *[Direct/0] 3w2d 20:46:04
                    > via lo0.0
10.6.0.21/32       *[BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.0.31/32       *[BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 64031 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.8.0/24        *[Direct/0] 3w2d 20:48:40
                    > via em0.0
                    [BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.8.12/32       *[Local/0] 3w2d 20:48:40
                      Local via em0.0
10.6.11.0/24       *[BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 64011 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.12.0/24       *[Direct/0] 3w2d 20:46:04
                    > via irb.12
10.6.12.254/32     *[Local/0] 3w2d 20:46:04
                      Local via irb.12
10.6.20.0/31       *[BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.20.2/31       *[Direct/0] 3w2d 20:46:04
                    > via xe-0/0/0.0
                    [BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.6.20.3/32       *[Local/0] 3w2d 20:46:04
                      Local via xe-0/0/0.0
10.6.30.0/31       *[BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
10.11.0.0/25       *[BGP/170] 00:30:06, localpref 100
                      AS path: 65000 I, validation-state: unverified
                    > to 10.6.12.81 via irb.12
10.11.0.29/32      *[BGP/170] 00:30:06, localpref 100
                      AS path: 65000 I, validation-state: unverified
                    > to 10.6.12.81 via irb.12
10.12.0.0/25       *[BGP/170] 00:30:07, localpref 100
                      AS path: 65000 I, validation-state: unverified
                    > to 10.6.12.81 via irb.12
10.12.0.23/32      *[BGP/170] 00:30:07, localpref 100
                      AS path: 65000 I, validation-state: unverified
                    > to 10.6.12.81 via irb.12
169.254.0.0/24     *[Direct/0] 3w2d 20:48:40
                    > via em1.0
                    [BGP/170] 1w4d 14:17:30, localpref 100
                      AS path: 64021 I, validation-state: unverified
                    > to 10.6.20.2 via xe-0/0/0.0
169.254.0.2/32     *[Local/0] 3w2d 20:48:40
                      Local via em1.0
```

