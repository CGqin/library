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

>以==MD5==隐藏密码, ==三次握手==机制, ==发起方为认证方==, 有效避免暴力破解, 在链路建立成功后具有==再次认证==检测机制,推荐使用

|报文类型|备注|
|:-------:|----|
|Challenge|认证方发送Challenge, 发起认证过程|
|Response|被认证方返回用户信息|
|Success|认证方发送认证成功信息|
|Failure|认真放发送认证失败信息|
![](https://cgqin.github.io/images//202212131531750.png)

# PPP认证配置

**认证方配置**
```
link-protocal ppp        // ppp链路默认配置
ppp authentication-mode pap/chap        // 配置PPP认证方式
aaa                                     // 创建PPP认证用户
	local-user wakin password cipher huawei 
	local-user wakin service-type ppp
```
**被认证方配置**
```
ppp pap local-user wakin password cipher huawei               // 配置PAP凭证

ppp chap user wakin                                           // 配置CHAP凭证
ppp chap passwd cipher huawei           
```

# NCP协议协商

## IPCP: 用于协商控制IP参数, 使PPP可用于传输IP数据包

>使用和LCP相同的协商机制、报文类型, 当IPCP并非调用LCP, 只是工作过程、报文等和LCP相同

## IPCP静态协商地址
![](https://cgqin.github.io/images//202212131951652.png)

## IPCP动态协商IP地址
![](https://cgqin.github.io/images//202212131954966.png)

## 配置

`ip address ppp-negotiate` : 配置接口的IP地址可协商属性
`remote address ip-address/pool name` : 配置为客户端分配的IP地址

**认证方**
```
aaa
	local-user huawei password cipher hello
	local-user huawei service-type ppp
int s1/0/0
	link-protocal ppp
	ip address 12.1.1.1 255.255.255.0
	remote address 12.1.1.2            // 为客户端分配地址
	ppp authentication-mode chap
	ppp chap user huawei
```
**被认证方**
```
int s1/0/0
	link-protocal ppp
	ip address ppp-negotiate           // 协商地址
	ppp chap user huawei 
	ppp chap passwd cipher hello
```

# MP: MultiLink PPP, 将多个PPP链路捆绑使用

- 实现**增加带宽、负载均衡、链路备份**
- 允许将报文分片,分片将多个点对点链路上送到同一个目的地,从而降低延迟
![](https://cgqin.github.io/images//202212132018262.png)
- **链路协商过程:**
	- LCP阶段, 需证对端接口是否工作在MP方式下
	- NCP阶段，根据MP-Group接口或指定虚拟接口模板的各项NCP参数（如IP地址等）进行NCP协商。
- **实现方式:**
	- 虚拟接口模块方式
	- MP-Group方式
- **相关配置:**
`int mp-group 0/0/1` : 创建并配置MP-Group
`ppp mp mp-group 0/0/1` : 将接口加入指定的MP-group
==PS: 配置完成后,重启所有成员接口==
`display ppp mp` : 验证
`display int mp-group` : 验证
![](https://cgqin.github.io/images//202212132034786.png)

# PPPoE: PPP over Ethernet

- 把==PPP帧封装到以太网帧==中的链路层协议
- 使以太网网路中的多台主机连接到远端的宽带接入服务器
- 即以太网网路中的多台主机接入,PPP提供访问控制和计费
- 具有适用范围广、安全性高、计费方便的特点
- 组网结构采用==Client/Server==模型
![](https://cgqin.github.io/images//202212151422488.png)
![](https://cgqin.github.io/images//202212151424188.png)

## PPPoE应用场景

![](https://cgqin.github.io/images//202212151431837.png)

![](https://cgqin.github.io/images//202212151439679.png)
## PPPoE报文结构和类型
![](https://cgqin.github.io/images//202212151507133.png)
|Code|类型|备注|
|:----:|:---:|----|
|0x09|PADI|PPPoE发现初始报文|
|0x07|PADO|PPPoE发现提供报文|
|0x19|PADR|PPPoE发现请求|
|0x65|PADS|PPPoE发现会话确认报文|
|0xa7|PADT|PPPoE发现终止报文|
|0x00| |会话数据|
## PPPoE会话建立过程
![](https://cgqin.github.io/images//202212151517703.png)
**Discovery :** 获取对方以太网地址, 已经确认唯一的PPPoE会话
**Session :** 包含两部分: PPP协商阶段和PPP报文传输阶段
**Terminate :** 会话建立以后的任意时刻, 发送报文结束PPPoE会话

### 完整流程

1. Discovery:
![](https://cgqin.github.io/images//202212151531971.png)
![](https://cgqin.github.io/images//202212151534808.png)
![](https://cgqin.github.io/images//202212151536190.png)
![](https://cgqin.github.io/images//202212151538853.png)
2. Session:
![](https://cgqin.github.io/images//202212151540003.png)
3. Terminate:
![](https://cgqin.github.io/images//202212151541959.png)
