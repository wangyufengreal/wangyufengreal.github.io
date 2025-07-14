---
title: (2) Python 工程化之路 - RTFM?
date: 2025-07-12 11:25:00 +0800
categories: [技术]
tags: [Python]
---

## RTFM?

- `RTFM` 即 `Read The Fucking Manual`, 意指很多基础性问题的解答都可以在文档中找到, 无休止地不具备启发性的提问会被视作无理取闹, 这也正是 `Fucking` 的缘起🤣.

- 对于许多初创性质的企业, 开发团队常常陷入 **用户要求 -> 加班修改 -> 匆忙交付** 的怪圈, 根本无暇顾及注释和文档的完整性, 加之开发人员绝对数量上的缺乏, 使得代码库中除了少量且不合理的注释以外, 根本见不到文档的存在. 所以当未来的某一天, 原有的开发者/后来的开发者想要 `RTFM`, 居然陷入无语凝噎🫢的境地.

- 本篇文章作为本人痛定思痛后的产物, 在此对于代码文档的编制进行较为全面的思考.

## 为什么使用文档生成工具

- 相比较于直接编写 `Markdown` 格式的文档, 文档生成工具给我们提供了更多的便利, 本文章编写的目的就在于理解这些便利. 

- 其实就是一句话, 参照最佳实践, 社区都在使用文档生成工具, 那我们也使用, 相信群体智慧.

## Why Sphinx?

- `Numpy, Django` 等众多大型 `Python` 项目都在使用, 互联网上有相当全面的参考资料. 甚至可以说是因此, `LLM` 可以生成出更有效的回答, 更好地协助我们在文档方面的工作.

## 安装

```bash
# sphinx 是基础命令行工具
# sphinx_rtd_theme 是流行的 Read the Docs 主题
pip install sphinx sphinx_rtd_theme
```

## 初始化项目

- 以下操作默认均在 `docs` 文件夹下执行.

- 使用 `sphinx-quickstart` 初始化文档, 建议在 `Separate source and build directories (y/n)` 问题上选择 `y`, 即在 `source` 文件中存放文档的源文件, 而在 `build` 中存放生成的文档, 这样从结构上讲比较清晰一些.

```bash
mkdir docs
cd docs
sphinx-quickstart
```

- 在 `source/conf.py` 文件中, 插入以下代码, 用来指定项目的路径.

```python
import os
import sys
# 这里我们的项目文件夹在 docs 的上两级, 因此是 ../..
sys.path.insert(0, os.path.abspath('../..'))
```

- 在 `source/conf.py` 文件的 `extensions` 列表里添加以下插件.

```python
extensions = [
    'sphinx.ext.autodoc', # 自动从 docstrings 中提取文档.
    'sphinx.ext.napoleon', # 支持 Google 和 NumPy 风格的 docstrings.
    'sphinx.ext.viewcode', # 在文档中添加源代码链接.
]
```

- 在 `source/conf.py` 文件中设置主题为常用的 `Read The Docs`.

```python
html_theme = 'sphinx_rtd_theme'
```

- 在 `source/conf.py` 文件中对主题目录栏的行为进行约束.

```python
html_theme_options = {
    # 如果设置为 False，侧边栏目录树会始终展开所有层级，不会折叠
    'collapse_navigation': False,
    # 推荐设置为 True，这样在页面滚动时，侧边栏会固定在屏幕上
    'sticky_navigation': True,
    # -1 表示不限制导航深度，会显示所有层级
    'navigation_depth': -1,
}
```

## 基本文档框架

- 首先需要明确的是, 在本人尝试了自动生成文档结构的功能以后, 发现它严重依赖于项目原有文件结构, 否则生成的文档目录是相当混乱的, 且在修改的过程中会遇到很多异常(表现为在 `make html` 阶段的红字报错😭). 在一顿摸不着头脑的尝试之后, 我认为从零手动编写 `*.rst` 文档结构的方式更可控, 工作量的差异其实也并不大.

- `index.html` 基础根结构, 在 `toctree` 之下的内容, 如 `installation, quickstart, api/index`, 是 `*.rst` 的文件名.

```bash
.. xxx.

xxx 文档
========================================

xxx.


.. toctree::
   :maxdepth: 2
   :caption: 目录

   installation
   quickstart
   api/index

```

- 单页面文档结构, 适合如安装流程, 快速使用等一个页面就可以说明清楚的部分.

```bash
.. 这里写注释 ..

这里是首页标题 
==========

本页面文档介绍.

本页面子话题一
--------
* 正常 markdown 即可.
* ...
* ...
* ...

本页面子话题二
--------
* 正常 markdown 即可.
* ...
* ...
* ...
```

- 包含子级别的文档结构, 如 API 文档, 就需要新建一个文件夹, 在里面编写一个 `index.rst` 作为导航文件. 其它内容和单页面文档结构一致, 不再赘述.

```bash
.. 这里写注释 ..

这里是首页标题
==========

本页面文档介绍。

.. toctree::
   :maxdepth: 1

   common.modbus

```

## Docstings 编写示范

```python
class ModbusManager(QObject):
    """Modbus管理器类
    
    这个类用来统一管理一个基于 modbus-tcp 协议的连接的通信.
    它能以队列的方式依次处理来自外部的请求, 并以 Qt 信号的
    方式将结果发送出去.

    Attributes:
        finished: 运行结束信号. 当管理器停止工作时, 向外部发送此信号, 以安全地处理(关闭)线程.

            类型:
                Signal

        response_ready: 响应可用信号. 当管理器处理完一个请求的时候, 通过该信号将结果发送到外部.

            类型:
                Signal
            
            例子:
                manager.response_ready.connect(self.on_response_ready)
    """
    pass

    def __init__(self, host: str, port: int):
        pass

    @Slot()
    def run(self):
        """主循环

        负责不断的从请求队列中获取请求, 并将处理请求得到的响应, 通过信号发送出去.
        """
        pass

    def submit_request(self, request: ModbusRequest):
        """提交任务

        提交一个任务给管理器处理.

        Args:
            request: 任务
        """
        pass

    def stop(self):
        """停止任务

        通过修改标志位, 停止运行中的不断请求 modbus 服务器的任务.
        """
        pass
```

## The End

- 以上的方法, 其实已经足够用来管理起一套完善的文档了, 更细致的使用应当在实际使用过程中, 再行探索.

- 核心的问题在于, 真正地发挥文档的价值, 这是我们应当不断思考的问题, 不然文档写也白写😅.
