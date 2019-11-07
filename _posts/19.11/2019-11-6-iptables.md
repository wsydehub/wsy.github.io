---
layout:     post
title:      "iptables详解"
subtitle:   ""
date:       2019-11-6 20:28:00
author:     "Dsyzxml"
header-img: "img/in-post/PostBackground/ab.jpeg"
catalog: true
tags:
    - Linux
---

# iptables简介

iptables组成了linux内核防火墙的命令行工具，是netfilter项目的一部分。它可以完成封包过滤、封包重定向和网络地址转换等功能。

# iptables基本概念

理解iptables如何工作的关键是理解下图。图中上面的小写字母代表表，下面的大写字母代表链。从任何网络端口进来的IP数据包都要从上到下穿过这张图。
![iptables_traverse](/wsy.github.io/img/in-post/19.11/tables_traverse.jpg)

**Tables**
iptables包含5张表:
    1.***raw***用于配置数据包，***raw***中的数据包不会被系统跟踪
    2.***filter***用于存放所有与防火墙相关操作的默认表
    3.***nat***用于网络地址转换
    4.***mangle***用于特定数据包的修改
    5.***security***用于强制访问控制网络规则
大部分情况下，只使用filter和nat。

**Chains**
表由链组成，链是一些按顺序排列的规则的列表。默认的filter表包含***INPUT***，***OUTPUT***和***FORWARD***。从上面的流程图可以看出***nat***表包含了***PREROUTING***，***POSTROUTING***和***OUTPUT***三条链。
默认情况下，链内没有规则。链的默认规则通常设置为***ACCEPT***，如果想保证任何包都不能通过规则集，可可以重置为***DROP***。默认的规则总是在链的最后一条生效，所以在默认规则生效前数据包要通过所有存在的规则。

**Rules**
数据包的过滤基于规则。规则由多个matches（包使用该规则所必须满足的条件）以及一个target（当包满足所有条件后执行的行动）定义。一个规则典型的matches是数据包进入的interface（例如：eth0或eth1）、数据包的类型（ICMP、TCP或UDP）和数据包的目的端口。
target使用***-j***或***-jump***指定。target可以是用户定义的chains（例如，如果条件匹配跳转到用户定义的链进行后续处理）、一个special built-in targets或一个target extension。Built-in targets是***ACCEPT***、***DROP***、***QUEUE***和***RETURN***，target extension可以是***REJECT***或***LOG***等。If the target is a built-in target, the fate of the packet is decided immediately and processing of the packet in current table is stopped. If the target is user-defined chain and the fate of the packet is not decided by this second chain, it will be filtered against the remaining rules of the original chain. Target extension can be either teminating(as built-in targets) or non-terminting(as user-defined chains).

**Travesing Chains**
不论从哪个interface进来的数据包都会按上面的流程图的顺序经过每一条链。第一个路由策略包括决定数据包的目的地是本地主机（这种情况下，数据包穿过 INPUT 链），还是其他主机（数据包穿过 FORWARD 链）；中间的路由策略包括决定给传出的数据包使用那个源地址、分配哪个接口；最后一个路由策略存在是因为先前的 mangle 与 nat 链可能会改变数据包的路由信息。


# iptables CLI
### 列出当前的iptables rules
```
iptables -L

//列出端口代替服务名
iptables -L -n

//同时列出匹配数
iptables -L -v
```

### 添加规则

```
[root@server ~]# iptables -A INPUT -p tcp --dport 80 -j ACCEPT
[root@server ~]# iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

### 删除规则

```
//删除INPUT链的第5条规则
[root@server ~]# iptables -D INPUT 5
[root@server ~]# iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

### 插入规则

```
[root@server ~]# iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
[root@server ~]# iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

### 替换规则

```
[root@server ~]# iptables -R INPUT 1 -p tcp -s 192.168.0.0/24 --dport 80 -j ACCEPT
```

### 清除规则

```
iptables -F <chain>
```

# 编写规则

### -p, --protocol <u>protocol<u>
    The  protocol  of  the  rule  or of the packet to check.  The specified protocol can be one of tcp, udp, udplite,icmp, icmpv6,esp, ah, sctp, mh or the special keyword "all", or it can be a numeric value,  representing  one  of these  protocols or a different one.  A protocol name from /etc/protocols is also allowed.  A "!" argument beforethe protocol inverts the test.  The number zero is equivalent to all. "all" will match with all protocols and  is taken as default when this option is omitted.  Note that, in ip6tables, IPv6 extension headers except esp are not allowed.  esp and ipv6-nonext can be used with Kernel version 2.6.11 or later.  The number zero is equivalent to all, which means that you cannot test the protocol field for the value 0 directly. To match on a HBH header, even if it were the last, you cannot use -p 0, but always need -m hbh.

### -s, --source <u>address<u>[/<u>mask<u>]:[,...]
    Source specification. Address can be either a network name, a hostname, a network IP address (with /mask),  or  a plain  IP address. Hostnames will be resolved once only,before the rule is submitted to the kernel.  Please note that specifying any name to be resolved with a remote query such as DNS is a really bad idea.  The  mask  can  be either  an  ipv4  network mask (for iptables) or a plain number, specifying the number of 1's at the left side of the network mask.  Thus, an iptables mask of 24 is equivalent  to  255.255.255.0.   A  "!"  argument  before  the address  specification  inverts  the  sense of the address. The flag --src is an alias for this option.  Multiple addresses can be specified, but this will expand to multiple rules (when adding with -A), or will cause  multiple rules to be deleted (with -D).

### -d, --destination <u>address<u>[/<u>mask<u>]:[,...]
    Destination specification.  See the description of the -s (source) flag for a detailed description of the syntax. The flag --dst is an alias for this option.

### -m, --match <u>match<u>
    Specifies a match to use, that is, an extension module that tests for a specific property.  The  set  of  matches make  up  the  condition under which a target is invoked. Matches are evaluated first to last as specified on the command line and work in short-circuit fashion, i.e. if one extension yields false, evaluation will stop.

### -j, --jump <u>target<u>
    This specifies the target of the rule; i.e., what to do if the packet matches it.  The  target  can  be  a  user-defined  chain  (other than the one this rule is in), one of the special builtin targets which decide the fate of the packet immediately, or an extension (see EXTENSIONS below).  If this option is omitted in a rule (and  -g  is not  used), then matching the rule will have no effect on the packet's fate, but the counters on the rule will be incremented.

### -g, --goto <u>chain<u>
    This specifies that the processing should continue in a user specified chain. Unlike  the  --jump  option  return will not continue processing in this chain but instead in the chain that called us via --jump.

### -i, --in-interface <u>name<u>
              Name of an interface via which a packet was received (only for packets entering the INPUT, FORWARD and PREROUTING
              chains).  When the "!" argument is used before the interface name, the sense is inverted.  If the interface  name
              ends  in a "+", then any interface which begins with this name will match.  If this option is omitted, any inter‐
              face name will match.

### -o, --out-interface <u>name<u>
    Name of an interface via which a packet is going to be  sent  (for  packets  entering  the  FORWARD,  OUTPUT  and POSTROUTING  chains).   When  the  "!" argument is used before the interface name, the sense is inverted.  If the interface name ends in a "+", then any interface which begins with this name will match.  If this option is omitted, any interface name will match.

### -f, --fragment
    This  means that the rule only refers to second and further IPv4 fragments of fragmented packets.  Since there is no way to tell the source or destination ports of such a packet (or ICMP type), such a packet will not match  any rules which specify them.  When the "!" argument precedes the "-f" flag, the rule will only match head fragments, or unfragmented packets. This option is IPv4 specific, it is not available in ip6tables.

### -c, --set-counters <u>packets<u> <u>bytes<u>
    This enables the administrator to initialize the packet and byte counters  of  a  rule  (during  INSERT,  APPEND, REPLACE operations).