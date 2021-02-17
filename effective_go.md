# 格式
- gofmt自动输出标准格式的缩进和对齐
- 使用tab缩进，没有行长度限制，运算符优先级也更简洁
```golang
x<<8 + y<<16 // 等价于(x<<8) + (y<<16)
```

# 注释

# 命名
- 函数和变量的首字母是否大写决定了在包之外被import之后的可见性
- field的Getter和Setter通常被命名为Field()和SetField()
- 多单词名使用驼峰命名法

# 分号
```golang
// 分行会被隐式地插入在每个语句结尾
if i < f() {
// 如果将{放在这里，会导致分号插在之前，语义就不同了
  g()
}
```

# 控制流
- 重声明与重赋值
```golang
f, err := os.Open(name)
d, err := f.Stat() // 这里err被再次声明和赋值是合法的
```
- for循环
```golang
// 因为没有逗号运算符，for语句中对多变量赋值需要这样写
for i, j := 0, len(a) - 1; i < j; i, j = i + 1, j - 1 {
  a[i], a[j] = a[j], a[i]
}
```

- switch
```golang
f := Fun()
switch t := f.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
}
```
# 函数
- 返回值可以有多个，可以在函数声明中有名字
```golang
func (file *File) Write(b []byte) (n int, err error) {
  n, err = file.Write(b)
  return 
}
```
- defer类似于C++的RAII，多用于资源必须释放的场合，例如释放互斥锁和关闭文件

# 数据
- new只分配一块零值内存并返回其地址而不做初始化，零值对象一样可以直接用
```golang
type SyncedBuffer struct {
  lock sync.Mutex
  buf bytes.Buffer
}
p := new(SyncedBuffer) // p.lock和p.buf均可以直接使用
```
- 返回局部变量的地址是完全ok的。Go会在编译时做逃逸分析，自动将其在堆上分配
```golang
func NewFile(fd int, name stirng) *File {
  return &File{fd: fd, name: name} // 以键值对方式构造可以以任意顺序，未给出的field为零值
}
```
- make只用于创建map,slice和channel并返回类型T的引用
- Array
```golang
arraya := [3]int{1,2,3}
arrayb := arraya // Array是值类型,所以会复制所有元素，a和b是不同的数组
var arrayc [4]int
arrayc = arrayb // error：[3]int和[4]int是不同的类型
func(arrayb) // 数组参数b会被复制到函数中
```
- Slice
```golang
// slice是引用类型，传递参数
func Append(slice, data[]byte) []byte {
  l := len(slice)
  if l + len(data) > cap(slice) { // 重新分配
    // 为了后面的增长， 需分配两份。
    newSlice := make([]byte, (l+len(data))*2)
    // copy 函数是预声明的， 且可用于任何切片类型。
    copy(newSlice, slice)
    slice = newSlice
  } 
  slice = slice[0:l+len(data)]
  for i, c := range data {
    slice[l+i] = c
  } 
  return slice
}
```
- Map 
```golang
// Map是引用类型
ma := make(map[string]int)
mb := ma // 指向同一个Map
// Map的key必须是定义了相等关系的类型
mc := make(map[[]int]int) // error: slice没有定义相等关系，不能作为key类型
// 判断映射key是否存在并取对应值的习惯用法
if val, ok := ma[key]; ok {
  return val
}
// delete不需要保证key存在
delete(ma, "nonexist")
```
- 打印
- append
```golang
// append的签名大体如此，但实际上golang并不支持泛型，所以其实现需要编译器支持
func append(slice []T, ele ...T) []T
// slice追加slice
append(sx, sy...)
```
# 初始化
- 常量
```golang
const a = 1 << 2 // ok
const b = math.Sin(math.Pi/4) // error: 常量的初始值必须在编译时可求值
var home = os.Getenv("HOME") // ok: 变量的初始值在运行时才被计算

type ByteSize float64
const (
  _ = iota // 忽略不需要的第一个值
  KB ByteSize = 1 << (10 * iota) // 即 1 << (10 * 1)
  MB                             // 即 1 << (10 * 2)，依此类推
  GB
  TB
  PB
  EB
  ZB
  YB
)
```
- init函数
```golang
var WhatIsThe = AnswerToLife()

func AnswerToLife() int {
    return 42
}

// init的执行时机在导入pkg初始化完成以及当前pkg的所有变量完成初始化求值后才执行
// 被导入的pkg中如果有init()也会在导入时被执行（从而可能带来意想不到的结果）
func init() {
    WhatIsThe = 0
}

func main() {
    if WhatIsThe == 0 {
        fmt.Println("It's all a lie.") // 将会输出
    }
}
```
# 方法
- （除了指针或接口的）已命名类型都可以定义方法
- 指针方法可以修改接收者，而值调用会以复制的方式传递接收者
- 方法以指针为接收者只能通过指针调用，以值为接收者可通过指针和值调用
- 如果是以指针为接收者，而变量b是可寻址时，b.Write会被自动改写为(&b).Write
# 接口和类型
```golang
// type switch
type Stringer interface {
  String() string
}
func toString(value interface{}) string {
  switch s := value.(type) {
  case string:
    return s
  case Stringer:
    return s.String()
  default:
    return ""
  }
}

// type assertion
if s, ok := value.(string); ok {
}
```
# 空白标识符
```golang
// 仅为了pprof包的init函数而导入，不用到包的内容
import _ "net/http/pprof" 

// 为了确保类型RawMessage实现了Marshaler
// 将nil转换为*RawMessage并赋给Marshaler，以此要求RawMessage实现Marshaler
// 如果没有实现则会编译报错
var _ json.Marshaler = (*RawMessage)(nil)
```
# 内嵌
```golang
// 接口内嵌

```

