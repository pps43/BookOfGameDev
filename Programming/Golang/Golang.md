# Go

# Why Use Go
- 编译快（语法简单）
- 启动快，占用内存小（无运行时）
- 部署快（docker/k8s支持好）
- 容易编写高并发（goroutine等）、网络服务（官方网络通信库、序列化库、加解密等）
- 内置工具丰富(CGO/Unit Test/Benchmark/pprof/template/...)
- 强类型，但支持duck-typing这种动态语言的能力


# Type
- 值类型：`built-in`, `array`, `struct`
- 引用类型： `slice, map, channel, interface, function`
- 无`enum`, 通过 `const+iota` 模拟
- 无`class`，只有值类型`struct`
    - 无继承，通过`type embedding`模拟
    - 无多态，通过将子类型真实对象传入父类型保存下来，显式调用真实对象的函数
- 鸭子类型，但必须结合`interface`使用
- 无法显式实现某个`interface`
    - 解决方案（如果没有实现会报错）。`var _ IMyInterface = (*MyType)(nil)`
- `interface{}`

[运行时判断Type的几种方法](https://stackoverflow.com/questions/20170275/how-to-find-the-type-of-an-object-in-go)
- `fmt.Sprintf(%T, a)`
- `cat, ok := animal.(Cat)` 这种用法非常常见，注意有时判断ok还不够
- `swtich t:= animal.(type)`
- `reflect.TypeOf(a)`
- [How to check variable type at runtime in Go language](https://stackoverflow.com/questions/6996704/how-to-check-variable-type-at-runtime-in-go-language) 
- [X does not implement Y (... method has a pointer receiver)](https://stackoverflow.com/questions/40823315/x-does-not-implement-y-method-has-a-pointer-receiver)

## slice

- 引用类型，可以为`nil`
- 占用3个指针大小. 指向底层数组容器、长度、容量
- 建议初始化时长度等于容量
- 自动动态扩容（实际存储的array会发生变化，尤其注意）

```go
var s []int // a nil slice
s := make([]SomeType, 0, 3) // empty slice with capacity 3
s := make([]SomeType, 3) // slice filled with 3 nils (do NOT use this)

s := []int{1, 2, 3} // len=cap=3
s = append(s, 4) // len=4, expanded cap=6

s2 := s[2:4] // len=2 cap=4
s2 = append(s2, 44, 55, 66)  // len=5, cap=8 (a new underlying array)
s2 = s2[:0] //len=0, cap=8. Just clear, no GC
s2 = nil // len=cap=0. Trigger GC

for idx, v := range s {
    ... // v is a copy
}

s4 := [][]int {{1}, {1, 2}} // high dimension
```

## map

- 基于HashMap，但性能一般。
- 非线程安全。[如何并发读写，stackoverflow](https://stackoverflow.com/questions/36167200/how-safe-are-golang-maps-for-concurrent-read-write-operations)
- `for range` 遍历顺序随机（常见于CPU负载高时）。因此业务逻辑一定不能依赖遍历顺序。[官方解释有意设计成随机](https://go.dev/blog/maps)。

```go
m := map[string]int{"three": 3, "four": 4}
m["one"] = 1 // add or modify
delete(m, "three") // remove (but does NOT free any memory)
m = nil // free memory

v, ok := m["two"] // read key safely
if ok {
  // ...
}

for k, v := range m { // random order!
  // v is a copy
}
```

## struct
- 无法显式声明实现某个interface。
- 无继承，通过组合实现封装、代码复用。为此还提供两个语法糖：`anonymous field`和`promoted field`。
- 没有成员函数，而使用function with receiver 来模拟成员函数。

```go
type Animal struct {
    name string
}
type Cat struct {
    Animal //anonymous field
    age int
}

// c as reference. More common.
func (c *Cat) Meow() {
    fmt.Println(c.Animal.name) 
    fmt.Println(c.name) // promoted field
}
// c as copy
func (c Cat) Meow() {
}
```

## interface
- go的interface本身也是一个对象，这个对象包含两个指针，分别指向实际对象、实际对象类型的方法表。这个设定的好处是实现动态语言般的`duck typing`，坏处是给判空带来额外的复杂性。
```go
type iface struct {
  tab *itab
  data unsafe.Pointer
}
```

- `interface{}`可以接收任意类型。

更多资料
- https://halfrost.com/go_interface/
- https://zhaolion.com/post/golang/upgrade/interface/
- https://github.com/teh-cmc/go-internals/blob/master/chapter2_interfaces/README.md

## channel

- No-buffer channel: 收发端需要同时准备好，否则阻塞。
- Buffered (Queue) channel. 当channel空时接收端阻塞，当channel满时发送端阻塞。
- 只读型 `<-chan`, 只写型 `chan<-`
- 可用于同步
- 需要手动关闭`close(c)`, 而不是通过发送`nil`


```go
nobufferChannel := make(chan int) // `int` can be other types, including pointer
bufferedChannel := make(chan int, 10)

bufferedChannel <- 9 // send one int to channel

v, ok := <- bufferedChannel // receive one int from channel
if !ok {
  // channel is closed
}

for msg := range bufferedChannel {
  // auto break when channel close. "range" here is a sugar
}

// only sender should close. Sending to a closed channel will panic
close(bufferedChannel)

for { // inf loop
  select { // select randomly when any channel has content
  case msg1 := <- nobufferChannel:
    // do something
  case msg2 := <- bufferChannel:
    // do something
  }
}

// worker pool pattern
func worker(jobs <-chan int, results chan<- int) {
  for n:= range jobs {
    results <- n*n
  }
}
func main(){
  jobs := make(chan int, 10)
  results := make(chan int, 10)
  go worker(jobs, results)
  go worker(jobs, results)
  go worker(jobs, results)
  for i:= 0; i < 10; i++ {
    jobs <- i
  }
  close(jobs)
  for j:=0; j < 10; j++ {
    fmt.Println(<-results)
  }
}
```


## Reflection
- https://halfrost.com/go_reflection/


# Variable
- 大小学决定package外的可见性
- 自带gc，使用静态逃逸分析决定变量在堆还是在栈上，无法人工控制。
- `new(T)`只分配内存并清零。返回一个`*T`对象（指针）。简化写法`&T{}`。
- `make`只用于`slice, map, channel`

```go
s := make([]int, 100) // Allocate memory of 100 ints, initialize to 0
m := make(map[string]int) // Allocate memory for an empty <string, int> map
c := make(chan int)
m2 := map[string]int {} // Same effect as m. Called composite literals
```

# Control flow
- switch-case-fallthrough
- select-case-channel
- panic-recover
- defer 离开当前scope时执行，多个defer按照LIFO（栈）顺序执行

## goroutine
- goroutine类似协程，是逻辑概念，由go调度。
- goroutine和线程没有从属关系，不同goroutine可能分布在不同线程里。
    - 需要注意多线程race-condition：通过`go build -race`检测。
    - 必要时使用`atomic`, `sync.Mutex`等操作共享数据。
    - 常常使用`channel`起到同步和共享数据的作用。
- 新建一个goroutine很简单，在普通函数调用前加`go`即可。

## concurrency
专门讨论基于goroutine的并发和同步。
- 简单情况下，使用channel即可同步。
- 使用`sync.WaitGroup`。
- 使用`Context`管理复杂情况。多层级或一组关联的goroutine（尤见于web server）。 https://pkg.go.dev/context


```go
// Use channel to sync
func SyncByChannel() {
  num := 3
  channels := make([]chan int, num)
  for i:= 0; i<num; i++ {
    channels[i] = make(chan int)
    go DoJob(channels[i])
  }
  for i, ch := channels {
    <-ch
    // wait until goroutine i finishes
  }
  // all should finishes
}

func DoJob(ch chan int) {
  // when finish working, send a int as a signal
  ch <- 1
}

//Use WaitGroup to sync
func SyncByWaitGroup() {
  num := 3
  var wg sync.WaitGroup
  wg.Add(num)

  for i:= 0; i<num; i++ {
    go DoJob2(&wg)
  }

  wg.Wait()
  // all finishes
}

func DoJob2(wg *sync.WaitGroup) {
  // working
  wg.Done() // counter minus 1
}

```

# Tool
- go build
- go test
- go run race MyTest.go 检查goroutine是否有竞态
- go run -gcflags "-m -l" .MyTest.go 检查是否有逃逸
- go tool pprof
- go generate
- text template

## Compiler Tag
- go没有宏，但可以通过`+build [tag1, tag2, ...]`条件编译。支持多个tag（&&）。支持内置tag如linux, arm64，支持自定义tag（通过`go build -tags ...`指定）

- `//go:inline`建议内联，`//go:noinline`强制不内联
- `go build -ldflags "-s -w"`可以让编译出的包更小（因为剥离了符号表），也更加安全。