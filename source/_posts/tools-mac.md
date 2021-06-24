---
title: 效率-Mac技巧
date: 2019-01-07 10:17:08
tags:
- tool
- mac
categories:
- 其他
---

# 命令行方式打开文件

执行两条命令即可，以“Sublime Text” APP为例： 

```bash
# 设置命令 sublime
$ echo "open -a 'Sublime Text' \$*" > /usr/local/bin/sublime
$ chmod +x /usr/local/bin/sublime
# 用 sublime 打开文件
$ sublime readme
```

其实质是使用`open`命令以某个APP打开文件。 

> 注意：应用名称"Sublime Text"取自"Applications/"文件夹下的对应的应用名称，无后缀名。

