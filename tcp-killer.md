---
title: "Tcp Killer"
date: 2022-11-18T19:55:20+08:00
draft: false
---

## 缘起

最近发生了这么一件事，我的一个程序 listen 的端口被别人占用了，但是不是 listen 冲突，而是凑巧，他的客户端随机选了一个用到了。

我去找人沟通对方拒绝重启服务，我正在措辞中，听到运维同学 kill 了该进程重启，端口就空出来了。

一方面我就不太善于和别人争执，另一方面，也没有那个“死道友不死贫道”的勇气。这促使我寻找一种更优雅的解决方案。好像也更阴暗一点。唉。

## 方案
实际上掐断一个 TCP 连接很简单，尤其是你可以使用连接的某一端机器时。给通信的双方发3个合适的 RST 包，连接就会断开。

我首先找到了一个叫 [d sniff](https://github.com/ggreer/dsniff) 的项目，不过该项目年久失修，编译不容易，遂放弃。

第二个项目叫 [tcpkill](https://github.com/Colstuwjx/tcpkill)，这个比较合适，稍微修改了一下就可以使用，效果拔群。

## 使用方式

其实也没有什么使用方式，通常情况下，客户端连接服务端，服务端的端口是一个常用的端口，例如 80 或者 443，而客户端是随机的大数，例如 10058。

那你找到 TCP 包之后，使用 source port 或者 destination port 是 10058 来过滤，就可以精准定义到该连接，然后在窗口内向两边发送 RST 包即可。

## 代码

代码很短，我干脆就全部附上了。
```golang
package main

import (
	"flag"
	"fmt"
	"log"
	"net"
	"os"
	"strings"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
)

// google 提供的 gopacket 项目已经做了足够多的事情

// sport 指的是客户端端口
var sport = flag.Int("sp", 0, "source port")

type tcpKill struct {
	Iface        string
	RstSendCount int
	IsPromisc    bool
}

// SendRST 当各个数据准备好以后，可以发送一个 RST 到目标地址端口
func (tk *tcpKill) SendRST(srcMac, dstMac net.HardwareAddr, 
	srcIP, dstIP net.IP,
	srcPort, dstPort layers.TCPPort,
	seq uint32,
	handle *pcap.Handle) error {

	log.Printf("send %v:%v > %v:%v [RST] seq %v", srcIP.String(), srcPort.String(),
		dstIP.String(), dstPort.String(), seq)

    // 伪造 eth 层
	eth := layers.Ethernet{
		SrcMAC:       srcMac,
		DstMAC:       dstMac,
		EthernetType: layers.EthernetTypeIPv4,
	}

    // 伪造 ip 层，ipv6 类似
	iPv4 := layers.IPv4{
		SrcIP:    srcIP,
		DstIP:    dstIP,
		Version:  4,
		TTL:      64,
		Protocol: layers.IPProtocolTCP,
	}

    // 伪造 tcp 层
	tcp := layers.TCP{
		SrcPort: srcPort,
		DstPort: dstPort,
		Seq:     seq,
		RST:     true,
	}

    // 可以看注释，这个函数计算 ip 层的 checksum，依赖上层的 ipv4 或者 ipv6
	err := tcp.SetNetworkLayerForChecksum(&iPv4)
	if err != nil {
		return err
	}

	buffer := gopacket.NewSerializeBuffer()
	options := gopacket.SerializeOptions{
		FixLengths:       true,
		ComputeChecksums: true,
	}
	// 紧凑打包，上面的结构体有很多 buffer，这个函数把各层压缩成合法的二进制包
	err = gopacket.SerializeLayers(buffer, options, &eth, &iPv4, &tcp)
	if err != nil {
		return err
	}

	err = handle.WritePacketData(buffer.Bytes())
	if err != nil {
		return err
	}

	return nil
}

// Run 的目的是找到合适的 TCP 包
func (tk *tcpKill) Run() error {
	fmt.Printf("conn_killer listen on %v\n", tk.Iface)

	var handle *pcap.Handle
	var err error

	// snaplen and timeout hard-code
	// 某个 interface 的全部链接
	handle, err = pcap.OpenLive(tk.Iface, int32(65535), tk.IsPromisc, -1*time.Second)
	if err != nil {
		return err
	}

	defer handle.Close()

    // bpf 相关的内容可以参考 https://www.ntop.org/guides/nprobe/bpf_expressions.html
    // 这里设置 filer 比后面过滤更高效
	if len(flag.Args()) > 0 {
		bpffilter := strings.Join(flag.Args(), " ")
		fmt.Fprintf(os.Stderr, "Using BPF filter %q\n", bpffilter)
		if err = handle.SetBPFFilter(bpffilter); err != nil {
			return fmt.Errorf("set BPF filter error: %v", err)
		}
	}

	packetSource := gopacket.NewPacketSource(
		handle,
		handle.LinkType(),
	)

	for packet := range packetSource.Packets() {
		ethLayer := packet.Layer(layers.LayerTypeEthernet)
		if ethLayer == nil {
			continue
		}
		eth := ethLayer.(*layers.Ethernet)

		ipv4Layer := packet.Layer(layers.LayerTypeIPv4)
		if ipv4Layer == nil {
			continue
		}
		ip := ipv4Layer.(*layers.IPv4)

		tcpLayer := packet.Layer(layers.LayerTypeTCP)
		if tcpLayer == nil {
			continue
		}
		tcp := tcpLayer.(*layers.TCP)

		if tcp.SYN || tcp.FIN || tcp.RST {
			continue
		}

        // 这一行代码是我自己加的！
        // 把其他的链接过滤掉，不然所有链接都挂了
		if tcp.SrcPort != layers.TCPPort(*sport) &&
			tcp.DstPort != layers.TCPPort(*sport) {
			continue
		}

		log.Println("get tcp fin and ack and seq:", tcp.FIN, tcp.ACK, tcp.Seq)
		log.Println("find src dst port:", tcp.SrcPort, tcp.DstPort)
		log.Println("find src dst ip:", ip.SrcIP, ip.DstIP)

		for i := 0; i < tk.RstSendCount; i++ {
			seq := tcp.Ack + uint32(i)*uint32(tcp.Window)
			err = tk.SendRST(eth.DstMAC, eth.SrcMAC,
				ip.DstIP, ip.SrcIP,
				tcp.DstPort, tcp.SrcPort,
				seq, handle)
			if err != nil {
				log.Println("sending rst error:", err)
				return err
			}
		}
	}

	return nil
}

func main() {
	tk := &tcpKill{
		Iface:        "eth0",
		RstSendCount: 3,
		IsPromisc:    true,
	}
	if err := tk.Run(); err != nil {
		log.Fatal(err)
	}
}
```
