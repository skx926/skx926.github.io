---
title: 制作服务端推送时使用的.pem证书
date: 2017-06-01 17:45:41
tags: [APNS, 证书]
categories: [iOS]
---

服务端要给移动端发送推送消息就需要用到推送证书，但是服务端通常需要的证书 `.pem` 格式的，并不是我们从Apple的开发者中心下载到的 `.cer` 格式的文件，所以我们需要对其做一个转换。

申请推送证书的过程我就不赘述了，这里假设你已经申请好了开发环境和生产环境的推送证书并下载导入到 Mac 钥匙串当中。

下面我们开始转换，这里我以开发环境为例，生产环境同理：

1. 从钥匙串中分别导出 `Apple Development iOS Push Serives` 证书和私钥为 `cer.p12` 和 `key.p12`（导出的时候会提示输入密码，可以随便设置一个）

2. 使用 openssl 将 `cer.p12` 和 `key.p12` 转换为 `cer.pem` 和 `key.pem`：
```bash
$ openssl pkcs12 -clcerts -nokeys -out cer.pem -in cer.p12
$ openssl pkcs12 -nocerts -out key.pem -in key.p12
```
    转换的时候会提示输入刚才导出的时候设置的密码。转换私钥的时候还会要求你输入一个新的密码，设置的这个新的密码在服务端连接 APNS 服务器时需要使用。

3. 测试生成的文件是否可用：
```bash
// 生产环境请使用 gateway.push.apple.com:2195
$ openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert cer.pem -key key.pem
```

    上面的命令执行之后会有一大堆的信息输出，我们查看最底部的信息：
```bash
SSL-Session:
    Protocol  : TLSv1
    Cipher    : AES256-SHA
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: 13BFA2DC5D6550B06BB351A0A16AC1F9CFE343EEC02F71C45F1F451613D689ED507B4445160C10589ECEFEDD6EBF60CD
    Key-Arg   : None
    Start Time: 1496308341
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
```

    最后一行输出为 `Verify return code: 0 (ok)` 即为测试通过。

4. 合并证书和私钥。服务端在使用时只需要一个文件，所以我们需要将 `cer.pem` 和 `key.pem` 合并成一个 `ck.pem` 文件：
```bash
$ cat cer.pem key.pem > ck.pem
```




