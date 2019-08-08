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
- 线程对象的当线程对象析构时，如果与一个系统线程关联并且是joinable，但没有被join()或者detach()，则会调用std::terminate()终止程序。为了确保线程对象创建后一定被join/detach，可以利用RAII构造一个简单的wrapper类在析构函数中完成（相比而言还是Go的defer简洁好用呀）。
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
并发中的race condition指的是程序运行结果依赖于线程间操作的相对执行执行。避免race condition的方法主要有：
- lock-based: 使用某种锁保护机制将数据保护起来，使得同一时刻只有一个操作线程可以修改数据
- lock-free: 使用原子数据结构、CAS指令和内存模型，使修改对不同线程有正确的可见性
- transaction_based: 使用事务，发生错误时回滚并重试

mutex的名字解释了它的功能：任意时刻只有一个线程能够对其进行lock操作，从而使不同线程互斥。尝试对一个已经lock的mutex执行lock会导致当前线程
