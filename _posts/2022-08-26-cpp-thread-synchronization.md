---
title: C++ 线程同步
tags: C++
---



本文介绍多线程同步相关内容。

<!--more-->

### 1 线程同步

+ **线程同步**：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作， 其他线程才能对该内存地址进行操作，而其他线程又处于等待状态，实现线程同步的方法有很多，临界区对象就是其中一种。

+ 线程同步的必要性

    线程间为什么需要同步？直接来看一个例子：

    ```cpp
    int a = 0;
    void foo()
    {
        for (int i = 0; i < 10000000; ++i)
        {
            a += 1;
        }
    }
    int main()
    {
        clock_t start, end;
        start = clock();
        thread t1(foo);
        thread t2(foo);
        t1.join();
        t2.join();
        end = clock();
        cout << a << endl;
        cout << end - start << endl;
        return 0;
    }
    ```

    输出结果如下：

    ```cpp
    10514859
    140297
    ```

    a的最终结果为10514859，共使用了140297毫秒的时间。在两个线程对a进行自增的过程中可能会因为线程调度的问题使得最终结果并不正确。比如当前a的值为1，线程x现在将a的值读到寄存器中，而线程y也将a读到寄存器中，完成了自增并将新的值放入内存中，现在a的值为2，而线程x现在也对寄存器中的值进行自增，并将得到的结果放入内存中，a的值为2。可以看到两个线程都对a进行了自增，但是却得到的错误的结果。

    这种情况便需要对线程间进行同步。

+ 线程间的同步方式总共有5种， 分别是**互斥锁**，**自旋锁**，**读写锁**，**条件变量**和**屏障**。

    
### 2 互斥锁

+ 操作系统中提供一把互斥锁mutex（也称之为互斥量）。

+ 每个线程在对资源操作前都尝试先加锁，成功加锁才能操作，操作结束解锁。但通过“锁”就将资源的访问变成互斥操作，而后与时间有关的错误也不会再产生了。

+ 应注意：**同一时刻，同一临界区，只能有一个线程持有该锁**。

+ 当A线程对某个全局变量加锁访问，B在访问前尝试加锁，拿不到锁，B阻塞。C线程不去加锁，而直接访问该全局变量，依然能够访问，但会出现数据混乱。

+ 所以，互斥锁实质上是操作系统提供的一把“建议锁”（又称“协同锁”），建议程序中有多线程访问共享资源的时候使用该机制。但，并没有强制限定。

+ 因此，即使有了mutex，如果有线程不按规则来访问数据，依然会造成数据混乱。

+ 在 C++ 中， 包含mutex头文件后，就可以使用mutex类，实现互斥锁。

+ mutex头文件包含了以下内容：

    >   mutex类4种
    >   +   std::mutex，最基本的 Mutex 类。
    >   +   std::recursive_mutex，递归 Mutex 类。
    >   +   std::time_mutex，定时 Mutex 类。
    >   +   std::recursive_timed_mutex，定时递归 Mutex 类。
    >
    >   Lock 类（两种）
    >   +   std::lock_guard，与 Mutex RAII 相关，方便线程对互斥量上锁。
    >   +   std::unique_lock，与 Mutex RAII 相关，方便线程对互斥量上锁，但提供了更好的上锁和解锁控制。
    >   其他类型   
    >   +   std::once_flag
    >   +   std::adopt_lock_t
    >   +   std::defer_lock_t
    >   +   std::try_to_lock_t
    >
    >   函数
    >   +   std::try_lock，尝试同时对多个互斥量上锁。
    >   +   std::lock，可以同时对多个互斥量上锁。
    >   +   std::call_once，如果多个线程需要同时调用某个函数，call_once 可以保证多个线程对该函数只调用一次。 


+ 我们使用以下代码定义互斥锁：

    ```cpp
    mutex _mutex;
    _mutex.lock();//加锁
    _mutex.unlock();//解锁
    _mutex.try_lock();//尝试加锁，成功返回bool，失败返回false不阻塞
    ```
    
    现在使用互斥锁来实现两个线程对同一变量自增的功能：

    ```cpp
    int a = 0;
    mutex _mutex;
    void foo()
    {
        for (int i = 0; i < 10000000; ++i)
        {
            _mutex.lock();
            a += 1;
            _mutex.unlock();
        }
    }
    int main()
    {
        clock_t start, end;
        start = clock();
        thread t1(foo);
        thread t2(foo);
        t1.join();
        t2.join();
        end = clock();
        cout << a << endl;
        cout << end - start << endl;
        return 0;
    }
    ```

    输出结果如下:

    ```cpp
    20000000
    5449543
    ```

    可以看到，上述程序得到了正确的输出结果，但时间确实一开始例子的几十倍，原因如下：

    1. 锁的争用造成了线程阻塞

    2. 互斥锁的获取需要陷入内核态，即每次上锁解锁，都需要从用户态进入内核态，再回到用户态。而foo函数本身执行自增操作只需要两条指令就能完成，而内核态的切换可能需要上百条指令。

+ 互斥锁缺点：

    1. 等待互斥锁会消耗时间，等待延迟会损害系统的可伸缩性。

    2. 优先级倒置。低优先级的线程可以获得互斥锁，因此会阻碍需要同一互斥锁的高优先级线程。

    3.  锁护送（lock convoying）。如果持有互斥锁的线程分配的时间片结束，线程被取消调度，则等待同一互斥锁的其它线程需要等待更长时间。

### 3 自旋锁

+ 自旋锁（spin lock）属于busy-waiting类型锁。在多处理器环境中，自旋锁最多只能被一个可执行线程持有。如果一个可执行线程试图获得一个被其它线程持有的自旋锁，那么线程就会一直进行忙等待，自旋（空转），等待自旋锁重新可用。如果自旋锁未被争用，请求锁的执行线程便立刻得到自旋锁，继续执行。

+ 获取锁的线程一直处于活跃状态，但并没有执行任何有效的任务，使用自旋锁会造成busy-waiting。

+ 自旋锁特点:

    1. 自旋锁不会使线程状态发生切换，一直处于用户态，即线程一直都是active的；不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快

    2. 非自旋锁在获取不到锁的时候会进入阻塞状态，从而进入内核态，当获取到锁时需要从内核态恢复，导致线程在用户态与内核态之间来回切换，严重影响锁的性能。

+ 自旋锁原理：

    自旋锁的原理比较简单，如果持有锁的线程能在短时间内释放锁资源，那么等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞状态，只需要等一等(自旋)，等到持有锁的线程释放锁后即可获取，避免用户进程和内核切换的消耗。

    自旋锁避免了操作系统进程调度和线程切换，通常适用在时间极短的情况，因此操作系统的内核经常使用自旋锁。但如果长时间上锁，自旋锁会非常耗费性能。线程持有锁时间越长，则持有锁的线程被 OS调度程序中断的风险越大。如果发生中断情况，那么其它线程将保持旋转状态(反复尝试获取锁)，而持有锁的线程并不打算释放锁，导致结果是无限期推迟，直到持有锁的线程可以完成并释放它为止。

    自旋锁的目的是占着CPU资源不进行释放，等到获取锁立即进行处理。如果自旋执行时间太长，会有大量的线程处于自旋状态占用CPU资源，进而会影响整体系统的性能，因此可以给自旋锁设定一个自旋时间，等时间一到立即释放自旋锁。

+ 在C++11中并没有直接提供自旋锁的实现。但是在C++11中提供了原子操作的实现，可以借助原子操作实现简单的自旋锁。实现代码如下：

    ```cpp
    atomic_flag flag; // 头文件： thread
    int a = 0;
    void foo()
    {
        for (int i = 0; i < 10000000; ++i)
        {
            while (flag.test_and_set())
            {
            
            }//加锁
            a += 1;
            flag.clear();//解锁
        }
    }
    int main()
    {
        flag.clear();//初始化为clear状态

        clock_t start, end;
        start = clock();
        thread t1(foo);
        thread t2(foo);
        t1.join();
        t2.join();
        end = clock();
        cout << a << endl;
        cout << end - start << endl;
        return 0;
    }
    ```

    输出结果如下：

    ```cpp
    20000000
    4827258
    ```

    atomic_flag是一个原子变量，共有set和clear两种状态。在clear状态下test_and_set会将其状态置于set并返回false，在set状态下test_and_set会返回true。可以看到自旋锁其实和CAS方式实现的乐观锁很相似，使用原子操作改变标志为的值，并不断地轮询标志位。

+ 自旋锁和互斥锁的优缺点和使用场景：

    1. 互斥锁不会浪费CPU资源，在无法获得锁时使线程阻塞，将CPU让给其他线程使用。比如多个线程使用打印机等公共资源时，应该使用互斥锁，因为等待时间较长，不能让CPU长时间的浪费。

    2. 自旋锁效率更高，但是长时间的自旋可能会使CPU得不到充分的应用。**在临界区代码较少，执行速度快的时候应该使用自旋锁**。比如多线程使用malloc申请内存时，内部可能使用的是自旋锁，因为内存分配是一个很快速的过程。

### 4 条件变量

+   在C++11中，我们可以使用条件变量（std::condition_variable）实现多个线程间的同步操作；当条件不满足时，相关线程被一直阻塞，并释放CPU，直到某种条件出现，这些线程才会被唤醒。条件变量需要和互斥量（锁）一起搭配使用。

+ 条件变量是利用线程间共享的全局变量进行同步的一种机制，主要包括两个动作：

    1. 一个线程等待条件变量的条件成立而挂起;

    2. 另一个线程使条件成立（给出条件成立信号）。

    为了防止竞争，条件变量的使用总是和一个互斥量结合在一起。

+ 条件变量头文件为`<condition_variable>`

    条件变量有以下两种：
    1. `condition_variable`

    2. `condition_variable_any`

    相同点：两者都能与std::mutex一起使用。

    不同点：前者仅限于与 std::mutex 一起工作，而后者可以和任何满足最低标准的互斥量一起工作，从而加上了_any的后缀。condition_variable_any会产生额外的开销。

    一般只推荐使用`condition_variable`。除非对灵活性有硬性要求，才会考虑`condition_variable_any`。

+ 常用的成员函数：

    1. `wait()`:阻塞当前线程，直到条件变量被唤醒。

    2. `wait_for`:阻塞当前线程，直到条件变量被唤醒或到达指定时长后。

    3. `wait_until`:阻塞当前线程，直到条件变量被唤醒，或知道抵达指定的时间点。

    4. `notify_once`: 随机通知一个等待的线程。

    5. `notify_all`:通知所有等待的线程。

+ 条件变量在C++11中有现成的类可以使用，比unix风格的接口更加方便。用法和unix的条件变量类似，需要配合互斥锁使用。

    现在我们使用C++11的条件变量完成三个线程顺序打印0，1，2：

    ```cpp
    condition_variable cond;
    mutex _mutex;
    int a = 0;
    void first() {

        
        while (true)
        {
            unique_lock<mutex> lck(_mutex);
            while (a % 3 != 0)
                cond.wait(lck);
                // wait(arg_1, arg_2)
                // wait有两个参数， arg_1代表互斥锁
                // arg_2代表 条件 可以为返回值为bool的lambda表达式
                // 当 arg_2 返回false时，将解锁互斥量，并阻塞到本行
                // 当条件变量调用 notify_one() 或者 notify_all()时
                // 解除阻塞
            a++;
            printf("%d\n", a % 3);
            cond.notify_all();
            lck.unlock();
        }
    }
    void second() {

        while (true) {
            unique_lock<mutex> lck(_mutex);
            while (a % 3 != 1)
                cond.wait(lck);
            a++;
            printf("%d\n", a % 3);
            cond.notify_all();
        }
    }
    void third() {

        while (a<100) {
            unique_lock<mutex> lck(_mutex);
            while (a % 3 != 2)
                cond.wait(lck);
            a++;
            printf("%d\n", a % 3);
            cond.notify_all();
        }
    }

    int main()
    {
        thread t1(first);
        thread t2(second);
        thread t3(third);
        getchar();
        return 0;
    }
    ```

    其中unique_lock是对mutex的一种RAII使用手法，unique_lock是lock_guard的加强版，它具有 lock_guard 的所有功能，同时具有如下特点：

    + 创建时可以不锁定（通过指定第二个参数为std::defer_lock）

    + 可以随时加锁解锁（通过lock，try_lock, unlock）

    + 允许延迟锁定（通过try_lock_for），限时锁定（通过 try_lock_until）

    + 不可复制，可移动（通过移动构造函数或移动赋值函数转移所有权）

    + 可以主动释放所有权（通过release）

    + 作用域规则同 lock_grard，析构时自动释放锁

    + 条件变量需要该类型的锁作为参数

    unique_lock也是一个类，如上述代码中的unique_lock<mutex> lck(m);就是创建了一个unique_lock对象lck，并将其与互斥量m绑定，**同时对其上锁**， 创建对象默认加锁。

    与lock_guard不同的是，unique_lock可以进行临时解锁和再上锁，如在构造对象之后使用lck.unlock()就可以进行解锁，lck.lock()进行上锁，而不必等到析构时自动解锁。

    除此之外，unique_lock还接受第二个参数来进行构造。两个参数构造的形式有以下几种：

    + unique_lock<mutex> lck(m,adopt_lock)：用互斥量来初始化unique_lock对象，但是构造时不会自动lock()；

    + unique_lock<mutex> lck(m,defer_lock)：仅仅是将lck与m绑定，不会自动进行lock()和unlock()；

    + unique_lock<mutex> lck(m,try_to_lock)：将lck与m绑定，并且尝试对其进行加锁，如果加锁失败也不会阻塞，加锁是否成功可以根据lck.owns_lock()来判断是否加锁成功；    

    上述用到了条件变量的 `wait` 方法，其有两种重载

    ```cpp
    void wait( std::unique_lock<std::mutex> & lock );
    template <class Predicate>
    void wait(std::unique_lock<std::mutex> & lock, Predicate lock);
    ```

    `wait`使用如下：

    1. wait函数会先对传入的unique_lock进行unlock，并且阻塞当前线程，unlock和block的操作，是一个原子行为。然后将当前线程添加到当前condition_variable对象的等待线程列表中；

    2. 当唤醒线程调用notify_all或者notify_one时，就会解除wait线程的阻塞。解除阻塞之后，就会重新对unique_lock进行lock，然后wait函数返回；

    3. 如果wait函数还传入了第二个参数pred，pred参数应当是个bool型的可调用对象。此时调用wait，会先判断第二个参数的返回值，如果返回false，就会执行第1步，否则直接返回；当被唤醒后再次判断第二个参数的返回值，如果返回false，会再次进入阻塞，否则直接返回；

    4. 由于unique_lock的存在，保护了对可调用对象pred的访问。

    5. 如果调用notify_all或者notify_one，实际上就是唤醒当前condition_variable对象的等待线程列表中的线程。

+ 条件变量的优点：

    支持使用`notify`唤醒`wait`休眠，避免了长时间占用cpu。








