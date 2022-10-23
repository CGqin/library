# docker 容器

## docker 容器使用

### 启动容器

命令: `docker run [OPTIONS]`



常用选项说明:

	- **--name="容器的名字"** : 为容器指定一个名称
	- **-d** : 后台运行容器并返回容器id
	- **-i** : 以交互模式运行容器，通常与-t同时使用 -it
	- **-t**: 终端
	- **-P** : 随机端口映射，大写P
	    - **-p** : 指定端口映射，小写p



实例:使用 **centos** 镜像启动一个容器，参数为以命令行模式进入该容器：

```shell
[root@localhost ~]# docker run -it --name=centos01 centos /bin/bash
[root@a09cb672fdb1 /]# 

#-i: 交互式操作
#-t: 终端
#centos: 镜像
#/bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash
```



### 列出正在运行的容器

命令: `docker ps`



常用选项说明:

 - **-a** : 列出历史和正在运行的容器实例

- **-q**: 仅列出容器的 id



```shell
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
cfd4c82f058f   centos    "/bin/bash"   About a minute ago   Up About a minute             centos01
[root@localhost ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS                     PORTS     NAMES
105908430bed   centos    "/bin/bash"   11 seconds ago       Exited (0) 7 seconds ago             centos02
cfd4c82f058f   centos    "/bin/bash"   About a minute ago   Up About a minute                    centos01
[root@localhost ~]# docker ps -aq
105908430bed
cfd4c82f058f
```

### 退出容器

命令 : `exit`  # 退出后容器关闭

快捷键: `Ctrl+P+Q` #退出后容器不关闭



```shell
[root@localhost ~]# docker run -it --name=centos01 centos /bin/bash
[root@cfd4c82f058f /]# exit
exit
[root@localhost ~]# docker ps			#发现 centos01 关闭了
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
[root@localhost ~]# docker run -it --name=centos02 centos /bin/bash
[root@105908430bed /]# 按下 Ctrl+P+Q
[root@localhost ~]# docker ps			#发现 centos02 没有关闭
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
105908430bed   centos    "/bin/bash"   2 minutes ago   Up 1 minutes             centos02
```



### 进入容器

命令:

-  `docker attach`
- `docker exec`

> 推荐使用 docker exec 命令，因为此命令会退出容器终端，但不会导致容器的停止



#### attach 命令

```shell
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
9fbf82c30ab0   centos    "/bin/bash"   8 seconds ago    Up 6 seconds              centos02
973d557edeed   centos    "/bin/bash"   18 seconds ago   Up 16 seconds             centos01
[root@localhost ~]# docker attach centos01		#进入 centos01
[root@973d557edeed /]# exit
exit
[root@localhost ~]# docker ps			#退出 centos01 后, 容器停止
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
9fbf82c30ab0   centos    "/bin/bash"   About a minute ago   Up About a minute             centos02
```

==**注意：** 如果从这个容器退出，会导致容器的停止。==



#### exec 命令

```shell
[root@localhost ~]# docker exec -it centos02 /bin/bash				# 进入 centos02 容器
[root@9fbf82c30ab0 /]# exit
exit
[root@localhost ~]# docker ps				#退出 centos02 后, 容器还在运行
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
9fbf82c30ab0   centos    "/bin/bash"   10 minutes ago   Up 10 minutes             centos02
```

==**注意：** 如果从这个容器退出，容器不会停止，这就是为什么推荐大家使用 **docker exec** 的原因。==



### 删除容器

命令: `docker rm -f 容器ID或者容器名称`



选项:

	- **-f** : 强制删除

 



```shell
[root@localhost ~]# docker rm -f centos01					# 删除一个容器
centos01
[root@localhost ~]# docker rm -f centos01 centos02 			# 删除多个容器
centos01 centos02
[root@localhost ~]# docker rm -f $(docker ps -aq)			# 删除全部容器
d59dc85cb721
37579630a5c4
```



### 导出和导入容器

#### 导出容器

命令: `docker export 容器名称或ID > *.tar`



```shell
[root@localhost mycentos]# docker export centos01 > centos.tar            #导出 centos01 快照到本地文件
[root@localhost mycentos]# ll
total 232984
-rw-r--r--. 1 root root 238572544 Sep 17 13:15 centos.tar
```

#### 导入容器快照

命令:

- 通过本地导入:`cat 容器快照 | docker import - 镜像名字:标签`
- 通过指定 URL 或者某个目录来导入: ` docker import http://example.com/exampleimage.tgz example/imagerepo`



```shell
[root@localhost mycentos]# cat centos.tar | docker import - test/centos:v1		# 导入 centos 快照
sha256:5682537a080204aeee00477b2e315a8794bb1da1bbf7d9e7d2fdb31300087bbe
[root@localhost mycentos]# docker images		# test/centos 导入成功
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
test/centos           v1        5682537a0802   6 seconds ago   231MB
```



### 容器的启动/重启/停止

- 启动已停止运行的容器: `docker start 容器ID或者容器名称`
- 重启容器: `docker restart 容器ID或者容器名称`
- 停止容器: `docker stop 容器ID或者容器名称`



### 查看启动容器的日志

命令: `docker logs 容器ID或者容器名称`





### 查看容器端口映射

命令: `docker port 容器ID或者容器名称`





### 查看Docker底层细节

命令: `docker inspect 容器ID`





### 从容器中拷贝文件到主机

命令: `docker cp 容器ID:/容器内文件路径 主机文件路径`