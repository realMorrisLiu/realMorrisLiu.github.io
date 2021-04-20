---
title: 关于 SameSite 的巨大困惑
date: 2021-04-20 19:17:43
categories:
- Web Development
tags:
- SameSite
- cookies
- origin
- same-site
- site
- subdomain
- subdomain takeover
---

> 本文翻译自 Julien Cretel 的 The great SameSite confusion
>
> 原文链接：https://jub0bs.com/posts/2021-01-29-great-samesite-confusion/

在本文中，我剖析了对 cookie 中 `SameSite` 属性的一个常见误解，并且探讨了这个问题对于 Web 安全的潜在影响。

## 太长不看

- cookie 的 `SameSite` 属性不太好理解。
- 把 *site* 和 *origin* 混为一谈是一种很常见但是有害的错误。
- *site* 的概念不够直观，很难理解。
- 有些请求是 cross-origin 的，但同时又是 same-site 的。
- `SameSite` 只对 cross-site 的请求有意义。
- `SameSite` 在你的子域上设置了一个目标。
- 被误导的开发人员可能会过度避免使用 `SameSite=Strict`。

## `SameSite` 的到来

毫无疑问，你肯定听说过 cookie 的 `SameSite` 这个属性。2020 年 2 月份的时候，Chrome 的一个决定让 `SameSite` 登上头条：Chrome 决定改变 `SameSite` 的默认行为了；几个月后，Firefox 也做了相同的决定。

自从 `SameSite` 在 2016 年被首次引入以来，它就一直是实现浏览器时要考虑的一个核心功能，旨在对 cross-site 攻击，例如 CSRF（cross-site request forgery）和 XSSI（cross-site script inclusion），进行深度防御。

`SameSite` 在 2020 年初的正式启用，导致一些网站需要做一些比较困难的适配才能继续实现第三方访问的功能，虽然如此，但还是因为它是浏览器防御的一个补充而广受好评。这也直接导致了网络上出现了大量的关于 cookie 这个“新”属性机制的文章和讨论。

## 术语玩的很溜，其实不明白含义

这些帖子里，有一些写的很准确，比如 [Rowan Merwood](https://twitter.com/rowan_m) 的 [web.dev piece](https://web.dev/samesite-cookies-explained/) 中题为“SameSite cookies 的解释”一文。但大多数文章都没有好好解释清楚 *site* 这个概念，而 *same-site 请求* 和 *cross-site 请求* 正是从 *site* 这里衍生出来的两个概念。

*To be continued*