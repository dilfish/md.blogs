---
title: "老司机邮箱翻车记"
date: 2023-11-27T20:43:01+08:00
draft: false
---

### 缘起

早上打开邮箱，看到一坨退信，相关信息我附在最后。给我吓了一跳，以为是邮箱被黑了。

### 处理

我的 gmail 邮箱使用了唯一的密码，并且开启了二次验证，陌生的环境登录一般会要求手机验证等，按理说不应该被黑。我又仔细阅读了一下退信的内容。发现是以我的邮箱为名，发送到其他未知邮箱的信被退回来了。邮箱协议我也懒得研究，但我记得是可以冒充发送人的。

打开 gmail 主界面，点击右下方详细信息，可以查看最近的几次登录记录。发现没有异常。所以可以肯定不是被黑了。

### 原因

搜索了一圈发现有人遇到过类似问题，据猜测是回复了某个邮件，导致别人确认了这个邮件可用，然后发垃圾邮件。而邮箱协议的设计问题，导致你无法阻止这样的情况，以后会有源源不断的退信。呵呵。

我找了找最近发送的邮件，发现有一个地址是三天前回复的，unsubscribe@academia-noreply.com，你收到了一封垃圾邮件，然后你点垃圾邮件，如果里面有个退订链接，gmail 会贴心地问你是不是要退订，如果你不小心点了是，那么对方就知道你这个邮箱是真实的邮箱。真聪明。

这个域名使用了亚马逊的解析服务器，但我也懒得深究了。

### 结论

如果有人根据关键字搜索到这里，请放心，你的邮箱没有被黑，但是在 gmail 你应该直接删除垃圾邮件，而不是点垃圾邮件按钮。

### 邮件信息

- 钓鱼邮件发起方：academia-noreply.com
- 发送地址：CenterUSA365@outlook.com, datacenterus@126.com 等等
- 标题：Report the user!
- 内容：Block Him
- gmail 退信服务：Mail Delivery Subsystem <mailer-daemon@googlemail.com>
- 退信错误信息：找不到地址，由于系统找不到电子邮件地址 CenterUSA365@outlook.com，或该地址无法接收邮件，因此无法递送您的邮件。了解详情
- 退信错误信息2：响应如下：550 5.2.1 The email account that you tried to reach is inactive. Learn more at <https://support.google.com/mail/?p=DisabledUser> q5-20020a819905000000b005cea6de99ccsor609363ywg.0 - gsmtp
- 退信错误信息3：邮件未递送，远程服务器配置错误，因此系统无法将您的邮件地送至 news@mass.eu.com。请参阅下方的技术详细信息了解情况。
- 退信错误信息4：以下为远程服务器的响应：554 5.7.1 news@mass.eu.com: Relay access denied
- 退信错误信息5：发生临时性问题，无法将您的邮件递送至 news@top.gr.com。在接下来的 47 小时内，Gmail 会重新尝试递送。如果邮件始终无法递送，系统会通知您。
- 退信错误信息6：451 Temporary local problem - please try later
