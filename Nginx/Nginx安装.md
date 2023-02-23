1. 去官网http://nginx.org下载对应的nginx包，推荐使用稳定版本

2. 上传ngxin到inux系统

3. 安装依赖环境
	1. 安装gcc环境 
		-  `yum install gcc-c++`
	2. 安装PCRE库，用于解析正则表达式
		- `yum install-y pcre pcre-devel`
	3. zlib压缩和解压依赖
		- `yum install -y zlib zlib-devel`
	4. SSL安全的加密的套接字协议层，用于HTTP安全传输，也就是https
		- `yum install -y openssl openssl-devel`

4. 解压, 需要注意,解压后得到的是源码,源码编译后才能安装
```
tar -zcvf nginx-1.16.1.tar.gz
```

5. 编译之前，先创建nginx临时目录，如果不创建，在启动nginx的过程中会报错
```
mkdir -p /var/temp/nginx
```

6. 在nginx目录，输入如下命令进行配置，目的是为了创建makefile文件
```
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

- 配置命令:
	