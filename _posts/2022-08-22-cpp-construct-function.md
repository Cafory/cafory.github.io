---
title: C++ 构造函数与虚构函数
tags: C++
---

<!--more-->

# C++ 构造函数与虚构函数

构造函数具有如下几个特点

+ 名字与类名相同，可以有参数，**但是不能有返回值（void也不行）**

+ 作用是对对象进行初始化工作，如给成员变量赋值等。

+ 如果定义类时没有写构造函数，系统会生成一个默认的无参构造函数，默认构造函数没有参数，不做任何工作。

+ 如果定义了构造函数，系统不再生成默认的无参构造函数

+ 对象生成时构造函数自动调用，对象一旦生成，不能在其上再次执行构造函数

+ 一个类可以有多个构造函数，为重载关系



以下以Student类为例

### 1. 默认构造函数

+ 当未显式提供构造函数时，系统将会提供一个无参的默认构造函数。

+ 当显式定义有参构造函数时，无参构造函数需要自己定义；


+ 定义:

    ```cpp
    Student(); // 无参构造函数
    Student(int num=0;int age=0); // 所有参数都有默认值的构造函数
    ```
    

### 2. 普通构造函数

+ C++用于构建类的新对象时需要调用的函数，

+ 根据不同的参数列表找到对应的构造函数。

+ 定义：
    ```cpp
    Student(int num，int age）;//有参数
    ```

### 3. 复制（拷贝）构造函数

+ 特征只有一个参数，是同类对象的引用或常引用

+ 定义：
    ```cpp
    Student(Student & stu); // 同类对象的引用
    Student(const Student & stu); // 同类对象的常引用
    ```
        
    
+ 复制构造函数在以下三种情况下会被调用。

1. 当用一个对象去初始化同类的另一个对象时，会引发复制构造函数被调用。例如，下面的两条语句都会引发复制构造函数的调用，用以初始化 s2。

    ```cpp
    Student s2(s1); 
    Student s2 = s1; 
    ```          

    注意，第二条语句是初始化语句，不是赋值语句。赋值语句的等号左边是一个早已有定义的变量，赋值语句不会引发复制构造函数的调用。例如：

    ```cpp
    Student s1, s2; s1 = s2 ;
    s1=s2;
    ```

    这条语句不会引发复制构造函数的调用，因为 s1 早已生成，已经初始化过了。
    
2. 如果函数 F 的参数是类 Student 的对象，那么当 F 被调用时，类 Student 的复制构造函数将被调用。换句话说，作为形参的类A的对象，是用复制构造函数初始化的，而且调用复制构造函数时的参数，就是调用函数时所给的实参。

    例如：
    ```cpp
    int F(Student s){...} 
    //调用 F(s1) 时，调用拷贝构造函数产生临时对象s
    int F(Student & s){...} 
    //则不会调用拷贝构造函数
    ```

3. 如果函数的返回值是类Student的对象，则函数返回时，类Student的复制构造函数被调用。换言之，作为函数返回值的对象是用复制构造函数初始化的，而调用复制构造函数时的实参，就是 return 语句所返回的对象。

    例如：
    ```cpp
    Student F()
    {
        Student s;
        return s; // 调用拷贝构造函数，返回临时对象
    }
    ```
        
+ 关于拷贝构造函数的一些问题：

    > Q1：为什么需要用户自定义拷贝构造函数？
    >
    >A1：当用户没有定义拷贝构造函数时，编译器会自动为类添加一个默认的拷贝构造函数。但默认情况下是浅拷贝。浅拷贝造成内存重叠的问题，容易重复释放内存，形成野指针，因此需要用户自定义拷贝构造函数完成深拷贝。

    >Q2：拷贝构造函数为什么需要引用传递？
    >
    >A2：假设拷贝构造函数使用值传递，根据调用拷贝构造函数情况之二，会出现无限递归调用的问题 。

4. 转换构造函数

+ 一个构造函数接收【一个不同于其类类型】的形参，可以视为将其形参转换成类的一个对象。像这样的构造函数称为转换构造函数。

+ 定义：
    ```cpp
    Student(int a); //形参时其他类型变量，且只有一个形参
    ```
        

5. 移动构造函数

+ 移动构造函数的参数和拷贝构造函数不同，拷贝构造函数的参数是一个左值引用，但是移动构造函数的初值是一个【右值引用】。这意味着，移动构造函数的参数是一个右值或者将亡值的引用。也就是说，只用用一个右值，或者将亡值初始化另一个对象的时候，才会调用移动构造函数。

+ 定义
    ```cpp
    Student(Student && stu); 
    Student(const Student && stu); //右值引用可以设置为const吗 可以
    ```
    
    【&&】 代表右值引用。当类中同时包含拷贝构造函数和移动构造函数时，如果使用临时对象初始化当前类的对象，编译器会优先调用移动构造函数来完成此操作。只有当类中没有合适的移动构造函数时，编译器才会退而求其次，调用拷贝构造函数。

    移动构造函数相比于拷贝构造函数提高了创建效率，因为直接获得临时对象的内存所有权，临时对象不需要进行拷贝销毁操作。移动构造函数首先将传递参数的内存地址空间接管，然后将内部所有指针设置为nullptr，并且在原地址上进行新对象的构造，最后调用原对象的的析构函数，这样做既不会产生额外的拷贝开销，也不会给新对象分配内存空间。

6. 赋值构造函数

+  当一个类的对象对另一个对象进行直接赋值时会调用赋值函数

+ 拷贝构造函数就是在对象被创建的时候调用的，而赋值函数只能被创建好的对象调用

+ 定义
    ```cpp
    void operator = (Student &s)
    void operator = (Student &&s)
    ```
+ 如果没有自定义赋值运算符函数。系统会调用自动生成的默认赋值函数。默认的赋值构造函数是浅拷贝的。

+ 赋值构造函数也可以返回`*this`的引用，用来链式操作

类初始化自带的默认函数

```cpp
class Test
{
public:
    Test();
    ~Test();
    Test(const Test &);
    Test & operator =(const Test &);
    Test(Test &&);
    Test & operator =(const Test &&);
}
```
