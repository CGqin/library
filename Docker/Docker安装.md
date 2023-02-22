# Docker安装

==运行环境: Centos8==



### 1. 卸载原先的Docker

```shell
 sudo yum remove docker \ 
                 docker-client \                  
                 docker-client-latest \                  
                 docker-common \                  
                 docker-latest \                  
                 docker-latest-logrotate \                  
                 docker-logrotate \                  
                 docker-engine
```


### 2. 安装gcc

```shell
yum -y install gcc
yum -y install gcc-c++
```

### 3. 安装所需要的软件包

```shell
yum install -y yum-utils
```

### 4. 设置stable镜像仓库（采用Ali云仓库）

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 5. 更新yum软件包索引

```shell
yum makecache fast
```

### 6. 安装DOCKER CE

```shell
 yum install docker-ce docker-ce-cli containerd.io
```

### 7. 配置阿里云镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{  
"registry-mirrors": ["https://le2tknys.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 8. 查看安装Docker版本

```shell
docker version
```

![image-20220916232055300](C:\Users\Cxuejia\AppData\Roaming\Typora\typora-user-images\image-20220916232055300.png)


```
#!/bin/sh
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
yum -y install gcc
yum -y install gcc-c++
yum install -y yum-utils 
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache 
yum install docker-ce docker-ce-cli containerd.io
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{  
"registry-mirrors": ["https://le2tknys.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```