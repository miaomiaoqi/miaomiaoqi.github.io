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

## 生成器

* 通过列表生成式，我们可以直接创建一个列表。但是，受到内存限制，列表容量肯定是有限的。而且，创建一个包含100万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。

### 创建生成器

* 第一种方法: 只要把一个列表生成式的 [ ] 改成 ( )

        L = [x*2 for x in range(5)]
        G = (x*2 for x in range(5))

    创建 L 和 G 的区别仅在于最外层的 [ ] 和 ( ) ， L 是一个列表，而 G 是一个生成器。我们可以直接打印出L的每一个元素，但我们怎么打印出G的每一个元素呢？如果要一个一个打印出来，可以通过 next() 函数获得生成器的下一个返回值：

        next(G)
        next(G)
        next(G)

    生成器保存的是算法，每次调用 next(G) ，就计算出 G 的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出 StopIteration 的异常。当然，这种不断调用 next() 实在是太变态了，正确的方法是使用 for 循环，因为生成器也是可迭代对象。所以，我们创建了一个生成器后，基本上永远不会调用 next() ，而是通过 for 循环来迭代它，并且不需要关心 StopIteration 异常。

* 第二种方法: 函数实现

    generator非常强大。如果推算的算法比较复杂，用类似列表生成式的 for 循环无法实现的时候，还可以用函数来实现。

    比如，著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：

    1, 1, 2, 3, 5, 8, 13, 21, 34, ...

    斐波拉契数列用列表生成式写不出来，但是，用函数把它打印出来却很容易：

        def fib(times):
            n = 0
            a,b = 0,1
            while n<times:
                print(b)
                a,b = b,a+b
                n+=1
            return 'done'
        fib(5)

    仔细观察，可以看出，fib函数实际上是定义了斐波拉契数列的推算规则，可以从第一个元素开始，推算出后续任意的元素，这种逻辑其实非常类似generator。

    也就是说，上面的函数和generator仅一步之遥。要把fib函数变成generator，只需要把print(b)改为yield b就可以了：

        def fib(times):
            n = 0
            a,b = 0,1
            while n<times:
                yield b
                a,b = b,a+b
                n+=1
            return 'done'
        
        F = fib(5)
        next(F)

    在上面fib 的例子，我们在循环过程中不断调用 yield ，就会不断中断。当然要给循环设置一个条件来退出循环，不然就会产生一个无限数列出来。同样的，把函数改成generator后，我们基本上从来不会用 next() 来获取下一个返回值，而是直接使用 for 循环来迭代：

        for n in fib(5):
            print(n)

    但是用for循环调用generator时，发现拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中：

        g = fib(5)
        while True:
            try:
                x = next(g)
                print("value:%d"%x)      
            except StopIteration as e:
                print("生成器返回值:%s"%e.value)
                break

### send

* 执行到yield时，gen函数作用暂时保存，返回i的值;temp接收下次c.send("python")，send发送过来的值，c.next()等价c.send(None)

        def gen():
            i = 0
            while i<5:
                temp = yield i
                print(temp)
                i+=1

    使用next函数

        f = gen()
        next(f)
        next(f)

    使用__next__()方法

        f = gen()
        f.__next__()
        f.__next__()

    使用send

        f = gen()
        f.__next__()
        f.send('haha')

### 总结

* 生成器是这样一个函数，它记住上一次返回时在函数体中的位置。对生成器函数的第二次（或第 n 次）调用跳转至该函数中间，而上次调用的所有局部变量都保持不变。

* 生成器不仅“记住”了它数据状态；生成器还“记住”了它在流控制构造（在命令式编程中，这种构造不只是数据值）中的位置。

* 生成器的特点：

    节约内存

    迭代到下一次的调用时，所使用的参数都是第一次所保留下的，即是说，在整个所有函数调用的参数都是第一次所调用时保留的，而不是新创建的


## 迭代器

* 迭代是访问集合元素的一种方式。迭代器是一个可以记住遍历的位置的对象。迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。

### 可迭代对象

* 以直接作用于 for 循环的数据类型有以下几种:

    一类是集合数据类型, 如 list、 tuple、 dict、 set、 str 等;

    一类是 generator, 包括生成器和带 yield 的generator function

    这些可以直接作用于 for 循环的对象统称为可迭代对象: Iterable

* 判断是否可以迭代

    可以使用 isinstance() 判断一个对象是否是 Iterable 对象:

        from collections import Iterable, Iterator

        isinstance([], Iterable)

        isinstance({}, Iterable)

        isinstance('abc', Iterable)

        isinstance((x for x in range(10)), Iterable)

        isinstance(100, Iterable)

    而生成器不但可以作用于 for 循环，还可以被 next() 函数不断调用并返回下一个值，直到最后抛出 StopIteration 错误表示无法继续返回下一个值了。

### 迭代器

* 可以被next()函数调用并不断返回下一个值的对象称为迭代器: Iterator, 生成器一定是迭代器

### iter()函数

* 生成器都是 Iterator 对象，但 list, dict, str 虽然是 Iterable, 却不是 Iterator把 list, dict, str 等 Iterable 变成 Iterator 可以使用 iter() 函数：

### 总结

* 可作用于 for 循环的对象都是 Iterable 类型(可迭代对象)

* 凡是可作用于 next() 函数的对象都是 Iterator 类型(迭代器)

* 集合数据类型如 list 、 dict 、 str 等是 Iterable 但不是 Iterator ，不过可以通过 iter() 函数获得一个 Iterator 对象。


## 闭包

### 函数引用

    def test1():
        print("--- in test1 func----")
        
    #调用函数
    test1()
        
    #引用函数
    ret = test1
        
    print(id(ret))
    print(id(test1))
        
    #通过引用调用函数
    ret()

### 什么是闭包

    #定义一个函数
    def test(number):
    
        #在函数内部再定义一个函数，并且这个函数用到了外边函数的变量，那么将这个函数以及用到的一些变量称之为闭包
        def test_in(number_in):
            print("in test_in 函数, number_in is %d"%number_in)
            return number+number_in
        #其实这里返回的就是闭包的结果
        return test_in
    
    
    #给test函数赋值，这个20就是给参数number
    ret = test(20)
    
    #注意这里的100其实给参数number_in
    print(ret(100))
    
    #注意这里的200其实给参数number_in
    print(ret(200))

### 实例

* 看一个闭包的实际例子：

        def line_conf(a, b):
            def line(x):
                return a*x + b
            return line
            
        line1 = line_conf(1, 1)
        line2 = line_conf(4, 5)
        print(line1(5))
        print(line2(5))

    这个例子中，函数line与变量a,b构成闭包。在创建闭包的时候，我们通过line_conf的参数a,b说明了这两个变量的取值，这样，我们就确定了函数的最终形式(y = x + 1和y = 4x + 5)。我们只需要变换参数a,b，就可以获得不同的直线表达函数。由此，我们可以看到，闭包也具有提高代码可复用性的作用。

    如果没有闭包，我们需要每次创建直线函数的时候同时说明a,b,x。这样，我们就需要更多的参数传递，也减少了代码的可移植性。

### 总结

* 闭包似优化了变量，原来需要类对象完成的工作，闭包也可以完成
* 由于闭包引用了外部函数的局部变量，则外部函数的局部变量没有及时释放，消耗内存


## 装饰器

* 装饰器是程序开发中经常会用到的一个功能，用好了装饰器，开发效率如虎添翼，所以这也是Python面试中必问的问题，但对于好多初次接触这个知识的人来讲，这个功能有点绕，自学时直接绕过去了，然后面试问到了就挂了，因为装饰器是程序开发的基础知识，这个都不会，别跟人家说你会Python, 看了下面的文章，保证你学会装饰器。

        #### 第一波 ####
        def foo():
            print('foo')
        
        foo     #表示是函数
        foo()   #表示执行foo函数
        
        #### 第二波 ####
        def foo():
            print('foo')
        
        foo = lambda x: x + 1
        
        foo()   # 执行下面的lambda表达式，而不再是原来的foo函数，因为foo这个名字被重新指向了另外一个匿名函数

* 初创公司有N个业务部门，1个基础平台部门，基础平台负责提供底层的功能，如：数据库操作、redis调用、监控API等功能。业务部门使用基础功能时，只需调用基础平台提供的功能即可。如下：

        ############### 基础平台提供的功能如下 ###############

        def f1():
            print('f1')
        
        def f2():
            print('f2')
        
        def f3():
            print('f3')
        
        def f4():
            print('f4')
        
        ############### 业务部门A 调用基础平台提供的功能 ###############
        
        f1()
        f2()
        f3()
        f4()
        
        ############### 业务部门B 调用基础平台提供的功能 ###############
        
        f1()
        f2()
        f3()
        f4()

    目前公司有条不紊的进行着，但是，以前基础平台的开发人员在写代码时候没有关注验证相关的问题，即：基础平台的提供的功能可以被任何人使用。现在需要对基础平台的所有功能进行重构，为平台提供的所有功能添加验证机制，即：执行功能前，先进行验证。


    **老大把工作交给 Low B，他是这么做的：**

    跟每个业务部门交涉，每个业务部门自己写代码，调用基础平台的功能之前先验证。诶，这样一来基础平台就不需要做任何修改了。太棒了，有充足的时间泡妹子...
    
    当天Low B 被开除了…

    **老大把工作交给 Low BB，他是这么做的：**

        ############### 基础平台提供的功能如下 ############### 
        
        def f1():
            # 验证1
            # 验证2
            # 验证3
            print('f1')
        
        def f2():
            # 验证1
            # 验证2
            # 验证3
            print('f2')
        
        def f3():
            # 验证1
            # 验证2
            # 验证3
            print('f3')
        
        def f4():
            # 验证1
            # 验证2
            # 验证3
            print('f4')
        
        ############### 业务部门不变 ############### 
        ### 业务部门A 调用基础平台提供的功能### 
        
        f1()
        f2()
        f3()
        f4()
        
        ### 业务部门B 调用基础平台提供的功能 ### 
        
        f1()
        f2()
        f3()
        f4()
        
    过了一周 Low BB 被开除了…

    **老大把工作交给 Low BBB，他是这么做的：**
    
        只对基础平台的代码进行重构，其他业务部门无需做任何修改
        
        ############### 基础平台提供的功能如下 ############### 
        
        def check_login():
            # 验证1
            # 验证2
            # 验证3
            pass
        
        
        def f1():
        
            check_login()
        
            print('f1')
        
        def f2():
        
            check_login()
        
            print('f2')
        
        def f3():
        
            check_login()
        
            print('f3')
        
        def f4():
        
            check_login()
        
            print('f4')
            
    老大看了下Low BBB 的实现，嘴角漏出了一丝的欣慰的笑，语重心长的跟Low BBB聊了个天：

    **老大说：**

    写代码要遵循开放封闭原则，虽然在这个原则是用的面向对象开发，但是也适用于函数式编程，简单来说，它规定已经实现的功能代码不允许被修改，但可以被扩展，即：
    
    封闭：已实现的功能代码块
    开放：对扩展开发
    如果将开放封闭原则应用在上述需求中，那么就不允许在函数 f1 、f2、f3、f4的内部进行修改代码，老板就给了Low BBB一个实现方案：
    
        def w1(func):
            def inner():
                # 验证1
                # 验证2
                # 验证3
                func()
            return inner
        
        @w1
        def f1():
            print('f1')
            
        @w1
        def f2():
            print('f2')
            
        @w1
        def f3():
            print('f3')
            
        @w1
        def f4():
            print('f4')
            
    对于上述代码，也是仅仅对基础平台的代码进行修改，就可以实现在其他人调用函数 f1 f2 f3 f4 之前都进行【验证】操作，并且其他业务部门无需做任何操作。
    
    Low BBB心惊胆战的问了下，这段代码的内部执行原理是什么呢？
    
    老大正要生气，突然Low BBB的手机掉到地上，恰巧屏保就是Low BBB的女友照片，老大一看一紧一抖，喜笑颜开，决定和Low BBB交个好朋友。
    
    详细的开始讲解了：
    
    单独以f1为例：
    
        def w1(func):
            def inner():
                # 验证1
                # 验证2
                # 验证3
                func()
            return inner
        
        @w1
        def f1():
            print('f1')
            
    python解释器就会从上到下解释代码，步骤如下：
    
        def w1(func): ==>将w1函数加载到内存
        @w1
        
    没错， 从表面上看解释器仅仅会解释这两句代码，因为函数在 没有被调用之前其内部代码不会被执行。
    
    从表面上看解释器着实会执行这两句，但是 @w1 这一句代码里却有大文章， @函数名 是python的一种语法糖。

    上例@w1内部会执行一下操作：

    执行w1函数
    
    执行w1函数 ，并将 @w1 下面的函数作为w1函数的参数，即：@w1 等价于 w1(f1) 所以，内部就会去执行：
    
    def inner(): 
        #验证 1
        #验证 2
        #验证 3
        f1()     # func是参数，此时 func 等于 f1 
    return inner # 返回的 inner，inner代表的是函数，非执行函数 ,其实就是将原来的 f1 函数塞进另外一个函数中
    w1的返回值
    
    将执行完的w1函数返回值 赋值 给@w1下面的函数的函数名f1 即将w1的返回值再重新赋值给 f1，即：
    
    新f1 = def inner(): 
                #验证 1
                #验证 2
                #验证 3
                原来f1()
            return inner
    所以，以后业务部门想要执行 f1 函数时，就会执行 新f1 函数，在新f1 函数内部先执行验证，再执行原来的f1函数，然后将原来f1 函数的返回值返回给了业务调用者。
    
    如此一来， 即执行了验证的功能，又执行了原来f1函数的内容，并将原f1函数返回值 返回给业务调用着
    
    Low BBB 你明白了吗？要是没明白的话，我晚上去你家帮你解决吧！！



### 再议装饰器

* 定义函数：完成包裹数据

        def makeBold(fn):
            def wrapped():
                return "<b>" + fn() + "</b>"
            return wrapped

* 定义函数：完成包裹数据

        def makeItalic(fn):
            def wrapped():
                return "<i>" + fn() + "</i>"
            return wrapped
        
        @makeBold
        def test1():
            return "hello world-1"
        
        @makeItalic
        def test2():
            return "hello world-2"
        
        @makeBold
        @makeItalic
        def test3():
            return "hello world-3"
        
        print(test1()))
        print(test2()))
        print(test3()))

    运行结果:

        <b>hello world-1</b>
        <i>hello world-2</i>
        <b><i>hello world-3</i></b>

* 装饰器(decorator)功能

    引入日志
    函数执行时间统计
    执行函数前预备处理
    执行函数后清理功能
    权限校验等场景
    缓存

* 例1:无参数的函数

        from time import ctime, sleep
        
        def timefun(func):
            def wrappedfunc():
                print("%s called at %s"%(func.__name__, ctime()))
                func()
            return wrappedfunc
        
        @timefun
        def foo():
            print("I am foo")
        
        foo()
        sleep(2)
        foo()
    上面代码理解装饰器执行行为可理解成
        
        foo = timefun(foo)
        #foo先作为参数赋值给func后,foo接收指向timefun返回的wrappedfunc
        foo()
        #调用foo(),即等价调用wrappedfunc()
        #内部函数wrappedfunc被引用，所以外部函数的func变量(自由变量)并没有释放
        #func里保存的是原foo函数对象

* 例2:被装饰的函数有参数

        from time import ctime, sleep
        
        def timefun(func):
            def wrappedfunc(a, b):
                print("%s called at %s"%(func.__name__, ctime()))
                print(a, b)
                func(a, b)
            return wrappedfunc
        
        @timefun
        def foo(a, b):
            print(a+b)
        
        foo(3,5)
        sleep(2)
        foo(2,4)

* 例3:被装饰的函数有不定长参数

        from time import ctime, sleep
        
        def timefun(func):
            def wrappedfunc(\*args, \*\*kwargs):
                print("%s called at %s"%(func.__name__, ctime()))
                func(\*args, \*\*kwargs)
            return wrappedfunc
        
        @timefun
        def foo(a, b, c):
            print(a+b+c)
        
        foo(3,5,7)
        sleep(2)
        foo(2,4,9)

* 例4:装饰器中的return

        from time import ctime, sleep
        
        def timefun(func):
            def wrappedfunc():
                print("%s called at %s"%(func.__name__, ctime()))
                func()
            return wrappedfunc
        
        @timefun
        def foo():
            print("I am foo")
        
        @timefun
        def getInfo():
            return '----hahah---'
        
        foo()
        sleep(2)
        foo()
        
        
        print(getInfo())
    
    执行结果:
    
        foo called at Fri Nov  4 21:55:35 2016
        I am foo
        foo called at Fri Nov  4 21:55:37 2016
        I am foo
        getInfo called at Fri Nov  4 21:55:37 2016
        None
    
    如果修改装饰器为return func()，则运行结果：
        
        foo called at Fri Nov  4 21:55:57 2016
        I am foo
        foo called at Fri Nov  4 21:55:59 2016
        I am foo
        getInfo called at Fri Nov  4 21:55:59 2016
        ----hahah---

    总结：

    一般情况下为了让装饰器更通用，可以有return

* 例5:装饰器带参数,在原有装饰器的基础上，设置外部变量

        from time import ctime, sleep
        
        def timefun_arg(pre="hello"):
            def timefun(func):
                def wrappedfunc():
                    print("%s called at %s %s"%(func.__name__, ctime(), pre))
                    return func()
                return wrappedfunc
            return timefun
        
        @timefun_arg("itcast")
        def foo():
            print("I am foo")
        
        @timefun_arg("python")
        def too():
            print("I am too")
        
        foo()
        sleep(2)
        foo()
        
        too()
        sleep(2)
        too()
        可以理解为
        
        foo()==timefun_arg("itcast")(foo)()
    
* 例6：类装饰器（扩展，非重点）

    装饰器函数其实是这样一个接口约束，它必须接受一个callable对象作为参数，然后返回一个callable对象。在Python中一般callable对象都是函数，但也有例外。只要某个对象重写了 \_\_call\_\_() 方法，那么这个对象就是callable的。

        class Test():
            def \_\_call\_\_(self):
                print('call me!')
        
        t = Test()
        t()  # call me
        类装饰器demo


        class Test(object):
            def \_\_init\_\_(self, func):
                print("---初始化---")
                print("func name is %s"%func.\_\_name\_\_)
                self.\_\_func = func
            def \_\_call\_\_(self):
                print("---装饰器中的功能---")
                self.__func()
                
    说明：
    
    当用Test来装作装饰器对test函数进行装饰的时候，首先会创建Test的实例对象并且会把test这个函数名当做参数传递到__init__方法中即在__init__方法中的func变量指向了test函数体test函数相当于指向了用Test创建出来的实例对象当在使用test()进行调用时，就相当于让这个对象()，因此会调用这个对象的__call__方法为了能够在__call__方法中调用原来test指向的函数体，所以在__init__方法中就需要一个实例属性来保存这个函数体的引用所以才有了self.__func = func这句代码，从而在调用__call__方法中能够调用到test之前的函数体
    
            @Test
            def test():
                print("----test---")
            test()
            showpy()#如果把这句话注释，重新运行程序，依然会看到"--初始化--"
            运行结果如下：
                
            ---初始化---
            func name is test
            ---装饰器中的功能---
            ----test---


## 作用域

### 命名空间

* 比如有一个学校，有10个班级，在7班和8班中都有一个叫“小王”的同学，如果在学校的广播中呼叫“小王”时，7班和8班中的这2个人就纳闷了，你是喊谁呢！！！如果是“7班的小王”的话，那么就很明确了，那么此时的7班就是小王所在的范围，即命名空间


### globals, locals

* 在之前学习变量的作用域时，经常会提到局部变量和全局变量，之所有称之为局部、全局，就是因为他们的自作用的区域不同，这就是作用域

        # 查看全局变量
        print(globals())
        
        # 查看局部变量
        print(locals())

### LEGB 规则

* Python 使用 LEGB 的顺序来查找一个符号对应的对象

* locals，当前所在命名空间（如函数、模块），函数的参数也属于命名空间内的变量

* enclosing，外部嵌套函数的命名空间（闭包中常见）
        
        def fun1():
          a = 10
          def fun2():
              # a 位于外部嵌套函数的命名空间
              print(a)

* globals，全局变量，函数定义所在模块的命名空间

        a = 1
        def fun():
          # 需要通过 global 指令来声明全局变量
          global a
          # 修改全局变量，而不是创建一个新的 local 变量
          a = 2

* builtins，内建模块的命名空间。

    Python 在启动的时候会自动为我们载入很多内建的函数、类，
    比如 dict，list，type，print，这些都位于 \_\_builtin\_\_ 模块中，
    可以使用 dir(\_\_builtin\_\_) 来查看。
    这也是为什么我们在没有 import任何模块的情况下，
    就能使用这么多丰富的函数和功能了。
    
    在Python中，有一个内建模块，该模块中有一些常用函数;在Python启动后，
    且没有执行程序员所写的任何代码前，Python会首先加载该内建函数到内存。
    另外，该内建模块中的功能可以直接使用，不用在其前添加内建模块前缀，
    其原因是对函数、变量、类等标识符的查找是按LEGB法则，其中B即代表内建模块
    比如：内建模块中有一个abs()函数，其功能求绝对值，如abs(-20)将返回20。


## Python是动态语言

* 动态编程语言 是 高级程序设计语言 的一个类别，在计算机科学领域已被广泛应用。它是一类 在运行时可以改变其结构的语言 ：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。动态语言目前非常具有活力。例如JavaScript便是一个动态语言，除此之外如 PHP 、 Ruby 、 Python 等也都属于动态语言，而 C 、 C++ 等语言则不属于动态语言。

### 动态添加属性

    class Person(object):
        def __init__(self, name = None, age = None):
            self.name = name
            self.age = age
    
    p = Person("小明", "24")

在这里，我们定义了1个类Person，在这个类里，定义了两个初始属性name和age，但是人还有性别啊！如果这个类不是你写的是不是你会尝试访问性别这个属性呢？

    p.sex = "male"

这时候就发现问题了，我们定义的类里面没有sex这个属性啊！怎么回事呢？ 这就是动态语言的魅力和坑！ 这里 实际上就是 动态给实例绑定属性！

### 动态添加方法

    import types
    
    #定义了一个类
    class Person(object):
        num = 0
        def __init__(self, name = None, age = None):
            self.name = name
            self.age = age
        def eat(self):
            print("eat food")
    
    #定义一个实例方法
    def run(self, speed):
        print("%s在移动, 速度是 %d km/h"%(self.name, speed))
    
    #定义一个类方法
    @classmethod
    def testClass(cls):
        cls.num = 100
    
    #定义一个静态方法
    @staticmethod
    def testStatic():
        print("---static method----")
    
    #创建一个实例对象
    P = Person("老王", 24)
    #调用在class中的方法
    P.eat()
    
    #给这个对象添加实例方法
    P.run = types.MethodType(run, P)
    #调用实例方法
    P.run(180)
    
    #给Person类绑定类方法
    Person.testClass = testClass
    #调用类方法
    print(Person.num)
    Person.testClass()
    print(Person.num)
    
    #给Person类绑定静态方法
    Person.testStatic = testStatic
    #调用静态方法
    Person.testStatic()

## \_\_slots\_\_

* 现在我们终于明白了，动态语言与静态语言的不同

* 动态语言：可以在运行的过程中，修改代码

* 静态语言：编译时已经确定好代码，运行过程中不能修改

* 如果我们想要限制实例的属性怎么办？比如，只允许对Person实例添加name和age属性。

    为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的__slots__变量，来限制该class实例能添加的属性：

        class Person(object):
        __slots__ = ("name", "age")
        
        P = Person()
        P.name = "老王"
        P.age = 20
        P.score = 100

    **使用\_\_slots\_\_要注意，\_\_slots\_\_定义的属性仅对当前类实例起作用，对继承的子类是不起作用的**


## 元类

* 类也是对象

    在大多数编程语言中，类就是一组用来描述如何生成一个对象的代码段。在Python中这一点仍然成立：

        class ObjectCreator(object):
            pass
        
        my_object = ObjectCreator()
        print my_object

    但是，Python中的类还远不止如此。类同样也是一种对象。是的，没错，就是对象。只要你使用关键字class，Python解释器在执行的时候就会创建一个对象。

    下面的代码段：

        class ObjectCreator(object):
            pass

    将在内存中创建一个对象，名字就是ObjectCreator。这个对象（类对象ObjectCreator）拥有创建对象（实例对象）的能力。但是，它的本质仍然是一个对象，于是乎你可以对它做如下的操作：

    你可以将它赋值给一个变量  
    你可以拷贝它   
    你可以为它增加属性   
    你可以将它作为函数参数进行传递   


        print ObjectCreator     # 你可以打印一个类，因为它其实也是一个对象
    
        def echo(o):      
            print o
    
        echo(ObjectCreator)                 # 你可以将类做为参数传给函数
    
        print hasattr(ObjectCreator, 'new_attribute')
    
        ObjectCreator.new_attribute = 'foo' #  你可以为类增加属性
        print hasattr(ObjectCreator, 'new_attribute')
    
        print ObjectCreator.new_attribute
    
        ObjectCreatorMirror = ObjectCreator # 你可以将类赋值给一个变量
        print ObjectCreatorMirror()

* 动态地创建类

    因为类也是对象，你可以在运行时动态的创建它们，就像其他任何对象一样。首先，你可以在函数中创建类，使用class关键字即可。

        def choose_class(name):
            if name == 'foo':
                class Foo(object):
                    pass
                return Foo     # 返回的是类，不是类的实例
            else:
                class Bar(object):
                    pass
                return Bar
            
        MyClass = choose_class('foo')
        print MyClass              # 函数返回的是类，不是类的实例
        print MyClass()            # 你可以通过这个类创建类实例，也就是对象

    但这还不够动态，因为你仍然需要自己编写整个类的代码。由于类也是对象，所以它们必须是通过什么东西来生成的才对。当你使用class关键字时，Python解释器自动创建这个对象。但就和Python中的大多数事情一样，Python仍然提供给你手动处理的方法。

    还记得内建函数type吗？这个古老但强大的函数能够让你知道一个对象的类型是什么，就像这样：

        print type(1) # 数值的类型 <type 'int'>
        
        print type("1") # 字符串的类型 <type 'str'>
        
        print type(ObjectCreator()) # 实例对象的类型 <class '__main__.ObjectCreator'>
        
        print type(ObjectCreator) # 类的类型 <type 'type'>

    仔细观察上面的运行结果，发现使用type对ObjectCreator查看类型是，答案为type， 是不是有些惊讶。。。看下面


* 使用type创建类

    type还有一种完全不同的功能，动态的创建类。

    type可以接受一个类的描述作为参数，然后返回一个类。（要知道，根据传入参数的不同，同一个函数拥有两种完全不同的用法是一件很傻的事情，但这在Python中是为了保持向后兼容性）

    type可以像这样工作：

    type(类名, 由父类名称组成的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)

    比如下面的代码：

        class Test: #定义了一个Test类
             pass
        
        Test() #创建了一个Test类的实例对象

    可以手动像这样创建：

        Test2 = type("Test2",(),{}) #定了一个Test2类
        Test2() #创建了一个Test2类的实例对象
        
    我们使用"Test2"作为类名，并且也可以把它当做一个变量来作为类的引用。类和变量是不同的，这里没有任何理由把事情弄的复杂。即type函数中第1个实参，也可以叫做其他的名字，这个名字表示类的名字

        MyDogClass = type('MyDog', (), {})

        print MyDogClass
        
    使用help来测试这2个类

        help(Test) #用help查看Test类

        Help on class Test in module __main__:

        class Test(builtins.object)
         |  Data descriptors defined here:
         |
         |  __dict__
         |      dictionary for instance variables (if defined)
         |
         |  __weakref__
         |      list of weak references to the object (if defined)
         
        help(Test2) #用help查看Test2类
        
        Help on class Test2 in module __main__:
        
        class Test2(builtins.object)
         |  Data descriptors defined here:
         |
         |  __dict__
         |      dictionary for instance variables (if defined)
         |
         |  __weakref__
         |      list of weak references to the object (if defined)


* 使用type创建带有属性的类

    type 接受一个字典来为类定义属性，因此

        Foo = type('Foo', (), {'bar':True})

        可以翻译为：

        class Foo(object):
            bar = True
        
        并且可以将Foo当成一个普通的类一样使用：
        
        print Foo
        
        print Foo.bar
        f = Foo()
        print f
        print f.bar
        
        当然，你可以向这个类继承，所以，如下的代码：
        
        class FooChild(Foo):
            pass
        就可以写成：
        
        FooChild = type('FooChild', (Foo,),{})
        print FooChild
        print FooChild.bar   # bar属性是由Foo继承而来
        
    **注意：type的第2个参数，元组中是父类的名字，而不是字符串
    添加的属性是类属性，并不是实例属性**


* 使用type创建带有方法的类

    最终你会希望为你的类增加方法。只需要定义一个有着恰当签名的函数并将其作为属性赋值就可以了。

    添加实例方法

        def echo_bar(self): #定义了一个普通的函数
            print(self.bar)

        FooChild = type('FooChild', (Foo,), {'echo_bar': echo_bar}) #让FooChild类中的echo_bar属性，指向了上面定义的函数

        hasattr(Foo, 'echo_bar') #判断Foo类中，是否有echo_bar这个属性
        
        hasattr(FooChild, 'echo_bar') #判断FooChild类中，是否有echo_bar这个属性

        my_foo = FooChild()

        my_foo.echo_bar()

    添加静态方法

        @staticmethod
        def testStatic():
            print("static method ....")

        Foochild = type('Foochild', (Foo,), {"echo_bar":echo_bar, "testStatic":
        testStatic})

        fooclid = Foochild()

        fooclid.testStatic

        fooclid.testStatic()

        fooclid.echo_bar()
        
    添加类方法

        @classmethod
        def testClass(cls):
            print(cls.bar)

        Foochild = type('Foochild', (Foo,), {"echo_bar":echo_bar, "testStatic":
        testStatic, "testClass":testClass})

        fooclid = Foochild()

        fooclid.testClass()

    你可以看到，在Python中，类也是对象，你可以动态的创建类。这就是当你使用关键字class时Python在幕后做的事情，而这就是通过元类来实现的。


* 到底什么是元类（终于到主题了）

    元类就是用来创建类的“东西”。你创建类就是为了创建类的实例对象，不是吗？但是我们已经学习到了Python中的类也是对象。

    元类就是用来创建这些类（对象）的，元类就是类的类，你可以这样理解为：

        MyClass = MetaClass() #使用元类创建出一个对象，这个对象称为“类”
        MyObject = MyClass() #使用“类”来创建出实例对象

    你已经看到了type可以让你像这样做：

        MyClass = type('MyClass', (), {})

    这是因为函数type实际上是一个元类。type就是Python在背后用来创建所有类的元类。现在你想知道那为什么type会全部采用小写形式而不是Type呢？好吧，我猜这是为了和str保持一致性，str是用来创建字符串对象的类，而int是用来创建整数对象的类。type就是创建类对象的类。你可以通过检查__class__属性来看到这一点。Python中所有的东西，注意，我是指所有的东西——都是对象。这包括整数、字符串、函数以及类。它们全部都是对象，而且它们都是从一个类创建而来，这个类就是type。

        age = 35
        age.__class__ #<type 'int'>
        
        name = 'bob'
        name.__class__ #<type 'str'>
        
        def foo(): 
            pass
        foo.__class__ #<type 'function'>
        
        class Bar(object): 
            pass
        b = Bar()
        b.__class__ #<class '__main__.Bar'>

    现在，对于任何一个__class__的__class__属性又是什么呢？

        a.__class__.__class__ #<type 'type'>
        age.__class__.__class__ #<type 'type'>
        foo.__class__.__class__ #<type 'type'>
        b.__class__.__class__ #<type 'type'>

    因此，元类就是创建类这种对象的东西。type就是Python的内建元类，当然了，你也可以创建自己的元类。


* __metaclass__属性

    你可以在定义一个类的时候为其添加__metaclass__属性。

        class Foo(object):
            __metaclass__ = something…
    ...省略...
    如果你这么做了，Python就会用元类来创建类Foo。小心点，这里面有些技巧。你首先写下class Foo(object)，但是类Foo还没有在内存中创建。Python会在类的定义中寻找\_\_metaclass\_\_属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类。把下面这段话反复读几次。当你写如下代码时 :

        class Foo(Bar):
            pass

    Python做了如下的操作：
    
    Foo中有\_\_metaclass\_\_这个属性吗？如果是，Python会通过\_\_metaclass\_\_创建一个名字为Foo的类(对象)
    如果Python没有找到\_\_metaclass\_\_，它会继续在Bar（父类）中寻找\_\_metaclass\_\_属性，并尝试做和前面同样的操作。
    如果Python在任何父类中都找不到\_\_metaclass\_\_，它就会在模块层次中去寻找\_\_metaclass\_\_，并尝试做同样的操作。
    如果还是找不到\_\_metaclass\_\_,Python就会用内置的type来创建这个类对象。
    现在的问题就是，你可以在\_\_metaclass\_\_中放置些什么代码呢？答案就是：可以创建一个类的东西。那么什么可以用来创建一个类呢？type，或者任何使用到type或者子类化type的东东都可以。

* 自定义元类

    元类的主要目的就是为了当创建类时能够自动地改变类。通常，你会为API做这样的事情，你希望可以创建符合当前上下文的类。

    假想一个很傻的例子，你决定在你的模块里所有的类的属性都应该是大写形式。有好几种方法可以办到，但其中一种就是通过在模块级别设定__metaclass__。采用这种方法，这个模块中的所有类都会通过这个元类来创建，我们只需要告诉元类把所有的属性都改成大写形式就万事大吉了。

    幸运的是，__metaclass__实际上可以被任意调用，它并不需要是一个正式的类。所以，我们这里就先以一个简单的函数作为例子开始。

    python2中

        #-*- coding:utf-8 -*-
        def upper_attr(future_class_name, future_class_parents, future_class_attr):
        
            #遍历属性字典，把不是__开头的属性名字变为大写
            newAttr = {}
            for name,value in future_class_attr.items():
                if not name.startswith("__"):
                    newAttr[name.upper()] = value
        
            #调用type来创建一个类
            return type(future_class_name, future_class_parents, newAttr)
        
        class Foo(object):
            __metaclass__ = upper_attr #设置Foo类的元类为upper_attr
            bar = 'bip'
        
        print(hasattr(Foo, 'bar'))
        print(hasattr(Foo, 'BAR'))
        
        f = Foo()
        print(f.BAR)

    python3中

        #-*- coding:utf-8 -*-
        def upper_attr(future_class_name, future_class_parents, future_class_attr):
        
            #遍历属性字典，把不是__开头的属性名字变为大写
            newAttr = {}
            for name,value in future_class_attr.items():
                if not name.startswith("__"):
                    newAttr[name.upper()] = value
        
            #调用type来创建一个类
            return type(future_class_name, future_class_parents, newAttr)
        
        class Foo(object, metaclass=upper_attr):
            bar = 'bip'
        
        print(hasattr(Foo, 'bar'))
        print(hasattr(Foo, 'BAR'))
        
        f = Foo()
        print(f.BAR)
        现在让我们再做一次，这一次用一个真正的class来当做元类。
        
        #coding=utf-8
        
        class UpperAttrMetaClass(type):
            # __new__ 是在__init__之前被调用的特殊方法
            # __new__是用来创建对象并返回之的方法
            # 而__init__只是用来将传入的参数初始化给对象
            # 你很少用到__new__，除非你希望能够控制对象的创建
            # 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__
            # 如果你希望的话，你也可以在__init__中做些事情
            # 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用
            def __new__(cls, future_class_name, future_class_parents, future_class_attr):
                #遍历属性字典，把不是__开头的属性名字变为大写
                newAttr = {}
                for name,value in future_class_attr.items():
                    if not name.startswith("__"):
                        newAttr[name.upper()] = value
        
                # 方法1：通过'type'来做类对象的创建
                # return type(future_class_name, future_class_parents, newAttr)
        
                # 方法2：复用type.__new__方法
                # 这就是基本的OOP编程，没什么魔法
                # return type.__new__(cls, future_class_name, future_class_parents, newAttr)
        
                # 方法3：使用super方法
                return super(UpperAttrMetaClass, cls).__new__(cls, future_class_name, future_class_parents, newAttr)
    
    python2的用法
        
        class Foo(object):
            __metaclass__ = UpperAttrMetaClass
            bar = 'bip'

    python3的用法
        
        class Foo(object, metaclass = UpperAttrMetaClass):
            bar = 'bip'

        print(hasattr(Foo, 'bar'))
        # 输出: False
        print(hasattr(Foo, 'BAR'))
        # 输出:True
        
        f = Foo()
        print(f.BAR)
        # 输出:'bip'

    就是这样，除此之外，关于元类真的没有别的可说的了。但就元类本身而言，它们其实是很简单的：

    拦截类的创建
    修改类
    返回修改之后的类
    究竟为什么要使用元类？

    现在回到我们的大主题上来，究竟是为什么你会去使用这样一种容易出错且晦涩的特性？好吧，一般来说，你根本就用不上它：

    **“元类就是深度的魔法，99%的用户应该根本不必为此操心。如果你想搞清楚究竟是否需要用到元类，那么你就不需要它。那些实际用到元类的人都非常清楚地知道他们需要做什么，而且根本不需要解释为什么要用元类。” —— Python界的领袖 Tim Peters**

## 垃圾回收(一)

### 小整数对象池

* 整数在程序中的使用非常广泛，Python为了优化速度，使用了小整数对象池， 避免为整数频繁申请和销毁内存空间。

* Python 对小整数的定义是 [-5, 257) 这些整数对象是提前建立好的，不会被垃圾回收。在一个 Python 的程序中，所有位于这个范围内的整数使用的都是同一个对象.

* 同理，单个字母也是这样的。

* 当定义2个相同的字符串时，引用计数为0，触发垃圾回收

### 大整数对象池

* 每一个大整数，均创建一个新的对象。

### intern机制

    a1 = "HelloWorld"
    a2 = "HelloWorld"
    a3 = "HelloWorld"
    a4 = "HelloWorld"
    a5 = "HelloWorld"
    a6 = "HelloWorld"
    a7 = "HelloWorld"
    a8 = "HelloWorld"
    a9 = "HelloWorld"

python会不会创建9个对象呢？在内存中会不会开辟9个”HelloWorld”的内存空间呢？ 想一下，如果是这样的话，我们写10000个对象，比如a1=”HelloWorld”…..a1000=”HelloWorld”， 那他岂不是开辟了1000个”HelloWorld”所占的内存空间了呢？如果真这样，内存不就爆了吗？所以python中有这样一个机制——intern机制，让他只占用一个”HelloWorld”所占的内存空间。靠引用计数去维护何时释放。

### 总结

* 小整数[-5,257)共用对象，常驻内存
* 单个字符共用对象，常驻内存
* 单个单词，不可修改，默认开启intern机制，共用对象，引用计数为0，则销毁
* 字符串（含有空格），不可修改，没开启intern机制，不共用对象，引用计数为0，销毁 
* 大整数不共用内存，引用计数为0，销毁 
* 数值类型和字符串类型在 Python 中都是不可变的，这意味着你无法修改这个对象的值，每次对变量的修改，实际上是创建一个新的对象 

## 垃圾回收(二)

###Garbage collection(GC垃圾回收)

* 现在的高级语言如java，c#等，都采用了垃圾收集机制，而不再是c，c++里用户自己管理维护内存的方式。自己管理内存极其自由，可以任意申请内存，但如同一把双刃剑，为大量内存泄露，悬空指针等bug埋下隐患。 对于一个字符串、列表、类甚至数值都是对象，且定位简单易用的语言，自然不会让用户去处理如何分配回收内存的问题。 python里也同java一样采用了垃圾收集机制，不过不一样的是: **python采用的是引用计数机制为主，标记-清除和分代收集两种机制为辅的策略**

* 引用计数机制：

    python里每一个东西都是对象，它们的核心就是一个结构体：PyObject

        typedef struct_object {
            int ob_refcnt;
            struct_typeobject *ob_type;
        } PyObject;

    PyObject是每个对象必有的内容，其中ob_refcnt就是做为引用计数。当一个对象有新的引用时，它的ob_refcnt就会增加，当引用它的对象被删除，它的ob_refcnt就会减少

        #define Py_INCREF(op)   ((op)->ob_refcnt++) //增加计数
        #define Py_DECREF(op) \ //减少计数
            if (--(op)->ob_refcnt != 0) \
                ; \
            else \
                __Py_Dealloc((PyObject *)(op))

    当引用计数为0时，该对象生命就结束了。

    引用计数机制的优点：

    * 简单       
    * 实时性：一旦没有引用，内存就直接释放了。不用像其他机制等到特定时机。实时性还带来一个好处：处理回收内存的时间分摊到了平时。
    
    引用计数机制的缺点：

    * 维护引用计数消耗资源        
    * 循环引用     
        list1 = []     
        list2 = []     
        list1.append(list2)    
        list2.append(list1)     
        list1与list2相互引用，如果不存在其他对象对它们的引用，list1与list2的引用计数也仍然为1，所占用的内存永远无法被回收，这将是致命的。 对于如今的强大硬件，缺点1尚可接受，但是循环引用导致内存泄露，注定python还将引入新的回收机制。(标记清除和分代收集)

### 画说 Ruby 与 Python 垃圾回收

* 英文原文: [http://patshaughnessy.net/2013/10/24/visualizing-garbage-collection-in-ruby-and-python]()

* GC系统所承担的工作远比"垃圾回收"多得多。实际上，它们负责三个重要任务。它们

    * 为新生成的对象分配内存
    * 识别那些垃圾对象，并且
    * 从垃圾对象那回收内存

* 如果将应用程序比作人的身体：所有你所写的那些优雅的代码，业务逻辑，算法，应该就是大脑。以此类推，垃圾回收机制应该是那个身体器官呢？（我从RuPy听众那听到了不少有趣的答案：腰子、白血球 :) ）

* 我认为垃圾回收就是应用程序那颗跃动的心。像心脏为身体其他器官提供血液和营养物那样，垃圾回收器为你的应该程序提供内存和对象。如果心脏停跳，过不了几秒钟人就完了。如果垃圾回收器停止工作或运行迟缓,像动脉阻塞,你的应用程序效率也会下降，直至最终死掉。

## 垃圾回收(三)-gc模块

### 垃圾回收机制

* Python中的垃圾回收是以引用计数为主，分代收集为辅。

* 导致引用计数+1的情况

    * 对象被创建，例如a=23
    * 对象被引用，例如b=a
    * 对象被作为参数，传入到一个函数中，例如func(a)
    * 对象作为一个元素，存储在容器中，例如list1=[a,a]

* 导致引用计数-1的情况

    * 对象的别名被显式销毁，例如del a
    * 对象的别名被赋予新的对象，例如a=24
    * 一个对象离开它的作用域，例如f函数执行完毕时，func函数中的局部变量（全局变量不会）
    * 对象所在的容器被销毁，或从容器中删除对象

* 查看一个对象的引用计数

        import sys
        a = "hello world"
        sys.getrefcount(a)

    可以查看a对象的引用计数，但是比正常计数大1，因为调用函数的时候传入a，这会让a的引用计数+1

### 循环引用导致内存泄露

* 引用计数的缺陷是循环引用的问题

        import gc

        class ClassA():
            def __init__(self):
                print('object born,id:%s'%str(hex(id(self))))
        
        def f2():
            while True:
                c1 = ClassA()
                c2 = ClassA()
                c1.t = c2
                c2.t = c1
                del c1
                del c2
        
        #把python的gc关闭
        gc.disable()
        
        f2()

    执行f2()，进程占用的内存会不断增大。

    创建了c1，c2后这两块内存的引用计数都是1，执行c1.t=c2和c2.t=c1后，这两块内存的引用计数变成2.      
    
    在del c1后，内存1的对象的引用计数变为1，由于不是为0，所以内存1的对象不会被销毁，所以内存2的对象的引用数依然是2，在del c2后，同理，内存1的对象，内存2的对象的引用数都是1。      
    
    虽然它们两个的对象都是可以被销毁的，但是由于循环引用，导致垃圾回收器都不会回收它们，所以就会导致内存泄露。

### 垃圾回收

    #coding=utf-8
    import gc
    
    class ClassA():
        def __init__(self):
            print('object born,id:%s'%str(hex(id(self))))
        # def __del__(self):
        #     print('object del,id:%s'%str(hex(id(self))))
    
    def f3():
        print("-----0------")
        # print(gc.collect())
        c1 = ClassA()
        c2 = ClassA()
        c1.t = c2
        c2.t = c1
        print("-----1------")
        del c1
        del c2
        print("-----2------")
        print(gc.garbage)
        print("-----3------")
        print(gc.collect()) #显式执行垃圾回收
        print("-----4------")
        print(gc.garbage)
        print("-----5------")
    
    if __name__ == '__main__':
        gc.set_debug(gc.DEBUG_LEAK) #设置gc模块的日志
        f3()

* 有三种情况会触发垃圾回收：

    * 调用gc.collect()
    * 当gc模块的计数器达到阀值的时候
    * 程序退出的时候

### gc模块常用功能解析

* gc模块提供一个接口给开发者设置垃圾回收的选项。上面说到，采用引用计数的方法管理内存的一个缺陷是循环引用，而gc模块的一个主要功能就是解决循环引用的问题。

* 常用函数：

    * gc.set_debug(flags) 设置gc的debug日志，一般设置为gc.DEBUG_LEAK

    * gc.collect([generation]) 显式进行垃圾回收，可以输入参数，0代表只检查第一代的对象，1代表检查一，二代的对象，2代表检查一，二，三代的对象，如果不传参数，执行一个full collection，也就是等于传2。 返回不可达（unreachable objects）对象的数目
    
    * gc.get_threshold() 获取的gc模块中自动执行垃圾回收的频率。
    
    * gc.set_threshold(threshold0[, threshold1[, threshold2]) 设置自动执行垃圾回收的频率。
    
    * gc.get_count() 获取当前自动执行垃圾回收的计数器，返回一个长度为3的列表

* gc模块的自动垃圾回收机制

    必须要import gc模块，并且is_enable()=True才会启动自动垃圾回收。

    这个机制的主要作用就是发现并处理不可达的垃圾对象。

    垃圾回收 = 垃圾检查+垃圾回收

    在Python中，采用分代收集的方法。把对象分为三代，一开始，对象在创建的时候，放在一代中，如果在一次一代的垃圾检查中，改对象存活下来，就会被放到二代中，同理在一次二代的垃圾检查中，该对象存活下来，就会被放到三代中。

    gc模块里面会有一个长度为3的列表的计数器，可以通过gc.get_count()获取。

    例如(488,3,0)，其中488是指距离上一次一代垃圾检查，Python分配内存的数目减去释放内存的数目，注意是内存分配，而不是引用计数的增加。例如：

        print gc.get_count() # (590, 8, 0)
        a = ClassA()
        print gc.get_count() # (591, 8, 0)
        del a
        print gc.get_count() # (590, 8, 0)

    3是指距离上一次二代垃圾检查，一代垃圾检查的次数，同理，0是指距离上一次三代垃圾检查，二代垃圾检查的次数。

    gc模快有一个自动垃圾回收的阀值，即通过gc.get_threshold函数获取到的长度为3的元组，例如(700,10,10) 每一次计数器的增加，gc模块就会检查增加后的计数是否达到阀值的数目，如果是，就会执行对应的代数的垃圾检查，然后重置计数器

    例如，假设阀值是(700,10,10)：

    当计数器从(699,3,0)增加到(700,3,0)，gc模块就会执行gc.collect(0),即检查一代对象的垃圾，并重置计数器为(0,4,0)       
    当计数器从(699,9,0)增加到(700,9,0)，gc模块就会执行gc.collect(1),即检查一、二代对象的垃圾，并重置计数器为(0,0,1)      
    当计数器从(699,9,9)增加到(700,9,9)，gc模块就会执行gc.collect(2),即检查一、二、三代对象的垃圾，并重置计数器为(0,0,0)    

    **注意点:
    gc模块唯一处理不了的是循环引用的类都有\_\_del\_\_方法，所以项目中要避免定义\_\_del\_\_方法, object的del方法才是真正释放对象的内存空间**


## 内建属性








































