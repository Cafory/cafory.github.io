---
title: C++ 智能指针
tags: C++
---

<!--more-->

# C++ 智能指针

### 0 堆 与 栈 ： RAII

+ 在程序中定义一个变量，它会被放到以下内存中：
    + 如果进行动态分配，放入堆中（例如使用 `new`）
    + 如果无动态分配，放入栈中

+ 栈中变量当生命周期结束时，会自动释放内存，而堆中变量需要手动释放

+ 例子：
    ```cpp
    class A
    {
    public:
        A(string _s):s(_s){}
        ~A()
        {
            cout<<"析构"<<s<<endl;
        }
    private:
        string s;
        
    };

    void test()
    {
        A * a = new A("动态分配");
        A b("非动态分配");
    }

    int main(int argc,char* argv[])
    {
        int a;
        test();
        cin>>a;
        return 0;
    }
    ```

+ RAII机制： RAII是Resource Acquisition Is Initialization（wiki上面翻译成 “资源获取就是初始化”）的简称，是C++语言的一种管理资源、避免泄漏的惯用法。利用的就是C++构造的对象最终会被销毁的原则。RAII的做法是使用一个对象，在其构造时获取对应的资源，在对象生命期内控制对资源的访问，使之始终保持有效，最后在对象析构的时候，释放构造时获取的资源。

+ RAII的实现原理
    + 栈上变量生命周期结束时自动释放内存
    + C++ 类对象析构函数的自动调用

    当我们在一个函数内部使用局部变量，当退出了这个局部变量的作用域时，这个变量也就别销毁了；当这个变量是类对象时，这个时候，就会自动调用这个类的析构函数，而这一切都是自动发生的，不要程序员显示的去调用完成。RAII便是这样做的。

    我们使用类封装资源，然后类内操作资源，使用的时候只须定义此类的变量即可，当生命周期结束时，便会自动调用析构函数，释放其内的资源。


### 1 普通指针的问题
+ 普通指针一般都放在堆上，需要手动释放内存；
+ 忘记释放资源，导致资源泄漏(常发生内存泄漏);
+ 同一资源释放多次，导致释放野指针(程序崩溃);
+ 明明程序代码在后面写了释放资源的代码，但由于程序的逻辑，从中间return回去，导致释放资源的代码并未执行；
+ 代码运行过程中发生异常，随着异常栈展开，导致释放资源的代码未执行；

### 2 智能指针

+ 智能指针是利用RAII机制 （**栈上的对象出作用域自动析构的特征来做到资源的自动释放**），把资源释放的代码放到这个对象的析构函数中；用户可以不关注资源如何释放，无论程序逻辑如何运行，正常执行或者异常执行，资源在到期的情况下，一定会释放；

+ 面试题：可以把智能指针放到堆上吗？智能指针一般定义在栈上(利用栈中对象出作用域自动析构特点)

+ 智能指针的作用：C++程序设计中使用堆内存是非常频繁的操作，堆内存的申请和释放都由程序员自己管理。程序员自己管理堆内存可以提高程序的效率，但是整体来说堆内存的管理是麻烦的，C++11中引入了智能指针的概念，方便管理堆内存。使用普通指针，容易造成堆内存泄露（忘记释放），二次释放，程序发生异常时内存泄露等问题等，使用智能指针能更好的管理堆内存。

+ 智能指针的原理：智能指针(smart pointer)的通用实现技术是使用引用计数(reference count)。智能指针类将一个计数器与类指向的对象相关联，引用计数跟踪该类有多少个对象的指针指向同一对象。每次创建类的新对象时，初始化指针就将引用计数置为1；当对象作为另一对象的副本而创建时，拷贝构造函数拷贝指针并增加与之相应的引用计数；对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数；调用析构函数时，析构函数减少引用计数（如果引用计数减至0，则删除基础对象）。

### 3 unique_ptr

+ unique_ptr是独享被管理对象指针所有权(owership)的智能指针。unique_ptr对象封装一个原始指针，并负责其生命周期。当该对象被销毁时，会在其析构函数中删除关联的原始指针。

+ “唯一”拥有其所指对象，同一时刻只能有一个unique_ptr指向给定对象（禁止拷贝、赋值），可以释放所有权，转移所有权。

+ unique_ptr是将左值引用的拷贝构造函数和拷贝赋值运算符删除了，外部不能调用（等同于私有化），防止智能指针浅拷贝问题的发生；但unique_ptr提供了带右值引用的拷贝构造函数和拷贝赋值运算符，这样临时对象就可以使用了，非常有用；

+ 例子

    ```cpp
    int main(int argc,char* argv[])
    {
        int * a = new int;
        *a = 10;
        unique_ptr<int> p(a);
        unique_ptr<int> q;
        //q = a; 报错，unique_ptr删除拷贝赋值函数

        q  = move(p); // 资源转移

        //cout<<*p; 报错, 此时 p 为nullptr，因为资源转移给了q
        cout<<*q; // 输出 10

        return 0;
    }
    ```

### 4 shared_ptr

+ shared_ptr的原理是使用引用计数实现对同一块内存的多个引用。在最后一个引用被释放时，指向的内存才释放，这也是和 unique_ptr 最大的区别。当对象的所有权需要共享(shared_ptr)时，shared_ptr可以进行赋值拷贝。

+ shared_ptr使用引用计数（计算的时同一个shared_ptr的引用个数，而不是原始指针的引用个数），每一个shared_ptr的拷贝都指向相同的内存。每使用他一次，内部的引用计数加1，每析构一次，内部的引用计数减1，减为0时，删除所指向的堆内存。

+ 例子
    ```cpp
    int main(int argc,char* argv[])
    {
        // shared_ptr<int> p = new int; 错误使用
        // 以下为正确赋值，shared_ptr 重写了拷贝构造与拷贝赋值
        shared_ptr<int> sp = make_shared<int>(1);
        shared_ptr<int> sp2(sp); 
        shared_ptr<int> sp3 = sp;
        int * a = new int(5);
        shared_ptr<int> sp4(a);

        int * ori_sp = sp.get() ; // get方法可以获取原始指针
        
        cout<<sp.use_count(); // 所有同一shared_ptr的拷贝指向同一个
        cout<<sp2.use_count(); //内存，use_count返回shared_ptr的引用个数
        
        return 0;
    }
    ```

+ 不能用同一原始指针初始化多个shared_ptr, 

    ```cpp
    void test() {
        int *p0 = new int(1);
        shared_ptr<int> p1(p0);
        shared_ptr<int> p2(p0);
        cout<<*p1<<endl;
    }
    // 执行异常报错，重复析构p0
    ```

+ 循环引用

    ```cpp
    struct Son
    {
        shared_ptr<Father> father_;
    };

    struct Father
    {
        shared_ptr<Son> son_;
    };



    int main(int argc,char* argv[])
    {
        auto father = make_shared<Father>();
        auto son = make_shared<Son>();

        father->son_ = son;
        son->father_ = father;
        
        return 0;
    }
    ```

    该部分代码会有内存泄漏问题。原因是:
    > 1.main 函数退出之前，Father 和 Son 对象的引用计数都是2 。
    >
    > 2.son 指针销毁，这时 Son 对象的引用计数是 1。
    >
    > 3.father 指针销毁，这时 Father 对象的引用计数是 1。
    >
    > 4.由于 Father 对象和 Son 对象的引用计数都是 1，这两个对象都不会被销毁，从而发生内存泄露。

### 5 weak_ptr

+ weak_ptr:弱引用指针，不会改变资源的引用计数，辅助shared_ptr工作的，防止循环引用。

+ 弱智能指针只是资源的观察者，它是不能直接使用资源的，不会增加引用计数，它没有提供*,->运算符重载函数;要通过它访问资源必须通过lock方法，得到一个强智能指针，然后就可以使用资源了；

+ 为避免循环引用导致的内存泄露，就需要使用 weak_ptr。weak_ptr 并不拥有其指向的对象，也就是说，让 weak_ptr 指向 shared_ptr 所指向对象，对象的引用计数并不会增加。使用 weak_ptr 就能解决前面提到的循环引用的问题，方法很简单，只要让 Son 或者 Father 包含的 shared_ptr 改成 weak_ptr 就可以了。

    ```cpp
    struct Father
    {
        shared_ptr<Son> son_;
    };

    struct Son
    {
        weak_ptr<Father> father_;
    };

    int main()
    {
        auto father = make_shared<Father>();
        auto son = make_shared<Son>();

        father->son_ = son;
        son->father_ = father;

        return 0;
    }

    ```

    > 1.main 函数退出前，Son 对象的引用计数是 2，而 Father 的引用计数是 1。
    >
    > 2.son 指针销毁，Son 对象的引用计数变成 1。
    >
    > 3.father 指针销毁，Father 对象的引用计数变成 0，导致 Father 对象析构，Father 对象的析构会导致它包含的 son_ 指针被销毁，这时 Son 对象的引用计数变成 0，所以 Son 对象也会被析构。

+ 在访问所引用的对象前必须先转换为 std::shared_ptr。

    ```cpp
     
    std::weak_ptr<int> gw;
    
    void f()
    {
        if (auto spt = gw.lock()) { // 使用之前必须复制到 shared_ptr
            std::cout << *spt << "\n";
        }
        else {
            std::cout << "gw is expired\n";
        }
    }
    int main()
    {
        {
            auto sp = std::make_shared<int>(42);
            gw = sp;
    
            f();
        }
    
        f();
            return 0；
    }
    ```

+ weak_ptr虽然是一个模板类，但是不能用来直接定义指向原始指针的对象。

+ weak_ptr接受shared_ptr类型的变量赋值，但是反过来是行不通的，需要使用lock函数。

+ weak_ptr设计之初就是为了服务于shared_ptr的，所以不增加引用计数就是它的核心功能。

+ 由于不知道什么之后weak_ptr所指向的对象就会被析构掉，所以使用之前请先使用expired函数检测一下。




