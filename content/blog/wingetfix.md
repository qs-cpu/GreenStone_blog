+++
authors = ["青石"]
title = "winget版本老旧解决方法"
description = ""
date = 2026-01-07
[taxonomies]
tags = ["win"]

+++


在使用UniGet的时候遇到了winget源下载失败，原因是winget版本太旧，这里给出更新winget的方法。

先问了一手AI，他最早给出的回答是通过微软商店更新App Installer或者输入如下命令
```bash
winget upgrade Microsoft.AppInstaller
```
但是命令的执行之后并没有进行更新，而微软商店更新又太麻烦了，那么有没有更简单快捷的方法呢？

在github中有一个winget-cli项目可以用来更新winget
[](https://github.com/microsoft/winget-cli)
进入网页后下载Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle 
接下来有一些坑，所以先列出正确的解决方法
下载之后双击该包会发现无法运行，这时要打开终端，输入以下命令
```powershell
Add-AppxPackage -Path <你的路径>\Microsoft.DesktopAppInstaller_*.msixbundle -ForceApplicationShutdown
```
然后问题就解决了。