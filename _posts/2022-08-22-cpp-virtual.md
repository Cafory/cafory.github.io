---
title: C++ 虚函数
tags: C++
---


# C++ 虚函数

### 1 虚函数

+ 静态绑定与动态绑定

    > C++在调用函数的时候，会把该调用与合适的函数定义相匹配，这种匹配发生在编译期，也称为**静态绑定**。（编译期确定）
    >
    >也可以告诉编译器，把函数调用与函数定义之间的匹配放到运行期去做，称为**动态绑定**。(运行期确定)


+ 关键字 virtual 指定虚函数
    ```cpp
    virtual func() //虚函数定义
    ```

+ 虚函数是**动态绑定**的，对于指向子类的父类指针，调用虚函数时，只有在运行期间才能确定需要调用哪个子类的实现

+ 虚函数是指一个类中你希望重载的成员函数 ，当你用一个 **基类指针或引用**  指向一个继承类对象的时候，调用一个虚函数时, 实际调用的是继承类的版本。 如果基类函数为声明为虚函数，则指向派生类的基类指针调用此函数时，将调用基类实现，而不是子类实现。

+ 虚函数是C++实现多态的一种方式，基类的函数被声明为`virtual`后，派生类可以根据需要重写基类中的虚函数，实现同一调用方式实现不同效果。如果基类中的函数被声明为纯虚函数，该基类就变成一个抽象类，则派生类必须重写该纯虚函数。

+ 虚函数可以重载

+ 虚函数可以通过`override`关键字进行重写（覆盖）
    ```cpp
    class A
    {
        virtual void func();
    };
    class B : A
    {
        void func() override; // 重写
    }
    ```
+ 使用`override`关键字进行重写的函数，必须是【父类虚函数/纯虚函数】，否则编译不通过。`override`关键字使程序更加严谨。

+ 不使用`override`关键字重写，但在子类中定义了与父类相同的同名函数，则实现了**隐藏**，即子类不会继承父类的缺省实现，只是用自己的实现隐藏了父类的实现，此时，指向子类的父类指针调用此方法，不会引用子类的实现。

+ 虚拟继承（虚基类）

    虚继承主要解决多继承的问题。为了解决从不同途径继承来的同名的数据成员在内存中有不同的拷贝造成数据不一致问题，将共同基类设置为虚基类。这时从不同的路径继承过来的同名数据成员在内存中就只有一个拷贝，同一个函数名也只有一个映射。这样不仅就解决了二义性问题，也节省了内存，避免了数据不一致的问题。

    ```cpp
    using namespace std;
    class Person{
    public:    
        Person(){ cout<<"Person构造"<<ENDL; }
        ~Person(){ cout<<"Person析构"<<ENDL; }
    };
    class Teacher : virtual public Person{
    public:    
        Teacher(){ cout<<"Teacher构造"<<ENDL; }
        ~Teacher(){ out<<"Teacher析构"<<ENDL; }
    };
    class Student : virtual public Person{
    public:      
        Student(){ cout<<"Student构造"<<ENDL; }
        ~Student(){ cout<<"Student析构"<<ENDL; }
    };
    class TS : public Teacher,  public Student{
    public:            
        TS(){ cout<<"TS构造"<<ENDL; }
        ~TS(){ cout<<"TS析构"<<ENDL; }
    };
    int main(int argc,char* argv[])
    {
        TS ts;
        return 0;
    }
    ```

    输出结果

    ```cpp
    // with virtual         no virtual
    Person构造              Person构造        
    Teacher构造             Teacher构造
    Student构造             Person构造
    TS构造                  Student构造
    TS析构                  TS构造
    Student析构             TS析构
    Teacher析构             Student析构
    Person析构              Person构造
                            Teacher析构
                            Person析构
    ```
    在构造TS的时候需要先构造他的基类，也就是Teacher类和Student类。而Teacher类和Student类由都继承于Person类。这样就导致了构造TS的时候实例化了两个Person类。同样的道理，析构的时候也是析构了两次Person类，这是非常危险的。

+ 虚析构函数

    如果一个类用作基类，我们通常需要virtual来修饰它的析构函数，这点很重要。如果基类的析构函数不是虚析构，当我们用delete来释放基类指针(**它其实指向的是派生类的对象实例**)占用的内存的时候，只有基类的析构函数被调用，而派生类的析构函数不会被调用，这就可能引起内存泄露。如果基类的析构函数是虚析构，那么在delete基类指针时，继承树上的析构函数会被自低向上依次调用，即最底层派生类的析构函数会被首先调用，然后一层一层向上直到该指针声明的类型。

    ```cpp
    using namespace std;
    class Person{
    public:        
        Person()  {name = new char[16];cout<<"Person构造"<<ENDL;}
        virtual  ~Person()  {delete []name;cout<<"Person析构"<<ENDL;}
    private:
            char *name;
    };
    class Teacher :virtual public Person{
    public:
        Teacher(){ cout<<"Teacher构造"<<endl; }
        ~Teacher(){ cout<<"Teacher析构"<<ENDL; }
    };
    class Student :virtual public Person{
    public: 
        Student(){ cout<<"Student构造"<<ENDL; }
        ~Student(){ cout<<"Student析构"<<ENDL; }
    };
    class TS : public Teacher,public Student{
    public:
        TS(){ cout<<"TS构造"<<ENDL; }
        ~TS(){ cout<<"TS析构"<<ENDL; }
    };
    int main(int argc,char* argv[])
    {
        Person *p = new TS();
        delete p;
        return 0;
    }
    ```
    输出结果：
    ```cpp
    // with virtual    no virtual deconstruction function
    Person构造          Person构造
    Teacher构造         Teacher构造
    Student构造         Student构造
    TS构造              TS构造
    TS析构              Person析构
    Student析构         程序崩溃
    Teacher析构
    Person析构
    ```
    对于没有虚析构的类来讲，指向Ts类的Person父类指针进行析构时，只会调用Person类的析构函数而不调用Ts类析构函数，从而会造成内存泄漏等未知的结果。

### 2 纯虚函数

+ 在基类中仅仅给出声明，不对虚函数实现定义，而是在派生类中实现。这个虚函数称为纯虚函数。普通函数如果仅仅给出它的声明而没有实现它的函数体，这是编译不过的。纯虚函数没有函数体。纯虚函数需要在声明之后加个=0；

    ```cpp
    class <基类名>
    {
        virtual <返回类型><函数名>(<参数表>)=0; 
    };
    ```

+ 子类必须重写父类的虚函数，否则编译不通过

### 3 抽象类


+ 含有【纯虚函数】的类被称为抽象类。抽象类只能作为派生类的基类，不能**定义对象**，但可以**定义指针**。在派生类实现该纯虚函数后，定义抽象类对象的指针，并指向或引用子类对象。

+
    > 1）在定义纯虚函数时，不能定义虚函数的实现部分；
    >
    > 2）在没有重新定义这种纯虚函数之前，是不能调用这种函数的。
    >
    > 3）如果派生类没有实现完父类所有的纯虚函数，那么派生类依然是抽象类，
    要么将剩下的纯虚函数实现，要么进一步派生子类在子类中实现。

+ 抽象类的唯一用途是为派生类提供基类，纯虚函数的作用是作为派生类中的成员函数的基础，
并实现【动态多态性】。继承于抽象类的派生类如果不能实现基类中所有的纯虚函数，那么这个派生类也就成了抽象类。因为它继承了基类的抽象函数，只要含有纯虚函数的类就是抽象类。纯虚函数已经在抽象类中定义了这个方法的声明，
其它类中只能按照这个接口去实现。

+ 抽象类可以定义实现普通函数，【只要有纯虚函数的类就是抽象类】。

+ 接口和抽象类的区别

    > 1）中我们一般说的接口，表示对外提供的方法，提供给外部调用。
    > 是沟通外部跟内部的桥梁。也是以类的形式提供的，
    > 但一般该类只具有成员函数，不具有数据成员；
    > 
    >2）抽象类可以既包含数据成员又包含方法。

+ `final`关键字

    + 禁止重写
    ```cpp
    class Super
    {
    public:
        Super();
        virtual void SomeMethod() final;//禁止子类重写该函数
    };
    ```

    + 禁止继承
    ```cpp
    class Super final
    {
    //...... Super 不能被继承
    };
    ```

### 4 虚函数表

+ 虚表是一个**指针数组**，其元素是虚函数的指针，每个元素对应一个虚函数的函数指针。需要指出的是，普通的函数即非虚函数，其调用并不需要经过虚表，所以虚表的元素并不包括普通函数的函数指针。

+ 虚表内的条目，即虚函数指针的赋值发生在编译器的编译阶段，也就是说在代码的编译阶段，虚表就可以构造出来了。

+ 每个包含了虚函数的类都包含一个虚表。

+ 所以如果一个基类包含了虚函数，那么其继承类也可调用这些虚函数，换句话说，一个类继承了包含虚函数的基类，那么这个类也拥有自己的虚表。

+ 虚表的拥有者是类，一个类的所有实例共用一个虚表

+ 示例如下

    ```cpp
    class A {
    public:

        virtual void vfunc1();
        virtual void vfunc2();
        void func1();
        void func2();
    private:

        int m_data1, m_data2;
    };
    ```

![](/My_Assets/virtual_table.png)



### 5 虚表指针

+ 虚表是属于类的，而不是属于某个具体的对象，一个类只需要一个虚表即可。同一个类的所有对象都使用同一个虚表。
为了指定对象的虚表，对象内部包含一个虚表的指针，来指向自己所使用的虚表。为了让每个包含虚表的类的对象都拥有一个虚表指针，编译器在类中添加了一个指针，***__vptr**，用来指向虚表。这样，当类的对象在创建时便拥有了这个指针，且这个指针的值会自动被设置为指向类的虚表。

+ 每个对象都有自己的虚表指针

+ 示例如下, 一个继承类的基类如果包含虚函数，那个这个继承类也有拥有自己的虚表，故这个继承类的对象也包含一个虚表指针，用来指向它的虚表。

![](/My_Assets/virtual_table_1.png)


### 6 虚函数与动态绑定

+ C++的三大特性之一多态，分为静态联编和动态联编，在编译过程中进行联编称为静态联编，编译器必须生成能够在程序运行时选择正确的代码，称为动态联编。

+ 虚函数是动态联编的关键，使用virtual关键字创建虚函数，在派生类中进行重写。发生动态绑定的条件是：
    > 1、只有被定义为虚函数的成员函数才能在派生类重写(不是重载)，非虚函数不能进行动态绑定。
    >
    > 2、必须通过【基类类型】的【指针或引用】进行函数调用（当然，一个实例也可以直接调用本类的函数，但不会有动态绑定）

+   一个类中的虚函数的地址都会储存在虚表中，所以虚表里存的是一个个地址，而类中的虚指针指向虚表，在发生动态绑定时基类虚函数的地址会被替换成子类重写函数的地址。

+ 派生类会继承基类的虚表，父类和子类都有自己的虚表指针，有自己的虚表。

+ 虚表的查找主要靠虚函数指针，当存在指向子类的父类指针调用虚函数时，虚函数指针依然指向的是子类虚函数表，故调用的是子类虚方法。

+ 示例如下:
    ```cpp
    #include<iostream>
    using namespace std;
    
    class A {
    public:
        virtual void vfun1() {
            cout << "vfun1 base" << endl;
        }
        virtual void vfun2() {
            cout << "vfun2 base" << endl;
        }
        void fun1();
        void fun2();
    private:
        //int m_data, m_data;
    };
    class B :public A {
    public:
        void vfun1() {
            cout << "vfun1 son" << endl;
        }
        /*void vfun2() {
            cout << "vfun2 son" << endl;
        }*/
        void fun1() {
            cout << "son_fun1" << endl;
        }
    };
    class C :public B {
    public:
        void vfun2() {
            cout << "C_vfun2" << endl;
        }
        void fun2();
    
    };
    int main() {
        //当父类指针指向子类地址
        //在类创建时会发生动态绑定
        //虚函数在子类重写，父类虚表的函数被子类重写的函数覆盖
        //调用的是子类的重写函数
        A* a = new B;
        a->vfun1();
        //不发生动态绑定，没有实例化派生类，虚表地址没有更改
        A* b = new A;
        b->vfun1();
        //可直接通过虚表调用函数
        //通过函数指针调用
        typedef void(*p)();
        //调用虚函数vfun1()
        //*(int*)a将指针类型转化为四字节的，得到虚表地址
        ((p)(*((int*)*(int*)a + 0)))();
        //调用虚函数vfun2(),没有被重写
        ((p)(*((int*)*(int*)a + 1)))();
        
        return 0;
    }
    ```








