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
![[OSPF#非骨干区域划分特殊区域]]
# ABR(area border router)

定义：当路由器连接了多个区域，且在骨干区域有一个活动接口
功能：
1. 将直连区域内通过1类和2类LSA计算出的最优路由，转换成其他区域的3类LSA
2. 将骨干区域的3类LSA转换成其他区域的3类LSA

一旦路由器满足ABR条件，就会在1类LSA中置位ABR，来通告其他路由器自己是ABR角色
如果在OSPF进程中只创建了区域0，但是区域0中没有宣告任何活动接口，那么路由器亦然会在
1类LSA中置位ABR，但是不干ABR的工作。

非骨干区域的1类和2类计算出的路由只能被转换到骨干区域作为3类，然后再通过骨干区域的3类转换到其他非骨干区域。

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
	4. 同样选候选总开销(根到达部分节点的开销)小的选为树干节点
	5. 以此类推直到候选表为空
> 候选表中存在两个相同的节点时,删掉总开销大的,留下开销小的
> 候选表中出现总开销相同的, 将两个节点都画为树干
2. 再画叶子
3. 计算根节点到每一个叶子的开销, 选出根到相同叶子开销最小的, 加入OSPF路由表
![候选表](https://cgqin.github.io/images//202211162251323.png)
![最短路径生成树](https://cgqin.github.io/images//202211162250253.png)

# OSPF域间路由计算

## OSPF区域设计原则

1. 骨干区域有且只能有一个
2. 非骨干区域必须和骨干区域相连
3. 多区域的时候，必须有骨干区域
区域编号0，为骨干区域
区域编号非0，为非骨干区域

# 3类LSA(Sum-Net)

```
Type      : Sum-Net            LSA类型，sum-net表示三类LSA，用来描述区域间的路由信息
Ls id     : 3.3.3.3            3类LSA使用该路由信息的网络号表示
Adv rtr   : 2.2.2.2            通告者：产生这条LSA的路由器（ABR）
Ls age    : 571                老化时间
Len       : 28                 长度
Options   :  E                 特殊区域标识
seq#      : 80000001           序列号
chksum    : 0xae99             校验和
Net mask  : 255.255.255.255    用来描述这条路由信息的网络掩码
Tos 0  metric: 1               ABR到达这个目的路由的开销值
```

# OSPF域间路由防环原则

**原则一:** 
为了避免区域间的环路，OSPF规定不同区域间的路由器交互只能通过ABR实现。
ABR是连接到骨干区域的，所以在区域设计上规定，所有非骨干区域要连接到
骨干区域。区域间的通讯需要通过骨干区域，行成逻辑上的==星状拓扑==，且无环。

**原则二:**
1. OSPF不会将非骨干区域的3类LSA(不是计算生成的3类LSA,即收到的区域内泛洪的3类LSA)传递到骨干区域
2. ABR在骨干区域存在邻居时, 不会计算非骨干区域的3类LSA
3. ABR在骨干区域不存在邻居时, 会计算非骨干区域的3类LSA

**原则三:**
无论cost, 1类LSA优于3类LSA

# vlink虚链路 

>将非骨干区域的路由器接入骨干区域,使之成为ABR

1. 虚链路属于区域0的逻辑链路
2. 虚链路只能穿越1个非骨干区域
3. 虚链路不能穿越特殊区域

**存在冗余链路vlink计算最优路径方法**
配置了vlink的路由器会计算2棵SPF树。
   1棵是以自己为根，在穿越区域内计算最短路径树
   另一颗是以vlinkpeer端来计算到达自己的最短路径树。
综上，决定了最优路径的接口IP地址来收发报文
同时由于存在冗余路径，当线路断的时候，SPF树不会断，从而vlink也不会断

## vlink配置

```
[R1]                          // 骨干区(区域0)路由器配置
ospf 1
 area 0.0.0.1                 // 要跨越的区域
  vlink-peer x.x.x.x          // 指定对端路由器router-id
[R2]                          // 非骨干区路由配置
ospf 1
 area 0.0.0.1                 // 要跨越的区域
  vlink-peer x.x.x.x          // 指定对端路由器router-id
```

# OSPF外部路由

>路由引入(重分发)(重分布)

## 配置

```
ospf 1                      //进入OSPF进程
 import-route ?        //引入外部路由
 import-route static type 1/2        // 更改开销类型
```

## ASBR

当OSPF路由器使用了命令import-route以后，那么路由器就会在自己的1类LSA中置位ASBR。不论他是否真正引入的外部路由。
```
1类LAS报文Flag中的 AS boundary router 置为1
Flags：0x02，(E）AS boundary router
.... .0..=（V）Virtual link endpoint:No
.... ..1.=（E）AS boundary router:Yes
.... ...0= (B)Areaborder router:No
```

## 5类LSA(External)

路由器OSPF进程开启外部路由引入后, 会查看全局路由表,如果路由表中存在要引入的路由,会通过5类LSA通告**邻接**的路由器
邻接路由器收到5类LSA后同样会通告**邻接**路由器
```
Type      : External               LSA类型，external代表外部路由 
Ls id     : 5.5.5.5                使用外部路由的网络号来填充
Adv rtr   : 4.4.4.4                通告者，使用ASBR路由器的router-id填充
Ls age    : 708 
Len       : 36 
Options   :  E  
seq#      : 80000001 
chksum    : 0x9f0d
Net mask  : 255.255.255.255        描述了该外部路由器的网络掩码 
TOS 0  Metric: 1                   描述了该5类LSA到达目标网络的开销值cost
E type    : 2                      描述了该5类LSA的开销值类型
Forwarding Address : 0.0.0.0       转发地址：当转发地址为0.0.0.0时，那么路由器在计算5类LSA时回去找ASBR计算
                                            当转发地址为具体IP时，那么路由器在计算5类LSA时，就不在找ASBR了，
	                                            会去通过SPF算法直接找FA地址作为下一跳


type 2:OSPF引入的外部路由默认type类型为2，当路由器计算type2的外部路由时，不会计算OSPF内部的cost，只计算ASBR到达目标网络的cost
type 1:OSPF引入外部路由时，可以通过命令更改type类型，当路由器计算type1的外部路由时，会同时计算OSPF内部cost+ASBR到达目标网络的cost
使用场景：当不在意OSPF内部的次优路径问题时可以使用type2
          当在意OSPF内部次优路径问题时需要使用type1
          如果在数据库中存在相同目的的5类LSA且type不一致时，type1优于type2
```

## 4类LSA(Sum-Asbr) 

4类LSA作用: 让其他区域的路由器在计算5类LSA时，可以**找到ASBR**在哪(ASBR的介绍信)
4类LSA产生者: ABR
4类LSA遵循域间路由防环原则
当路由器(ABR)收到这条ASBR置位的1类LSA后，由于自身是一台ABR，那么他就会根据这条ASBR置位的1类LSA生成一条4类LSA，在其他直连区域内进行传递
如果自身不是ABR角色，那么将不具备生成4类LSA的能力

```
Type      : Sum-Asbr     LSA类型，ASBR表示4类LSA，用来描述ASBR信息
Ls id     : 4.4.4.4      使用ASBR路由器的router-id填充
Adv rtr   : 3.3.3.3      通告者，产生这条4类LSA的路由器router-id（一般由ABR产生）
Ls age    : 1784 
Len       : 28 
Options   :  E  
seq#      : 80000002 
chksum    : 0x52eb
Tos 0  metric: 1         用来描述ABR路由器到达ASBR路由器的开销值
```

## 外部路由引入过程

![](https://cgqin.github.io/images//202211191513832.png)
1. 在路由器4的OSPF进程中使用命令import-route static来引入路由表中的静态路由
2. R4会触发更新2条LSU，其中第一条是更新了R4的1类LSA其中的ASBR置位，来告诉域内小伙伴，我变成ASBR了，我要引入外部路由了
   第二条LSU是更新的外部路由（5类LSA）
3. 5类LSA的同步和泛洪原则：5类LSA的同步不基于区域的传递原则，是在整个OSPF域中进行泛洪，只要和其他路由器建立了邻接关系，就会泛洪这条5类LSA
   特殊区域除外
4. 所有的外部路由在引入OSPF后，会以5类LSA存在于OSPF数据库中，外部的cost将不在计算，将赋予5类LSA一个新的种子度量值（cost默认为1）
   这个默认开销值就会被认为是ASBR到达目标网络的开销值
5. (4类LSA和5类LSA同步完毕)路由器开始计算路由
	- FA为0.0.0.0, 寻找ASBR
	- FA为确定ip, 计算ip路由


## FA地址

FA地址：[[OSPF#5类LSA(External)|5类LSA]]的一个参数,
			  就是引入路由的下一跳地址

### FA地址产生情况

1. ASBR去往外部路由的出接口被宣告进OSPF进程中
2. ASBR去往外部路由的出接口没有被设置为静默接口
3. ASBR去往外部路由的出接口不是P2P

### FA地址的作用

==用来做OSPF链路优化的==
1. 当5类LSA中FA地址是0.0.0.0时，代表没有FA地址，那么路由器在计算这条5类LSA时是要找通告者来计算
2. 当5类LSA中的FA地址非0时，代表存在FA地址，那么路由器在计算这条5类LSA时，不在找ASBR计算了，而是要找FA地址

## OSPF默认路由

通告者一般为NAT出口路由器
![](https://cgqin.github.io/images//202211191559473.png)
如图为==R3==

## 配置

```
ospf 1                                //进入OSPF进程
 default-route-advertise              // 通告默认路由
 default-route-advertise always       //带always参数时,不管路由表中有没有默认路由,都会通告默认路由的5类LSA
```

# OSPF不同Site的路由引入(不同进程路由引入)

同一个路由器不同OSPF进程是不相互干扰的, 不同进程会产生2个不同的LSDB 

![](https://cgqin.github.io/images//202211191612769.png)
如图,由于R3开启了两个进程, 所以它会产生两个不同的LSDB,从而OSPF会被分隔成两个Site,所以需要路由引入

## 引入配置

```
[R3]
  ospf 1
    import-route ospf 2 (+路由策略)
```

# 特殊区域

作用: 特殊区域是OSPF优化的一种手段，当路由器无法承载大量的LSA时，会考虑减少LSA的数量来优化

## 非骨干区域划分特殊区域

1. Stub       末节区域
2. 完全Stub   完全末节区域
3. NSSA       非完全末节区域
4. 完全NSSA   完全非完全末节区域

## Stub

==不处理外部路由==
==stub区域不传递4类和5类LSA==

当区域1配置为Stub区域时，发出的==hello包==中，==option字段==中的==Ebit置位为0==，代表该区域没有处理外部路由的能力
由于Stub区域内没有了5类和4类LSA，所以没有办法计算外部路由的明细，因此ABR会产生一条3类缺省路由，
让Stub区域内的路由器可以访问到外部路由，这条3类缺省LSA的==cost默认为1==

Stub区域如果有多台ABR时，每个ABR都会产生一条3类缺省
那么区域内的IR路由器就会有负载的可能性，由于Stub区域没有4类和5类LSA，
一旦负载，那么久会缺失对外部cost的感知能力，那么就会有次优路径的风险

**解决办法**：(只能通过人为干预)
1. 针对Stub区域内的IR路由器接口改变cost来人为选路
2. 在ABR调整3类缺省路由的种子度量值
   area 0.0.0.1
      default-cost 19   //修改ABR下发的3类缺省LSA的种子度量值
3. 在IR路由器针对下一跳修改权重值
   ospf 1 router-id 1.1.1.1
       nexthop 12.1.1.2 weight 1  //针对下一跳是R2的路由修改权重
   PS：权重值默认为255，数值越小越优先。如果没有配置权重，默认认为就是255



### Stub配置

在IR和ABR的区域视图下敲Stub
```
[IR]        //IR的区域视图下敲Stub
ospf 1 
 area 0.0.0.1
  stub
[ABR]       //IR的区域视图下敲Stub
ospf 1
 area 0.0.0.1
  stub
```

## 完全Stub 

完全Stub区域是在Stub的基础上，将该区域内的其他3类明细LSA过滤掉。
完全Stub区域内只保留该区域的1类和2类LSA，以及ABR下发的3类缺省LSA
完全Stub区域继承了StuB区域的所有特性，包括多ABR时次优路径的风险问题，解决方法一致

### 完全Stub区域配置

在IR的区域视图下敲stub，在ABR的区域视图下敲stub no-sumary
```
[IR]        //IR的区域视图下敲Stub
ospf 1 
 area 0.0.0.1
  stub
[ABR]       //IR的区域视图下敲stub no-sumary
ospf 1
 area 0.0.0.1
  stub no-sumary
```

## NSSA

在stub的基础下,自身又要引入外部路由
![](https://cgqin.github.io/images//202211201630935.png)
NSSA区域不存在4类和5类LSA 
R1 引入外部路由后会成为ASBR, 然后把引入路由信息转换成7类LSA, 并在NSSA区域内泛洪,
R2 和 R8 收到7类LSA后,会成为ASBR, 然后将7类LSA转换成5类LSA泛洪,


### NSSA配置

```
[IR]        //IR的区域视图下敲nssa
ospf 1 
 area 0.0.0.1
  nssa
[ABR]       //IR的区域视图下敲nssa
ospf 1
 area 0.0.0.1
  nssa
```