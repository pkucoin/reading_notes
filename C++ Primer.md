对于这种大而全的大部头书，写读书笔记并不希望变成逐章节的摘抄，而是希望扔掉那些不重要或已经烂熟于心的点，记录最重要或者不了解的点。此外，C++ 11的不少内容在Effective Modern C++中有更详细的阐释，这里不过多展开。

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
- constexpr指针是一个常量指针，类似于顶层const。

## 2.5 处理类型
- 正确阅读别名。typedef的语法相对晦涩，使用对应的using会清晰不少。using支持模板别名而typedef不支持。
```cpp
//https://en.cppreference.com/w/cpp/language/typedef

// simple typedef
typedef unsigned long ulong;
using ulong = unsigned long;
 
// the following two objects have the same type
unsigned long l1;
ulong l2;
 
// more complicated typedef
typedef int int_t, *intp_t, (&fp)(int, ulong), arr_t[10];
using int_t = int;
using intp_t = int*;
using fp = (*int)(int, ulong)&;

// the following two objects have the same type
fp 
int a1[10];
arr_t a2;
 
// common C idiom to avoid having to write "struct S"
typedef struct {int a; int b;} S, *pS;
 
// the following two objects have the same type
pS ps1;
S* ps2;
 
// error: storage-class-specifier cannot appear in a typedef declaration
// typedef static unsigned int uint;
 
// typedef can be used anywhere in the decl-specifier-seq
long unsigned typedef int long ullong;
// more conventionally spelled "typedef unsigned long long int ullong;"
 
// std::add_const, like many other metafunctions, use member typedefs
template< class T>
struct add_const {
    typedef const T type;
};
 
typedef struct Node {
    struct listNode* next; // declares a new (incomplete) struct type named listNode
} listNode; // error: conflicts with the previously declared struct name
```
- auto会忽略顶层const，而保留底层const。希望auto是顶层const则需要明确指出。
```cpp
const int ci = i, &cr = ci;
auto b = ci, c= cr, d = &ci; // b和c是整数，d是指向整数常量的指针
const auto e = ci; // e是const int
```
- decltype
```cpp
const int ci = 0, & cj = ci;
decltype(ci) x = 0; // x是const int
decltype(cj) y = x; // y是const int&
int i = 42, *p = &i, &r = i;
decltype(r + 0) b; // b是（未初始化的）int
decltype(*p) c; // error: c是int&，必须初始化
decltype((i)) d; // error: d是int&
```

## 2.6 自定义数据结构
- 类定义后记得分号！

# 3. 字符串、向量和数组
## 3.1 命名空间的using声明
- 头文件不应该使用using声明

## 3.2 std::string
- 不要混用int和string::size_type

## 3.3 std::vector


## 3.4 迭代器
- 容器为空时begin和end返回的是同一个迭代器（off-the-end iterator）
- 一些操作可能使迭代器失效，要避免在循环中使用。例如vector的push_back

## 3.5 数组
- 复杂数组声明可以从数组名字开始由内向外阅读
```cpp
int *ptrs[10]; // ptrs是一个包含10个整形指针的数组
int (*Parray)[10] = &arr; // Parray指向一个包含10个整数的数组
int &refs[10]; // error: 不存在引用数组
int (&arrRef)[10] = arr; // arrRef引用一个含有10个整数的数组
```
