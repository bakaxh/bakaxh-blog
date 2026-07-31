---
title: 给 Xournal++ 配数位板侧键？我写了个小插件解决这个问题
description: ToggleActions 是一个 Xournal++ 插件，可以通过快捷键在工具之间切换，并自动恢复之前使用的工具。
author: 小狐
pubDate: 2026-07-26
updatedDate: 2024-07-27
tags: [折腾,Xournal++,插件,数位板,手写,效率工具]
categories: [项目]
featured: false
toc: true
---

平时记笔记、批 PDF 的时候，我一直在用 Xournal++。

这软件其实挺舒服的：开源、轻量，手写延迟也不错，对压感支持也比较完善。对于需要大量手写的人来说，它基本已经够用了。

不过有一个问题一直让我很难受：

**数位板上的侧键，在 Xournal++ 里并不好用。**

# 数位板侧键的问题
我的数位板有两个侧键，正常情况下我希望它们能分别对应：

- 切换到画笔；
- 切换到橡皮。

这样写东西的时候不用去点工具栏，也不用频繁拿鼠标。

但实际情况比较尴尬。

有时候驱动里明明设置好了按键，Xournal++ 却没有反应；有时候能识别，但行为又不稳定。折腾了几次驱动、按键映射之后，我发现问题并不完全出在数位板或者驱动上。

Xournal++ 本身对于“数位板按键直接绑定某个工具”这件事，并没有提供特别方便的接口。

最后导致的结果就是：

写着写着需要换工具，只能停下来：

> 放下笔 → 找鼠标 → 点工具栏 → 再继续写。

次数多了以后，确实挺影响体验。

# 所以我写了 ToggleActions
既然数位板侧键不好直接控制工具，那就换个思路。

我写了一个 Xournal++ 插件：

**ToggleActions**

它做的事情很简单：

给工具绑定一个快捷键，并让这个快捷键支持“切换 / 返回”。

比如给画笔设置：

```
Ctrl + Shift + Alt + P
```

那么：

- 第一次按下：切换到画笔；
- 再按一次：恢复之前使用的工具。
这样数位板侧键只需要负责发送一个普通快捷键，工具切换逻辑交给插件处理。

原本不好控制的“数位板按钮”，就变成了一个普通键盘输入。

# 其它功能
除了快捷键切换，插件还会在 Xournal++ 的插件菜单里生成一个 ToggleActions 菜单。

里面会列出支持切换的工具，可以直接点击使用。

快捷键配置保存在插件目录下自动生成的：

```settings.lua```
里面类似这样：

```lua
ACTION_TOOL_PEN = "<Ctrl><Shift><Alt>p"
ACTION_TOOL_ERASER = "<Ctrl><Shift><Alt>e"
```

如果某个快捷键不想使用，直接改成：

```lua
ACTION_TOOL_PEN = ""
```

然后重新启动 Xournal++ 即可。

安装方法
安装很简单。

1. 将 `ToggleActions` 文件夹复制到 Xournal++ 插件目录：
Windows：

```
%APPDATA%\xournalpp/plugins/
```
Linux：

```
~/.xournalpp/plugins/
```
macOS：

```
~/Library/Application Support/xournalpp/plugins/
```

2. 打开 Xournal++：
```
插件管理器（Plugin Manager）
```
启用：

```
ToggleActions
```

3. 重启 Xournal++。
Linux 用户也可以直接：

```bash
cp -r ToggleActions ~/.xournalpp/plugins/
```
项目地址：

👉 [https://github.com/yourusername/ToggleActions](https://github.com/yourusername/ToggleActions)

# 最后
如果你也在用：

- Xournal++
- 数位板
- 手写笔记
并且遇到过侧键不好用的问题，可以试试看。

当然，即使没有数位板，它也可以作为一个简单的快捷键工具，用来快速切换画笔、橡皮、选择工具等。

后面如果有需求，也许会继续加一些功能，比如长按触发、多工具循环切换之类。

如果有其它想法，也欢迎在仓库里提 issue。
