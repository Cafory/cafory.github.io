---
title: C++ 运算符重载
tags: C++
---

<!--more-->

<style>
    table{
        margin:auto
    }
    </style>

# C++ 运算符重载

运算符重载的方法是**定义一个运算符重载函数**，也就是说，运算符重载函数是通过定义一个函数来实现的，**运算符重载实质上是函数的重载**。

### 1 运算符重载规则

+ 运算符重载的格式：

    ```cpp
    函数类型 operator 运算符名称 (形参列表);
    ```

+ 在上面的格式中，operator是c++的关键字，是专门用于定义重载运算符的函数的，运算符名称就是c++已经有的运算符。注意：函数名是由operator和运算符组成。

+ 重载运算符时,运算符的运算顺序和优先级不变,操作数个数不变。

+ 不能创造新的运算符,只能重载C++中已有的运算符,并且规定有6个运算符不能重载。

    | 含义 | 不能重载的运算符 |
    | :---: | :---: |
    | 类属关系运算符 | **.** |
    | 成员指针运算符 | **.\*** |
    | 作用域运算符 | **::** |
    | 条件运算符 | **?:** |
    | 编译预处理运算符 | **#** |
    | 取数据类型长度运算符 | **sizeof()** |
    |||

+ 允许重载的运算符
    | 含义 | 允许重载的运算符 |
    | :---: | :---: |
    |双目关系运算符|**+，-，，/，%**|
    |关系运算符|**==，!=，<，>，<=，>=** |
    |逻辑运算符 | **\|\|，&&，!** |
    |单目运算符 | **+(正号)，-(负号)，指针，&** |
    |自增自减运算符 | **++，--** |
    |位运算符 | **\|，&，~，^，<<，>>** |
    |赋值运算符 | **=，+=，-=，=，/=，%=，&=，\|=，^=，<<=，>>=**|
    |空间申请与释放| **new，delete，new\[\]，delete\[\]**|
    |其他运算符| **()函数调用，->，->，逗号，[]下标**|
    |||


+ 为了防止用户对标准类型进行运算符重载，C++规定重载后的运算符的操作对象必须至少有一个是用户定义的类型。主要是避免重载后的二义性导致程序不知道该执行哪一个（是自带的的还是重载后的函数）。

+ 使用运算符不能违法运算符原来的句法规则。如不能将% 重载为一个操作数。

+ 大多数运算符可以通过成员函数和非成员函数进行重载但是下面这四种运算符只能通过成函数进行重载：
    + = 赋值运算符
    + ()函数调用运算符
    + []下标运算符
    + ->通过指针访问类成员的运算符

### 2 运算符重载的方式

+ 运算符函数重载一般有两种形式： **重载为类的成员函数**和**重载为类的非成员函数**。

+ 重载为类的成员函数

    ```cpp
    <函数类型> operator <运算符>(<参数表>)
    {
        <函数体>
    }　　

    ```

    当运算符重载为类的成员函数时，函数的参数个数比原来的操作数要少一个（后置单目运算符除外），这是因为成员函数用this指针隐式地访问了类的一个对象，它充当了运算符函数最左边的操作数。因此：

    > (1) 双目运算符重载为类的成员函数时，函数只显式说明一个参数，该形参是运算符的右操作数。
    >
    >(2) 前置单目运算符重载为类的成员函数时，不需要显式说明参数，即函数没有形参。
    >
    > (3) 后置单目运算符重载为类的成员函数时，函数要带有一个整型形参。

    例子如下：

    ```cpp
    #include <iostream>
    using namespace std;
    class complex
    {
    public:
        complex() { real = 0; imag = 0; }//定义构造函数
        complex(double r, double i) { real = r; imag = i; }//构造函数重载
        complex operator +(complex& c2) //定义重载运算符+的函数
        {
            complex c;
            c.real = real + c2.real;
            c.imag = imag + c2.imag;
            return c;//返回局部对象
        }
        void display()//定义输出函数
        {
            cout << "(" << real << "," << imag<< "i)"<< endl;
        }
    private:
        double real;
        double imag;
    };
    int main()
    {
        complex c1(3, 4), c2(5, -10), c3;
        c3 = c1+c2;//c3=c1.operator(c2)
        c1.display();
        c2.display();
        c3.display();
        return 0;
    }

    ```

    重载 `++`例子如下：

    ```cpp
    #include <iostream>
    #include <string>
    using namespace std;
    class Time
    {
    public:
        Time() { minute = 0; sec = 0; }//定义默认构造函数
        Time(int m,int s): minute(m), sec(s){ }//构造函数重载
        Time operator++()//定义前置++运算符
        {
            if (++sec >= 60)
            {
                sec-=60; ++minute;
            }
            return *this;//返回当前对象值
        }
        Time operator++(int)//定义后置++运算符
        {
            Time  temp(*this);//建立临时对象
            sec++;
            if (sec >= 60)
            {
                sec -= 60; ++minute;
            }
            return temp;//返回的是自加前的对象
        }
        void display() { cout << minute << ":" << sec << endl; }
    private:
        int minute;
        int sec;
    };
    int main()
    {
        Time t1(34, 59),t2;
        t1.display();//原值
        ++t1;//前置++
        t1.display();//执行++t1后t1的值
        t2 = t1++;//后置++
        t1.display();//再执行t1++后t1的值
        t2.display();//t2保存的是执行t1++前t1的值
        return 0;
    }

    ```

+ 重载为类的非成员函数

    非成员函数通常是友元。（可以把一个运算符作为一个非成员、非友元函数重载。但是，这样的运算符函数访问类的私有和保护成员时，必须使用类的公有接口中提供的设置数据和读取数据的函数，调用这些函数时会降低性能。可以内联这些函数以提高性能。）

    ```cpp
     friend  <函数类型> operator <运算符>(<参数表>)
    {
        <函数体>
    }　
    ```
    
    例子如下:

    ```cpp
    #include <iostream>
    #include <string.h>
    using namespace std;
    class String
    {
    public:
        //声明运算符函数为友元函数
        friend bool operator==(String &str1,String &str2); 
        friend bool operator>(String &str1,String &str2);
        friend bool operator<(String &str1,String &str2);
        String() { p = NULL; }//定义默认构造函数 
        String(char *str) { p = str; }//重载构造函数 
        void display() { cout << p << endl; }
    private:
        char * p;
    };
    //定义重载运算符的实现
    bool operator==(String &str1,String &str2)
    {
        if(strcmp(str1.p,str2.p)==0)
            return true;
        else
            return false;
    }
    bool operator>(String &str1,String &str2)
    {
        if(strcmp(str1.p,str2.p)>0)
            return true;
        else
            return false;
    }
    bool operator<(String &str1,String &str2)
    {
        if(strcmp(str1.p,str2.p)<0)
            return true;
        else
            return false;
    }
    int main()
    {
        String str1("hello"), str2("book");
        str1.display();
        str2.display();
        cout<<(str1==str2)<<endl;
        cout<<(str1>str2)<<endl;
        cout<<(str1<str2)<<endl;
        return 0;
    }

    ```

+ 重载方式的选择

    (1) 一般情况下，单目运算符最好重载为类的成员函数；双目运算符则最好重载为类的友元函数。

    (2) 以下一些双目运算符不能重载为类的友元函数：=、()、[]、->。

    (3) 类型转换函数只能定义为一个类的成员函数而不能定义为类的友元函数。

    (4) 若一个运算符的操作需要修改对象的状态，选择重载为成员函数较好。

    (5) 若运算符所需的操作数（尤其是第一个操作数）希望有隐式类型转换，则只能选用友元函数。

    (6) 当运算符函数是一个成员函数时，最左边的操作数（或者只有最左边的操作数）必须是运算符类的一个类对象（或者是对该类对象的引用）。如果左边的操作数必须是一个不同类的对象，或者是一个内部类型的对象，该运算符函数必须作为一个友元函数来实现。

    (7) 当需要重载运算符具有可交换性时，选择重载为友元函数。









