# SwitchA
```
#
sysname SwitchA
#
vlan batch 100 300
#
interface Vlanif100
 ip address 10.1.1.1 255.255.255.0
 vrrp vrid 1 virtual-ip 10.1.1.111
 vrrp vrid 1 priority 120 配置较高优先级
 vrrp vrid 1 preempt-mode timer delay 20  20秒抢占延时
```
# SwitchB
```
#
sysname SwitchB
#
vlan batch 100 200
#
interface Vlanif100
 ip address 10.1.1.2 255.255.255.0
 vrrp vrid 1 virtual-ip 10.1.1.111
#
inter
```