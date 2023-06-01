---
title: "golang 在 HTTP 中使用 scope link 的 IPv6 地址"
date: 2020-10-15T12:18:42+08:00
draft: false
---

例如 server 端接口名称为 bond0

server 端命令：
```sh
./http -s=true -i=:: -p=19999
```

client 端命令：
```sh
./http -i="fe80::468a:5bff:fe3d:c6d2%25bond0" -p=19999
```

其中 %25 是 '%' 的 HTTP 转码，代码如下：

```golang
package main

import (
	"flag"
	"io/ioutil"
	"log"
	"net/http"
)

var FlagServer = flag.Bool("s", false, "if run as a server")
var FlagIP = flag.String("i", "::1", "ip address")
var FlagPort = flag.String("p", "80", "port")

func Handler(w http.ResponseWriter, r *http.Request) {
	log.Println("w and r are", w, r)
	w.Write([]byte("ok"))
}

func RunServer(ip, port string) {
	http.HandleFunc("/", Handler)
	err := http.ListenAndServe("["+ip+"]:"+port, nil)
	if err != nil {
		log.Println("run server error:", err)
	}
}

func RunClient(ip, port string) {
	resp, err := http.Get("http://[" + ip + "]:" + port)
	if err != nil {
		log.Println("get error:", err)
		return
	}
	log.Println("resp is", resp, err)
	defer resp.Body.Close()
	bt, err := ioutil.ReadAll(resp.Body)
	log.Println("body is", string(bt), err)
}

func main() {
	flag.Parse()
	log.SetFlags(log.LstdFlags | log.Lshortfile)
	if *FlagServer == true {
		log.Println("we run as server", *FlagIP, *FlagPort)
		RunServer(*FlagIP, *FlagPort)
		return
	}
	log.Println("we run as client:", *FlagIP, *FlagPort)
	RunClient(*FlagIP, *FlagPort)
	return
}

```
