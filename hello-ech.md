---
title: "Ech介绍"
date: 2025-07-02T14:52:34+08:00
draft: false
---

## ECH 介绍

#### 缘起
前些年就听说了 cloudflare 在推动 ech 协议，这个协议简单来说就是 https 协议在最开始握手阶段是未加密的，而这个阶段中，客户端需要向服务端提供 server name，即你访问网站的名字是未加密的。

cloudflare 推出的 ech 协议结合 DNS 的 HTTP 类型，以及 DNS over HTTPS 这几个产品组合，将访问的整个流程都加密了。

我很早都知道这个事情，但是也没动心思学习，前几天抽奖抽到了一个 alice networks 的免费的 v6 vps，可以提供香港，台湾和新加坡的家庭宽带 IP 地址，于是可以访问 dcard 网站了。但是我在做分流的时候一直有问题，根据域名来路由到家庭宽带 IP 一直不成功。将日志打开到 debug 时看到了，访问 dcard.tw 时，实际访问的是 cloudflare-ech.com，怪不得域名匹配不上。看到这个域名我就知道是 chrome 和 cloudflare 联合搞的鬼。

#### 协议
对于 ECH 协议来说，访问流程变成了从 DNS 协议中获取 IP 地址和加密公钥，然后直接加密访问服务器，这时候，中间的 box 就不知道域名的信息了。

例如 dcard.tw 的 HTTPS 记录长这样：

```console
root@arm:~ # dig dcard.tw https +short
1 . alpn="h3,h2" ipv4hint=104.19.247.7,104.19.248.7 ech=AEX+DQBB5QAgACDjock9BDJAAfTxtjI3o215PxkD2hUKHM5YIiEBiKfTEQAEAAEAAQASY2xvdWRmbGFyZS1lY2guY29tAAA= ipv6hint=2606:4700::6813:f707,2606:4700::6813:f807
```

浏览器拿到这个信息之后，就可以用 AEX... 这个信息去加密通信了。

可是为什么能看见访问的是 cloudflare-ech.com 呢？我问了下 AI，答案是：

`
cloudflare-ech.com as a Placeholder: When you visit an ECH-enabled website hosted by Cloudflare, Chrome and other browsers supporting ECH will use cloudflare-ech.com as a generic, unencrypted Server Name Indication (SNI). This outwardly indicates that you're connecting to an encrypted website on Cloudflare, but it doesn't reveal the actual website you're visiting.
`

就是说，原来协议的地方空着也不好看，就用 cloudflare-ech.com 当成占位符，实际的域名确实加密了，代理协议或者抓包程序都看不见。

#### 阅读

[Quick set up guide for Encrypted Client Hello (ECH)](https://guardianproject.info/2023/11/10/quick-set-up-guide-for-encrypted-client-hello-ech/)。

[Good-bye ESNI, hello ECH!](https://blog.cloudflare.com/encrypted-client-hello/)

#### golang 的支持

```golang
package main

import (
	"crypto/tls"
	"encoding/base64"
	"fmt"
	"io"
	"log"
	"net/http"
)

func GetEchClientConfigListBytes() ([]byte, error) {
	// dig dcard.tw https
	// :wq
	// alpn="h3,h2"
	// ipv4hint=104.19.247.7,104.19.248.7
	// ech=AEX+DQBBkAAgACD/3ipAGDU5MWP3tzwOQJNStltFjam0kTteAnlR7IvceAAEAAEAAQASY2xvdWRmbGFyZS1lY2guY29tAAA=
	//ipv6hint=2606:4700::6813:f707,2606:4700::6813:f807
	ech := "AEX+DQBBkAAgACD/3ipAGDU5MWP3tzwOQJNStltFjam0kTteAnlR7IvceAAEAAEAAQASY2xvdWRmbGFyZS1lY2guY29tAAA="
	return base64.StdEncoding.DecodeString(ech)
}

func main() {
	echConfigList, err := GetEchClientConfigListBytes()
	if err != nil {
		log.Println("get ech client config list bytes error:", err)
		return
	}

	cli := &http.Client{Transport: &http.Transport{
		TLSClientConfig: &tls.Config{
			EncryptedClientHelloConfigList: echConfigList,
		},
	}}

	resp, err := cli.Get("https://dcard.tw")
	if err != nil {
		fmt.Printf("making request error: %v\n", err)
		return
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("reading response body error: %v\n", err)
		return
	}

	fmt.Printf("status code: %s\n", resp.Status)
	fmt.Println("body len: ", len(body))

	if resp.TLS != nil {
		if resp.TLS.ECHAccepted {
			fmt.Println("ECH was accepted by the server with sni:", resp.TLS.ServerName)
		} else {
			fmt.Println("ECH was not accepted by the server")
		}
	}
}
```
