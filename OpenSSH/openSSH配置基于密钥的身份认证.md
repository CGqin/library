1.  客户端生成公钥、私钥（id_rsa、id_rsa.pub）
```bash
$ ssh-keygen -t rsa -C "xxxx@qq.com"
-t：指定密钥类型，有rsa
-C：注释文字，常用邮箱
-f：指定密钥名（不建议使用）
```
2. 将密钥复制给服务器 
```bash
$ ssh-copy-id -i .ssh/id_rsa.pub root@node2
```