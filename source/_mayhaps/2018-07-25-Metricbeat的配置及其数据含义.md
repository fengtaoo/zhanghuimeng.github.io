---
title: Metricbeat的配置及其数据含义
urlname: metricbeat-configuration-and-data-meaning
toc: true
date: 2018-07-25 00:57:07
updated: 2018-07-25 00:57:07
tags:
---

# 系统字段（system field）

来自[https://www.elastic.co/guide/en/beats/metricbeat/current/exported-fields-system.html](https://www.elastic.co/guide/en/beats/metricbeat/current/exported-fields-system.html)。

从操作系统收集的系统状态指标，如CPU和内存使用情况。

## `system`指标

`system`：本地系统测量数据。（不知道这个的具体内容是什么？）

### `core`指标

`system-core`类别（也就是`system.core.xxx`）中包含了多核系统中单个核的CPU测量数据。

* `system.core.id`
  * type: long
  * CPU核心编号（大概是第几个核的意思？）
* `system.core.user.pct`
  * type: scaled_float
  * format: 百分比（这是意味着已经换算成百分制了吗？一个旧版的[参考](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)指出，它是通过把一个浮点数乘上一个固定的`double`类型的数，然后用`long`类型来表示浮点数。）
  * 在用户空间中花费的CPU时间百分比。（大概说的是用户态。）
* `system.core.user.ticks`
  * type: long
  * 在用户空间中花费的CPU时间量。（相比之下，这个应该是总量。但问题是，什么时间区间内的总量呢？）
* `system.core.system.pct`
  * type: scaled_float
  * format: 百分比
  * 在内核空间中花费的CPU时间百分比。（大概说的是内核态。）
* `system.core.system.ticks`
  * type: long
  * 在内核空间中花费的CPU时间量。
* `system.core.nice.pct`
  * type: scaled_float
  * format: 百分比
  * 在低优先级进程上花费的时间占CPU时间的百分比。（不知道这个“低”是多低。）
* `system.core.nice.ticks`
  * type: long
  * 在低优先级进程上花费的CPU时间量。
* `system.core.idle.pct`
  * type: scaled_float
  * format: 百分比
  * 空闲时间占CPU时间的百分比。
* `system.core.idle.ticks`
  * type: long
  * 空闲CPU时间的总量。
* `system.core.iowait.pct`
  * type: scaled_float
  * format: 百分比
  * 等待（磁盘）所花费的时间占CPU时间的百分比。
* `system.core.iowait.ticks`
  * type: long
  * 等待（磁盘）所花费的CPU时间总量。
* `system.core.irq.pct`
  * type: scaled_float
  * format: 百分比
  * 处理硬件中断所花费的时间占CPU时间的百分比。
* `system.core.irq.ticks`
  * type: long
  * 处理硬件中断所花费的CPU时间总量。
* `system.core.softirq.pct`
  * type: scaled_float
  * format: 百分比
  * 处理软件中断所花费的时间占CPU时间的百分比。
* `system.core.softirq.ticks`
  * type: long
  * 处理软件中断所花费的CPU时间总量。
* `system.core.steal.pct`
  * type: scaled_float
  * format: 百分比
  * 当管理程序（[hypervisor](https://zh.wikipedia.org/wiki/Hypervisor)，事实上就是VMM）在服务其他处理器时，虚拟CPU在非自愿等待中花费的时间占CPU时间的百分比。仅在Unix系统中可用。
* `system.core.steal.ticks`
  * type: long
  * 当管理程序在服务其他处理器时，虚拟CPU在非自愿等待中花费的CPU时间总量。仅在Unix系统中可用。

### `cpu`指标

`cpu`类别（也就是`system.cpu.xxx`）中包含了本地的CPU统计信息。（我猜测上一个指的是某一个CPU的统计数据；这一个是所有CPU的整体统计数据，所以百分比可能会超过100%。）

* `system.cpu.cores`
  * type: long
  * 主机上的CPU核心数量。那些未经规格化的百分比数值的最大值为100% * 核心数量。而经过规格化的百分比数值已将此值考虑在内，最大值为100%。（我猜这些“经过标准化”指的是其他的数值，那些format是百分比的值。但是我们怎么知道哪些数值是经过标准化的呢？）
* `system.cpu.user.pct`
  * type: scaled_float
  * format: 百分比
  * 在用户空间中花费的时间占CPU时间的百分比。在多核系统上，这个数值可能会超过100%。例如，如果3个核心使用率均为60%，则`system.cpu.user.pct`的值将为180%。
* `system.cpu.system.pct`
  * type: scaled_float
  * format: 百分比
  * 在内核空间中花费的时间占CPU时间的百分比。
* `system.cpu.nice.pct`
  * type: scaled_float
  * format: 百分比
  * 在低优先级进程上花费的时间占CPU时间的百分比。
* `system.cpu.idle.pct`
  * type: scaled_float
  * format: 百分比
  * 空闲时间占CPU时间的百分比。
* `system.cpu.iowait.pct`
  * type: scaled_float
  * format: 百分比
  * 等待（磁盘）所花费的时间占CPU时间的百分比。
* `system.cpu.irq.pct`
  * type: scaled_float
  * format: 百分比
  * 处理硬件中断所花费的时间占CPU时间的百分比。
* `system.cpu.softirq.pct`
  * type: scaled_float
  * format: 百分比
  * 处理软件中断所花费的时间占CPU时间的百分比。
* `system.cpu.steal.pct`
  * type: scaled_float
  * format: 百分比
  * 当管理程序在服务其他处理器时，虚拟CPU在非自愿等待中花费的时间占CPU时间的百分比。仅在Unix系统中可用。
* `system.cpu.total.pct`
  * type: scaled_float
  * format: 百分比
  * 非空闲CPU时间的百分比。
* `system.cpu.user.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 在用户空间中花费的时间占CPU时间的百分比。（规格化）
* `system.cpu.system.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 在内核空间中花费的时间占CPU时间的百分比。（规格化）
* `system.cpu.nice.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 在低优先级进程上花费的时间占CPU时间的百分比。（规格化）
* `system.cpu.idle.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 空闲时间占CPU时间的百分比。（规格化）
* `system.cpu.iowait.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 等待（磁盘）所花费的时间占CPU时间的百分比。（规格化）
* `system.cpu.irq.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 处理硬件中断所花费的时间占CPU时间的百分比。（规格化）
* `system.cpu.softirq.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 处理软件中断所花费的时间占CPU时间的百分比。（规格化）
* `system.cpu.steal.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 当管理程序在服务其他处理器时，虚拟CPU在非自愿等待中花费的时间占CPU时间的百分比。仅在Unix系统中可用。
* `system.cpu.total.norm.pct`
  * type: scaled_float
  * format: 百分比
  * 非空闲时间占CPU时间的百分比。
* `system.cpu.total.value`
  * type: long
  * 启动进程后CPU的使用率。（什么进程？Metricbeat自己吗？）
* `system.cpu.user.ticks`
  * type: long
  * 在用户空间中花费的CPU时间量。
* `system.cpu.system.ticks`
  * type: long
  * 在内核空间中花费的CPU时间量。
* `system.cpu.nice.ticks`
  * type: long
  * 在低优先级进程上花费的CPU时间量。
* `system.cpu.idle.ticks`
  * type: long
  * 空闲CPU时间的总量。
* `system.cpu.iowait.ticks`
  * type: long
  * 等待（磁盘）所花费的时间占CPU时间的百分比。
* `system.cpu.irq.ticks`
  * type: long
  * 处理硬件中断所花费的CPU时间总量。
* `system.cpu.softirq.ticks`
  * type: long
  * 处理软件中断所花费的CPU时间总量。
* `system.cpu.steal.ticks`
  * type: long
  * 当管理程序在服务其他处理器时，虚拟CPU在非自愿等待中花费的CPU时间总量。仅在Unix系统中可用。

### `diskio`指标

`disk`类别中包含从操作系统收集的磁盘IO指标。

* `system.diskio.name`
  * type: keyword
  * 例: sda1
  * 磁盘名称。
* `system.diskio.serial_number`
  * type: keyword
  * 磁盘的序列号。可能不是所有操作系统都提供此信息。
* `system.diskio.read.count`
  * type: long
  * 已成功完成的读操作总数。
* `system.diskio.write.count`
  * type: long
  * 已成功完成的写操作总数。
* system.diskio.read.bytes
type: long

format: bytes

The total number of bytes read successfully. On Linux this is the number of sectors read multiplied by an assumed sector size of 512.

system.diskio.write.bytes
type: long

format: bytes

The total number of bytes written successfully. On Linux this is the number of sectors written multiplied by an assumed sector size of 512.

system.diskio.read.time
type: long

The total number of milliseconds spent by all reads.

system.diskio.write.time
type: long

The total number of milliseconds spent by all writes.

system.diskio.io.time
type: long

The total number of of milliseconds spent doing I/Os.

system.diskio.iostat.read.request.merges_per_sec
type: float

The number of read requests merged per second that were queued to the device.

system.diskio.iostat.write.request.merges_per_sec
type: float

The number of write requests merged per second that were queued to the device.

system.diskio.iostat.read.request.per_sec
type: float

The number of read requests that were issued to the device per second

system.diskio.iostat.write.request.per_sec
type: float

The number of write requests that were issued to the device per second

system.diskio.iostat.read.per_sec.bytes
type: float

format: bytes

The number of Bytes read from the device per second.

system.diskio.iostat.write.per_sec.bytes
type: float

format: bytes

The number of Bytes write from the device per second.

system.diskio.iostat.request.avg_size
type: float

The average size (in sectors) of the requests that were issued to the device.

system.diskio.iostat.queue.avg_size
type: float

The average queue length of the requests that were issued to the device.

system.diskio.iostat.await
type: float

The average time spent for requests issued to the device to be served.

system.diskio.iostat.service_time
type: float

The average service time (in milliseconds) for I/O requests that were issued to the device.

system.diskio.iostat.busy
type: float

Percentage of CPU time during which I/O requests were issued to the device (bandwidth utilization for the device). Device saturation occurs when this value is close to 100%.
