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
  delete(m, "foo")
  _, prs = n["foo"]
  _, prs = n["foo"]
}

//range
for _, num := range s {
  
}
for k, v := range m {
  fmt.Println("%s -> %s", k, v)
}


```
