---
title: (1) Python 工程化之路 - 日志何志
date: 2025-07-11 10:40:00 +0800
categories: [技术]
tags: [Python]
---

## Why Log?

在简单的场景下,  `print(...)` 能够满足调试信息打印的需求, 但是对比专业的日志系统, `print(...)` 方式有如下缺陷:

* **格式混乱**: 在调试过程中, 往往根据场景和数据的不同, 打印出来的内容五花八门, 思维负载沉重. 而许多基础信息如 `时间戳`, `来源模块`, `重要程度` 等, 需要反复在 `print(...)` 语句中设置, 格式控制相当繁琐.
* **默认无法按级别打印**: 一部分打印信息只需要在调试阶段输出, 当软件进入生产环境, 只能移除相关代码或增加判断条件, 增加了额外的工作量.
* **持久化困难**: 无法直接输出到文件, 需要自己实现相应操作.
* **...**

上面的问题, 在常规的日志系统中, 都得到了很好的解决, 这就是为什么工程化的软件, 一定要使用日志系统的原因, 本文章中我们以 `Python` 内置的 `logging` 模块为例, 因为日志模块的功能和使用, 皆是大同小异.


## 常规配置

我们直接使用代码的方式进行展示.

```python
# encoding:utf-8

import os
import sys
import logging
from logging.handlers import TimedRotatingFileHandler


# 日志格式定义
LOG_FORMAT = "%(asctime)s - %(levelname)s - %(threadName)s - %(name)s:%(lineno)d - %(message)s"
DATE_FORMAT = "%Y-%m-%d %H:%M:%S"


def setup_logging(log_level=logging.DEBUG):
    """配置全局日志记录器"""

    root_logger = logging.getLogger()
    root_logger.setLevel(log_level)
    
    # 清空原有日志记录器
    if root_logger.hasHandlers():
        root_logger.handlers.clear()

    # 创建控制台日志记录器
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(logging.Formatter(LOG_FORMAT, datefmt=DATE_FORMAT))
    
    # 创建文件日志记录器
    # * 自动创建日志文件夹
    if not os.path.exists('logs'):
        os.makedirs('logs')
    # 每天午夜创建新的日志文件，并保留最近7天的日志
    file_handler = TimedRotatingFileHandler(
        "logs/app.log", when="midnight", interval=1, backupCount=7, encoding='utf-8'
    )
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(logging.Formatter(LOG_FORMAT, datefmt=DATE_FORMAT))
    
    # 添加记录器
    root_logger.addHandler(console_handler)
    root_logger.addHandler(file_handler)
    
    logging.info("日志系统配置完成.")

```

这个实例可以满足绝大多数的日志需求, 下面我们来对它的配置进行剖析.

### 统一格式, 降低思维负载

`logging` 提供了非常多的格式配置项目, 具体可参考 [logging docs](https://docs.python.org/3/library/logging.html#formatter-objects), 不过我们如下的配置已经相当全能.

* `%(asctime)s`: 即 `DATE_FORMAT` 格式的时间, 用来判断该条日志输出的时间.
* `%(levelname)s`: 即 `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` 中的一个, 由日志输出时调用的方法确定, 如使用 `logger.info(...)`, 那 `levelname` 必然就是 `INFO`.
* `%(threadName)s`: 线程名称, 对于有多个子线程同时运行的应用, 这是相当关键且必须的,
* `%(name)s`: 我们使用 `logger = logging.getLogger(__name__)` 获取日志记录器实例的话, 当前 `*.py` 文件下的日志输出的 `name` 自然就是 `python` 文件的 `__name__)`.
* `%(lineno)d`: 和 `name` 配合使用, 快速定位日志发生的位置, 以便于结合源码进行分析.
* `%(message)s`： 就调用时填入的 `message` 信息.
* `...`: 按需要探究, 添加即可.

```python
LOG_FORMAT = "%(asctime)s - %(levelname)s - %(threadName)s - %(name)s:%(lineno)d - %(message)s"
DATE_FORMAT = "%Y-%m-%d %H:%M:%S"
```

### 根日志记录器是什么?

使用 `logging.getLogger()` 方式获取的日志记录器, 即是根日志记录器, 而在其它模块中, 使用 `logging.getLogger(__name__)` 获取的日志记录器, 默认会继承根日志记录器的配置, 这也是为什么我们会在入口文件如 `main.py` 中调用 `setup_logging()` 的原因.

### 怎样添加日志处理器?

为了防止根日志记录器在已有日志处理器的基础上, 重复添加处理器, 所以在创建日志处理器前, 对原有日志处理器进行了清理.

```python
if root_logger.hasHandlers():
    root_logger.handlers.clear()
```

配置显示在命令行的日志, 也即是我们在调试时可以一眼看到的日志. 为了防止大把依赖库的 `DEBUG` 级别的日志将命令行污染得一塌糊涂, 命令行日志处理器的优先级必须要设置为 `INFO` 级别.

```python
console_handler = logging.StreamHandler(sys.stdout)
console_handler.setLevel(logging.INFO)
console_handler.setFormatter(logging.Formatter(LOG_FORMAT, datefmt=DATE_FORMAT))
```

配置持久化于文件中的日志, 也就是在应对一些深层次的问题时需要看到的日志, 为了避免信息的遗漏, 文件日志处理器的优先级最好设置为 `DEBUG` 级别. 常规来说, 在配置处理器的同时, 还会对 `logs` 文件夹的存在性进行检查, 以及配置日志文件滚动的频率, 防止大量无用, 老旧的日志信息占据存储空间, 以及影响运行效率.

```python
if not os.path.exists('logs'):
    os.makedirs('logs')
file_handler = TimedRotatingFileHandler(
    "logs/app.log", when="midnight", interval=1, backupCount=7, encoding='utf-8'
)
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(LOG_FORMAT, datefmt=DATE_FORMAT))
```

最后将处理器添加到根日志记录器即可.

```python
root_logger.addHandler(console_handler)
root_logger.addHandler(file_handler)
```

## 怎样使用?

在任意的 `*.py` 文件中, 只需要 `logger = logging.getLogger(__name__)` 来引入日志记录器实例, 然后就可以愉快地玩耍了😜.

## The End

引入日志是为了长久的方便, 将必要的信息持久化到日志文件中, 能够帮助快速的查找问题. 从短期的目标出发, 实现很可能不需要日志系统的帮助. 但是从长期的观点看, 在基础设施中添加日志系统, 是非常重要的一步.
