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

也可以到官网选择手动下载安装

[https://packagecontrol.io/](https://packagecontrol.io/)



## 安装插件

使用 **command + shift + p**, 输入 **install,** 等待几秒钟后，在新弹出的输入框里，输入你想要安装的插件名称

### ConvertToUTF8

通过本插件，可以编辑并保存目前编码不被 Sublime Text 支持的文件，特别是中日韩用户使用的 GB2312，GBK，BIG5，EUC-KR，EUC-JP 等。



### Alignment

代码对齐，如写几个变量，选中这几行，Ctrl+Alt+A，哇，齐了。



### Ctags

函数跳转，我的电脑上是Alt+点击 函数名称，会跳转到相应的函数



### [Trailing spaces](https://github.com/SublimeText/TrailingSpaces)

功能：检测并一键去除代码中多余的空格

简介：还在纠结代码中有多余的空格而显得代码不规范? 或是有处女座情节? 次插件帮你实现发现多余空格、一键删除空格、保存时自动删除多余空格，让你的代码规范清爽起来

使用：安装插件并重启，即可自动提示多余空格。一键删除多余空格：CTRL+SHITF+T（需配置），更多配置请点击标题。快捷键配置：在Preferences / Key Bindings – User加上代码（数组内）

`{ "keys": ["ctrl+shift+t"], "command": "delete_trailing_spaces" }`



### [FileDiffs](https://github.com/colinta/SublimeFileDiffs)

功能：强大的比较代码不同工具

简介：比较当前文件与选中的代码、剪切板中代码、另一文件、未保存文件之间的差别。可配置为显示差别在外部比较工具，精确到行。

使用：右键标签页，出现FileDiffs Menu或者Diff with Tab…选择对应文件比较即可



### [AutoFileName](https://sublime.wbond.net/packages/AutoFileName)

功能：快捷输入文件名

简介：自动完成文件名的输入，如图片选取

使用：输入”/”即可看到相对于本项目文件夹的其他文件



### [AllAutocomplete](https://github.com/alienhard/SublimeAllAutocomplete)

Sublime Tex中的经典自动补全，只适用于当前文件。AllAutocomplete在当前窗口的所有打开文件中搜索可以大大简化开发过程。还有CodeIntel，具体化了IDE的功能，并为若干语言带来了“代码智能”，这些语言包括：JavaScript，Mason，XBL，XUL，RHTML，SCSS，Python，HTML，Ruby，Python3，XML，Sass，XSLT，Django，HTML5，Perl，CSS，Twig，Less，Smarty，Node.js，Tcl，TemplateToolkit，PHP。



### 调整当前语言

使用 `command + shift + p`, 输入当前使用的编程语言, sublime 会自动转换相应的语法



## 快捷键使用

| 快捷键组合          | 功能                                |
| ------------------- | ----------------------------------- |
| shift + cmd + p     | 打开命令面板                        |
| control +          | 控制台                              |
| cmd + n           | 新建标签                            |
| cmd + 数字        | 标签切换                            |
| cmd + option + 2  | 分成两屏                            |
| control + 数字    | 分屏时移动到不同的屏幕              |
| cmd + delelte     | 删除光标前所有字符, 貌似是Mac快捷键 |
| cmd + f           | 查找                                |
| option + cmd + f  | 查找替换                            |
| cmd + t           | 文件跳转                            |
| control + g       | 行跳转, 类似vim中的num + gg         |
| cmd + r           | 函数跳转                            |
| cmd + /           | 给选中行添加或去掉注释              |
| cmd + [或 cmd + ] | 智能行缩进                          |
| cmd + k + b       | 开关侧边栏                          |

