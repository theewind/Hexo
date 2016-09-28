---
title: iOS证书之CSR文件
date: 2016-09-28 16:28:47
tags: iOS
---
在iOS开发的过程中， 必然需要制作证书，开发者证书，发布证书，但是在制作他们的第一步都是先生成一个CSR文件
![证书机构申请](http://o981ibvmi.bkt.clouddn.com/Screen%20Shot%202016-09-28%20at%204.35.22%20PM.png)

那么到底CSR是什么东西呢？可以参考下面官方介绍

> [What is a CSR (Certificate Signing Request)?](https://www.sslshopper.com/what-is-a-csr-certificate-signing-request.html)

简单来说，CSR就是Certificate Signing Request的缩写，是从证书颁发机构生成证书时，需要的一串加密的文本信息，也是从证书颁发机构获取的，用文本工具打开，可以看到他是一串

<!--more-->

-----BEGIN CERTIFICATE REQUEST-----
MIICkDCCAXgCAQAwSzEkMCIGCSqGSIb3DQEJARYVc2FuZmVuZ2ZseWluZ0AxMjYu
Y29tMRYwFAYDVQQDDA1zYW5mZW5nZmx5aW5nMQswCQYDVQQGEwJDTjCCASIwDQYJ
KoZIhvcNAQEBBQADggEPADCCAQoCggEBALeOs2Pd65KYGUFXio02QrUyFUSImQ2L
NEFNpEga6yWPEsXG+yYAsd4dx5DOkWx4Iiybsdg0v8uB1b9d0GfjIzRiMyhTM24V
cgODmijKT2DnxvagDZH9+ikErvlvcb4KtLKNoRYOJ8T0Oc80ibamQULuTRyrixlQ
u8G5gIul3MxQpv+ZmJO3DcamqnfKkrQNVw3D9ALUuk7o8W1V8vgea/LzsVMm5JzC
mxq1zOnnxpV/7KHdVtrK+rRMOqr2tdTnKtgK9yNVqCjWDgJmmvVtJ1uvmmDWevjg
S985hH2niif9zlim4c3tjceGkAaYpuGM+24kQnYvYt6JvNh9XTkeVh0CAwEAAaAA
MA0GCSqGSIb3DQEBCwUAA4IBAQAId4jHlTMryJHTDhL/cgSb5rRIYJC7wJ2fvGfu
Jwlr6HZplZBZGDr271lYcEJ56VDyXzLEx1ea/4f+I3IFowS6AlcvY2YrJwY7KET+
R98GDsprBdgg2RpSxpJPe4rF1rWDjUdPJUCFi+AczQ0aA6ckIzYDhlQpXieTXKPd
nrG0cV6vsJBMQ6oAo8eZxDl7HV29sK/RJRa1k0q3GpzyhpKmPTfiA0ig4H4zHE3n
TUtwa9EOBuhxcGItZRypdZTOX1Rrp5XFyxdtUgx9eKsg0sys/SGgrTtU+YAiorr1
TH4jOKnTy1lATjIn3MQhoBQhPl6WxmtVcWx2+pLklYEW+M+Y
-----END CERTIFICATE REQUEST-----

这样的内容，它里面实际包含了一些你的个人信息，比如email，国家等，但是最重要的是，他包含了你签名要用的公钥信息，而对应的私钥则存储在你的mac上，打开KeyChain Asscess --login --Keys 发现里面多了两组信息，就是公钥私钥

采用如下命令，可以将CSR进行解密：

	openssl req -in CertificateSigningRequest.certSigningRequest -noout -text
	
可以得到如下内容：

```
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: emailAddress=sanfengflying@126.com, CN=sanfengflying, C=CN
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (2048 bit)
                Modulus (2048 bit):
                    00:b7:8e:b3:63:dd:eb:92:98:19:41:57:8a:8d:36:
                    42:b5:32:15:44:88:99:0d:8b:34:41:4d:a4:48:1a:
                    eb:25:8f:12:c5:c6:fb:26:00:b1:de:1d:c7:90:ce:
                    91:6c:78:22:2c:9b:b1:d8:34:bf:cb:81:d5:bf:5d:
                    d0:67:e3:23:34:62:33:28:53:33:6e:15:72:03:83:
                    9a:28:ca:4f:60:e7:c6:f6:a0:0d:91:fd:fa:29:04:
                    ae:f9:6f:71:be:0a:b4:b2:8d:a1:16:0e:27:c4:f4:
                    39:cf:34:89:b6:a6:41:42:ee:4d:1c:ab:8b:19:50:
                    bb:c1:b9:80:8b:a5:dc:cc:50:a6:ff:99:98:93:b7:
                    0d:c6:a6:aa:77:ca:92:b4:0d:57:0d:c3:f4:02:d4:
                    ba:4e:e8:f1:6d:55:f2:f8:1e:6b:f2:f3:b1:53:26:
                    e4:9c:c2:9b:1a:b5:cc:e9:e7:c6:95:7f:ec:a1:dd:
                    56:da:ca:fa:b4:4c:3a:aa:f6:b5:d4:e7:2a:d8:0a:
                    f7:23:55:a8:28:d6:0e:02:66:9a:f5:6d:27:5b:af:
                    9a:60:d6:7a:f8:e0:4b:df:39:84:7d:a7:8a:27:fd:
                    ce:58:a6:e1:cd:ed:8d:c7:86:90:06:98:a6:e1:8c:
                    fb:6e:24:42:76:2f:62:de:89:bc:d8:7d:5d:39:1e:
                    56:1d
                Exponent: 65537 (0x10001)
        Attributes:
            a0:00
    Signature Algorithm: sha256WithRSAEncryption
        08:77:88:c7:95:33:2b:c8:91:d3:0e:12:ff:72:04:9b:e6:b4:
        48:60:90:bb:c0:9d:9f:bc:67:ee:27:09:6b:e8:76:69:95:90:
        59:18:3a:f6:ef:59:58:70:42:79:e9:50:f2:5f:32:c4:c7:57:
        9a:ff:87:fe:23:72:05:a3:04:ba:02:57:2f:63:66:2b:27:06:
        3b:28:44:fe:47:df:06:0e:ca:6b:05:d8:20:d9:1a:52:c6:92:
        4f:7b:8a:c5:d6:b5:83:8d:47:4f:25:40:85:8b:e0:1c:cd:0d:
        1a:03:a7:24:23:36:03:86:54:29:5e:27:93:5c:a3:dd:9e:b1:
        b4:71:5e:af:b0:90:4c:43:aa:00:a3:c7:99:c4:39:7b:1d:5d:
        bd:b0:af:d1:25:16:b5:93:4a:b7:1a:9c:f2:86:92:a6:3d:37:
        e2:03:48:a0:e0:7e:33:1c:4d:e7:4d:4b:70:6b:d1:0e:06:e8:
        71:70:62:2d:65:1c:a9:75:94:ce:5f:54:6b:a7:95:c5:cb:17:
        6d:52:0c:7d:78:ab:20:d2:cc:ac:fd:21:a0:ad:3b:54:f9:80:
        22:a2:ba:f5:4c:7e:23:38:a9:d3:cb:59:40:4e:32:27:dc:c4:
        21:a0:14:21:3e:5e:96:c6:6b:55:71:6c:76:fa:92:e4:95:81:
        16:f8:cf:98
```
可以看到他的全部内容，公钥以及其他内容

---
当然我们也可以使用如下命令

	openssl req -new -keyout server.key -out server.csr

生成CSR和私钥key，有了这两个之后，就可以通过CA机构生成证书。

---

ps：
证书相关的有太多的格式，.pem, .cer, .key, .p12 等等，为什么这么多呢？
首先要知道，X.509是常见通用的证书格式。所有的证书都符合为Public Key Infrastructure (PKI) 制定的 ITU-T X509 国际标准。

.cer/.crt是用于存放证书，它是2进制形式存放的，不含私钥。

.pem跟crt/cer的区别是它以Ascii来表示。

pfx/p12用于存放个人证书/私钥，他通常包含保护密码，2进制方式

der,cer文件一般是二进制格式的，只放证书，不含私钥

pem文件一般是文本格式的，可以放证书或者私钥，或者两者都有

key文件，pem如果只含私钥的话，一般用.key扩展名，而且可以有密码保护

p12文件是二进制格式，同时含私钥和证书，通常有保护密码

> [http://blog.chinaunix.net/uid-26575352-id-3073802.html](http://blog.chinaunix.net/uid-26575352-id-3073802.html)



