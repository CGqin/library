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