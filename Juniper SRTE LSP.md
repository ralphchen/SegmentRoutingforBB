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

