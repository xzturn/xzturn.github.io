---
layout: post
title:  "关于账号安全性"
date:   2015-09-06 07:51:59
categories: security
---

之前申请了几个[微软][microsoft]的 [outlook][outlook] 邮箱，专门用来注册和登录 [VPS][vps], 域名, 存储云服务 和 微信公众号及开发者账号。今天登录 [outlook.com][outlook] 邮箱的时候，突然发现微软给了一个安全风险提示，疑似邮箱被人攻破：
![Security-Risk](http://7xljs8.com1.z0.glb.clouddn.com/outlook-risk.png "Outlook Securiety Risk")
当时惊出一身冷汗：邮箱要是被攻破了，主机不很快成肉机了嘛，赶紧换密码……

这里还是要赞一下 [Microsoft][microsoft]，它不是让你直接改密码，而是给出了非常贴心的安全保护措施：身份验证器，提供 **密码 + 验证码** 的两级认证登录方式[^1]。

而且它推荐的身份验证器竟然是 [Google][google] 的 [Authenticator][auth]（考虑到 [google][google] 与 [microsoft][microsoft] 的关系，这至少在鲍尔默时代是不可想象的）。

闲话少说，按照提示把那几个 [outlook][outlook] 邮箱都加到身份验证器中，心里踏实了许多：

![Auth-Outlook](http://7xljs8.com1.z0.glb.clouddn.com/auth-outlook.png "Outlook Securiety Risk")

很自然地，又想起来，是不是 [Linode][linode]（笔者的 [VPS][vps] 用的 [Linode][linode]） 也支持这种两步验证方式呢？果然 login -> my profile -> Password & Authentication -> Two Factor Authentication 这里同样可以使用 [Authenticator][auth] 来进行身份验证[^2]。

设置好以后，心里就踏实多了。总结起来，为了让你的站更加安全，至少需要额外做两件事情：

### <span style="font-size: 130%; color: green; font-weight: 600;">&#9312;</span> 注册邮箱的两步验证
> 这里推荐 [outlook.com][outlook] 邮箱，密码+验证码做得一点也不麻烦；[Gmail](https://gmail.com) 邮箱我特地找了一下，还不支持自家的 [Authenticator][auth] ...

### <span style="font-size: 130%; color: green; font-weight: 600;">&#9313;</span> [VPS][vps] 账号的两步验证
> 一脉相承使用 [Authenticator][auth] 做第二步验证，这也便于统一管理。

最后，你的手机里面 [Authenticator][auth] 的界面大概就是这个样子了：

![Auth-All](http://7xljs8.com1.z0.glb.clouddn.com/auth-all.png "Auth from Risk")

最最后，多说一句，其实如果你自己想对外提供这种两步验证服务，可以首先将信息使用 google 的 [infographics api](https://developers.google.com/chart/infographics/docs/qr_codes) 生成 QR Code，然后再结合 [Authenticator][auth] 对外提供服务。

****
<br>

[^1]: 这里确实给了我惊喜，好的产品的真谛就是真正地是想用户所想
[^2]: 这里需要注意的是 [Linode][linode] 生成你的 QR Code 是调用的 [google][google] 的 chart api，你需要保证 <googleapis.com> 能够正常访问，否则你懂得 ...

[google]:    https://google.com/ncr
[microsoft]: http://microsoft.com
[outlook]:   https://outlook.com
[linode]:    https://linode.com
[vps]:       https://en.wikipedia.org/wiki/Virtual_private_server
[auth]:      https://itunes.apple.com/us/app/google-authenticator/id388497605?mt=8
