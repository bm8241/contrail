* [TOC](Multi-Cloud.md#toc)

## A.2 BIRD

### A.2.1 Protocols on private MC-GW
```
[root@mc-gw ~]# docker exec bird_bird_1 birdcl show protocol all
BIRD 1.6.6 ready.
name     proto    table    state  since       info
mast_mc  Pipe     master   up     2019-08-15  => t_mc
  Preference:     70
  Input filter:   REJECT
  Output filter:  ACCEPT
  Routes:         0 imported, 20 exported
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:           8883       8883          0          0          0
    Import withdraws:         1254          0        ---          0          0
    Export updates:           8883          0          0          2       8881
    Export withdraws:         1254          0        ---          0       1254

mc       Kernel   t_mc     up     2019-08-15  
  Preference:     10
  Input filter:   REJECT
  Output filter:  no_def_hub
  Routes:         0 imported, 18 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:           6642          0          1        ---       6641
    Export withdraws:          953        ---        ---        ---        953

main     Kernel   master   up     2019-08-15  
  Preference:     210
  Input filter:   import_local_net
  Output filter:  REJECT
  Routes:         1 imported, 0 exported, 2 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              5          0          3          0          2
    Import withdraws:            3          0        ---          5          1
    Export updates:           6642          5       6637        ---          0
    Export withdraws:          953        ---        ---        ---          0

device1  Device   master   up     2019-08-15  
  Preference:     240
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

direct1  Direct   master   up     2019-08-15  
  Preference:     240
  Input filter:   import_local_net
  Output filter:  REJECT
  Routes:         1 imported, 0 exported, 2 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:             36          0         34          0          2
    Import withdraws:           18          0        ---         51          1
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

bfd1     BFD      master   up     2019-08-15  
  Preference:     0
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

ospf1    OSPF     master   up     2019-08-15  Running
  Preference:     150
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         3 imported, 0 exported, 6 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:           2135          0          0          0       2135
    Import withdraws:          353          0        ---          0        353
    Export updates:           6643       2135       4508        ---          0
    Export withdraws:          953        ---        ---        ---          0

static1  Static   master   up     2019-08-15  
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  ACCEPT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

blackhole_noexport Static   master   up     2019-08-15  
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  ACCEPT
  Routes:         2 imported, 0 exported, 4 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              2          0          0          0          2
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

blackhole_export Static   master   up     2019-08-15  
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  ACCEPT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

gw1006502 BGP      master   up     18:21:47    Established   
  Preference:     100
  Input filter:   ACCEPT
  Output filter:  local_net
  Routes:         3 imported, 12 exported, 4 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              3          0          0          0          3
    Import withdraws:            0          0        ---          0          0
    Export updates:             68         39         17        ---         12
    Export withdraws:            0        ---        ---        ---          0
  BGP state:          Established
    Neighbor address: 100.65.0.2
    Neighbor AS:      65000
    Neighbor ID:      100.65.0.2
    Neighbor caps:    refresh enhanced-refresh restart-aware llgr-aware AS4 add-path-rx add-path-tx
    Session:          internal multihop route-reflector AS4 add-path-rx add-path-tx
    Source address:   100.65.0.1
    Hold timer:       149/240
    Keepalive timer:  22/80

tor10612254 BGP      master   up     2019-08-15  Established   
  Preference:     100
  Input filter:   import_tor
  Output filter:  local_net
  Routes:         7 imported, 6 exported, 12 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              7          0          0          0          7
    Import withdraws:            0          0        ---          0          0
    Export updates:           6640         19       2137        ---       4484
    Export withdraws:          953        ---        ---        ---        602
  BGP state:          Established
    Neighbor address: 10.6.12.254
    Neighbor AS:      64512
    Neighbor ID:      10.6.0.12
    Neighbor caps:    refresh restart-aware llgr-aware AS4
    Session:          external multihop AS4
    Source address:   10.6.12.81
    Hold timer:       76/90
    Keepalive timer:  16/30

gw1006503 BGP      master   up     18:21:46    Established   
  Preference:     100
  Input filter:   ACCEPT
  Output filter:  local_net
  Routes:         3 imported, 12 exported, 4 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              3          0          0          0          3
    Import withdraws:            0          0        ---          0          0
    Export updates:             68          3         17        ---         48
    Export withdraws:            0        ---        ---        ---          0
  BGP state:          Established
    Neighbor address: 100.65.0.3
    Neighbor AS:      65000
    Neighbor ID:      100.65.0.3
    Neighbor caps:    refresh enhanced-refresh restart-aware llgr-aware AS4 add-path-rx add-path-tx
    Session:          internal multihop route-reflector AS4 add-path-rx add-path-tx
    Source address:   100.65.0.1
    Hold timer:       150/240
    Keepalive timer:  11/80
```


### A.2.2 Routes on private MC-GW
```
[root@mc-gw ~]# docker exec bird_bird_1 birdcl show route
BIRD 1.6.6 ready.
0.0.0.0/0          via 10.6.8.254 on eth0 [main 2019-08-15] * (210)
                   via 100.64.0.2 on tap34-8-E6-2B [gw1006502 18:21:46 from 100.65.0.2] (100/15) [i]
                   via 100.64.0.3 on tap34-9-A7-A4 [gw1006503 18:21:45 from 100.65.0.3] (100/15) [i]
10.6.20.0/31       via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] * (100/0) [AS64021e]
10.6.30.0/31       via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] * (100/0) [AS64021e]
10.6.11.0/24       via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] * (100/0) [AS64011e]
10.6.12.0/24       dev vhost0 [direct1 2019-08-15] * (240)
                   via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] (100/0) [AS64512e]
10.11.0.0/25       via 100.64.0.2 on tap34-8-E6-2B [gw1006502 18:21:46 from 100.65.0.2] * (100/15) [i]
10.12.0.0/25       via 100.64.0.3 on tap34-9-A7-A4 [gw1006503 18:21:45 from 100.65.0.3] * (100/15) [i]
10.6.0.11/32       via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] * (100/0) [AS64011e]
10.6.0.21/32       via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] * (100/0) [AS64021e]
10.11.0.29/32      via 100.64.0.2 on tap34-8-E6-2B [gw1006502 18:21:46 from 100.65.0.2] * (100/15) [i]
100.65.0.0/16      blackhole [blackhole_noexport 2019-08-15] * (200)
100.64.0.0/16      blackhole [blackhole_noexport 2019-08-15] * (200)
100.65.0.1/32      dev lo [ospf1 2019-08-15] * I (150/0) [100.65.0.1]
10.12.0.23/32      via 100.64.0.3 on tap34-9-A7-A4 [gw1006503 18:21:45 from 100.65.0.3] * (100/15) [i]
100.65.0.2/32      via 100.64.0.2 on tap34-8-E6-2B [ospf1 19:04:08] * I (150/15) [100.65.0.2]
100.65.0.3/32      via 100.64.0.3 on tap34-9-A7-A4 [ospf1 18:21:35] * I (150/15) [100.65.0.3]
10.6.0.31/32       via 10.6.12.254 on vhost0 [tor10612254 2019-08-15] * (100/0) [AS64031e]
```


### A.2.3 Protocols on public MC-GW
```
[root@mc-gw-vpc-1 ~]# docker exec bird_bird_1 birdcl show protocol all
BIRD 1.6.6 ready.
name     proto    table    state  since       info
mast_mc  Pipe     master   up     2019-08-15  => t_mc
  Preference:     70
  Input filter:   REJECT
  Output filter:  ACCEPT
  Routes:         0 imported, 20 exported
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:           6885       6885          0          0          0
    Import withdraws:         1983          0        ---          0          0
    Export updates:           6885          0          0          3       6882
    Export withdraws:         1983          0        ---          0       1983

mc       Kernel   t_mc     up     2019-08-15  
  Preference:     10
  Input filter:   REJECT
  Output filter:  no_def_hub
  Routes:         0 imported, 19 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:           4733          0          1        ---       4732
    Export withdraws:         1531        ---        ---        ---       1531

main     Kernel   master   up     2019-08-15  
  Preference:     210
  Input filter:   import_local_net
  Output filter:  REJECT
  Routes:         1 imported, 0 exported, 2 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              2          0          1          0          1
    Import withdraws:            0          0        ---          1          0
    Export updates:           4579          4       4575        ---          0
    Export withdraws:         1531        ---        ---        ---          0

device1  Device   master   up     2019-08-15  
  Preference:     240
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

direct1  Direct   master   up     2019-08-15  
  Preference:     240
  Input filter:   import_local_net
  Output filter:  REJECT
  Routes:         1 imported, 0 exported, 2 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:             13          0         11          0          2
    Import withdraws:            1          0        ---         11          1
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

bfd1     BFD      master   up     2019-08-15  
  Preference:     0
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

ospf1    OSPF     master   up     2019-08-15  Running
  Preference:     150
  Input filter:   ACCEPT
  Output filter:  REJECT
  Routes:         3 imported, 0 exported, 6 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:           1053          0          0          0       1053
    Import withdraws:          179          0        ---          0        179
    Export updates:           4580       1053       3527        ---          0
    Export withdraws:         1531        ---        ---        ---          0

static1  Static   master   up     2019-08-15  
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  ACCEPT
  Routes:         0 imported, 0 exported, 0 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              0          0          0          0          0
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

blackhole_noexport Static   master   up     2019-08-15  
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  ACCEPT
  Routes:         2 imported, 0 exported, 4 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              2          0          0          0          2
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

blackhole_export Static   master   up     2019-08-15  
  Preference:     200
  Input filter:   ACCEPT
  Output filter:  ACCEPT
  Routes:         1 imported, 0 exported, 2 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:              1          0          0          0          1
    Import withdraws:            0          0        ---          0          0
    Export updates:              0          0          0        ---          0
    Export withdraws:            0        ---        ---        ---          0

gw1006501 BGP      master   up     18:22:27    Established   
  Preference:     100
  Input filter:   ACCEPT
  Output filter:  local_net
  Routes:         12 imported, 3 exported, 18 preferred
  Route change stats:     received   rejected   filtered    ignored   accepted
    Import updates:             12          0          0          0         12
    Import withdraws:            0          0        ---          0          0
    Export updates:             80         60         17        ---          3
    Export withdraws:            0        ---        ---        ---          0
  BGP state:          Established
    Neighbor address: 100.65.0.1
    Neighbor AS:      65000
    Neighbor ID:      100.65.0.1
    Neighbor caps:    refresh enhanced-refresh restart-aware llgr-aware AS4 add-path-rx add-path-tx
    Session:          internal multihop AS4 add-path-rx add-path-tx
    Source address:   100.65.0.2
    Hold timer:       218/240
    Keepalive timer:  54/80
```


### A.2.4 Routes on public MC-GW
```
[root@mc-gw-vpc-1 ~]# docker exec bird_bird_1 birdcl show route
BIRD 1.6.6 ready.
0.0.0.0/0          via 10.11.0.129 on eth0 [main 2019-08-15] * (210)
                   via 100.64.0.3 on tap1 [gw1006501 18:22:27 from 100.65.0.1] (100/15) [i]
                   via 100.64.0.1 on tap0 [gw1006501 18:22:27 from 100.65.0.1] (100/15) [i]
10.6.20.0/31       unreachable [gw1006501 18:22:27 from 100.65.0.1] * (100/-) [AS64021e]
10.6.30.0/31       unreachable [gw1006501 18:22:27 from 100.65.0.1] * (100/-) [AS64021e]
10.6.11.0/24       via 100.64.0.1 on tap0 [gw1006501 18:22:27 from 100.65.0.1] * (100/15) [AS64011e]
10.6.12.0/24       via 100.64.0.1 on tap0 [gw1006501 18:22:27 from 100.65.0.1] * (100/15) [i]
                   via 100.64.0.1 on tap0 [gw1006501 18:22:27 from 100.65.0.1] (100/15) [AS64512e]
10.11.0.0/25       dev vhost0 [direct1 2019-08-15] * (240)
10.12.0.0/25       via 100.64.0.3 on tap1 [gw1006501 18:22:27 from 100.65.0.1] * (100/15) [i]
10.6.0.11/32       unreachable [gw1006501 18:22:27 from 100.65.0.1] * (100/-) [AS64011e]
10.6.0.21/32       unreachable [gw1006501 18:22:27 from 100.65.0.1] * (100/-) [AS64021e]
10.11.0.29/32      blackhole [blackhole_export 2019-08-15] * (200)
100.65.0.0/16      blackhole [blackhole_noexport 2019-08-15] * (200)
100.64.0.0/16      blackhole [blackhole_noexport 2019-08-15] * (200)
100.65.0.1/32      via 100.64.0.1 on tap0 [ospf1 19:04:49] * I (150/15) [100.65.0.1]
10.12.0.23/32      via 100.64.0.3 on tap1 [gw1006501 18:22:27 from 100.65.0.1] * (100/15) [i]
100.65.0.2/32      dev lo [ospf1 2019-08-15] * I (150/0) [100.65.0.2]
100.65.0.3/32      via 100.64.0.3 on tap1 [ospf1 2019-08-15] * I (150/15) [100.65.0.3]
10.6.0.31/32       unreachable [gw1006501 18:22:27 from 100.65.0.1] * (100/-) [AS64031e]
```

### A.2.5 Configuration on private MC-GW
#### /etc/multicloud/bird/bird.conf
```
include "/etc/multicloud/bird/bird_header.inc";

log stderr  { info, remote, warning, error, auth, fatal, bug };

filter no_def_hub {
    if net = 0.0.0.0/0 then {
        if 0.0.0.0/0 ~ local_lan then {
            reject;
        }
    }
    if source = RTS_BGP then {
        krt_prefsrc = local_ip;
        accept;
    }
    accept;
}

filter local_net {
    if net ~ local_lan then {
        if source != RTS_BGP then {
                bgp_next_hop = local_lo;
                accept;
        }
        if bgp_origin=0 then {
            accept;
        } else {
            bgp_next_hop = local_lo;
            accept;
        }
    }
    if proto = "blackhole_export" then {
        bgp_next_hop = local_lo;
        accept;
    }

    if source = RTS_BGP then {
        accept;
    }
    reject;
}

filter import_local_net {
    if net ~ local_lan then accept;
    reject;
}

filter import_tor {
    bgp_origin=1;
    accept;
}

table t_mc;

protocol pipe mast_mc
{
  table master;
  peer table t_mc;
  import none;
  export all;
}

protocol kernel mc {
    table t_mc;
    scan time 60;
    import none;
    export filter no_def_hub;
    device routes;
    merge paths yes;
    kernel table 42;
}

protocol kernel main{
    preference 210;
    scan time 60;
    import filter import_local_net;
    learn;
    export none;
    merge paths yes;
}

protocol device {
    scan time 60;
}

protocol direct {
    import filter import_local_net;
}

protocol bfd {
    interface "vti*", "tun*", "tap*", "en*", "eth*", "vx-*", "vhost0" {
      interval 200ms;
      multiplier 5;
    };

    multihop {
      interval 500ms;
      multiplier 5;
    };
};

protocol ospf {
    import all;
    area 0 {
        interface "vx-*" {
            bfd on;
            cost 5;
            type pointopoint;
            hello 5;
            retransmit 2;
            wait 10;
            dead 20;
        };
        interface "vti*" {
            bfd on;
            cost 10;
            type pointopoint;
            hello 5;
            retransmit 2;
            wait 10;
            dead 20;
        };
        interface "tap*", "tun*" {
            bfd on;
            cost 15;
            type pointopoint;
            hello 5;
            retransmit 2;
            wait 10;
            dead 20;
        };
        interface "lo*" {
            stub;
        };
    };
}

#use this template when your role is GW (not BGP_RR)
template bgp GW {
    med metric on;
    connect retry time 10;
    error wait time 10, 120;
    local as 65000;
    source address local_lo;
    export filter local_net;
    bfd on;
    import all;
    password "bgp_secret";
    add paths yes;
}

#use this template when your role is BGP_RR (not GW)
template bgp BGP_RR {
    med metric on;
    error wait time 10, 120;
    local as 65000;
    source address local_lo;
    export filter local_net;
    rr client;
    bfd on;
    import all;
    password "bgp_secret";
    add paths yes;
}

#use this template when need to connect to TOR switch using eBGP
template bgp TOR {
    local as 65000;
    export filter local_net;
    password "contrail_secret";
    bfd on;
    multihop;
    import filter import_tor;
}

include "/etc/multicloud/bird/bird_footer.inc";
```

#### /etc/multicloud/bird/bird_header.inc 
```
define local_lo=100.65.0.1;
define local_lan=[ 10.6.11.0/24,10.6.12.0/24,0.0.0.0/0  ];
router id from 100.65.0.0/16;
define local_ip=10.6.12.81;
```

#### /etc/multicloud/bird/bird_footer.inc 
```
protocol static {
    # no support for single GW for multi-subnet
    # in release 5.1
    check link;
    export all;
}

protocol static blackhole_noexport {
    route 100.65.0.0/16 blackhole;
    route 100.64.0.0/16 blackhole;
    export all;
}

protocol static blackhole_export {
    check link;
    export all;
}

protocol bgp gw1006502 from BGP_RR {
    neighbor 100.65.0.2 as 65000;
}

protocol bgp tor10612254 from TOR {
    local 10.6.12.81 as 65000;
    neighbor 10.6.12.254 as 65012;
}
```

### A.2.6 Configuration on public MC-GW
#### /etc/multicloud/bird/bird.conf
```
include "/etc/multicloud/bird/bird_header.inc";

log stderr  { info, remote, warning, error, auth, fatal, bug };

filter no_def_hub {
    if net = 0.0.0.0/0 then {
        if 0.0.0.0/0 ~ local_lan then {
            reject;
        }
    }
    if source = RTS_BGP then {
        krt_prefsrc = local_ip;
        accept;
    }
    accept;
}

filter local_net {
    if net ~ local_lan then {
        if source != RTS_BGP then {
                bgp_next_hop = local_lo;
                accept;
        }
        if bgp_origin=0 then {
            accept;
        } else {
            bgp_next_hop = local_lo;
            accept;
        }
    }
    if proto = "blackhole_export" then {
        bgp_next_hop = local_lo;
        accept;
    }

    if source = RTS_BGP then {
        accept;
    }
    reject;
}

filter import_local_net {
    if net ~ local_lan then accept;
    reject;
}

filter import_tor {
    bgp_origin=1;
    accept;
}

table t_mc;

protocol pipe mast_mc
{
  table master;
  peer table t_mc;
  import none;
  export all;
}

protocol kernel mc {
    table t_mc;
    scan time 60;
    import none;
    export filter no_def_hub;
    device routes;
    merge paths yes;
    kernel table 42;
}

protocol kernel main{
    preference 210;
    scan time 60;
    import filter import_local_net;
    learn;
    export none;
    merge paths yes;
}

protocol device {
    scan time 60;
}

protocol direct {
    import filter import_local_net;
}

protocol bfd {
    interface "vti*", "tun*", "tap*", "en*", "eth*", "vx-*", "vhost0" {
      interval 200ms;
      multiplier 5;
    };

    multihop {
      interval 500ms;
      multiplier 5;
    };
};

protocol ospf {
    import all;
    area 0 {
        interface "vx-*" {
            bfd on;
            cost 5;
            type pointopoint;
            hello 5;
            retransmit 2;
            wait 10;
            dead 20;
        };
        interface "vti*" {
            bfd on;
            cost 10;
            type pointopoint;
            hello 5;
            retransmit 2;
            wait 10;
            dead 20;
        };
        interface "tap*", "tun*" {
            bfd on;
            cost 15;
            type pointopoint;
            hello 5;
            retransmit 2;
            wait 10;
            dead 20;
        };
        interface "lo*" {
            stub;
        };
    };
}

#use this template when your role is GW (not BGP_RR)
template bgp GW {
    med metric on;
    connect retry time 10;
    error wait time 10, 120;
    local as 65000;
    source address local_lo;
    export filter local_net;
    bfd on;
    import all;
    password "bgp_secret";
    add paths yes;
}

#use this template when your role is BGP_RR (not GW)
template bgp BGP_RR {
    med metric on;
    error wait time 10, 120;
    local as 65000;
    source address local_lo;
    export filter local_net;
    rr client;
    bfd on;
    import all;
    password "bgp_secret";
    add paths yes;
}

#use this template when need to connect to TOR switch using eBGP
template bgp TOR {
    local as 65000;
    export filter local_net;
    password "contrail_secret";
    bfd on;
    multihop;
    import filter import_tor;
}

include "/etc/multicloud/bird/bird_footer.inc";
```

#### /etc/multicloud/bird/bird_header.inc 
```
define local_lo=100.65.0.2;
define local_lan=[ 10.11.0.0/24,0.0.0.0/0  ];
router id from 100.65.0.0/16;
define local_ip=10.11.0.162;
```

#### /etc/multicloud/bird/bird_footer.inc 
```
protocol static {
    # no support for single GW for multi-subnet
    # in release 5.1
    check link;
    export all;
}

protocol static blackhole_noexport {
    route 100.65.0.0/16 blackhole;
    route 100.64.0.0/16 blackhole;
    export all;
}

protocol static blackhole_export {
    check link;
    route local_ip/32 blackhole;
    export all;
}

protocol bgp gw1006501 from GW {
    neighbor 100.65.0.1 as 65000;
}
```

