# 常用配置
```
ddns-update-style (none|interim|ad-hoc)
作用：定义所支持的DNS动态更新类型。
none：表示不支持动态更新
interim：表示DNS互动更新模式
ad-hoc：表示特殊DNS更新模式
因为DHCP 客户端所取得的IP 通常是一直变动的，所以某部主机的主机名与IP 的对应就很难处理。此时DHCP 可以透过ddns 来更新主机名与IP 的对应。如果需要这个配置，则要放置在第一行

gnore client-updates
作用：忽略客户端更新

option   domain-name "example.org";      
域

option   domain-name-servers  8.8.8.8 ;    
DNS域名解析服务

default-lease-time 600;     ip地址的默认租约时间，单位是秒
max-lease-time 7200;        租约的最大时间
log-facility local7;        日志的级别保存，可以查看/etc/rsyslog.conf

subnet 10.0.0.0 netmask 255.255.255.0 {
注意：网络号必须与DHCP服务器的网络号相同
上面的是全局配置，{}里面是局部配置，局部配置能覆盖全局配置
  range 10.0.0.2  10.0.0.254;  起始IP地址结束IP地址
  option domain-name-servers 8.8.8.8; 为客户端指定DNS服务器地址
  option domain-name "internal.example.org";
  option routers 10.0.0.1; 为客户端指定默认网关
  option broadcast-address 10.0.0.255; 
  设定广播地址而已。如果没有设定的话，系统应该会自动依据class A, B, C 的原则来计算出广播地址
  default-lease-time 600;
  max-lease-time 7200;
}

subnet 10.152.187.0 netmask 255.255.255.0 {         
可以设置多个网段的配置（可选）
}

host fantasia {
IP地址和MAC绑定必须是一个单独的配置，可以有多个配置（可选），fantasia是名字可以随意选定
  hardware ethernet 08:00:07:26:c0:a5;      IP地址和MAC绑定
  fixed-address  10.0.0.222;
}

host passacaglia {
  hardware ethernet 0:0:c0:5d:bd:95;
  filename "vmunix.passacaglia";
  引导文件一般是pxelinux.0             关于无人值守的配置
  server-name "toccata.fugue.com";       tftp文件存放的服务器
}

```