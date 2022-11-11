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
