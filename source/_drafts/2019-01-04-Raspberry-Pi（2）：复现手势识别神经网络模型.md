---
title: Raspberry Pi（2）：复现手势识别神经网络模型
urlname: raspberry-pi-2-model
toc: true
date: 2019-01-04 22:21:14
updated: 2019-01-04 22:21:14
tags: [Raspberry Pi]
---

两个月过去了，终于有时间开机搞树莓派了……

## 树莓派的环境

### 分辨率问题

目前树莓派的显示比较小。但是可以进行一些调整。

### 安装OpenCV（py2）和TensorFlow（py3）

我本来想直接按照[^guessing]中给出的方法直接安装的，结果遇到了很多问题。

[^guessing]: [CSDN - Tensorflow+树莓派，自制“猜拳神器”](https://blog.csdn.net/Lauyeed/article/details/79345685)

在python2下安装OpenCV倒是足够简单，按下列步骤并没有什么问题：

```bash
sudo apt-get update
sudo apt-get install libopencv-dev
sudo apt-get install python-opencv
```

然后在python2中就可以打开cv2了：

```py
import cv2
cv2.__version__
```

但是安装TensorFlow的时候遇到了谜一样多的问题。现在TensorFlow有官方发行的树莓派版本了[^medium]，所以我决定不用之前的[TensorFlow on Raspberry Pi](https://github.com/samjabrahams/tensorflow-on-raspberry-pi)，而是改用官方版本，据说执行以下两句就可以了：

```bash
sudo apt install libatlas-base-dev
pip3 install tensorflow
```

[^medium]: [Medium - TensorFlow 1.9 Officially Supports the Raspberry Pi](https://medium.com/tensorflow/tensorflow-1-9-officially-supports-the-raspberry-pi-b91669b0aa0)

结果pip3不断报以下错误：

```
Exception:
Traceback (most recent call last):
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/connectionpool.py", line 391, in _make_request
    six.raise_from(e, None)
  File "<string>", line 2, in raise_from
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/connectionpool.py", line 387, in _make_request
    httplib_response = conn.getresponse()
  File "/usr/lib/python3.5/http/client.py", line 1198, in getresponse
    response.begin()
  File "/usr/lib/python3.5/http/client.py", line 297, in begin
    version, status, reason = self._read_status()
  File "/usr/lib/python3.5/http/client.py", line 258, in _read_status
    line = str(self.fp.readline(_MAXLINE + 1), "iso-8859-1")
  File "/usr/lib/python3.5/socket.py", line 576, in readinto
    return self._sock.recv_into(b)
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/contrib/pyopenssl.py", line 271, in recv_into
    raise timeout('The read operation timed out')
socket.timeout: The read operation timed out

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/connectionpool.py", line 594, in urlopen
    chunked=chunked)
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/connectionpool.py", line 393, in _make_request
    self._raise_timeout(err=e, url=url, timeout_value=read_timeout)
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/connectionpool.py", line 313, in _raise_timeout
    raise ReadTimeoutError(self, url, "Read timed out. (read timeout=%s)" % timeout_value)
requests.packages.urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='pypi.org', port=443): Read timed out. (read timeout=15)

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/pip/basecommand.py", line 215, in main
    status = self.run(options, args)
  File "/usr/lib/python3/dist-packages/pip/commands/install.py", line 353, in run
    wb.build(autobuilding=True)
  File "/usr/lib/python3/dist-packages/pip/wheel.py", line 749, in build
    self.requirement_set.prepare_files(self.finder)
  File "/usr/lib/python3/dist-packages/pip/req/req_set.py", line 380, in prepare_files
    ignore_dependencies=self.ignore_dependencies))
  File "/usr/lib/python3/dist-packages/pip/req/req_set.py", line 554, in _prepare_file
    require_hashes
  File "/usr/lib/python3/dist-packages/pip/req/req_install.py", line 278, in populate_link
    self.link = finder.find_requirement(self, upgrade)
  File "/usr/lib/python3/dist-packages/pip/index.py", line 465, in find_requirement
    all_candidates = self.find_all_candidates(req.name)
  File "/usr/lib/python3/dist-packages/pip/index.py", line 423, in find_all_candidates
    for page in self._get_pages(url_locations, project_name):
  File "/usr/lib/python3/dist-packages/pip/index.py", line 568, in _get_pages
    page = self._get_page(location)
  File "/usr/lib/python3/dist-packages/pip/index.py", line 683, in _get_page
    return HTMLPage.get_page(link, session=self.session)
  File "/usr/lib/python3/dist-packages/pip/index.py", line 792, in get_page
    "Cache-Control": "max-age=600",
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/sessions.py", line 501, in get
    return self.request('GET', url, **kwargs)
  File "/usr/lib/python3/dist-packages/pip/download.py", line 386, in request
    return super(PipSession, self).request(method, url, *args, **kwargs)
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/sessions.py", line 488, in request
    resp = self.send(prep, **send_kwargs)
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/sessions.py", line 630, in send
    history = [resp for resp in gen] if allow_redirects else []
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/sessions.py", line 630, in <listcomp>
    history = [resp for resp in gen] if allow_redirects else []
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/sessions.py", line 190, in resolve_redirects
    **adapter_kwargs
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/sessions.py", line 609, in send
    r = adapter.send(request, **kwargs)
  File "/usr/share/python-wheels/CacheControl-0.11.7-py2.py3-none-any.whl/cachecontrol/adapter.py", line 47, in send
    resp = super(CacheControlAdapter, self).send(request, **kw)
  File "/usr/share/python-wheels/requests-2.12.4-py2.py3-none-any.whl/requests/adapters.py", line 423, in send
    timeout=timeout
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/connectionpool.py", line 643, in urlopen
    _stacktrace=sys.exc_info()[2])
  File "/usr/share/python-wheels/urllib3-1.19.1-py2.py3-none-any.whl/urllib3/util/retry.py", line 315, in increment
    total -= 1
TypeError: unsupported operand type(s) for -=: 'Retry' and 'int'
```

我在这里折腾了一段时间，甚至upgrade了系统，但是毫无用处。最后我发现确实是pip本身的问题，需要手动卸载重装[^pip]：

```bash
apt-get remove python-pip python3-pip
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
sudo python3 get-pip.py
```

[^pip]: [stackoverflow - Python PIP Install throws TypeError: unsupported operand type(s) for -=: 'Retry' and 'int'](https://stackoverflow.com/questions/37495375/python-pip-install-throws-typeerror-unsupported-operand-types-for-retry)

然后执行了

```bash
sudo pip3 install tensorflow
```

这回TensorFlow倒是真的装上了，但是直接用`python3`进入命令行无法`import tensorflow`，会报以下错误：

```
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/home/pi/.local/lib/python3.5/site-packages/tensorflow/__init__.py", line 22, in <module>
    from tensorflow.python import pywrap_tensorflow  # pylint: disable=unused-import
  File "/home/pi/.local/lib/python3.5/site-packages/tensorflow/python/__init__.py", line 49, in <module>
    from tensorflow.python import pywrap_tensorflow
  File "/home/pi/.local/lib/python3.5/site-packages/tensorflow/python/pywrap_tensorflow.py", line 25, in <module>
    from tensorflow.python.platform import self_check
ImportError: No module named 'tensorflow.python.platform'
```

如果用`sudo python3`会报以下警告：

```
/usr/lib/python3.5/importlib/_bootstrap.py:222: RuntimeWarning: compiletime version 3.4 of module 'tensorflow.python.framework.fast_tensor_util' does not match runtime version 3.5
  return f(*args, **kwds)
/usr/lib/python3.5/importlib/_bootstrap.py:222: RuntimeWarning: builtins.type size changed, may indicate binary incompatibility. Expected 432, got 412
  return f(*args, **kwds)
```

不过至少勉强能跑了……

后来发现有这个警告好像还算是正常的。

## 训练数据

### 用OpenCV调用摄像头拍摄图片

我最开始直接用了[^guessing]中的代码，结果发现跑不起来，`cv2.VideoCapture.read()`总是返回`false`。经过查找资料[^videocapture]，发现应该执行以下命令：

[^videocapture]: [stackoverflow - VideoCapture.open(0) won't recognize pi cam](https://stackoverflow.com/questions/29583533/videocapture-open0-wont-recognize-pi-cam)

```bash
sudo modprobe bcm2835-v4l2
```

然后倒是能拍了，效果如图：

![这是白平衡非常不对么](picam-balance.png)

我不太懂摄影这套理论，不知道这个状况应该叫什么，但总之很不对头就是了……

之后改善了一下光线状况，重启了一下树莓派（因为没找到单独重启摄像头的方法……）之后，拍出来的东西总算能看了，但是不知为何发绿：

![这个白平衡真的不太对](picam-green.jpg)

但是我当时没找到通过OpenCV设置白平衡的方法，或者说，它已经在自动调整白平衡了，我不知道怎么能手动把它设得更好。以及上图是左右和上下颠倒的，这好像是默认设置，但我认为没有调整的必要，反正训练的时候是颠倒的，测试的时候也是颠倒的……

后来（训练了几轮之后），我找到了一个自适应调节白平衡的方法[^white-balance]：

[^white-balance]: [stackexchange - Raspberry Pi - Custom White Balancing with picamera](https://raspberrypi.stackexchange.com/questions/22975/custom-white-balancing-with-picamera)

```py
# Start off with ridiculously low gains
rg, bg = (0.5, 0.5)
camera.awb_gains = (rg, bg)
with picamera.array.PiRGBArray(camera, size=(128, 72)) as output:
    # Allow 30 attempts to fix AWB
    for i in range(30):
        # Capture a tiny resized image in RGB format, and extract the
        # average R, G, and B values
        camera.capture(output, format='rgb', resize=(128, 72), use_video_port=True)
        r, g, b = (np.mean(output.array[..., i]) for i in range(3))
        print('R:%5.2f, B:%5.2f = (%5.2f, %5.2f, %5.2f)' % (
            rg, bg, r, g, b))
        # Adjust R and B relative to G, but only if they're significantly
        # different (delta +/- 2)
        if abs(r - g) > 2:
            if r > g:
                rg -= 0.1
            else:
                rg += 0.1
        if abs(b - g) > 1:
            if b > g:
                bg -= 0.1
            else:
                bg += 0.1
        camera.awb_gains = (rg, bg)
        output.seek(0)
        output.truncate()
```

简单来说就是根据当前采集的图像的平均颜色调整白平衡红色和蓝色增益的大小，使得平均颜色更接近灰色。使用之后效果如图：

![看起来正常多了](good_balance.jpg)

### 数据拍摄和初步数据处理

开始时，我拍摄数据的代码和[^guessing]基本上是类似的，也就是用OpenCV去拍；后来我发现，OpenCV只在python2下好使，在python3下虽然也勉强能用，但一调整分辨率，整个画面就会开始扭曲：

![说扭曲也不太对头，不好描述，反正肯定没法用](opencv-py3-fault.png)

也许是我装的不对吧。大部分网上的教程都要求从源代码编译，但事实上（大概是因为现在有了为树莓派准备的二进制版本？）没有这个必要，直接按照下列步骤装就可以[^bouvet][^sto-opencv-py3]：

```bash
sudo apt-get install python-opencv
sudo pip3 install opencv-python
sudo apt-get install libjasper-dev
sudo apt-get install libqtgui4
sudo apt-get install libqt4-test
```

[^bouvet]: [Mark West - Adding AI to the Raspberry Pi with the Movidius Neural Compute Stick](https://www.bouvet.no/bouvet-deler/adding-ai-to-edge-devices-with-the-movidius-neural-compute-stick)

[^sto-opencv-py3]: [stackoverflow - Problems importing open cv in python](https://stackoverflow.com/questions/52378554/problems-importing-open-cv-in-python)

最后不得不改成用`PiCamera`，拍摄和实际部署时都是。至于为啥非要用py3，主要是因为Movidius那个API是固定装在py3上的，这使得降级回py2不太可行……

```
此处应有用PiCamera的代码……
```

区别只包括，我把训练时直接处理数据的代码改成了先处理数据，然后存成`.npz`，训练的时候再直接读`.npz`。

## 训练

在本机（windows）上进行训练来着。效果特别好。

<!--
## movidius

>I found the Movidius documentation to be either missing or incomplete — a lot of links indexed by Google now 404, the forums are pretty quiet and the Youtube videos and other resources imply that a host PC running full Ubuntu is required. That’s not the case — it is possible to do a full install straight onto your Raspberry Pi 3 as we outline below.[^quick-movidius]

[^quick-movidius]: [Movidius Neural Compute Stick and Raspberry 3 — quick start guide](https://medium.com/@hsheil/movidius-neural-compute-stick-and-raspberry-3-quick-start-guide-a89ff5e1d7ca)
-->

## 实际测试

### 舵机的使用

###