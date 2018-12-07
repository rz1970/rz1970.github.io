---
title: OpenSSL vs. OpenSSH
tags: ["openssl", "openssh"]
---

# OpenSSH

OpenSSH是SSH协议的开源版本（SSH：Secure SHell）。使用SSH透过计算机网络实现加密通讯，可以进行远程控制，在计算机之间传送文件等等。SSH传输的数据都进行了加密，比telnet,rcp,ftp,rlogin,rsh等以明文传输密码的工具更安全。

OpenSSH提供了实现SSH协议的很多工具。其中ssh-keygen用于生成，管理和转换用于认证的密钥和证书。

# OpenSSL

OpenSSL 是一个强大的安全套接字层密码库，包括了加密算法，常用密钥和证书管理，SSL协议等功能。

1.密钥生成（genrsa，genpkey，req在证书请求时同时生成密钥, gendh, gendsa）

1.1 openssl genrsa [options] [bits_num]

```bash
#生成RSA密钥对。位长度为2048，保持到rsakey0.pem文件中。
$ openssl genrsa -out rsakey0.pem 2048  

#生成RSA密钥对。使用DES3加密，密钥使用密码保护，位长度为1024
$ openssl genrsa -des3 -out rootca.key -passout pass:123456 1024
```

1.2 openssl genpkey [options]

```bash
#生成RSA密钥，位长度为2048，格式为DER
$ openssl genpkey -algorithm RSA -out rsapriKey.pem -pkeyopt rsa_keygen_bits:2048 -outform DER
```

1.3 openssl req请求时生成新的密钥对

```bash
openssl req -x509  -days 365 -newkey rsa:2048 -keyout private.pem -out public.pem -nodes
```

2.密钥文件管理和转换（rsa, pkey）

2.1 openssl rsa [options] < infile > outfile

```bash
#提取密钥公钥到单独的文件
$ openssl rsa -in rsakey0.pem -pubout -out rsakey0.pub

#转换密钥格式(DER->PEM)
$ openssl rsa -in rsakeypair.der -inform DER -out rsakeypair.pem

#改加密算法，移除密码保护
$ openssl rsa -in rsakeypair.pem -passin pass:123456  -des3 -out rsakeypair1.pem
```

2.2 openssl pkey [options]

2.3 将PEM格式密钥转换成Java JCE 能使用的DER格式密钥的另一种方式

```bash
openssl pkcs8 -topk8 -inform PEM -outform DER -in <rsa_pem.key> -out <pkcs8_der.key> -nocrypt
```

2.4 OpenSSL公钥和OpenSSH公钥格式转换

OpenSSL生成的公钥格式和OpenSSH公钥格式不一致，把OpenSSL生成的公钥用于配置SSH连接，验证会失败。

2.4.1 OpenSSL公钥（PEM）格式为：

```plain
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC7vbqajDw4o6gJy8UtmIbkcpnk
O3Kwc4qsEnSZp/TR+fQi62F79RHWmwKOtFmwteURgLbj7D/WGuNLGOfa/2vse3G2
eHnHl5CB8ruRX9fBl/KgwCVr2JaEuUm66bBQeP5XeBotdR4cvX38uPYivCDdPjJ1
QWPdspTBKcxeFbccDwIDAQAB
-----END PUBLIC KEY-----
```

2.4.2 OpenSSH公钥格式为：

```plain
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQC7vbqajDw4o6gJy8UtmIbkcpnkO3Kwc4qsEnSZp/TR+fQi62F79RHWmwKOtFmwteURgLbj7D/WGuNLGOfa/2vse3G2eHnHl5CB8ruRX9fBl/KgwCVr2JaEuUm66bBQeP5XeBotdR4cvX38uPYivCDdPjJ1QWPdspTBKcxeFbccDw==
```

2.4.3 从私钥重新生成OpenSSH格式公钥

```bash
ssh-keygen -y -f priKey.pem   > sshPubkey.pub
```

2.4.4 将OpenSSL格式公钥转换成OpenSSH格式

```bash
# -m支持 PEM，PKCS8，RFC4716
ssh-keygen -i -m PKCS8 -f sslPubKey.pub 〉 sshPubKey.pub
```

2.4.5 将OpenSSH格式公钥转换成OpenSSL格式公钥

```bash
# -m支持 PEM，PKCS8，RFC4716
$ ssh-keygen -e -m PEM -f sshPubKey.pub >sslPubKey.pub
```

3.使用密钥（rsautl），不支持大文件

```bash
#使用公钥加密
$ openssl rsautl -in test.txt  -out test_enc.txt -inkey rsakeypair.pub -pubin -encrypt
#使用私钥解密
$ openssl rsautl -in test_enc.txt -out text_dec.txt -inkey rsakeypair.pem -decrypt
#使用私钥签名
$ openssl rsautl -in test.txt -out test_sign.txt -inkey rsakeypair.pem -sign
#使用公钥验证签名
$ openssl rsautl -in test_sign.txt -out test_unsign.txt -inkey rsakeypari.pub -pubin -verify
```

4.证书管理

4.1 生成X509格式的自签名证书

```bash
openssl req -x509 -new -days 365 -key rsakey.pem -out cert0.crt
```

会要求输入区别名DN的各项信息（国家，城市，组织，姓名，email等）。

根证书是认证中心机构（Certificate Authority）给自己签发的证书，签发者就是自身，是信任链的起点。里面包含了CA信息、CA公钥、用自身的私钥对这些信息的签名。下载并使用根证书就表示你信任它的来源机构，自然也信任证书以下签发的所有证书。某个证书可以用签发他的证书中的公钥验证，签发他的证书又需要上一层签发证书来验证，直到通过根证书中的公钥验证，那么这个证书就是可信任的。

4.2 生成要求根证书签发子证书的请求文件

```bash
openssl req -new -key rsakey1.pem -out subcertreq.csr
```

会要求输入区别名DN的各项信息（国家，城市，组织，姓名，email等），还需要额外属性：密码 和 可选公司名。

4.3 使用根证书签发子证书

```bash
openssl x509 -req -in subcertreq.csr -CA cert0.crt -CAkey rsakey0.pem -CAcreateserial -days 365 -out subcert.crt
```

4.4 也可创建一个ca的配置文件，通过ca管理子命令来签发子证书（未试验）

```bash
openssl ca -config ca.config -out user.crt -infiles user.csr
```

4.5 将证书和密钥打包为pkcs12格式的库中

```bash
openssl pkcs12 -export -in subcert.crt -inkey rsakey1.pem -out subcert.p12
```

需要输入pkcs12文件密码。

4.6 查看证书内容

```bash
openssl x509 -noout -text -in rootca.crt 
```

4.7 验证证书

```bash
openssl verify -CAfile rootca.crt subcert.crt
```

用rootca.crt的公钥验证subcert.crt中的签名

4.8 从证书中提取公钥

```bash
openssl x509 -in cert.pem  -noout -pubkey > pubkey.pem
```

4.9 提取密钥对

```bash
openssl pkcs12 -in cert.pfx  -nocerts -nodes -out keypari.pem
```

4.10 从PKCS#8提取公钥

```bash
openssl req -in public.pem -noout -pubkey
```

4.11 查看pkcs7签名内容的证书信息

```bash
openssl pkcs7 -in  <pkcs7singedFile>  -inform DER  -print_certs
```

5.Openssl 帮助命令

```bash
openssl command [command_opts] [ command_args ]

openssl [ list-standard-commands | list-message-digest-commands | list-cipher-commands | list-cipher-algorithms | list-message-digest-algorithms | list-public-key-algorithms]

openssl no-XXX [ arbitrary options ]
```