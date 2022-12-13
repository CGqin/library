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
|PS|Address和Control一起标识此报文为PPP报文, 即PPP报文头为FF03.|
|Protocal|Protocal域可用来区分PPP数据帧中信息域所承载的数据包类型.|
|Information|填充域的内容, 最大长度称为最大接收单元MRU, 默认为1500字节.|
|FCS|对PPP数据帧传输的正确性进行校验.|
|Code|标识LCP数据报文的类型|

## 类型
|Code值|报文类型|备注|
|:---:|:-------:|:----:|
|0x01|Configure-Request|匹配请求|
|0x02|Configure-Ack|匹配确认|
|0x03|Configure-Nak|匹配否认-不接受|
|0x04|Config-Reject|匹配拒接-不识别|
|0x05|Terminate-Request|终止请求|
|0x06|Terminate-Ack|终止确认|
|0x07|Code-Reject|代码拒接|
|0x08|Protocal-Reject|协议拒接|
|0x09|Echo-Request|回拨请求|
|0x0A|Echo-Reply|回拨确认|
|0x0B|Discard-Request|抛弃请求|
|0x0C|Reserved|保留|

# PPP链路建立过程

![](https://cgqin.github.io/images//202212131234942.png)

|阶段|备注|
|:---:|:---:|
|==Dead==|==链路不可用阶段==<br/>当两端检测到物理线路激活时, 迁移至Establish阶段|
|==Establish==|==链路建立阶段==<br/>进行LCP参数协商,内容包括MRC、认证方式、魔术字等<br/>协商成功后进行Opened状态,表示底层链路已经建立|
|==Authenticate==|==认证阶段==<br/>进行CHAP后PAP验证|
|==Network==|==网络层协商阶段==<br/>进行NCP协商,只有响应的网路层协议协商成功后,才可以发送报文|
|==Terminate==|==网络终止协议==<br/>中断连接,物理链路断开、认证失败、超时定时器时间到、管理源通过配置关闭<br/>如果所用的资源都被释放,通信双方将回到Dead阶段,直到通信双方重新建立PPP连接|

# LCP协商参数
|参数名称|功能描述|协商规则|默认值|
|:-------:|:-------:|:--------|:------:|
|最大接受单元MRU|PPP数据帧中Information字段的总长度|使用两端设置的较小值|1500|
|认证协议|认证对端使用的认证协议|被认证方必须支持认证方使用的认证协议并正确配置, 否则协商不成功|不认证|
|魔术字Magic-Number|魔术字为一个随机产生的数字，用于检测链路环路,<br/>如果收到的LCP报文中的魔术字和本地产生的魔术字相同，则认为链路有环路。|一端支持而另一端不支持,表示链路无环路，认为协商成功；两端都支持则使用检测机制检测环路。|启用|
## 协商成功
![](https://cgqin.github.io/images//202212131352139.png)
## 协商失败(不接受)
![](https://cgqin.github.io/images//202212131356400.png)
## 协商失败(无法识别)
![](https://cgqin.github.io/images//202212131400457.png)
## 链路检测(默认周期为10s)
![](https://cgqin.github.io/images//202212131438489.png)
## 连接关闭
![](https://cgqin.github.io/images//202212131448606.png)

# PPP认证协议

## PAP: Password Authentication Protocol, 密码认证协议

>以明文发送密码, 二次握手机制,发起方为被认证方,可以无限次的尝试(暴力破解), 只在链路建立的阶段进行认证,一旦链路建立成功将不在认证 

|报文类型|备注|
|:-------:|----|
|Authenticate-Request|被认证方发送用户名和密码|
|Authenticate-Ack|认证方发送验证成功信息|
|Authenticate-Nak|认证方发送验证失败信息|
![](https://cgqin.github.io/images//202212131514695.png)

## CHAP: Challenge Handshake Authentication Protocal , 挑战/质询握手认证协议

