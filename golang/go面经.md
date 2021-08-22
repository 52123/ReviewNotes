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

