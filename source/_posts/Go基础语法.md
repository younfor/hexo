---
title: Go基础语法.md
date: 2018-06-09 21:11:17
categories: Go语言
---
 # 编码风格
## gofmt
一般会自动规范代码风格
 ##注释&驼峰命名
/* 
    需要注释的内容
*/
## 包名
```
import "bytes"
```
# 变量
## 常量
```
type ByteSize float64
const (
	_           = iota
	KB ByteSize = 1 << (2 * iota)
	MB
	GB
)
// 主函数
func main() {
	fmt.Println("Hello World!")
	fmt.Println("size %f", GB)
	// size %f 64
}
```
## 声明
```
var (
    ErrInternal = errors.New("error1")
    ErrInternal2 = errors.New("error2")
)
```
## 初始化
Go语言提供了New和make
### new
```
// 0值初始化
type SyncedBuffer struct {
    lock sync.Mutex
    buffer bytes.Buffer
}
p := new(SyncedBuffer)     // type *SyncedBuffer
var v SyncedBuffer            // type SyncedBuffer

// 如果是构造函数
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0} // 如果不按顺序，就需要加名字
    return &f
}
// 复合字面可以初始化多种结构
a := [...]string {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```
### make
`make一般用于初始化切片，映射，信道
```
make([]int, 10, 100)
// 一般用法, 返回的不是指针
v := make([]int, 100) 
```
# 分支循环
## if
```
if i < f() {
    g()
}

if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```
## for
```
for init; condition; post {}
for condition {}
for {}
// 举例
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
// 遍历复合结构
for key, value := range oldMap {
    newMap[key] = value
}
// 只遍历第一个
for key := range oldMap {
}
// 只遍历第二个
for _, value := range array {
}
// 遍历字符串
for pos, char := range "日本 \ x80 语" {
    fmt.Printf()
}
// 反转数字
for i, j := 0, len(a) - 1; i < j; i,j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```
## switch
```
switch {
case '0' <= c && c <= '9' :
    return c - '0'
case 'a' <= c && c <= 'f' :
    return c - 'a' + 10
}
// 处理相同条件
switch c {
case 'a', 'b', 'c':
    return true
}
// 判断类型
var t interface{}
t = getType()
switch t := t.(type) {
default :
    fmt.Printf("%T", t)
case bool :
    fmt.Printf("boolean %t\n", t)
case int:
    fmt.Printf("integer %d\n", t)
}
```
# 函数
## 多值返回
```
func (file *File) Write(b []byte) (n int, err error)
```
## defer
无论何种路径都能返回，如果定义多个按定义顺序相反顺序执行
# 切片(数组)
```
// 切片是按值传递，但是底层可能是同一份数组
// 二维切片
type Transform [3][3]float64
type LinesOfText [][]byte
test := LinesOfTest{
    []byte("abc"),
    []byte("abc"),
    []byte("abc"),
}
```
## append
```
// 增加元素
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
// 增加切片
y := []int{1,2,3}
x = append(x, y...)
fmt.Println(x)
```
# Map映射
```
attend := map[string]bool{
    "Ann" : true,
    "Joe" : true,
    ...
}
// 判断是否存在
var seconds int
var ok bool
seconds, ok = timeZone[tz]
if _, ok := timeZone[tz]; ok {
    doSomething()
}
// 删除映射
delete(timeZone, "PDT")
```
初始化方式：
File{fd, name, nil, 0}
File{fd: fd, name: name}
make([]int, 100)
# 函数
## init
在变量初始化之后，导入包初始化之后，就会初始化
`以指针或值为接收者的区别在于：值方法可通过指针和值调用， 而指针方法只能通过指针来调用。
# 接口
## 断言
```
str, ok := value.(string)
if ok {
    fmt.Printf("")
} else {
}
```
## 内嵌
```
// 接口
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
type ReaderWriter interface {
    Reader
    Writer
}
// 结构体
type ReaderWriter struct {
    *Reader
    *Writer
}
```
# 并发
## chan
```
cj := make(chan int)                // 无缓冲信道
cj := make(chan int, 0)            // 无缓冲信道
cj := make(chan *os.File, 100) // 指向文件指针的带缓冲信道
```
## 例子
```
c := make(chan int)
go func() {
    list.Sort()
    c <- 1
}()
doSomethingForAWhile()
<- c

// 考虑带缓冲的任务
func Serve(queue chan *Request) {
       for req := range queue {
            req := req
           sem <- 1
            go func() {
                process(req)
                <-sem
            }()
        }
}
```
# panic&recover
