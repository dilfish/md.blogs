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
- 亚马逊作为反薅羊毛大户，注意非预期的费用
- 暂时还用 IPv4，以后可以丢掉 EIP，从服务器角度关闭 IPv4
- nat64.net 这个服务 DNS 速度不行，正常连接500ms，用了之后2000ms
- telegram 不支持 IPv6
- https://gitlab.com/fscarmen/warp 用这个项目找了一个出口，入口先不要了
- 目前 ipv4 和 ipv6 vps 使用 warp 连接在一起