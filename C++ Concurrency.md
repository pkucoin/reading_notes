# 目的

这里记录对C++并发的学习，学习材料主要是《C++ Concurrency in Action，2nd Edition》和cppreference.com。两者均没有质量能看的中文版本，所以看的都是英文原版。看到了不少对前者质量的争议，但是总的来说并没有看到什么基础的、常识的或者直接违背标准的错误，而直接以modern C++为基础系统介绍并发的书籍甚少，perfbook也推荐了这一本，所以还是值得一学的。cppreference.com可以看做C++标准说人话的版本加上质量不错的示例，是学习时看到语焉不详的地方时一个不错的补充。

不同于纯粹的编程语言语法学习，学习并发本身需要对操作系统中的进程、线程、锁以及体系结构中CPU Cache、指令重排等基础知识也有一定了解。当然，我认为学习并发的目的只是为了能在业务并发层面有所精进，并不是为了写内核、编译器、基础库这一级别的代码，所以很多内容点到为止即可。

# 并发概述

# 线程基础
std::thread类是C++ 11对操作系统线程的一个wrapper类，其构造函数原型为：
```
template< class Function, class... Args > 
explicit thread( Function&& f, Args&&... args );
```
看起来很眼熟？是的它和std::bind的函数原型很像，f是一个Callable的object，之后的模板参数包args通过std::forward转发，最终通过std::invoke调用。


