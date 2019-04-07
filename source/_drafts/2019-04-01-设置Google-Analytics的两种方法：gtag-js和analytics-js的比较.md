---
title: 设置Google Analytics的两种方法：gtag.js和analytics.js的比较
urlname: 2-methods-of-setting-google Analytics的两种方法：gtag.js和analytics.js的比较
toc: true
date: 2019-04-01 15:22:18
updated: 2019-04-01 15:22:18
tags: [Google Analytics]
categories:
---

前几天，我发现[material-x](https://github.com/xaoxuu/hexo-theme-material-x)主题的Google Analytics推送功能可能出了一些问题：自从换了这个主题之后，我的用户量骤降了90%，但是搜索结果中的曝光次数和总点击次数没有太大变化。因为我80%的用户都来自点击，所以我猜测问题在于推送而不在于排名。

<!--more-->

于是我尝试在主题的`<head>`中直接加入了从Google Analytics上拿的全局网站代码（gtag.js），然后就好了。

![用户数量恢复了](user-cnt.png)

![点击量一直没有大的变化](search-cnt.png)

现在这个bug大概已经修好了[^issue][^commit]，不过我还是很好奇，为什么会发生这样的问题。

[^issue]: [hexo-theme-material-x issue #63: Google Analytics推送问题](https://github.com/xaoxuu/hexo-theme-material-x/issues/63)

[^commit]: [hexo-theme-material-x commit 14492c1: fix ga](https://github.com/xaoxuu/hexo-theme-material-x/commit/14492c135057c93043b7f2f0df227165a97ad310?diff=unified)

## 推送代码被更新了？

就像[^commit]中显示的那样，之前的推送代码长这样：

```js
<script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
    ga('create', '<%- config.google_analytics_key %>', '<%- config.root_domain ? config.root_domain : config.url %>');
    ga('send', 'pageview');
</script>
```

现在的推送代码长这样：

```js
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=<%- config.google_analytics_key %>"></script>
<script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', '<%- config.google_analytics_key %>');
</script>
```

然而，我之前尝试使用Google Analytics时，只见过第一种代码，没见过第二种代码。所以难道是谷歌更新了，前一种不能用了？

## 两种推送代码？

事实上并不是这样。截止到我写这篇文章的时候，这两种跟踪代码仍然是都可以使用的。在[Google Analytics指南 - 设置 Google Analytics（分析）](https://developers.google.com/analytics/devguides/collection/)中，可以看到有两种可以用来跟踪的js库——也就对应着这两种代码了。

![gtag.js和analytics.js](dev-guide.png)

不过的确，第二种代码是谷歌2017年8月才发布的[^tw]，所以我之前并没有见过。

[^tw]: [medium - gtag.js 還是 analytics.js ?](https://medium.com/@ennovysun0629/gtag-js-%E9%82%84%E6%98%AF-analytics-js-de39f799fe7c)

## 它们的工作原理？

### analytic.js

[analytic.js](https://developers.google.com/analytics/devguides/collection/analyticsjs/)是较旧的库，