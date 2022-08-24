---
title: C++ 万能引用与完美转发
tags: C++
---

<!--more-->

# C++ 万能引用与完美转发


### 1 万能引用

+ 例子

    ```cpp
    // 例1
    #include <iostream>

    int func_rvalue_input(int &&a) {
        std::cout << ">>>func_rvalue_input" << std::endl;
        return a + 1;
    }

    int func_lvalue_input(int &a) {
        std::cout << ">>>func_lvalue_input" << std::endl;
        return a + 1;
    }

    int main() {
        int a = 1;
        auto res_rvalue = func_rvalue_input(1);
        std::cout << "func_rvalue_input result: " << res_rvalue << std::endl;
        auto res_lvalue = func_lvalue_input(a);
        std::cout << "func_rvalue_input result: " << res_lvalue << std::endl;
        return 0;
    }
    ```
    两个函数`func_r/lvalue_input`功能是完全一致，唯一的区别就是入参一个要求是右值引用一个是左值引用，这就需要两个不同接口来完成这样的任务(或者是需要涉及一个接口，这个接口既能接受右值引用入参，也可能接受左值引用入参)。所以有没有一种方法，入参既能接受左值引用，也能接受右值引用呢？
    当然有，这就是C++中的万能引用。


+ 万能引用：顾名思义就是即可引用左值，也可能引用右值。

    ```cpp
    // 例2
    int func_rvalue_input(int &&a) {} // a是右值引用
    template<class T>
    int func_all_input(T &&a) {}  // a是万能引用

    int get_five() { return 5;}
    int &&x = get_five();  // x是右值引用
    auto &&y = get_five();  // y是万能引用
    ```
    通过例2可以发现，之所以是万能引用主要是因为发生了类型推导，在`T&&`和`auto&&`的初始化过程会发生类型推导，推导时如果初始化源是左值，那么目标对象就会推导出左值引用，否则就是右值引用；需要注意的是无论如何都会推导出一个引用类型。

    万能引用能如此灵活地引用对象，实际上是因为在C++11中添加了一套引用叠加推导的规则——**引用折叠**；在这套规则中规定了在不同的引用类型互相作用的情况下应该如何推导出最终类型，如下表所示：

    |模板类型| T实际类型 | 最终类型 |
    | :---: | :---: | :---: |
    | T & | int | int & |
    | T & | int & | int & |
    | T & | int && | int & |
    | T && | int | int && |
    | T && | int & | int & |
    | T && | int && | int && |

    上面的表格显示了引用折叠的推导规则，可以看出在整个推导过程中，只要有左值引用参与进来，最后推导的结果就是一个左值引用。只有实际类型是一个非引用类型或者右值引用类型时，最后推导出来的才是一个右值引用。

    所以如果我们的类模板是T&&类型，只有模板参数T推导为左值引用时引用折叠后最终类型才会为左值引用其他都是右值引用。从而也可以利用这样的规则达成万能引用的目标。

### 2 完美转发

+ 例子 

    ```cpp
    #include <iostream>

    template<typename T>
    void print(T & t){
        std::cout << "左值" << std::endl;
    }

    template<typename T>
    void print(T && t){
        std::cout << "右值" << std::endl;
    }

    template<typename T>
    void testForward(T && v){
        print(v);
        print(std::forward<T>(v));
        print(std::move(v));
    }

    int main(int argc, char * argv[])
    {
        testForward(1);

        std::cout << "======================" << std::endl;

        int x = 1;
        testFoward(x);
    }
    ```

    执行结果

    ```cpp
    左值
    右值
    右值
    =========================
    左值
    左值
    右值
    ```

    从上面第一组的结果我们可以看到，传入的1虽然是右值，但经过函数传参之后它变成了左值（在内存中分配了空间）；而第二行由于使用了std::forward函数，所以不会改变它的右值属性，因此会调用参数为右值引用的print模板函数；第三行，因为std::move会将传入的参数强制转成右值，所以结果一定是右值。

    再来看看第二组结果。因为x变量是左值，所以第一行一定是左值；第二行使用forward处理，它依然会让其保持左值，所以第二也是左值；最后一行使用move函数，因此一定是右值。

+ 完美转发:不用担心实参在传入到转发函数时，形参发型类型改变。

    调用模板函数给另一个函数传值时，为了将参数类型保持原本状态传入函数，需要使用完美转发`std:forwrad<T>()`( 也可以使用强制转换，但不简洁 )，完美转发必须与模板一起使用，不能单独使用。


### 3 move与forward

+ `std::move`

    c++11中提供了std::move()来将左值引用转换为右值引用，从而使用移动语义避免复制构造。move是将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存拷贝。

    c++中所有容器都实现了move语义，方便我们实现性能优化。move对于拥有形如对内存、文件句柄等资源的成员的对象有效。如果是一些基本类型，比如int或char[10]数组等，如果使用move，仍然会发生拷贝（因为没有对应的移动构造函数)

    别看它的名字叫move，其实std::move并不能移动任何东西，**它唯一的功能是将一个左值/右值强制转化为右值引用**，继而可以通过右值引用使用该值，所以称为移动语义。

    std::move的作用：将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存的搬迁或者内存拷贝所以可以提高利用效率,改善性能。 

    实现如下：

    ```cpp
    template <typename T>
    typename remove_reference<T>::type&& move(T&& t)
    {
        //上句使用typename关键字告诉编译器 其后是 类型而非是变量，利用 typename的一个作用为：使用嵌套依赖类型(nested depended name)。
        return static_case<typename remove_reference<T>::type &&>(t);
    }
    ```

    由上述代码可以看出`std::move`的返回类型为`typename remove_reference<T>::type&&`

    move的返回类型非常奇特，我们在开发时很少会这样写，它表示的是什么意思呢？

    这就要提到C++的另外一个知识点，即类型成员。你应该知道C++的类成员有成员函数、成员变量、静态成员三种类型，但从C++11之后又增加了一种成员称为类型成员。类型成员与静态成员一样，它们都属于类而不属于对象，访问它时也与访问静态成员一样用::访问。

    了解了这点，我们再看move的返类型是不是也不难理解了呢？它表达的意思是返回remove_reference类的type类型成员。而该类是一个模板类，所以在它前面要加typename关键字。

    remove_reference看着很陌生，接下来我们再分析一下remove_reference类，看它又起什么作用吧。其实，通过它的名子你应该也能猜个大概了，就是通过模板去除引用。我们来看一下它的实现吧。

    ```cpp
    template <typename T>
    struct remove_reference{
        typedef T type;  //定义T的类型别名为type
    };

    template <typename T>
    struct remove_reference<T&> //左值引用
    {
        typedef T type;
    }

    template <typename T>
    struct remove_reference<T&&> //右值引用
    {
    typedef T type;
    }
    ```

    通过上面的代码我们可以知道，经过remove_reference处理后，T的引用被剔除了。假设前面我们通过move的类型自动推导得到T为int&&，那么再次经过模板推导remove_reference的type成员，这样就可以得出type的类型为int了。

    remove_reference利用模板的自动推导获取到了实参去引用后的类型。

+ `std::forward`

    和移动语义的情况一样，显式使用static_cast类型转换进行转发不是一个便捷的方法。在C++11的标准库中提供了一个std::forward函数模板，在函数内部也是使用static_cast进行类型转换，只不过使用std::forward转发语义会表达得更加清晰。

    实现如下：

    ```cpp
    template <typename T>
    T&& forward(typename std::remove_reference<T>::type& param)
    {
        return static_cast<T&&>(param);
    }

    template <typename T>
    T&& forward(typename std::remove_reference<T>::type&& param)
    {
        return static_cast<T&&>(param);
    }
    ```

    forward实现了两个模板函数，一个接收左值，另一个接收右值。std::forward模板函数对传入的参数进行强制类型转换，转换的目标类型符合引用折叠规则，因此左值参数最终转换后仍为左值，右值参数最终转成右值。




