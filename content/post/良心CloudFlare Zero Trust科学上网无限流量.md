---
title: 良心CloudFlare Zero Trust科学上网无限流量
date: 2023-10-04T19:44:08+08:00
draft: false
tags: ["CloudFlare", "科学上网", "WARP"]
categories: ["笔记"]
author: dcq
contentCopyright: <a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>
---
## CloudFlare Zero Trust介绍

Cloudflare Zero Trust是Cloudflare推出的一项安全框架，旨在帮助组织实现"零信任"（Zero Trust）的网络安全策略。零信任是一种安全理念，它假定任何用户或设备都不可信，无论其是否位于企业网络内部。

Cloudflare Zero Trust通过以下方式提供安全保护：

1. 访问控制：基于用户身份、设备状态和上下文信息，Cloudflare Zero Trust提供了细粒度的访问控制。它使用多种身份验证方法和安全策略来确保只有经过授权的用户和设备可以访问企业资源。
2. 安全连接：通过使用Cloudflare的网络，Zero Trust提供了安全的连接机制。它通过加密和认证来保护用户和企业资源之间的通信，防止数据泄露和中间人攻击。
3. 网络分段：Zero Trust通过网络分段将企业网络划分为多个安全区域。这样可以限制不同用户和设备之间的访问，减少横向扩展攻击的风险。
4. 威胁情报和分析：Cloudflare Zero Trust利用实时的威胁情报和分析来检测和阻止潜在的安全威胁。它可以识别异常行为、恶意流量，并采取相应的安全措施。
![](https://gitee.com/spring3th/pubpic/raw/master/img/cf.avif)
## 准备就绪

1. 电脑/手机
2. 科学上网环境

## 下载并安装客户户端

1. 打开https://1.1.1.1
2. 根据平台自行下载客户端

## 注册流程

要注册Cloudflare，您可以按照以下步骤进行操作：

1. 访问Cloudflare官方网站（[](https://www.cloudflare.com/%EF%BC%89%E3%80%82)[https://www.cloudflare.com/）。](https://www.cloudflare.com/%EF%BC%89%E3%80%82)
2. 点击网页右上角的"Sign Up"（注册）按钮。
3. 在注册页面上，您可以选择适合您的计划类型。Cloudflare提供免费和付费的计划选项。对于个人网站或小型网站，免费计划通常已经足够。
4. 输入您的电子邮件地址和设置密码。确保提供一个有效的电子邮件地址，因为您将收到与注册和设置Cloudflare账户相关的通知和信息。
5. 点击"Create Account"（创建账户）按钮。
6. 随后，您将被要求添加您的网站。输入您的网站域名，然后点击"Add Site"（添加网站）按钮。
7. Cloudflare将扫描您的域名并显示相关的DNS记录。请确保这些DNS记录正确，然后点击"Next"（下一步）按钮。
8. 在接下来的页面上，您可以选择Cloudflare的额外功能和服务，例如CDN、防火墙、SSL等。根据您的需求选择适当的选项，然后点击"Next"（下一步）按钮。
9. 在下一个页面上，Cloudflare将为您提供新的DNS记录。您需要将这些DNS记录更新到您的域名注册商或DNS提供商的设置中。具体步骤可能因您使用的注册商或提供商而有所不同。
10. 完成DNS记录的更新后，返回Cloudflare注册界面，点击"Done, check nameservers"（完成，检查域名服务器）按钮。
11. Cloudflare将开始验证您的DNS设置，并在验证成功后将您的流量路由通过他们的网络。这个过程可能需要一些时间，通常在几分钟内完成。

## 启动客户端并连接节点

为什么我会在这里说呢？因为我们如果正常启动CloudFlare WARP时候，由于注册服务域名被GFW阻断，因此就有可能遇到无法注册的问题。

1. 第一次打开客户端之前，需要打开代理软件的真全局模式（例如：Clash的TUN模式），如在手机上则打开任意代理软件即可
2. 进入设置选项，如Device ID出现非0000的字符串时，即为注册成功
3. 选择团队
4. 选择域名后缀，注意：qq邮箱收不到邮件，改gmail就正常


![](https://gitee.com/spring3th/pubpic/raw/master/img/cv2.avif)
## 测试

把原来的ShadowSocks关闭，启动WARP，速度很快，可以实现全局科学上网，解锁WARP, 刚好VPN要到期了。