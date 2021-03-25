## 一、Maven POM
### 1. POM
POM是Maven工程的基本工作单元，包含了项目的基本信息，用于描述项目如何构建，声明项目依赖等
```xml
<project xmlns = "http://maven.apache.org/POM/4.0.0"
    xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <!-- 模型版本 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成， 如com.companyname.project-group，maven会将该项目打成的jar包放本地路径：/com/companyname/project-group -->
    <groupId>com.companyname.project-group</groupId>
 
    <!-- 项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
    <artifactId>project</artifactId>
 
    <!-- 版本号 -->
    <version>1.0</version>

   <!--项目产生的构件类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型，所以前面列的不是全部构件类型 -->
    <packaging>jar</packaging>


    <!--发现依赖和扩展的远程仓库列表。 -->
    <repositories>
        <!--包含需要连接到远程仓库的信息 -->
        <repository>
            <!--如何处理远程仓库里发布版本的下载 -->
             <releases>
              <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
               <enabled>true</enabled>
               <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
               <updatePolicy>never</updatePolicy>
               <!--当Maven验证构件校验文件失败时该怎么做：ignore（忽略），fail（失败），或者warn（警告）。 -->
               <checksumPolicy>ignore</checksumPolicy>
             </releases>
             <!-- 如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的 
                策略。例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
                <snapshots>
                 <enabled />
                <updatePolicy />
                <checksumPolicy />
             </snapshots>
             <!--远程仓库唯一标识符。可以用来匹配在settings.xml文件里配置的远程仓库 -->
             <id>banseon-repository-proxy</id>
             <!--远程仓库名称 -->
             <name>banseon-repository-proxy</name>
             <!--远程仓库URL，按protocol://hostname/path形式 -->
             <url>http://192.168.1.169:9999/repository/</url>
             <!-- 用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然 
                    而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
             <layout>default</layout>
         </repository>
     </repositories>

</project>
```

### 2. 父POM
父POM是Maven默认的POM，它包含了一些可以被继承的默认设置


## 二、依赖管理

### 1. 依赖调节
决定当多个手动创建的版本同时出现时，哪个依赖版本将会被使用。 如果两个依赖版本在依赖树里的深度是一样的时候，第一个被声明的依赖将会被使用

### 2. 依赖管理
Maven提供的一种高度控制的方法，直接的指定手动创建的某个版本被使用

### 3. 依赖范围

- 编译阶段（compile）：该范围表明相关依赖是只在项目的类路径下有效。默认取值。编译，测试，运行阶段都需要
- 供应阶段（provided）：该范围表明相关依赖是由运行时的 JDK 或者 网络服务器提供的。 在编译和测试时有效
- 运行阶段（runtime）：该范围表明相关依赖在编译阶段不是必须的，但是在执行阶段是必须的。在测试和运行时有效
- 测试阶段（test）：该范围表明相关依赖只在测试编译阶段和执行阶段。
- 系统阶段（system）：该范围表明你需要提供一个系统路径。<systemPath/>需要绝对路径。在编译和测试时有效，与本机系统关联，可移植性差
                                                    


scope的依赖传递：

A -> B -> C, 当C是test或者provided时，C直接被丢弃，A不依赖C；
             否则A依赖C，C的scope继承于B的scope。
