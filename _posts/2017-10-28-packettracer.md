---
layout: post
title:  "PacketTracer使用"
date:   2017-10-28 14:26:00
categories: Net
tags: PacketTracer
author: miaoqi
---

* content
{:toc}

## 模式

* 用户模式：查看统计信息
* 特权模式：查看并修改设备的配置
* 全局模式：针对整个交换机修改配置参数
* 接口模式：针对设备的接口修改配置参数

## 命令

* 模式切换:

        > 用户模式(特权模式下 disable)
        # 特权模式(用户模式下 enable 全局模式下 end)
        config 全局模式(在特权模式下 config terminal)
        config-if 接口模式(在全局模式下 interface f0/0)

* 修改主机名

        hostname [name]

* 配置ip地址

        ip address [ip地址] [子网掩码]

* 重启接口

        no shutdown

* 静态路由

        ip route [目的网段] [子网掩码] [路由下一跳地址]

* 查看路由配置

        show ip route


    
    
    
    
    
    
    