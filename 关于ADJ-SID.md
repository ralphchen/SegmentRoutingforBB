# 关于ADJ-SID

### cisco NCS5500 

```
segment-routing
 local-block 190000 200000
 
router isis 1
 interface Bundle-Ether1001.101
  point-to-point
  address-family ipv4 unicast
   adjacency-sid absolute 190103 protected
   
 interface TenGigE0/0/0/0
  point-to-point
  address-family ipv4 unicast
   adjacency-sid absolute 190102
  !
```

注意，这里把190103设置了protected，190102没有protect，看后面ISIS LSP的verbose可以看到190103这个ADJ-SID的标志位B置位，表示backup，被保护。

验证：

```
RP/0/RP0/CPU0:PE1#show isis segment-routing label adjacency persistent  
Wed Oct 23 06:47:54.830 UTC

IS-IS 1 Manual Adjacency SID Table

190102 AF IPv4
      TenGigE0/0/0/0: IPv4, Not protected 1/255/N, Active

190103 AF IPv4
      Bundle-Ether1001.101: IPv4, Protected 1/65/N, Active
```



```
RP/0/RP0/CPU0:PE1#show isis database PE1.00-00  verbose 
Wed Oct 23 05:23:44.740 UTC

IS-IS 1 (Level-2) Link State Database
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime/Rcvd  ATT/P/OL
PE1.00-00           * 0x000015bd   0x8441        1194 /*            0/0/0
  Area Address:   47.0004.004d.0001
  NLPID:          0xcc
  IP Address:     1.1.1.1
  Router ID:      1.1.1.1
  Metric: 10         IP-Extended 1.1.1.1/32
    Prefix-SID Index: 1001, Algorithm:0, R:0 N:1 P:0 E:0 V:0 L:0
    Prefix Attribute Flags: X:0 R:0 N:1
    Source Router ID: 1.1.1.1
  Metric: 10         IP-Extended 10.1.2.0/24
    Prefix Attribute Flags: X:0 R:0 N:0
  Metric: 10         IP-Extended 10.1.3.0/24
    Prefix Attribute Flags: X:0 R:0 N:0
  Metric: 0          IP-Extended 19.19.19.0/24
    Prefix Attribute Flags: X:0 R:0 N:0
  Hostname:       PE1
  Router Cap:     1.1.1.1 D:0 S:0
    Segment Routing: I:1 V:0, SRGB Base: 16000 Range: 8000
    SR Local Block: Base: 190000 Range: 10001
    SR Algorithm: 
      Algorithm: 0
      Algorithm: 1
    Node Maximum SID Depth: 
      Label Imposition: 6
  Metric: 10         IS-Extended CR3.00
    Affinity: 0x00000000
    Interface IP Address: 10.1.3.1
    Neighbor IP Address: 10.1.3.3
    Physical BW: 10000000 kbits/sec
    Reservable Global pool BW: 10000000 kbits/sec
    Global Pool BW Unreserved: 
      [0]: 10000000 kbits/sec          [1]: 10000000 kbits/sec
      [2]: 10000000 kbits/sec          [3]: 10000000 kbits/sec
      [4]: 10000000 kbits/sec          [5]: 10000000 kbits/sec
      [6]: 10000000 kbits/sec          [7]: 10000000 kbits/sec
    Admin. Weight: 10
    Ext Admin Group: Length: 32
      0x00000000   0x00000000
      0x00000000   0x00000000
      0x00000000   0x00000000
      0x00000000   0x00000000
    Link Maximum SID Depth: 
      Label Imposition: 6
    ADJ-SID: F:0 B:0 V:1 L:1 S:0 P:0 weight:0 Adjacency-sid:24005
    ADJ-SID: F:0 B:1 V:1 L:1 S:0 P:1 weight:0 Adjacency-sid:190103
  Metric: 10         IS-Extended CR2.00
    Affinity: 0x00000000
    Interface IP Address: 10.1.2.1
    Neighbor IP Address: 10.1.2.2
    Physical BW: 10000000 kbits/sec
    Reservable Global pool BW: 0 kbits/sec
    Global Pool BW Unreserved: 
      [0]: 0        kbits/sec          [1]: 0        kbits/sec
      [2]: 0        kbits/sec          [3]: 0        kbits/sec
      [4]: 0        kbits/sec          [5]: 0        kbits/sec
      [6]: 0        kbits/sec          [7]: 0        kbits/sec
    Admin. Weight: 10
    Ext Admin Group: Length: 32
      0x00000000   0x00000000
      0x00000000   0x00000000
      0x00000000   0x00000000
      0x00000000   0x00000000
    Link Maximum SID Depth: 
      Label Imposition: 6
    ADJ-SID: F:0 B:0 V:1 L:1 S:0 P:0 weight:0 Adjacency-sid:24003
    ADJ-SID: F:0 B:0 V:1 L:1 S:0 P:1 weight:0 Adjacency-sid:190102
```

ADJ-SID FLAG：

https://tools.ietf.org/html/draft-ietf-isis-segment-routing-extensions-25#page-9

```
F-Flag: Address-Family flag.  If unset, then the Adj-SID refers
         to an adjacency with outgoing IPv4 encapsulation.  If set then
         the Adj-SID refers to an adjacency with outgoing IPv6
         encapsulation.
```

```
B-Flag: Backup flag.  If set, the Adj-SID is eligible for
         protection (e.g.: using IPFRR or MPLS-FRR) as described in
         [I-D.ietf-spring-resiliency-use-cases].
```

```
V-Flag: Value flag.  If set, then the Adj-SID carries a value.
         By default the flag is SET.
```

```
L-Flag: Local Flag.  If set, then the value/index carried by
         the Adj-SID has local significance.  By default the flag is
         SET.
```

```
S-Flag.  Set Flag.  When set, the S-Flag indicates that the
         Adj-SID refers to a set of adjacencies (and therefore MAY be
         assigned to other adjacencies as well).
```

```
P-Flag.  Persistent flag.  When set, the P-Flag indicates that
         the Adj-SID is persistently allocated, i.e., the Adj-SID value
         remains consistent across router restart and/or interface flap.
```





