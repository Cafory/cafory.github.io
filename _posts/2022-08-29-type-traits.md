---
title: C++ 类型萃取
tags: C++
---


本文介绍C++中的类型萃取。

<!--more-->

### 1 类型萃取

+ 类型萃取是使用模板技术来萃取类型(包含自定义类型和内置类型)的某些特性，用以判断该类型是否含有某些特性，从而在泛型算法中来对该类型进行特殊的处理用来提高效率或者其他。

+ 类型萃取，可简单理解为类型获取，萃取的典型应用是在模板函数中区分T的类型是原生类型POD，还是自定义类型，POD全称plain old data，简单理解就是c++从c继承来的基本数据类型，如int、double等。

+ 之所以需要区分类型，主要是因为POD类型与自定义类型的很多处理方法不同，典型的就是copy，POD可以直接使用c库提供的memcpy，它主要是实现内存层面的拷贝，而非POD类型需要使用for循环挨个拷贝，因为涉及到深拷贝与浅拷贝的问题，所以在模板中需要识别数据类型，再做不同处理。

### 2 判断POD的例子

+ 一般来说，如果我们涉及到一个类型决定一批类型的时候，那么就可以设置一个模板类trait,把这些受影响的类型都放在trait中。我们可以利用模板的特化实现，下面是一个利用类型萃取判断是否是POD的例子：

    ```cpp
    //类型萃取
    #pragma once

    #include<iostream>

    using namespace std;

    struct __TrueType//定义类 普通类型（基本类型的）
    {
        bool Get()
        {
            return true;
        }
    };

    struct __FalseType//定义类 非基本类型
    {
        bool Get()
        {
            return false;
        }
    };

    template <class _Tp>//模板类 （类型萃取）
    struct TypeTraits //非基本类型
    {
        typedef __FalseType   __IsPODType;
    }; 
    //将基本类型类 与非基本类型类 重命名（typedef）为相同名
    //调用__IsPODType.Get() 时编译器会根据TypeTraits<class T> T 的实际类型调用 	
    //__FalseType	 或 Truetype 的 Get()函数  得到不同 bool值	

    //	以下为常用基本类型特化
    struct TypeTraits< bool>			  
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< char>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< unsigned char >
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< short>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< unsigned short >
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< int>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< unsigned int >
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< long>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< unsigned long >
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< long long >
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< unsigned long long>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< float>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< double>
    {
        typedef __TrueType     __IsPODType;
    };

    template <>
    struct TypeTraits< long double >
    {
        typedef __TrueType     __IsPODType;
    };

    template <class _Tp>
    struct TypeTraits< _Tp*>
    {
        typedef __TrueType     __IsPODType;
    };

    ```
