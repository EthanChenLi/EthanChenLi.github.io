---
layout: post
title:  "CentOS 7 下搭建自己的 ngrok 内网穿透服务"
date:   2019-10-23 09:56:00 +0800
categories: default
tags: ngrok linux CentOS
comments: 1
---
现在的家庭宽带或者部分公司的宽带都没有直接分配公网 IP ，这个给我们做开发调试时带来的很大的不便，比如我们需要对接微信公众号、支付宝等第三方系统时，第三方系统需要通过异步回调通知我们的服务，但是我们自己电脑上的应用程序都无法对外访问，这样也就通知不了，无法很好的进行联调测试。

在这样的背景下，ngrok 对于我们来说就提供了很大的帮助，他可以让我们内网的应用可以对外访问，无论是 HTTP(s) 服务还是 TCP/SSH 等场景，均可以通过 ngrok 服务内网穿透来达到对外提供访问的目的。

# 搭建步骤
## 第一步：安装 golang 环境
博主的服务器统一使用的 CentOS 7 发行版本，所以通过 ```yum install golang``` 就可以安装好 golang 环境，如果是其他操作系统可以参考官方文档 [How to install golang](https://golang.org/doc/install)

## 第二步：下载 ngrok 源码
目前 ngrok 2.x 版本已经闭源，我们能下载到的最新版本为 [ngrok-1.7.1.zip](https://github.com/inconshreveable/ngrok/archive/1.7.1.zip)，
下载完成之后解压 zip 包，这里我们需要修改 ```src/ngrok/log/logger.go``` 中的源代码，
将 ```log "code.google.com/p/log4go"``` 修改为 ```log "github.com/alecthomas/log4go"```，
否则编译时会报错 ```build fails: 'package code.google.com/p/log4go: unable to detect version control system for code.google.com/ path'```

## 第三步：生成 TLS 证书
例如我们以 4kb.cn 域名为例，我们在生成证书的时候可以指定 ngrok.4kb.cn 这个域名指定 ngrok 服务器地址（需要 DNS 解析到 ngrok 所在服务器的 IP 上），同时我们对外提供 ngrok 域名为 *.4kb.cn （需要泛域名解析 *.4kb.cn 到 ngrok 所在服务器的 IP 上），这里的 ngrok.4kb.cn 和 *.4kb.cn 解析并不冲突。

生成 TLS 证书的命令如下：
```
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=ngrok.4kb.cn" -days 3650 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=ngrok.4kb.cn" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 3650
```

## 第四步：覆盖 ngrok 默认 TLS 证书
```
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt 
cp device.key assets/server/tls/snakeoil.key
```

## 第五步：设置 Golang 编译目录和参数
```
export GOPATH=/go
```
为了可以在 docker 环境下运行，我们还需要运行下面的命令，否则会报错 ```Unable to run a go program inside docker /bin/sh: ./ngrokd: not found```
```
export CGO_ENABLED=0
```

## 第六步：编译 ngrok 服务端和客户端
Linux 32 位系统 ngrok 编译命令
```
export GOOS=linux GOARCH=386
make release-server release-client
```
Linux 64 位系统 ngrok 编译命令
```
export GOOS=linux GOARCH=amd64
make release-server release-client
```
Windows 32 位系统 ngrok 编译命令
```
export GOOS=windows GOARCH=386
make release-server release-client
```
Windows 64 位系统 ngrok 编译命令
```
export GOOS=windows GOARCH=amd64
make release-server release-client
```
macOS 32 位系统 ngrok 编译命令
```
export GOOS=darwin GOARCH=386
make release-server release-client
```
macOS 64 位系统 ngrok 编译命令
```
export GOOS=darwin GOARCH=amd64
make release-server release-client
```
其他操作系统修改 GOOS 和 GOARCH 为对应的即可：[Go (Golang) GOOS and GOARCH](https://gist.github.com/asukakenji/f15ba7e588ac42795f421b48b8aede63)

## 第七步：运行 ngrokd 服务端
编译完成之后可以在 bin 目录下看到 ngrokd（服务端）和 ngrok（客户端），运行服务端时需要将 assets/server/tls/snakeoil.crt 和 assets/server/tls/snakeoil.key
复制到同级目录，启动的命令参考如下：
```
./ngrokd -tlsKey=snakeoil.key -tlsCrt=snakeoil.crt -domain="4kb.cn" -httpAddr=":80" -httpsAddr=":443" -tunnelAddr=":4443"
```
如果使用默认 80、443、4443 端口，则可以使用以下命令运行：
```
./ngrokd -tlsKey=snakeoil.key -tlsCrt=snakeoil.crt -domain="4kb.cn"
```

## 第八步：运行 ngrok 客户端
运行之前我们需要编写一个 ```ngrok.cfg``` 文件，内容如下
```
server_addr: "ngrok.4kb.cn:4443"
trust_host_root_certs: false
```
这里的 server_addr 需要和第三步生成 TLS 时保持一致，否则服务端会报 ```tls: bad certificate``` 证书错误，
编辑完成之后使用 ngrok 客户端运行即可
```
./ngrok -config=ngrok.cfg 8080
```
指定域名运行命令
```
./ngrok -config=ngrok.cfg -subdomain=xxx 8080
```
映射 SSH 22 端口，使用 tcp 协议
```
./ngrok -config=ngrok.cfg -proto=tcp 22
```
# 总结
我们搭建完成 ngrok 服务端后，任意能连接到服务器的应用程序均可以通过 ngrok 内网穿透，不仅 Java、PHP 等开发的网站可以通过 ngrok 映射对外提供域名访问，也可以让 SSH 的 22 端口映射之后，我们连接无公网 IP 的服务器（比如在家里面搭建一个私有服务器），甚至还可以将 MySQL 的 3306 端口映射之后，在任意地点远程连接内网的数据库。这给我们带来来极大的方便，也不需要购买第三方服务（例如花生壳）。

# 参考文献
 - [编译自己的ngrok服务](http://www.ifengse.com/post/ngrok-install/)
 - [Go (Golang) GOOS and GOARCH](https://gist.github.com/asukakenji/f15ba7e588ac42795f421b48b8aede63)
 - [Unable to run a go program inside docker /bin/sh: < program>: not found](https://stackoverflow.com/questions/48124388/unable-to-run-a-go-program-inside-docker-bin-sh-program-not-found)

# 版权
 > 版权声明：自由转载-非商用-非衍生-保持署名（创意共享3.0许可证）

原创作者 10086@xiaoi.me 发表于阿里云·云栖社区：[https://yq.aliyun.com/users/y4epujtm5wye6](https://yq.aliyun.com/users/y4epujtm5wye6)

扫码关注我，在线与我沟通、咨询
![image](/assets/res/qrcode.png)

**转载请保留原文链接以及版权信息**