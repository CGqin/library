源码：https://github.com/openssh/openssh-portable/ 
下载：https://github.com/openssh/openssh-portable/tags

**1. 下载源码包**
```shell
git clone https://github.com/openssh/openssh-portable
```
**2. 生成configure**
```shell
autoreconf
```
> 异常: 
```shell
-bash: autoreconf: command not found
```
> 解决:
```shell
yum -y install autoconf
```
> 异常:
```shell
Can't exec "aclocal": No such file or directory at /usr/share/autoconf/Autom4te/FileUtils.pm line 326.
autoreconf: failed to run aclocal: No such file or directory
```
> 解决: 
```shell
yum install -y libtool
```
**3. `./configure`**
```shell
./configure
```
> 如果遇见 `#include <**xxx**.h>`时，什么都不用考虑，直接取执行安装命令：`yum -y install **xxx**-devel`，然后进行重试
1. 编译
```shell
make
```
5. 安装
```
make install
```