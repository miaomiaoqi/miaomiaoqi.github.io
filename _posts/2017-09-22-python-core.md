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




























