---
title: Go高级编程.md
date: 2018-06-09 11:11:17
categories: Go语言
tags: [Go]
---

https://books.studygolang.com/advanced-go-programming-book/ch2-cgo/ch2-04-memory.html
# 1.数组
## 1.1数组定义方式
```
var a [3]int                    // 定义一个长度为3的int类型数组, 元素全部为0

var b = [...]int{1, 2, 3}       // 定义一个长度为3的int类型数组, 元素为 1, 2, 3

var c = [...]int{2: 3, 1: 2}    // 定义一个长度为3的int类型数组, 元素为 0, 2, 3

var d = [...]int{1, 2, 4: 5, 6} // 定义一个长度为6的int类型数组, 元素为 1, 2, 0, 0, 5, 6

```
## 1.2数组指针
```
var a = [...]int{1, 2, 3} // a 是一个数组

var b = &a                // b 是指向数组的指针


fmt.Println(a[0], a[1])   // 打印数组的前2个元素

fmt.Println(b[0], b[1])   // 通过数组指针访问数组元素的方式和数组类似


for i, v := range b {     // 通过数组指针迭代数组的元素

    fmt.Println(i, v)

}
```
## 1.3数组遍历
```
    for i := range a {

        fmt.Printf("b[%d]: %d\n", i, b[i])

    }

    for i, v := range b {

        fmt.Printf("b[%d]: %d\n", i, v)

    }

    for i := 0; i < len(c); i++ {

        fmt.Printf("b[%d]: %d\n", i, b[i])

    }

```
## 1.4函数数组、管道数组
```
// 图像解码器数组

var decoder1 [2]func(io.Reader) (image.Image, error)

var decoder2 = [...]func(io.Reader) (image.Image, error){

    png.Decode,

    jpeg.Decode,

}


// 接口数组

var unknown1 [2]interface{}

var unknown2 = [...]interface{}{123, "你好"}


// 管道数组

var chanList = [2]chan int{}

```
## 1.5空数组
长度为0的数组在内存中并不占用空间。空数组虽然很少直接使用，但是可以用于强调某种特有类型的操作时避免分配额外的内存空间，比如用于管道的同步操作：
```
    c1 := make(chan [0]int)

    go func() {

        fmt.Println("c1")

        c1 <- [0]int{}

    }()

    <-c1

```
在这里，我们并不关心管道中传输数据的真实类型，其中管道接收和发送操作只是用于消息的同步。对于这种场景，我们用空数组来作为管道类型可以减少管道元素赋值时的开销。当然一般更倾向于用无类型的匿名结构体代替：
```
    c2 := make(chan struct{})

    go func() {

        fmt.Println("c2")

        c2 <- struct{}{} // struct{}部分是类型, {}表示对应的结构体值

    }()

    <-c2
```
# 2.字符串
## 2.1类似切片
字符串虽然不是切片，但是支持切片操作，不同位置的切片底层也访问的同一块内存数据（因为字符串是只读的，相同的字符串面值常量通常是对应同一个字符串常量）：
```
s := "hello, world"

hello := s[:5]

world := s[7:]


s1 := "hello, world"[:5]

s2 := "hello, world"[7:]

```
和数组一样，内置的len和cap函数返回相同的结果，都对应字符串的长度。也可以通过reflect.StringHeader结构访问字符串的长度（这里只是为了演示字符串的结构，并不是推荐的做法）：
```
fmt.Println("len(s):", (*reflect.StringHeader)(unsafe.Pointer(&s)).Len)   // 12

fmt.Println("len(s1):", (*reflect.StringHeader)(unsafe.Pointer(&s1)).Len) // 5

fmt.Println("len(s2):", (*reflect.StringHeader)(unsafe.Pointer(&s2)).Len) // 5
```
# 3.切片(slice)
```
var (

    a []int               // nil切片, 和 nil 相等, 一般用来表示一个不存在的切片

    b = []int{}           // 空切片, 和 nil 不相等, 一般用来表示一个空的集合

    c = []int{1, 2, 3}    // 有3个元素的切片, len和cap都为3

    d = c[:2]             // 有2个元素的切片, len为2, cap为3

    e = c[0:2:cap(c)]     // 有2个元素的切片, len为2, cap为3

    f = c[:0]             // 有0个元素的切片, len为0, cap为3

    g = make([]int, 3)    // 有3个元素的切片, len和cap都为3

    h = make([]int, 2, 3) // 有2个元素的切片, len为2, cap为3

    i = make([]int, 0, 3) // 有0个元素的切片, len为0, cap为3

)
``
## 3.1添加切片
```
var a []int

a = append(a, 1)               // 追加1个元素

a = append(a, 1, 2, 3)         // 追加多个元素, 手写解包方式

a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要解包


```
`在容量不足的情况下，append的操作会导致重新分配内存，从而导致巨大的内存分配和复制数据代价`
```
var a = []int{1,2,3}

a = append([]int{0}, a...)        // 在开头添加1个元素

a = append([]int{-3,-2,-1}, a...) // 在开头添加1个切片
```
```
var a []int

a = append(a[:i], append([]int{x}, a[i:]...)...)     // 在第i个位置插入x

a = append(a[:i], append([]int{1,2,3}, a[i:]...)...) // 在第i个位置插入切片
a = append(a, x...)       // 为x切片扩展足够的空间

copy(a[i+len(x):], a[i:]) // a[i:]向后移动len(x)个位置

copy(a[i:], x)            // 复制新添加的切片
```
## 3.2删除切片
需要重新赋值切片防止内存泄漏
```
func FindPhoneNumber(filename string) []byte {

    b, _ := ioutil.ReadFile(filename)

    b = regexp.MustCompile("[0-9]+").Find(b)

    return append([]byte{}, b...) // 这里

}

```
删除切片有时候会影响GC
```
var a []*int{ ... }

a = a[:len(a)-1]    // 本删除的最后一个元素依然被引用, 可能导致GC操作被阻碍

var a []*int{ ... }
a[len(a)-1] = nil // GC回收最后一个元素内存

a = a[:len(a)-1]  // 从切片删除最后一个元素

```
可以用copy和append组合可以避免创建中间的临时切片，同样是完成添加元素的操作：
```
a = append(a, 0)     // 切片扩展1个空间

copy(a[i+1:], a[i:]) // a[i:]向后移动1个位置

s[i] = x             // 设置新添加的元素

```
第一句append用于扩展切片的长度，为要插入的元素留出空间。第二句copy操作将要插入位置开始之后的元素向后挪动一个位置。第三句真实地将新添加的元素赋值到对应的位置。操作语句虽然冗长了一点，但是相比前面的方法，可以减少中间创建的临时切片。
用copy和append组合也可以实现在中间位置插入多个元素(也就是插入一个切片):
```
a = append(a, x...)       // 为x切片扩展足够的空间

copy(a[i+len(x):], a[i:]) // a[i:]向后移动len(x)个位置

copy(a[i:], x)            // 复制新添加的切片
```
也可以用copy完成删除开头的元素：
```
a = []int{1, 2, 3}

a = a[:copy(a, a[1:])] // 删除开头1个元素

a = a[:copy(a, a[N:])] // 删除开头N个元素

```
对于删除中间的元素，需要对剩余的元素进行一次整体挪动，同样可以用append或copy原地完成：
```
a = []int{1, 2, 3, ...}


a = append(a[:i], a[i+1:]...) // 删除中间1个元素

a = append(a[:i], a[i+N:]...) // 删除中间N个元素


a = a[:copy(a[i:], a[i+1:])]  // 删除中间1个元素

a = a[:copy(a[i:], a[i+N:])]  // 删除中间N个元素
```
# 4.并发模型
## 4.1加锁
```
var total struct {

    sync.Mutex

    value int

}
通过sync.lock()和sync.Done()
```
## 4.2原子
```
import (

    "sync"

    "sync/atomic"

)
//用法：
func worker(wg *sync.WaitGroup) {

    defer wg.Done()


    var i uint64

    for i = 0; i <= 100; i++ {

        atomic.AddUint64(&total, i)

    }

}


func main() {

    var wg sync.WaitGroup

    wg.Add(2)


    go worker(&wg)

    go worker(&wg)

    wg.Wait()

}
```
## 4.3单例
```
我们可以将通用的代码提取出来，就成了标准库中sync.Once的实现：
type Once struct {

    m    Mutex

    done uint32

}


func (o *Once) Do(f func()) {

    if atomic.LoadUint32(&o.done) == 1 {

        return

    }


    o.m.Lock()

    defer o.m.Unlock()


    if o.done == 0 {

        defer atomic.StoreUint32(&o.done, 1)

        f()

    }

}

基于sync.Once重新实现单件模式：
var (

    instance *singleton

    once     sync.Once

)


func Instance() *singleton {

    once.Do(func() {

        instance = &singleton{}

    })

    return instance

}
```
## 4.4带缓冲chan
测试了下才知道原来
<-chan int  像这样的只能接收值
chan<- int  像这样的只能发送值
```
var limit = make(chan int, 3)


func main() {

    for _, w := range work {

        go func() {

            limit <- 1

            w()

            <-limit

        }()

    }

    select{}

}
func main() {

    done := make(chan int, 10) // 带 10 个缓存


    // 开N个后台打印线程

    for i := 0; i < cap(done); i++ {

        go func(){

            fmt.Println("你好, 世界")

            done <- 1

        }()

    }


    // 等待N个后台线程完成

    for i := 0; i < cap(done); i++ {

        <-done

    }

}
//其实也可以用
sync.WaitGroup
```
## 4.5 select
```
基于select实现的管道的超时判断：
    select {

    case v := <-in:

        fmt.Println(v)

    case <-time.After(time.Second):

        return // 超时

    }

通过select的default分支实现非阻塞的管道发送或接收操作：
    select {

    case v := <-in:

        fmt.Println(v)

    default:

        // 没有数据

    }

通过select来阻止main函数退出：
func main() {

    // do some thins

    select{}

}
```
但是管道的发送操作和接收操作是一一对应的，如果要停止多个Goroutine那么可能需要创建同样数量的管道，这个代价太大了。其实我们可以通过close关闭一个管道来实现广播的效果，所有从关闭管道接收的操作均会收到一个零值和一个可选的失败标志。
```
func worker(cannel chan bool) {

    for {

        select {

        default:

            fmt.Println("hello")

            // 正常工作

        case <-cannel:

            // 退出

        }

    }

}


func main() {

    cancel := make(chan bool)


    for i := 0; i < 10; i++ {

        go worker(cancel)

    }


    time.Sleep(time.Second)

    close(cancel)

}
```
# 5.异常
异常必须放在defer func () {}()
