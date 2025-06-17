---
layout: post
title: Visual Studio.md
categories: [软件]
description: 
keywords: Visual Studio.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Visual Studio

## 通用配置

### 解决方案多目录多项目配置

一个解决方案好多个项目，网站、动态库、测试等，每个项目放在不同的文件夹下。



#### 创建空白解决方案

首先创建一个【空白解决方案】。

![image-20240118092143648](https://www.xubighead.top/api/oss/img/LpymbaWO.png)



#### 新建解决方案文件夹

然后在【解决方案】上右键菜单中选择【添加】> 【新建解决方案文件夹】，**新建解决方案文件夹不会创建对应的真实目录，需手工在解决方案目录下创建。**

![image-20240118092412521](https://www.xubighead.top/api/oss/img/LpypTLZg.png)



#### 新建项目

最后在【解决方案文件夹】上右键菜单选择【添加】> 【新建项目】来创建不同的项目。

![image-20240118092529432](https://www.xubighead.top/api/oss/img/LpyrNero.png)



### 跨项目引用命名空间

通过右键【项目】 > 【添加】 > 【项目引用】，勾选想要引用的项目即可。

<img src="https://www.xubighead.top/api/oss/img/LpysOXjs.png" alt="image-20240118092835579"  />



完成上述操作步骤后，Visual Studio会自动在项目的.csproj配置文件中添加引用配置：

```xml
  <ItemGroup>
    <ProjectReference Include="..\G.Basement\G.Basement.csproj" />
  </ItemGroup>
```



### 安装Nuget包

通过右键【项目】 > 【管理NugGet程序包】使用NuGet 包管理器安装。

![image-20240123090716450](https://www.xubighead.top/api/oss/img/MIuEhTdI.png)



## 快捷键

![Printable cheatsheet for keyboard shortcuts.](https://www.xubighead.top/api/oss/img/NDQAnm2i.png)



| 描述              | 快捷键           | 描述             | 快捷键               |
| ----------------- | ---------------- | ---------------- | -------------------- |
| 删除行            | Ctrl + Shift + L | 重命名           | Ctrl + R,Ctrl + R    |
| 删除无引用的using | Ctrl+R,G         | 批量替换         | Ctrl+H               |
| 查看所有引用      | Shift+F12        | 新建类           | Ctrl+Shift+A         |
| 格式化代码        | Ctrl+K,Ctrl+D    | 在当前行插入空行 | Ctrl+Enter           |
|                   |                  | Shift+Enter      | 在当前行下方插入空行 |
|                   |                  |                  |                      |
|                   |                  |                  |                      |



### 参考资料

- [Visual Studio 中的键盘快捷方式](https://learn.microsoft.com/zh-cn/visualstudio/ide/default-keyboard-shortcuts-in-visual-studio?view=vs-2022)



## 扩展

Visual Studio可以通过【扩展】> 【管理扩展】来安装扩展工具。



### Visual-Studio-Translator

Visual-Studio-Translator翻译插件可以通过右键触发，翻译出选中的内容。支持切换不同的翻译引擎如Google、Bing等。

![image-20240318132711386](https://www.xubighead.top/api/oss/img/RqiYxZTc.png)