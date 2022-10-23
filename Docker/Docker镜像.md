# docker 镜像

> 镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好行程一个可交付的运行环境（包括代码、运行时需要的库、环境变量和配置文件等），这个打包好的运行环境就是image镜像文件。

## Docker 镜像加载原理

###  联合文件系统

1. Docker 中的文件存储驱动叫做 storage driver。
2. Docker 最早支持的stotage driver是 AUFS，它实际上由一层一层的文件系统组成，这种层级的文件系统叫UnionFS。
3. 联合文件系统（UnionFS）：Union 文件系统，是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite serveral directories into a single virtual filesystem）。
4. Union文件系统是Docker镜像的基础。镜像可以通过分层来进行集成，基于基础镜像可以制作具体的应用镜像。
5. 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。
6. 后来出现的docker版本中，除了AUFS，还支持OverlayFS、Btrfs、Device Mapper、VFS、ZFS等storage driver。

### bootfs和rootfs

1.  bootfs（boot file system）主要包含 bootloader 和 kernel，bootloader主要是引导加载 kernel，Linux刚启动时会加载bootfs文件系统。
2. 在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已经由 bootfs 转交给内核，此时系统也会卸载 bootfs。
3. rootfs（root file system），在bootfs之上，包含的就是典型Linux系统中的 /dev、/proc、/bin、/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu、CentOS等。
4. docker镜像底层层次：

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2022/09/10/kuangstudy1aaaed78-be57-4dff-97ae-884c66844cc5.jpg)

> 对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接使用Host的Kernel，自己只需要提供rootfs就可以。所以，对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，不同的发行版可以共用bootfs。





> 有差别的rootfs:

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2022/09/10/kuangstudya4497a6c-2a9a-48c3-aeef-89d1fbb9c2dd.jpg)

### 镜像分层



Docker支持扩展现有镜像，创建新的镜像。新镜像是从base镜像一层一层叠加生成的。
例如：

```dockerfile
# Version: 0.0.1
FROM debian  # 直接在debain base镜像上构建
MAINTAINER mylinux
RUN apt-get update && apt-get install -y emacs # 安装emacs
RUN apt-get install -y apache2 # 安装apache2
CMD ["/bin/bash"] # 容器启动时运行bash
```

镜像创建过程：

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2022/09/10/kuangstudybc7c227e-d2a4-4dcb-b638-6bf3b32a41af.jpg)

### 镜像分层的优势

> 镜像分层的一个最大好处就是共享资源，方便复制迁移，方便复用。

### 容器层

> 当容器启动时，一个新的可写层将被加载到镜像的顶部，这一层通常被称为容器层，容器层之下的都叫镜像层。
> 所有对容器的改动，无论添加、删除、还是修改文件都只会发生在容器层中。
> 只有容器层是可写的，容器层下面的所有镜像层都是只读的。

如图：
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2022/09/10/kuangstudyd8bbb73b-fb84-4781-899b-31adb22fe25e.jpg)

## Docker 镜像使用

### 列出镜像列表

命令: `docker images`

```shell
[root@localhost ~]# docker images           
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        5 days ago          188 MB
php                 5.6                 f40e9e0f10c8        9 days ago          444.8 MB
nginx               latest              6f8d099c3adc        12 days ago         182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago         324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago         194.4 MB
ubuntu              15.10               4e3b13c8a266        4 weeks ago         136.3 MB
hello-world         latest              690ed74de00f        6 months ago        960 B
training/webapp     latest              6fae60ef3446        11 months ago       348.8 MB
```



各个选项说明:

- **REPOSITORY：**表示镜像的仓库源
- **TAG：**镜像的标签
- **IMAGE ID：**镜像ID
- **CREATED：**镜像创建时间
- **SIZE：**镜像大小

### 查找镜像

命令: `docker search 镜像名称`

```shell
[root@localhost ~]# docker search httpd
NAME                                 DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
httpd                                The Apache HTTP Server Project                  4159      [OK]       
centos/httpd-24-centos7              Platform for running Apache httpd 2.4 or bui…   44                   
centos/httpd                                                                         35                   [OK]
clearlinux/httpd                     httpd HyperText Transfer Protocol (HTTP) ser…   2                    
solsson/httpd-openidc                mod_auth_openidc on official httpd image, ve…   2                    [OK]
hypoport/httpd-cgi                   httpd-cgi                                       2                    [OK]
dockerpinata/httpd                                                                   1                    
inanimate/httpd-ssl                  A play container with httpd, ssl enabled, an…   1                    [OK]
publici/httpd                        httpd:latest                                    1                    [OK]
centos/httpd-24-centos8                                                              1                    
dariko/httpd-rproxy-ldap             Apache httpd reverse proxy with LDAP authent…   1                    [OK]
manageiq/httpd                       Container with httpd, built on CentOS for Ma…   1                    [OK]
nnasaki/httpd-ssi                    SSI enabled Apache 2.4 on Alpine Linux          1                    
lead4good/httpd-fpm                  httpd server which connects via fcgi proxy h…   1                    [OK]
patrickha/httpd-err                                                                  0                    
manasip/httpd                                                                        0                    
amd64/httpd                          The Apache HTTP Server Project                  0                    
httpdss/archerysec                   ArcherySec repository                           0                    [OK]
e2eteam/httpd                                                                        0                    
manageiq/httpd_configmap_generator   Httpd Configmap Generator                       0                    [OK]
paketobuildpacks/httpd                                                               0                    
sandeep1988/httpd-new                httpd-new                                       0                    
19022021/httpd-connection_test       This httpd image will test the connectivity …   0                    
ppc64le/httpd                        The Apache HTTP Server Project                  0                    
httpdocker/kubia                                                                     0                    
```

各个选项说明:

- **NAME:** 镜像仓库源的名称

- **DESCRIPTION:** 镜像的描述

- **OFFICIAL:** 是否 docker 官方发布

- **stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思。

- **AUTOMATED:** 自动构建。

### 获取一个新镜像

命令: `docker pull 镜像名称`

```shell
[root@localhost ~]# docker pull ubuntu:13.10
13.10: Pulling from library/ubuntu
6599cadaf950: Pull complete 
23eda618d451: Pull complete 
f0be3084efe9: Pull complete 
52de432f084b: Pull complete 
a3ed95caeb02: Pull complete 
Digest: sha256:15b79a6654811c8d992ebacdfbd5152fcf3d165e374e264076aa435214a947a3
Status: Downloaded newer image for ubuntu:13.10
```

### 删除镜像

命令: `docker rmi -f 镜像名称/镜像Id`

```shell
[root@localhost ~]# docker rmi --help

Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
[root@localhost ~]# docker rmi -f hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
```

### 查看镜像/容器/数据卷所占的空间

命令: `docker system df`

```shell
[root@localhost ~]# docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          8         2         3.139GB   2.766GB (88%)
Containers      4         3         1.132kB   0B (0%)
Local Volumes   3         0         271.9kB   271.9kB (100%)
Build Cache     0         0         0B        0B
```

