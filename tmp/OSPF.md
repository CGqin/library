# OSPF 工作过程

1. 邻居建立
2. 同步链路数据库
3. 计算最有路由

# router-id
>router-id是作为OSPF路由器的==唯一标识==
>用于在自治系统中唯一标识一台运行OSPF路由器

官方的说法:
1. 建议手动配置OSPF的router-id(可以不是接口的IP地址)
2. 如果没有手动配置，那么会使用环回接口IP地址最大的作为router-id
3. 如果路由器没有环回接口，那么会使用物理接口IP地址最大的作为router-id 
实际情况:
1. 建议手动配置OSPF的router-id(可以不是接口的IP地址)
2. 如果没有手动配置，会使用路由器==全局router-id==作为OSPF的router-id
```
[AR1]display router id
//查看路由器全局router-id
RouterID:12.1.1.1
```
3、全局router-id是咋来的呢？==路由器第一个UP的接口将会作为全局router-id==

# OSPF报文

## OSPF 通用头部

```
OSPF通用头部：
Version 2                  OSPF版本信息（版本2版本3）
Message Type:              OSPF报文类型12345
Packet length:             OSPF报文长度（头部+具体报文）
Source OSPF router:        发出该报文的路由器router-id
Area ID                    区域ID    骨干区域和非骨干区域之分
Auth type                  认证类型
Auth data                  认证数据（秘钥）
```

## Hello 包

```
Network Mask            发出该报文接口的子网掩码
Hello Interval          hello时间
option                  特殊区域标记
		                E=1说明是普通区域（可以是骨干区域，也可以是非骨干区域）
		                E=0 N=0 说明这是stub区域 
		                E=0 N=1 说明这是NSSA区域 
router PRI              路由器接口的DR优先级
Router Dead Interval    邻居失效时间，一般是hello时间的4倍
DR                      选举出的DR是谁
BDR                     选举出的BDR是谁
Active Neighbor         激活的邻居router-id,用来标识存在的邻居信息
```

## DD报文

```
Interface MTU           发出该报文接口的TU值，默认为B,(华为在实现osPF时，忽略了MTU的比较，可以手动开
Option                  特殊区域的标记
		                E=1说明是普通区域（可以是骨干区域，也可以是非骨干区域）
		                E=0 N=0 说明这是stub区域 
		                E=0 N=1 说明这是NSSA区域 
DB description          数据库描述信息
                        I 初始DD报文，用来选举主从关系master slave 
                        M 用来标识是否还有后续报文，如果为0，代表这是最后一个摘要信息
					                              如果为1，代表后续还有摘要信息
						MS 代表主从置位，如果为1： 代表我是master
										如果为0： 代表我是slave 
DD Sequence             序列号：用来做隐式确认
```

## LSR报文

>链路状态请求报文(携带LSA的三要素)

```
LS Type
Link State ID
Advertising Router
```

## LSU报文

>携带真正的LSA信息

## LSAck报文

>确认报文(使用LSA摘要信息确认)

# OSPF邻居建立过程

>[[OSPF 的邻居状态]]

## OSPF邻居建立过程图

![](https://cgqin.github.io/images//202211122239470.png)

## OSPF邻居建立过程概述

1. 两台路由器宣告了OSPF,发送Hello报文
2. 两台路由器收到对方发来的Hello后,会将对方的router-id放到自己的Hello包作为`active neighbor`,发出.此时邻居状态为`init`,也叫`1-way`
3. 两台路由器再次收到对方发来的携带了`active neighbor`为自己router-id的Hello包以后,建立邻居.此时邻居状态为`2-way`
	(从init到2-way这个阶段,OSPF要判断是否和邻居建立邻接关系.也决定着路由器是否要发送DD报文给邻居)
4. 两台路由器分别发送DD报文, `init=1 m=1 ms=1` 开始选主从, 两台路由器都认为自己是主, 此时邻居状态为`exstart` (选主从是用来做[[OSPF#DD报文的隐式确认过程|隐式确认]])
5. 比较router-id后, 从路由器开始发送DD-LSA摘要信息, `init=0 m=0 ms=0` (m取决于后续是否还有摘要信息). 
	此时从路由器状态为`exchange`, 主路由器仍为`exstart`
6. 主路由器收到DD摘要后, 发送LSA. 之后发送DD摘要给从路由器, `init=0 m=0 ms =1` (m取决于后续是否还有摘要信息).
	此时从路由器状态为 `loading` , 主路由器为`exchange`
7. 从路由器收到DD摘要后, 发送LSR. 之后发送DD报文给主路由器, 这个DD报文没有摘要, `init=0 m=0 ms=0`, 为了告诉主路由摘要已经交换完毕.
	此时主路由为`loading`
8. 路由器之间更新LSU后,回复LSAck,后置位为`FULL`

# OSPF同一广播域内DR BDR选举

>**[[MA网络.png|MA网络]]问题**
>1. n×(n-1)/2个邻接关系，管理复杂。
>2. 重复的LSA泛洪，造成资源浪费。
>所以需要选举DR BDR

## DR与BDR作用

- 减少邻接关系
- 降低OSPF协议流量

## 选举规则

- 接口的DR优先级越大越优先。 (Hello报文中的 `router PRI` 字段)
- 接口的DR优先级相等时，Router ID越大越优先。

## 选举过程


# OSPF报文的确认机制

1. Hello        10s 40s Deadinterval 
2. DD           seg序列号来做隐式确认
3. LSR          使用LSU来确认
4. LSU 
5. LSACK     本身就是一个确认报文，确认所有LSU同步完成 

## DD报文的隐式确认过程

R2--->R1 第一个DD 用来选主从，此时 `SEQ：129`
R1--->R2 第一个DD 用来选主从，此时 `SEQ：124`
以上2个SEQ是没有人任何关联性的
R1--->R2 第2个DD，认怂，自己是从路由器，此时 `SEQ：129` (此时的SEQ会使用和R2给R1发送的第一个DD报文的SEQ) 用来给R2做隐式确认的。
R2--->R1 第2个DD，自己仍然是主路由器，发送了LSR后会给从路由器发送DD摘要，此时SEQ为 `129+1=130`
R1--->R2 最后一个DD，从路由器认为LSA摘要全部交换完毕，发送空DD帮助主路由器进入loading状态，此时空DD的`SEQ=129+1`

# DD报文中的MTU

1. DD报文中第一个字段就是MTU，正常来说，OSPF邻居建立是要求2个接口的MTU值一致，否则无法建立邻接，
	但是华为在实现邻居建立过程时，对于MTU的检测默认是忽略的。所以MTU值对于华为设备默认不影响。
	(友商默认情况下是检查MTU的)
2. 可以认为使用命令开启MTU检测
	`[AR1-GigabitEtherneto/e/e]ospf mtu-enable`
	MTU检测功能如果只在一段开启，另一端默认，那么不影响邻接建立
	所以，两端都要开启

# DD报文的三个作用

1. 选举主从
2. 传递摘要信息
3. 摘要传递完毕，将对方置位loading 

# OSPF四种网络类型

1. Broadcast(广播)
2. NBMA(非广播多路访问)
3. P2MP(点到多点)
4. P2P(点到点)

| 网路类型 | 是否建立邻居关系|
|---------|-----------------|
|P2P|是|
|Broadcast|
|NBMA|
|P2MP|是|



## 1. 广播(缺省)

单链路状态为以太网时OSPF会认为数据类型时Broadcast
需要选举DR和BDR
==hello时间10s，Dead时间40s==
DR需要和其他接口角色建立FULL关系
BDR需要和其他接口角色建立FULL关系
DRother和DRother建立2way
有2个组播更新地址：224.0.0.5 224.0.0.6
所有设备使用单播形式交互DD和LSR报文
所有设备固定使用组播地址224.0.0.5交互hello报文
LSU和LSACK比较特殊：
a、小弟发送更新LSU报文通过224.0.0.6发送给DR和BDR的；DR通过224.0.0.5发送给其他的小弟和BDR；
小弟收到DR的LSU报文后，通过224.0.0.6发送ACK
BDR收到DR的LSU报文后，通过224.0.0.5发送ACK
b、DR发送更新LSU报文通过224.0.0.5发送给BDR和所有小弟；
BDR收到更新报文后，通过224.0.0.5发送ACK
小弟收到更新报文后，通过224.0.0.6发送ACK
C、BDR发送更新LSU报文通过224.0.0.5发送给DR和所有小弟；
DR收到更新报文后，通过224.0.0.5发送ACK
小弟收到更新报文后，通过224.0.0.6发送ACK
DR和BDR同时监听224.0.0.5和0.6两个组播地址，小弟只监听224.0.0.5

## 2. 非广播多路访问(NBMA)

如果链路层协议是帧中继、ATM这种，OSPF会认为网络类型为NBMA
如果需要建立NBMA网络类型的邻居，需要使用peer ip来指定单播邻居，只有指定的单播邻居IP，收到hello报文
所有报文通过单播方式发送
需要选举DR和BDR
==hello时间30s，dead时间120s==
DR和任何角色都要建立FULL，BDR和任何角色都要建立FULL，小弟之间不需要建立FULL

## 3.点到点(P2P)

如果链路层协议为PPP HDLC，OSPF会认为网络类型为P2P
==hello时间10s，dead时间40s==
不需要选举DR和BDR
直接建立FULL关系
所有报文是通过224.0.0.5组播更新

## 4. 点到多点(P2MP)                (常用于DSVPN场景)

没有任何一种链路层协议被认为是P2MP，这种类型是人为手动配置的
==hello时间30s，dead时间120s==
不需要选举DR和BDR
直接建立FULL关系
Hello报文通过224.0.0.5组播更新，其他所有报文通过单播更新

# OSPF度量方式

- 接口cost=参考带宽/实际带宽
>如果计算结果         大于0且小于2                     那么cost=1
                               大于等于2且小于3              那么cost=2
                               以此类推
- 更改cost的两种方式：
   1. 直接在接口下配置;
   2. 修改参考带宽（所有路由器都需要修改，确保选路一致性）。

# 静默接口

配置了静默接口的接口，只会生成LSA，==不会收发OSPF报文==

**静默接口配置**

```
ospf 1 router-id 1.1.1.1
	silent-interface all        // 宣告所有接口为静态接口
	undo silent-interface GigabitEthernet0/0/0          // 开放非静默接口
	area 0.0.0.0
		network 12.1.1.1 0.0.0.0
```


# LSA的种类

- 1类LSA: Router LSA
- 2类LSA:Network LSA
- 3类LSA:Network summary LSA
- 4类LSA: ASBR summary link states
- 5类LSA: AS external link states
- 7类LSA:NSSA external link states
- 其他LSA:6类（组播OSPF）8类9类(OSPFv3）10类11类(MPLS)

# LSA头部信息

```
Ls age                                  LSA的老化时间
option                                  用来描述特殊区域的
LS Type                                 LSA类型     router-lsa代表1类LSA，叫做路由器
LINK State ID                           LSA的名字  不同类型的LSA取值不同
ADvertise ROUTER                        通告者 发出该LSA的路由器router-id
SEQ                                     序列号   用来比较LSA新旧
Chechsum                                校验和   用来比较LSA新I旧
Length                                  长度
```

# LSA三要素

如果确定一条唯一的LSA呢？
1. LS type
2. LS State ID
3. ADV router

# LSDB同步原则

![](https://cgqin.github.io/images//202211152217979.png)

# OSPF区域划分

- 骨干区域
- 非骨干区域

# ABR


# 1类LSA(Router)

当路由器的一个接口宣告进了OSPF，那么这个接口就会生成链路状态，存放到LSA当中，
每个路由器都会生成一个1类LSA，==用来描述自身的直连链路状态==

```
Type      : Router              LSA类型，router代表1类LSA，用来描述路由器自身直连接口的链路状态信息
Ls id     : 1.1.1.1             链路状态ID，也就是这条LSA的名字，在1类LSA中，使用本路由器的router-id充当
Adv rtr   : 1.1.1.1             通告者，产生这条LSA的路由器router-id
Ls age    : 140                 老化时间
Len       : 48                  报文长度
Options   :  E                  特殊区域标识 E = 1 代表普通区域 E = 0 代表末节区域  E=0且N=1 代表NSSA区域
seq#      : 80000002            序列号 
chksum    : 0xdf28              校验和
Link count: 2
```

在1类LSA当中，使用4种==link-type==来描述链路状态：
```
1. 当link-type为StubNet：用来描述一条路由信息的（叶子节点）
     Link ID: 12.1.1.0          用来描述这条路由信息的网络号
     Data   : 255.255.255.0     用来描述这条路由信息的网络掩码
     Metric : 1562              用来描述该路由器达到目标网络的开销值


2. 当link-type为P-2-P:   用于描述直连链路上网络类型为P2P或者P2MP的邻居信息（树干信息）
     Link ID: 2.2.2.2           用来描述该邻居的router-id
     Data   : 12.1.1.1          用来描述直连到该邻居接口的IP地址
     Metric : 1562              用来描述该路由器达到该邻居的开销值


3. 当link-type为TransNet:  用来描述直连链路上网络类型为广播或者NBMA的邻居信息
   * Link ID: 192.168.1.2  用来描述伪节点的router-id，用DR的接口IP来充当
     Data   : 192.168.1.2  用来描述自身直连伪节点的接口IP地址 
     Metric : 1            用来描述自身到达伪节点的开销值


4. 当link-type为vlink:     用来描述网络类型为虚链路的邻居信息（树干)
     link-id:              用来描述虚链路邻居信息（router-id)
     data:                 用来描述自身直连该邻居接口的IP地址
     metric:               用来描述自身到达该虚链路邻居的开销值
```

# 2类LSA (Network)

```
Type      : Network               LSA类型，用network来标识2类LSA，用来描述伪节点的信息
Ls id     : 192.168.1.4           LSA名字，在不同类型的LSA中取值不同，2类LSA用DR的接口IP地址充当
Adv rtr   : 4.4.4.4               通告者,DR所在路由器的router-id来标识
Ls age    : 1165                  老化时间
Len       : 36                    报文长度
Options   :  E                    特殊区域标识
seq#      : 80000004              序列号
chksum    : 0x5a60                校验和
Net mask  : 255.255.255.0         2类LSA不仅可以标识邻居信息（树干信息），同时也可以描述路由信息（叶子）
Priority  : Low
   Attached Router    2.2.2.2     用来描述树干信息的，描述该伪节点直连的邻居信息
   Attached Router    3.3.3.3     用来描述树干信息的，描述该伪节点直连的邻居信息
   Attached Router    4.4.4.4     用来描述树干信息的，描述该伪节点直连的邻居信息
```


# 计算最短路径树

1. 先画树干
	1. 以自身路由器为根, 将`P-2-P`链路状态和`TransNet`链路状态加入候选表中
	2. 候选总开销(根到达部分节点的开销)小的选为树干节点
	3. 树干节点同样将自己的`P-2-P`链路状态和`TransNet`链路状态加入候选表中(或者为2类LSA的树干信息加入候选表中)
	4. 
# 3类LSA()

