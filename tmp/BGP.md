# 路由协议的分类

1. BGP属于外部网关协议
2. BGP属于路径矢量路由协议

# AS号

运行BGP的路由器必须有一个AS号,并且只有一个. AS号是由国际组织IANA分配的
公有AS号: 1~64511
私有AS号: 64512~65535
目前使用的1~65535都叫2字节AS号
因为2字节AS号不能满足使用, 后来有推出了4字节AS号

# BGP的工作特点

1. BGP基于TCP目的==端口179==工作,是工作在==应用层==
2. 如果邻居和自己的AS号一致, 那么建立的邻居叫IBGP邻居
	 如果邻居和自己的AS号不一致, 那么建立的邻居叫做EBGP邻居

# IBGP和EBGP建立邻居的常规操作

**IBGP邻居建立时通常使用环回接口作为收发报文接口**
**EBGP邻居建立时通常使用直连接口作为收发报文接口**
IBGP建立邻居时报文中TTL默认为255, 所以可以跨越多条建立邻居
EBGP建立邻居时报文中TTL默认为1, 所以建议使用直联接口
(但是这个TTL可以手动更改：[AR4-bgp]peer 34.1.1.3 ebgp-max-hop)

# BGP的router-id

1. 人为手动指定
2. 自动选举(如果没有人为手动指定,那么将会选择路由器的全局router id)
BGP的router-id和OSPF一样,代表唯一一台BGP路由器

# BGP的更新方式

1. 触发更新
2. 管理手动更新
		refresh bgp all import //从所有邻居更新一份最新的路由给我
		refresh bgp all export //我给所有邻居更新一份我最新的路由

# BGP的防环机制

IBGP邻居水平分割：从IBGP邻居接收到的路由不会传递给另外一个IBGP邻居
EBGP的as-path属性防环：当BGP设备从EBGP邻居收到路由后，
如果这条路由的AS-PATH列表中有我本地的AS号，此时将不接收这条路由

# 对于下一跳的修改问题

1. 对于路由器始发的的路由, 传递给IBGP或EBGP邻居时都会自动修改下一跳
2. 从IBGP邻居收到的路由,转发发给EBGP邻居时,会自动修改下一跳
3. 从IBGP收到的路由, 不会转发给IBGP邻居, 因此不考虑下一跳修改问题
4. 从EBGP邻居收到的路由,转发给IBGP邻居时,不修改下一跳. 我们可以手动修改下一跳
	[AR4-bgp]peer 2.2.2.2 next-hop-local //针对路由器R2传递过去的路由强制修改下一跳为我
1. 从EBGP邻居收到的路由, 传递给EBGP邻居时, 会修改下一跳
==注意: BGP只传递最有且有效的路由给其他邻居==

# BGP的同步功能

在BGP进程中，默认会有undo synchronization，并且不能开启（思科里可以开启）
   原理：从IBGP收到路由后，如果在IGP路由表中不存在这条路由，则认为不同步，将不会传递给EBGP邻居
   注释：为什么要和IGP表去同步呢？因为只要IGP路由表中存在，即在本AS内不会存在黑洞
   目的：BGP同步功能是为了检测本AS内针对这条路由是否存在黑洞的问题
华为如何解决AS内部路由黑洞的问题：
1. AS内部采用全互联模式（fullmesh）
2. 在边界路由器上将BGP的路由引入IGP协议中（已经被淘汰了）
3. 在边界路由器做GRE隧道
4. 通过LSP隧道解决路由黑洞问题（MPLS）

# BGP的路由如何产生

1. network宣告路由表中已存在的路由条目
2. import引入的方式（引入的路由也需要在路由表中存在）
3. 自动聚合产生的路由条目（只能聚合import的路由条目，主类聚合）[AR1-bgp]summary automatic 
    自动聚合会进行主类汇总，明细路由将被抑制，无法传递给其他邻居
4. 手动聚合 aggregate 172.16.0.0 23（detail-suppressed）
     手动聚合后明细路由默认不会被抑制，可以通过命令让其抑制

# BGP报文类型

1. Open报文
	用于建立BGP邻居的连接, 协商BGP参数的报文
2. update报文
	用于BGP邻居交互路由信息的报文
3. notification报文
	差错报文,用于报错,且中断邻居关系的报文
4. Keepalive报文
	用于保持邻居连接的报文, 用于保活
5. route-refresh报文
	用于在策略改变之后, 请求邻居重新发送路由信息, 并且只有支持路由刷新能力的设备才会响应这个报文

# BGP报文

>BGP报文默认是由两部分组成，分别是BGP报文头，和具体报文内容

## BGP报文头

```
Marker:占用16字节，默认全F，用于检查BGP邻居头部的消息是否完整
Length:占用2字节，用于描述BGP报文的总长度，包括了头部和具体部分
Type  :类型，用于描述当前BGP报文类型，分为 1 2 3 4 5
```

## Open报文 

```
version  ：BGP版本，默认都是版本4
my as    : 描述发送次报文的BGP路由器所处的AS号，如果对端的AS号和本地配置不一致，
           则协商失败，发送notification报文
hold time: 描述路由器邻居失效时间，默认情况是keepalive的3倍，当两端holdtime不一致时，
           需要协商为数值低的使用
bgp id   : 用来描述发出该报文的路由器BGP router-id
optional parameters length:BGP协商参数字段长度
optional parameters:BGP协商参数
```

### holdtime

1. 默认情况下holdtime是keepalive的3倍关系，Keepalive默认60秒。
2. 如果从邻居接收到的holdtime和自己的holdtime相同，则不做改变。
3. 如果从邻居接收到的holdtime大于自己的holdtime，则不做改变。
4. 如果从邻居接收到的holdtime小于自己的holdtime,则协商结果是使用数值小的holdtime执行。
	1. 如果自己的keepalive值小于协商后的holdtime值除以3，那么不做改变。
	2. 如果自己的keepalive值大于协商后的holdtime值除以3，那么选用数值小的执行
   
## update报文 

```
withdrwan routes length:用来描述失效的路由长度
total path attribute length:用来描述携带的属性长度
path attributes:携带的属性
NLRI:网络层可达信息，用来描述所携带的路由的网络号和掩码长度
```
**针对属性不同的路由条目，需要分开Update报文发送**
**具有相同属性的路由条目，可以在一条update报文发送**
withdrwan routes length：如果这个参数不为空，那么下面就会显示携带的需要删除的路由信息

# BGP邻居状态机

![](https://cgqin.github.io/images//202212061457029.png)

## idle

BGP初始状态，一旦配置了BGP的peer以后，或者重置了已存在的peer之后，就会进入这个状态，在这个状态下BGP不会向这个peer发送TCP三次握手，
同时也会拒绝这个peer发来的TCP3次握手。
在进入这个状态时有一个start事件，这个事件会维持32秒，在这个之后开始建立该peer的TCP3次握手，开始建立TCP连接，发送SYN以后，
进入到connect状态
**常见的几种idle状态原因：**
1.  如果没有去往peer的路由，那么就无法发送SYN，此时该peer会一直卡在idle状态。
2.  收到了错误信息之后回退到idle
3.  手动挂起邻居  peer 12.1.1.2 ignore  Idle(Admin)

## connect(连接状态)

在这个状态下，BGP会启动连接重传定时器（connect retry 默认32秒），等待TCP3次握手完成
1. 向邻居发起SYN以后就会进入这个状态，在这个状态要完成TCP3次握手。
2. 如果TCP3次握手完成，则向邻居发送open报文，然后转换到opensent状态。
3. 如果TCP3次握手失败，这将会把这个peer状态改为active
4. 如果重传定时器超时,BGP没有收到邻居的响应，那么会卡在connect状态。

**常见的几种connect状态原因：**
1. 邻居没有给我响应
2. 我的SYN在沿途中可能遇到了阻碍，没有到达对方（也可能是在沿途中没有到达的路由所致）
3. EBGP邻居没有配置TTL多跳
总结一句话，就是SYN没有被邻居响应

## active

当TCP3次握手失败，才会进入到这个状态。
在这个状态下，BGP总是试图去建立TCP3次握手
1. 如果在多次尝试下，TCP3次握手成功了，那么BGP会向该peer发送Open报文，关闭重传定时器，转至opensent状态。
2. 如果在多次尝试下，TCP3次握手仍然失败，那么BGP会将该peer停留在active状态。
3. 如果重传定时器超时（32s），且没有得到该peer的响应，那么会转至connect状态。

## opensent（open报文已发送）

在这个状态下，BGP已经向该peer发送了OPEN报文，在等待对方给我发送OPEN报文。
1. 如果收到了对方发来的OPEN报文，参数协商成功，则会向该peer发送keepalive报文，然后转到openconfirm状态
2. 如果收到了对方发来的OPEN报文，参数协商失败，则会发送notification报文，然后转到idle

## openconfirm（open报文已确认）

1. 在这个状态下 BGP等待对方的keepalive报文，如果收到了对方的Keepalive报文则转换为established状态
2. 在这个状态下 BGP如果收到了notification报文，则转换为idle状态

## established（连接已建立）

在这个状态下说明邻居建立完毕。在这个状态可以交互的报文有：update，notification，Keepalive，route-refresh
1. 如果在这个状态下，收到正确的update和keepalive报文，那么BGP会认为邻居处于正常运行状态，继续保持。
2. 如果在这个状态下，收到错误的update和Keepalive报文，那么BGP会认为邻居处于异常状态，会发送notification报文，并转到idle状态
   route-refresh报文的发送是不会影响邻居关系

# BGP属性

## BGP属性分类

1. 公认必遵属性：所有BGP路由器都可以识别此类属性，且必须存在于Update报文中传递，如果缺少这类属性，路由信息就会出错
2. 公认任意属性：所有BGP路由器都可以识别此类属性，但不要求存在于Update报文中传递，及时缺少了此类属性，路由信息也不会出错
3. 可选过渡：BGP设备可以不识别此类属性，如果一个BGP路由器不识别该属性，它仍然可以接收这类属性，并通告给其他邻居
4. 可选非过渡：BGP设备可以不识别此类属性，如果一个BGP路由器不识别该属性，他会忽略这个属性，也不会通告给其他邻居

# BGP选路原则

1、丢弃下一跳不可达的路由。
2、优选Preference_Value值最高的路由（私有属性，仅本地有效）。
3、优选本地优先级（Local_Preference）最高的路由。
4、优选手动聚合>自动聚合>network>import>从对等体学到的。
5、优选AS_Path短的路由。
6、起源类型IGP>EGP>Incomplete。
7、对于来自同一AS的路由，优选MED值小的。
8、优选从EBGP学来的路由（EBGP>IBGP）。
9、优选AS内部IGP的Metric最小的路由。
10、优选Cluster_List最短的路由。
11、优选Orginator_ID最小的路由。
12、优选Router_ID最小的路由器发布的路由。
13、优选具有较小IP地址的邻居学来的路由。
注释：自上而下序号从小到大依次匹配，且唯一匹配