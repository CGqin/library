vsftpd，ftp服务端
参考文档:
- https://www.cnblogs.com/rxysg/p/15698426.html
- https://wiki.ubuntu.com.cn/Vsftpd

# stand alone和super daemon

- stand alone: 指的是一直运行vsftpd，占用资源，提供ftp服务。
- super daemon: 指的是有需要时由xinetd启动vsftpd服务。
- 如果服务器不是那种长期开ftp，提供大量的上传下载服务的话，会选择后者。

# 文件结构

- 匿名用户根路径
```bash
/srv/ftp
```
- 配置文件
```bash
/etc/vsftpd/
├── ftpusers
├── user_list
├── vsftpd.conf
└── vsftpd_conf_migrate.sh
```
- 查阅配置文件详细信息
```bash
man 5 vsftpd.conf
```

# 运行

## standalone

最普遍的方式
```bash
sudo systemctl start vsftpd
```

## super daemon 

1. 修改vsftpd.conf
```
listen=NO
```
---
这里若不改成NO，会出现下列错误
500 OOPS: could not bind listening IPv4 socket

2. 安装并配置xinetd
```bash
$ sudo yum install -y xinetd
$ sudo vi /etc/xinetd.conf
service ftp
{
        socket_type             = stream
        wait                    = no
        user                    = root
        server                  = /usr/sbin/vsftpd
        log_on_success          += DURATION USERID
        log_on_failure          += USERID
        nice                    = 10
        disable                 = no
}
```
3. 停止vsftpd，启动xinetd
```bash
sudo systemctl stop vsftpd 
sudo service xinetd start 
```

# /etc/vsftpd/vsftpd.conf

```bash
listen=<YES/NO> :设置为YES时vsftpd以独立运行方式启动，设置为NO时以xinetd方式启动（xinetd是管理守护进程的，将服务集中管理，可以减少大量服务的资源消耗）
listen_port=<port> :设置控制连接的监听端口号，默认为21
listen_address=<ip address> :将在绑定到指定IP地址运行，适合多网卡
connect_from_port_20=<YES/NO> :若为YES，则强迫FTP－DATA的数据传送使用port 20，默认YES
pasv_enable=<YES/NO> :是否使用被动模式的数据连接，如果客户机在防火墙后，请开启为YES
pasv_min_port=<n>
pasv_max_port=<m> :设置被动模式后的数据连接端口范围在n和m之间,建议为50000－60000端口
message_file=<filename> :设置使用者进入某个目录时显示的文件内容，默认为 .message
dirmessage_enable=<YES/NO> :设置使用者进入某个目录时是否显示由message_file指定的文件内容
ftpd_banner=<message> :设置用户连接服务器后的显示信息，就是欢迎信息
banner_file=<filename> :设置用户连接服务器后的显示信息存放在指定的filename文件中
connect_timeout=<n> :如果客户机连接服务器超过N秒，则强制断线，默认60
accept_timeout=<n> :当使用者以被动模式进行数据传输时，服务器发出passive port指令等待客户机超过N秒，则强制断线，默认60
accept_connection_timeout=<n> :设置空闲的数据连接在N秒后中断，默认120
data_connection_timeout=<n> : 设置空闲的用户会话在N秒后中断，默认300
max_clients=<n> : 在独立启动时限制服务器的连接数，0表示无限制
max_per_ip=<n> :在独立启动时限制客户机每IP的连接数，0表示无限制（不知道是否跟多线程下载有没干系）
local_enable=<YES/NO> :设置是否支持本地用户帐号访问
guest_enable=<YES/NO> :设置是否支持虚拟用户帐号访问
write_enable=<YES/NO> :是否开放本地用户的写权限
local_umask=<nnn> :设置本地用户上传的文件的生成掩码，默认为077
local_max_rate<n> :设置本地用户最大的传输速率，单位为bytes/sec，值为0表示不限制
local_root=<file> :设置本地用户登陆后的目录，默认为本地用户的主目录
chroot_local_user=<YES/NO> :当为YES时，所有本地用户可以执行chroot
chroot_list_enable=<YES/NO> 
chroot_list_file=<filename> :当chroot_local_user=NO 且 chroot_list_enable=YES时，只有filename文件指定的用户可以执行chroot
anonymous_enable=<YES/NO> :设置是否支持匿名用户访问
anon_max_rate=<n> :设置匿名用户的最大传输速率，单位为B/s，值为0表示不限制
anon_world_readable_only=<YES/NO> 是否开放匿名用户的浏览权限
anon_upload_enable=<YES/NO> 设置是否允许匿名用户上传
anon_mkdir_write_enable=<YES/NO> :设置是否允许匿名用户创建目录
anon_other_write_enable=<YES/NO> :设置是否允许匿名用户其他的写权限（注意，这个在安全上比较重要，一般不建议开，不过关闭会不支持续传）
anon_umask=<nnn> :设置匿名用户上传的文件的生成掩码，默认为077
```

# 允许匿名访问

```bash
# Allow anonymous FTP? (Disabled by default)
anonymous_enable=YES
```

# 允许匿名上传

```bash
write_enable=YES
anon_mkdir_write_enable=YES
anon_upload_enable=YES
```
---
注意:
1. 匿名用户就是**ftp**，想要匿名用户写入，必须文件夹的权限为ftp可写。
2. 匿名用户的根目录不允许**写**，所以根目录的权限绝对不能是ftp可写和**其他用户**可写，如果根目录所有者为ftp的话，所有者的权限也不能写。

所以解决方法是建个单独的public文件夹用于上传文件，设置其为ftp可写或”其他用户可写“
还可建个download文件夹只用于下载，设置其他用户没有写权限便可。

# 允许匿名用户重命名、删除文件

开放重命名，删除文件等权限，不开的话没法续传。
```bash
anon_other_write_enable=YES
```

# 仅能上传，无法下载

```bash
write_enable=YES
anon_mkdir_write_enable=YES
anon_upload_enable=YES
chown_uploads=YES
chown_username=root         # 上传的文件所有者被改为root，匿名用户的ftp用户就无法读取，下载了。
```

# 配置vsftpd 系统认证

```
local_enable=YES          # 设定vsftp认证系统用户
write_enable=YES          # 允许他们上传文件
```

# Chroot

## 限制所有

```
chroot_local_user=YES
```

## 开放所有，限制特定(黑名单)

```
chroot_local_user=NO
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list    # 将限制的用户写在此列表中
```

## 限制所有，开放特定(白名单)

```
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list    # 将开放的用户写在列表中
```

# 账号登录

## /etc/ftpusers文件

该文件内的用户一律禁止ftp连接，默认列表包括了root, daemon, nobody等。需要禁止某个用户，添加进来便是。
这个文件是由PAM模块的 /etc/pam.d/vsftpd 指定的
```bash
$ sudo cat /etc/pam.d/vsftpd
# Standard behaviour for ftpd(8).
auth    required        pam_listfile.so item=user sense=deny file=/etc/ftpusers onerr=succeed

# Note: vsftpd handles anonymous logins on its own. Do not enable pam_ftp.so.

# Standard pam includes
@include common-account
@include common-session
@include common-auth
auth    required        pam_shells.so
```

## *userlist_file* 文件

vsftpd自订的列表，跟/etc/ftpusers类似，具体文件名和路径是由用户自己指定的。
```bash
userlist_enable=YES
userlist_deny=YES/NO
userlist_file=/etc/vsftpd.user_list
```

- 当 `userlist_deny=YES` 时, userlist_file该内的用户一律禁止ftp连接
- 当 `userlist_deny=NO` 时, 限制一切用户登录，只允许列表文件中的用户

# 欢迎信息

```bash
dirmessage_enable=YES
```

在各个用户home目录下的 `.message` 编辑欢迎信息(匿名用户就放到/var/ftp)
```bash
$ cat .message 
欢迎来到vsftpd
```

# 使用虚拟用户登入

这样就可以用虚拟账号登入了 ftpadmin、ftpuser分别登入之后访问的是ftp3、ftp1目录

```bash
#创建两个目录
[root@localhost ~]#mkdir /home/vsftpd/ftp1
[root@localhost ~]# mkdir /home/vsftpd/ftp3

[root@localhost ~]# vi /etc/vsftpd/loginuser.txt

#加入两个用户  奇数行代表用户名 偶数行代表密码
xftpadmin
123456
xftpuser
123456

#执行命令 生成虚拟数据库
[root@localhost ~]# db_load -T -t hash -f /etc/vsftpd/loginuser.txt /etc/vsftpd/login.db
#设置数据库文件的访问权限
[root@localhost ~]# chmod 600 /etc/vsftpd/login.db

[root@localhost ~]# vi /etc/pam.d/vsftpd

#将以下内容增加的原文件前面两行：
auth required pam_userdb.so db=/etc/vsftpd/login
account required pam_userdb.so db=/etc/vsftpd/login
       #我们建立的虚拟用户将采用PAM进行验证，这是通过/etc/vsftpd.conf文件中的 语句pam_service_name=vsftpd.vu来启用的。

vsftpd使用的pam文件
auth    sufficient      pam_userdb.so db=/etc/vsftpd/login
account sufficient      pam_userdb.so db=/etc/vsftpd/login

#auth       required     pam_listfile.so item=user sense=deny file=/etc/vsftpd.ftpusers onerr=succeed
#auth       required     pam_stack.so service=system-auth
#auth       required     pam_shells.so
#account    required     pam_stack.so service=system-auth
#session    required     pam_stack.so service=system-auth

可以看出前面两行是对虚拟用户的验证，后面是对系统用户的验证。 为了安全我一般把系统用户的登入关闭  使用虚拟账号登入ftp
对虚拟用户的验证使用了sufficient这个控制标志。
这个标志的含义是如果这个模块验证通过，就不必使用后面的层叠模块进行验证了；但如果失败了，
就继续后面的认证，也就是使用系统真实用户的验证。
虚拟用户创建本地系统用户

#新建一个系统用户vsftpd, 用户登录终端设为/bin/false(即使之不能登录系统)
[root@localhost ~]# useradd vsftpd -d /home/vsftpd -s /bin/false 
[root@localhost ~]# chown vsftpd:vsftpd /home/vsftpd #改变目录所属用户组

根据需要创建/etc/vsftpd/vsftpd.conf，以下设置： 
listen=YES                       #监听为专用模式
anonymous_enable=NO              #禁用匿名登入
dirmessage_enable=YES
xferlog_enable=YES
xferlog_file=/var/log/vsftpd.log   #记录ftp操作日志
xferlog_std_format=YES
chroot_local_user=YES          #对用户访问只限制在主目录 不能访问其他目录
guest_enable=YES               #启用guest
guest_username=vsftpd           #使用虚拟账号形式
user_config_dir=/etc/vsftpd_user_conf  #虚拟账号配置目录
pam_service_name=vsftpd              #对vsftpd的用户使用pam认证
local_enable=YES

#执行以下命令
[root@localhost ~]# mkdir /etc/vsftpd/user_conf
[root@localhost ~]# cd /etc/vsftpd/user_conf
[root@localhost ~]# touch xftpuser xftpadmin #创建两个文件

[root@localhost ~]# vi /etc/vsftpd/user_config/xftpadmin

#加入以下内容 拥有所有权限
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
local_root=/home/vsftpd/ftp3

[root@localhost ~]# vi /etc/vsftpd/user_config/xftpuser

#加入以下内容 只读权限
local_root=/home/vsftpd/ftp1

#登入 使用ftp1用户登入看看 
如果不能读写操作 可能是目录权限不够需要设置权限 试试看
[root@localhost ~]# chmod 777 /home/vsftpd/ftp3
```

# 使用ssl登入

这样重启vsftpd 就可以用客户端来尝试进行SSL加密连接了

```bash
[root@localhost ~]# cd /etc/pki/tls/certs/
[root@localhost ~]# openssl req -new -x509 -nodes -out vsftpd.pem -keyout vsftpd.pem

4 修改vsftpd.conf文件
[root@localhost ~]#vi /usr/local/etc/vsftpd.conf .

ssl_enable=YES(开启vsftpd对ssl协议的支持)
ssl_sslv2=YES（支持SSL v2 protocol）
ssl_sslv3=YES（支持SSL v3 protocol）
ssl_tlsv1=YES（支持TSL v1）
rsa_cert_file=/etc/pki/tls/certs/vsftpd.pem（证书的路径）

ssl_enable=YES
ssl_sslv2=YES
ssl_sslv3=YES
ssl_tlsv1=YES
```