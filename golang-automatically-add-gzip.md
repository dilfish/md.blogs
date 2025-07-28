---
title: "Golang 自动处理 gzip"
date: 2024-08-05T21:06:37+08:00
draft: false
---

## 起因

我在写测试代码时发现，go 的客户端会自动加上 Accept-Encoding: gzip 的头，同时服务端又会自动取消 Content-Encoding:gzip 的头。

```golang
type MockHTTPServerManagerServer struct{}

func (m *MockHTTPServerManagerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// 服务端自动去掉 Content-Encoding gzip 头，哪怕真的写 gzip 数据
	if r.RequestURI == "/gzip" {
		log.Println("server request hreaders are:", r.Header)
		w.Header().Set("Content-Encoding", "gzip")
		wr := gzip.NewWriter(w)
		wr.Write([]byte("gzip information"))
		wr.Close()
		return
	}
	w.Write([]byte("mock server manager server does not support"))
}

// 客户端自动加 Accept-Encoding:gzip 头
func TestGoHTTP(t *testing.T) {
	log.SetFlags(log.LstdFlags | log.Llongfile)
	var m MockHTTPServerManagerServer
	srv := httptest.NewServer(&m)
	log.Println("url is:", srv.URL)
	req, err := http.NewRequest(http.MethodGet, srv.URL+"/gzip", nil)
	if err != nil {
		t.Fatal("new req error")
	}
	log.Println("client request header are:", req.Header)
	resp, err := http.DefaultClient.Do(req)
	log.Println("client resp headers are:", resp.Header)
	defer resp.Body.Close()
	bt, _ := io.ReadAll(resp.Body)
	log.Println("body is:", string(bt))
	log.Println(err)
}
```

上面这段代码会输出：
```bash
Running tool: /usr/local/go/bin/go test -timeout 30s -run ^TestGoHTTP$ github.com/dilfish/httpserver

=== RUN   TestGoHTTP
2024/08/05 20:46:32 /Users/dilfish/go/src/httpserver/m_test.go:32: url is: http://127.0.0.1:60939
2024/08/05 20:46:32 /Users/dilfish/go/src/httpserver/m_test.go:37: client request header are: map[]
2024/08/05 20:46:32 /Users/dilfish/go/src/httpserver/m_test.go:17: server request hreaders are: map[Accept-Encoding:[gzip] User-Agent:[Go-http-client/1.1]]
2024/08/05 20:46:32 /Users/dilfish/go/src/httpserver/m_test.go:39: client resp headers are: map[Date:[Mon, 05 Aug 2024 12:46:32 GMT]]
2024/08/05 20:46:32 /Users/dilfish/go/src/httpserver/m_test.go:42: body is: gzip information
2024/08/05 20:46:32 /Users/dilfish/go/src/httpserver/m_test.go:43: <nil>
--- PASS: TestGoHTTP (0.00s)
PASS
ok      github.com/dilfish/httpserver   0.705s
```


## 发送添加 gzip

客户端发送的 header 为空，go 自动加上了 User-Agent 我们先不管，他还自动加上了 Accept-Encoding。代码路径如下：
```golang
http.DefaultClient.Do(req) ->
net/http.Client.do ->
net/http.Client.send ->
net/http.Client.transport().RoundTrip()
```

这个 RoundTrip 实际是
```golang
net/http.Transport.RoundTrip() ->
net/http.persistConn.roundTrip
```

然后就加上了：

![golang http client](/pics/golang.http.client.add.gzip.png)

## 返回 Content-Encoding 消失

返回时 Content-Encoding 没有了，要么是服务端去掉了，要么是客户端自动解压然后去掉了。

net/http.Transport 在初始化时会调用 dialConn 函数，返回一个持久连接。
里面为每一个 persistConn 分配了一个 go routine 用 readLoop 函数做读操作。
在这里自动解压，并删除了 Content-Encoding 头:

![golang http client del gzip](/pics/golang.client.del.gzip.png)

这里 addedGzip 的值，实际上上面的 requestedGzip，也就是说，自动加的 Accept-Encoding，这里会自动解压。

要不要自动加除了用户没设置，不是 Range，不是 HEAD 之外，还有一个控制条件：

![golang disable compress](/pics/golang.disable.compress.png)

我们来试一下，完美：

![golang disable compress code](/pics/golang.disable.compress.code.png)

