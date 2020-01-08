* [TOC](Multi-Cloud.md#toc)

## A.3 Host

### A.3.1 Host route on private MC-GW
```
[root@mc-gw ~]# ip route
default via 10.6.8.254 dev eth0 
10.6.8.0/24 dev eth0 proto kernel scope link src 10.6.8.81 
10.6.12.0/24 dev vhost0 proto kernel scope link src 10.6.12.81 
10.32.0.0/12 via 10.6.12.254 dev vhost0 
100.64.0.2 dev tap34-8-E6-2B proto kernel scope link src 100.64.0.1 
100.64.0.3 dev tap34-9-A7-A4 proto kernel scope link src 100.64.0.1 
198.18.0.0/24 dev docker0 proto kernel scope link src 198.18.0.1 

[root@mc-gw ~]# ip rule
0:      from all lookup local 
1700:   not from all iif lo lookup 42 
1701:   from 100.65.0.0/16 iif lo lookup 42 
1702:   from all to 100.65.0.0/16 iif lo lookup 42 
1703:   from all to 10.6.11.61 iif lo lookup 42 
32766:  from all lookup main 
32767:  from all lookup default 

[root@mc-gw ~]# ip route show table 42
10.6.0.11 via 10.6.12.254 dev vhost0 proto bird src 10.6.12.81 
10.6.0.21 via 10.6.12.254 dev vhost0 proto bird src 10.6.12.81 
10.6.0.31 via 10.6.12.254 dev vhost0 proto bird src 10.6.12.81 
10.6.11.0/24 via 10.6.12.254 dev vhost0 proto bird src 10.6.12.81 
10.6.12.0/24 dev vhost0 proto bird scope link 
10.6.20.0/31 via 10.6.12.254 dev vhost0 proto bird src 10.6.12.81 
10.6.30.0/31 via 10.6.12.254 dev vhost0 proto bird src 10.6.12.81 
10.11.0.0/25 via 100.64.0.2 dev tap34-8-E6-2B proto bird src 10.6.12.81 
10.11.0.29 via 100.64.0.2 dev tap34-8-E6-2B proto bird src 10.6.12.81 
10.12.0.0/25 via 100.64.0.3 dev tap34-9-A7-A4 proto bird src 10.6.12.81 
10.12.0.23 via 100.64.0.3 dev tap34-9-A7-A4 proto bird src 10.6.12.81 
blackhole 100.64.0.0/16 proto bird 
blackhole 100.65.0.0/16 proto bird 
100.65.0.1 dev lo proto bird scope link 
100.65.0.2 via 100.64.0.2 dev tap34-8-E6-2B proto bird 
100.65.0.3 via 100.64.0.3 dev tap34-9-A7-A4 proto bird
```


### A.3.2 Host route on public MC-GW
```
[root@mc-gw-vpc-1 ~]# ip route
default via 10.11.0.129 dev eth0 proto dhcp metric 100 
10.11.0.0/25 dev vhost0 proto kernel scope link src 10.11.0.29 
10.11.0.128/25 dev eth0 proto kernel scope link src 10.11.0.167 metric 100 
10.32.0.0/12 via 10.11.0.1 dev vhost0 
100.64.0.1 dev tap0 proto kernel scope link src 100.64.0.2 
100.64.0.3 dev tap1 proto kernel scope link src 100.64.0.2 
198.18.0.0/24 dev docker0 proto kernel scope link src 198.18.0.1 

[root@mc-gw-vpc-1 ~]# ip rule
0:      from all lookup local 
1700:   not from all iif lo lookup 42 
1701:   from 100.65.0.0/16 iif lo lookup 42 
1702:   from all to 100.65.0.0/16 iif lo lookup 42 
1703:   from all to 10.6.11.61 iif lo lookup 42 
32766:  from all lookup main 
32767:  from all lookup default 

[root@mc-gw-vpc-1 ~]# ip route show table 42
unreachable 10.6.0.11 proto bird src 10.11.0.29 
unreachable 10.6.0.21 proto bird src 10.11.0.29 
unreachable 10.6.0.31 proto bird src 10.11.0.29 
10.6.11.0/24 via 100.64.0.1 dev tap0 proto bird src 10.11.0.29 
10.6.12.0/24 via 100.64.0.1 dev tap0 proto bird src 10.11.0.29 
unreachable 10.6.20.0/31 proto bird src 10.11.0.29 
unreachable 10.6.30.0/31 proto bird src 10.11.0.29 
10.11.0.0/25 dev vhost0 proto bird scope link 
blackhole 10.11.0.29 proto bird 
10.12.0.0/25 via 100.64.0.3 dev tap1 proto bird src 10.11.0.29 
10.12.0.23 via 100.64.0.3 dev tap1 proto bird src 10.11.0.29 
blackhole 100.64.0.0/16 proto bird 
blackhole 100.65.0.0/16 proto bird 
100.65.0.1 via 100.64.0.1 dev tap0 proto bird 
100.65.0.2 dev lo proto bird scope link 
100.65.0.3 via 100.64.0.3 dev tap1 proto bird 
```

