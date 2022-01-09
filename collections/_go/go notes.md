---
title: Notes for Learning Go
category: Go
date: 2022-1-8
---



### 运算

```go
// %取余，与被取数符号相同；a/b=trunc(a÷b)：与C相同，与Python不同
```



### 命令行参数

`os.Args`：数组切片。 例如`os.Args[1:]`表示所有参数。

`flag`:

```go
import "flag"
var n = flag.Bool("n", false, "document")
func main() {
    flag.Parse()
    *n  // bool
    flag.Args()  // 所有参数的值和剩下的
}
```



### 声明

```go
var const type func 四个关键字

i, j = j, i  // 交换值
i, j := 0, 1  // 只能用于函数内，不能用于包变量
// f, err := os.Open(name)  若f已被声明也可，但不能全都被声明，否则用=
var s, t = "s", 1
var s, t string  // 利用默认零值，数组和结构体为各个元素赋零值
var s string = ""  // 常用于隐式类型转换

constant (
	a = 1
    b = 2
)

new() new函数创建匿名变量，返回其地址，一般每次返回的是新变量
```



### 字符串(不可变)

字符串可以直接相加，或+=

将字符串转为`[]byte`切片时会拷贝字符串数据的副本

字符串可以包括ascii 0

len返回字节数，切片也基于字节

```go
strings.Join(SliceOfString, delim)  // Python: delim.join(SliceOfString)
strings.Split(String, delimiter)    // Python: String.split(delimiter)

```

```go
func (v Type) String() string { return fmt.Sprintf("sth", v) }  // Python: __str__
```





### 文件

```go
os.Open(filename)  // 返回 *os.File, err
*os.File 有 Read() 和 Write() 方法

// 流输入
import "bufio"
input := bufio.NewScanner(os.Stdin)  // 参数为 *os.File
for input.Scan() {  // 读入整行时返回true，不再有输入返回false
    do sth with input.Text()
}

// 整个读入
import "io/ioutil"
data, err := ioutil.ReadFile(filename)  // 得到字节序列
ioutil.WriteFile(filename, bytes, filemode)
ioutil.ReadAll(io.Reader)  // 比如http.Get()返回的resp.Body，读完resp.Body.Close()
```



### 随机数

```go
import "math/rand"
rand.Seed(time.Now().UTC().UnixNano())  // 随机种子
rand.Float64()  // [0.0,1.0)
```





### race condition

在go routine中会遇到读写同一个变量

```go
import "sync"
var mu sync.Mutex
mu.Lock(); read or write; mu.Unlock()
```



### 指针

go语言可以返回函数中局部变量的指针，每次调用会返回不同的指针

*指针 是变量的别名。



### 类型断言

```go
v, ok = x.(T)  // 无ok断言失败时panic异常
```







### `map`

传参时传入copy，指向统一地址的另一指针，修改会反映到原变量

```go
v, ok = m[key]  // ok为bool；若无ok，查找失败返回零值
```



### `channel`

```go
v, ok = <-ch  // ok为bool；若无ok，失败则返回零值(阻塞不算失败)
```





### `range` 

```go
for index, content := range array[:] {  // Python: for index, content in enumerate(array):
}  // for循环没有括号

counts := make(map[string]int)  // 键为string，值为int
for str, num := range counts {
}
```



### `type`

```go
type name basetype  // name大写则可被外部包调用
```





### `switch`

默认执行完`case`中的语句后退出，（用fallthrough覆盖）

```go
switch {   等价于    switch true {
}                    }
```

`switch`后可跟函数调用等，`case`后可跟条件
