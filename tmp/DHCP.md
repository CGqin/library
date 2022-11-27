# 基于接口地址池配置

``` 
interface vlanif 10
    dhcp select interface  //开启接口dhcp分配
    dhcp server lease day 30  //租期的缺省值为1天，修改租期为30天
    dhcp server static-bind ip-address 10.1.1.100 mac-address 00e0-fc12-3456  //为Client_1分配固定的IP地址
dhcp server database       //开启dhcp数据保存功能
```

# 基于全局地址池

```

```