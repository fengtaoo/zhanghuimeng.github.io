---
title: 如何debug Hexo博客
urlname: how-to-debug-a-hexo-blog
toc: true
date: 2019-04-04 20:40:58
updated: 2019-04-06 01:23:00
tags: [Hexo, WebStorm]
categories: Blogging
---

今天我尝试把博客的renderer换回[hexo-renderer-markdown-it](https://github.com/hexojs/hexo-renderer-markdown-it)，结果换完之后编译不出来了，`hexo s`报如下错误：

<!--more-->

```
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
TypeError [ERR_INVALID_ARG_TYPE]: The "id" argument must be of type string. Received type object
    at validateString (internal/validators.js:125:11)
    at Module.require (internal/modules/cjs/loader.js:632:3)
    at require (internal/modules/cjs/helpers.js:22:18)
    at C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\hexo-renderer-markdown-it\lib\renderer.js:13:25
    at Array.reduce (<anonymous>)
    at Hexo.module.exports (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\hexo-renderer-markdown-it\lib\renderer.js:12:26)
    at Hexo.tryCatcher (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\util.js:16:23)    at Hexo.<anonymous> (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\method.js:15:34)
    at Promise.then.text (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\hexo\lib\hexo\render.js:61:21)
    at tryCatcher (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\util.js:16:23)
    at Promise._settlePromiseFromHandler (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\promise.js:512:31)
    at Promise._settlePromise (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\promise.js:569:18)
    at Promise._settlePromiseCtx (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\promise.js:606:10)
    at Async._drainQueue (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\async.js:138:12)
    at Async._drainQueues (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\async.js:143:10)
    at Immediate.Async.drainQueues [as _onImmediate] (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\async.js:17:14)
    at runCallback (timers.js:705:18)
    at tryOnImmediate (timers.js:676:5)
    at processImmediate (timers.js:658:5)
FATAL The "id" argument must be of type string. Received type object
TypeError [ERR_INVALID_ARG_TYPE]: The "id" argument must be of type string. Received type object
    at validateString (internal/validators.js:125:11)
    at Module.require (internal/modules/cjs/loader.js:632:3)
    at require (internal/modules/cjs/helpers.js:22:18)
    at C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\hexo-renderer-markdown-it\lib\renderer.js:13:25
    at Array.reduce (<anonymous>)
    at Hexo.module.exports (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\hexo-renderer-markdown-it\lib\renderer.js:12:26)
    at Hexo.tryCatcher (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\util.js:16:23)    at Hexo.<anonymous> (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\method.js:15:34)
    at Promise.then.text (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\hexo\lib\hexo\render.js:61:21)
    at tryCatcher (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\util.js:16:23)
    at Promise._settlePromiseFromHandler (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\promise.js:512:31)
    at Promise._settlePromise (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\promise.js:569:18)
    at Promise._settlePromiseCtx (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\promise.js:606:10)
    at Async._drainQueue (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\async.js:138:12)
    at Async._drainQueues (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\async.js:143:10)
    at Immediate.Async.drainQueues [as _onImmediate] (C:\Users\lenovo\Documents\GitHub\zhanghuimeng.github.io\node_modules\bluebird\js\release\async.js:17:14)
    at runCallback (timers.js:705:18)
    at tryOnImmediate (timers.js:676:5)
    at processImmediate (timers.js:658:5)
```

看了半天，也没搞明白renderer觉得哪里有问题；如果是某篇文章导致的问题的话，那它也不会告诉我到底是哪一篇。所以Hexo真要debug起来还挺麻烦的。

## 法1：命令行运行+WebStorm监听

如果以“Hexo debug”作为关键词搜索的话，很可能会找到这篇文章：[Debug hexo blog generation in webstorm](http://www.codeblocq.com/2016/02/Debug-hexo-blog-generation-in-webstorm/)。这篇文章讲的是怎么在当时（2016年）的WebStorm里配置nodejs debug configuration，不过它在现在的WebStorm（和nodejs）版本中都不再适用了。

如果还想那么配置的话，需要再配合一个Listener。首先新建Node.js Run Configuration如下图：

![配置示意图](deprecated-approach-config.png)

其中值得注意的参数包括：

* Working directory：博客根目录
* Node parameters：`--inspect-brk=9229`[^brk]（这是Node V8的更新[^node8]）；9229是被监听的端口（默认就是9229）
* JavaScript file：本质上是`hexo-cli`对应的源文件，可能随操作系统有差别（此处可参见[^node-cli]）
  * \*nix：`node_modules/.bin/hexo`（这本质上是一个symlink）
  * Windows：`node_modules/hexo/node_modules/hexo-cli/bin/hexo`（这大概相当于主程序）[^win-hexo]
* Application paramaters：我觉得填`g`和`s`都可以，按需求即可

[^node8]: [node Issue #13564 - Node v 8: bad option: --expose_debug_as=v8debug](https://github.com/nodejs/node/issues/13564)

[^brk]: `--inspect-brk`的意思是在第一行代码打上断点。如果直接用这个参数且没有成功attach Debugger的话，会出现一直hang着不动的情况（因为没有办法continue）。当然，我觉得hang也是为了让debugger能attach上去，要不然应用都执行完了。参见[How to configure WebStorm debugger to work with my Express app](https://stackoverflow.com/questions/52274440/how-to-configure-webstorm-debugger-to-work-with-my-express-app)

然后新建Attach to Run/Debug Configuration如下图：

![Debugger的配置](listener-config.png)

监听刚才配置的端口（或者默认的9229）。

现在，**Run**刚才配置的hexo debug应用（注意不是**Debug**）：

![开始运行hexo应用](start-hexo-run.png)

再启动attach configuration debug：

![启动listener](start-listener.png)

从图中可以看出，这个configuration不能被run，只能被debug，它启动之后就监听到`localhost:9229`正在debug的应用了。现在它停在了`#!/usr/bin/env node`这一行上。

然后就可以继续debug了。

## 法2：直接在WebStorm上配置Node.js应用

事实上，现在（2019年4月）直接在WebStorm上配置一个Node.js的Hexo run configuration，然后debug这个configuration就可以了。

（所以刚才的那些步骤显得有些傻x……）

首先新建Node.js Run Configuration如图：

![配置示意图](config.png)

其中值得注意的参数包括：

* Node parameters：不需要填`--inspect-brk`了
* Working directory：同上
* JavaScript file：同上
* Application paramaters：同上

[^node-cli]: [stackoverflow - how can I debug a node app that is started via the command line (cli) like forever or supervisor?](https://stackoverflow.com/questions/30942953/how-can-i-debug-a-node-app-that-is-started-via-the-command-line-cli-like-forev)

[^win-hexo]: [Debug Hexo with VS Code](https://gary5496.github.io/2018/03/nodejs-debugging/)

……然后直接开始Debug就行了。

![Debug中](direct-debug.png)

---

当然，这只能Debug Hexo本身生成的过程，生成出来的静态网页是debug所不管的。而且有时候仍然缺乏帮助……如果不是因为我Node.js水平太差的话。

至于上面的错误，我最后发现，我把hexo-renderer-markdown-it的配置项的格式填错了。改掉就好了。
