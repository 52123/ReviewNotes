# 一、Go架构实践



## 1.1 微服务概念



### 1.1.1  单体架构

只有一个应用，并把它打包并部署为单体式应用



问题： 复杂，逻辑多，难以实现敏捷性开发和部署。 单体应用扩展较难

应对思路： 化繁为简，分为治之   





### 1.1.2  微服务起源

SOA（面向服务）的架构模式，微服务可以想成是SOA的一种实践

- 小既是美。
- 单一职责。
- 尽可能早地创建原型。
- 可移植性比效率更重要。





### 1.1.3  微服务定义

围绕业务功能构建的，服务关注单一业务，服务间采用轻量级地通信机制，可以全自动独立部署，可以使用不同地编程语言和数据存储技术。

- 原子服务
- 独立进程
- 隔离部署
- 去中心化服务治理

缺点：基础设施的建设、复杂度高



### 1.1.4  微服务不足

there are no silver bullets

- 复杂性。分布式系统不得不使用RPC或者消息传递，来实现进程间通信。消息传递的速度，局部不可用
- 分区的数据库架构。同时更新多个业务主体的事务很普遍，考虑分布式一致性的问题
- 测试一个基于微服务架构的应用也比较复杂
- 服务模块间的依赖，应用的升级有可能会波及多个服务模块的修改
- 对运维基础设施的挑战比较大，比如日志



### 1.1.5 组件服务化

**微服务的核心**

传统实现组件的方式是通过库，库是和应用一起运行在进程中的，库的局部变化意味着整个应用的重新部署。



通过服务来实现组件，意味着将应用拆散为一系列的服务运行在不同的进程中，那么单一服务的局部变化只需要重新部署对应的服务进程

- kit：一个微服务的基础库
- service：业务代码 + kit依赖 + 第三方依赖组成的业务微服务
- rpc + message queue： 轻量级通讯



本质： 多个微服务组合完成了一个完整的用户场景





### 1.1.6 按业务组织服务

服务提供的能力和业务功能对应



### 1.1.7 去中心化

每个服务面临的业务场景不同，可以针对性的选择合适的技术解决方案

- 数据去中心化。 避免大SQL或缓存搞挂数据库，所以要隔离
- 治理去中心化。比如，通过Nginx实现代理，那么全部流量必先经过Nginx，虽然可以扩容，但是收益非常低
- 技术去中心化。不局限一门技术



### 1.1.8 基础设施自动化

- CICD： Gitlab + Gitlab Hooks + k8s
- testing：测试环境、单元测试、API自动化测试
- 在线运行时：k8s，Prometheus、ELK、Control Panle





### 1.1.9 可用性 和 兼容性涉及

Design For Failure 

- 隔离
- 超时控制
- 负载保护
- 限流
- 降级
- 重试
- 负载均衡



## 1.2 微服务设计

### 1.2.1 API Gateway

面向用户业务场景API



不用API Gateway的问题：

- 客户端到微服务直接通信，强耦合
- 需要多次请求，客户端聚合数据，工作量巨大，延迟高
- 协议不利于统一，各部门间有差异
- 统一逻辑无法收敛，比如安全认证、限流

优势：

- 轻量交互：协议精简、聚合
- 差异服务：数据裁剪一级聚合、针对终端定制化API
- 动态升级：原有系统兼容升级，更新服务而非协议
- 沟通效率提升



BFF：（业务场景聚合）可以认为是一种适配服务，将后端的微服务进行适配（包括聚合裁剪和格式适配等逻辑），向无线端设备保留友好和统一的接口

存在问题：single point of failure， 严重代码缺陷或者浏览洪峰可能引发集群宕机



移动端 -> API Gateway -> BFF -> Mircoservice

跨横切面的功能，需要协调更新框架升级发版（路由、认证、限流、安全），因此全部上沉到API Gateway



### 1.2.2 Mircoservice划分

如何划分服务的边界

- Business Capability：不同部门提供的职能
- Bounded Context：限界上下文



CQRS：将应用程序分为命令端和查询端。 （Polling publisher -> Transaction log tailiing）

- 命令端处理程序创建、更新和删除请求，并在数据更改时发出时间
- 查询端通过针对一个或多个物化视图执行查询来处理查询



### 1.2.3 Mircoservice 安全

通常在API Gateway进行统一的认证拦截。

API Gateway -> BFF -> Service

Biz AUth       ->  JWT  -> Request Args



对于服务内部，一般要区分身份认证和授权

- Full Trust
- Half Trust
- Zero Trust





## 1.3 gRPC和服务发现



### 1.3.1 什么是gRPC

A high-performance, open-source universal RPC framework



- 多语言：语言中立，支持多种语言
- 轻量级、高性能：序列化支持Protocol Buffer和JSON
- IDL：基于文件定义服务，通过proto3工具生成指定语言的数据结构
- 移动端：基于标准的HTTP2设计，支持双向流、消息头压缩、单TCP的多路复用、服务端推送等特性
- 服务而非对象、消息而非引用：促进微服务的系统间粗粒度消息交互设计理念
- 负载无关的：不同服务需要不同的消息类型
- 流：Streaming API
- 元数据交换：常见的横切关注点，如认证或跟踪，依赖数据交换
- 标准化状态码：客户端通常以有效的方式响应API调用返回的错误





### 1.3.2 HealthCheck

主动健康检查，可以再服务提供者服务不稳定时，被消费者所感知，临时从负载均衡中摘除，减少错误请求

平滑上下线



### 1.3.3 服务发现-客户端发现

一个服务示例被启动时，他的网络地址会被写到注册表上；当服务实例终止时，再从注册表中删除。注册表通过心跳机制动态刷新。

客户端使用一个负载均衡算法，去选择一个可用的服务



### 1.3.4 服务发现-服务端发现

客户端通过负载均衡器向一个服务发送请求，这个负载均衡器会查询服务注册表，并将请求路由到可用的服务实例上。

服务示例在服务注册表上被注册合注销



客户端通过LVS提供的虚拟IP来连接负载均衡器



### 1.3.5 服务中心的选型

实际场景：海量服务发现和注册，服务状态可以弱一致性，需要的说AP系统

阿里开源的nacos





## 1.4 多集群和多租户



### 1.4.1 多集群

L0服务，类似账号服务，一旦故障影响返回巨大，所以多集群的必要性：

- 从单一集群考虑，多个节点保证可用性，通常采用N+2的方式来冗余节点
- 从单一集群故障带来的影响面角度考虑冗余多套集群



**不同集群可以隔离使用不同的缓存资源**

- 多套冗余的集群对应多套独占的缓存，带来更好的性能和冗余能力
- 尽量避免业务隔离使用或者sharding带来的cache hit影响（按照业务划分集群资源）

业务隔离集群带来的问题时cache hit ratio下降，不同业务形态数据正交，我们退而求其次整个集群全部连接

- 客户端全部连接集群，通过负载均衡把热点数据均衡到所有集群



**客户端忽略服务发现中的cluster信息，连接全部节点**

- 长连接导致的内存和CPU开销，HealthCheck可以高达30%
- 短链接极大的资源成本和延迟

解决：选择合适的子集大小和选择算法

- 通常返回部分后端子集给客户端
- 后端平均分给客户端
- 客户端重启，保持重新均衡，同时对后端重启保持透明，同时连接的变动最小



#### MySQL-Redis一致性

缓存清理， 订阅binlog，通过后台异步的job，再广播到所有缓存集群



### 1.4.2 多租户

一个微服务架构中允许多系统共存时利用微服务稳定性以及模块化最有效的方式之一

存在问题

- 混用环境导致的不可靠测试
- 多套环境带来的硬件成本
- 难以做负载测试，仿真线上真实流量情况



#### **解决方法**

染色发布。

- 流量路由：能够基于流入栈中的流量类型叫做路由
- 隔离性：能够可靠的隔离测试和生产中的资源，保证对关键业务没有副作用

本质：跨服务传递请求携带上下文，数据隔离的流量路由法案



#### 全链路压测

传入上下文，跨服务使用metadata传递，每一个基础架构都能理解租户信息，并且能够基于组合路由隔离流量

网关：根据染色路由到指定的容器

redis: 根据染色，在同一个节点复制不同的db或者key，作为影子数据库

mysql：根据ddl自动复制出另一个影子mysql

数据准备：自己造，或者录制流量







# 二、error



## 2.1 Error vs Exception



### 2.1.1 Error

error就是普通的一个接口，普通的值

通常试用`errors.New`来返回一个error对象



Go的异常处理逻辑是不引入exception，在函数签名上实现error interface的对象



**Go的panic机制跟其他语言的exception不一样**

panic意味着fatal error，意味着代码不能继续运行，跟error差不多，表示不可恢复的程序错误



### 2.1.2 Error的好处

- 简单
- 考虑失败，而不是成功
- 没有隐藏的控制流
- 完全交给你来控制error
- Error are values



## 2.2 Error types

### 2.2.1 Sentinel Error

预定义的特定错误，我们称为sentinel error

```if err == ErrSometing{...}```

- **使用sentinel是最不灵活的错误处理策略**，返回不同的错误将破坏相等性检查
- 会在不同的包之间创建依赖

不依赖检查error.Error的输出



### 2.2.2 Error types

Error types是实现了error接口的自定义类型，能提供更多的上下文

- 会跟调用者产生强耦合，导致API变得脆弱。 因为需要用switch，并让自定义的error变为public
- 共享error values许多相同的问题，避免使用或者作为公共API的一部分



### 2.2.3 Opaque errors

最灵活的错误处理策略，与调用者之间的耦合最少。（不透明错误处理，秩序返回错误而不假设其内容）

Assert errors for behaviour，not type

- 少数情况下，二分错误处理方法是不够的，例如是否需要判断重试
- 通过断言错误是否实现了特定的行为来实现，而不是断言错误是特定的类型或值
- 可以不引入定义错误的包或者实际上不了解err的底层类型的情况下实现



## 2.3 Handling error



### 2.3.1 Eliminate error handling by eliminating errors

定义：

```go
type errWriter struct {
    io.Writer
    err error
}

func (e *errWriter) Write(buf []byte) (int, error) {
    if e.err != nil {
        return 0, e.err
    }
    
    var n int
    n, e.err = e.Writer.Write(buf)
    return n, nil
}
```

使用：

```go
func WriteResponse(w io.Writer, st Status, headers []Header, body io.Reader) error {
    ew := &errWriter{Writer: w}
    
    for _, h := range headers {
        fmt.Fprintf(ew, "%s: %s\r\n", h.Key, h.Value)
    }
    
    fmt.Fprint(ew, "\r\n")
    io.Copy(ew, body)
    return ew.err
}
```



### 2.3.2 Wrap errors

```go
func AuthenticateRequest(r *Request) error {
    return authenticate(r.User)
}
```

- 没有生成错误得file:line信息
- 没有导致错误得调用堆栈的堆栈信息

```go
func AuthenticateRequest(r *Request) error {
    err := authenticatre(r.User)
    if err != nil {
        return fmt.Errorf("authenticate failed:%v", err)
    }
    return nil
}
```

- 与sentinel errors或type assertions的使用不兼容
- fmt.Errorf波坏了原始错误，导致等值判定失败
- 记录多个错误，一直返回到程序的顶部

 

you should only handle errors once. Hadnling an error means inspecting the error value, and making a single decision



日志记录与错误无关且对调试没有邦族的信息应被视为噪音。

记录的原因是因为某些东西失败了，而日志包含了答案

- 错误要被日志记录
- 应用程序处理错误，保证100%完整性
- 之后不再报告当前错误



#### github.com/pkg/errors

```go
```





- 在你的应用代码中，试用errorsNew或者errors.Errorf返回错误
- 如果调用其他包内的函数，通常简单的直接返回（不然会有两倍日志）
- 如果和其他库进行协作，考虑试用errors.Wrap或者errors.Wrapf保存堆栈信息
- 直接返回错误，而不是到处打日志
- 在程序的顶部或者是工作的goroutine顶部（请求入口），使用%+v把堆栈详情记录
- 使用errors.Cause获取root error，再跟sentinel error判定

#### 总结

- Packages that are reusable across many projects only return root error values
- If the error is not going to be handled， wrap and return up the call stack
- Once an error is handled， it is not allowed to be passed up the call stack any longer





### 2.3.3 1.13版本的异常处理

