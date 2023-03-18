vsftpd，ftp服务端

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
