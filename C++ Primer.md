对于这种大而全的大部头书，写读书笔记并不希望变成逐章节的摘抄，而是希望扔掉那些不重要或已经烂熟于心的点，记录最重要或者不了解的点。书中一些知识点的实例不如cppreference.com，故从cppreference.com摘录了一些。此外，C++ 11的不少内容在Effective Modern C++中有更详细的阐释，这里不过多展开。

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
- 内置数组下标可以是负数
```cpp
int a[] = {0, 2, 4, 6, 8};
int *p = &a[2];
p[-2]; // ia[0]
```
- string的c_str()方法返回的常量字符串指针生命期与string对象一致，可能会失效/变化。

## 3.6 多维数组
- 范围for语句遍历多维数组需要引用类型的控制变量

# 4. 表达式
## 4.1 基础
- 对象被用作左值时，用的是对象的身份（在内存中的位置）；被用作右值时，用的是对象的值（内容）
- 只有&&、||、?:、,四种运算符明确规定了求值顺序。在一条表达式中的一处改变了运算对象的值，其他地方不要再使用该运算对象，否则就可能产生UB
## 4.2 算术运算符
- 运算符规则
```cpp
m % (-n) == m % n;
(-m) % n == - (m % n);
```
## 4.3 逻辑和关系运算符
- 不要与true和false进行比较运算

## 4.4 赋值运算符
- 赋值运算符优先级较低
```cpp
while ((i = next_num()) != 42) {
   ...
}
```

## 4.5 递增和递减运算符
- 后置版本需要将原始值保存以便返回，这会有额外开销。除非必须，否则不适用后置版本
- 防止因为改变对象而产生的UB
```cpp
*beg++ // 等价于 *(begin++) ，也等价于 返回*begin然后++begin
*beg = *beg++; // 先求左侧还是右侧的值会导致不同的结果。UB
```

## 4.6 成员访问运算符
## 4.7 条件运算符
## 4.8 位运算符
## 4.9 sizeof运算符
- sizeof(\*ptr)在ptr有类型但为空时也是安全的
- sizeof(MyDataClass::data)
## 4.10 逗号运算符
## 4.11 类型转换
- 隐式转换
- 显式转换。dynamic_cast留到19.2节中介绍
```cpp
// static_cast用于具有明确定义的、非底层const的类型转换
double slope = static_cast<double>(j) / i;
void *p = &d;
double *dp = static_cast<double*>(p);

// const_cast用于改变运算对象的底层const。但不能通过其对常量对象进行写，也不能操作函数指针。
struct type {
    int i;
    type(): i(3) {}
    void f(int v) const {
        // this->i = v;                 // compile error: this is a pointer to const
        const_cast<type*>(this)->i = v; // OK as long as the type object isn't const
    }
};
type t; // if this was const type t, then t.f(4) would be undefined behavior
t.f(4);

void (type::* pmf)(int) const = &type::f; // pointer to member function
// const_cast<void(type::*)(int)>(pmf);   // compile error: const_cast does not work on function pointers
                                        
const int j = 3; // j is declared const
int* pj = const_cast<int*>(&j);
// *pj = 4;      // undefined behavior

// reinterpret_cast converts between types by reinterpreting the underlying bit pattern
int f() { return 42; }
int i = 7;

// pointer to integer and back
std::uintptr_t v1 = reinterpret_cast<std::uintptr_t>(&i); // static_cast is an error
int* p1 = reinterpret_cast<int*>(v1);
assert(p1 == &i);

// pointer to function to another and back
void(*fp1)() = reinterpret_cast<void(*)()>(f);
// fp1(); undefined behavior
int(*fp2)() = reinterpret_cast<int(*)()>(fp1);
std::cout << std::dec << fp2() << '\n'; // safe

// type aliasing through pointer
char* p2 = reinterpret_cast<char*>(&i);
if(p2[0] == '\x7')
  std::cout << "This system is little-endian\n";
else
  std::cout << "This system is big-endian\n";

// type aliasing through reference
reinterpret_cast<unsigned int&>(i) = 42;
std::cout << i << '\n';
```
## 4.12 运算符优先级表

# 5. 语句
## 5.1 简单语句
## 5.2 语句作用域
## 5.3 条件语句
- 在switch的case标签内如果需要定义变量，应该定义在块内已保证作用域正确。
## 5.4 循环
## 5.5 跳转语句
- 除了用于跳出多层循环外，不应使用goto
## 5.6 try语句块和异常处理

# 6. 函数
## 6.1 函数基础
- 局部静态变量在程序的执行路径第一次经过对象定义语句时初始化（该初始化是线程安全的），直到程序终止才被销毁。

## 6.2 参数传递
- 应该把函数不会改变的形参定义成常量引用。定义成普通引用的形参会极大限制实参类型。
```cpp
int fun(int &s) { return 0; }
int foobar(const int &s) { return 1; }
int i = 0;
const int ci = i;
string::size_type ctr = 0;
fun(ci), fun(42), fun(ctr); // error
foobar(i), foobar(ci), foobar(42), foobar(ctr); // pass
```
- 数组作为形参
```cpp
// 以下三种形式，后两者等价于第一种
void fun(const int* a);
void fun(const int[] a);
void fun(const int[10] a);

// 注意括号在指向数组的指针/引用的位置
void fun(int *arr[10]); // arr是一个数组，大小为10，类型为整数指针
void fun(int (*arr)[10]); // arr是一个指针，指向含有10个整数的数组
void fun(int (&arr)[10]); // arr是一个引用，绑定到含有10个整数的数组
void fun(int &arr[10]); // error: 没有引用数组
```

- initializer_list是一种轻量级的代理对象，用于提供对const T类型的对象数组的访问。花括号初始化列表会在对象列表初始化、赋值运算符的右操作数、函数参数、auto、范围for（并且能够进行构造）时自动构造成initializer_list。
```cpp
```

## 6.3 返回类型和return语句
- 可以使用尾置返回类型或decltype来简化复杂返回类型的函数
```cpp
// 以下函数声明等价
int (*func(int i))[10];
auto func(int i) -> int(*)[10];
int a[10];
decltype(a) *func(int i);
```

## 6.4 函数重载
- 名字查找在类型检查之前

## 6.5 特殊用途语言特性
- inline只是向编译器发出的一个请求，编译器可以忽略这个请求。

## 6.6 函数匹配
- 1. 选定候选函数：与被调用函数同名并且其声明在调用点可见的函数。
- 2. 选定可行函数：形参数量与提供的实参数量相等；实参类型与对应的形参类型相同或者能够转换。
- 3. 寻找最佳匹配：该函数每个实参的匹配都不劣于其他可行函数需要的匹配；该函数至少有一个实参的匹配由于其他可行函数提供的匹配。

## 6.7 函数指针

# 7. 类
类的基本思想是数据抽象和封装。
- 数据抽象：类的接口包括用户所能执行的操作；类的实现包括类的数据成员、接口实现的函数体
- 封装：类的接口与实现分离，隐藏了实现细节。用户只能使用接口而无法访问其实现部分。

## 7.1 定义抽象数据类型
- 定义在类内部的函数是隐式的inline函数
- 成员函数和数据成员都是通过this指针来隐式访问的
```cpp
//  默认情况下this的类型是指向非常量对象的常量指针，即T \*const；成员函数通过在参数列表后加上const来表示this是一个指向常量对象的常量指针，即const T \*const。
string Sales_data::isbn() const { return bookNo; }
const Sales_data sd;
sd.isbn();
// 实际上等价于如下形式的伪代码：
string Sales_data::isbn(const Sales_data *const this) { return this->bookNo; }
Sales_data::isbn(&sd); // 如果isbn非const，则this的类型为Sales_data *const，这个this不能指向常量对象sd。也就是说，常量对象及指向常量对象的指针/引用只能调用const成员函数。
```
- 类成员函数在类成员变量编译之后才编译，因此无需在意类成员变量与类成员函数出现的次序。
- return \*this来返回当前对象
- const对象在其构造函数完成初始化后才取得其“常量”属性。
- 在类没有声明任何构造函数时，会自动生成默认构造函数，对类成员使用类内初始值（如果有）/默认初始化。
```cpp
Sales_data() = default;
```

## 7.2 访问控制与封装
- class和struct定义类唯一的区别就是成员的默认访问权限

## 7.3 类的其他特性
- mutable成员在const成员函数中也可以被修改（一个有实际意义的用法是作为保护类成员访问的mutex通常声明为mutable）
- 类内初始值必须使用=或{}的初始化形式
- const成员函数
```cpp
class Screen {
   int r, c;
public:
   const Screen &display() const { cout << "const called" << endl; return *this; } 
   Screen &display() { cout << "nonconst called" << endl; return *this; }
   void set(int _r, int _c) { r = _r; c = _c; }
};

Screen screen;
// 如果只有const版本的display，那么display返回的*this是一个常量引用，不能set
screen.display().set(3, 2);

const Screen const_screen;
// display会根据调用对象是否是const来重载，输出const called
const_screen.display();
```
- 前向声明的类在声明后定义之前是一个不完全类型，此时可以定义该类型的指针或引用，声明以该类型为形参类型/返回类型的函数
- 友元没有传递性

## 7.4 类的作用域
- 编译器处理完类中的全部声明后才会处理成员函数的定义

## 7.5 构造函数再探
- 使用构造函数初始值列表好于先定义然后在构造函数中赋值，原因包括：
  - 省去了一次默认初始化过程
  - 对于const， 引用，或没有默认构造函数的类类型，不能先定义然后赋值
- 成员的初始化顺序与其在初始值列表出现的顺序无关，只与成员的声明顺序一致
- 委托构造函数会先执行受委托的构造函数的初始值列表和函数体，然后才执行自身的函数体
- 默认初始化：参见以下用例中的语法
```cpp
struct T1 { int mem; };
 
struct T2
{
    int mem;
    T2() { } // "mem" is not in the initializer list
};

int n; // static non-class, a two-phase initialization is done:
       // 1) zero initialization initializes n to zero
       // 2) default initialization does nothing, leaving n being zero
 
int main()
{
    int n;            // non-class, the value is indeterminate
    std::string s;    // class, calls default ctor, the value is "" (empty string)
    std::string a[2]; // array, default-initializes the elements, the value is {"", ""}
//  int& r;           // error: a reference
//  const int n;      // error: a const non-class
//  const T1 t1;      // error: const class with implicit default ctor
    T1 t1;            // class, calls implicit default ctor
    const T2 t2;      // const class, calls the user-provided default ctor
                      // t2.mem is default-initialized (to indeterminate value)
}
```
- 值初始化：参见以下用例中的语法
```cpp
struct T1
{
    int mem1;
    std::string mem2;
}; // implicit default constructor
 
struct T2
{
    int mem1;
    std::string mem2;
    T2(const T2&) { } // user-provided copy constructor
};                    // no default constructor
 
struct T3
{
    int mem1;
    std::string mem2;
    T3() { } // user-provided default constructor
};
 
std::string s{}; // class => default-initialization, the value is ""
 
int main()
{
    // int n();             // n is a function that has no args and return int
    int n{};                // scalar => zero-initialization, the value is 0
    double f = double();    // scalar => zero-initialization, the value is 0.0
    int* a = new int[10](); // array => value-initialization of each element
                            //          the value of each element is 0
    T1 t1{};                // class with implicit default constructor =>
                            //     t1.mem1 is zero-initialized, the value is 0
                            //     t1.mem2 is default-initialized, the value is ""
//  T2 t2{};                // error: class with no default constructor
    T3 t3{};                // class with user-provided default constructor =>
                            //     t3.mem1 is default-initialized to indeterminate value
                            //     t3.mem2 is default-initialized, the value is ""
    std::vector<int> v(3);  // value-initialization of each element
                            // the value of each element is 0
}
```
- 内置类型的默认初始化值是未确定值，大部分情况下直接使用该值；而内置类型的值初始化会是0
- 隐式的类类型转换只会自动执行一步。可以使用explicit阻止隐式转换。使用了explicit的构造函数只能用于直接初始化。是否需要使用explicit取决于构造函数的参数类型可能发生的隐式类型转换是否会引起意料之外的歧义
- constexpr构造函数用于生成constexpr对象

## 7.6 类的静态成员
- 类的静态成员在类外定义时不能重复static关键字
- 类的静态成员可以是不完全类型而不仅仅是不完全类型的指针或引用；也可以作为默认实参

# 8. IO库
- io对象不可拷贝或赋值，通常以引用形式作为参数或返回值。读写io对象会改变其状态，故其引用不能是const的

# 9. 顺序容器
## 9.1 顺序容器概述
- 不同顺序容器是在添加删除元素和非顺序访问元素的代价之间做tradeoff

## 9.2 容器库概述
- 容器的拷贝构造，容器及元素类型必须完全一致；以迭代器参数拷贝时只要求元素类型之间可以转换
- 赋值运算符将左边容器内元素替换为右边元素的拷贝，因而指向左边容器元素的迭代器、引用、指针失效
- 除array外，swap只交换容器的内部数据结构，不对容器元素进行拷贝删除或插入，可以保证在常数时间完成，也不会使迭代器、引用、指针失效

## 9.3 顺序容器操作
- emplace函数在容器中直接构造元素。emplace的参数与元素类型的构造函数匹配。
- 除emplace之外的插入操作放入容器的是元素的拷贝
- 顺序容器操作是否会使指向元素的迭代器、引用、指针失效取决于操作是否改变了操作前容器内元素存放的位置

## 9.4 vector对象是如何增长的
- size()是已存放元素个数，capacity()是在不重新分配内存的情况最多可存放元素个数
- shrink_to_fit只是一个请求，并不保证一定退回内存
- vector内存重分配是插入操作可能会导致所有迭代器失效的原因
- 虽然在内存重分配时需要拷贝所有元素，但插入操作的均摊时间复杂度仍然是O(1)

## 9.5 额外的string操作
- string与数值互转：to_string, sto(d|i|l|f)

## 9.6 容器适配器
- adpator接受一种已有的容器类型，使其行为看起来像一种不同的类型

# 10. 泛型算法
## 10.1 概述
- 泛型算法不会执行容器操作，只执行迭代器的操作。故泛型算法不会添加或删除容器内元素

## 10.2 初识泛型算法

## 10.3 定制操作
```cpp
int n = 42;
auto f = [n](ostream &os, string &func_name) mutable -> int { os << "func " + func_name + " called: " << ++n << endl; return n; }; // 值捕获的变量默认以const方式传递，不能改变其值。如果不加mutable则会编译报错。
n = 0;
f(cout, string("f")); // output: func f called: 43。以声明时捕获的值为准
f(cout, string("f"));
cout << n << endl; // 0。值捕获不会改变被捕获变量的值
auto g = bind(f, ref(cout), placeholders::_1);
g(string("g")); // output: func g called: 44。在第一次加1的结果43的基础上再加1
```

## 
