# PPP: Point-to-Point Protocal, 点到点链路层协议

- 对物理层而言, 即支持同步链路又异步链路
- 具有良好的扩展性，例如在以太网链路上承载PPP时，可以扩展为PPPoE。
- 提供LCP协议，用于各种链路层参数的协商。
- 提供各种NCP协议(如：IPCP、MPLSP），用于各种网络层参数的协商，更好地支持了网络层协议。
- 提供认证协议CHAP、PAP，更好的保证了网络的安全性。
- 无重传机制，网络开销小，速度快。
![](https://cgqin.github.io/images//202212122338878.png)

# PPP三大协议组件 

|协议|备注|
|:----:|:-----:|
|LCP(Link Control Protocal)|链路控制协议<br/>建立、拆除和监控PPP数据链路|
|NCP(Network Control Protocal)|网络层控制协议<br/>对不同的网路层协议进行连接建立和参数协商|
|CHAP、PAP|扩展协议, 用于认证|

# PPP报文

## 结构
<img src="https://cgqin.github.io/images//202212122356718.png" style="zoom:230%;" />
|字段|备注|
|:---:|:----|
|Flag|标识一个物理帧的起始和结束, 该字节为0x7E|
|Address|唯一标识对端. 该字节填充为全1的广播帧,无实际意义.|
|Control|默认为0x03, 表示无序号帧, PPP默认没有序列号和确定来实现可靠传输.|
|Protocal|Protocal域可用来区分PPP数据帧中信息域所承载的数据包类型.|
|Information|填充域的内容, 最大长度称为最大接收单元MRU, 默认为1500字节.|
|FCS|对PPP数据帧传输的正确性进行校验.|
|Code|标识LCP数据报文的类型|


