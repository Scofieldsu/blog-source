---
title: 装机App备忘
date: 2018-09-08 21:13:45
categories: 编程
tags: [工具,Python]
---
windows系统重装之后整理需要安装软件和环境：

1. 环境
  - python2.7  目前暂不用virtualenv管理多版本python环境，安装python2.7时勾选add to path省去配置环境变量过程。

  - java1.8    下载jdk安装后，配置环境变量
  ```
  JAVA_HOME = C:\Program Files\Java\jdk1.8.0_91  #实际路径
  PATH = %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
  CLASSPATH = .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
  ```
 - nodejs 安装时默认勾选add to path

 - git


2. 代码器
   - IDEA，Pycharm，notepad++，Atom
   ```
   atom 插件列表：
   主题：atom-monkai
   扩展：
   active-power-mode
   markdown-preview-plus
   platformio-ide-terminal
   simplified-chinese-menu
   file-icons
   edit-background
   ```


3. 其他工具
``` python
   - heidiSQL （数据库客户端）

   - mobaXterm （终端工具）

   - PDman （数据库建模工具）
```
---

4. mac app推荐
``` python
   - Termius  （ssh client）

   - Viper FTP （sftp client）

   - PAW （http client）

   - postico （postgresql client）

   - tickeys （键盘音效）

   - LiceCap （gif tool）

   - snip （截图）

   - pap.er (壁纸)
```

5. chrome插件推荐
``` python
   - Tampermonkey（脚本扩展）

   - fireshot （截图）

   - xPath finder， chropath  （xpath选择器）

   - octotree （github代码在线树状结构）

   - onetab （tab收集器）
```

6. 一些好玩的
 ```
  - 在线协作画板  https://witeboard.com/

 ```

----
