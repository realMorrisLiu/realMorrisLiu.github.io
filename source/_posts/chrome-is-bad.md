---
title: Chrome 不好
date: 2021-05-06 17:58:07
tags:
- Chrome
- Browser
- macOS

---

> 本文翻译自 Loren Brichter 的 Chrome is Bad
>
> 原文链接：https://chromeisbad.com

[2020 年 12 月 12 日](https://twitter.com/lorenb/status/1337832978253230081)

简单版：Google Chrome 会在你电脑上装一个叫 Keystone 的更新程序，这与 WindowServer（系统进程）中大量无法解释的 CPU 使用率[异常相关](https://twitter.com/lorenb/timelines/1338892756752732169)[[1]](#hiding)，而且，<u>**即使 Chrome 没有运行**</u>**，我的电脑也会非常卡**。把 Chrome 和 Keystone 删掉之后，我电脑就*一直都很快了*。[点这儿看看怎么做](#delete)。

完整版：我感觉自己的全新 16 寸 MacBook Pro 即使简单滑一滑也会有点卡。在活动监视器中 \*完全看不到\* Chrome 占用 CPU，但是 *WindowServer* 的占用异常的高，大概有 80%（正常来说应该是少于 10% 的）。

做了所有常规操作（退出应用，登出其他用户，重启，重置 PRAM/SMC 等）之后，并没有任何卵用，然后我就想起来之前我为了测试一个网站装了个 Chrome。

我把 Chrome 删了，在删 Chrome 的一些配置文件和缓存的时候，也注意到了 Keystone。我把我能找到的所有 Google 的东西都删了，重启电脑，然后简直就是天壤之别。*所有应用都瞬间响应并且肉眼可见的快了，并且 WindowServer 的 CPU 占用率也降到了 10% 以下*。

然后我又想到了其它事情，自从我买了 2015 款 iMac，我的家人就一直在抱怨它很卡。我试了我能想到的所有方法 —— 它搭载了 Fusion 磁盘，然后症状表现的跟 SSD 坏了一样 —— 但是磁盘诊断没有任何毛病。我们甚至把电脑抹掉然后全新安装过好几次。

然后我想起来了，*每次我们重装完电脑之后，第一件事就是装 Chrome*。我把 Chrome 和 Keystone 产生的所有垃圾文件都删了，重启，然后它快的 *就像一台全新的电脑*。

没错，我知道这听起来就像不靠谱的电视广告，但是这个方法如此有用以至于我花了整整 5 美元买了这个域名并且建了这个网站，即使这样让我看起来像疯了一样。

## <a name="delete"></a>OK 这很奇怪，你怎么把 Chrome 和 Keystone 删掉的？

> 2020 年 12 月 15 日更新，Google Chromium 团队的一些好人很严肃的对待这个问题，如果你对这很感兴趣并且可以贡献一些 sample 的话，请看[这个主题](https://twitter.com/lorenb/status/1339364446305710088) —— 到 [crbug 1158402](https://bugs.chromium.org/p/chromium/issues/detail?id=1158402#c18) 的直链。

1. 打开 **`/应用程序`** 文件夹把 **`Chrome`** 拖到废纸篓里。
2. 在 **`访达`** 中点击 **`前往`** 菜单（在屏幕最上方），然后点击 “ **`前往文件夹...`** ”。
3. 在里面输入 **`/资源库`** 然后按回车。
   - 检查下列文件夹： **`LaunchAgents`** ， **`LaunchDaemons`** ， **`Application Support`** ， **`Caches`** ， **`Preferences`** 。
   - 删掉所有 **`Google`** 文件夹和所有以 **`com.google...`** 和 **`com.google.keystone...`** 开头的东西
4. 再打开 “ **`前往文件夹...`** ”。
5. 在里面输入 **`~/资源库`** 然后按回车。（注意这个 “~”）
   - 检查下列文件夹： **`LaunchAgents`** ， **`Application Support`** ， **`Caches`** ， **`Preferences`** 。
   - 删掉所有 **`Google`** 文件夹和所有以 **`com.google...`** 和 **`com.google.keystone...`** 开头的东西
6. 清倒废纸篓，重启电脑。

## 那现在我该用什么浏览器呢？

[Safari](https://www.apple.com/safari/) 挺好的并且是你 Mac 上预装的。它又快又高效。如果你需要一个基于 Chromium 的浏览器的话，可以试试 [Brave](https://brave.com/)，[Opera](https://www.opera.com/)，或者 [Vivaldi](https://vivaldi.com/)。（Brave 和 Vivaldi 这俩都用了一个叫 [Sparkle](https://sparkle-project.org/) 的开源库，用来提供更新功能，就不可能出现这个问题了。）Firefox 有一个令我很在意的指针输入延迟非常明显的问题，可能其他人觉得还好。（Mozilla 是一堆目光短浅的笨蛋，他们解雇了 Servo 团队。如果 Servo 团队重组的话，我会推荐他们做的所有东西）。

## Keystone 到底是怎么一回事？

2009 年，Google 把它加到 Google Earth 里的时候，Wired 第一次[报道了 Keystone](https://www.wired.com/2009/02/why-googles-sof/)。一直以来它都在做一些会让 Mac 崩溃的奇怪的事情，对于一个自动更新程序来说，这些事情完全没必要。

我要对所有在 Google 从事 Chrome 相关工作的好人们说：在你们写的代码和人们电脑上发生的事情之间有点问题。我希望你们可以跟踪这个问题并且给我们一个诚实的事件剖析。

------

更新于 2020 年 12 月 15 日：

收集的一些轶事：

https://twitter.com/lorenb/timelines/1338892756752732169

## FAQ

### 只需要重启可能就能解决这个问题。

不会的，这个问题即使是重启，没有任何正在建立索引或者正在运行程序等情况的时候，也一直会出现，否则完全无法解释。活动监视器里没有任何明显的问题，许多人都认真地尝试进行故障排除，但无济于事（显然包括重启），并且唯一可以 *立即且永远* 解决这个问题的方法就是删掉 Chrome 和 Keystone。

### 这是观察者效应

确实，活动监视器本身也占用 CPU，所以只是寻找这个问题的话也会让 WindowsServer 的占用稍稍升高。这并不是实际正在发生的事情。这个问题十分严重，活动监视器的 CPU 占用的影响完全无法跟 WindowServer CPU 占用的抖动相提并论（用 ‘top’ 命令也不会有任何肉眼可见的区别）。

*未完待续*

