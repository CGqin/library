# 基于接口地址池配置

``` 
interface vlanif 10
    dhcp select interface  //使能接口采用接口地址池的DHCP服务器功能，缺省未使能
    dhcp server lease day 30  //租期的缺省值为1天，修改租期为30天
    dhcp server static-bind ip-address 10.1.1.100 mac-address 00e0-fc12-3456  //为Client_1分配固定的IP地址
```
