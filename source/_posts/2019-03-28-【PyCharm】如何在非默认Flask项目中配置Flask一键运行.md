---
title: 【PyCharm】如何在非默认Flask项目中配置Flask一键运行
urlname: how-to-configure-flask-run-in-pycharm
toc: true
date: 2019-03-28 16:18:17
updated: 2019-03-28 16:18:17
tags: [PyCharm, Flask]
categories: 
---

最近遇到了一个这样的需求：用PyCharm打开的项目中有若干个子文件夹，其中一个是Flask模块，还有一个是机器学习模块。

<!--more-->

```
project_base/
    ml/
        ...
    demo/
        demo.py
        template/
        static/
```

demo模块需要接收用户输入，调用ml模块，得到输出，再展示在前端。但是这个项目是Pure Python项目。于是我在命令行中设置了`FLASK_APP=demo/demo.py`，直接在`project_base`下执行`flask run`。但这样还是不太方便，于是我尝试在PyCharm中配置Run Configuration，如图：

![Run Configuration](config-screenshot.png)

## 法1：直接配置

在Linux下，我首先遇到了relative import的问题，最后通过在venv中设置`PYTHONPATH`解决了。然后就直接配置。

![Configuration Settings](config-settings.png)

设置过程参考了[^set]，包括：

* 将Script path设为`venv/bin/flask`
* 将Parameters设为`run`
* 添加环境变量`FLASK_APP=demo/demo.py`（虽然之前已经配置过了，也可以单独加在这里）
* 将Working directory设为`project_base`

[^set]: [Setting Up a Flask Application in PyCharm](https://blog.miguelgrinberg.com/post/setting-up-a-flask-application-in-pycharm)

然后就可以跑了。

## 法2：使用配置模板

在Windows下虽然可以按照类似的步骤来配置，但是虚拟环境中并没有`venv/bin/flask`这个东西，我也不知道怎么用`flask-cli`。不过有一个更简单的方法，即复制配置模板：

![复制配置模板](template-config.png)

只需要填一下Target，它就会帮你把其他事情（比如`PYTHONPATH`之类的）完成，然后就可以随便Run/Debug/...了。

![运行中](run.png)