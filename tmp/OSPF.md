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

# OSPF网络类型

1. 