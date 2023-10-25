---
title: VS Code + Python 开发 QGIS 插件环境配置指南
date: 2023-10-22 18:07:22
tags: 
- VS Code
- Python
- QGIS
categories: 
- GIS 
---

# 写在前面
{% note info %}
整篇教程 For Windows，好像Mac没有这么多事。前一段开发插件的时候给同组的同学写的配置文档，恰好发现中文互联网上相关的内容比较少，也不太系统，做一个系统的整理。
{% endnote %}

# QGIS 官方Plugin 文档
## Official Version (What's Next For Dialog Plugins)
1. If resources.py is not present in your plugin directory, compile the resources file using pyrcc5 (simply use `pb_tool` or `make` if you have automake)
2. Optionally, test the generated sources using `make test` (or run tests from your IDE)
3. Copy the entire directory containing your new plugin to the QGIS plugin directory (see Notes below)
4. Test the plugin by enabling it in the QGIS plugin manager
5. Customize it by editing the implementation file `viewshed_analysis.py`
6. Create your own custom icon, replacing the default `icon.png`
Modify your user interface by opening `viewshed_analysis_dialog_base.ui` in Qt Designer


# QGIS 插件开发的环境配置
## Environment Configuration

最基础的版本：[知乎网站：从0开始开发QGIS插件](https://zhuanlan.zhihu.com/p/344965380)

用于自动创建 GUI 界面的 QGIS 插件：[Building a Processing Plugin](https://www.qgistutorials.com/en/docs/3/processing_python_plugin.html) ([In Chinese](https://www.osgeo.cn/qgis-tutorial/docs/3/processing_python_plugin.html))

> 上面这个工具插件有效解决了插件GUI的问题，可以直接用与QGIS风格非常统一的GUI进行开发，这个工具只适用于算法处理相关的插件

## 如何把 Tool 导入 QGIS 中

1. 在 Debug 之前首先我们需要将开发好的插件加入到 QGIS 的环境中，这里提供两种方式：

   1. 直接把用 Plugin Builder 生成好的整个插件目录拷到`C:\Users\---\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins`

   2. 这里更推荐用 symbolic link 的方法直接创建软链接 

      ```powershell
      mklink /D "C:\Users\---\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\your_plugin" "${your_plugin_workspace}\your_plugin"
      ```

    {% note warning %}
    WARNING: There is a huge risk though of losing your code if you uninstall the plugin by accident.
    {% endnote %}

2. 重启 QGIS, Menu Plugins -> Manage and Install Plugins

3. 把 viewshed analysis 打钩

4. 在 Plugins 菜单或者 Toolbox 中可以找到对应的插件

# 关于使用 VS Code 进行开发的问题
{% note warning %}
用 VS Code 开发是一件非常 tricky 的事情，这可能需要非常长的配置时间
{% endnote %}

**再次声明，好像只有Windows有这么多麻烦的问题**

另外，根据官方网站的推荐，最好使用 long-term-release 的版本进行开发 非 long-term-release 的版本的路径可能与下文中的描述略有区别，主要是各种路径和文件名中 qgis-ltr 和 qgis 的区别

## 如何在 Integrated terminal 运行

有几种方案解决import qgis相关的问题：

- 根据以下文件自己配置 settings.json，这些文件基本可以在 QGIS 的根目录下找到
```bash
%OSGEO4W_ROOT%/bin/python-qgis-ltr.bat    
%OSGEO4W_ROOT%/bin/o4w_env.bat    
%OSGEO4W_ROOT%/bin/o4w_env.bat
```
- 在 Integrated terminal 中运行 `%OSGEO4W_ROOT%/bin/python-qgis-ltr-env.bat `后再运行python程序

  ```powershell
  rem python-qgis-ltr-env.bat    
  @echo off    call "%~dp0\o4w_env.bat"   
  @echo off    path %OSGEO4W_ROOT%\apps\qgis-ltr\bin;%PATH%    
  set QGIS_PREFIX_PATH=%OSGEO4W_ROOT:\=/%/apps/qgis-ltr   
  set GDAL_FILENAME_IS_UTF8=YES    
  rem Set VSI cache to be used as buffer, see #6448    
  set VSI_CACHE=TRUE    
  set VSI_CACHE_SIZE=1000000    
  set QT_PLUGIN_PATH=%OSGEO4W_ROOT%\apps\qgis-ltr\qtplugins;%OSGEO4W_ROOT%\apps\qt5\plugins    
  set PYTHONPATH=%OSGEO4W_ROOT%\apps\qgis-ltr\python;%PYTHONPATH%
  ```

- Thanks to https://gist.github.com/thbaumann/73c873d4c49d8c1add8dc97359cebabe，有一个更简单的方法可以直接将.bat作为 vscode 的解释器 (在 vscode 的设置中无法将非.exe的文件设为解释器)
  > The bread and butter of this configuration is "python.pythonPath": "C:/OSGeo4W64/bin/python-qgis.bat", which sets all the correct paths and bindings for us. I don't really know why it works to refer a .bat file as python interpreter, but apparently it's fine and fixes alot of linting issues.

```json
// settings.json    
{ 
        // 第一个选项貌似是过时的
        "python.pythonPath":" %OSGEO4W_ROOT%\\bin\\python-qgis-ltr.bat",
        "python.defaultInterpreterPath": "%OSGEO4W_ROOT%\\bin\\python-qgis-ltr.bat"
}
```

还有一个 tricky 的点是相对路径引用的问题：
{% note info %}
这段东西是针对 dialog 的那种自己写 GUI 的 plugin 的，processing 系列的插件不太受到这个困扰
{% endnote %}

```python
# This works for integragted terminal
# Initialize Qt resources from file resources.py
from resources import *
# Import the code for the dialog
from visibility_analysis_dialog import VisibilityAnalysisDialog
# --------------------------------------
# This works for qgis 
# Initialize Qt resources from file resources.py
from .resources import *
# Import the code for the dialog
from .visibility_analysis_dialog import VisibilityAnalysisDialog
```
## Debugging in VS Code

> Thanks to
> https://gispofinland.medium.com/cooking-with-gispo-qgis-plugin-development-in-vs-code-19f95efb1977
> https://gist.github.com/thbaumann/73c873d4c49d8c1add8dc97359cebabe

利用 VS Code 对 QGIS 插件进行开发需要利用 QGIS 中的 Debugvs 插件，这个插件会在本地的 5678 端口创建一个 debug 的传输通道，让 VS Code 的 debug launch 可以进行远程的 attach debugging。

在安装这个插件之前，我们需要在 **QGIS 对应的 python 目录下**安装 ptvsd

```powershell
pip install ptvsd
```

{% note info %}
这里建议一定在安装之前检查一下当前 Python 的版本是否正确，建议直接打开 OSGeo4W shell 进行配置（通常在`%OSGeo4w_ROOT\OSGeo4W.bat`）
之后直接在 QGIS 中安装 Debugvs 即可
{% endnote %}

我们还需要额外配置一下 VS Code 的 `launch.json`

```json
{
    "name": "Python: Remote Attach",
    "type": "python",
    "request": "attach",
    "port": 5678,
    "host": "localhost",
    "pathMappings": [
        {
            "localRoot": "${workspaceFolder}/your_plugin", // path to your plugin where you are developing
            "remoteRoot": "C:\\Users\\---\\AppData\\Roaming\\QGIS\\QGIS3\\profiles\\default\\python\\plugins\\your_plugin" // path to where the QGIS plugin folder lives 
        }
    ]
},
```

安装好之后在 QGIS 中点击 `Plugins -> Enable Debug for Visual Studio -> Enable Debug for Visual Studio` 即可打开 Debug Port (localhost:5678)，随后在 VS Code 中直接运行 Debug 即可

每一次修改之后都要用 `Plugin Reloader` reload 一次
