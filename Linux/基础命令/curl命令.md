# -i
`-i` 参数打印出服务器回应的 HTTP标头和网站源码.
```
# curl -i 192.168.1.99
HTTP/1.1 200 OK
Date: Wed, 08 Feb 2023 20:55:37 GMT
Server: Apache/2.4.37 (centos)
Last-Modified: Wed, 08 Feb 2023 14:41:23 GMT
ETag: "c-5f431421dc151"
Accept-Ranges: bytes
Content-Length: 12
Content-Type: text/html; charset=UTF-8

hello world
```
# -I
`-I` 参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。
```
# curl -I 192.168.1.99
HTTP/1.1 200 OK
Date: Wed, 08 Feb 2023 20:56:46 GMT
Server: Apache/2.4.37 (centos)
Last-Modified: Wed, 08 Feb 2023 14:41:23 GMT
ETag: "c-5f431421dc151"
Accept-Ranges: bytes
Content-Length: 12
Content-Type: text/html; charset=UTF-8
```
# -v
`-v` 参数输出通信的整个过程，用于调试。
```
# curl -v 192.168.1.99
* Rebuilt URL to: 192.168.1.99/
*   Trying 192.168.1.99...
* TCP_NODELAY set
* Connected to 192.168.1.99 (192.168.1.99) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.1.99
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Wed, 08 Feb 2023 21:05:34 GMT
< Server: Apache/2.4.37 (centos)
< Last-Modified: Wed, 08 Feb 2023 14:41:23 GMT
< ETag: "c-5f431421dc151"
< Accept-Ranges: bytes
< Content-Length: 12
< Content-Type: text/html; charset=UTF-8
<
hello world
* Connection #0 to host 192.168.1.99 left intact
```