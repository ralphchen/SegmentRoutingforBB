#  SR-TE LSP

## 1. 基于COMMUNITY，选择LSP

![image-20200323003612152](img/image-20200323003612152.png)

在CR5上，定义了，两条non-color SR-TE LSP

```
[edit protocols source-packet-routing]
ctrip@CR5# show 
lsp-external-controller pccd;
maximum-segment-list-depth 16;
segment-list LIST2 {
    hop1 label 34002;
    hop2 label 34003;
    hop3 label 34004;
}
segment-list LST_CR4_2 {
    hop1 label 34003;
    hop2 label 34002;
    hop3 label 34004;
}


source-routing-path toCR4_ECMP {
    to 4.4.4.4;
    preference 1;
    primary {
        LIST2;
    }
}
source-routing-path toCR4_2 {
    to 4.4.4.4;
    preference 1;
    primary {
        LST_CR4_2;
    }
}
```



定义了一条policy-statement

community 为14:14和114:114的，分别会调用不同的LSP:

```
ctrip@CR5# run show configuration policy-options 
policy-statement LSP_SLCT {
    term 1 {
        from community comm14
        then {
            install-nexthop lsp toCR4_ECMP;
            accept;
        }
    }
    term 2 {
        from community comm114;
        then {
            install-nexthop lsp toCR4_2;
            accept;
        }
    }
}

community comm114 members 114:114;
community comm14 members 14:14;
```



查看路由表，可以发现到14.14.14.14/32和 114.114.114.14/32 分别选择了不同的路径：

![image-20200323004234415](img/image-20200323004234415.png)

## 

## 2. admin group



CR5 -CR2 的两条链路metric为1和5，拓扑中其余链路metric均为10

将CR2-CR5的链路设置为admin-group green

CR5定义一跳到CR2的LSP，约束条件为exclude green，可以看到，这条隧道是绕开了和CR2的直连链路。而是经由CR3到达。

![image-20200321003103685](img/image-20200321003103685.png)

```
set protocols isis traffic-engineering igp-topology
set protocols isis traffic-engineering advertisement always

set protocols mpls admin-groups green 1
set protocols mpls interface xe-0/1/2.0 admin-group green
set protocols mpls interface xe-0/1/3.0 admin-group green

set protocols source-packet-routing compute-profile non_hadoop admin-group exclude green
set protocols source-packet-routing source-routing-path CR5-CR2 to 2.2.2.2
set protocols source-packet-routing source-routing-path CR5-CR2 primary pri_path compute non_hadoop
```



```
ctrip@CR5# run show spring-traffic-engineering lsp name  ？ detail 
Name: CR5-CR2
Tunnel-source: Static configuration
To: 2.2.2.2
State: Up
Telemetry statistics:
Sensor-name: ingress-CR5-CR2, Id: 3758096387
  Path: pri_path
  Outgoing interface: NA
  Auto-translate status: Disabled Auto-translate result: N/A
  Compute Status:Enabled , Compute Result:success , Compute-Profile Name:non_hadoop
  Total number of computed paths: 1
    Computed-path-index: 1
  BFD status: N/A BFD name: N/A
    computed segments count: 2
      computed segment : 1 (computed-node-segment): 
        node segment label: 17003
        router-id: 3.3.3.3
      computed segment : 2 (computed-node-segment): 
        node segment label: 17002
        router-id: 2.2.2.2
```





问题： 

到第一跳 3.3.3.3，是否是负载均衡