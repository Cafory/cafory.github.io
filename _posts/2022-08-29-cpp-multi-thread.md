---
title: C++ 多线程
tags: C++
---



本文介绍C++多线程相关内容。

<!--more-->

### 1 多线程相关概念

+ 线程

    线程是进程中的一个实体，是被系统独立分配和调度的基本单位。也就是说线程是CPU可执行调度的最小单位。引入线程之后将进程的两个基本属性分开了，线程作为调度和分配的基本单位，进程作为独立分配资源的单位。线程基本上不拥有资源，只拥有一点运行中必不可少的资源，在同一个进程中的所有线程都共享地址空间，线程间的大部分数据可以共享并且相比多进程来说，多线程间的通信开销小、启动速度快、占用资源少。

+ 多线程并发

    多线程是实现并发(双核的真正并行或者单核机器的任务切换都叫并发）的一种手段，多线程并发即多个线程同时执行,一般而言，多线程并发就是把一个任务拆分为多个子任务，然后交由不同线程处理不同子任务,使得这多个子任务同时执行。

    为什么要进行多线程并发呢？主要是用于任务拆分和提高性能：将程序划分成不同的任务，每个线程执行一个或多个任务，可以将整个程序的逻辑变的更加清晰；任务之间各自并发降低运算时间。

    并发与并行有什么区别呢？首先串行很容易理解就是说所有任务都按先后顺序执行。而并行和并发的区别在于多个任务是否同时执行。并发是指在**同一时间段内，能够处理多个事件**，在并发程序中可以同时拥有两个或者多个线程。这意味着，如果程序在单核处理器上运行，那么这两个线程将交替地换入或者换出内存。这些线程是同时“存在”的——每个线程都处于执行过程中的某个状态。 并行是指在**同一时刻上，能够处理多个事件**。如果程序能够并行执行，那么就一定是运行在多核处理器上。此时程序中的每个线程都将分配到一个独立的处理器核上，因此可以同时运行。

+ 多线程同步与互斥

    多个线程同时执行任务肯定存在线程间的同步和互斥：

    + **线程同步**：指线程之间所具有的一种制约关系，一个线程的执行依赖另外一个线程的消息，当它没有得到另一个线程的消息时应等待，直到消息到达时才被唤醒；

    + **线程互斥**: 指对于共享的进程系统资源，每个线程访问时的排他性。当有若干个线程都要使用某一个共享资源时，任何时刻最多只允许一个线程去使用，其他线程必须等待，知道占用占用资源者释放该资源。线程互斥可以看成是一种特殊的线程同步。

    线程间的同步方法大体可以分为两类:

    1. 用户模式(使用时不需要切换内核态，只在用户态完成操作)：

        **临界区**：适合一个进程内的多线程访问公共区域或代码段时使用

    2. 内核模式(利用系统内核对象的单一性来进行同步，使用时需要切换内核态与用户态)：

        **事件**：通过线程间触发事件实现同步互斥

        **互斥量**：适合不同进程内多线程访问公共区域或代码段时使用，与临界区相似

        **信号量**：与临界区和互斥量不同，可以实现多个线程同时访问公共区域数据，原理与操作系统中PV操作类似，先设置一个访问公共区域的线程最大连接数，每有一个线程访问共享区资源数就减一，直到资源数小于等于零。

+ C++多线程并发
    
    简单情况下，实现C++多线程并发程序的思路如下：将任务的不同功能交由多个函数分别实现，创建多个线程，每个线程执行一个函数，一个任务就这样同时分由不同线程执行了。



### 2 C++ 多线程编程

+ C++11中提供了thread库管理线程、保护共享数据、线程间同步等功能。头文件是`#include<thread>`。一个进程至少要有一个线程，在C++中可以认为main函数就是我们的主线程。而在创建thread对象的时候，就是在这个线程之外创建了一个独立的子线程。只要创建了这个子线程并且开始运行了，主线程就完全和它没有关系了，不知道CPU会什么时候调度它运行，什么时候结束运行。

+ 线程的创建方式



    **1 调用thread类去创建线程对象**

        头文件与子线程处理函数如下，所有代码一致：

    ```cpp
    #include <iostream>
    #include <thread>

    using namespace std;

    void print()
    {
        Sleep(3000);
        cout<<"====== start new thread ======"
    }
    ```

    1.1 使用`join()`函数加入，汇合线程、阻塞主线程，等待子线程结束，才会回到主线程。

    ```cpp
    int main()
    {
        thread t1(print);
        t1.join();  // 阻塞主进程，
        // 只有当t1子线程执行完毕后，才会继续执行主线程
        cout<<"====== main thread is running ======"<<endl;
    }
    ```

    运行结果如下：

    ```shell
    【程序启动3s之后】
    ====== start new thread ======
    ====== main thread is running ======
    ```

    如果创建一个线程而不做处理，会调用`abort`函数中止程序。一个线程也只能`join`一次，否则也会`abort`。


    1.2 使用`detach()`函数，打破依赖关系，把子线程驻留后台

    detach用于主线程和当前线程分离，主线程可以先执行结束，如果主线程执行完了，子线程会在C++后台运行，一旦使用detach，与这个子线程关联的对象会失去对这个主线程的关联，此时这个子线程会驻留在C++后台运行，当主线程执行完毕结束，子线程会移交给C++运行时库管理，这个运行时库会清理与这个线程相关的资源（守护线程），detach会使子线程失去进程的控制;

    ```cpp
    int main()
    {
        thread t1(print);
        t1.detach();  // 主线程子线程分离
        sleep(4);
        cout<<"====== main thread is running ======"<<endl;
    }
    ```

    运行结果如下：

    ```shell
    【程序启动3s之后】
    ====== start new thread ======
    【程序启动4s之后】
    ====== main thread is running ======
    ```

    1.3 使用joinable()函数判断当前线程是否可以做join或者detach操作，若可以，返回true。

    ```cpp
    int main()
    {
        thread t1(print);
        t1.detach();  // 主线程子线程分离
        cout<<"====== main thread is running ======"<<endl;
        if( t1.joinable() )
            t1.join();
        else
            cout<<"thread t1 is running"<<endl;
    }
    ```

    输出如下：

    ```shell
    ====== main thread is running ======
    thread t1 is running
    ```

    **2 通过类和对象创建线程**

    ```cpp
    class Test
    {
    public:
        // STL 仿函数
        void operator()()
        {
            sleep(3); // unistd.h
            cout << "====== start new thread ======" << endl;
        }
    };

    int main()
    {
        //正常写法:对象充当线程处理函数
        Test test;
        thread t1(test);
        t1.join(); 
        
        //Test();
        thread t2( (Test() )); //这里如果不多写一个括号，
        // 编译器就会把t2解析成一个函数，Test()解析成一个参数，从而出错
        t2.join();
    }
    ```

    **3 带参的方式创建线程**

    ```cpp
    void print(int & num)
    {
        cout<<"thread get params value is: "<< num << endl;
    }

    int main()
    {
        int num = 10;
        //ref 用于包装 “引用传递值”
        // 此处必须要用 std::ref
        thread t1( print, ref(num) );
        t1.join();
        cout<<"main thread end"<<endl;
    }
    ```

    **4 Lambda表达式创建线程**

    ```cpp
    int main()
    {
        thread t1( [](){cout<<"new thread has joint<<endl"<<endl;} );
        t1.join();
        cout<<"main thread end"<<endl;
    }
    ```

    **5 以智能指针为参数创建线程**

    ```cpp
    void print(unique_ptr<int> ptr)
    {
        cout<<"thread get params value is: "<< *ptr << endl;
    }

    int main()
    {
        int * p = new int(10);
        unique_ptr<int> ptr(p);
        thread t1( print, move(ptr) ); //move移动语义，
        //通俗的说就是把ptr移动到子线程中了，
        //这样的话，主线程中的ptr就没了 
        t1.join();
        cout<<"main thread end"<<endl;
    }
    ``` 

    **6 以类的成员函数充当线程处理函数来创建线程**

    ```cpp
    class Test
    {
    public:
        void print(int & num)
        {
            cout << "thread get params value is: " << num << endl;
        }
    };

    int main()
    {
        int num = 10;
        Test test;
        //需要说明 是哪个对象
        thread t1(&Test::print, test, ref(num));
        t1.join();
        cout << "main thread end" << endl;
    }
    ```




