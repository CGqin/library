# DHCP报文类型


# DHCP基本工作过程



# 基于接口地址池配置

``` 
dhcp enable
interface vlanif 10
    dhcp select interface  //开启接口dhcp分配
    dhcp server lease day 30  //租期的缺省值为1天，修改租期为30天
    dhcp server static-bind ip-address 10.1.1.100 mac-address 00e0-fc12-3456  //为Client_1分配固定的IP地址
dhcp server database       //开启dhcp数据保存功能
```

# 基于全局地址池

```
dhcp enable
ip pool pool1      //创建一个地址池
	network 10.1.1.0 mask 255.255.255.128
	dns-list 10.1.2.3
	gateway-list 10.1.1.1
	lease day 10
interface g0/0/0
    dhcp select global
```

# DHCP中继

```
#dhcp服务器
dhcp enable 
ip pool poll1       //创建一个地址池
	network 10.1.1.0 mask 255.255.255.0
	dns-list 10.1.1.254
	gateway-list 10.1.1.254
	lease day 30
#中继
dhcp enable 
interface g0/0/0
	dhcp select relay           //开启中继功能
	dhcp relay server-ip 23.1.1.1       //配置DHCP中继代理的DHCP服务器的IP地址
#多中继
dhcp enable 
interface g0/0/0                        //dhcp流量进接口处配置
	dhcp select relay 
	dhcp relay server-ip 12.1.1.1       //指向下一个中继
```


