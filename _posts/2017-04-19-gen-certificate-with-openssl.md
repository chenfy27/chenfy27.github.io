---
layout: post
title: openssl制作证书
category: 运维
keywords:
---

出于安全考虑，在网络上传输的数据应该加密，加密可以采用对称或非对称方式，如果采用非对称方式，常见的做法是RSA公私钥加解密。在实际开发工作中，由于找第三方机构签发证书要花钱，一般采用自签名证书。

下面简单介绍通过openssl工具生成私钥和证书的方法，首先对常见的文件扩展名做说明：

- key文件：包含私钥信息。
- csr文件：certificate signing request的缩写，证书签名请求文件，包含公钥信息。
- crt文件：certificate的缩写，证书文件。
- pem文件：用于导入、导出证书时的证书格式，有证书的开头和结尾分隔线。

### 生成CA根证书

流程大致为：生成CA私钥 => 生成CA证书请求 => 自签名得到CA证书

```
$ openssl genrsa -out ca.key 2048
$ openssl req -new -key ca.key -out ca.csr
$ openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```
### 生成用户证书

流程大致为：生成私钥 => 生成用户证书请求 => 用CA根证书签名得到用户证书

```
$ openssl genrsa -out server.key 2048
$ openssl req -new -key server.key -out server.csr
$ openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```

如果最后一步会报错，则做以下操作再尝试：

```
$ mkdir -p ./demoCA/newcerts
$ touch demoCA/index.txt
$ echo "01" >demoCA/serial
```

如果客户端也需要证书，用同样的方法生成，注意不同的用户在申请证书时Common Name尽量设置成不一样的。

有时需要用到pem格式的证书，将crt和key文件合并即可。

```
$ cat server.crt server.key >server.pem
$ cat client.crt client.key >client.pem
```

如果需要显示设置dh参数，可以生成并追加到pem文件中。

```
$ openssl dhparam -out dh.txt 2048
$ cat dh.txt >>server.pem
```
