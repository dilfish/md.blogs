---
title: "DNSPod 降级 IPv6 解析"
date: 2024-12-06T00:48:39+08:00
draft: false
---

# DNSPod 对 AAAA 的降级策略

### 背景
DNS 解析是把域名变成对应 IP 的过程，其中有一个功能叫只能解析，意思是按照运营商和区域进行划分。例如你的服务器在全国各省的每个运营商都有。那你自然希望全国的用户可以就近同运营商访问。

这个策略里面有矛盾的地方，例如区域优先还是运营商优先，好比你的服务器在广东电信和上海移动。现在来一个上海电信的用户，你让他访问哪个？目前大部分的策略是运营商优先，就是说哪怕广东离上海很远，但是因为都是电信，所以他比上海电信跑到上海移动还要快。当然实际情况更复杂，总之这里有这么些个不同优先级的策略。

### 问题
今天公司有人问我这么一个问题，我简化一下。有一个域名，配置了 A 记录的默认区域、电信区域和上海电信区域。配置了 AAAA 记录的默认区域和电信区域。

那一个很自然的想法是说，按照优先级来，你请求 A 记录的时候，如果是上海电信则返回对应结果；如果是其他电信，则返回电信区域结果；如果都不是，那走到默认区域。AAAA 记录也类似，先电信，后默认。

现在有个问题是，上海电信客户端去请求 AAAA 记录时，返回了空的结果，非上海电信的客户端去请求正常。

我也不知道为啥啊，但是这是我之前的公司，而且软件也是我开发的。给我整挺尴尬。

### 结论
去问技术支持，他们也不知道，只是发了一个链接，[关于 IPv6 客户端解析策略的变更说明
](https://docs.dnspod.cn/notices/ipv6-line-adjust/)，然后我看明白了，原来是根据实际情况进行的策略调整。当配置了 A 和 AAAA 记录时，解析器会把两种记录通盘考虑，这也比较合理，因为客户端一般都是双栈并且 IPv6 优先嘛。所以当一个上海电信客户来请求 AAAA 的时候，他不往更通用的区域，即电信区域走。而是查看 A 记录是否配置了上海电信区域，如果配置了，则返回一个空结果，让客户端降级再次请求 A。如果没配置 A，当然还按照区域来继续进行。

但我看上述文档的意思是从2024年5月份开始要改回去到更符合直觉的那种方式了，我不知道为什么要改，也不知道为什么有些情况下还没改。