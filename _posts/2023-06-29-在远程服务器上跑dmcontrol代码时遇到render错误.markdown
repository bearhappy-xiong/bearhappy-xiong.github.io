---
title: 在远程服务器上跑dmcontrol代码时遇到render错误
date: 2023-06-29 09:30:00 +0800
categories: [code]
tags: [practice]   # TAG names should always be lowercase
toc: true
comments: true
math: true
mermaid: true  # 图表生成工具
---

# 在远程服务器上跑dmcontrol代码时遇到render错误

报错内容如下:
```python
Traceback (most recent call last):
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/hydra/_internal/utils.py", line 211, in run_and_report
    return func()
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/hydra/_internal/utils.py", line 368, in <lambda>
    lambda: hydra.run(
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/hydra/_internal/hydra.py", line 110, in run
    _ = ret.return_value
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/hydra/core/utils.py", line 233, in return_value
    raise self._return_value
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/hydra/core/utils.py", line 160, in run_job
    ret.return_value = task_function(task_cfg)
  File "train.py", line 239, in main
    from train import Workspace as W
  File "/root/autodl-nas/srpa_drqv2/train.py", line 19, in <module>
    import dmc
  File "/root/autodl-nas/srpa_drqv2/dmc.py", line 10, in <module>
    from dm_control import manipulation, suite
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/manipulation/__init__.py", line 20, in <module>
    from dm_control import composer as _composer
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/composer/__init__.py", line 18, in <module>
    from dm_control.composer.arena import Arena
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/composer/arena.py", line 20, in <module>
    from dm_control import mjcf
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/mjcf/__init__.py", line 18, in <module>
    from dm_control.mjcf.attribute import Asset
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/mjcf/attribute.py", line 28, in <module>
    from dm_control.mujoco.wrapper import util
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/mujoco/__init__.py", line 18, in <module>
    from dm_control.mujoco.engine import action_spec
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/mujoco/engine.py", line 41, in <module>
    from dm_control import _render
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/_render/__init__.py", line 86, in <module>
    Renderer = import_func()
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/_render/__init__.py", line 36, in _import_egl
    from dm_control._render.pyopengl.egl_renderer import EGLContext
  File "/root/miniconda3/envs/drqv2/lib/python3.8/site-packages/dm_control/_render/pyopengl/egl_renderer.py", line 75, in <module>
    raise ImportError('Cannot initialize a headless EGL display.')
ImportError: Cannot initialize a headless EGL display.
```



在没有显示器的服务器上运行需要图形显示的代码可以通过使用虚拟显示（Virtual Display）来实现。虚拟显示可以模拟一个图形显示环境，让您的代码能够在无头服务器上正常运行。以下是一种常用的方法来设置虚拟显示：

1. 安装 Xvfb：Xvfb 是一个虚拟的 X11 显示服务器，可以在无头服务器上创建虚拟显示。您可以使用适合您的操作系统的软件包管理器来安装 Xvfb。例如，在 Ubuntu 上，您可以使用以下命令安装 Xvfb：

   ```
   sqlCopy codesudo apt-get update
   sudo apt-get install xvfb
   ```

2. 启动虚拟显示：在您的脚本运行之前，需要启动 Xvfb 并创建虚拟显示。可以使用以下命令：

   ```
   arduinoCopy code
   xvfb-run -s "-screen 0 1280x1024x24" python train.py
   ```

   这将在后台启动虚拟显示，并将您的脚本与虚拟显示关联起来。

3. 调整显示参数（可选）：如果您需要调整虚拟显示的分辨率、颜色深度或其他参数，可以在 `-screen` 参数后面的引号内修改相应的值。
