

# L2VPN  over SR

测试目的：

验证多厂商路由器对于L2VPN over SR的兼容性功能。

思科和Juniper路由器做PE，P为路由反射器，反射L2VPN EVPN路由

以下配置，基于vlan-based配置模式

PE (CISCO)  配置，只显示EVPN必须配置

```
evpn
 evi 55
  bgp
   rd 2000:2000
   route-target import 2000:2000
   route-target export 2000:2000
  !
  advertise-mac
  !
 !
!

l2vpn
 bridge group EVPN_ALL_ACTIVE
  bridge-domain EVPN_55
   interface Bundle-Ether1001.201
   !
   evi 55
   !
  !
 !
!
interface Bundle-Ether1001.201 l2transport
 encapsulation dot1q 201
 rewrite ingress tag pop 1 symmetric #一定要配置
!


router bgp 65510	
 address-family link-state link-state
 
  neighbor 2.2.2.2
  address-family l2vpn evpn
   soft-reconfiguration inbound always
  !
```



验证：

```
RP/0/RP0/CPU0:PE1#show l2vpn forwarding bridge-domain EVPN_ALL_ACTIVE:EVPN_55 mac-address location 0/0/CPU0 
Wed Jan 22 14:18:37.116 UTC
 To Resynchronize MAC table from the Network Processors, use the command...
    l2vpn resynchronize forwarding mac-address-table location <r/s/i>

Mac Address    Type    Learned from/Filtered on    LC learned Resync Age/Last Change Mapped to       
-------------- ------- --------------------------- ---------- ---------------------- --------------  
000c.29e3.136a dynamic Gi0/0/0/26.55               N/A        01 Jan 00:00:00        N/A             
000c.292d.985e EVPN    BD id: 1                    N/A        N/A                    N/A             
f04b.3aef.c67c EVPN    BD id: 1                    N/A        N/A                    N/A             
```

```
RP/0/RP0/CPU0:PE1#show evpn evi vpn-id 55 mac
Thu Jan 23 04:18:03.306 UTC

VPN-ID     Encap  MAC address    IP address                               Nexthop                                 Label   
---------- ------ -------------- ---------------------------------------- --------------------------------------- --------
55         MPLS   000c.292d.985e ::                                       5.5.5.5                                 86      
55         MPLS   000c.292d.985e 111.111.111.15                           5.5.5.5                                 86      
55         MPLS   000c.29e3.136a ::                                       GigabitEthernet0/0/0/26.55              24138   
55         MPLS   f04b.3aef.c67c ::                                       2.2.2.2                                 24014   
```

```
RP/0/RP0/CPU0:PE1#show bgp l2vpn evpn bridge-domain EVPN_55 route-type 2
Thu Jan 23 04:20:47.950 UTC
BGP router identifier 1.1.1.1, local AS number 65510
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 0
BGP main routing table version 248
BGP NSR Initial initsync version 1 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1000:1000 (default for vrf EVPN_55)
*>i[2][0][48][000c.292d.985e][0]/104
                      5.5.5.5                       100      0 i
*>i[2][0][48][000c.292d.985e][32][111.111.111.15]/136
                      5.5.5.5                       100      0 i
*> [2][0][48][000c.29e3.136a][0]/104
                      0.0.0.0                                0 i
*>i[2][0][48][f04b.3aef.c67c][0]/104
                      2.2.2.2                       100      0 i

Processed 4 prefixes, 4 paths
```



PE (juniper配置）

```
instance-type evpn;
protocols {
    evpn {
        no-arp-suppression;
    }
}
vlan-id none;
interface xe-0/1/7.100;
route-distinguisher 5001:5001;
vrf-target target:2000:2000;


[edit interfaces xe-0/1/7]
ctrip@PE5# show 
flexible-vlan-tagging;
encapsulation flexible-ethernet-services;
gigether-options {
    auto-negotiation;
    speed 1g;
}
unit 100 {
    encapsulation vlan-bridge;
    vlan-id 100;
}
```

验证：

```
ctrip@PE5> show route forwarding-table table evpn-l2    
Routing table: evpn-l2.evpn
EVPN:
Enabled protocols: ACKed by all peers, EVPN, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    dscd      611     1
00:0c:29:2d:98:5e/48 user     0                  ucst      655     4 xe-0/1/7.100
00:0c:29:e3:13:6a/48 user     0                  indr  1048594     4
                                                 ulst  1048588     2
                              10.3.5.3          Push 24138, Push 17001(top)      642     2 ae0.101
                              10.2.5.2          Push 24138, Push 17001(top)      613     2 xe-0/1/1.101
                              10.5.6.6          Push 24138, Push 17001(top)      640     2 xe-0/1/4.0
0x30003/51         user     0                    comp      664     2
f0:4b:3a:ef:c6:7c/48 user     0                  indr  1048597     3
                                                 ulst  1048596     2
                              10.2.5.2          Push 24014      665     2 xe-0/1/1.101
                              10.3.5.3          Push 24014, Push 17002(top)      671     2 ae0.101
xe-0/1/7.100       intf     0                    ucst      655     4 xe-0/1/7.100
0x30002/51         user     0                    comp      663     2
0x30001/51         user     0                    comp      662     2
```



P (CISCO)配置：

```
router bgp 65510
 address-family l2vpn evpn
 !
 neighbor 1.1.1.1
  remote-as 65510
  update-source Loopback0
  address-family l2vpn evpn
   route-reflector-client
  !
 !
```



P(Juniper)配置：

```
ctrip@CR6# show 
group vpnv4 {
    type internal;
    local-address 6.6.6.6;
    family inet-vpn {
        unicast;
    }
    family evpn {
        signaling;
    }
    cluster 6.6.6.6;
    neighbor 1.1.1.1;
    neighbor 4.4.4.4;
    neighbor 5.5.5.5;
}
```

