---
title: Bongo Cat 自动领取箱子、缩短开箱时间、点击倍率修改教程
description: 通过 dnSpy 修改 Assembly-CSharp.dll，实现自动开箱、自定义刷新时间和点击倍率，仅供技术交流。
author: 小狐
cover: https://bed.foxmoe.top/file/images/1785896673513_image.png
coverAlt: Bongo Cat 修改教程
pubDate: 2025-11-27
updatedDate: 2025-11-27
tags: [Bongo Cat, 修改, dnSpy, Unity]
categories: [逆向工程]
featured: false
toc: true
---

## 步骤

1. 根据你的系统版本（64位/32位）下载 [dnSpy](https://github.com/dnSpy/dnSpy/releases)。

2. 打开 `dnSpy.exe`，点击左上角 **文件 → 全部关闭**。

3. 点击左上角 **文件 → 打开**，选择 `Steam/steamapps/common/BongoCat/BongoCat_Data/Managed/Assembly-CSharp.dll`。  
   如果找不到目录，在 Steam 游戏库中右键 Bongo Cat → **管理 → 浏览本地文件**。

4. 在程序集资源管理器中选择 `Assembly-CSharp.dll` → `BongoCat`。  
   **修改前请关闭游戏**。

---

### 修改自动开箱

- 选择 `Shop` 类，在右侧编辑窗中找到 `TimerUpdate()` 方法。
- 右键该方法，选择 **编辑方法 (C#)...**。
- 在 `if (this._showChestPopup.Value && this._shopItem.CanBuy())` 内的最后一句添加：

```csharp
this._shopItem.Buy();
```

- 点击 **编译**。

![修改自动开箱](https://bed.foxmoe.top/file/images/1785896565464_image.png)

---

### 修改箱子刷新时间

- 选择 `Shop` 类，找到 `_stockRefreshTime` 属性。
- 右键该属性，选择 **编辑类 (C#)...**。
- 修改为想要的刷新时间（单位：秒）。  
  **不建议时间过短**，否则可能触发 Steam Error。

![修改箱子刷新时间](https://bed.foxmoe.top/file/images/1785896612593_image.png)

---

### 修改点击倍率

- 选择 `Pets` 类，找到 `AddPet(int value)` 方法。
- 右键该方法，选择 **编辑方法 (C#)...**。
- 添加倍率代码，例如：

```csharp
value = value * 1000;   // 放大 1000 倍
// 或 value += 666;
```
![修改点击倍率](https://bed.foxmoe.top/file/images/1785896458606_image.png)

---

### 保存

- 点击 **文件 → 保存模块** → **确定**。

重新打开游戏即可看到效果。  
---