---
title: "lsof 显示 IP 类型为 IPv6"
date: 2024-05-22T20:58:18+08:00
draft: false
---

几天前看到一个问题说，为什么 lsof 显示的 IP 类型是 IPv6 的，例如：

```bash
root@gcp:~ # lsof -i:9999
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
a.out   81466 root    3u  IPv6 805392      0t0  TCP *:9999 (LISTEN)
```

但是这个接口却可以被 IPv4 客户端连接。

go 语言里面有一个相关的函数叫 net.ListenTCP，里面有个函数选择协议：
```golang
// favoriteAddrFamily returns the appropriate address family for the
// given network, laddr, raddr and mode.
//
// If mode indicates "listen" and laddr is a wildcard, we assume that
// the user wants to make a passive-open connection with a wildcard
// address family, both AF_INET and AF_INET6, and a wildcard address
// like the following:
//
//   - A listen for a wildcard communication domain, "tcp" or
//     "udp", with a wildcard address: If the platform supports
//     both IPv6 and IPv4-mapped IPv6 communication capabilities,
//     or does not support IPv4, we use a dual stack, AF_INET6 and
//     IPV6_V6ONLY=0, wildcard address listen. The dual stack
//     wildcard address listen may fall back to an IPv6-only,
//     AF_INET6 and IPV6_V6ONLY=1, wildcard address listen.
//     Otherwise we prefer an IPv4-only, AF_INET, wildcard address
//     listen.
//
//   - A listen for a wildcard communication domain, "tcp" or
//     "udp", with an IPv4 wildcard address: same as above.
//
//   - A listen for a wildcard communication domain, "tcp" or
//     "udp", with an IPv6 wildcard address: same as above.
//
//   - A listen for an IPv4 communication domain, "tcp4" or "udp4",
//     with an IPv4 wildcard address: We use an IPv4-only, AF_INET,
//     wildcard address listen.
//
//   - A listen for an IPv6 communication domain, "tcp6" or "udp6",
//     with an IPv6 wildcard address: We use an IPv6-only, AF_INET6
//     and IPV6_V6ONLY=1, wildcard address listen.
//
// Otherwise guess: If the addresses are IPv4 then returns AF_INET,
// or else returns AF_INET6. It also returns a boolean value what
// designates IPV6_V6ONLY option.
//
// Note that the latest DragonFly BSD and OpenBSD kernels allow
// neither "net.inet6.ip6.v6only=1" change nor IPPROTO_IPV6 level
// IPV6_V6ONLY socket option setting.
func favoriteAddrFamily(network string, 
laddr, raddr sockaddr, 
mode string) (family int, ipv6only bool) {
```

我写了一个 c 语言程序验证：
```c
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <arpa/inet.h>
#include <stdio.h>

void handle_client(int c)
{
    char buf[8192];
    char *lastpos;
    int size;

    while (1) {
        size = recv(c, buf, 8192, 0);
        if (size == 0) {
            break;
        }
        lastpos = strchr(buf, '\n');
	printf("received: %d\n", size);
	sleep(1000);
        send(c, buf, lastpos+1-buf, 0);
    }
}

int main()
{
    int s, c;
    int reuseaddr = 1;
    struct sockaddr_in6 addr;
    struct sockaddr_in6 client;
    int pid;
	socklen_t client_addr_len;

	client_addr_len = sizeof(client);


    s = socket(AF_INET6, SOCK_STREAM, 0);

    addr.sin6_family = AF_INET6;
    addr.sin6_port = htons(9999);
    addr.sin6_addr = in6addr_any;

    bind(s, (struct sockaddr *)&addr, sizeof(addr));
    listen(s, 5);

    while (1) {
        c = accept(s, (struct sockaddr*)&client, &client_addr_len);
	printf("client is: %d, %d\n", client.sin6_family, client.sin6_port);
	int i = 0;
	for (i = 0;i < 16;i ++) {
		printf("0x%x:", client.sin6_addr.s6_addr[i]);
	}
	printf("\n");
        pid = fork();
        if (pid == -1) {
            exit(1);
        } else if (pid == 0) {
            close(s);
            handle_client(c);
            close(c);
            return 0;
        } else {
            close(c);
            waitpid(pid, NULL, 0);
        }
    }
}
```

因此这是一个 linux kernel 支持的功能，细节上我还没研究。
