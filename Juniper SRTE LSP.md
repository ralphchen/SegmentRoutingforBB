#  SR-TE LSP

## 1. admin group



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
admin@CR5# run show spring-traffic-engineering lsp name  ？ detail 
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







到第一跳 3.3.3.3，是否是负载均衡



## 2. PRI_PATH之间的负载均衡



在一条LSP中，配置了多条primary path，相同权重

```
admin@CR5> show configuration protocols source-packet-routing 
segment-list SGL_CR4_24 {
    hop1 label 34002;
    hop2 label 34004;
}
segment-list SGL_CR4_34 {
    hop1 label 34003;
    hop2 label 34004;
}
source-routing-path toCR4_ECMP {
    to 4.4.4.4;
    preference 1;
    primary {
        SGL_CR4_24 weight 50;
        SGL_CR4_34 weight 50;
    }
}
```



定义多路径解析的policy-statement

```
admin@CR5> show configuration policy-options 

policy-statement mpath-resolv {
    term 1 {
        then multipath-resolve;
    }
}
```



在routing option中对bgp vpn路由解析，使用多条LSP路径

```
admin@CR5> show configuration routing-options 
resolution {
    rib bgp.l3vpn.0 {
        import mpath-resolv;
    }
}
```



查看路由表可以看到在两条LSP之间进行了50%的负载均衡

```
admin@CR5> show route 114.114.114.14 active-path detail | match "entri|weight"    
114.114.114.14/32 (2 entries, 1 announced)
        Next hop: ELNH Address 0x7e270ec weight 0x1 balance 50%, selected
                Next hop: 10.3.5.3 via xe-0/1/0.0 weight 0x1
                Next hop: 10.33.55.3 via xe-0/1/1.0 weight 0x1
        Next hop: ELNH Address 0x7e2717c weight 0x1 balance 50%
                Next hop: 10.2.5.2 via xe-0/1/2.0 weight 0x1
                Next hop: 10.22.55.2 via xe-0/1/3.0 weight 0xf000
```



如果没有应用mpath-resolv这条解析策略：

```
admin@CR5# run show route 114.114.114.14 active-path detail | match "entr|weight" 
0.0.0.0/0 (1 entry, 1 announced)
114.114.114.14/32 (2 entries, 1 announced)
                Next hop: 10.3.5.3 via xe-0/1/0.0 weight 0x1, selected
                Next hop: 10.33.55.3 via xe-0/1/1.0 weight 0x1, selected
```



## 
