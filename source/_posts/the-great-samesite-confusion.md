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

`SameSite` 在 2020 年初的正式启用，导致一些网站需要做一些比较困难的适配才能继续实现第三方访问的功能，虽然如此，但还是因为它是浏览器防御的一个补充而广受好评。这也直接导致了网络上出现了大量的关于 cookie 这个 “新” 属性机制的文章和讨论。

## 术语玩的很溜，定义马马虎虎

这些帖子里，有一些写的很准确，比如 [Rowan Merwood](https://twitter.com/rowan_m) 的 [web.dev piece](https://web.dev/samesite-cookies-explained/) 中题为 “SameSite cookies 的解释” 一文。但大多数文章都没有好好解释清楚 *site* 这个概念，而 *same-site 请求* 和 *cross-site 请求* 正是从 *site* 这里衍生出来的两个概念。

不仅如此，还有很多帖子混淆了 “origin” 和 “site” 这两个术语，至少算是使用不严谨，这包括信息安全社区很有影响力的人写的一些东西。

回到 2019 年 2 月份，Kristian Bremberg 在 Detectify 博客上[写了以下内容](https://blog.detectify.com/2019/02/05/guide-http-security-headers-for-better-web-browser-security/)：

> SameSite 是一个很新并且对防御 CSRF 攻击很有效的属性。如果一个 cookie 中使用了 SameSite 这个属性，那么浏览器会确保携带有这个 cookie 的请求一定来自这个 cookie 所属的 **origin** [原文如此]。

（重点是我加的）

信息安全大佬 [Troy Hunt](https://twitter.com/troyhunt) 在一篇发布于 2020 年 1 月份题为 “ SameSite 策略使混乱的 Cookies 即将终结” [开创性的帖子](https://www.troyhunt.com/promiscuous-cookies-and-their-impending-death-via-the-samesite-policy/)中，描述了 `SameSite` 属性的不同取值的效果：

> 1. None：现在没有设置 SameSite 的值的时候，Chrome 的默认值是这个
> 2. Lax：在 **cross-origin** 请求上携带 cookies 时有一些限制
> 3. Strict：在 **cross-origin** 请求上携带 cookies 时有很严格的限制

（重点是我加的）

然后在几个月之后，在另一篇分析了 `SameSite` 的出现将会如何影响黑客们所珍视的一系列漏洞的引人入胜的[文章](https://blog.reconless.com/samesite-by-default/)中，[Reconless 团队](https://twitter.com/0xReconless)写了下面这段话：

> 在更新之后，所有没有显式携带 `SameSite` 属性的 cookies 会被认为是 `SameSite=Lax`。这意味着除了顶级导航之外， **cross-origin** 请求再也不会携带 cookies。

（重点是我加的）

*Domain*，*host*，*origin*，*site*…… 在平时的交流中很自然的会无法准确使用这些术语；当然，可以原谅对这些术语不严谨的使用，如果你对此感到负罪感，那更好。

## “site” 和 “origin” 可以混用吗？

然而，`SameSite` 属性的出现带来了一些问题……

- 在这种场景下，是否有必要仔细区分 “origin” 和 “site”？
- 这两者是否只是拼写不同，而本质是一样的？
- *cross-site* 请求和 *cross-origin* 请求之间毫无区别？
- 那这个 cookie 属性也能被叫做 “SameOrigin” 吗？
- 或者说，“site” 和 “origin” 之间到底有没有真正的区别，对实现有影响吗？
- 然后，如果这个区别确实有影响的话，那是什么样的影响？

你可能已经根据本文的标题猜到了答案：在 `SameSite` 这个上下文中，“site” 有一个非常技术性的含义，然而却被过度忽视了； *site* 和 *origin* 之间的区别也确实会有影响，然而这两个概念经常会被混淆。

对这个术语的误用会对有些人有影响。在 Chrome 启用 `SameSite` 之后的几个月后，Google 的 Web 开发布道师北村英治（Eiji Kitamura）认为写一篇完整的关于 “origin” 和 “site” 区别的文章很有必要。

为了理解为什么这个区别很重要，首先你需要搞懂 *origin* 和 *site* 之间的区别