---
layout: post
title:  "Python高级语法"
date:   2017-09-22 16:53:00
categories: Language
tags: Python
author: miaoqi
---

* content
{:toc}

## import导入模块

### 搜索路径

* 路径搜索

        import sys
        sys.path

    从上面列出的目录里依次查找要导入的模块文件      
    '' 表示当前路径

* 添加导入模块路径

        sys.path.append('/home/itcast/xxx')
        sys.path.insert(0, '/home/itcast/xxx')    #可以确保先搜索这个路径

### 重新导入模块

* 模块被导入后，import module不能重新导入模块，重新导入需用 reload(module)方法

        from imp import *
        import reload_test
        reload(reload_test)

### 循环导入

* a模块导入了b模块, 而b模块同时导入了a模块, 就形成了循环导入, 要避免该问题的发生

## ==, is

### ==

* == 是比较两个对象的值是否相等。(相当于Java中的equals方法)

### is

* is 是比较两个引用是否指向了同一个对象（引用比较, 相当于Java中的==运算符）

## 深拷贝, 浅拷贝

### 深拷贝

* 深拷贝是对于一个对象所有层次的拷贝(递归)deepcopy

        import copy
        a = [11, 22, 33]
        b = copy.deepcopy(a)

* copy和deepcopy的不同

        a = [11, 22, 33]
        b = [44, 55, 66]
        c = [a, b]
        d = c # 浅拷贝
        e = copy.deepcopy(c) # 会递归拷贝全部东西
        f = copy.copy(c) # 不会递归拷贝全部东西

* 拷贝元组时的特点

        a = [1, 2, 3]
        b = [4, 5, 6]
        c = (a, b)
        d = copy.copy(c) # 会自动判断当前的拷贝类型是否是可变类型, 如果是可变类型, 就拷贝一层, 否则一层都不拷贝
        e = copy.deepcopy(c) # 不管是不是可变类型, 全部都会拷贝

### 浅拷贝

* 浅拷贝是对于一个对象的顶层拷贝

    通俗的理解是：拷贝了引用，并没有拷贝内容

        a = [11, 22, 33]
        id(a)
        
        b = a
        id(b)

## 位运算

### 进制转换

* 十进制->二进制

        bin(18)
        
* 十进制->八进制

        oct(18)

* 十进制->十六进制

        hex(18)

### 位运算

* & 按位与

    全1才1否则0 :只有对应的两个二进位均为1时,结果位才为1,否则为0
用6和3这个例子。不要用9 和13的例子

* | 按位或

    有1就1 只要对应的二个二进位有一个为1时,结果位就为1,否则为0

* ^ 按位异或

    不同为1 当对应的二进位相异(不相同)时,结果为1,否则为0

* ~ 按位取反

    ~9 = -10

    9的原码 ==> 0000 1001 因为正数的原码=反码=补码，所以在 真正存储的时候就是0000 1001

    接下来进行对9的补码进行取反操作
    
    进行取反==> 1111 0110 这就是对9 进行了取反之后的补码
    
    既然已经知道了补码，那么接下来只要转换为 咱们人能识别的码型就可以，因此按照规则 ，把这个1111 0110 这个补码 转换为原码即可
    
    符号位不变，其它位取反==> 1000 1001
    
    然后+1 ，得到原码 =======>1000 1010 这就是 -10

* << 按位左移(8<<3 等同于8*2^3)

    各二进位全部左移n位,高位丢弃,低位补0

    **a. 左移可能会改变一个数的正负性**

    **b. 左移1位相当于 乘以2**

* \>\> 按位右移

    各二进位全部右移n位,保持符号位不变

    x >> n x的所有二进制位向右移动n位,移出的位删掉,移进的位补符号位 右移不会改变一个数的符号


## 私有化

### 几种命名规则

* xx: 公有变量

* \_x: 单前置下划线,私有化属性或方法，from somemodule import *禁止导入,类对象和子类可以访问

* \_\_xx：双前置下划线,避免与子类中的属性命名冲突，无法在外部直接访问(名字重整所以访问不到)

* \_\_xx\_\_:双前后下划线,用户名字空间的魔法对象或属性。例如:\_\_init\_\_ , __ 不要自己发明这样的名字

* xx_:单后置下划线,用于避免与Python关键词的冲突

**通过name mangling（名字重整(目的就是以防子类意外重写基类的方法或者属性)如：_Class__object）机制就可以访问private了。**

    class Test(object):
        def __init__(self):
            self.__num = 10
            
    t = Test()
    dir(t) # 可以查看t中结构, 从而可以看到__num的名字被改变了, 知道了这个原理就可以直接访问私有属性了

### 总结

* 父类中属性名为__名字的，子类不继承，子类不能访问

* 如果在子类中向__名字赋值，那么会在子类中定义的一个与父类相同名字的属性

* \_名的变量、函数、类在使用from xxx import *时都不会被导入


## 属性property

* 私有属性添加getter和setter方法

        class Money(object):
            def __init__(self):
                self.__money = 0
        
            def getMoney(self):
                return self.__money
        
            def setMoney(self, value):
                if isinstance(value, int):
                    self.__money = value
                else:
                    print("error:不是整型数字")

* 使用property升级getter和setter方法

        class Money(object):
            def __init__(self):
                self.__money = 0
            
            def getMoney(self):
                return self.__money
            
            def setMoney(self, value):
                if isinstance(value, int):
                    self.__money = value
                else:
                    print("error:不是整型数字")
        money = property(getMoney, setMoney)
        
        m = Money()
        m.money = 50 # 等同于m.setMoney(50)
        m.money # 等同于m.getMoney()

* 使用property取代getter和setter方法

    @property称为属性函数，可以对属性赋值时做必要的检查，并保证代码的清晰短小，主要有2个作用

    将方法转换为只读, 重新实现一个属性的设置和读取方法, 可做边界判定

        class Money(object):
            def __init__(self):
                self.__money = 0
        
            @property
            def money(self):
                return self.__money
        
            @money.setter
            def money(self, value):
                if isinstance(value, int):
                    self.__money = value
                else:
                    print("error:不是整型数字")


## 迭代器

* 迭代是访问集合元素的一种方式。迭代器是一个可以记住遍历的位置的对象。迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。

### 可迭代对象

* 以直接作用于 for 循环的数据类型有以下几种:

* 一类是集合数据类型, 如 list、 tuple、 dict、 set、 str 等;

* 一类是 generator, 包括生成器和带 yield 的generator function

* 这些可以直接作用于 for 循环的对象统称为可迭代对象: Iterable

* 判断是否可以迭代

    可以使用 isinstance() 判断一个对象是否是 Iterable 对象:

        from collections import Iterable

        isinstance([], Iterable)

        isinstance({}, Iterable)

        isinstance('abc', Iterable)

        isinstance((x for x in range(10)), Iterable)

        isinstance(100, Iterable)

    而生成器不但可以作用于 for 循环，还可以被 next() 函数不断调用并返回下一个值，直到最后抛出 StopIteration 错误表示无法继续返回下一个值了。

### 迭代器

* 可以被next()函数调用并不断返回下一个值的对象称为迭代器: Iterator

### iter()函数

* 生成器都是 Iterator 对象，但 list, dict, str 虽然是 Iterable, 却不是 Iterator把 list, dict, str 等 Iterable 变成 Iterator 可以使用 iter() 函数：

### 总结

* 可作用于 for 循环的对象都是 Iterable 类型；

* 凡是可作用于 next() 函数的对象都是 Iterator 类型

* 集合数据类型如 list 、 dict 、 str 等是 Iterable 但不是 Iterator ，不过可以通过 iter() 函数获得一个 Iterator 对象。




























