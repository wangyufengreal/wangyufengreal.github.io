---
title: (3) Python 工程化之路 - Dev Containers
date: 2025-08-22 15:30:00 +0800
categories: [技术]
tags: [Python]
---


## Why Dev Containers?

在软件开发领域, 有一句常常听到的充满挫败感的话, 就是"在我的电脑上都是好的啊", VSCode 的 `Dev Containers` 扩展就是一个为了解决这个问题而生的工具. 通过对 `devcontainer.json` 和 `Dockerfile` 的配置, 就可以创建一个完全一致的开发环境.

## Dev Containers 适用的场景

Dev Containers 听起来很美妙, 但是也并非适用于所有的场景. 对于合适的场景, 它是利器, 对于不那么合适的场景, 为了用而用, 就会徒增困扰. 因此理解使用的场景很重要.

适合的场景:

* 团队协作, 如开发人员有人用 Windows, 有人用 Linux, 有人用 macOS, 用 Dev Containers 就可以创建一个完全一致的开发环境.
* 复杂的依赖管理, 项目依赖于数量庞大的特定版本的包, 对比于在每个人在本机上重复执行配置, 使用 Dev Containers 一键让依赖就位非常便利.
* 微服务开发, 开发多个相互依赖的微服务时, Dev Container 可以轻松地为每个服务维护独立的开发环境, 又能通过 Docker 的网络功能轻松地将他们连接起来进行联调.
* 安全沙盒: 对于包含潜在风险的代码, 可以隔离运行, 避免对主机造成影响.

不适合的场景:

* GUI 开发: 对于原生GUI开发, Windows(WPF/WinForms), macOS(Cocoa)等, UI框架高度依赖系统的组件, 虽然技术上可以实现(转发绘图指令), 但是从复杂度和效率上考虑, 不推荐使用. 但是对于 Electron 等 Web GUI 开发, Dev Container 是很好的选择, 因为在这种场景下, 依赖管理变得更重要, 而且通过端口转发可以很方便地调试网页.
* 需要直接访问硬件的开发: 如编写设备驱动, 或需要与 USB/串口等打交道的场景. 容器的设计目的就是隔绝底层硬件, 因此这么做有点本末倒置.
* 内核开发: 高度依赖内核本身, 那么容器是不适合的.
* 非常简单的项目: 如果项目非常简单, 那么 Dev Container 可能会显得过于复杂.

## 开始前的准备

* 安装 VSCode, 作为我们的操作界面和编辑器.
* 安装 Docker Desktop, 作为我们的容器引擎.
* 安装 VSCode 的 Dev Containers 扩展.

## 配置 devcontainer.json 和 Dockerfile

* 按下 `Ctrl + Shift + P`, 然后输入并选择 `Dev Containers: Add Dev Container Configuration Files...`.
* 选择 `Python 3`, 然后选择一个Python版本, 这里不用太过纠结, 具体镜像后面可以在 `Dockerfile` 中自定义.
* 提示 `Select features to install` 时, 可以选择相关工具, 如 `Ruff`, `Mypy`等.
* 完成上述步骤, VSCode 会自动创建 `devcontainer.json`.
* 修改 `devcontainer.json` 为如下内容:

```bash
{
    # 名称
    "name": "FastAPI Lab",
    # 构建方式选用指定 Dockerfile的方式
    "build": {"dockerfile": "Dockerfile"},
    # 这里的features在初始的时候可以选择, 后续也可以添加
    "features": {
        "ghcr.io/devcontainers-extra/features/mypy:2": {},
        "ghcr.io/devcontainers-extra/features/ruff:1": {}
    },
    # 向外暴露端口, 这里是以FastAPI应用为例, 我们暴露8000端口
    "forwardPorts": [8000],
    # 创建虚拟环境, 并安装依赖(包含dev依赖)
    "postCreateCommand": "uv venv && uv pip install -e .[dev]",

    # VSCode的配置, 包括指定python解释器, 安装并配置扩展(如激活ruff提供的自动格式化功能). 
    "customizations": {
        "vscode": {
            "settings": {
                "python.defaultInterpreterPath": "${containerWorkspaceFolder}/.venv/bin/python",
                "[python]": {
                    "editor.defaultFormatter": "charliermarsh.ruff",
                    "editor.formatOnSave": true
                }
            },
            "extensions": [
                "ms-python.python",
                "charliermarsh.ruff"
            ]
        }
    }
}
```
* 添加 `Dockerfile`:

```bash
# 指定Python版本
ARG PYTHON_VERSION=3.11
FROM python:${PYTHON_VERSION}

# 安装 pipx, 用来安装全局功能
RUN python3 -m pip install --no-cache-dir pipx

# 安装全局功能
ARG PIPX_PACKAGES="ruff"
RUN pipx install --pip-args='--no-cache-dir --force-reinstall' -q ${PIPX_PACKAGES}

# 安装uv包管理工具
RUN pip install uv
```

## 定义 pyproject.toml

```toml
[project]
name = "fastapi-lab"
version = "0.1.0"
dependencies = [
    "fastapi",
    "uvicorn[standard]" # uvicorn是运行FastAPI的ASGI服务器
]

[project.optional-dependencies]
dev = [
    "ruff"
]
```

## 编写FastAPI服务器

* 我们采用经典的 `src-layout` 结构.

```bash
/fastapi-container-lab
├── .devcontainer/
├── src/
│   └── fastapi_lab/
│       ├── __init__.py  (空文件)
│       └── main.py
└── pyproject.toml
```

* 编写一个最简单的FastAPI服务器.

```python
# src/fastapi_lab/main.py
from fastapi import FastAPI
import platform

app = FastAPI()

@app.get("/")
def read_root():
    """
    根路径，返回欢迎信息。
    """
    return {"message": "Hello from inside the FastAPI Dev Container!"}

@app.get("/system-info")
def get_system_info():
    """
    返回容器的操作系统信息。
    """
    return {
        "system": platform.system(), # 将会显示 "Linux"
        "release": platform.release()
    }
```

## 构建 Dev Containers 运行环境

* 按下 `Ctrl + Shift + P`, 然后输入并选择 `Dev Containers: Rebuild and Reopen in Container`.

* 过程中出现任何构建问题, 可以在 `Dev Container` 的输出窗口中查看详细信息.

## 运行 FastAPI 服务器

* 构建完成以后, 会默认激活虚拟环境, 我们运行以下命令:

```bash
uvicorn src.fastapi_lab.main:app --host 0.0.0.0 --port 8000 --reload
```

* 然后在宿主机的浏览器上, 我们输入 `http://localhost:8000`, 就可以看到json响应 `{"message":"Hello from inside the FastAPI Dev Container!"}`.

## 后记

* 从此以后, `.devcontainer` 目录就成了环境本身, 结合 `pyproject.toml` 就能够完美实现Python应用的开发环境的可复现性.
* 本文章只是一个最基础的演示, 用来说明 Dev Container 的基础使用.
