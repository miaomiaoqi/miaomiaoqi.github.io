---
layout: post
title:  "Python系统编程"
date:   2017-10-10 11:40:00
categories: Language
tags: Python
author: miaoqi
---

* content
{:toc}

## 进程

### 多任务

什么叫“多任务”呢？简单地说，就是操作系统可以同时运行多个任务。打个比方，你一边在用浏览器上网，一边在听MP3，一边在用Word赶作业，这就是多任务，至少同时有3个任务正在运行。还有很多任务悄悄地在后台同时运行着，只是桌面上没有显示而已。

现在，多核CPU已经非常普及了，但是，即使过去的单核CPU，也可以执行多任务。由于CPU执行代码都是顺序执行的，那么，单核CPU是怎么执行多任务的呢？

答案就是操作系统轮流让各个任务交替执行，任务1执行0.01秒，切换到任务2，任务2执行0.01秒，再切换到任务3，执行0.01秒……这样反复执行下去。表面上看，每个任务都是交替执行的，但是，由于CPU的执行速度实在是太快了，我们感觉就像所有任务都在同时执行一样。

真正的并行执行多任务只能在多核CPU上实现，但是，由于任务数量远远多于CPU的核心数量，所以，操作系统也会自动把很多任务轮流调度到每个核心上执行。

### 进程的创建-fork

* 进程 VS 程序

    编写完毕的代码，在没有运行的时候，称之为程序

    正在运行着的代码，就成为进程
    
    进程，除了包含代码以外，还有需要运行的环境等，所以和程序是有区别的

* fork()

    Python的os模块封装了常见的系统调用，其中就包括fork，可以在Python程序中轻松创建子进程：

        import os

        # 注意，fork函数，只在Unix/Linux/Mac上运行，windows不可以
        pid = os.fork()
    
        if pid == 0:
            print('哈哈1')
        else:
            print('哈哈2')	

    说明：

    程序执行到os.fork()时，操作系统会创建一个新的进程（子进程），然后复制父进程的所有信息到子进程中      
    然后父进程和子进程都会从fork()函数中得到一个返回值，在子进程中这个值一定是0，而父进程中是子进程的 id号        
    在Unix/Linux操作系统中，提供了一个fork()系统函数，它非常特殊。
    
    普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。
    
    子进程永远返回0，而父进程返回子进程的ID。
    
    这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。
            
* getpid()、getppid()

        import os

        rpid = os.fork()
        if rpid<0:
            print("fork调用失败。")
        elif rpid == 0:
            print("我是子进程（%s），我的父进程是（%s）"%(os.getpid(),os.getppid()))
            x+=1
        else:
            print("我是父进程（%s），我的子进程是（%s）"%(os.getpid(),rpid))
        
        print("父子进程都可以执行这里的代码")

### 多进程修改全局变量

    #coding=utf-8
    import os
    import time
    
    num = 0
    
    # 注意，fork函数，只在Unix/Linux/Mac上运行，windows不可以
    pid = os.fork()
    
    if pid == 0:
        num+=1
        print('哈哈1---num=%d'%num)
    else:
        time.sleep(1)
        num+=1
        print('哈哈2---num=%d'%num)

**多进程中，每个进程中所有数据（包括全局变量）都各有拥有一份，互不影响**

### 多次fork问题

* 如果在一个程序，有2次的fork函数调用，是否就会有3个进程呢？

        #coding=utf-8
        import os
        import time
        
        # 注意，fork函数，只在Unix/Linux/Mac上运行，windows不可以
        pid = os.fork()
        if pid == 0:
            print('哈哈1')
        else:
            print('哈哈2')
        
        pid = os.fork()
        if pid == 0:
            print('哈哈3')
        else:
            print('哈哈4')
        
        time.sleep(1)

### multiprocessing

* 如果你打算编写多进程的服务程序，Unix/Linux无疑是正确的选择。由于Windows没有fork调用，难道在Windows上无法用Python编写多进程的程序？

    由于Python是跨平台的，自然也应该提供一个跨平台的多进程支持。multiprocessing模块就是跨平台版本的多进程模块。

    multiprocessing模块提供了一个Process类来代表一个进程对象，下面的例子演示了启动一个子进程并等待其结束：

        #coding=utf-8
        from multiprocessing import Process
        import os
        
        # 子进程要执行的代码
        def run_proc(name):
            print('子进程运行中，name= %s ,pid=%d...' % (name, os.getpid()))
        
        if __name__=='__main__':
            print('父进程 %d.' % os.getpid())
            p = Process(target=run_proc, args=('test',))
            print('子进程将要执行')
            p.start()
            p.join()
            print('子进程已结束')

    * 创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动，这样创建进程比fork()还要简单。
    * join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

* Process语法结构如下：

    Process([group [, target [, name [, args [, kwargs]]]]])
    
    target：表示这个进程实例所调用对象；
    
    args：表示调用对象的位置参数元组；
    
    kwargs：表示调用对象的关键字参数字典；
    
    name：为当前进程实例的别名；
    
    group：大多数情况下用不到；
    
    Process类常用方法：
    
    is_alive()：判断进程实例是否还在执行；
    
    join([timeout])：是否等待进程实例执行结束，或等待多少秒；
    
    start()：启动进程实例（创建子进程）；
    
    run()：如果没有给定target参数，对这个对象调用start()方法时，就将执行对象中的run()方法；
    
    terminate()：不管任务是否完成，立即终止；
    
    Process类常用属性：
    
    name：当前进程实例别名，默认为Process-N，N为从1开始递增的整数；
    
    pid：当前进程实例的PID值；

* 进程的创建-Process子类

    创建新的进程还能够使用类的方式，可以自定义一个类，继承Process类，每次实例化这个类的时候，就等同于实例化一个进程对象，请看下面的实例：

        from multiprocessing import Process
        import time
        import os
        
        #继承Process类
        class Process_Class(Process):
            #因为Process类本身也有__init__方法，这个子类相当于重写了这个方法，
            #但这样就会带来一个问题，我们并没有完全的初始化一个Process类，所以就不能使用从这个类继承的一些方法和属性，
            #最好的方法就是将继承类本身传递给Process.__init__方法，完成这些初始化操作
            def __init__(self,interval):
                Process.__init__(self)
                self.interval = interval
        
            #重写了Process类的run()方法
            def run(self):
                print("子进程(%s) 开始执行，父进程为（%s）"%(os.getpid(),os.getppid()))
                t_start = time.time()
                time.sleep(self.interval)
                t_stop = time.time()
                print("(%s)执行结束，耗时%0.2f秒"%(os.getpid(),t_stop-t_start))
        
        if __name__=="__main__":
            t_start = time.time()
            print("当前程序进程(%s)"%os.getpid())        
            p1 = Process_Class(2)
            #对一个不包含target属性的Process类执行start()方法，就会运行这个类中的run()方法，所以这里会执行p1.run()
            p1.start()
            p1.join()
            t_stop = time.time()
            print("(%s)执行结束，耗时%0.2f"%(os.getpid(),t_stop-t_start))


### 进程池Pool

* 当需要创建的子进程数量不多时，可以直接利用multiprocessing中的Process动态成生多个进程，但如果是上百甚至上千个目标，手动的去创建进程的工作量巨大，此时就可以用到multiprocessing模块提供的Pool方法。

    初始化Pool时，可以指定一个最大进程数，当有新的请求提交到Pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到指定的最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来执行，请看下面的实例：

        from multiprocessing import Pool
        import os,time,random
        
        def worker(msg):
            t_start = time.time()
            print("%s开始执行,进程号为%d"%(msg,os.getpid()))
            #random.random()随机生成0~1之间的浮点数
            time.sleep(random.random()*2) 
            t_stop = time.time()
            print(msg,"执行完毕，耗时%0.2f"%(t_stop-t_start))
        
        po=Pool(3) #定义一个进程池，最大进程数3
        for i in range(0,10):
            #Pool.apply_async(要调用的目标,(传递给目标的参数元祖,))
            #每次循环将会用空闲出来的子进程去调用目标
            po.apply_async(worker,(i,))
        
        print("----start----")
        po.close() #关闭进程池，关闭后po不再接收新的请求
        po.join() #等待po中所有子进程执行完成，必须放在close语句之后
        print("-----end-----")

* apply堵塞式

        from multiprocessing import Pool
        import os,time,random
        
        def worker(msg):
            t_start = time.time()
            print("%s开始执行,进程号为%d"%(msg,os.getpid()))
            #random.random()随机生成0~1之间的浮点数
            time.sleep(random.random()*2) 
            t_stop = time.time()
            print(msg,"执行完毕，耗时%0.2f"%(t_stop-t_start))
        
        po=Pool(3) #定义一个进程池，最大进程数3
        for i in range(0,10):
            po.apply(worker,(i,))
        
        print("----start----")
        po.close() #关闭进程池，关闭后po不再接收新的请求
        po.join() #等待po中所有子进程执行完成，必须放在close语句之后
        print("-----end-----")

### 进程间通信-Queue

Process之间有时需要通信，操作系统提供了很多机制来实现进程间的通信。

* Queue的使用

    可以使用multiprocessing模块的Queue实现多进程之间的数据传递，Queue本身是一个消息列队程序，首先用一个小实例来演示一下Queue的工作原理：

        #coding=utf-8
        from multiprocessing import Queue
        q=Queue(3) #初始化一个Queue对象，最多可接收三条put消息
        q.put("消息1") 
        q.put("消息2")
        print(q.full())  #False
        q.put("消息3")
        print(q.full()) #True
        
        #因为消息列队已满下面的try都会抛出异常，第一个try会等待2秒后再抛出异常，第二个Try会立刻抛出异常
        try:
            q.put("消息4",True,2)
        except:
            print("消息列队已满，现有消息数量:%s"%q.qsize())
        
        try:
            q.put_nowait("消息4")
        except:
            print("消息列队已满，现有消息数量:%s"%q.qsize())
        
        #推荐的方式，先判断消息列队是否已满，再写入
        if not q.full():
            q.put_nowait("消息4")
        
        #读取消息时，先判断消息列队是否为空，再读取
        if not q.empty():
            for i in range(q.qsize()):
                print(q.get_nowait())

    初始化Queue()对象时（例如：q=Queue()），若括号中没有指定最大可接收的消息数量，或数量为负值，那么就代表可接受的消息数量没有上限（直到内存的尽头）；

    Queue.qsize()：返回当前队列包含的消息数量；
    
    Queue.empty()：如果队列为空，返回True，反之False ；
    
    Queue.full()：如果队列满了，返回True,反之False；
    
    Queue.get([block[, timeout]])：获取队列中的一条消息，然后将其从列队中移除，block默认值为True；
    
    1）如果block使用默认值，且没有设置timeout（单位秒），消息列队如果为空，此时程序将被阻塞（停在读取状态），直到从消息列队读到消息为止，如果设置了timeout，则会等待timeout秒，若还没读取到任何消息，则抛出"Queue.Empty"异常；
    
    2）如果block值为False，消息列队如果为空，则会立刻抛出"Queue.Empty"异常；
    
    Queue.get_nowait()：相当Queue.get(False)；
    
    Queue.put(item,[block[, timeout]])：将item消息写入队列，block默认值为True；
    
    1）如果block使用默认值，且没有设置timeout（单位秒），消息列队如果已经没有空间可写入，此时程序将被阻塞（停在写入状态），直到从消息列队腾出空间为止，如果设置了timeout，则会等待timeout秒，若还没空间，则抛出"Queue.Full"异常；
    
    2）如果block值为False，消息列队如果没有空间可写入，则会立刻抛出"Queue.Full"异常；
    
    Queue.put_nowait(item)：相当Queue.put(item, False)；


* Queue实例

    我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：

        from multiprocessing import Process, Queue
        import os, time, random
        
        # 写数据进程执行的代码:
        def write(q):
            for value in ['A', 'B', 'C']:
                print 'Put %s to queue...' % value
                q.put(value)
                time.sleep(random.random())
        
        # 读数据进程执行的代码:
        def read(q):
            while True:
                if not q.empty():
                    value = q.get(True)
                    print 'Get %s from queue.' % value
                    time.sleep(random.random())
                else:
                    break
        
        if __name__=='__main__':
            # 父进程创建Queue，并传给各个子进程：
            q = Queue()
            pw = Process(target=write, args=(q,))
            pr = Process(target=read, args=(q,))
            # 启动子进程pw，写入:
            pw.start()    
            # 等待pw结束:
            pw.join()
            # 启动子进程pr，读取:
            pr.start()
            pr.join()
            # pr进程里是死循环，无法等待其结束，只能强行终止:
            print ''
            print '所有数据都写入并且读完'


* 进程池中的Queue

    如果要使用Pool创建进程，就需要使用multiprocessing.Manager()中的Queue()，而不是multiprocessing.Queue()，否则会得到一条如下的错误信息：

    RuntimeError: Queue objects should only be shared between processes through inheritance.

        #coding=utf-8

        #修改import中的Queue为Manager
        from multiprocessing import Manager,Pool
        import os,time,random
        
        def reader(q):
            print("reader启动(%s),父进程为(%s)"%(os.getpid(),os.getppid()))
            for i in range(q.qsize()):
                print("reader从Queue获取到消息：%s"%q.get(True))
        
        def writer(q):
            print("writer启动(%s),父进程为(%s)"%(os.getpid(),os.getppid()))
            for i in "dongGe":
                q.put(i)
        
        if __name__=="__main__":
            print("(%s) start"%os.getpid())
            q=Manager().Queue() #使用Manager中的Queue来初始化
            po=Pool()
            #使用阻塞模式创建进程，这样就不需要在reader中使用死循环了，可以让writer完全执行完成后，再用reader去读取
            po.apply(writer,(q,))
            po.apply(reader,(q,))
            po.close()
            po.join()
            print("(%s) End"%os.getpid())


## 线程
		
		
		
		
		
		
	
		
		
		
 		
		
	
	
	

	
	
	
	
	
	
	
	
	
	
		
		
		
		
	
	
		
		








































