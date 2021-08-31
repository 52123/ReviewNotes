## 1. ORM框架忽略0值

例如，gorm会忽略结构体中的0值



解决方案：

- gostyle里推荐用 sql.NullInt64 这个结构体， 有一个是否有效的bool值
- 直接写SQL


##
1. 
res := make([]int, 0, 4) 
创建res的容量为0


for i, _ := range xx {
}

听说这个可能有问题，会乱序，验证
for _, x := range xx {
}

