对于这种大而全的大部头书，写读书笔记并不希望变成逐章节的摘抄，而是希望扔掉那些不重要或已经烂熟于心的点，记录最重要或者不了解的点。

# 2. 变量与基本类型
## 2.1 基本内置类型
- 赋给无符号数超出范围的值，结果是对范围取模
```cpp
unsigned char a = -1; // a = 255
```
- 赋给有符号数超出范围的值，是UB
```cpp
char a = 256; // undefined
```
- 不要混用有符号数和无符号数
```cpp
int a = -1;
unsigned int b = 1;
cout << a * b << endl; // 4294967295
```
## 2.2 变量
- 对象是具有某种数据结构的内存空间。本书中变量与对象可以互换
- 初始化是在创建变量变量时赋予其初始值，而赋值是以一个新值替代变量的当前值
- 使用列表初始化时如果初始值存在丢失信息的风险，会报编译错误
```cpp
int a = {0.1}; // error: narrowing conversion of ‘1.0000000000000001e-1’ from ‘double’ to ‘int’ inside { } [-Wnarrowing]
```

## 2.3 复合类型
本节所说的引用均指左值引用。
- 引用即别名。引用并不是一个对象，只是为一个已存在对象所起的另一个名字。引用在定义后不能再绑定另一个对象。
- 指针point to对象。指针本身也是一个对象，允许赋值拷贝，可以指向不同对象，无须赋初值
- 指针与引用，大部分情况下都需要与指向对象的类型一致
- nullptr是一个特殊指针类型（std::nullptr_t）的字面值，NULL是一个定义为0的预编译宏。
```cpp
void f(int *pi)
{
   std::cout << "Pointer to integer overload\n";
}
void f(double *pd)
{
   std::cout << "Pointer to double overload\n";
} 
void f(std::nullptr_t nullp)
{
   std::cout << "null pointer overload\n";
}

f(0); // error: call of overloaded ‘f(int)’ is ambiguous
f(NULL); // error: call of overloaded ‘f(NULL)’ is ambiguous
f(nullptr); // output: null pointer overload
```

## 2.4 const
- 顶层const表示对象本身不能直接被修改，底层const表示对象不能通过指针或引用等方式间接修改
```cpp
int q = 10;
const int* const p = &q; // 左边是底层const，限制不能通过指针p来修改q；右边是顶层const，限制不能修改指针p本身（如上所述，指针也是对象）。
```
- constexpr
