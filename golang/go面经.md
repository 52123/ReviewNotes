# 1. 数据结构



## 1.1 nil切片和空切片的区别

- nil切片和空切片指向的地址不一样
  - nil空切片引用数组指针地址为0
  - 空切片的引用数组指针地址是有的，且固定为一个值



```go
// 切片的数据结构
type SliceHeader struct {
 Data uintptr  //引用数组指针地址
 Len  int     // 切片的目前使用长度
 Cap  int     // 切片的容量
}
```



## 1.2 字符串转成byte数组，会发生内存拷贝吗

严格来说，只要是发生类型强转都会发生内存拷贝



```go
// 字符串在go的底层数据结构
type StringHeader struct {
 Data uintptr
 Len  int
}
```

```go
// 切片在go的底层数据结构
type SliceHeader struct {
 Data uintptr
 Len  int
 Cap  int
}
```



转换步骤：

- unsafe.Pointer(&a)方法得到字符串a的地址
- (*reflect.StringHeader)(unsafe.pointer(&a))把字符串a转成底层结构的形式
- (*[]byte)(unsafe.Pointer(&ssh)) 把ssh底层结构体转成byte的切片的指针
- 通过*转为指针指向的实际内容

```go
func main() {
 a :="aaa"
 ssh := *(*reflect.StringHeader)(unsafe.Pointer(&a))
 b := *(*[]byte)(unsafe.Pointer(&ssh))  
 fmt.Printf("%v",b)
}
```



## 1.3 翻转含有中文、数字、英文字母的字符串

```go
func main() {
 src := "你好abc啊哈哈"
 dst := reverse([]rune(src))
 fmt.Printf("%v\n", string(dst))
}

func reverse(s []rune) []rune {
 for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
  s[i], s[j] = s[j], s[i]
 }
 return s
}
```





## 1.4 理解go中的rune数据类型

```go
// rune is an alias for int32 and is equivalent to
// int32 in all ways. It is
// used, by convention, to distinguish character
// values from integer values.

// int32的别名，几乎在所有方面等同于int32
// 它用来区分字符值和整数值

type rune = int32
```



由于编码的原因，我们如果按照一个个字节的方式去处理字符串，会导致处理规则没办法知道是按几个字节处理

所以在go语言中引进了rune的概念。在我们对字符串进去处理的时候只需要将字符串通过range去遍历，会按照rune为单位自动去处理，极其便利。

```go
var str = "go算法"
	for k, v := range str {
		fmt.Printf("v type: %T\n", v)
		fmt.Println(v,k)
	}
```



- 通过上面的代码我们已经很清楚的知道rune类型实质其实就是int32，他是go语言内在处理字符串及其便捷的字符单位。它会自动按照字符独立的单位去处理方便我们在遍历过程中按照我们想要的方式去遍历。
- 另外一个使用场景就是：我们在处理字符串的时候可以通过map[rune]int类型方便的判断字符串是否存在，其中 rune表示字符的UTF-8的编码值，int表示字符在字符串中的位置(按照字节的位置)
- 比起byte，科表示更多的字符



## 1.5 拷贝大切片一定比小切片代价大吗

并不是，所有切片的大小相同；**三个字段**（一个 uintptr，两个int）。切片中的第一个字是指向切片底层数组的指针，这是切片的存储空间，第二个字段是切片的长度，第三个字段是容量。将一个 slice 变量分配给另一个变量只会复制三个机器字。所以 **拷贝大切片跟小切片的代价应该是一样的**。





## 1.6 map不初始化使用会怎样

可以对未初始化的map进行取值，但取出来的东西是空

不能对未初始化的map进行赋值，这样将会抛出一个异常：panic: assignment to entry in nil map



## 1.7 map不初始化长度和初始化长度的区别

空map和nil map结果是一样的，都为map[]





## 1.8 map的参数初始值，扩容机制

















- new和make的区别
- GC内存管理机制
- 函数传值传啥会更好
  - 传指针，逃逸
- 线程有几种模型，Goroutine的原理，实现，优缺势
- Goroutine什么时候发生阻塞
- PMG模型中Goroutine有哪几种状态？线程呢
- 每个线程/协程占用多少内存
- 如果Goroutine一直占用资源怎么办，PMG模型如何解决这个问题
- 如果若干线程中一个线程OOM，会发生什么？如果是Goroutine呢？项目中出现过OOM吗？怎么解决的
- defer可以捕获到其Goroutine的子Goroutine的panic吗
- Gin的错误处理使用过吗？Gin中自定义校验规则知道是怎么做的吗，自定义校验返回值呢
- 反射了解过吗?反射的原理
- golang的锁机制。Mutex锁有哪几种模式。Mutex底层如何实现。Mutex业务场景
- channel业务场景。需要注意的地方





- 如何做性能优化
  - 开启火焰图
  - 调用链
  - https://zhuanlan.zhihu.com/p/27800985

- 问你channel的实现
- gc 内存回收
- mpg
- context



\2. 代码效率分析，考察局部性原理
\3. 多核CPU场景下，cache如何保持一致、不冲突？
\4. uint类型溢出
\5. 介绍rune类型
\6. 编程题：3个函数分别打印cat、dog、fish，要求每个函数都要起一个goroutine，按照cat、dog、fish顺序打印在屏幕上100次。
\7. 介绍一下channel，无缓冲和有缓冲区别
\8. 是否了解channel底层实现，比如实现channel的数据结构是什么？
\9. channel是否线程安全？
\10. Mutex是悲观锁还是乐观锁？悲观锁、乐观锁是什么？
\11. Mutex几种模式？
\12. Mutex可以做自旋锁吗？
\13. 介绍一下RWMutex
\14. 项目中用过的锁？
\15. 介绍一下线程安全的共享内存方式
\16. 介绍一下goroutine
\17. goroutine自旋占用cpu如何解决（go调用、gmp）
\18. 介绍linux系统信号
\19. goroutine抢占时机（gc 栈扫描）
\20. Gc触发时机
\21. 是否了解其他gc机制
\22. Go内存管理方式
\23. Channel分配在栈上还是堆上？哪些对象分配在堆上，哪些对象分配在栈上？
\24. 介绍一下大对象小对象，为什么小对象多了会造成gc压力？
\25. 项目中遇到的oom情况？
\26. 项目中使用go遇到的坑？
\27. 工作遇到的难题、有挑战的事情，如何解决？
\28. 如何指定指令执行顺序？
