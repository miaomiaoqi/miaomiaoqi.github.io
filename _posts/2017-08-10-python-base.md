---
layout: post
title:  "Python基础语法"
date:   2017-08-10 15:12:38
categories: Language
tags: Python
author: miaoqi
---

* content
{:toc}

## 注释

* 单行注释: 以#号开头, 右边的内容当做注释
    
* 多行注释: '''-----'''      
          """-----"""

* 作用: 提高可读性
    
## 中文处理

* 首行: #coding=utf-8         
       #-\*- coding:utf-8 -\*-(推荐)

* 作用: 使python2识别中文

## 变量

    age = 18
    
## 变量命名规则

* 标示符由字母、下划线和数字组成，且数字不能开头
    
* 使用驼峰命名规则

## 关键字

|        |     |       |     |       |        |     |      |
|--------|-----|-------|-----|-------|--------|-----|------|
|and     |as   |assert |break|class  |continue|def  |del   |
|elif    |else |except |exec |finally|for     |from |global|
|if      |in   |import |is   |lambda |not     |or   |pass  |
|print   |raise|return |try  |while  |with    |yield|      |
    
* 查看当前版本的关键字

        import keyword
        keyword.kwlist                         


## 数据类型

    Number: int(有符号整形)

            long(长整形[也可以代表八进制和十六进制])

            float(浮点型)
            
            complex(复数)
            
    布尔类型: True
             
             False

    String(字符串)
    
    List(列表)

    Tuple(元组)
    
    Dictionary(字典)

## 类型转换

    type(variable): 查看变量类型

    int(x [,base ]): 将x转换为一个整数
    long(x [,base ]): 将x转换为一个长整数
    float(x): 将x转换到一个浮点数
    complex(real [,imag ]): 创建一个复数
    str(x): 将对象 x 转换为字符串
    repr(x): 将对象 x 转换为表达式字符串
    eval(str): 用来计算在字符串中的有效Python表达式,并返回一个对象
    tuple(s): 将序列 s 转换为一个元组
    list(s): 将序列 s 转换为一个列表
    chr(x): 将一个整数转换为一个字符
    unichr(x): 将一个整数转换为Unicode字符
    ord(x): 将一个字符转换为它的整数值
    hex(x): 将一个整数转换为一个十六进制字符串
    oct(x): 将一个整数转换为一个八进制字符串

## 输入

    input("请输入姓名:")

    输入的东西被当做字符串(python3)
    
    输入的东西被当做语句去执行(python2)
    
    raw_input("请输入姓名:")
    
    输入的东西被当做字符串(python2)

## 输出

    name = "miaoqi"

    print("name is %s"%name): 输出单个变量
    
    print("姓名是:%s, 年龄是:%d, 地址是:%s"%(name, age, addr)): 输出多个变量
    print("*", end = ""): 打印结束后不换行


    常用的格式符号
    格式符号    转换
    %c         字符
    %s         通过str() 字符串转换来格式化
    %i         有符号十进制整数
    %d         有符号十进制整数
    %u         无符号十进制整数
    %o         八进制整数
    %x         十六进制整数（小写字母）
    %X         十六进制整数（大写字母）
    %e         索引符号（小写'e'）
    %E         索引符号（大写“E”）
    %f         浮点实数
    %g         ％f和％e 的简写
    %G         ％f和％E的简写
    
    \n: 换行输出
    print("1234567890\n-------")
    
## 判断

* if

        if 条件:
            条件成立的时候, 做的事情
        
* if...else...

        if 条件:
            条件成立的时候, 做的事情
        else:
            条件不成立的时候, 做的事情

* if...elif...else...
    
        if 条件1:
            条件1成立做的事情
        elif 条件2:
            条件2成立做的事情
        elif 条件3:
            条件3成立做的事情
        ...
        else:
            以上都不满足做的事情

* if的各种真假判断

        条件为假: 相当于False
            ""
            None
            0
            []
            ()
            {}
    

## 运算符

* 算术运算符
    
        下面以a=10 ,b=20为例进行计算
        +	加	两个对象相加 a + b 输出结果 30
        -	减	得到负数或是一个数减去另一个数 a - b 输出结果 -10
        *	乘	两个数相乘或是返回一个被重复若干次的字符串 a * b 输出结果 200
        /	除	x除以y b / a 输出结果 2
        //	取整除	返回商的整数部分 9//2 输出结果 4, 9.0//2.0 输出结果 4.0
        %	取余	返回除法的余数 b % a 输出结果 0
        **	幂	返回x的y次幂 a**b 为10的20次方， 输出结果 100000000000000000000
        
        字符串*10 输出10次该字符串
    
    
* 复合赋值运算符
    
        运算符	描述	实例
        +=	加法赋值运算符	c += a 等效于 c = c + a
        -=	减法赋值运算符	c -= a 等效于 c = c - a
        *=	乘法赋值运算符	c *= a 等效于 c = c * a
        /=	除法赋值运算符	c /= a 等效于 c = c / a
        %=	取模赋值运算符	c %= a 等效于 c = c % a
        **=	幂赋值运算符	c **= a 等效于 c = c ** a
        //=	取整除赋值运算符	c //= a 等效于 c = c // a
    
    
* 比较运算符如下表
    
        运算符	描述	示例
        ==	检查两个操作数的值是否相等，如果是则条件变为真。如a=3,b=3则（a == b) 为 true.
        !=	检查两个操作数的值是否相等，如果值不相等，则条件变为真。如a=1,b=3则(a != b) 为 true.
        <>	检查两个操作数的值是否相等，如果值不相等，则条件变为真。如a=1,b=3则(a <> b) 为 true。这个类似于 != 运算符
        >	检查左操作数的值是否大于右操作数的值，如果是，则条件成立。如a=7,b=3则(a > b) 为 true.
        <	检查左操作数的值是否小于右操作数的值，如果是，则条件成立。如a=7,b=3则(a < b) 为 false.
        >=	检查左操作数的值是否大于或等于右操作数的值，如果是，则条件成立。如a=3,b=3则(a >= b) 为 true.
        <=	检查左操作数的值是否小于或等于右操作数的值，如果是，则条件成立。如a=3,b=3则(a <= b) 为 true.


* 逻辑运算符
    
        运算符	逻辑表达式	描述	实例
        and	x and y	布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。	(a and b) 返回 20。
        or	x or y	布尔"或" - 如果 x 是 True，它返回 True，否则它返回 y 的计算值。	(a or b) 返回 10。
        not	not x	布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。	not(a and b) 返回 False

## 循环

* while
    
        while 条件:(通过条件来控制循环次数)
            条件满足做的事情

* for in
    
        for temp in variable:(根据变量的内容来控制循环次数)
            遍历variable
    
        break: 跳出当前循环
    
        continue: 跳过当前循环, 进入下次循环
        
        pass: 什么都不干, 占位使用

## 字符串

* 字符串长度都是不可变得, 所以操作会返回新的字符串

        num = 100 占1个字节
    
        num = "100" 占3个字节(实际上最后一位还有\0)

* 拼接
    
        a = "lao"
        
        b = "wang"
        
        方式一: c = "===" + a + b + "==="
        
        方式二: c = "===%s==="%(a + b)


* 下标
    
        name = "abcde"
        
        name[2] 取第三个元素
        
        name[-1] 取最后一个元素


* 切片
    
        name = "abcdefABCDEF"
    
        name[2:]: cdefABCDEF
        name[2:5]: cde
        name[2:-1]: cdefABCDE
        name[2:-1:2]: ceACE 第三个参数代表步长
        
        [起始位置:终止位置:步长]
    
        步长为正, 向右找
        步长为负, 向左找
        不写位置, 代表从头开始
    
    
* 倒叙
    
        [-1::-1]
        [::-1]

## 字符串常见操作

    str = "hello world miaoqi and miaoqicpp"
    
    查找:
    str.find("world"): 返回下标, 找不到返回-1
    str.find("world", start, end)
    str.rfind("world"): 从右边找
    str.rfind("world", start, end):
    str.index("world"): 返回下标, 找不到产生异常
    str.index("world", start, end):
    str.rindex("world"): 从右边找
    str.rindex("world", start, end):


    统计:
    str.count("miaoqi", start, end): 在start和end之间出现的个数, 找不到返回0
    
    
    替换:
    str.replace("world", "WORLD", 1): 替换不超过1个


    切割:
    str.split(): 会按照空白符全部分割
    str.split(" ")
    str.split(" ", 2): 只分割两个
    
    
    首字母大写:
    str.capitalize()


    每个单词首字母大写:
    str.title()

    
    以hello开头:
    str.startswith("hello")

    
    以cpp结尾:
    str.endswith("cpp")


    转换为小写:
    str.lower()

    
    转换为大写:
    str.upper()

    
    返回一个原字符串左/右对齐,并使用空格填充至长度 width 的新字符串
    str.ljust(50)
    str.rjust(50)


    返回一个原字符串居中,并使用空格填充至长度 width 的新字符串
    str.center(50)


    删除左/右边的空白字符
    str.lstrip()
    str.rstrip()


    删除mystr字符串两端的空白字符
    str.strip()


    把str以miaoqi分割成三部分, miaoqi前，miaoqi和miaoqi后
    str.partition("miaoqi")
    str.rpartition("miaoqi")


    按照行分隔，返回一个包含各行作为元素的列表
    str = "hello\nworld"
    str.splitlines()


    如果 str 所有字符都是字母 则返回 True,否则返回 False
    str.isalpha()
    
    
    如果 str 只包含数字则返回 True 否则返回 False
    str.isdigit()


    如果 str 所有字符都是字母或数字则返回 True,否则返回 False
    str.isalnum()


    如果 str 中只包含空格，则返回 True，否则返回 False
    str.isspace()


    str 中每个字符后面插入str,构造出一个新的字符串
    str = ["aaa", "bbb", "ccc"]
    b = "="
    b.join(str)

## 列表

    存储多个数据, 也存在下标和切片
    
    java中的数组只能存储同一个数据类型
    python中的列表可以存储不同类型的数据

    names = ["老王", "老李", "老刘", 1]

    增:
    names.append("老赵"): 在最后添加一个元素
    names.insert(位置, 内容)
    names3 = names + names2
    names.extend(names2): names2会合到names中
    
    
    删:
    names.pop(): 删除最后一个元素并返回
    names.remove("aaa"): 根据内容去删除
    del names[0]: 根据下标去删除
    
    
    改:
    names[0] = "xxx"
    
    
    查:
    "xxx" in names
    "xxx" not in names
    
    index, count与字符串操作一样


    从小到大排序:
    names.sort()
    
    
    从大到校排序:
    names.sort(reverse=True)
    
    
    倒序:
    names.reverse()
    

## 字典

    info = {建:值, 键:值}
    
    info = {"name":"班长", "addr":"山东", "age":18}

    print("%s    %d    %s"%(info['name'], info['age'], info['addr']))
    
    增:
    info[key] = value


    删:
    del info[key]
    
    
    改:
    info[已存在的key] = newValue


    查:
    info.get(key)
    info[key]


    获取键的列表:
    info.keys()
    
    
    获取值的列表:
    info.values()
    
    
    获取键值:
    info.items()


## 元组

    只能查, 不能改

    a = (11, 22)
    
    b = a
    
    c, d = a
    
    index, count与字符串操作一样
    
    元组只有一个元素时必须加一个",":
    a = (11, )
    
        

## 遍历

    针对字符串:
    遍历: 遍历的是每一个字符
    str = "abcde"
    for s in str:
        print(s)


    针对列表
    遍历: 遍历列表中的元素
    names = ["miaoqi", "lele", "miao", "wang"]
    for name in names:
        print(name)


    针对字典
    遍历: 遍历的是key值
    info = {"name": "miaoqi", "age": 20}
    for key in info:
        print(key)


    针对元组
    遍历: 遍历的是元素, 可以遍历多个
    info = {"name": "miaoqi", "age": 20}
    for i, j in info.items():
        print("%s%s"%(i, j))

    for...else...语法:
    for info in infos:
        xxx
        #break
    else:(else中的代码一定会执行)
        xxx


## 函数

如果在开发程序时，需要某块代码多次，但是为了提高编写的效率以及代码的重用，所以把具有独立功能的代码块组织为一个小模块，这就是函数

    定义函数的格式如下：

    def 函数名():
        代码

    
    def 函数名(a, b): 带参, 带返回值
        c = a+b
        return c
        
    函数名(): 调用


    文档说明:
    def 函数名():
        """文档说明"""
        ...自己的代码
        
    help(函数名) 查看文档说明


## 参数

在定义函数的时候可以让函数接收数据，这就是函数的参数

    def add2num(a, b):
        c = a+b
        print c

缺省参数(只能放在最后)

    def test(a,d,b=22,c=33):
        print(a)
        print(b)
        print(c)
        print(d)
  
    test(d=11,a=22,c=44): 命名参数


不定长参数一(只能放在最后)

    def sum_2_nums(a,b,*args): 多出的参数会放在args元组中
        print("-"*30)
        print(a)
        print(b)
        print(args)
  
        result = a+b
        for num in args:
            result+=num
        print("result=%d"%result)
 
    sum_2_nums(11,22,33,44,55,66,77)


不定长参数二
    
    def test(a,b,c=33,*args,**kwargs):#在定义的时候 *,**用来表示后面的变量有特殊功能
       print(a)
       print(b)
       print(c)
       print(args)
       print(kwargs)
  
    #test(11,22,33,44,55,66,77,task=99,done=89)
 
    A = (44,55,66)
    B = {"name":"laowang","age":18}
 
    test(11,22,33,*A,**B)#在实参中*,**表示对元祖/字典进行拆包
    
    * 存放多余的非命名参数, ** 存放多余的命名参数


## 返回值

    想要在函数中把结果返回给调用者，需要在函数中使用return

    def divid(a, b):
        shang = a//b
        yushu = a%b 
        return shang, yushu
    
    sh, yu = divid(5, 2)
    
    本质是利用了元组, 还可以返回列表


## 局部变量

    局部变量: 就是在函数内部定义的变量   
    不同的函数: 可以定义相同的名字的局部变量，但是各用个的不会产生影响  
    局部变量的作用: 为了临时保存数据需要在函数中定义变量来进行存储，这就是它的作用


## 什么是全局变量

    如果一个变量，既能在一个函数中使用，也能在其他的函数中使用，这样的变量就是全局变量
    
    在函数外边定义的变量叫做全局变量    
    全局变量能够在所有的函数中进行访问      
    如果在函数中修改全局变量，那么就需要使用global进行声明     
    如果全局变量的名字和局部变量的名字相同，那么使用的是局部变量的，小技巧强龙不压地头蛇
    
    建议全局变量gName以g开头代表全局变量
    
    列表, 元组, 字典的全局变量可以不使用global声明就可以进行改变


## 拆包

    A = (44,55,66)
    B = {"name":"laowang","age":18}
    
    *A 一个*拆元组
    **B 两个*拆字典


## 引用

    id(variable) 查看变量的内存地址
    
    a = 100
    b = a
    
    id(a)
    id(b)

    python中全是引用


## 可变, 不可变类型

    可变: 列表, 字典
    不可变: 数字, 字符串, 元组
    
    字典的key只能是不可变类型, 相当于HashMap


## 匿名函数

    用lambda关键词能创建小型匿名函数。这种函数得名于省略了用def声明函数的标准步骤

    Lambda函数能接收任何数量的参数但只能返回一个表达式的值

    sum = lambda arg1, arg2: arg1 + arg2

    #调用sum函数
    print("Value of total : ", sum( 10, 20 ))

    # 列表排序
    infors = [{"name":"laowang","age":10},{"name":"xiaoming","age":20},{"name":"banzhang","age":10}]
  
    infors.sort(key=lambda x:x['age']): 按照key去排序

    会将infors中的每一个元素赋值给x, 返回age的值, 按照age的值去排序
    
    
    当做实参传递(day08/03)
    def test(a,b,func):
        result = func(a,b)
        return result
        
    num = test(11,22,lambda x,y,z=10:x+y+z)
    print(num)
    
    
    eval(str): 会将字符串转成代码(day08/04)


## 知识点补充

    num += num: 如果num指向的是一个可变类型, 则直接修改变量内容本身, 如果num指向的是不可变类型, 那么相当于num = num + num, 从新定义了一个变量, 让新的变量指向新的内容区域.
    
    num = num + num: 从新让num指向一个新的内容区域


## 文件

    打开文件: f = open("test.py", "r")
    
    关闭文件: f.close()
    
    访问模式	说明
    r	以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
    w	打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
    a	打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
    rb	以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。
    wb	以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
    ab	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
    r+	打开一个文件用于读写。文件指针将会放在文件的开头。
    w+	打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
    a+	打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
    rb+	以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。
    wb+	以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
    ab+	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。


    读写: 
    read(num): num代表读出的数据长度, 单位是字节, 不传代表读出全部
    readlines(): 把整个文件中的内容一次性读取, 返回一个列表, 每一行的数据为一个元素
    readline(): 只读一行


    定位读写:
    seek(10, [0, 1, 2]): 0: 文件开头 1: 当前位置 2: 文件末尾
    tell(): 查看当前指针位置
    
    
    文件相关操作:
    import os
    os.rename("", "")
    os.remove("")
    
    
    文件夹相关操作:
    import os
    os.mkdir("")
    os.rmdir("")
    
    os.getcwd(): 获取当前操作的绝对路径
    os.chdir(""): 改变默认路径
    
    os.listdir("./")


## 面向对象

    面向过程：根据业务逻辑从上到下写代码
    面向对象：将数据与函数绑定到一起，进行封装，这样能够更快速的开发程序，减少了重复代码的重写过程
    

## 类和对象

    类: 具有相似内部状态和运动规律的实体的集合(或统称为抽象)。具有相同属性和行为事物的统称
    对象: 某一个具体事物的存在 ,在现实世界中可以是看得见摸得着的。可以是直接使用的
    
    类(Class) 由3个部分构成
        类的名称: 类名
        类的属性: 一组数据
        类的方法: 允许对属性进行操作的方法 (行为)
        
    定义类:
        class Car:
            # 方法
            def getCarInfo(self):
                print('车轮子个数:%d, 颜色%s'%(self.wheelNum, self.color))
        
            def move(self):
                print("车正在移动...")
                
    创建对象:
        bmw = Car()
        
    添加属性:
        bmw.color = "red"
        bmw.wheelNum = 4
        
    self:
        对象调用方法默认会在第一个位置将自己传入
        bmw.move() == bmw.move(bmw)
        self指向对象的地址值


## 魔法方法:

    以__xxx__形式命名的方法在python中都有特殊功能, 所以叫做"魔法方法"


    __init__(self):
    创建对象后在__new__()方法中执行，python解释器默认调用, 是初始化方法


    __str__(self):
    输出对象信息, 打印对象时会自动调用该方法, 相当于java的toString方法
    
    
    __del__(self):
    删除对象时，python解释器会默认调用, 相当于C++的析构函数(只有当对象的引用计数是0时才会调用, 否则会等程序执行完毕后, gc清理对象再执行该方法)
    
    import sys
    sys.getrefcount(a): 获取a的引用计数
    
    
    __new__(cls):
    object中有默认的该方法的实现, 是创建对象的方法


## 魔法属性
    
    __mro__: 魔法属性, 查看方法的搜索顺序



## 私有属性:

    Python中没有像Java中public和private这些关键字来区别公有属性和私有属性

    它是以属性命名方式来区分，如果在属性名前面加了2个下划线'__'，则表明该属性是私有属性，否则为公有属性（方法也是一样，方法名前面加了2个下划线的话表示该方法是私有的，否则为公有的）。

    class People(object):
        def __init__(self, name):
            self.__name = name
    
        def getName(self):
            return self.__name


## 私有方法:

    __xx()
        

## 单继承:
  
    class Animal:
        def eat(self):
            print("-----吃-----")
        def drink(self):
            print("-----喝-----")
        def sleep(self):
            print("-----睡-----")
        def run(self):
            print("-----跑-----")
     
    class Dog(Animal):
        def bark(self):
            print("----汪汪叫----")
     
    class Cat(Animal):
        def catch(self):
            print("----抓老鼠----")
            
    class XiaoTianQuan(Dog):
        def fly(self):
            print("----我会飞----")
     
    # a = Animal()
    # a a.sleep()
    wangcai = Dog()
    wangcai.bark()
    wangcai.eat()
     
    tom = Cat()
    tom.run()
    
    Dog类可以使用Dog类中的方法, 可以使用Animal中的方法, 但是不可以使用Cat类中的方法
    XiaoTianQuan继承了Dog类, 间接也继承了Animal类
    
    私有方法和私有属性不会被继承, 但是可以通过公有方法间接调用私有方法和私有属性


## 多继承

    class Base(object): # 新式类,在python3中默认继承object
        def test(self):
            print("----base----")
    class A(Base):
        def testA(self):
            print("----testA----")
    class B(Base):
        def testB(self):
            print("----testB----")
    class C(A, B):
        pass
        
    c = C()
    c.testA()
    c.testB()
    c.test()
    
    注意点:
    
    class Base(object): 
        def test(self):
            print("----Base----")
    class A(Base):
        def test(self):
            print("----A----")
    class B(Base):
        def test(self):
            print("----B----")
    class C(A, B):
        pass
        
    c = C()
    c.test()
    
    print(C.__mro__) # 查看方法的搜索顺序
    


## 重写

* 子类不满意父类中的方法, 可以定义一个一模一样的方法, 调用时会优先调用自己的方法

        class Dog(Animal):
            def bark(self):
                print("----汪汪叫----")
                
        class XiaoTianQuan(Dog):
            def bark(self):
                print("----狂叫不止----")
                super().bark()
            def fly(self):
                print("----我会飞----")
                
        quan = XiaoTianQuan()
        quan.bark()
        
        调用父类被重写的方法
        方式一:
        Dog.bark(self)
        方式二:
        super().bark()


## 多态

    Python “鸭子类型”
    class F1(object):
        def show(self):
            print 'F1.show'
    
    class S1(F1):
    
        def show(self):
            print 'S1.show'
    
    class S2(F1):
    
        def show(self):
            print 'S2.show'
    
    def Func(obj):
        print obj.show()
    
    s1_obj = S1()
    Func(s1_obj) 
    
    s2_obj = S2()
    Func(s2_obj)


## 实例属性, 类属性

* 类属性就是类对象所拥有的属性，它被所有类对象的实例对象所共有，在内存中只存在一个副本，这个和Java中类的静态成员变量有点类似。对于公有的类属性，在类外可以通过类对象和实例对象访问

        class Tool:
            
            # 类属性
            num = 0
            
            # 方法
            def __init__(self, new_name):
                # 实例属性
                self.name = new_name
                Tool.num += 1
                
        tool1 = Tool("铁锹")
        tool2 = Tool("工兵铲")
        tool3 = Tool("水桶")


## 实例方法, 类方法, 静态方法

* 实例方法:

* 类方法:

    是类对象所拥有的方法，需要用修饰器@classmethod来标识其为类方法，对于类方法，第一个参数必须是类对象，一般以cls作为第一个参数（当然可以用其他名称的变量作为其第一个参数，但是大部分人都习惯以'cls'作为第一个参数的名字，就最好用'cls'了），能够通过实例对象和类对象去访问。
    
* 静态方法:

    需要通过修饰器@staticmethod来进行修饰，静态方法不需要多定义参数, 既不属于实例对象, 又不属于类对象
    
        class Game(object):
    
            #类属性
            num = 0
        
            #实例方法
            def __init__(self):
                #实例属性
                self.name = "laowang"
        
            #类方法
            @classmethod
            def add_num(cls): # 会将类对象当做参数第一个传进来
                cls.num = 100
        
            #静态方法
            @staticmethod
            def print_menu():
                print("----------------------")
                print("    穿越火线V11.1")
                print(" 1. 开始游戏")
                print(" 2. 结束游戏")
                print("----------------------")
    
        game = Game()
        #Game.add_num()#可以通过类的名字调用类方法
        game.add_num()#还可以通过这个类创建出来的对象 去调用这个类方法
        print(Game.num)
        
        #Game.print_menu()#通过类 去调用静态方法
        game.print_menu()#通过实例对象 去调用静态方法
    
    
* 实例属性: 实例对象调用(属于某一个实例)
* 类属性: 类对象和实例对象都可以调用(属于一个类)
* 实例方法: 实例对象调用(默认传入当前实例对象的引用)
* 类方法: 实例对象和类对象都可以调用(默认传入当前类对象的引用)
* 静态方法: 实例对象和类对象都可以调用(不传任何对象的引用)

## 设计模式

### 工厂模式

* 定义了一个创建对象的接口(可以理解为函数)，但由子类决定要实例化的类是哪一个，工厂方法模式让类的实例化推迟到子类，抽象的CarStore提供了一个创建对象的方法createCar，也叫作工厂方法。

* 子类真正实现这个createCar方法创建出具体产品。 创建者类不需要直到实际创建的产品是哪一个，选择了使用了哪个子类，自然也就决定了实际创建的产品是什么。


### 单例模式

* 举个常见的单例模式例子，我们日常使用的电脑上都有一个回收站，在整个操作系统中，回收站只能有一个实例，整个系统都使用这个唯一的实例，而且回收站自行提供自己的实例。因此回收站是单例模式的应用。

* 确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，单例模式是一种对象创建型模式。
    
        # 实例化一个单例
        class Singleton(object):
            __instance = None
            
            def __new__(cls, age, name):
                #如果类数字能够__instance没有或者没有赋值
                #那么就创建一个对象，并且赋值为这个对象的引用，保证下次调用这个方法时
                #能够知道之前已经创建过对象了，这样就保证了只有1个对象
                if not cls.__instance:
                    cls.__instance = object.__new__(cls)
                return cls.__instance
            
        a = Singleton(18, "dongGe")
        b = Singleton(8, "dongGe")
            
        print(id(a))
        print(id(b))
            
        a.age = 19 #给a指向的对象添加一个属性
        print(b.age)#获取b指向的对象的age属性


## 异常

### 异常基础

* 当Python检测到一个错误时，解释器就无法继续执行了，反而出现了一些错误的提示，这就是所谓的"异常"

### 捕获异常

* 使用try...except...语句进行异常的捕获

        try:
            print num
        except NameError:
            print('产生错误了')
    
* 捕获多个异常

        #coding=utf-8
        try:
            print('-----test--1---')
            open('123.txt','r') # 如果123.txt文件不存在，那么会产生 IOError 异常
            print('-----test--2---')
            print(num)# 如果num变量没有定义，那么会产生 NameError 异常
            
        except (IOError,NameError): 
            #如果想通过一次except捕获到多个异常可以用一个元组的方式
    
            # errorMsg里会保存捕获到的错误信息
            print(errorMsg)

    **当捕获多个异常时，可以把要捕获的异常的名字，放到except 后，并使用元组的方式仅进行存储**

* else:

        try:
            num = 100
            print num
        except NameError as errorMsg:
            print('产生错误了:%s'%errorMsg)
        else:
            print('没有捕获到异常，真高兴')

* try...finally...

        import time
        try:
            f = open('test.txt')
            try:
                while True:
                    content = f.readline()
                    if len(content) == 0:
                        break
                    time.sleep(2)
                    print(content)
            except:
                #如果在读取文件的过程中，产生了异常，那么就会捕获到
                #比如 按下了 ctrl+c
                pass
            finally:
                f.close()
                print('关闭文件')
        except:
            print("没有这个文件")

* 捕获所有异常

    1. 捕获所有异常(不包括ctrl + c)

            try:
                open("a.txt")
            except Exception as result: # as result获取了异常信息描述
                print("产生了一个异常")

    2. 捕获所有异常(包括ctrl + c)

            try:
                open("a.txt")
            except:
                print("产生了一个异常")

### 异常传递

    def test1():
        print("----test1-1----")
        print(num)
        print("----test1-2----")


    def test2():
        print("----test2-1----")
        test1()
        print("----test2-2----")


    def test3():
        try:
            print("----test3-1----")
            test1()
            print("----test3-2----")
        except Exception as result:
            print("捕获到了异常，信息是:%s"%result)

        print("----test3-2----")



    test3()
    print("------华丽的分割线-----")
    test2()

* 如果try嵌套，那么如果里面的try没有捕获到这个异常，那么外面的try会接收到这个异常，然后进行处理，如果外边的try依然没有捕获到，那么再进行传递。。。

* 如果一个异常是在一个函数中产生的，例如函数A---->函数B---->函数C,而异常是在函数C中产生的，那么如果函数C中没有对这个异常进行处理，那么这个异常会传递到函数B中，

* 如果函数B有异常处理那么就会按照函数B的处理方式进行执行；如果函数B也没有异常处理，那么这个异常会继续传递，以此类推。。。如果所有的函数都没有处理，那么此时就会进行异常的默认处理，即通常见到的那样
注意观察上图中，当调用test3函数时，在test1函数内部产生了异常，此异常被传递到test3函数中完成了异常处理，而当异常处理完后，并没有返回到函数test1中进行执行，而是在函数test3中继续执行

### 自定义异常

* 你可以用raise语句来引发一个异常。异常/错误对象必须有一个名字，且它们应是Error或Exception类的子类


        class ShortInputException(Exception):
            '''自定义的异常类'''
            def __init__(self, length, atleast):
                #super().__init__()
                self.length = length
                self.atleast = atleast
    
        def main():
            try:
                s = input('请输入 --> ')
                if len(s) < 3:
                    # raise引发一个你定义的异常
                    raise ShortInputException(len(s), 3)
            except ShortInputException as result:#x这个变量被绑定到了错误的实例
                print('ShortInputException: 输入的长度是 %d,长度至少应是 %d'% (result.length, result.atleast))
            else:
                print('没有异常发生.')
        
        main()

### 异常处理中抛出异常

    class Test(object):
        def __init__(self, switch):
            self.switch = switch #开关
        def calc(self, a, b):
            try:
                return a/b
            except Exception as result:
                if self.switch:
                    print("捕获开启，已经捕获到了异常，信息如下:")
                    print(result)
                else:
                    #重新抛出这个异常，此时就不会被这个异常处理给捕获到，从而触发默认的异常处理
                    raise


    a = Test(True)
    a.calc(11,0)
    
    print("----------------------华丽的分割线----------------")
    
    a.switch = False
    a.calc(11,0)

## 模块

### 介绍

* 有过Java语言编程经验的朋友都知道在Java中要使用他人写好的工具类, 要引入相应的jar包, 否则是无否正常调用的
    
    那么在Python中，如果要引用一些其他的函数，该怎么处理呢？
    
    在Python中有一个概念叫做模块（module），这个和C语言中的头文件以及Java中的包很类似，比如在Python中要调用sqrt函数，必须用import关键字引入math这个模块，下面就来了解一下Python中的模块。

    **说的通俗点：模块就好比是工具包，要想使用这个工具包中的工具(就好比函数)，就需要导入这个模块**

### 使用

* import

    在Python中用关键字import来引入某个模块，比如要引用模块math，就可以在文件最开始的地方用import math来引入

        import module1,mudule2...

    当解释器遇到import语句，如果模块在当前的搜索路径就会被导入。

    在调用math模块中的函数时，必须这样引用：

    模块名.函数名
    想一想:

    为什么必须加上模块名调用呢？

    答:

    因为可能存在这样一种情况：在多个模块中含有相同名称的函数，此时如果只是通过函数名来调用，解释器无法知道到底要调用哪个函数。所以如果像上述这样引入模块的时候，调用函数必须加上模块名

        import math

        #这样会报错
        print sqrt(2)

        #这样才能正确输出结果
        print math.sqrt(2)

    有时候我们只需要用到模块中的某个函数，只需要引入该函数即可，此时可以用下面方法实现：

        from 模块名 import 函数名1,函数名2....

    不仅可以引入函数，还可以引入一些全局变量、类等

    注意:

    通过这种方式引入的时候，调用函数时只能给出函数名，不能给出模块名，但是当两个模块中含有相同名称函数的时候，后面一次引入会覆盖前一次引入。也就是说假如模块A中有函数function( )，在模块B中也有函数function( )，如果引入A中的function在先、B中的function在后，那么当调用function函数的时候，是去执行模块B中的function函数。

    如果想一次性引入math中所有的东西，还可以通过from math import *来实现

* from…import

    Python的from语句让你从模块中导入一个指定的部分到当前命名空间中

    语法如下：

        from modname import name1[, name2[, ... nameN]]
    
    例如，要导入模块fib的fibonacci函数，使用如下语句：

        from fib import fibonacci

* from … import *

    把一个模块的所有内容全都导入到当前的命名空间也是可行的，只需使用如下声明：
    
        from modname import *

* as

        import time as tt

* \_\_all__

    如果导入这个模块的方式是 from 模块名 import * ,那么仅仅会导入__all__的列表中包含的名字

        __all__ = ["test1","Test"]    

* 定位模块

    当你导入一个模块，Python解析器对模块位置的搜索顺序是：
    
    当前目录
    如果不在当前目录，Python则搜索在shell变量PYTHONPATH下的每个目录。
    如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/
    模块搜索路径存储在system模块的sys.path变量中。变量里包含当前目录，PYTHONPATH和由安装过程决定的默认目录。

* 模块安装

        pip install 模块名

### 自定义模块

* 定义自己的模块

    在Python中，每个Python文件都可以作为一个模块，模块的名字就是文件的名字。

    比如有这样一个文件test.py，在test.py中定义了函数add
    
        def add(a,b):
            return a+b 

* 调用自己定义的模块

    那么在其他文件中就可以先import test，然后通过test.add(a,b)来调用了，当然也可以通过from test import add来引入

        import test

        result = test.add(11,22)
        print(result)

### 测试模块

* 在实际开中，当一个开发人员编写完一个模块后，为了让模块能够在项目中达到想要的效果，这个开发人员会自行在py文件中添加一些测试信息，例如：

        def add(a,b):
            return a+b

        # 用来进行测试
        ret = add(12,22)
        print('int test.py file,,,,12+22=%d'%ret)
        
* 如果此时，在其他py文件中引入了此文件的话，想想看，测试的那段代码是否也会执行呢！

        import test
        
        result = test.add(11,22)
        print(result)

* 至此，可发现test.py中的测试代码，应该是单独执行test.py文件时才应该执行的，不应该是其他的文件中引用而执行

    为了解决这个问题，python在执行一个文件时有个变量\_\_name__

    可以根据\_\_name__变量的结果能够判断出，是直接执行的python脚本还是被引入执行的，从而能够有选择性的执行测试代码

## 包

* 包将有联系的模块组织在一起，即放到同一个文件夹下，并且在这个文件夹创建一个名字为\_\_init__.py 文件，那么这个文件夹就称之为包, 有效避免模块名称冲突问题，让应用组织结构更加清晰

* \_\_init__.py文件有什么用?

    \_\_init__.py 控制着包的导入行为

* \_\_init__.py为空

    仅仅是把这个包导入，不会导入包中的模块

* \_\_all__

    在\_\_init\_\_.py文件中，定义一个\_\_all__变量，它控制着 from 包名 import *时导入的模块

* 包的嵌套

    假定我们的包的例子有如下的目录结构：

        Phone/
            __init__.py
            common_util.py
            Voicedta/
                __init__.py
                Pots.py
                Isdn.py
            Fax/
                __init__.py
                G3.py
            Mobile/
                __init__.py
                Analog.py
                igital.py
            Pager/
                __init__.py
                Numeric.py

    Phone 是最顶层的包，Voicedta 等是它的子包。 我们可以这样导入子包：

        import Phone.Mobile.Analog
        Phone.Mobile.Analog.dial()

    你也可使用 from-import 实现不同需求的导入

    第一种方法是只导入顶层的子包，然后使用属性/点操作符向下引用子包树：

        from Phone import Mobile
        Mobile.Analog.dial('555-1212')

    此外，我们可以还引用更多的子包：

        from Phone.Mobile import Analog
        Analog.dial('555-1212')
    
    事实上，你可以一直沿子包的树状结构导入：

        from Phone.Mobile.Analog import dial
        dial('555-1212')

    在我们上边的目录结构中，我们可以发现很多的 \_\_init\_\_.py 文件。这些是初始化模块，from-import 语句导入子包时需要用到它。 如果没有用到，他们可以是空文件。

    包同样支持 from-import all 语句：

        from package.module import *

    然而，这样的语句会导入哪些文件取决于操作系统的文件系统。所以我们在\_\_init\_\_.py 中加入 \_\_all\_\_ 变量。该变量包含执行这样的语句时应该导入的模块的名字。它由一个模块名字符串列表组成.


## 制作模块

1. mymodule目录结构体如下：

        .
        ├── setup.py
        ├── suba
        │   ├── aa.py
        │   ├── bb.py
        │   └── __init__.py
        └── subb
            ├── cc.py
            ├── dd.py
            └── __init__.py
        
2. 编辑setup.py文件

        py_modules需指明所需包含的py文件
        
        from distutils.core import setup
        
        setup(name="dongGe", version="1.0", description="dongGe's module", author="dongGe", py_modules=['suba.aa', 'suba.bb', 'subb.cc', 'subb.dd'])

3. 构建模块

        python setup.py build
    
        构建后目录结构
    
        .
        ├── build
        │   └── lib.linux-i686-2.7
        │       ├── suba
        │       │   ├── aa.py
        │       │   ├── bb.py
        │       │   └── __init__.py
        │       └── subb
        │           ├── cc.py
        │           ├── dd.py
        │           └── __init__.py
        ├── setup.py
        ├── suba
        │   ├── aa.py
        │   ├── bb.py
        │   └── __init__.py
        └── subb
            ├── cc.py
            ├── dd.py
            └── __init__.py
    
4. 生成发布压缩包

        python setup.py sdist

    打包后,生成最终发布压缩包dongGe-1.0.tar.gz , 目录结构

        .
        ├── build
        │   └── lib.linux-i686-2.7
        │       ├── suba
        │       │   ├── aa.py
        │       │   ├── bb.py
        │       │   └── __init__.py
        │       └── subb
        │           ├── cc.py
        │           ├── dd.py
        │           └── __init__.py
        ├── dist
        │   └── dongGe-1.0.tar.gz
        ├── MANIFEST
        ├── setup.py
        ├── suba
        │   ├── aa.py
        │   ├── bb.py
        │   └── __init__.py
        └── subb
            ├── cc.py
            ├── dd.py
            └── __init__.py

## 强化练习

### 给程序传参

    import sys

        print(sys.argv)
    
    python xx.py aa bb cc

### 列表生成式

* 所谓的列表推导式，就是指的轻量级循环创建列表

        a = [i for i in range(1, 18)]
        
        b = [11 for i in range(1, 18)] 

    **for只负责循环, 第一个i的值会被赋到a列表中**
    
* 在列表生成式中使用if

        c = [i for i in range(10) if i%2 == 0]

* 使用2个for循环

        d = [(i, j) for i in range(3) for j in range(2)]

* 使用3个for循环

        e = [(i, j, k) for i in range(3) for j in range(2) for k in range(3)]


## tuple, list, set 

* 元组

        a = (11, 22, 33, 11, 22, 33)
        type(a)
    
* 列表

        b = [11, 22, 33, 11, 22, 33]
        type(b)

* 集合

        c = {11, 22, 33, 11, 22, 33}
        type(c)
    
* 列表去重

        d = [11, 22, 33, 11, 22, 33]
        e = set(d)
        d = list(e)

## 飞机大战
    
    
    



    
![asf](/images/eclipse4.png)

[1]:  www.miaomiaoqi.cn









    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    