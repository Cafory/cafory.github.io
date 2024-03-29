---
title: C++ 友元函数与友元类
tags: C++
---


本文介绍C++中的友元函数与友元类。

<!--more-->

### 1 友元函数

+ 友元函数时可以直接访问类的私有成员或保护成员，它是定义在类外的普通函数，它不属于任何类，但需要在类的定义中加以声明。

+ 特点：

    1. 与类的成员函数具有一样的权限；

    2. 可以访问类的私有成员；

    3. 它是定义在类外的普通函数，不属于任何类；

    4. 没有this指针；

+ 用法：

    1. 友元函数需要在类中进行声明，前面需要加上friend，可以放在公有部分也可以放在私有部分；

    2. 一个函数可以是多个类的友元函数，只需要在个各类中分别进行声明；

    3. 友元函数的调用与一般函数的调用方式和原理一致；

    4. 普通友元函数一般在类外实现，如果在类内实现，至少有一个参数是将其作为友元函数的类

    5. 在类外实现的友元函数，不一定要传对应类的类型参数

+ 示例：

    **1) 普通函数作为友元函数**

    ```cpp
    class A
    {

        int value;
    public:
        A(){ value = 0; }
        ~A(){};
        friend void test_1(A & a)
        {
            cout<<"test_1:"<<a.value<<endl;
        }

        friend void test_2()
        {
            cout<<"test_2:"<<endl;
        }

        friend void test_3( A & a );
        friend void test_4( );
    };

    void test_3(A & a)
    {
        cout<<"test_3:"<<a.value<<endl;
    }

    void test_4()
    {
        cout<<"test_4:"<<endl;
    }


    int main()
    {
        A a;
        test_1(a);
        //test_2(); //类内实现，必须有对应类的类型参数 报错
        test_3(a);
        test_4();
    }
    ```

    **2) 类成员函数作为友元函数**

    ```cpp
    class B;

    class A
    {

        string value;
    public:
        A(){ value = "A"; }
        ~A(){};
        void test( B & b );
    };

    class B
    {
        string value;
    public:
        B(){ value = "B"; }
        ~B(){};
        friend void A::test(B & b);
    };

    void A::test(B & b)
    {
        // 这里我们必须在B实现之后再实现test函数
        // 因为test使用了B的成员变量，则必须先知道
        // B的实现，故先在A中声明test
        // 然后在B实现之后实现test函数
        cout<<b.value<<endl;
    }


    int main()
    {
        B b;
        A a;
        a.test(b);
    }
    ```

### 2 友元类

+ 友元类中的所有成员函数都是另一个类的友元函数，可以直接访问友元类中的所有成员；

+ 特点：

    1. 友元关系不能被继承；

    2. 友元关系是单向的，不具有交换性。若类B是类A的友元，类A不一定是类B的友元，要看在类中是否有相应的声明；

    3. 友元关系不具有传递性。若类B是类A的友元，类C是B的友元，类C不一定是类A的友元，同样要看类中是否有相应的声明；

+ 示例：

    ```cpp
    class B;

    class A
    {

        string value;

    public:
        A() { value = "A"; }
        ~A(){}
        void test_1(B &b);
    };

    class B
    {
        string value;
    public:
        B() { value = "B"; }
        ~B(){}
        // 声明A作为B的友元类，
        // 则A的成员函数都可以访问B的私有成员
        friend A;
    };

    void A::test_1(B &b)
    {
        cout << "test_1" <<b.value<< endl;
    }

    int main()
    {
        B b;
        A a;
        a.test_1(b);
    }

    ```

### 3 模板类与友元函数

+ 模板类的约束模板友元函数

    约束模板友元，指的是**模板友元函数实例化取决于模板类被实例化时的类型**。


    写法一 ： `T(function) = T(class)`

    这种写法是保持模板友元函数的参数类型和模板类的参数类型一致。可以将友元函数的声明和定义分开，也可以直接在模板类内部直接定义友元函数。需要注意两种实现写法上的不同。

    1） 友元函数声明和定义分开

    ```cpp
    template <typename T> class A;
    template <typename T> void test(A<T> & a); // 模板声明

    template <typename T>
    class A
    {
    public:
        friend void test<T> ( A<T> & a ); // 声明友元函数
        
        
    };

    template <typename T>
    void test( A<T> & a )
    {
        cout<<"test"<<endl;
    }
                                                        
    
    int main()
    {
        A<int> a;
        test(a);
        return 0;
    }
    ```

    2） 友元函数直接定义在模板类内部

    ```cpp
    template <typename T>
    class A
    {
    public:
        friend void test ( A<T> & a ) // 声明友元函数
        {
            cout<<"test"<<endl;
        }
        
    };

    int main()
    {
        A<int> a;
        test(a);
        return 0;
    }
    ```

    写法二 ： `T(function) = class<T>`

    ```cpp
    template <typename T> void test(T & a); 

    template <typename T>
    class A
    {
    public:
        friend void test<A<T>> ( A<T> & a ); // 声明友元函数
    };

    template <typename T>
    void test( T & a )
    {
        cout<<"test"<<endl;
    }
                                          
    
    int main()
    {
        A<int> a;
        test(a);
        return 0;
    }
    ```

+ 模板类的非约束模板友元函数

    这里所谓的约束模板友元，指的是友元函数的所有实例化版本都是模板类的每一个实例化版本的友元。

    写法一：`T(function) = T(class)`

    这种写法是保持模板友元函数的参数类型和模板类的参数类型一致。

    ```cpp
    template <typename T>
    class A
    {
    public:
        
        template <typename D>
        friend void test ( A<D> & a ); // 声明友元函数
    };

    template <typename T>
    void test( A<T> & a )
    {
        cout<<"test"<<endl;
    }
                                                        
    
    int main()
    {
        A<int> a;
        test(a);
        return 0;
    }
    ```

    写法二：`T(function) = class<T>`

    ```cpp
    template <typename T>
    class A
    {
    public:
        
        template <typename D>
        friend void test ( D & a ); // 声明友元函数
    };

    template <typename T>
    void test( T & a )
    {
        cout<<"test"<<endl;
    }
                                                        
    
    int main()
    {
        A<int> a;
        test(a);
        return 0;
    }
    ```
    

