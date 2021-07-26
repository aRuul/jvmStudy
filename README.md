# 概述

## java虚拟机JVM

java虚拟机是一台执行java字节码的虚拟计算机，它拥有独立的运行机制，其运行的java字节码也未必由java语言编译而成。

JVM平台的各种语言可以共享java虚拟机带来的跨平台性、优秀的垃圾回收机制以及可靠的即时编译器。java技术的核心就是java虚拟机（jvm，java virtual machine）

## JVM的位置

![image-20210716200357558](https://gitee.com/aruul/a-ru-img/raw/master/img/20210716200358.png)

jvm是运行在操作系统之上的，没有和硬件直接交互

## JVM的整体结构

![image-20210716200826280](https://gitee.com/aruul/a-ru-img/raw/master/img/20210716200827.png)

​															==要求会自己画出这张图==

- HotSport VM是目前市面上最高性能的虚拟机的代表作之一。
- 它采用解释器与即时编译器并存的架构

## java代码执行流程

比如我们现在写了一个 HelloWorld.java 好了，那这个 HelloWorld.java 抛开所有东西不谈，那是不是就类似于一个文本文件，只是这个文本文件它写的都是英文，而且有一定的缩进而已。

那我们的 **JVM** 是不认识文本文件的，所以它需要一个 **编译** ，让其成为一个它会读二进制文件的 **HelloWorld.class**

如果 **JVM** 想要执行这个 **.class** 文件，我们需要将其装进一个 **类加载器** 中，它就像一个搬运工一样，会把所有的 **.class** 文件全部搬进JVM里面来。

![image-20210716200204379](https://gitee.com/aruul/a-ru-img/raw/master/img/20210716200206.png)

==由于跨平台的设计，java的指令都是根据栈来设计的。==跨平台性，指令集小，指令多。

相对于基于寄存器的架构，基于栈的架构的执行性能相比寄存器来说要差一些

## JVM的生命周期

**java虚拟机的启动**是通过引导类类(bootstrap class loader)创建一个初始化类(initial class)来完成，这个类是有虚拟机的具体实现指定的。

**虚拟机的执行:** 一个运行中的Java虚拟机有着一个清晰的任务：执行Java程序
程序开始执行时他才运行，程序结束时他就停止
执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程

**虚拟机的退出**
有如下的几种情况：

- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而异致Java虚拟机进程终止
- 某线程调用 Runtime类或 system类的exit方法，或 Runtime类的halt
  方法，并且Java安全管理器也允许这次exit或halt操作
- 除此之外，JNI( Java Native Interface)规范描述了用JNI Invocation APｴ来加载或卸载Java虚拟机时，Java虚拟机的退出情况

## HotSpot虚拟机

不管是JDK6还是最常用的JDK8，默认虚拟机都是HotSpot

> 名称中的 Hotspot指的就是它的热点代码探测技术。
> 通过计数器找到最具编译价值代码，触发即时编译或上替拧
> 通过编译器与解释器协同工作，在最优化的程序响应时间与最佳执行性能中取得平衡

---

Graal VM  ---"Run Programs Faster Anywhere"  最有可能替代HotSpot。

# 类加载子系统

![image-20210717153811503](https://gitee.com/aruul/a-ru-img/raw/master/img/20210717153813.png)

**加载--> 链接--> 初始化**

**如果自己手写一个java虚拟机的话，主要考虑哪些结构？  类加载器和执行引擎**

## 类的加载过程

![image-20210717154600485](https://gitee.com/aruul/a-ru-img/raw/master/img/20210717154605.png)

- 类加载子系统负责从文件系统或者网络系统中加载Class文件，class文件在文件开头有特定的文件标识
- ClassLoader只负责class文件的加载，至于它是否可以运行，则是由Execution Engine来jued
- 加载的**类信息存放在一块成为方法区**的内存空间。
- 除了类的信息之外，**方法区还会存放运行时的常量池信息**，可能还包括**字符串常量和数字常量**（这部分常量信息是Class文件中常量池部分的内存映射）

---

具体的类加载过程图

![image-20210717160035120](https://gitee.com/aruul/a-ru-img/raw/master/img/20210717160036.png)

### 第一步 加载 --生成 java.lang.Class对象

1. 通过一个类的全限定名获取此类的二进制字节流
2. 将这个字节流所代表的静态数据结构转化为方法区的运行时数据结构
3. 在内存中**生成**一个代码这个类的  **java.lang.Class对象**，作为方法区这个类的各种数据的访问入口

### 第二步 链接

#### 1.**验证( Verify):**

- 目的在于确保class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全。
- 主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证

> java .class文件开头是COFFBABE

#### 2.**准备( Prepare):**

- 为**类变量**分配内存并且设置该类变量的**默认初始值**，即零值。

  ```java
  class Demo{
      //这时a会被赋值为0，在初始化时才会被初始化为1
      private static int a = 1;
  }
  ```

- 这里**不包含用final修饰的 static**,因为**final在编译的时候就会分配了**，准备阶段会显式初始化

- 这里不会为实例变量分配初始化，类变量会分配在方法区中，而**实例变量**是会随着对象一起分
  配到**java堆**中。

#### 3.**解析( Resolve)**

- 将常量池内的符号引用转换为直接引用的过程
- 事实上，解析操作住住会伴随着JVM在执行完初始化之后再执行。
- 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
- 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的
  CONSTANT Class info CONSTANT Fieldref info, CONSTANT Methodref info

### 第三步 初始化

- 初始化阶段就是执行**类的构造器方法 clinit** 的过程（cl可以理解为 class  ，init为 inital）

- 此方法不需要定义，是javac编译器自动收集类中的所有变量的赋值动作和静态代码块中的语句合并而来的。

- 构造器方法按语句顺序执行所有**类变量**（static修饰的成员变量）显式初始化和**静态代码块中语句**

  ```java
  /*这里的话，a最终的值为1
  * 首先在第二步链接的准备阶段，会将a初始化为0
  * 在第三步初始化时，类的构造器方法按照顺序执行，先将a赋值为10，再将a赋值为1，所以最后a=1
  */
  class Demo{
      static{
          a = 10;
      }
      private static int a = 1;
  }
  ```

- **类的构造器方法 clinit**不同于类的构造函数，==当一个类中不存在类变量的时候，则.class文件中没有<clinit>()==

- 若该类有父类，JVM会保证父类的 clinit()执行完毕之后再执行子类的clinit()

  ```java
  class Father{
      public static int a = 1;
      static{
          a = 2;
      }
  }
  
  class Son extends Father{
      public static int b = a;
      
      public static void main(String []args){
          //最终的结果为2
          Sysytem.out.println(Son.b);
      }
  }
  ```

- 虚拟机必须保证一个类的<clinit>方法在多线程下被同步加锁，即==一个类只会被加载一次==

## 类的加载器

### 类加载器分类

JVM支持**两种**类型的**类加载器**，分别为

1. **引导加载器**(Bootstrap ClassLoader)
2. **自定义加载器(**User-Defined ClassLoader)，**是指继承自ClassLoader的加载器**

---

### 虚拟机自带的加载器

![image-20210718143802701](https://gitee.com/aruul/a-ru-img/raw/master/img/20210718143804.png)

#### 启动类加载器(引导类加载器 Bootstrap ClassLoader)

- 我们不能直接获取到

- java的**核心类库**都是使用**引导类加载器**来进行加载的。
- 这个类加载使用C++/C来实现的，嵌套在JVM内部
- 并不继承于java.lang.ClassLoader
- **加载扩展类和程序类加载器，并指定为它们的父加载器**
- 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类

#### 拓展类加载器(Extension ClassLoader)

- java语言编写
- 派生于ClassLoader类
- 父加载器为启动类加载器
- 从java。ext.dirs系统属性所指定的目录中加载类库，或者从JDK安装目录的jre/lib/ext子目录下加载类库。**如果用户创建的jar在此目录下，也会自动由扩展类加载器加载**

#### 应用程序类加载器(系统类加载器AppClassLoader)

- java语言编写
- 派生于ClassLoader类
- **父加载器为拓展类加载器**
- 负责加载**环境变量classpath**或**系统属性java.class.path**指定路径下的类库
- **我们自己写的类中默认加载器就是应用程序类加载器**

### 用户自定义加载类

![image-20210718151625921](https://gitee.com/aruul/a-ru-img/raw/master/img/20210718151628.png)

## ClassLoader

**ClassLoader类是一个抽象类，其后所有的类加载器都继承自ClassLoader(不包括启动加载类)**

这里暂且了解就够了（嗯，大概

## 双亲委派机制

> Java虚拟机对class文件采用的是**按需加载**的方式，也就是需要的时候才会将它的class文件加载到内存，生成class对象。
>
> 而在加载某个类的class文件时，JVM采用**双亲委派模式**，即把请求交给父类处理，是一种任务委派模式。

**双亲委派机制**：

1. 当一个类加载器收到了加载请求时，它是不会先自己去尝试加载的，而是委派给父类去完成，

2. 如果父类还有父类加载器，则进一步向上委托，以此类推，直到顶层的启动类加载器
3. 如果父类加载器可以完成加载任务，则成功返回；否则，子类加载器尝试去加载

==可以理解为 类加载器这个家族很重亲情，能爸爸做的就爸爸做，爸爸实在做不了就儿子做==

![image-20210718154731162](https://gitee.com/aruul/a-ru-img/raw/master/img/20210718154732.png)

优势：

- 避免类的重复加载

- 保护程序的安全，防止核心api被随意的篡改

  比如你自定义一个包 java.lang,包下面有一个类Demo

  ```java
  package java.lang
  class Demo{
      public static void main(String[] args){
          System.out.println("test");
      }
  }
  ```

  当你运行的时候，由于双亲委派机制，这和Demo类是java开头的，所以最终的加载器是引导类加载器。(引导类加载器只加载包名为java、javax、sun等开头的类)，而java核心类包中并没有Demo这个类，所以就会报错。

## 其他小的点

### 在JVM中表示两个class对象相同的两个必要条件：

1. 类的完整类名要一致，包括包名
2. 加载 这个类的ClassLoader(指ClassLoader实例对象)必须相同。

### 对类加载器的引用

1. JVM必须知道一个类型是由启动加载器加载的还是由用户类加载器加载的。
2. 如果一个类型是由用户类加载器加载的，那么JVM会将这个类加载器的一个引用作为类型信息的一部分保存在方法区中。
3. 当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的类加载器是相同的



----

# 运行时数据区

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210718163557.png" alt="image-20210718163554539" style="zoom:80%;" />

   红色的为**线程共享区域**，**一个虚拟机实例对应一份**

​	**蓝色的是一个线程对应一份**

## 程序计数器(PC计数器)

> **实际上，JVM中的程序计数器(PC计数器)是对物理pc寄存器的一种抽象模拟**，并非是广义上的物理寄存器。
>
> 
>
> 程序计数器其实就是一个指针，它指向了我们程序中下一句需要执行的指令，它也是内存区域中唯一一个不会出现OutOfMemoryError的区域，而且占用内存空间小到基本可以忽略不计。这个内存仅代表当前线程所执行的字节码的行号指示器，字节码解析器通过改变这个计数器的值选取下一条需要执行的字节码指令。
>
> 如果执行的是native方法，那这个指针就不工作了。

**程序计数器其实就是一个指针，它指向了我们程序中下一句需要执行的指令**

**每个线程都有自己的程序计数器**，生命周期与线程一致。

**作用**： PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎读取下一条指令。

下面是一个java代码的字节码文件main方法的一部分：

![image-20210719150122811](https://gitee.com/aruul/a-ru-img/raw/master/img/20210719150124.png)

### 使用PC寄存器存储字节码指令地址有什么用呢？

(为什么使用PC寄存器记录当前线程的执行地址呢？)

因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行。
JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。

### PC寄存器为什么会被设定为线程私有？

cpu在执行多线程的时候，实际上会不停地做任务切换，这样必然导致经常中断或恢复，如何保证分毫无差呢？为了能够准确地记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。

### 并行 并发

- **并发：** 同一时间段，多个任务都在执行 (单位时间内不一定同时执行)；
- **并行：** 单位时间内，多个任务同时执行（可以与串行相理解）

## 虚拟机栈

**栈管运行，堆管存储。则虚拟机栈负责运行代码，而虚拟机堆负责存储数据。**

> Java虚拟机栈( Java virtua1 Machine Stack),早期也叫Java栈。
> 每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧( Stack Frame),对应着一次次的Java方法调用。
>
> **是线程私有的**

- **作用**：主管java程序的运行，它保存方法的局部变量(8中基本数据类型/对象的引用)、部分结果，并参与方法的调用和返回。

- **优点：**1. 栈是一种快速有效的分配存储方式，访问速度仪次于程序计数器。

  ​			2.JVM直接对Java栈的操作只有两个：每个方法执行，伴随着进（入栈、压栈）、执行结束后的出栈工作

  ​			3.对于栈来说不存在垃圾回收问题

#### 异常

如果线程请求的栈的深度大于虚拟机栈的最大深度，就会报 **StackOverflowError** （这种错误经常出现在递归中）。

Java虚拟机也可以动态扩展，但随着扩展会不断地申请内存，当无法申请足够内存时就会报错 **OutOfMemoryError**(OOM)。

#### 设置栈内存大小

我们可以使用参数-Xss选项来设置线程的最大栈空间，栈大小直接决定了函数调用的最大可达深度。

#### 栈的存储单元-栈帧

- 每个线程都有自己的栈，栈中的数据都是以**栈帧( Stack Frame)**格式存在。
- 在这个线程上正在执行的**每个方法**都各自**对应一个栈帧( Stack Frame)**
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。
- 在一条活动线程中，一个时间点上，只会有一个活动的栈帧。
  - 即只有当前正在执行的方法的栈帧（**栈顶栈帧**）是有效的，这个栈帧被称为**当前栈帧( Current Frame)**,
  - 与**当前栈帧相对应的方法就是当前方法**（ CurrentMethod),
  - 定义这个方法的类就是**当前类**( Current Class).
- **执行引擎运行的所有字节码指令只针对当前栈帧进行操作**
  如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，成为新的当前帧。

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210719160147.png" alt="QQ图片20210719160131" style="zoom:80%;" />

- Java方法有两种返回函数的方式，一种是正常的函数返回，使用 return指令；另外一种是抛出异常。**不管使用哪种方式，都会导致栈帧被弹出。**
- 不同线程中所包含的栈帧是不允许存在相互引用的，即不可能在一个栈帧之中引用另外一个线程的栈帧。
- 如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧

#### 栈帧的内部结构【ToDo】

(这里我并没有仔细看，所以只粗略了写了一小点内容)

![image-20210719161844500](https://gitee.com/aruul/a-ru-img/raw/master/img/20210719161847.png)

- 局部变量表( Local Variables)

  ​     **定义为数字数组**

  ​	**局部变量表，最基本的存储单元是Slot（变量槽）**

  ![image-20210719162352247](https://gitee.com/aruul/a-ru-img/raw/master/img/20210719162353.png)

- 操作数栈( operand stack)（或表达式）

  > 操作数栈，在方法执行过程中，根据字节码指令，在栈中写入数据或者提取数据，即入栈、出栈

- 动态链接( Dynamic Linking)（或指向运行时常量池的方法引用）

  > 每个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用。包含这个引用的目的就是为了支持当前方法的代码能够实现**动态链接**
  >
  > 在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为**符号引用**( Symbolic Reference)保存在class文件的常量池里。
  > 比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**

- 方法返回地址( Return Address)（或方法正常退出或者异常退出的定义）

  > 无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。
  >
  > 方法正常退出时，调用者的pc计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址。
  >
  > 而通过异常退出的，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。

- 一些附加信息

  > 栈帧中还允许携带与Java虚拟机实现相关的一些附加信息。例如，对程序调试提供支持的信息。

#### 静态变量和局部变量

  **局部变量在使用前一定要显式赋值，否则编译不通过**

> 我们知道类变量表有两次初始化的机会，第一次是在“准备阶段”，执行系统初始化，对类变量设置零值，
>
> 另一次则是在“初始化”阶段，赋予程序员在代码中定义的初始值。
> 和类变量初始化不同的是，局部变量表不存在系统初始化的过程，这意味着一旦定义了局部变量则必须人为的初始化，否则无法使用。

**补充：**

- 在栈帧中，与性能调优关系最为密切的部分就是前面提到的局部变量表。在方法执行时，虚拟机使用局部变量表完成方法的传递。
- 局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收。

#### java中方法重写的本质？

![image-20210720135306618](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720135308.png)

#### 举例栈溢出的情况？

如果线程请求的栈的深度大于虚拟机栈的最大深度，就会报 **StackOverflowError** （这种错误经常出现在递归中）。

#### 调整栈大小，就能保证不出现溢出吗？

使用参数-Xss选项来设置线程的最大栈空间，但不一定保证不出现溢出，如果碰到递归，可能会出现溢出。

#### 分配的栈内存越大越好吗？

并不是，栈是私有的，如果设置的每一个栈的空间太大了，就会导致最后可分配的线程数量变少，甚至出OOM

#### 垃圾回收是否会涉及到虚拟机栈？

不会，他直接操作的就是出栈、入栈，不存在GC(垃圾回收)，存在Error

#### 方法中定义的局部变量是否线程安全？【ToDo】

视情况而定

## 本地方法接口的理解

![ ](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720135756.png)



> **本地方法(Native Method)**：一个 Native Method就是一个Java调用非Java代码的接口。
>
> 一个Native Method是这样一个Java方法：该方法的实现由非Java语言实现，比如C，这个特征并非Java所特有，很多其它的编程语言都有这一机制，比如在C++中你可以用 extern"C"告知C++编译器去调用一个C的函数。

## 本地方法栈

Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用。

> 并不是所有的JVM都支持本地方法。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。
>
> 如果JVM产品不打算支持 native方法，也可以无需实现本地方法栈。
> 在 Hotspot JVM中，直接将本地方法栈和虚拟机合二为一。

## 堆(heap)

![image-20210720144022441](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720144023.png)

**堆是线程共享区域**

### 核心概念

- **一个JVM实例只存在一个堆内存**，堆也是Java内存管理的核心区域。

  ***即，每个Java应用程序都使用一个独立的 JVM。***

- **Java堆区在JVM启动的时候即被创建，其空间大小也就确定了**，是JVM管理的最大一块内存空间。
  堆内存的大小是可以调节的。

- 《Java虚拟机规范》规定，堆可以处于**物理上不连续的内存空间**中，但在**逻辑上它应该被视为连续**的。

- 所有的**线程共享java堆**，在这里还可以划分线程**私有的缓冲区**（ Thread Local Allocation Buffer, TLAB)

- **所有的对象实例以及数组都应当在运行时分配在堆上**。( The heap is the run- time data area from
  which memory for all class instances and arrays is allocated
  *周老师说的是：“几乎”所有的对象实例都在这里分配内存。---------从实际使用角度看的。*

- **数组和对象可能永远不会存储在栈上**，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。

- **在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。**

- 堆，是GC( Garbage Collection,垃圾收集器)执行垃圾回收的重点区域

---

![image-20210720145811206](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720145812.png)

​														栈 、堆、方法区之间的关系

---

### 堆的细分内存结构

在JDK7以及之前的版本，堆通常被分为下面三个部分：

1. **新生区(Yong Generation)**
2. **养生区(Old Generation)**
3. 永久区/永久代(Permanent Generation)

Java8及之后堆内存逻辑上分为三部分：

1. **新生区**

   **年轻代**又可以划分为**Eden空间、 Survivor0空间和 Survivor1空间**（有时也叫做from区、to区）。

2. **养老区**

3. 元空间

> 由于翻译不同 叫法也不尽相同，下面是常见的叫法：
>
> 新生区=新生代= 年轻代   
> 养老区 =老年区 =老年代
> 永久区 =永久代

### 设置堆空间大小

Java堆区用于存储Java对象实例，堆的大小在JVM启动时就已经设定好了，

可以通过选项"-Xmx"和"Xms"来进行设置。

>“-Xms"用于表示堆区的起始内存，等价于ーXX: Initialheapsize
>“-Xmx"则用于表示堆区的最大内存，等价于-XX: Maxheaps1ze

### 年轻代与老年代

存储在JVM中的Java对象可以被划分为**两类**：

- **一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速**
- **另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致。**

Java**堆区**进一步细分的话，可以划分为**年轻代**和**老年代**
其中**年轻代**又可以划分为**Eden空间、 Survivor0空间和 Survivor1空间**（有时也叫做from区、to区）。

==几乎所有的java对象都是在Eden区被new出来的。==

==绝大部分的java对象的销毁都在新生代进行==

![image-20210720154016863](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720154018.png)

#### 调参与占比

新生代：老年代=1：2

新生代：Eden:s0:s1=8:1:1

![image-20210720154338541](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720154339.png)



![image-20210720154605960](https://gitee.com/aruul/a-ru-img/raw/master/img/20210720154607.png)

### 对象分配的一般过程

> 1.new的对象先放伊甸园区。此区有大小限制。
>
> **【注意】大对象直接进入老年代，大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。**
>
> 2.当伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收( Minor GC),将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区
>
> **【注意】只有伊甸园区的空间满的时候才会触发垃圾回收( Minor GC又叫YGC【Young GC】),幸存区满了并不会触发**
>
> 3.然后将伊甸园中的剩余对象移动到幸存者0区。
>
> 4.如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区。
>
> 5.如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。
>
> 6,啥时候能去养老区呢？可以设置次数。默认是15次。
> 可以设置参数：-XX: Maxtenuringthreshold=<N>进行设置

**针对幸存者s0,s1区的总结**：复制之后有交换，谁空谁是to

**关于垃圾回收**：频繁在新生区收集，很少在养老区收集，几乎不在永久区/元空间收集。

### 对象分配的特殊情况

![image-20210721151057588](https://gitee.com/aruul/a-ru-img/raw/master/img/20210721151059.png)

​																			配合P73来理解

### Minor GC、Major GC、Full GC

JVM在进行GC时，并非每次都对上面三个内存(新生代、老年代；方法区)区域一起回收的，**大部分时候回收的都是指新生代。**

针对 Hotspot JVM的实现，它里面的GC按照回收区域又分为两大种类型：

一种是**部分收集(Partial GC)**
一种是**整堆收集(Full GC)**

- 部分收集：不是完整收集整个java堆的垃圾收集。其中又分为：
  - 新生代收集( Minor GC/ Young GC):  只是新生代的垃圾收集
  - 老年代收集( Major GC/Old GC):  只是老年代的垃圾收集。
    *目前，只有 CMS GC会有单独收集老年代的行为。*
    ***注意，很多时候 Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。***
  - 混合收集( Mixed GC):收集整个新生代以及部分老年代的垃圾收集。
    *目前，只有G1 GC会有这种行为*
- 整堆收集(Full GC): 收集整个java堆和方法区的垃圾收集。

#### 年轻代GC( Minor GC)触发机制

- 当年轻代空间不足时，就会触发 Minor GC,这里的年轻代满指的是Eden代满， Survivor满不会引发GC。(每次 Minor GC会清理年轻代的内存）
- 因为java对象大多都具备**朝生夕灭**的特性，所以 **Minor GC非常频繁**，一般**回收速度也比较快**。这一定义既清晰又易于理解。
- Minor GC会引发STW,暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行

#### 老年代GC(Major GC)触发机制

- 指发生在老年代的GC,对象从老年代消失时，我们说“ Major GC”或“Full GC”发生了
- 出现了 Major GC，经常会伴随至少一次的 Minor GC(但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行 Major GC的策略选择过程）。也就是在老年代空间不足时，会先尝试触发 Minor GC。如果之后空间还不足，则触发 Major Gc
- Major GC的速度一般会比 Minor GC慢10倍以上，STW的时间更长。
- 如果 Major GC后，内存还不足，就报OOM了

#### Full GC触发机制

触发Full GC执行的情况有如下五种：
(1)调用 System. gc()时，系统建议执行Full GC,但是不必然执行
(2)老年代空间不足
(3)方法区空间不足
(4)通过 Minor GC后进入老年代的平均大小大于老年代的可用内存
(5)由Eden区、 survivor spacee( From Space)区向 survivor space1(To Space)区复制时，对象大小大于 To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小
*说明：full gc是开发或调优中尽量要避免的。这样暂时时间会短一些。*

**一般来说 出现OOM，则极大可能进行了Full GC**

### 为什么要把Java堆分代？不分代就不能正常工作了吗？

- 经研究，不同对象的生命周期不同。70%-99%的对象是临时对象。

  - 新生代：有Eden、两块大小相同的Survivor(又称作 s0/s1或from/to )构成，to总为空。
  - 老年代：存放新生代中经历多次GC依旧存活的对象

- 其实不分代完全可以，分代的唯一理由就是优化GC性能。

  - 如果没有分代，那所有的对象都在一块，就如同把一个学校的人都关在一个教室。GC的时候要找到哪些对象没用，这样就会对堆的所有区域进行扫描。
  - 而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当GC的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来。

---

###  针对不同年龄段的对象分配原则

- 优先分配到Eden

- 大对象直接分配到老年代：
  *尽量避免程序中出现过多的大对象*

- 长期存活的对象分配到老年代

- 动态对象年龄判断：
  **如果Survivor区中相同年龄的所有对象大小的总和 大于 Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。**

  > **【比如】**：Survivor区中有6个相同年龄的对象，都是10岁，而且这6个对象占用的空间大于 Survivor空间的一半，则其他大于10岁的对象可以直接进入老年代。

- 空间分配担保：
  *-XX:HandlePromotionFailure ，也就是经过Minor GC后，所有的对象都存活，因为Survivor比较小，所以就需要将Survivor无法容纳的对象，存放到老年代中。*

### TLAB【ToDo】

**TLAB(Thread Local Allocation Buffer)中文意思是线程本地分配缓冲区**

#### 堆空间都是共享的么？

不一定，因为还有TLAB这个概念，**在堆中划分出一块区域，为每个线程所独占**

#### 为什么要有TLAB？

- TLAB：Thread Local Allocation Buffer，也就是为每个线程单独分配了一个缓冲区
- 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
- 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。

#### 小结

- 年轻代是对象的诞生、成长和消亡的区域，一个对象在这里产生、应用，最后被垃圾回收器收集、结束生命。

- 老年代放置长生命周期的对象。通常都是从Survivor区域筛选拷贝过来的java对象。

  普通的对象会被分配在TLAB上，当对象较大的，JVM会试图直接分配在Eden其他位置上

  当对象太大，无法在新生代找到足够长的连续空闲空间，JVM就会把对象直接分配到老年代。

- 当GC只发生在年轻代，回收年轻代对象的行为称为Minor GC。

  当GC发生在老年代，则被成为Major GC或者Full GC。

  一般来说，Minor GC的发生频率比Major GC要高，即年轻代中垃圾回收频率大大高于年轻代。

---

## 方法区

### 栈 堆 方法区的交互关系

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210722145839.png" alt="image-20210722145836188" style="zoom:50%;" />

> **方法区** 是用于存放类似于元数据信息方面的数据的，比如类信息，常量，静态变量，编译后代码···等
>
> 类加载器将 .class 文件搬过来就是先丢到这一块上
>
> **堆** 主要放了一些存储的数据，比如对象实例，数组···等，
>
> **栈** 这是我们的代码运行空间。我们编写的每一个方法都会放到 **栈** 里面运行。

上面的图中，new PerSon()相当于创建了一个对象实例，则放在**堆**中

Person存放了这个类的信息，存放在方法区中

person是存放在Java栈的局部变量表中

![image-20210722151405350](https://gitee.com/aruul/a-ru-img/raw/master/img/20210722151407.png)

### 方法区的基本理解

> 《java虚拟机规范》中明确说明：“尽管所有的**方法区在逻辑上是属于堆的一部分**，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。”但对于HotSpot JVM而言，方法区还有一个别名叫Non-Heap(非堆)，目的就是要和堆分开。
>
> 所以，**方法区看作是独立于java堆的内存空间。**

- 方法区和堆一样，是各个线程共享的内存区域
- 方法区在jvm启动的时候被创建，而且他的实际物理内存空间中和java堆区一样都可以是不连续的
- 方法区的大小跟堆空间大小都可以选择固定大小或者可拓展
- 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，悉尼及同样会抛出内存溢出错误，
- 关闭JVM就会释放这个区域的内存

### 方法区的演进

**在 JDK7 及以前，习惯上把方法区，称为永久代。JDK8开始，使用元空间取代了永久代**

本质上，方法区和永久代并不等价

当年使用永久代，不是好的idea，因为实在JVM的内存中运行，导致Java程序更容易OOM

**而jdk1.8及以后，元空间不在虚拟机设置的内存中，而是使用本地内存**

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210722160956.png" alt="image-20210722160955144" style="zoom:80%;" />

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210722161025.png" alt="image-20210722161024342" style="zoom:80%;" />

### 方法区的内部结构

方法区主要存放的信息如下：

![image-20210722162315731](https://gitee.com/aruul/a-ru-img/raw/master/img/20210722162317.png)

《深入理解java虚拟机》中对方法区存储的内容描述如下：

> 它用于存储已被虚拟机加载的**类型信息**、**常量、静态变量**、**即时编译器编译之后的代码缓存**等。

**类型信息**：

包含了下面几个信息：

- 这个类的完整有效名称(全名=包名+类名)
- 这个类型直接父类的完整有效名(对于interface或者Object,都没有父类)
- 这个类型的修饰符(public,abstract,final)
- 这个类型直接接口的一个有序列表

**域信息**(成员变量)：

- JVM必须在方法区中保存类型的所有域相关的信息以及域的声明顺序。
- 域的相关信息包括：域名称、域类型、域修饰符(public、static、final、volatile等)

**non-final的类变量**

- 静态变量和类关联在一起，随着类的加载而加载，它们成为类数据在逻辑上一部分
- **类变量被所有的实例共享，即使没有类实例时也可以访问**

**【注意】被声明为final的类变量的处理方法则不同，每个全局常量在编译的时候就会被分配了**

**方法信息：**

- 方法名称
- 方法的返回类型(或void)
- 方法参数的数量和类型（按顺序）
- 方法的修饰符（ public, private, protected, static,final,synchronized, native, abstract的一个子集）
- 方法的字节码( bytecodes)、操作数栈、局部变量表及大小（ abstract和native方法除外）
- 异常表( abstract和 native方法除外)

### 常量池

在字节码文件内部，包含了**常量池**。

当通过类的加载器加载运行之后，就叫做**运行时常量池**。

- 一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述符信息外
- 还包含一项信息就是**常量池表**（**Constant Pool Table**），包括**各种字面量和对类型、域和方法的符号引用**

#### 为什么需要常量池？

一个java源文件中的类、接口，编译后产生一个字节码文件。而Java中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，所以我们将所需用到的结构信息记录在常量池中，并通过**引用的方式**，来加载、调用所需的结构。

#### 常量池里有什么？

- 数量值
- 字符串值
- 类引用
- 字段引用
- 方法引用

> **常量池，可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类**
> **名、方法名、参数类型、字面量等类型。**

### 方法区的演进

|   JDK版本    |                           演变细节                           |
| :----------: | :----------------------------------------------------------: |
| JDK1.6及以前 |               有永久代，静态变量存储在永久代上               |
|    JDK1.7    | 有永久代，但已经逐步“去永久代”，**字符串常量池、静态变量从永久代中移除，保存在堆中** |
|    JDK1.8    | 无永久代，**类型信息、字段、方法、常量保存在本地内存的元空间**，但**字符串常量池、静态变量仍然在堆中**。 |

- JDK1.6

![image-20210723162257646](https://gitee.com/aruul/a-ru-img/raw/master/img/20210723162259.png)

- JDK1.7

![image-20210723162441857](https://gitee.com/aruul/a-ru-img/raw/master/img/20210723162443.png)

- JDK1.8

![image-20210723162528585](https://gitee.com/aruul/a-ru-img/raw/master/img/20210723162529.png)

#### 为什么永久代要被元空间替代？

> JRockit是和HotSpot融合后的结果，因为JRockit没有永久代，所以他们不需要配置永久代
>
> 随着Java8的到来，HotSpot VM中再也见不到永久代了。但是这并不意味着类的元数据信息也消失了。这些数据被移到了一个与堆不相连的本地内存区域，这个区域叫做元空间（Metaspace）。
>
> 由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间，这项改动是很有必要的，原因有：
>
> - 为永久代设置空间大小是很难确定的。
>
> 在某些场景下，如果动态加载类过多，容易产生Perm区的oom。比如某个实际Web工 程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。
>
> “Exception in thread‘dubbo client x.x connector'java.lang.OutOfMemoryError:PermGen space”
>
> 而元空间和永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。 因此，默认情况下，元空间的大小仅受本地内存限制。
>
> - 对永久代进行调优是很困难的。
>   - 主要是为了降低Full GC
>
> 有些人认为方法区（如HotSpot虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区类型卸载的收集器存在（如JDK11时期的ZGC收集器就不支持类卸载）。 一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收有时又确实是必要的。以前sun公司的Bug列表中，曾出现过的若干个严重的Bug就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏
>
> 方法区的垃圾收集主要回收两部分内容：常量池中废弃的常量和不在使用的类型

#### String Table为什么要调整？

JDK7中将StringTable放在堆空间中。

因为永久代的回收效率很低，在Full GC时才会被执行永久代(方法区)的垃圾回收，而Full GC是老年代空间不足、永久代不足时才会触发。这就导致StringTable回收效率不高，而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。

放到堆里，能及时回收内存。

#### 静态变量存放在哪里？

静态引用对应的对象实体始终都存在堆空间。

### 方法区的垃圾回收

实际上java虚拟机规范中并没有规定方法区一定要垃圾回收

方法区的垃圾收集主要回收两个部分：**常量池中废弃的常量**和**不再使用的类型**

常量池之中主要存放的两大类常量：字面量和符号引用。

字面量比较接近Java语言层次的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

**HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。**

> 判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：
>
> - 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如osGi、JSP的重加载等，否则通常是很难达成的。
> - 该类对应的java.lang.C1ass对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。I Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。关于是否要对类型进行回收，HotSpot虚拟机提供了-Xnoclassgc参数进行控制，还可以使用-verbose:class 以及 -XX：+TraceClass-Loading、-XX：+TraceClassUnLoading查看类加载和卸载信息
> - 在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及oSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。

## 小结

![image-20210724145709334](https://gitee.com/aruul/a-ru-img/raw/master/img/20210724145710.png)

## 大厂面试题

百度 三面：说一下JVM内存模型吧，有哪些区？分别干什么的？

蚂蚁金服： Java8的内存分代改进 JVM内存分哪几个区，每个区的作用是什么？ 一面：JVM内存分布/内存结构？栈和堆的区别？堆的结构？为什么两个survivor区？ 二面：Eden和survior的比例分配

小米： jvm内存分区，为什么要有新生代和老年代

字节跳动： 二面：Java的内存分区 二面：讲讲jvm运行时数据库区 什么时候对象会进入老年代？

京东： JVM的内存结构，Eden和Survivor比例。 

JVM内存为什么要分成新生代，老年代，持久代。新生代中为什么要分为Eden和survivor。

> - 如果没有Survivor，Eden区每进行一次Minor GC，存活的对象就会被送到老年代。老年代很快被填满，触发Major GC.老年代的内存空间远大于新生代，进行一次Full GC消耗的时间比Minor GC长得多,所以需要分为Eden和Survivor。
> - Survivor的存在意义，就是减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代。
> - 设置两个Survivor区最大的好处就是解决了碎片化，刚刚新建的对象在Eden中，经历一次Minor GC，Eden中的存活对象就会被移动到第一块survivor space S0，Eden被清空；等Eden区再满了，就再触发一次Minor GC，Eden和S0中的存活对象又会被复制送入第二块survivor space S1（这个过程非常重要，因为这种复制算法保证了S1中来自S0和Eden两部分的存活对象占用连续的内存空间，避免了碎片化的发生）

天猫： 一面：Jvm内存模型以及分区，需要详细到每个区放什么。 一面：JVM的内存模型，Java8做了什么改

拼多多： JVM内存分哪几个区，每个区的作用是什么？

美团： java内存分配 jvm的永久代中会发生垃圾回收吗？ 一面：jvm内存分区，为什么要有新生代和老年代？

# 对象常见知识点

**首先，要明白对象是在堆中的。**

## 对象创建方式

- new :最常见的方式
- Class的newInstance方法
- Constructor的newInstance(XXX)，反射的方式，可以调用空参/带参的构造器
- 使用clone():不调用任何构造器，要求当前的类要实现Cloneable接口中的clone()
- 使用反序列化：从网络、文件中获取对象的二进制流
- 第三方库Objenesis

## 创建对象的步骤

### 1.判断对象对应的类是否加载、链接、初始化

- 虚拟机遇到一条new指令，首先检查这个指令的参数是否在Metaspace(元空间)的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化。
  - 如果没有，那么在双亲委派模式下，使用当前类加载器一ClassLoader+包名+类名为key进行查找对应的.class文件。
    - 如果没有找到文件，则抛出ClassNotFoundException异常，
    - 如果找到，则进行类的加载，并生成对应的Class类对象

> **符号引用**
> 符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能够无歧义地定位目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到内存中。

### 2.为对象分配内存

首先计算对象占用空间的大小，接着在堆中划分一块内存给新对象。如果实例成员变量是引用变量，仅分配引用变量空间即可，即4个字节大小。

在分配的内存空间的时候：

- 如果内存规整，使用的是**指针碰撞法**来分配内存
- 如果内存不规整，已使用的内存和未使用的内存相互交错，那么虚拟机使用**空闲列表**来分配。**意思是虚拟机维护了一个列表，记录上那些内存块是可用的，再分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容。这种分配方式成为了 “空闲列表（Free List）”**

【补充】

**指针碰撞法**：意思是所有用过的内存在一边，空闲的内存放另外一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针指向空闲那边挪动一段与对象大小相等的距离罢了。

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210724154035.png" alt="image-20210724154034553" style="zoom:80%;" />

<img src="https://gitee.com/aruul/a-ru-img/raw/master/img/20210724154221.png" alt="image-20210724154220080" style="zoom:80%;" />

### 3.处理并发安全问题

- 采用CAS配上失败重试保证更新的原子性
- 每个线程预先分配TLAB -----通过设置 -XX:+UseTLAB参数来设置（区域加锁机制）
  - 在Eden区给每个线程分配一块区域

### 4.初始化分配到的空间

给所有属性设置默认值，保证对象实例字段在不赋值可以直接使用

- 属性的默认

### 5.设置对象的对象头

将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现。

### 6.执行init方法进行初始化

在java程序的视角看来，初始化才正式开始。初始化成员变量，执行实例化代码块，调用类的构造器方法，并把堆内对象的首地址赋值给引用变量。

主要执行下面几个操作：

- 显示初始化
- 代码块中的初始化
- 构造器初始化

## 对象的内存布局

对象的布局包括：

- 对象头
- 实例数据
- 对齐填充

#### 对象头

对象头包含两个部分，分别是**运行时元数据**和**类型指针**（如果是数组还要记录数组的长度）

**运行时元数据**包括 **哈希值、GC分代年龄、锁状态标志、线程持有的锁、偏向线程id、偏向时间戳**

**类型指针**指向类元数据InstanceKlass，确定该对象所属的类型。指向的其实是方法区中存放的类元信息

#### 实例数据

![image-20210724163157790](https://gitee.com/aruul/a-ru-img/raw/master/img/20210724163159.png)

#### 对齐填充

不是必须的，也没有特别的含义，仅仅起到占位符的作用

#### 小结

```java
public class Customer{
    int id = 1001;
    String name;
    Account account;
    
    {
        name = "aRuul"
    }
    
    public Customer(){
        account = new Account();
	}
    
}
----------------------------------------------

public Account{
    
}

----------------------------------------------
public MyTest{
    public static void main(String[] args){
        Customer customer = new Customer();
    }
}
```

![image-20210724164202328](https://gitee.com/aruul/a-ru-img/raw/master/img/20210724164203.png)

​																										【纠错】方法区里的Klass改成Class

# 直接内存

这里先略过【p106】

# 执行引擎

这里也先略过

# String Table

## 基本特性

- String 字符串，使用一对""

- String声明是final的，不可被继承

- String实现了Serializable接口：表示字符串是支持序列化的。实现了Comparable接口：表示string可以比较大小

- string在jdk8及以前内部定义了final **char[] value**用于存储字符串数据。JDK9时改为**byte[] value** 

- String 代表不可变的字串序列

  - 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。 
  - 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
  - 当调用string的replace（）方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。 
  - 通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。

  ```java
  class Demo{
      public static void test1() {
          // 字面量定义的方式，“abc”存储在字符串常量池中
          String s1 = "abc";
          String s2 = "abc";
          System.out.println(s1 == s2);  //true
          s1 = "hello";
          System.out.println(s1 == s2);  //false
          System.out.println(s1);		//hello
          System.out.println(s2);		//abc
          System.out.println("----------------");
      }
  
      public static void test2() {
          String s1 = "abc";
          String s2 = "abc";
          // 只要进行了修改，就会重新创建一个对象，这就是不可变性
          s2 += "def";
          System.out.println(s1);  //abc
          System.out.println(s2);  //abcdef
          System.out.println("----------------");
      }
  
      public static void test3() {
          String s1 = "abc";
          String s2 = s1.replace('a', 'm');
          System.out.println(s1);  //abc
          System.out.println(s2);  //mbc
      }
  }
  ```

- 字符串常量池中是不会存储相同内容的字符串的。

  > String的string Pool是一个固定大小的Hashtable，默认值大小长度是1009。如果放进string Pool的string非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用string.intern时性能会大幅下降。
  >
  > 使用-XX:StringTablesize可设置stringTable的长度
  >
  > 在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTableSize设置没有要求
  >
  > 在jdk7中，stringTable的长度默认值是60013，
  >
  > 在JDK8中，StringTable可以设置的最小值为1009

## String的内存分配

在Java语言中有8种基本数据类型和一种比较特殊的类型String。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种**常量池**的概念。

常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，string类型的常量池比较特殊。它的主要使用方法有两种。

- 直接使用双引号声明出来的String对象会直接存储在常量池中。比如：string info="aRuul"；
- 如果不是用双引号声明的string对象，可以使用String提供的intern()方法。

> Java 6及以前，字符串常量池存放在永久代
>
> Java 7中 oracle的工程师对字符串池的逻辑做了很大的改变，即**将字符串常量池的位置调整到Java堆内**
>
> 所有的字符串都保存在堆（Heap）中，和其他普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。
>
> 字符串常量池概念原本使用得比较多，但是这个改动使得我们有足够的理由让我们重新考虑在Java 7中使用String.intern()。
>
> Java8元空间，字符串常量在堆
>
> 详情请看 上面**方法区-------方法区的演进**这一小节

### 为什么StringTable从永久代调整到堆中

- 永久代的默认比较小
- 永久代垃圾回收频率低

## 字符串拼接

1. 常量与常量的拼接结果在常量池，原理是编译器优化
2. 常量池中不会存在相同内容的常量
3. 只要其中有一个是变量，结果就在堆中(指的是堆中非字符串常量池的区域)。
4. 变量拼接的原理是StringBuilder
5. 如果拼接的结果调用intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象的地址。

```java
public static void test1() {
    String s1 = "a" + "b" + "c";  // 得到 abc的常量池
    String s2 = "abc"; // abc存放在常量池，直接将常量池的地址返回
    /**
     * 最终java编译成.class，再执行.class
     */
    System.out.println(s1 == s2); // true，因为存放在字符串常量池
    System.out.println(s1.equals(s2)); // true
}

public static void test2() {
    String s1 = "javaEE";
    String s2 = "hadoop";
    String s3 = "javaEEhadoop";
    String s4 = "javaEE" + "hadoop";    
    String s5 = s1 + "hadoop";
    String s6 = "javaEE" + s2;
    String s7 = s1 + s2;
	//只要其中有一个是变量，结果就在堆中(指的是堆中非字符串常量池的区域),就是在堆中new一个。
    System.out.println(s3 == s4); // true
    System.out.println(s3 == s5); // false
    System.out.println(s3 == s6); // false
    System.out.println(s3 == s7); // false
    System.out.println(s5 == s6); // false
    System.out.println(s5 == s7); // false
    System.out.println(s6 == s7); // false
	//如果拼接的结果调用intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象的地址。
    String s8 = s6.intern();
    System.out.println(s3 == s8); // true
}

public static void test4(){
    final String s1 = "a";  //注意！这里的话就算常量
    final String s2 = "b";  //注意！这里的话就算常量
    String s3 = "ab";
    String s4 = s1 + s2;
    System.out,println(s3==s4); //true
}
```

### 字符串拼接底层细节

```java
public void test3(){
	String s1 = "a";
    String s2 = "b";
    String s3 = "ab";
    String s4 = s1 + s2;
    System.out,println(s3==s4); //false
}
```

当执行`String s4 = s1 + s2;`时，

**底层实际上是新建了一个StringBuilder,然后将两个值进行拼接，细节如下：**

1. **StringBuilder temp = new StringBuilder();**   【补充：在jdk5.0之前用的是StringBuffer】
2. **temp.append("a");**
3. **temp.append("b");**
4. **temp.toString();     //toString()方法约等于 new String("ab")**

> String字符串拼接效率
>
> - 通过StringBuilder的append()方式添加字符串的效率，要远远高于String的字符串拼接方法
>
> 
>
>   StringBuilder好处如下
>
> - StringBuilder的append的方式，自始至终只创建一个StringBuilder的对象
> - 对于字符串拼接的方式，还需要创建很多StringBuilder对象和 调用toString时候创建的String对象
> - 内存中由于创建了较多的StringBuilder和String对象，内存占用过大，如果进行GC那么将会耗费更多的时间

## intern()的使用

**如果不是用双引号声明的string对象，可以使用string提供的intern方法：intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。**

比如：

```java
String myInfo = new string("abc").intern();
```

也就是说，如果在任意字符串上调用string.intern方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true

```java
（"a"+"b"+"c"）.intern（）=="abc"
```

通俗点讲，Interned string就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池（String Intern Pool）

###  new String("ab")会创建几个对象

   两个

- 一个是new String()，会在堆空间创建
- 一个是"ab"，存放在字符串常量池中的对象

### new String("a") + new String("b") 会创建几个对象

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("a") + new String("b");
    }
}
---------------------字节码文件如下----------------------------------
 0 new #2 <java/lang/StringBuilder>
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init>>
 7 new #4 <java/lang/String>
10 dup
11 ldc #5 <a>
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>
22 dup
23 ldc #8 <b>
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>
31 invokevirtual #9 <java/lang/StringBuilder.toString>
34 astore_1
35 return
```

​	六个

- 对象1：new StringBuilder()

- 对象2：new String("a")

- 对象3：常量池的 a

- 对象4：new String("b")

- 对象5：常量池的 b

- 对象6：StringBuilder调用toString()方法，

  toString中会创建一个 new String("ab")

  【注意】**调用toString方法，不会在常量池中生成"ab"**

  ```java
  	//StringBuilder中的toString()方法
  	@Override
      public String toString() {
          return new String(value, 0, count);     //这个方法并不会在常量池中生成字符串
      }
  ```

### 面试题【难】

```java
package top.aruul;

/**
 * @author aRu
 * @date 2021/7/17 17:56
 */
public class Demo {
    public static void main(String[] args) {
        String s= new String("1");
        s.intern();        //这里并没有赋值给其他字符串
        String ss = s.intern();
        String s2 = "1";
        System.out.println(s==s2);  //false
        System.out.println(ss==s);  //false
        System.out.println(ss==s2); //true

        /* s3记录的变量地址为 new String("11")
         *执行完下面这行代码后，常量池中并没有 "11" */
        String s3 = new String("1") + new String("1");

        //【注意】这里的话常量池中并没有 "11",
        // 所以执行下面的intern方法的时候，会在字符串常量池中生成"11"
        //对于jdk6来说，会直接创建一个 "11" 在字符串常量池中
        //对于jdk7/jdk8 来说，把字符串常量池移到了堆中，而上一行代码在堆中创建了一个对象：【new String("11")】
        //为了节省空间，此时，字符串常量池中会创建一个指向【堆中 new String("11")】的地址
        s3.intern();

        //所以这里创建的 s4指向了字符串常量池中的 {一个指向【堆中 new String("11")】的地址}
        String s4 = "11";

        //所以这里对于jdk6：false
        //对于jdk7/jdk8: true
        System.out.println(s3==s4);
    }
}
```

### 面试题的拓展

```java
/* s3记录的变量地址为 new String("11")
         *执行完下面这行代码后，常量池中并没有 "11" */
String s3 = new String("1") + new String("1");
String s4 = "11";  //在字符串常量池中生成对象 "11"
s3.intern();     //这个操作没啥用，因为字符串常量池中已经有 "11"了，而且并没把结果返回给任何对象
System.out.println(s3==s4);  //jdk8: false
```

---

```java
String s = new String("a") + new String("b");  //相当于在堆中new String("ab")
String s2 = s.intern(); //在字符串常量池中生成一个指向堆中new String("ab")的地址，并返回给s2
//以下结果均在jdk8中
//"ab"在字符串常量池中是一个指向new String("ab")的地址，所以和s、s2是同一个东西
System.out.println(s=="ab");    //true
System.out.println(s2=="ab");	//true
```

---

```java
String x = "ab";
String s = new String("a") + new String("b");
String s2 = s.intern();
//以下结果均在jdk8中
System.out.println(s2==x);  //true
System.out.println(s==x);   //false
```

### 小总结

总结String的intern()的使用：

JDK1.6中，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，会把此**对象复制一份**，放入串池，并返回串池中的对象地址

JDK1.7起，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，则会把**对象的引用地址**复制一份，放入串池，并返回串池中的引用地址

##  G1中的String去重操作

**注意这里说的重复，指的是在堆中的数据，而不是常量池中的，因为常量池中的本身就不会重复**

> 许多大规模的Java应用的瓶颈在于内存，测试表明，在这些类型的应用里面，Java堆中存活的数据集合差不多25%是string对象。更进一步，这里面差不多一半string对象是重复的，重复的意思是说： stringl.equals（string2）= true。堆上存在重复的string对象必然是一种内存的浪费。



 **G1中的String去重操作:**

- 当垃圾收集器工作的时候，会访问堆上存活的对象。**对每一个访问的对象都会检查是否是候选的要去重的String对象**。
- 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的string对象。
- 使用一个hashtable来记录所有的被string对象使用的不重复的char数组。当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。
- 如果存在，String对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。
- 如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共享这个数组了。

【p134】

