---
layout: post
title: "Mac 下 SublimeText3 使用"
categories: Mac
description:
keywords:
---

* content
{:toc}


## 修改默认配置

使用 command + , 打开个人配置, 其中 User Settings 是个人配置, 优先级高于全局配置, 我的配置如下

```
{
    "font_face": "JetBrains Mono", 
    "font_size": 13, // 设置字体大小, 我比较喜欢大一点的字体
    "ignored_packages":  // 设置忽略文件类型, 第二个是默认忽略的, 第一个markdown文件我使用另一种文件打开,
    [
        "Markdown",
        "Vintage"
    ],
    "create_window_at_startup": false, // 取消启动时,自动打开新窗口的设置, 这个设置很恶心, 每次启动后会自动生成一个空白窗口
    "open_files_in_new_window": false, // 取消打开文件时会新生成一个窗口, 默认设置每次打开一个项目会重新生成一个窗口
    "highlight_line": true, // 高亮当前编辑行, 其实高亮的不明显
    "highlight_modified_tabs": true, // 设置文件修改时, 标签高亮提示, 这样可以提示保存
    "show_encoding": true, // 在窗口右下角显示打开文件的编码
    "translate_tabs_to_spaces": true // 将tab键的形式改为四个空格
}
```

## 安装包管理工具

使用快捷键 control + `` 或者菜单栏选择`View > Show Console, 输入如下代码

```
import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

## 安装插件

使用 **command + shift + p**, 输入 **install,** 等待几秒钟后，在新弹出的输入框里，输入你想要安装的插件名称

### ConvertToUTF8

通过本插件，可以编辑并保存目前编码不被 Sublime Text 支持的文件，特别是中日韩用户使用的 GB2312，GBK，BIG5，EUC-KR，EUC-JP 等。

### 调整当前语言

使用 `command + shift + p`, 输入当前使用的编程语言, sublime 会自动转换相应的语法

## 快捷键使用

| 快捷键组合          | 功能                                |
| ------------------- | ----------------------------------- |
| `shift + cmd + p`   | 打开命令面板                        |
| **control + `**     | 控制台                              |
| `cmd + n`           | 新建标签                            |
| `cmd + 数字`        | 标签切换                            |
| `cmd + option + 2`  | 分成两屏                            |
| `control + 数字`    | 分屏时移动到不同的屏幕              |
| `cmd + delelte`     | 删除光标前所有字符, 貌似是Mac快捷键 |
| `cmd + f`           | 查找                                |
| `option + cmd + f`  | 查找替换                            |
| `cmd + t`           | 文件跳转                            |
| `control + g`       | 行跳转, 类似vim中的num + gg         |
| `cmd + r`           | 函数跳转                            |
| `cmd + /`           | 给选中行添加或去掉注释              |
| `cmd + [或 cmd + ]` | 智能行缩进                          |
| `cmd + k + b`       | 开关侧边栏                          |

