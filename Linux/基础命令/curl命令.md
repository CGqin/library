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
