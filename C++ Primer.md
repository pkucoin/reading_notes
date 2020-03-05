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
- 从实现上说，lambda函数会创建一个对应的匿名对象，其成员是捕获的变量，变量以lambda声明时的方式（值传递/引用传递）初始化，其值为lambda声明时变量的值；其成员函数是operator()的重载函数，函数体是该lambda函数，默认为const成员函数。
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

## 10.4 再探迭代器
## 10.5 泛型算法结构
## 10.6 特定容器算法
- 链表（list和forward_list）有自己的sort，merge，remove，reverse和unique

# 11. 关联容器
## 11.1 使用关联容器
## 11.2 关联容器概述
- 有序关联容器的关键字类型需要定义一个严格弱序的<运算符
## 11.3 关联容器操作
```cpp
// map迭代器指向一个pair，其值可以改变，但其关键字不能改变
map<string, int> m = {{pre", 1}};
auto m_it = m.begin();
//m_it->first = "after"; // error
m_it->second = 2;
// set也不能通过迭代器修改关键字
set<int> s = {1};
auto s_it = s.begin();
//*s_it = 2; // error
// insert的返回值是一个pair，second是插入是否成功，first是插入成功后指向该元素的迭代器
auto ret = m.insert({"pre", 2}); // ret type: pair<map<string, int>::iterator, bool>
// 使用不存在的关键字作为下标会导致具有该关键字的元素添加到map中
if (m["after"] == 1) {
   // m : {{"pre", 1}, {"after", 0}}
}
// 使用equal_range查找可重复关联容器中所有具有特定关键字的元素
multimap<string, int> mm = {{"pre", 1}, {"pre", 2}};
auto pos = mm.equal_range("pre"); // pos是一个pair，内容是一对迭代器，分别对应lower_bound和upper_bound的结果
```

## 11.4 无序容器
```cpp
// 自定义类型的无序容器定义
unordered_set<MyType, decltype(hasher)*, decltype(eqOp)*> us;
```

# 12. 动态内存
## 12.1 动态内存与智能指针
- shared_ptr的创建和基本使用
```cpp
// 使用make_shared创建shared_ptr
shared_ptr<string> p = make_shared<string>(10, '9');
auto q = make_shared<string>("42");
p = q; // 调用string(10, '9')的析构函数，并释放内存
vector<shared_ptr<string>> vs;
vs.push_back(p); // "42"一直存在直到被从vs中erase掉或vs的生存期结束
```
- shared_ptr多用于让多个对象能够共享一份数据
- 使用new和delete直接管理内存容易出错
```cpp
string *ps1 = new string; // 默认初始化，空string
string *ps2 = new string(); // 值初始化，空string
string *ps3 = new string(3, '9'); // 调用string的构造函数，"999"
int *pi1 = new int; // 默认初始化，未定义的值
int *pi2 = new int(); // 值初始化，0
auto pi3 = new const int(42); // pi3是一个指向const int的指针（const int*）
auto po = new (nothrow) MyObject; // placement new，nothrow告诉new失败不抛出bad_alloc异常，返回空指针
shared_ptr<int> pi4(new int(42)); // 接受指针参数的构造函数是explicit的
void process(shared_ptr<int> ptr);
process(pi4); // ok
//process(pi2); // error: 不能将int*转换为一个shared_ptr<int>
process(shared_ptr<int>(pi2)); // 危险，混用了普通指针和shared_ptr，pi2指向的内存将在执行完后被释放！
//int j = *pi2; // error: pi2是空悬指针
// shared_ptr<int> pi5(pi4.get()); // 不要将shared_ptr.get()获得的指针用于初始化另一个shared_ptr
```
- unique_ptr用于对一个对象的独占权：一个时刻只能有一个unique_ptr指向一个给定对象
```cpp
unique_ptr<int> p1(new int(42));
//unique_ptr<int> p2(p1), p3; // error: unique_ptr不支持拷贝
//p3 = p1; // error: unique_ptr不支持赋值
unique_ptr<int> p2(std::move(p1)); // ok: unique_ptr可以移动
unique_ptr<int> p3(7);
p3.reset(p2.release()); // 将p2指向的对象的所有权转移给p3
//p3.release(); // error: p3将丢失指针，指针指向的内存将一直不会被释放
```
- weak_ptr是指向一个由shared_ptr管理的对象的"弱引用"，以便可以检查该对象是否仍然存在，同时又不会意外地延长对象生存期
```cpp
weak_ptr<int> wp;
shared_ptr<int> check;
{
   auto sp = make_shared<int>(42);
   wp = sp;
   if (check = wp.lock()) { // return true
      ...
   }
}
if (check = wp.lock()) { // return false
   ...
}
```

## 12.2 动态数组
- new与数组
```cpp
int *a = new int[10]; // 默认初始化，元素是未定义的值
int *b = new int[10](); // 值初始化，元素是0
int *c = new int();
delete [] a; // 数组中的元素按逆序销毁
delete b; // UB
delete [] c; // UB
```
- allocator

# 13. 拷贝控制
- 类的拷贝控制操作包括对象的拷贝、移动、复制、销毁，是通过copy ctor, copy-assignment operator, move ctor, move-assignment operator, dtor这5个函数来实现的

## 13.1 拷贝、赋值与销毁
- 直接初始化是以函数匹配的方式在构造函数中选择与所提供参数最匹配的。拷贝初始化是将右侧运算对象拷贝到正在创建的对象中。
```cpp
string dots(10, '.');
// 在没有优化存在的情况下，以下语句均调用了拷贝构造函数
// 但由于拷贝消除的存在，在函数返回类类型对象、使用临时对象进行拷贝时都可能省略拷贝
string s1(dots); 
string s2 = dots; 
string s3 = ".........."; 
string s4 = string(10, '.');
string fun(string a) { return a };
fun(dots);
vector<string> strs;
strs.push_back(dots);
strs.emplace_back(dots);
```
- （包括拷贝在内的）赋值运算符通常返回一个指向左侧对象的引用（\*this）
- 析构函数在对象被销毁时被调用。对象的引用或普通指针离开作用域时不会调用对象的析构函数。智能指针在引用计数为0时会调用
```cpp
{ // 一个新的作用域
   Data *p1 = new Data;
   Data &ref_data = *p1;
   auto p2 = make_shared<Data>();
   Data d(*p1);
   vector<Data> vec = {*p2};
   // delete p1; // 普通指针只有主动delete时才会调用析构函数
}
// 离开作用域时，p1和ref_data指向的同一对象不会析构
// p2, d, vec的元素均会析构
```
- 合成析构函数通常为空。析构函数本身并不销毁成员。         
- 三五法则：这里的三指三个控制类拷贝的操作（copy ctor, copy assignment operator, dtor）；五还包括另外两个移动操作(move ctor, move assignment operator)
- 三五法则a：需要析构函数的类也需要拷贝和赋值操作
  - 需要析构意味着需要在销毁时释放成员的内存，合成的拷贝和赋值操作会直接拷贝这些成员指针，造成use after free/delete两次等问题
  - 显然反之并不成立：需要拷贝和赋值操作的类并不一定需要析构函数
- 三五法则b：需要拷贝操作的类也需要赋值操作，反之亦然
- =default和=delete
  - =default只能用于生成默认构造/拷贝构造函数，=delete可以用于除了析构函数以外（用于析构语法上没问题，但没有意义）的函数
  - =default可以在类内声明处或类外定义处，=delete必须在声明处
- 当类的某个成员是不能拷贝赋值或销毁的时，对应的合成拷贝控制成员函数就是=delete的


## 13.2 拷贝控制和资源管理
- 定义类时应明确类对象的拷贝语义：像值一样拷贝、像指针一样拷贝、不允许拷贝
- 拷贝赋值运算符需要保证将对象赋予自身时也是正确的，即使发生异常也能让左侧对象处于有意义的状态
```cpp
HasPtr& HasPtr::operator(const HasPtr &rhs)
{
   // 先将右侧对象拷贝到临时对象，然后销毁自己的对象，最后拷贝
   // 如果交换第一二步的顺序，当rhs就是自己时就会出错
   auto newp = new string(*rhs.ps);
   delete ps;
   ps = newp;
   return *this;
}
```

## 13.3 交换操作
- 交换指针而非成员本身可以提高swap效率
- 使用copy and swap的赋值运算符是异常安全的，并且能正确处理自赋值
```cpp
HasPtr& HasPtr::operator=(HasPtr rhs) // 参数是值传递的
{
   swap(*this, rhs);
   return *this;
}
```
## 13.6 对象移动
- 表达式分类：一个表达式要么返回左值lvalue，要么返回右值rvalue。
  - 左值lvalue：左值有持久的状态。变量、返回左值引用的函数、赋值、下标、解引用、前置递增递减运算符都是左值的例子。
  - 右值rvalue：右值有短暂的状态。字面常量、临时对象、返回非应用类型的函数、算数、关系、位、后置递增递减运算符都是右值的例子。
- 右值引用：顾名思义是指向右值的引用，即一个右值的另一个名字。右值引用指向的对象即将被销毁，没有其他用户，故右值引用可以自由接管该对象
- 标准库函数std::move将一个左值转换为对应的右值引用类型。转换后的源对象可以销毁或赋予新值，但不能直接使用该对象的值
```cpp
// 移动构造函数接受一个右值引用参数，从该对象中直接接管资源，并将源对象置为有效状态（可析构，且不指向被移动的资源）
StrVec::Strvec(StrVec &&s) noexcept : ele(s.ele) { // noexcept承诺不抛出异常
   s.ele = nullptr;
}

// 移动赋值运算符
StrVec& Strvec::operator=(StrVec &&rhs) noexcept
{
   if (this != &rhs) { // 正确处理自赋值的情况
      delete ele;
      ele = rhs.ele;
      rhs.ele = nullptr;
   }
}
```
- 合成的移动构造函数和移动赋值运算符
  - 只有在类没有定义任何拷贝控制成员函数并且每个非static成员都可以移动时才会生成...
  - 如果类成员没有定义或是const/引用，或不能合成/访问移动构造函数/析构函数，则类本身的...也是删除的
  - 移动和拷贝自己定义其中一个而不定义另一个，则另一个是删除的
- 同时有拷贝和移动构造函数/赋值运算符时
  - 根据函数匹配规则确定使用哪一个
  - 如果没有移动构造函数，右值也将被拷贝（这样做是安全的是因为拷贝构造函数会1.拷贝原对象2.不会改变原对象的值，也就处于有效状态）
- 更新的三五法则：五个拷贝控制成员应该看做一个整体，定义了任何一个就应该定义所有五个
- 成员函数用于拷贝时，其参数类型应该为const T&（因为不会改变原对象）；用于移动时，应该为T&&（因为会改变原对象）。
- 成员函数参数列表后的引用限定符类似于const，可以指定this是一个左值/右值。如果要定义带有引用限定符的重载版本成员函数，那么所有重载的成员函数都要有引用限定符

# 14. 重载运算和类型转换
```cpp
class MyStr {
   string data;
   int num;
public:
   // 赋值、下标、调用、成员访问运算符必须是成员
   // 复合赋值、递增、递减、解引用通常是成员
   
   // （复合）赋值运算符返回左侧对象的引用
   MyStr& operator=(string &s) {
      data = s;
      return *this;
   }
   
   // 下标运算符通常定义两个版本：返回普通引用和常量引用
   char& operator[](size_t n) { // 普通引用版本用于非const对象，可以修改
      return data[n];
   }
   const char& operator[](size_t n) const { // 常量版本用于const对象，不可修改
      return data[n];
   }
   
   // 递增递减通过参数区分前后置
   // MyStr ms; 
   // ms++(0); // 后置
   // ms++(); // 前置
   MyStr& MyStr::operator++() { // 无参数，表明是前置版本，返回引用
      ++num;
      return *this;
   }
   MyStr MyStr::operator++(int) { // 增加一个额外参数，表明是后置版本，返回的是值
      auto ret = *this;
      ++*this;
      return ret;
   }
   
   // 定义了函数调用运算符的对象成为函数对象（functor）
   void operator()() {
   }
   
   // 类型转换运算符通常为const成员，参数列表必须为空
   // 必须不声明返回值类型，其返回值类型与类型名一致；
   // 声明为explicit防止隐式转换，但用作条件判断时仍会隐式调用
   // 如果同时存在A->B和B->A的类型转换会引起二义性用
   explicit operator double() const {
      return static_cast<double>(num);
   }
};

// 重载输入运算符必须处理异常，并在发生错误时保证对象处于正确状态
istream &operator>>(istream &is, MyStr &ms) {
}

// 标准库类型function<retType(args)>统一定义了具有相同调用形式（参数个数、顺序、类型和返回值类型）的可调用对象，这些对象的实际类型可以是函数指针、functor、lambda等等不同的类型
map<string, function<int(int, int)>> ops = {
   {"+", add}, // 函数指针
   {"-", std::minus<int>()},  // 标准库functor
   {"/", divide()}, // 用户自定义functor
   {"*", [](int i, int k) { return i * j; }}, // 匿名lambda
   {"%", mod} }; // 命名lambda
};

// 重载函数的名字会已引起歧义，使用函数指针或lambda可以指明究竟是哪一个版本
int (*fp_add)(int, int) = add;
ops["+"] = [](int a, int b) { return add(a, b); };
```

# 15. 面向对象程序设计
## 15.1 OOP概述
- 数据抽象：类的接口与实现分离
- 继承：相似类型的层次关系。通过虚函数，派生类定义自己的操作
- 动态绑定：以统一的形式（基类指针/引用）使用继承关系的类型各自定义的操作

## 15.2 定义基类和派生类
- 虚函数：基类希望派生类override的成员函数，使用基类指针/引用调用时将被动态绑定
  - 构造函数之外的非静态函数，只能出现在类内部声明语句中
  - 派生类并不一定override基类的虚函数（此时则像普通成员函数那样继承）
  - 派生类不一定要再次声明virtual，可以在参数列表后使用override关键词
```cpp
class Base {
private:
   int b_int;
protected:  
   string b_str;
public:
   Base() = default;
   Base(int i, string &s) : b_int(i)
   virtual int fun(int t);
   virtual ~Base() = default; // 析构函数总应该是虚函数
};

class Derived; // 派生类的前置声明也不需要派生列表
class Derived final : public Base { // 1. 此时Base必须已经final阻止了Derived再被继承
private:
   double d_double;
public:
   Derived() = default;
   // 使用基类的构造函数初始化基类成员，然后按声明顺序初始化派生类成员
   Derived(int i, string &s, double d) : Base(i, s), d_double(d) {} 
   int fun(int t) override; // 使用override关键字可以在没有正确override（参数或返回类型不一致）的时候引发编译报错
};

// 派生类到基类的转换
Base b;
Derived d;
// 基类的指针和引用静态类型和动态类型可能不一致，动态类型在运行时才确定
// 派生类向基类的自动类型转换只对指针/引用有效
Base *p = &b;
p = &d;
p->fun(1); // 调用派生类fun
p->Base::fun(1); // 强制调用基类fun
Base &r = d;
Derived *pd = &b; // error: 不能将基类转换成派生类
b = d; // 调用Base::operator=(const Base&)，只会拷贝基类成员，派生类的成员被忽略
```

## 15.3 虚函数
- 通过对象的指针/引用访问虚函数，直到运行时才会确定调用哪个版本的函数
- 基类声明virtual的函数在其所有派生类中都是虚函数，无论在派生类是否声明为virtual。基类可以在成员函数声明final来阻止派生类override该函数

## 15.4 抽象基类
- 含有（或未经override直接继承）纯虚函数的类是抽象基类。不能创建抽象基类的对象

## 15.5 访问控制与继承
- 访问控制控制的对象是基类成员和派生类继承自基类的成员，访问者分为三类：一是类的成员函数/友元；二是类的用户（创建对象并通过成员运算符访问）；三是类的派生类
- 基类成员的访问说明符和派生类派生列表的访问说明符共同决定了派生类对继承成员的访问权限：
   - public继承：基类成员看做派生类中访问权限一致的成员
   - protected继承：基类中public与protected成员看做派生类的protected成员
   - private继承：基类中public与protected成员变为派生类的private成员
- 在基类公有成员可以被用户代码访问时才能进行派生类向基类的转换，反之则不可
- struct类的成员默认是public，派生struct类默认是public继承；class则均为private

## 15.6 继承中的类作用域
- 一个对象/指针/引用的静态类型决定了它可以访问对象的那些成员
- 派生类作用域位于基类作用域之内
- 派生类成员/成员函数将隐藏与其同名的基类成员/成员函数（无论参数是否一致）

## 15.7 构造函数与拷贝控制
- 基类的dtor必须是virtual。否则delete一个指向派生类对象的基类指针时将无法确定调用基类还是派生类的dtor，是UB
- 定义了析构函数的类不会有合成的移动操作
- 当一个基类的拷贝控制成员函数是delete或不可访问时，对应的派生类成员函数是delete的
- 派生类的拷贝/移动ctor/赋值需要处理包括基类在内的所有成员，而析构只负责销毁由派生类自己分配的资源（因为会逆序调用整个继承体系上的析构函数）
- 在ctor/dtor中调用虚函数，应该执行所在类的虚函数版本
- 派生类可以使用using Base::Base; 来继承直接基类的构造函数

## 15.8 容器与继承
- 在容器中存放具有继承关系的对象时，存放基类（智能）指针

# 16. 模板与泛型编程
## 16.1 定义模板
```cpp
// 函数模板
template <typename T>
int cmp(const T &v1, const T &v2)
{
   if (v1 == v2) return 0;
   
}
```
