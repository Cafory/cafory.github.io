---
title: C++ 
tags: Network


---


# C++ 模板

### 1 模板与泛型编程

+ 所谓泛型编程,是以独立于任何特定类型的方式编写代码,使用泛型编程时,我们需要提供具体程序实例所操作的类或值

+ **模板是泛型编程的基础**,模板是创建类或函数的蓝图或公式,我们给这些蓝图或公式足够的信息,让这些蓝图或公式真正的转变为具体的类或函数,这种转变发生在编译时

+ 模板支持将类型作为参数的程序设计方式,从而实现了对泛型程序设计的支持,也就是说C++模板机制允许将类型作为参数( **类型参数化** )。

### 2 函数模板

+ 模板(Templates)使得我们可以生成通用的函数，这些函数能够接受任意数据类型的参数，可返回任意类型的值，而不需要对所有可能的数据类型进行函数重载。这在一定程度上实现了宏（macro）的作用。

+ 函数模板代表了一个函数家族，该函数模板与类型无关，在使用时被参数化，根据实参类型产生函数的特定类型版本。

+ 定义

    ```cpp
    template<typename T1, typename T2,......,typename Tn> function_declaration ;

    template<class T1, class T2,......,class Tn> function_declaration ;
    // class 和 typename使用是等价的
    ```

+ 例子：定义一个实现值交换的函数模板

    ```cpp
    template<class T>
    void Swap(T& a, T& b)
    {
        T tmp = a;
        a = b;
        b = tmp;
    }

    ```

+ 在编译器编译阶段，对于模板函数的使用，编译器需要根据传入的实参类型来推演生成对应类型的函数以供调用。比如：当用 int 类型使用函数模板时，编译器通过对实参类型的推演，将T确定为int类型，然后产生一份专门处理int类型的代码。

+ 函数模板实例化

1. 隐式实例化 : 让编译器自己根据实参的类型推导模板参数的类型

    ```cpp
    template<class T>
    T Add(const T& a, const T& b)
    {
        return a + b;
    }
    int main()
    {
        int a = 1, b = 2;
        cout << Add(a,b) << endl;
    }

    ```

2. 显示实例化 : 在函数名后的<>中指定模板参数的实际类型

    ```cpp
    template<class T>
    T Add(const T& a, const T& b)
    {
        return a + b;
    }
    int main()
    {
        int a = 1;
        double b = 2.2;
        cout<<Add<int>(a,b)<<endl;
        cout<<Add<double>(a,b)<<endl;
    }
    ```

+ 模板参数的匹配原则

    一个非模板函数可以和一个同名的函数模板同时存在，而且该函数模板还可以被实例化为这个非模板函数。此时，调用规则如下：

    > 1 . 对于非模板函数和同名的函数模板，如果其他条件都是相同的话，那么在调用的时候，重载解析过程通常会优先调用非模板函数，而不会从该模板产生出一个实例。然而，如果模板可以产生一个具有更好匹配的函数，那么将选择模板。
    >
    > 2 . 可以显式地指定一个空的模板实参列表，这个语法好像是告诉编译器：只有模板才能匹配这个调用（即便非模板函数更符合匹配条件也不会被调用到），而且所有的模板参数都应该根据调用实参演绎出来。
    >
    > 3 . 因为模板是不允许自动类型转化的；但普通函数可以进行自动类型转换，所以当一个匹配既没有非模板函数，也没有函数模板可以匹配到的时候，会尝试通过自动类型转换调用到非模板函数（前提是可以转换为非模板函数的参数类型）。

    例子如下：

    ```cpp
    template <typename T>
    inline T const& max(T const & a, T const & b)
    {
        std::cout << "all" << std::endl;
        return a < b ? b : a;
    }

    inline int const& max(int const & a, int const & b)
    {
        std::cout << "int" << std::endl;
        return a < b ? b : a;
    }

    int main()
    {
        int a = 7;
        int b = 42;
        int maxInt = ::max(a,b);
        std::cout << maxInt << std::endl; // 调用非模板
        std::cout << max<>(a,b) << std::endl; // 调用模板
        std::cout << max<int>(a,b) << std::endl; // 调用模板
        std::cout << max(1.0,2.0) << std::endl; // 调用非模板
        std::cout << max("aaa", "bbb") << std::endl; // 调用模板
        std::cout << max('42', 1) << std::endl; // 调用非模板
    }
    ```
+ 非类型模板参数

    在模板参数列表里，还可以定义非类型参数，非类型参数代表的是一个值，既然非类型参数代表一个值，不是一个类型，肯定不能用typename/class关键字来修饰这个值，我们当然要用我们以往学习过的传统类型名来指定非类型参数了。

    当模板被实例化时,这种非类型模板参数的值，或者是用户提供的，或者是编译器自己推断的,但这些值必须都得是常量表达式,因为模板实例化发生在编译阶段。

    非类型模板参数的一些限制 :

    + 浮点数、类对象以及字符串是不允许作为非类型模板参数的，只能为**整形，指针或者左值引用**。

    + 非类型的模板参数必须在编译期就能确认结果。

    例子：

    ```cpp
    #include<iostream>
    using namespace std;
    template<int a,int b>
    int add1()
    {
        return a + b;
    }
    template<class T,int a,int b>
    int add2(T c)
    {
        return c + a + b;
    }
    template<unsigned L1,unsigned L2>
    int charscmp(const char(&p1)[L1],const char(&p2)[L2])
    {
        return strcmp(p1, p2);
    }
    int main()
    {
        cout << add1<1, 2>() << endl;
        cout << add2<int, 1, 2>(5) << endl;
        cout << add2<int, 1, 2>(1.6) << endl;
        cout << charscmp("test2", "test") << endl;
    }
    ```

### 3 类模板

+ 类模板（class templates），使得一个类可以有基于通用类型的成员，而不需要在类生成的时候定义具体的数据类型。

+ 定义

    ```cpp
    template<class T1, class T2, ..., class Tn>
    class 类模板名
    {
        // 类内成员定义
    };

    template<typename T1, typename T2, ..., typename Tn>
    class 类模板名
    {
        // 类内成员定义
    };

    ```
+ 编译器不能为类模板推断模板类型参数，因此,类模板实例化与函数模板实例化不同，类模板实例化**必须要在类模板名字后跟<>**，然后将实例化的类型放在<>中即可，类模板名字不是真正的类，而实例化的结果才是真正的类。

+ 类模板的成员函数

    特性

    + 类模板成员函数,可以写在类模板定义中,这种写在类模板定义中的成员函数会被隐式声明成inline函数。

    + 类模板一旦被实例化之后,那么这个模板的每个实例都会有自己版本的成员函数,所以，**类模板的成员函数是有模板参数的**，因此,如果要把类模板成员函数的定义写到类模板定义的外面，**须以关键字template开始**，后接模板参数列表，同时，在类模板名后用<>将模板参数列表里的所有模板参数名列出来。

    + 一个类模板可能有多个成员函数,当实例化模板以后,后续如果没有使用某个成员函数，则该成员函数不会实例化

    例子

    ```cpp
    template<typename T>
    class myvector
    {
    public:
        // 构造函数
        myvector();
        // 赋值运算符重载
        myvector& operator=(const myvector& v);
        // 会被隐式声明成内联函数
        void func()
        {
            // .....
        }
    };
    template<class T> // 需要声明为模板
    myvector<T>& myvector<T>:: operator=(const myvector& v)
    {
        // .....
    }
    ```
+ 类模板中的成员函数模板

    成员模板：类模板中的成员函数也为模板。举例如下：

    ```cpp
    template <typename T>
    class A {
    public:
        template <typename U>
        void assign(const D<U>& u)
        {
            v = u.getvalue();
        }
    
        T getvalue()
        {
            return v;
        }
    private:
        T v;
    }
    ```




+ 非类型模板参数

    与函数模板非类型模板参数类似，例子如下：

    ```cpp
    template<class T,int size = 10>
    class myarray
    {
    public:
        void func();
    private:
        T arr[size];
    };
    template<class T,int size>
    void myarray<T,size>::func()
    {
        cout << size << endl;
        return;
    }

    ```

+ 继承中类模板的使用

    父类是一般类，子类是模板类，正常继承。

    ```cpp
    class A {
    public:
        A(int temp = 0) {
            this->temp = temp;
        }
        ~A(){}
    private:
        int temp;
    };
    
    template <typename T>
    class B :public A{
    public:
        B(T t = 0) :A(666) {
            this->t = t;
        }
        ~B(){}
    private:
        T t;
    };
    ```
    
    子类是一般类，父类是模板类，继承时必须在子类里实例化父类的类型参数。

    ```cpp
    template <typename T>
    class A {
    public:
        A(T t = 0) {
            this->t = t;
        }
        ~A(){}
    private:
        T t;
    };
    
    
    class B:public A<int> {
    public:
        //也可以不显示指定，直接A(666)
        B(int temp = 0):A<int>(666) {
            this->temp = temp;
        }
        ~B() {}
    private:
        int temp;
    };
    ```

### 4 模板特殊化

+ 模板的特殊化是当模板中的pattern有确定的类型时，模板有一个具体的实现。

+ 特化分为全特化和偏特化
    
    + 全特化 : 将所有的类型参数全部明确指定。

    + 偏特化 : 将部分类型参数明确指定。（偏特化并不仅仅是指特化部分参数，而是针对模板参数更进一步的条件限制所设计出来的一个特化版本）。

+ 类模板支持全特化与偏特化。

+ 函数模板支持全特化，不支持偏特化，是因为模板特化版本不参与函数的重载抉策过程，因此在和函数重载一起使用的时候，可能出现不符合预期的结果。因此标准C++禁止了函数模板的偏特化。

+ 类模板特化例子：

    ```cpp
    template<typename T1, typename T2>
    class Test{
    public:
        Test(T1 x, T2 y):a(x),b(y){}
    private:
        T1 a;
        T2 b;
    }

    template<> //全特化，所有的类型参数全部指定
    class Test<int, char> {
        Test(int x, char y):a(x),b(y){}
    private:
        int  a;
        char b;
    }

    template<typename T> //偏特化，部分类型参数指定
    class Test<T, char> { 
        Test(T x, char y):a(x),b(y){}
    private:
        T    a;
        char b;
    }
    ```

+ 函数模板特化例子

    ```cpp
    template<class T>
    bool IsEqual(const T& left, const T& right)
    {
        return left == right;
    }
    template<> // 空参数模板
    bool IsEqual<const char*>(const char* const& left, const char* const& right)
    {
        return strcmp(left, right) == 0;
    }
    int main()
    {
        const char* p1 = "hello";
        const char* p2 = "hello";
        cout << IsEqual(p1, p2) << endl;
        return 0;
    }
    ```

### 5 可变模板参数

+ C++11的新特性–可变模版参数（variadic templates）是C++11新增的最强大的特性之一，它对参数进行了高度泛化，它能表示0到任意个数、任意类型的参数。

+ 定义：

    ```cpp
    template <class... T>
    void f(T... args);
    ```

    上面的可变模版参数的定义当中，省略号的作用有两个：

    + 声明一个参数包T… args，这个参数包中可以包含0到任意个模板参数；
    + 在模板定义的右边，可以将参数包展开成一个一个独立的参数。

    上面的参数args前面有省略号，所以它就是一个可变模版参数，我们把带省略号的参数称为“参数包”，它里面包含了0到N（N>=0）个模版参数。我们无法直接获取参数包args中的每个参数的，只能通过展开参数包的方式来获取参数包中的每个参数，这是使用可变模版参数的一个主要特点，也是最大的难点，即如何展开可变模版参数。

+ 简单使用

    ```cpp
    template<typename...T>
    void fun1(T...args)
    {
        //这里使用args必须使用 sizeof... 计算长度
        cout << "参数个数为：" << sizeof...(args) << endl;//打印变参的个数
    }

    int main()
    {
        fun1();
        fun1(1, "");
        fun1(1, 2, 3, 4);
        fun1('a', 2, "abc", " ", 'w');

        return 0;
    }

    ```

+ 参数展开

    展开可变模版参数函数的方法一般有两种：
    + 通过递归函数来展开参数包。

    + 通过逗号表达式来展开参数包。

    **(1) 递归方式展开参数包**

    通过递归函数展开参数包，需要提供一个参数包展开的函数和一个递归终止函数，递归终止函数正是用来终止递归的。

    例子如下:

    ```cpp
    //递归终结函数
    void print()
    {
        cout << "这里是终结函数" << endl;
    }

    //typename在简单的参数类型中使用
    //typename...在可变参数的类型中使用
    //比如下面，个人理解为：T1为普通参数类型，T2...为可变参数类型，即T2...是一个组合而不是...args
    //注意可以看成两个类型，T1(普通参数)和T2...(可变参数)
    template<typename T1, typename...T2>
    void print(T1 head, T2...args)
    {
        cout << head << endl;
        print(args...);//利用...解包，，当参数输出完时，这个相当于print()，会调用终结函数print()
    }


    int main()
    {
        print(1, 2, 3, 4);

        return 0;
    }

    ```

    **(2) 逗号表达式展开参数包**

    递归函数展开参数包是一种标准做法，也比较好理解，但也有一个缺点,就是必须要一个重载的递归终止函数，即必须要有一个同名的终止函数来终止递归，这样可能会感觉稍有不便。有没有一种更简单的方式呢？其实还有一种方法可以不通过递归方式来展开参数包，这种方式需要借助逗号表达式和初始化列表。

    例子如下：

    ```cpp
    template<typename  T>
    void printarg(T arg)
    {
        cout << arg << endl;
    }

    //可变参数模板
    template<typename...Args>
    void expand(Args... args)
    {
        int num[] = {(printarg(args), 0)...};//对每个参数都传入到printarg中
    }

    int main()
    {
        expand(1, 2, 3, 4, 5, 6,  7);
        return 0;
    }

    ```

    expand函数中的逗号表达式(printarg(args), 0)，先执行printarg(args)，再得到逗号表达式的结果0。同时还用到了C++11的另外一个特性——初始化列表，通过初始化列表来初始化一个变长数组,{(printarg(args), 0)…}将会展开成((printarg(arg1),0), (printarg(arg2),0), (printarg(arg3),0), etc… )，最终会创建一个元素值都为0的数组int arr[sizeof…(Args)]。由于是逗号表达式，在创建数组的过程中会先执行逗号表达式前面的部分printarg(args)打印出参数，也就是说在构造int数组的过程中就将参数包展开了，这个数组的目的纯粹是为了在数组构造的过程展开参数包。

+ 接收可变模板参数作为形参的函数，在使用args的时候必须利用`args...`解包，且最后一个形参为可变模板参数`T ... args`。

    


