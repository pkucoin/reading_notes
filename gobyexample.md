```golang
package main // main包main函数为入口

import fmt

func main() {
  // 变量定义
  var a = true // 推断类型
  var b, c int = 1, 2 // 指定类型
  var d int // 无初值则初始化为0
  e := "shorthand for declaring and initializing" // 省略var使用:=在同一行声明并初始化
  
  // 常量
  const n = 50000000 // n是常量，但还没有类型
  fmt.Println(int64(n)) // 类型为int64
  fmt.Println(math.Sin(n)) // 类型为float64
  
  // 控制流
  i := 1
  for i <= 3 { // while
    i = i + 1
  }
  for  j := 1; j <= 3; j++ { // 经典三段式
  }
  for { // 无条件循环
    if p := 1; p % 2 == 0 {
      fmt.Println(p)
    } else if p > 0 {
      continue
    } else {
      break;
    }
  }
  // 类型switch
  whatAmI := func(i interface{}) {
      switch t := i.(type) {
      case bool:
          fmt.Println("I'm a bool")
      case int:
          fmt.Println("I'm an int")
      default:
          fmt.Printf("Don't know type %T\n", t)
      }
  }
  
  // 数组
  var twoD [2][3]int // 定长，0值初始化，二维数组
  fmt.Println("len of twoD: ", len(twoD)) // 2
  fmt.Println("len of twoD[1]: ", len(twoD[1])) // 3
  
  // 切片
  s := make([]string, 3)
  t := []string{"g", "h", "i"}
  s = append(s, "a", "b")
  copy(c, s[2:5]) // c: {"", "a", "b"}
  
  // Map
  m := make(map[string]int)
  m["foo"] = 1
  _, prs := m["foo"]
  n := map[string]int{"foo": 1, "bar": 2}
  delete(n, "foo")
  _, prs = n["foo"]

  // range
  for _, num := range s { // 不需要index
  }
  for k, v := range m {
    fmt.Println("%s -> %s", k, v)
  }
  for k := range m { // 只需要key
    fmt.Println("key:", k)
  }
  for i, c := range "go" { // 遍历string得到每一个字符的Unicode code point
    fmt.Println(i, c)
  }
  
  // 函数
  func f(a, b, c int) (int, int) { // 多参数多返回值
    return a + b, c
  }
  
  // 可变参数
  func sum(nums ...int) int { 
    total := 0
    for _, num := range nums {
      total += num
    }
    return total
  }
  sum(1, 2, 3)
  nums := []int{1, 2, 3}
  sum(num...)
  
  // 闭包
  func f() func() int { // 返回值是一个void参数int返回值的函数
    i := 0
    return func() int {
      i++
      return i
    }
  }
  nextInt := f()
  nextInt() // 1
  nextInt() // 2
  nextInt2 := f()
  nextInt() // 1
  
  // 值和引用
  func zeroval(ival int) {
    ival = 0
  }
  func zeroptr(iptr *int) {
    *iptr = 0
  }
  i := 1
  zeroval(i)
  zeroptr(&i)
  fmt.Println(&i)
  
  // 结构体
  
}
```
