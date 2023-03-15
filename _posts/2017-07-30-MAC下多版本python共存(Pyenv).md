---
layout: post
title: MAC 下多版本 python 共存(Pyenv)
categories: [Python]
description:
keywords:
---

* content
{:toc}

## 安装配置pyenv

* 经常遇到这样的情况：
  
    系统自带的Python是2.x, 自己需要Python 3.x, 测试尝鲜；
    系统是2.6.x, 开发环境是2.7.x
    由于Mac机器系统保护的原因, 默认的Python中无法对PIP一些包升级, 需要组建新的Python环境. 
    此时需要在系统中安装多个Python, 但又不能影响系统自带的Python, 即需要实现Python的多版本共存. pyenv就是这样一个Python版本管理器. 

* 安装pyenv

        brew install pyenv

    **根据提示需要添加**

        if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
        export PYENV_ROOT=/usr/local/var/pyenv
    
    **这里注意了, 如果使用的zsh, 务必不要在zshrc配置里面的插件开启pyenv. 会导致终端无限循环退出, 只需要在你的zshrc结尾处追加上面两行就行了. 笔者亲测. **

## 查看版本

* 查看当前激活的是那个版本的Python
  
    ```bash
  pyenv version
  ```
  
* 查看已经安装了那些版本的Python

    ```bash
    pyenv versions
    ```

* 安装指定版本的Python

    ```bash
    pyenv install 3.5.0
    
    安装完成后必须rehash
    
    pyenv rehash
    ```
    
    **安装过程中, 也是出现报错BUILD FAILED (OS X 10.10.5 using python-build 20160130), 解决办法见[pyenv BUILD FAILED解决方法][1]**
    
## 切换和使用指定的版本Python版本有3种方法：

* 特别建议：
  
* 系统全局用系统默认的Python比较好, 不建议直接对其操作    

    ```bash
    pyenv global system
    ```

* 用local进行指定版本切换, 一般开发环境使用.     

    ```bash
    pyenv local 2.7.10
    ```

* 对当前用户的临时设定Python版本, 退出后失效

    ```bash
    pyenv shell 3.5.0
    ```

* 取消某版本切换    

    ```bash
    pyenv local 3.5.0 --unset
    ```

* 优先级关系：shell——local——global

## 参考

* [mac osx10.11通过pyenv让python2.7和3.5共存 - 简书][2]
* [mac上Python多版本共存(python2.7.10和python3.5.0) - mingaixin - 博客园][3]

[1]: http://www.cnblogs.com/mingaixin/p/6295799.html
[2]: http://www.jianshu.com/p/6e17dd5296d6
[3]: http://www.cnblogs.com/mingaixin/p/6295963.html