---
layout: post
title:  "Linux命令"
date:   2017-07-31 13:51:33
categories: OperatingSystem
tags: Linux
author: miaoqi
---

* content
{:toc}
                                    
## Linux基本命令
    
* 显示当前路径下的所有文件及文件夹
 
        ls
    
* 显示隐藏文件

        ls -a
        
* 以列表风格显示

        ls -l
        
* 格式化大小显示:
        
        ls -l -h
    
        /bin: 程序相关
        /boot: 开机相关
        /cdrom: 光盘相关
        /dev: 设备相关
        /etc: 配置相关
        /lib: 库相关
        /home: 家目录
    
* 显示当前所在路径

        pwd
    
        / 斜杠
        \ 反斜杠
        - 横岗
        _ 下划线
        | 竖杠
    
* 切换目录

        cd
    
* 回到上一次所在目录
        
        cd -
        
* 回到用户主目录
        
        cd ~
    
* 创建文件
        
        touch 1.txt:
    
* 创建文件夹
        
        mkdir A:

* 创建多级文件夹
        
        mkdir -p AA/BB/CC
    
* 清屏
        
        clear
    
* 查看帮助文档
        
        ls --help
    
        man ls: f 向下 b 向上
        
* 自动补全
        
        tab
        
* 查看文件内容
        
        cat 1.txt: 全部输出
        more: f 向下 b 向下
        cat 1.txt 2.txt > 3.txt 合并文件内容
    
* 查看历史命令
        
        history
        
* 执行历史行数的命令
        
        !line(!100)配合history命令使用
        
* 删除
        
        rm
        
* 删除文件夹
        
        rm -r
        
* 通配符
        
        *: 0~n位
        ?: 必须有任意一位
        ?[123]: 123中的一位
        rm *.txt
        rm ?1.txt
        rm ?[123].txt
    
* 重定向
    
        ls > xxx.txt
        ls >> xxx.txt 追加写入
        
* 管道符
        
        ls -alh /bin | more
    
* 终止
        
        ctrl + c
        
* 软连接
        
        ln -s 1.txt 1-softlink.txt
    
* 硬连接
        
        ln 1.txt 1-hardlink.txt
        
* 移动文件
        
        mv 1.txt 2.txt 重命名
        mv 1.txt A/2.txt 移动
    
* 复制粘贴
        
        cp 1.txt 2.txt
        cp -r A B/ 复制文件夹
        
* 搜索
        
        grep -n "content" 1.txt 在1.txt中搜索content的内容
        grep -v "content" 1.txt 在1.txt中搜索不包含content的内容
        grep -n "^content" 1.txt 在1.txt中搜索以content开头的内容
        grep -n "content$" 1.txt 在1.txt中搜索以content结尾的内容
    
* 查找
        
        find / -name "name" 按照名字查找
        find / -size [2M, +2M, -2M]
        find / -size +4k -size -5M 大于4k小于5M的
        find / -perm 777 查找777权限的文件
        
* 归档
    
        tar -cvf test.tar *.py 所有py结尾的文件打包成test.tar
        
* 解档
        
        tar -xvf test.tar
        
* 归档压缩
        
        tar -zcvf test.tar.gz *.py 所用py结尾的文件归档压缩
        tar -jcvf yyy.tar.bz2 *.py -C aa/ 解压到指定目录
        
* 解压缩解档

        tar -zxvf test.tar.gz
        tar -jxvf yyy.tar.bz2
        
* 查看命令所在路径

        which ls

* 后台执行命令

        nohup command &
        
## 系统管理相关命令
    
* 查看日历
    
        cal
        cal -y 2008
    
* 查看当前时间
    
        date
        date "+%Y年%m月%d日"
        
* 查看进程

        ps
        ps -aux 查看全部进程
        ps -ef 查看进程
        top 实时刷新
        htop
        
* 结束进程
    
        kill -9 pid
        
* 重启
        
        reboot
        
* 关机
    
        shutdown -h [now, +10, 20:20]
        
* 查看硬盘
    
        df -h
        
* 查看当前文件夹大小
    
        du -h
        
* 查看网络信息
    
        ifconfig
        ifconfig en0 192.168.22.60 修改本机ip地址
        
* 查看网络是否可以通信
    
        ping ip地址

## 用户相关命令
    
* 创建用户

        useradd shuaige -m
    
* 查看用户
    
        cat /etc/passwd
        
* 删除用户
    
        userdel shuaige
        userdel -r shuaige 连同家目录一起删除
    
* 创建用户组
    
        groupadd yyyy
    
* 查看用户组
    
        cat /etc/group
        
* 删除用户组
    
        groupdel yyyy
              
* 将用户添加到某个组中
    
        usermod -a -G 组名 用户名
        -a: add
        -G: group
        usermod: user modify
        
* 切换用户
    
        su shuaige
        su - shuaige 连同家目录一起切换
        sudo -s 切换到root用户(mac, ubuntu)
        su root(除去mac和ubuntu)
        
* 修改密码
    
        passwd shuaige
        
* 查看当前用户名
    
        whoami
        
* 查看所有登录的用户
    
        who
        
* 退出当前用户
    
        exit
        
* 修改文件所属用户
    
        chown 用户名 文件名
        
* 修改文件所属用户组
    
        chgrp 组名 文件名
        
## 权限相关命令

* 修改文件权限
    
        chmod u=rwx 2.py
        chmod g=rx 2.py
        chmod o=rwx 2.py
        chmod u=r, g=r, o=r 2.py
        
        chmod 137 2.py
        
        
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    