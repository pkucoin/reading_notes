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
