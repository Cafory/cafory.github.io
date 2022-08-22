---
title: C++ 类型转换
tags: C++
---


# C++ 类型转换

C++继承并扩展C语言的传统类型转换方式，提供了功能更加强大的转型机制（检查与风险）

### 1 const_cast
+ 去除指针或引用上的const和volatile属性。

+ 定义
    ```cpp
    const_cast <T*>(content) //去常转换，编译时执行
    ```

+ 主要作用于同一个类型之间的去长和添加常属性之间的转换，但不能用作不同类型之间的转换。它可以把一个不是常属性（const）的转换成常属性的，同时也可以对一个本是常属性的类型进行去长操作，即const变量转非const。

+ 常用于去除const类对象的只读属性，且强制转换的类型必须是【指针】或【引用】。

### 2  static_cast
+ 主要用于C++中内置的基本数据类型之间的转换，但是没有运行时类型的检测来保证转换的安全性。还用于各种隐式转换，比如非const转const，void*转指针等。

+ 定义
    ```cpp
    static_cast <T> content //静态转换，在编译期间处理
    ```

+ static_cast 可以执行指向相关类的指针之间的转换。
    > 可以向上转换（从指向派生的指针到指向基的指针）; 这是安全的
    > 
    > 可以执行向下转换（从指向基的指针到指向派生的指针）;不执行任何检查以保证安全

+ static_cast 还能够执行隐式允许的所有转换、及其相反的转换（不仅是那些具有指向类的指针的转换）。
    > 1 ）从 void* 转换为任何指针类型（不安全的）。在这种情况下，它保证如果 void* 值是通过从相同的指针类型转换获得的，则生成的指针值是相同的。
    > 
    > 2）将整数、浮点值和枚举类型转换为枚举类型。
    >
    > 3）非 const 转 const。

+ static_cast 还可以执行以下操作
    > 1）显式调用单参数构造函数或转换运算符。
    >
    > 2）转换为右值引用。
    >
    > 3）将枚举类值转换为整数或浮点值。
    >
    > 4）将任何类型转换为 void，评估并丢弃该值。
    >
    > 5）用于内置的基本数据类型之间的转换

+ 不能用于基本数据类型（char*、int*等）指针之间的转换

### 3 dynamic_cast

+ 通常用于基类和派生类之间的转换，转换时会进行类型安全检查。只能用于【含有虚函数的类】，用于类层次间的向上和向下转换。只能转【指针】或【引用】。

+ 定义

    ```cpp
    dynamic_cast <T*> content //动态类型转换，运行时执行
    ```

+ dynamic_cast 只能与类的指针和引用一起使用（或与void * 一起使用），其目的是确保类型转换的结果指向目标指针类型的有效完整对象。

    >1）向上转换：派生类指针转换为基类指针，与隐式转换所允许的方式相同。 
    >
    > 2）向下转换多态类（具有虚拟成员的类）：基类指针转换为派生类指针，当且仅当指向的对象是目标类型的有效完整对象。
    >
    > 返回值：
    > 
    > + 转换成功时返回相应正确的类型。
    > + 转换失败时：
    >  
    >   + 由于不是所需类的完整对象而无法转换指针时（子类成员多于父类成员），它将返回一个**空指针（NULL）**以指示失败。
    >   + 由于引用类型并且无法转换，则会抛出 bad_cast 类型的异常
    >
    >

+ dynamic_cast 还可以对指针执行其他隐式转换：

    > 在指针类型之间（甚至在不相关的类之间）转换空指针;
    > 
    > 将任何类型的任何指针转换为 void* 指针。

+ 不能用于内置的基本数据类型之间的转换

+ 不能用于基本数据类型指针（char*、int*）之间的转换

+ 转换成功，返回的是类的指针或引用，转指针失败，返回`NULL`, 转引用失败，返回`bad_cast`

+ 进行转换的时候基类中一定要有虚函数的支持。因为只有类中有虚函数，才说明它希望让基类指针或引用指向其派生类对象，这样转换才有意义。

+ 在类的转换时，在类次间进行转换的时候dynamic_cast和static_cast进行向上转换时，它们的转换效果是一样的；但是向下转换时，dynamic_cast会进行类型检查，所以它比static_cast更安全。它也可以让指向基类的指针转换为指向其子类的指针或是其兄弟类的指针（交叉关系的类指针）。

+ 具有类型检查的功能，编译时会去检查使用的方法是否正确。

+ dynamic_cast 运用 RTTI 技术在运行时判断变量类型和要转换的类型是否相同 来判断是否能够进行向下转换；RTTI 技术（Runtime Type Information，运行时类型信息），提供了运行时确定对象类型的方法。在 C++ 层面主要体现在 dynamic_cast 和 typeid ，在 vs 编译器中虚函数表的 -1 位置存放了指向 type_info 的指针，对于存在虚函数的类型，dynamic_cast 和typeid 都会去查询 type_info 。

### 4 reinterpret_cast

+ 基本上可以做任何类型的转换 ，既不检查指向的内容，也不检查指针类型本身。容易出问题。

+ 运算结果是从一个指针到另一个指针的 值的简单二进制副本。

+ 定义

    ```cpp
    reinterpret_cast <T*> content //重解释类型转换，几乎什么都可以转。
    ```

+ reinterpret_cast的机制是对二进制数据进行重新的解释，不会改变原来的格式，而static_cast则会改变原来的格式。

+ reinterpret_cast可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针。或者不同类型的指针的相互替换

+ reinterpret_cast不可以用于基本类型的转换