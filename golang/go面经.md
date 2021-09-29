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





## 1.9 结构体能不能比较

### 1.9.1 可比较的数据类型

```go
Integer
Floating-point
String
bool
Complex
Pointer
Channel
Interface
Array
```

### 1.9.2 不可比较的数据类型

Slice、Map、Function

为什么不可以比较



### 1.9.3 同一个struct的两个实例能不能比较

使用== ：当结构体内所有成员变量都可以比较的时候，那么就可以比较

使用DeepEqual:  对含有不可直接比较的成员变量时，可以用DeepEqual进行比较



### 1.9.4 两个不同的struct的实例能不能比较

可以通过强制转换来比较

如果成员变量中含有不可比较成员变量，即使可以强制转换，也不可以比较



## 2.1 new和make的区别

https://golang.org/doc/effective_go#allocation_new

Go语言中 new 和 make 是两个内置函数，**主要用来创建并分配类型的内存**

Go语言中的 new 和 make 主要区别如下：

- make 只能用来分配及初始化类型为 slice、map、chan 的数据。new 可以分配任意类型的数据；
- new 分配返回的是指针，即类型 *Type。make 返回引用，即 Type；
- new 分配的空间被清零。make 分配空间后，会进行初始化；



### 2.1.1 new

```go
// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```

new 函数只接受一个参数，这个参数是一个类型，并且返回一个指向该类型内存地址的指针。同时 new 函数会把分配的内存置为零，也就是类型的零值。



if a composite literal contains no fields at all, it creates a zero value for the type. The expressions `new(File)` and `&File{}` are equivalent



### 2.1.2 make

make 也是用于内存分配的，但是和 new 不同，它只用于 chan、map 以及 slice 的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
// Slice: The size specifies the length. The capacity of the slice is
// equal to its length. A second integer argument may be provided to
// specify a different capacity; it must be no smaller than the
// length, so make([]int, 0, 10) allocates a slice of length 0 and
// capacity 10.
// Map: An empty map is allocated with enough space to hold the
// specified number of elements. The size may be omitted, in which case
// a small starting size is allocated.
// Channel: The channel's buffer is initialized with the specified
// buffer capacity. If zero, or the size is omitted, the channel is
// unbuffered.
func make(t Type, size ...IntegerType) Type
```

The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use

### 2.1.3 make实现原理



在编译期的类型检查阶段，Go语言其实就将代表 make 关键字的 OMAKE 节点根据参数类型的不同转换成了 OMAKESLICE、OMAKEMAP 和 OMAKECHAN 三种不同类型的节点，这些节点最终也会调用不同的运行时函数来初始化数据结构。

![img](E:\study\ReviewNotes\docs\make-typecheck)

### 2.1.4 new实现原理

内置函数 new 会在编译期的 SSA 代码生成阶段经过 callnew 函数的处理，如果请求创建的类型大小是 0，那么就会返回一个表示空指针的 zerobase 变量，在遇到其他情况时会将关键字转换成 newobject：

```go
func callnew(t *types.Type) *Node {
    if t.NotInHeap() {
        yyerror("%v is go:notinheap; heap allocation disallowed", t)
    }
    dowidth(t)
    if t.Size() == 0 {
        z := newname(Runtimepkg.Lookup("zerobase"))
        z.SetClass(PEXTERN)
        z.Type = t
        return typecheck(nod(OADDR, z, nil), ctxExpr)
    }
    fn := syslook("newobject")
    fn = substArgTypes(fn, t)
    v := mkcall1(fn, types.NewPtr(t), nil, typename(t))
    v.SetNonNil(true)
    return v
}
```



哪怕当前变量是使用 var 进行初始化，在这一阶段也可能会被转换成 newobject 的函数调用并在堆上申请内存

```go
func walkstmt(n *Node) *Node {
    switch n.Op {
    case ODCL:
        v := n.Left
        if v.Class() == PAUTOHEAP {
            if prealloc[v] == nil {
                prealloc[v] = callnew(v.Type)
            }
            nn := nod(OAS, v.Name.Param.Heapaddr, prealloc[v])
            nn.SetColas(true)
            nn = typecheck(nn, ctxStmt)
            return walkstmt(nn)
        }
    case ONEW:
        if n.Esc == EscNone {
            r := temp(n.Type.Elem())
            r = nod(OAS, r, nil)
            r = typecheck(r, ctxStmt)
            init.Append(r)
            r = nod(OADDR, r.Left, nil)
            r = typecheck(r, ctxExpr)
            n = r
        } else {
            n = callnew(n.Type.Elem())
        }
    }
}
```

当然这也不是绝对的，如果当前声明的变量或者参数不需要在当前作用域外生存，那么其实就不会被初始化在堆上，而是会初始化在当前函数的栈中并随着函数调用的结束而被销毁



newobject 函数的工作就是获取传入类型的大小并调用 mallocgc 在堆上申请一片大小合适的内存空间并返回指向这片内存空间的指针：

```go
func newobject(typ *_type) unsafe.Pointer {
  return mallocgc(typ.size, typ, true)
}
```







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
