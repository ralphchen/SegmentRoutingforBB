# PCE 互操作实验

在该例子中，使用CISCO NCS5500 作PCE，华为NE40 X3及juniper mx204作PCC



juniper配置：

```
[edit protocols pcep]
admin@PE5# show 
pce pce1 {
    local-address 5.5.5.5;
    destination-ipv4-address 3.3.3.3;
    destination-port 4189;
    pce-type active stateful;
    lsp-provisioning;
    spring-capability;
}
```



juniper验证：

```
admin@PE5> show path-computation-client status 

Session              Type                        Provisioning   Status         Uptime
  pce1               Stateful Active             On             Up              24          

LSP Summary
  Total number of LSPs        : 0        
  Static LSPs                 : 0        
  Externally controlled LSPs  : 0        
  Externally provisioned LSPs : 0/16000 (current/limit)
  Orphaned LSPs               : 0        

pce1 (main)
  Delegated              : 0               
  Externally provisioned : 0               
```



```
admin@PE5# run show path-computation-client active-pce 

PCE pce1
--------------------------------------------
General
    PCE IP address           : 3.3.3.3
    Local IP address         : 5.5.5.5
    Priority                 : 0
    PCE status               : PCE_STATE_UP
    Session type             : PCE_TYPE_STATEFULACTIVE
    LSP provisioning allowed : On
    P2MP LSP report allowed  : Off
    P2MP LSP update allowed  : Off
    P2MP LSP init allowed    : Off
    PCE-mastership           : main

Counters
    PCReqs              Total: 0            last 5min: 0            last hour: 0        
    PCReps              Total: 0            last 5min: 0            last hour: 0        
    PCRpts              Total: 27           last 5min: 1            last hour: 1        
    PCUpdates           Total: 0            last 5min: 0            last hour: 0        
    PCCreates           Total: 20           last 5min: 0            last hour: 0        

Timers
    Local  Keepalive timer:   30 [s]  Dead timer:  120 [s]  LSP cleanup timer:    0 [s]
    Remote Keepalive timer:   30 [s]  Dead timer:  120 [s]  LSP cleanup timer:    0 [s]

Errors
    PCErr-recv
    PCErr-sent
            Type: 19            Value: 1            Count: 3          
    PCE-PCC-NTFS
    PCC-PCE-NTFS

Last errors
    Last-PCErr-sent
            Type: 19            Value: 1        

Pcupdate empty ero action counters
    Send-err               : 0
    Tear down path         : 0
    Routing decision       : 0
    Routing decision failed: 0
```



目前juniper设备不支持 pce delegation建立LSP

