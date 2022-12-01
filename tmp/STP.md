# STP的基本概念

1. 根桥: 为了防止和破除网络中环路, STP工作使用的是一个参考点,或者就叫生成树网路的大脑
2. 非根桥: 除了根桥以外的交换机都叫非根桥,也叫做非根交换器
3. 桥ID(BID): 桥优先级(缺省值32768,且必须是4096的倍数)+背板MAC地址
4. root ID: 是指根桥的桥ID
5. 端口ID(PID):  端口优先级(缺省128,且必须是16的倍数)+端口编号
6. PRC: root path cost, 根路径开销(非根桥到达根桥的开销值)
7. BPDU: 桥协议数据单元, 是STP工作的唯一报文

## 端口角色

1. 根端口(root port): 非根交换机上去往根桥最近的端口(用来接收根桥发来的BPDU)
2. 指定端口(D): 会在每一条链路选出一个指定端口,用来发送BPDU
3. 阻塞端口: 
4. 备份端口:

## 端口状态

1. disable: 没有开启STP的端口, 或者关闭接口处于disable状态,该状态下不处理BPDU
2. listening: 接受BPDU报文,发送BPDU报文,不学习MAC地址,不转发数据
3. learning: 接受BPDU报文,发送BPDU报文,学习MAC地址,不转发数据 
4. forwarding: 接受BPDU报文,发送BPDU报文,学习MAC地址,转发数据
5. blocking: 接受BPDU报文,不发送BPDU报文,不学习MAC地址,不转发数据 

listening ----->learning  需要一个转发时延,15s
learning------>forwarding 需要一个转发时延,15s

所有交换机初始状态下, 所有接口都会进入listening状态,开始收敛

## STP的几个计时器

1. hello time: 2s一次
2. max age: 20s 老化时间(20s没有收到上游发来的BPDU,就会造反)
3. forwarding delay: 转发时延  15s
4. message age: 每转发一次+1 默认最大不超过20跳


# STP的工作流程

## 1.选举根桥

根桥的选举使用==桥ID==来选举
先比价优先级==,数值越小越优先==
如果优先级比较不出来,那么就比较交换机背板MAC地址,==越小越优==

## 2.选举根端口

选举原则: 非根交换机接收到的最好的BPDU的接口为根端口
什么叫做好的BPDU? 标准是:
1. root id                    根桥的桥ID
2. RPC                        从接口收到的BPDU中携带的RPC数值越小越优
3. sender BID
4. sender PID
5. local PID

## 3.选取指定端口

在交换机选取出根端口以后, 将会自己为所有DP接口计算出一个未来需要发送的BPDU参数,
1. root id                     根桥的桥ID 
2. rpc                          自己到达根桥的RPC
3. sender BID              自己的桥ID
4. sender PID              自己认为是DP接口的PID 
参数准备完毕后, 这个链路上的两个接口都会发出认为自己的DP接口的BPDU, 就要进行
收到对端的BPDU后,比较之后,认为自己的BPDU更好的话, 那么DP接口不会改变角色
收到对端的BPDU后,比较之后,认为自己的BPDU不够优秀, 那么DP接口会该改变为AP

