

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