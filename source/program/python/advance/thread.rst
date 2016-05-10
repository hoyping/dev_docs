threading模块
=================
    我们应该避免使用thread模块，原因是它不支持守护线程。当主线程退出时，所有的子线程不论它们是否还在工作，都会被强行退出。有时我们并不期望这种行为，这时就引入了守护线程的概念。threading模块则支持守护线程。

常用方法
--------

join（[timeout]）
+++++++++++++++++
    如果一个线程或者一个函数在执行过程中要调用另外一个线程，并且待到其完成以后才能接着执行，那么在调用这个线程时可以使用被调用线程的join方法。

    timeout参数是可选的，代表线程运行的最大时间，即如果超过这个时间，不管这个线程有没有执行完毕，主线程或函数都会接着执行的。

setDaemon()
++++++++++++++++

    这个方法基本和join是相反的。当我们在程序运行中，执行一个主线程，如果主线程又创建一个子线程，主线程和子线程就分兵两路，分别运行，那么当主线程完成想退出时，会检验子线程是否完成。如果子线程未完成，则主线程会等待子线程完成后再退出。但是有时候我们需要的是，只要主线程完成了，不管子线程是否完成，都要和主线程一起退出，这时就可以用setDaemon方法啦

isAlive()
++++++++++

    当线程创建以后，可以使用Thread对象的isAlive方法查看线程是否运行

线程名
++++++++++

    当线程创建后可以设置线程名来区分不同的线程，以便对线程进行控制。线程名可以在类的初始化函数中定义，也可以使用Thread对象的setName方法设置。下面是不同的方法来设置线程名。::

    >>> import threading
    >>> class mythread(threading.Thread):
        def __init__(self,threadname):
            threading.Thread.__init__(self,name=threadname)
        def run(self):
            print self.getName()

    >>>
    >>> t1=mythread('t1')
    >>> t1.getName()
    't1'


GIL
====
GIL是什么
-----------

    首先需要明确的一点是 GIL 并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。就好比C++是一套语言（语法）标准，但是可以用不同的编译器来编译成可执行代码。有名的编译器例如GCC，INTEL C++，Visual C++等。Python也一样，同样一段代码可以通过CPython，PyPy，Psyco等不同的Python执行环境来执行。像其中的JPython就没有GIL。然而因为CPython是大部分环境下默认的Python执行环境。所以在很多人的概念里CPython就是Python，也就想当然的把 GIL 归结为Python语言的缺陷。所以这里要先明确一点：GIL并不是Python的特性，Python完全可以不依赖于GIL

    那么CPython实现中的GIL又是什么呢？GIL全称 Global Interpreter Lock 为了避免误导，我们还是来看一下官方给出的解释：

    In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple native threads from executing Python bytecodes at once. This lock is necessary mainly because CPython’s memory management is not thread-safe. (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)

    好吧，是不是看上去很糟糕？一个防止多线程并发执行机器码的一个Mutex，乍一看就是个BUG般存在的全局锁嘛！别急，我们下面慢慢的分析。

为什么会有GIL
--------------

    由于物理上得限制，各CPU厂商在核心频率上的比赛已经被多核所取代。为了更有效的利用多核处理器的性能，就出现了多线程的编程方式，而随之带来的就是线程间数据一致性和状态同步的困难。 即使在CPU内部的Cache也不例外 ，为了有效解决多份缓存之间的数据同步时各厂商花费了不少心思，也不可避免的带来了一定的性能损失。

    Python当然也逃不开，为了利用多核，Python开始支持多线程。 而解决多线程之间数据完整性和状态同步的最简单方法自然就是加锁。 于是有了GIL这把超级大锁，而当越来越多的代码库开发者接受了这种设定后，他们开始大量依赖这种特性（即默认python内部对象是thread-safe的，无需在实现时考虑额外的内存锁和同步操作）。

    慢慢的这种实现方式被发现是蛋疼且低效的。但当大家试图去拆分和去除GIL的时候，发现大量库代码开发者已经重度依赖GIL而非常难以去除了。有多难？做个类比，像MySQL这样的“小项目”为了把Buffer Pool Mutex这把大锁拆分成各个小锁也花了从5.5到5.6再到5.7多个大版为期近5年的时间，本且仍在继续。MySQL这个背后有公司支持且有固定开发团队的产品走的如此艰难，那又更何况Python这样核心开发和代码贡献者高度社区化的团队呢？

    所以简单的说GIL的存在更多的是历史原因。如果推到重来，多线程的问题依然还是要面对，但是至少会比目前GIL这种方式会更优雅。

GIL的影响
------------

从上文的介绍和官方的定义来看，GIL无疑就是一把全局排他锁。毫无疑问全局锁的存在会对多线程的效率有不小影响。甚至就几乎等于Python是个单线程的程序。那么读者就会说了，全局锁只要释放的勤快效率也不会差啊。只要在进行耗时的IO操作的时候，能释放GIL，这样也还是可以提升运行效率的嘛。或者说再差也不会比单线程的效率差吧。理论上是这样，而实际上呢？Python比你想的更糟。

下面我们就对比下Python在多线程和单线程下得效率对比。测试方法很简单，一个循环1亿次的计数器函数。一个通过单线程执行两次，一个多线程执行。最后比较执行总时间。测试环境为双核的Mac pro。注：为了减少线程库本身性能损耗对测试结果带来的影响，这里单线程的代码同样使用了线程。只是顺序的执行两次，模拟单线程。


当前GIL设计的缺陷
--------------------
基于pcode数量的调度方式

    按照Python社区的想法，操作系统本身的线程调度已经非常成熟稳定了，没有必要自己搞一套。所以Python的线程就是C语言的一个pthread，并通过操作系统调度算法进行调度（例如linux是CFS）。为了让各个线程能够平均利用CPU时间，python会计算当前已执行的微代码数量，达到一定阈值后就强制释放GIL。而这时也会触发一次操作系统的线程调度（当然是否真正进行上下文切换由操作系统自主决定）。

::
    while True:
        acquire GIL
        for i in 1000:
            do something
        release GIL
        /* Give Operating System a chance to do thread scheduling */


 这种模式在只有一个CPU核心的情况下毫无问题。任何一个线程被唤起时都能成功获得到GIL（因为只有释放了GIL才会引发线程调度）。但当CPU有多个核心的时候，问题就来了。从伪代码可以看到，从 release GIL 到 acquire GIL 之间几乎是没有间隙的。所以当其他在其他核心上的线程被唤醒时，大部分情况下主线程已经又再一次获取到GIL了。这个时候被唤醒执行的线程只能白白的浪费CPU时间，看着另一个线程拿着GIL欢快的执行着。然后达到切换时间后进入待调度状态，再被唤醒，再等待，以此往复恶性循环。

   PS：当然这种实现方式是原始而丑陋的，Python的每个版本中也在逐渐改进GIL和线程调度之间的互动关系。例如先尝试持有GIL在做线程上下文切换，在IO等待时释放GIL等尝试。但是无法改变的是GIL的存在使得操作系统线程调度的这个本来就昂贵的操作变得更奢侈了。
