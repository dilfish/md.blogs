---
title: "Ec2 迁移到 IPv6"
date: 2023-11-23T10:14:53+08:00
draft: false
---

鉴于 amazon 在明年2月份要开始对 IPv4 地址收费，我提前做一些迁移，免得到时候措手不及。

- 尝试了几次修改 VPC，subnets 依然无效，干脆删了 ec2 从头建一个
- 在新的 VPC 和 subnet 选择时，不选 IPv4 和 public address，但是 IPv4 CIDR 地址是内部地址可以选(好像也是必选的)
- 修改完 IPv6 的路由就可以登录进去了
- EIP 很方便，可以随时选择，随时去掉
- ipam 要选免费版的，而且要是对应区域的，同时验证了上述 IP 丢掉之后，没有使用 IPv4
- ec2 的源还不支持 IPv6，不过可以很方便修改成 archive.ubuntu.com
- github 这个小垃圾依然不支持 IPv6，去掉 EIP 之后，没法拉代码了
- 在 /etc/resolv.conf 里面改用 cloudflare + google 的 IPv6 DNS
- 我自己的网络手机+宽带支持 IPv6，公司还不支持
- 网站用户随缘
- 亚马逊作为反薅羊毛大户，注意非预期的费用(比如 warp 的 IPv4 是美国 IP，而 ec2 在亚洲，这个流量不免费)
- 暂时还用 IPv4，以后可以丢掉 EIP，从服务器角度关闭 IPv4
- nat64.net 这个服务 DNS 速度不行，正常连接500ms，用了之后2000ms
- telegram 不支持 IPv6
- https://gitlab.com/fscarmen/warp 用这个项目找了一个出口，入口先不要了
- 目前 ipv4 和 ipv6 vps 使用 warp 连接在一起
- [这个链接](https://www.reddit.com/r/aws/comments/17dmdmz/in_regards_to_the_upcoming_ipv4_pricing_update/)指出，免费试用时 IPv4 不收费，所以白折腾了，不过 EIP 比较方便，又给加上了。
- [这个链接](https://www.reddit.com/r/aws/comments/1agm1rf/charges_showing_for_ipv4_address_on_free_tier/)指出，还是收费了，删机了，不折腾了。
- [这个链接](https://erect.eu.org/sh/aws.1g.v4.iptables.cmd)和[这个链接](https://erect.eu.org/sh/aws.1g.v6.ip6tables.cmd) 增加了 iptables 工具，防止亚马逊内部流量消耗
