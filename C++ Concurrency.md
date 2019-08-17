# 目的

这里记录对C++并发的学习，学习材料主要是《C++ Concurrency in Action，2nd Edition》和cppreference.com。两者均没有质量能看的中文版本，所以看的都是英文原版。看到了不少对前者质量的争议，但是总的来说并没有看到什么基础的、常识的或者直接违背标准的错误，而直接以modern C++为基础系统介绍并发的书籍甚少，perfbook也推荐了这一本，所以还是值得一学的。cppreference.com可以看做C++标准说人话的版本加上质量不错的示例，是学习时看到语焉不详的地方时一个不错的补充。

不同于纯粹的编程语言语法学习，学习并发本身需要对操作系统中的进程、线程、锁以及体系结构中CPU Cache、指令重排等基础知识也有一定了解。当然，我认为学习并发的目的只是为了能在业务并发层面有所精进，并不是为了写内核、编译器、基础库这一级别的代码，所以很多内容点到为止即可。

# 并发概述

# 线程对象基础
std::thread类是C++ 11对操作系统线程的一个wrapper类，其构造函数原型为：
```cpp
template< class Function, class... Args > 
explicit thread( Function&& f, Args&&... args );
```
看起来很眼熟？是的它和std::bind的函数原型很像，f是一个Callable的object，模板参数包args通过std::forward转发，最终在另一个线程中通过std::invoke调用。以下是常见的创建一个线程对象的参数形式：
```cpp
std::thread t1; // t1 is not a thread
std::thread t2(f1, n + 1); // pass by value
std::thread t3(f2, std::ref(n)); // pass by reference
std::thread t4(std::move(t3)); // t4 is now running f2(). t3 is no longer a thread
std::thread t5(&Foo::bar, &foo); // t5 runs foo::bar() on object f
std::thread t6(b); // t6 runs baz::operator() on object b
//std::thread t7(baz()); // error, most vexing parse
std::thread t7{baz()}; // or t7{baz{}} or t7(baz{})
std::thread t8([](){ b(); }); // lambda
```

- 除了使用默认初始化的t1和被std::move移动构造到了t4的t3外，其他线程对象在构造函数完成后即创建线程开始执行，这称为“与一个系统线程关联”，或者“已关联线程对象”。
- 线程对象不可以拷贝，但可以移动。移动可以转移线程对象与系统线程的关联关系，也支持了将线程对象置于容器内的用法。
- 可以对已关联线程对象进行join()来等待其完成，或者detach()使其独立执行。只能对已关联线程对象执行一次join或detach，执行后joinable()会返回false
- 线程对象的当线程对象析构时，如果与一个系统线程关联并且是joinable，但没有被join()或者detach()，则会调用std::terminate()终止程序。为了确保线程对象创建后一定被join/detach，可以利用RAII构造一个简单的wrapper类在析构函数中完成（相比而言还是Go的defer简洁好用呀，希望同样效果的关键字能够出现在之后的标准中）。
```cpp
// 一个支持移动、自动join的线程类
class joining_thread
{
    std::thread t;
public:
    joining_thread() noexcept=default;
    template<typename Callable,typename ... Args>
    explicit joining_thread(Callable&& func,Args&& ... args):
        t(std::forward<Callable>(func),std::forward<Args>(args)...) {}
    explicit joining_thread(std::thread t_) noexcept: t(std::move(t_)) {}
    joining_thread(joining_thread&& other) noexcept: t(std::move(other.t)) {}
    joining_thread& operator=(joining_thread&& other) noexcept
    {
        if(joinable())
            join();
        t=std::move(other.t);
        return *this;
    }
    joining_thread& operator=(std::thread other) noexcept
    {
        if(joinable())
            join();
        t=std::move(other);
        return *this;
    }
    ~joining_thread() noexcept
    {
        if(joinable())
            join();
    }
    // 省略若干成员函数
    ...
};
```

# 线程间共享数据
并发中的race condition指的是程序运行结果依赖于线程间操作的相对执行顺序。避免race condition的方法主要有：
- lock-based: 使用某种锁保护机制将数据保护起来，使得同一时刻只有一个操作线程可以修改数据
- lock-free: 使用原子数据结构、CAS指令和内存模型，使修改对不同线程有正确的可见性
- transaction-based: 使用事务，发生错误时回滚并重试

## mutex的lock与unlock
- 线程A对mutex成功lock之后，另一个线程B尝试对这个已经lock的mutex执行lock会导致线程B阻塞直到lock成功，从而达到互斥目的
- 线程A对mutex成功lock之后，线程A再次对该mutex调用lock是UB
- 只有在本线程对mutex成功lock之后，才能由本线程对mutex调用unlock。如果本线程没有lock成功，或者mutex是由其他线程lock的，此时由本线程对mutex调用unlock是UB
- 为了保证mutex在加锁后一定会被解锁，不应该直接调用mutex的lock和unlock，而应该选择合适的RAII wrapper，包括lock_guard,unique_lock, scoped_lock等
- lock_guard是一个纯粹的mutex RAII wrapper，构造时lock()，析构时unlock()
- 相比lock_guard，unique_lock的构造函数提供了对mutex更精细的控制

## 死锁的四个必要条件：
- 资源互斥：同一时刻一个资源只有一个持有者可以持有
- 持有并等待：持有者持有资源的同时也在等待资源
- 不可剥夺：除非持有者自己释放，持有者对资源的持有不可剥夺
- 环形等待：资源持有者们等待资源的关系形成环

## 死锁避免
- 在实际代码中一个直接的避免死锁的做法就是所有线程必须以相同的加锁顺序对mutex加锁，但实际中这可能很难保证
- 在C++ 11中引入的std::lock()提供了一种使用死锁避免算法同时为多个mutex上锁的方法，在C++ 17中又引入了std::scoped_lock作为std::lock()的RAII wrapper
- 书中介绍了一个层次锁的实现，通过锁的层次大小来限定上锁顺序来避免死锁
```cpp
// 一个交换例子，说明unique_lock和scoped_lock
void swap(X& lhs, X& rhs)
{
    if(&lhs==&rhs)
        return;
    // 构造时不锁
    std::unique_lock<std::mutex> lock_a(lhs.m, std::defer_lock);
    std::unique_lock<std::mutex> lock_b(rhs.m, std::defer_lock);
    std::lock(lock_a,lock_b);
    swap(lhs.some_detail,rhs.some_detail);
    // 构造时当前线程已经取得了锁的所有权
    std::lock(lhs.m,rhs.m);
    std::lock_guard<std::mutex> lock_a(lhs.m,std::adopt_lock);
    std::lock_guard<std::mutex> lock_b(rhs.m,std::adopt_lock);
    swap(lhs.some_detail,rhs.some_detail);
    // 更简洁的写法：使用scoped_lock与类模板实参推导
    std::scoped_lock guard(lhs.m,rhs.m); // 省略了模板类型参数
    swap(lhs.some_detail,rhs.some_detail);
}
```

## 初始化数据的多线程保护
对于数据初始化的多线程保护最好的例子是lazy-init的单例模式。Meyers在2004年的《C++ and the Perils of Double-Checked Locking》一文（下简称04文）中详细讨论了这一问题。首先在单线程下lazy_init的单例模式示例代码:
```cpp
class Singleton {
public:
    static Singleton* getInstance() {
        if (!inst) {
            inst = new Singletion();
        }
        return inst;
    }
private:
    Singleton();
    ~Singleton();
    static Singleton* inst;
};
```
直接改写成多线程：
```cpp
    static Singleton* getInstance() {
        std::lock_guard<std::mutex> lck(mtx);
        if (!inst) {
            inst = new Singleton();
        }
        return inst;
    }
```
这样写最大的问题是每次调用getInstance都会进入临界区，而实际上只要第一个线程完成了inst的初始化，之后的线程发现inst非空就不需要再进入临界区。于是出现了Double-Checked Locking的写法：
```cpp
    static Singleton* getInstance() {
        if (!inst) {
            std::lock_guard<std::mutex> lck(mtx);
            if (!inst) {
                inst = new Singleton();
            }
        }
        return inst;
    }
```
初看起来这样写没有问题。但```inst = new Singleton();```这一句存在一个微妙的问题：
- 从C++ 11标准开始，operator new, delete是线程安全的
- 但new实际是一个包含3个步骤的过程
    - 1. 申请内存空间
    - 2. 在申请到的内存空间上完成对象构造
    - 3. 将inst指向申请到的内存起始地址
- 指令重排仍然可能使2和3交换次序，从而导致当线程A完成步骤1后先执行了3然后才执行2。注意这与operator new是线程安全的并不矛盾
- 另一个线程B可能在线程A完成步骤1和3之后观察到inst已非空，从而直接返回了inst指针，而此时线程A还未完成步骤2，inst指向还未完成对象构造的内存空间
- 所有希望通过加入volatile关键字来得到正确的多线程语义的方法都是错误的
- 可以通过加入memory barrier来改写这段程序使其获得正确的语义

由此可见，在C++ 11之前写出一个正确的lazy-init的单例模式并不容易。而在C++ 11之后，有两种简洁的写法可以达到目的，一种是使用静态局部变量，其线程安全来自于C++ 11标准规定：*If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.* 
```cpp
    static Singleton* getInstance() {
        static Singleton* inst = new Singleton();
        return inst;
    }
```
另一种是使用call_once（类似于pthread_once）和once_flag，call_once保证即使有多个线程同时执行，被调用函数也只会执行一次

```cpp
    static Singleton* getInstance() {
        std::call_once(init_flag, ()[]{ inst = new Singleton(); });
        return inst;
    }
private:
    static std::once_flag init_flag;
```
其实个人认为实际应用中单例对象生存期大都是和程序运行期一致，lazy-init并没有什么内存占用之类的优势。直接在main函数/主线程中将单例对象初始化好可能是更好的选择。

## volatile
这可能是被误解最多的C++关键字了。C++标准中的描述涉及到side effects和sequence point，基本上没有说人话。Meyers在04文中介绍了volatile的历史，volatile的目的是为了让编译器知道，在处理某些位于特殊内存位置（例如内存映射I/O，中断访问的内存等）的变量时，编译器优化需要注意：
- volatile变量的值是不稳定的，可能会被编译器不了解的途径意外地改变。这是针对读优化的限制。
```cpp
unsigned int *p = GetMagicAddress();
volatile unsigned int a, b;
a = *p;
b = *p // 这里，不应该将其优化为b = a;
```
- 对volatile变量的写应该是可观测的，这些写操作必须被executed religiously。这是针对写优化的限制。
```cpp
*p = a; // 这一句不能被优化掉
*p = b;
```
- 对volatile变量的所有操作应该以它们在代码中的顺序来执行。这是针对混合读写的限制。

最重要的是，上述限制都是针对单线程模型而言的。C++中volatile与线程安全没有任何关系。将变量声明为volatile，不使用其他同步手段就进行多线程读写是UB。

## 读写锁
在读多写少的应用场景中，每次读也使用互斥锁造成了不必要的锁争用开销。读写锁的设计便是为了解决这一问题：同一时刻它仍然只允许一个写线程修改数据，但是允许多个读线程同时读取数据。
