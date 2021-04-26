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

这些帖子里，有一些写的很准确，比如 [Rowan Merwood](https://twitter.com/rowan_m) 的 [web.dev piece](https://web.dev/samesite-cookies-explained/) 中题为 “SameSite cookie 的解释” 一文。但大多数文章都没有好好解释清楚 *site* 这个概念，而 *same-site 请求* 和 *cross-site 请求* 正是从 *site* 这里衍生出来的两个概念。

不仅如此，还有很多帖子混淆了 “origin” 和 “site” 这两个术语，至少算是使用不严谨，这包括信息安全社区很有影响力的人写的一些东西。

回到 2019 年 2 月份，Kristian Bremberg 在 Detectify 博客上[写了以下内容](https://blog.detectify.com/2019/02/05/guide-http-security-headers-for-better-web-browser-security/)：

> SameSite 是一个很新并且对防御 CSRF 攻击很有效的属性。如果一个 cookie 中使用了 SameSite 这个属性，那么浏览器会确保携带有这个 cookie 的请求一定来自这个 cookie 所属的 **origin** [原文如此]。

（重点是我加的）

信息安全大佬 [Troy Hunt](https://twitter.com/troyhunt) 在一篇发布于 2020 年 1 月份题为 “ SameSite 策略使混乱的 Cookie 即将终结” [开创性的帖子](https://www.troyhunt.com/promiscuous-cookies-and-their-impending-death-via-the-samesite-policy/)中，描述了 `SameSite` 属性的不同取值的效果：

> 1. None：现在没有设置 SameSite 的值的时候，Chrome 的默认值是这个
> 2. Lax：在 **cross-origin** 请求上携带 cookie 时有一些限制
> 3. Strict：在 **cross-origin** 请求上携带 cookie 时有很严格的限制

（重点是我加的）

然后在几个月之后，在另一篇分析了 `SameSite` 的出现将会如何影响黑客们所珍视的一系列漏洞的引人入胜的[文章](https://blog.reconless.com/samesite-by-default/)中，[Reconless 团队](https://twitter.com/0xReconless)写了下面这段话：

> 在更新之后，所有没有显式携带 `SameSite` 属性的 cookie 会被认为是 `SameSite=Lax`。这意味着除了顶级导航之外， **cross-origin** 请求再也不会携带 cookie。

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

为了理解为什么这个差别很重要，首先你需要搞懂 *origin* 和 *site* 之间的区别。

## 我们说的 “origin” 是什么意思？

如果你是 Web 开发人员，那你应该至少对 [Same-Origin 原则（SOP）](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)有所熟悉，这可以说是 Web 安全的主要支柱之一。显而易见，URI 的 *origin* 概念是 SOP 的核心，并且这个概念相对好理解一些。[RFC 6454 的 3.2 节](https://tools.ietf.org/html/rfc6454#section-3.2)中将 *origin* 定义成了一个三元组：

> 总的来说，如果两个 URI 有同样的 scheme，host，和 port 的话，那它们同属于一个 origin（即，表示相同的主体）。

其中 port 是可选的，如果没有指定 port 的话，会使用 scheme 的默认 port（例如 `http` 的 80 和 `https` 的 443）。[MDN Web 文档](https://developer.mozilla.org/en-us/docs/Glossary/Origin)中有一系列很有用的例子。

## 我们说的 “site” 是什么意思？

在这个令人头痛的通用术语之后，其实是 “site” 隐含了一种实际上比 *origin* 更难理解的概念。在一方面，术语 “site” 并不总是技术性的：它在 SOP 这个概念出现之前，通常被用于[出现 cross-site scripting 攻击的时候](https://blog.jeremiahgrossman.com/2006/07/origins-of-cross-site-scripting-xss.html)。另一方面，*site* 的现代概念充满了技术性困难。它与 host 的可注册 *域* 密切相关，*域* 在 [URL 当前标准](https://url.spec.whatwg.org/#host-registrable-domain)中是如下定义的：

> ……，一个域由最具体的公共后缀和紧随其后的域标签（如果有）组成。

（一个 host 的可注册域也称为 “eTLD+1”，是 “effective top-level domain plus one” 的缩写。）

在最简单的情况下，**origin 的 site 仅对应于 origin 的 host 的可注册域（如果有）**。

用这两个例子可以说明：

- `https://www.example.org` 的 site 是 `example.org`，因为 `org` 是这个 host 的最具体的公共后缀，因此，`example.org` 是这个 host 的 eTLD+1。
- `https://jub0bs.github.io` 的 site 是 `jub0bs.github.io`，因为 `github.io` 是这个 host 的最具体的公共后缀，因此，`jub0bs.github.io` 是这个 host 的 eTLD+1。

没错！也许令人惊讶，[但 `github.io` 是一个公共后缀](https://publicsuffix.org/list/public_suffix_list.dat)！

这里应该记一下，可注册域的概念是动态的，因为这个概念依赖于[公共后缀表](https://publicsuffix.org/list/)，这个表并不是固定不变的，它会随着时间变化。更不用说不同的浏览器不一定会紧跟公共后缀表的修改而更新。

技术细节不止于此！就像 web.dev 警告我们的那样，*site* 的概念仍在逐步形成中，并且会很快会纳入 scheme 之中。目前在 Chrome 中这个变动需要在 flag 里面去设置，但是不久之后就会变成一个正式的特性。但是，为了避免这种困难并延长这篇文章的相关性，在下面的内容中，我将仅考虑其 scheme 为 https 的 origins。

## Same-site 请求 vs. cross-site 请求

我们已经聊过了 *site* 的概念，现在我们终于可以聊一聊 *same-site* 和 *cross-site* 请求的概念了。一个给定的请求可以是 *same-site* 的，也可以是 *cross-site* 的。一个请求是 *same-site* 还是 *cross-site* 取决于请求的源 origin 和目标 origin 的 sites 的比较：

- 如果这两个 sites 是一样的，那么这个请求就是 *same-site*；
- 如果这两个 sites 不一样，那么这个请求就是 *cross-site*。

这里有三个例子：

1. 一个从 `https://foo.example.org` 发送到 `https://bar.example.org` 的请求是 same-site 的，因为这两个 origin 的 site 都是 `example.org`。
2. 一个从 `https://foo.github.io` 发送到 `https://bar.github.io` 的请求是 cross-site 的，因为第一个 origin 的 site 是 `foo.github.io`，而第二个 origin 的 site 是 `bar.github.io`。
3. 一个从 `https://foo.bar.example.org` 发送到 `https://bar.example.org` 的请求是 same-site 的，因为这两个 origin 的 site 都是 `example.org`。

如果你坚持读到了这里，我很感谢你的耐心。注意：重点要来了！

## Cross-origin，same-site 的请求

所有 cross-site 的请求都一定是 cross-origin 的；这已经很明确了。然而，就像上面的第一个和第三个例子描述的，并不是所有的 cross-origin 的请求都是 cross-site 的。

![Venn-diagram](cor_vs_csr.svg)

同时 `SameSite` 属性只和 cross-site 的请求有关；当一个请求是 cross-origin 但是是 same-site 的时候，这个属性不会生效。这就是为什么 *origin* 和 *site* 的区别很重要的原因。

## 在线示例

为了证明我的观点，我受到 Troy Hunt 的文章的启发，在 `samesitedemo.jub0bs.com` 上部署了一个包含两个 endpoint 的简单的 Go 服务。`/setcookie` 这个 endpoint 设置了一个 `SameSite=Strict` 的 cookie，就像这样：

```http
Set-Cookie: StrictCookie=foo; Path=/; Max-Age=3600; SameSite=Strict
```

`/readcookie` 打印请求所携带的 cookie（如果有的话）。同时我也设置了两个 “攻击” 页面：

- https://jub0bs.github.io/samesitedemo-attacker-foiled
- https://samesitedemo-attacker.jub0bs.com/

这两个页面都只包含一个去 `https://samesitedemo.jub0bs.com/readcookie` 的链接。

为了验证上面的想法，你可以这样做：

1. 访问 https://samesitedemo.jub0bs.com/setcookie。这样做会把 `Strict` cookie 设置到你的浏览器中。
2. 访问 https://jub0bs.github.io/samesitedemo-attacker-foiled 并且点击页面上的链接。因为攻击 URI 的 site（`jub0bs.github.io`）跟目标 URI 的 site（`jub0bs.com`）不同，浏览器在访问这个链接发送请求时不会携带上面设置的 cookie，所以响应中不会打印任何 cookie。`SameSite=Strict` 阻止了这个 “攻击”。
3. 现在访问 https://samesitedemo-attacker.jub0bs.com/ 然后点击页面上的链接。因为攻击 URI 的 site（`jub0bs.com`）跟目标 URI 的 site（`jub0bs.com`）相同，浏览器就会在访问链接的请求中携带上面设置的 cookie，然后响应中也会打印出这个 cookie 来。这种情况下，“攻击” 就生效了。

## 将 site 和 origin 混为一谈的代价

### 错误的安全感

暗示 `SameSite` 会对 *所有* cross-origin 请求生效是有害的，因为这会导致开发人员错误的认为 `SameSite` 可以保护他们的用户受到 *所有* cross-origin 攻击。这样的误解对忽视检查子域的安全级别的开发者来说尤其危险。尤其是，

- 子域接管，或
- 在相同 site 的子域上的 [cross-site scripting（XSS）](https://owasp.org/www-community/attacks/xss/)的实例，或
- 在相同 site 的子域上的 [HTML 注入](https://www.acunetix.com/vulnerabilities/web/html-injection/)实例

可能足以使攻击者绕过 `SameSite` 提供的相对保护。

子域接管是一种早在 2014 年[由 Detectify 带火](https://labs.detectify.com/2014/10/21/hostile-subdomain-takeover-using-herokugithubdesk-more/)的一种攻击方式。这种攻击方式利用子域上悬空的 DNS 记录来控制有问题的子域提供的部分或全部内容。攻击者可能会利用子域接管达到各种目的：污染、钓鱼等……还可以进行 cross-origin 攻击，否则就不可能有这样的效果！

例如，如果攻击者有能力接管 `https://vulnerable.example.org`，那么他或她可能就能够从易受攻击的子域提供的前端代码向 `https://example.org`（或其任何子域）发送恶意请求；并且这样的 same-site 的请求将携带所有相关的cookie，而不管其 `SameSite` 属性的值如何！

此类 cross-origin，same-site 的攻击甚至可能不需要子域接管。相同 site 的子域上存在 XSS 或者 HTML 注入漏洞可能足以满足攻击者发送恶意 cross-origin，same-site 请求的需要。

------

注：“跨站请求伪造” 在上面罗列的所有场景都是不恰当的用词，因为发出攻击 site 和成为目标的 site 是一致的；“跨域请求伪造”（CORF？）可以更贴切的描述这种攻击，不过我怀疑它能不能成为一个通用术语。

------

我发现令人着迷的是，与过去相比，`SameSite` 可能会使精明的攻击者更加关注你的子域和同级域，因为这些域正迅速成为针对坚韧的网络应用的 cross-origin 攻击的唯一漏洞。

### 对 `SameSite=Strict` 采用较慢

此外，这种误解也可能会由于对 `Lax` 值的偏好而减慢 `Strict` 值的采用。有些人确实在积极劝阻开发者们使用 `Strict` ，因为他们觉得 `Strict` 对可用性的影响比实际情况更严重。例如，在 Dareboost 的博客上，[你可以看到](https://web.archive.org/web/20201201211835/https://blog.dareboost.com/en/2017/06/secure-cookies-samesite-attribute/)下面的声明：

> 如果我们在 `dareboost.com` 上使用 `Strict` `SameSite` 的话，那么不管你有没有连接上，点击这个链接都不会登录成功。

这个声明是错误的：有问题的链接是在 `https://blog.dareboost.com` 上的，并且会跳转到 `https://www.dareboost.com` 上；因此，点击链接触发的请求是 same-site 请求，同时会携带 `www` 子域或者父级域的所有 cookie。

这篇文章的作者总结道：

> 最终用户会对这种行为产生困扰，所以你可能倾向于使用 `Lax` 模式。

这只是不公正的贬低 `Strict` 值的一个例子，但是

*To be continued...*

