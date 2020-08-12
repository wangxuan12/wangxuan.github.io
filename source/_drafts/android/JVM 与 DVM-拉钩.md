## JVM内存

一个 HelloWorld.java 文件被 JVM 加载到内存中的过程：

1. HelloWorld.java 文件首先需要经过编译器编译，生成 HelloWorld.class 字节码文件。

2. Java 程序中访问HelloWorld这个类时，需要通过 ClassLoader(类加载器)将HelloWorld.class 加载到 JVM 的内存中。

3. JVM 中的内存可以划分为若干个不同的数据区域，主要分为：程序计数器、虚拟机栈、本地方法栈、堆、方法区。

<img src="D:\project\blog_image\Ciqah158SbeAQdCSAAEEJ1oi53Y731.png" style="zoom:48%; background-color: #fff;" />



### 程序计数器

“程序计数器”是虚拟机中一块较小的内存空间，主要用于记录当前线程执行的位置。



### 虚拟机栈



#### 栈帧

栈帧（Stack Frame）是用于支持虚拟机进行方法调用和方法执行的数据结构，每一个线程在执行某个方法时，都会为这个方法创建一个栈帧。



一个线程包含多个栈帧，而每个栈帧内部包含局部变量表、操作数栈、动态连接、返回地址等

<img src="D:\project\blog_image\Ciqah157F6SAJiwFAACto3B4htg907.png" style="zoom:50%; background-color:#fff;" />



##### 局部变量表

局部变量表是变量值的存储空间，我们调用方法时传递的参数，以及在方法内部创建的局部变量都保存在局部变量表中。



##### 操作数栈

当一个方法刚刚开始执行的时候，这个方法的操作数栈是空的。在方法执行的过程中，会有各种字节码指令被压入和弹出操作数栈



##### 动态链接



##### 返回地址

一般来说，方法正常退出时，调用者的 PC 计数值可以作为返回地址，栈帧中可能保存此计数值。而方法异常退出时，返回地址是通过异常处理器表确定的，栈帧中一般不会保存此部分信息。



### 本地方法栈

本地方法栈和上面介绍的虚拟栈基本相同，只不过是针对本地（native）方法。在开发中如果涉及 JNI 可能接触本地方法栈多一些，在有些虚拟机的实现中已经将两个合二为一了（比如HotSpot）。



### 堆

Java 堆（Heap）是 JVM 所管理的内存中最大的一块，该区域唯一目的就是存放对象实例，几乎所有对象的实例都在堆里面分配

![](D:\project\blog_image\Cgq2xl57GDCAVkHYAABVMCBYUEE302.png)



### 方法区

方法区（Method Area）也是 JVM 规范里规定的一块运行时数据区。方法区主要是存储已经被 JVM 加载的类信息（版本、字段、方法、接口）、常量、静态变量、即时编译器编译后的代码和数据。该区域同堆一样，也是被各个线程共享的内存区域。



### 总结

![](D:\project\blog_image\Ciqah157GD2AYLFtAADxheNgCA0454.png)



## GC回收机制与分代回收策略



### 可达性分析



#### GC Root对象

1. Java 虚拟机栈（局部变量表）中的引用的对象。
2. 方法区中静态引用指向的对象。
3. 仍处于存活状态中的线程对象。
4. Native 方法中 JNI 引用的对象。



### 什么时候回收

1. Allocation Failure：内存分配失败时会触发一次GC
2. System.gc()：手动调用这个方法来请求一次 GC



### 如何回收 垃圾回收算法



#### 标记清除算法（Mark and Sweep GC）

算法执行过程：

1. **Mark 标记阶段**：找到内存中的所有 GC Root 对象，只要是和 GC Root 对象直接或者间接相连则标记为灰色（也就是存活对象），否则标记为黑色（也就是垃圾对象）。
2. **Sweep 清除阶段**：当遍历完所有的 GC Root 之后，则将标记为垃圾的对象直接清除。



**优点**：实现简单，不需要将对象进行移动。
**缺点**：这个算法需要中断进程内其他组件的执行（stop the world），**并且可能产生内存碎片，提高了垃圾回收的频率**。



#### 复制算法（Copying）

将现有的内存空间分为两快，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中。之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收。



**优点**：按顺序分配内存即可，**实现简单**、运行高效，**不用考虑内存碎片**。
**缺点**：可用的内存大小缩小为原来的一半，对象存活率高时会频繁进行复制。



#### 标记-压缩算法 (Mark-Compact)

需要先从根节点开始对所有可达对象做一次标记，之后，它并不简单地清理未标记的对象，而是将所有的存活对象压缩到内存的一端。最后，清理边界外所有的空间。因此标记压缩也分两步完成：

1. **Mark 标记阶段**：找到内存中的所有 GC Root 对象，只要是和 GC Root 对象直接或者间接相连则标记为灰色（也就是存活对象），否则标记为黑色（也就是垃圾对象）。
2. **Compact 压缩阶段**：将剩余存活对象按顺序压缩到内存的某一端。



**优点**：这种方法**既避免了碎片的产生，又不需要两块相同的内存空间**，因此，其性价比比较高。
**缺点**：所谓压缩操作，仍需要**进行局部对象移动，所以一定程度上还是降低了效率**。



### JVM分代回收策略

Java 虚拟机根据对象存活的周期不同，把堆内存划分为几块，**一般分为新生代、老年代**，这就是 JVM 的内存分代策略。*注意: 在 HotSpot 中除了新生代和老年代，还有永久代*



#### 新生代

新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低。所以一般采用的 GC 回收算法是**复制算法**。

新生代又可以继续细分为 3 部分：Eden、Survivor0（简称 S0）、Survivor1（简称S1）。



#### 老年代

一个对象如果在新生代存活了足够长的时间而没有被清理掉，则会被复制到老年代。老年代的内存大小一般比新生代大，能存放更多的对象。如果对象比较大（比如长字符串或者大数组），并且新生代的剩余空间不足，则这个大对象会直接被分配到老年代上。

老年代因为对象的生命周期较长，不需要过多的复制操作，所以一般采用标记压缩的回收算法。



#### GC Log 分析



#### 引用

![](D:\project\blog_image\Ciqah158ltqAHyEHAACoLz2II_g092.png)



## 字节码层面分析 class 类文件结构



### 上帝视角看 class 文件

如果从纵观的角度来看 class 文件，class 文件里只有两种数据结构：无符号数和表。



**无符号数**：属于基本的数据类型，以 u1、u2、u4、u8 来分别代表 1 个字节、2 个字节、4 个字节和 8 个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者字符串（UTF-8 编码）。

**表**：表是由多个无符号数或者其他表作为数据项构成的复合数据类型，class文件中所有的表都以“_info”结尾。其实，整个 Class 文件本质上就是一张表。



### class 文件结构

![](D:\project\blog_image\Cgq2xl6Qc1OACWWHAADJjiKcHuI014.png)



使用`javap -v TestClass.class`查看class文件



## 编译插桩操纵字节码，实现不可能完成的任务



### 编译插桩

顾名思义，所谓编译插桩就是在代码编译期间修改已有的代码或者生成新代码。



![](D:\project\blog_image\Ciqah16FrD2AcPLbAABSfiJwMz0698.png)

从上图可以看出，我们可以在 1、2 两处对代码进行改造。

1. 在 .java 文件编译成 .class 文件时，APT、AndroidAnnotation 等就是在此处触发代码生成。
2. 在 .class 文件进一步优化成 .dex 文件时，也就是直接操作字节码文件



一般情况下，我们经常会使用编译插桩实现如下几种功能：

- 日志埋点；

- 性能监控；

- 动态权限控制；

- 业务逻辑跳转时，校验是否已经登录；

- 甚至是代码调试等。



### 插桩工具

目前市面上主要流行两种实现编译插桩的方式：



#### AspectJ

AspectJ 是老牌 AOP（Aspect-Oriented Programming）框架，其主要优势是成熟稳定，使用者也不需要对字节码文件有深入的理解。



#### ASM

通过 ASM 可以修改现有的字节码文件，也可以动态生成字节码文件，并且它是一款完全以字节码层面来操纵字节码并分析字节码的框架（此处可以联想一下写汇编代码时的酸爽）。



### 使用 ASM 来实现简单的编译插桩效果

通过插桩实现在每一个 Activity 打开时输出相应的 log 日志。



#### 实现思路

1. *遍历项目中所有的 .class 文件。*

Android Studio 使用 Gradle 编译项目中的 .java 文件，并且从 Gradle1.5.0 之后，我们可以自己定义 Transform，来获取所有 .class 文件引用。但是 Transform 的使用需要依赖 Gradle Plugin。因此我们**第一步需要创建一个单独的 Gradle Plugin，并在 Gradle Plugin 中使用自定义 Transform 找出所有的 .class 文件。**



2. *遍历到目标 .class 文件 （Activity）之后，通过 ASM 动态注入需要被插入的字节码*

如果第一步进行顺利，我们可以找出所有的 .class 文件。接下来就需要**过滤出目标 Activity 文件，并在目标 Activity 文件的 onCreate 方法中，通过 ASM 插入相应的 log 日志字节码。**



#### 具体实现

==TODO==



## 深入理解 ClassLoader 的加载机制



### Java 中的类何时被加载器加载

通常情况下,Java 程序中的 .class 文件会在以下 2 种情况下被 ClassLoader 主动加载到内存中：

1. 调用类构造器
2. 调用类中的静态（static）变量或者静态方法



### Java 中 ClassLoader

1. 启动类加载器（Bootstrap ClassLoader）：这个加载器是Java虚拟机实现的一部分，不是Java语言实现的，一般是C++实现的，它负责加载Java的基础类，主要是<JAVA_HOME>/lib/rt.jar，我们日常用的Java类库比如String、ArrayList等都位于该包内。
2. 扩展类加载器（Extension ClassLoader）：这个加载器的实现类是sun.misc.Laun-cher$ExtClassLoader，它负责加载Java的一些扩展类，一般是<JAVA_HOME>/lib/ext目录中的jar包。 （JDK 1.9 之后，改名为 PlatformClassLoader）
3. 应用程序类加载器（Application ClassLoader）：这个加载器的实现类是sun.misc. Launcher$AppClassLoader，它负责加载应用程序的类，包括自己写的和引入的第三方法类库，即所有在类路径中指定的类。



#### 双亲委派模式（Parents Delegation Model）

当类加载器收到加载类或资源的请求时，通常都是先委托给父类加载器加载，也就是说，只有当父类加载器找不到指定类或资源时，自身才会执行实际的类加载过程。



为什么要先让父ClassLoader去加载呢？这样，可以避免Java类库被覆盖的问题。

![](D:\project\blog_image\Cgq2xl6MQCeAQezSAAQYyFDklrg999.png)



#### 自定义 ClassLoader

1. 自定义一个类继承抽象类 ClassLoader。
2. 重写 findClass 方法。
3. 在 findClass 中，调用 defineClass 方法将字节码转换成 Class 对象，并返回。



### Android 中的 ClassLoader

在 Android 虚拟机里是无法直接运行 .class 文件的，Android 会将所有的 .class 文件转换成一个 .dex 文件，并且 Android 将加载 .dex 文件的实现封装在 BaseDexClassLoader 中，而我们一般只使用它的两个子类：PathClassLoader 和 DexClassLoader。



#### PathClassLoader

PathClassLoader 用来加载系统 apk 和被安装到手机中的 apk 内的 dex 文件。



#### DexClassLoader

对比 PathClassLoader 只能加载已经安装应用的 dex 或 apk 文件，DexClassLoader 则没有此限制，可以从 SD 卡上加载包含 class.dex 的 .jar 和 .apk 文件，这也是插件化和热修复的基础，在不需要安装应用的情况下，完成需要使用的 dex 的加载。



#### 实现

==TODO==



### 总结

- ClassLoader 就是用来加载 class 文件的，不管是 jar 中还是 dex 中的 class。

- Java 中的 ClassLoader 通过双亲委托来加载各自指定路径下的 class 文件。

- 可以自定义 ClassLoader，一般覆盖 findClass() 方法，不建议重写 loadClass 方法。

- Android 中常用的两种 ClassLoader 分别为：PathClassLoader 和 DexClassLoader。



## Class 对象在执行引擎中的初始化过程



### 装载

装载是指 Java 虚拟机查找 .class 文件并生成字节流，然后根据字节流创建 java.lang.Class 对象的过程。

1. ClassLoader 通过一个类的全限定名（包名 + 类名）来查找 .class 文件，并生成二进制字节流
2. 把 .class 文件的各个部分分别解析（parse）为 JVM 内部特定的数据结构，并存储在方法区。
3. 在内存中创建一个 java.lang.Class 类型的对象



#### 加载时机

- 隐式装载：在程序运行过程中，当碰到通过 new 等方式生成对象时，系统会隐式调用 ClassLoader 去装载对应的 class 到内存中；
- 显示装载：在编写源代码时，主动调用 Class.forName() 等方法也会进行 class 装载操作，这种方式通常称为显示装载。



### 链接

链接过程分为 3 步：验证、准备、解析。



#### 验证

验证是链接的第一步，目的是为了确保 .class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危及虚拟机本身的安全。



因为工程师是可以对编译之后的 class 字节码文件进行篡改的。所以这也是为什么我们在项目中经常会使用混淆，甚至是使用一些三方的加固软件，来保证我们所编写的代码的安全性。  



#### 准备

准备是链接的第 2 步，这一阶段的主要目的是为类中的静态变量分配内存。

在准备阶段，JVM 会为 value 分配内存，并将其设置为 0。而真正的值 100 是在初始化阶段设置。



#### 解析

解析是链接的最后一步，这一阶段的任务是把常量池中的符号引用转换为直接引用，也就是具体的内存地址。



### 初始化

这是 class 加载的最后一步，这一阶段是执行类构造器<clinit>方法的过程，并真正初始化类变量。



#### 初始化的时机

1. 虚拟机启动时，初始化包含 main 方法的主类；

2. 遇到 new 指令创建对象实例时，如果目标对象类没有被初始化则进行初始化操作；
3. 当遇到访问静态方法或者静态字段的指令时，如果目标对象类没有被初始化则进行初始化操作；
4. **子类的初始化过程如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化；**
5. 使用反射 API 进行反射调用时，如果类没有进行过初始化则需要先触发其初始化；
6. 第一次调用 java.lang.invoke.MethodHandle 实例时，需要初始化 MethodHandle 指向方法所在的类。



#### 初始化类变量

在初始化阶段，只会初始化与类相关的静态赋值语句和静态语句，也就是有 static 关键字修饰的信息，而没有 static 修饰的语句块在实例化对象的时候才会执行。



#### 被动引用

上述的 6 种情况在 JVM 中被称为主动引用，除此 6 种情况之外所有引用类的方式都被称为被动引用。被动引用并不会触发 class 的初始化。



举:chestnut:：

对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过子类 Child 来引用父类 Parent 中定义的静态字段，只会触发父类 Parent 的初始化而不会触发子类 Child 的初始化。



#### class 初始化和对象的创建顺序

关于 class 的初始化还有一点经常会在面试中被提及，那就是对象的初始化顺序。当我们在代码中使用 new 创建一个类的实例对象时，类中的静态代码块、非静态代码块、构造函数之间的执行顺序是怎样的。



**总结一下对象的初始化顺序如下：**



静态变量/静态代码块 -> 普通代码块 -> 构造函数

1. 父类静态变量和静态代码块；
2. 子类静态变量和静态代码块；
3. 父类普通成员变量和普通代码块；
4. 父类的构造函数；
5. 子类普通成员变量和普通代码块；
6. 子类的构造函数。



### 总结：

.class 文件被加载到内存中所经过的详细过程，主要分 3 大步：装载、链接、初始化。其中链接中又包含验证、准备、解析 3 小步。

1. 装载：指查找字节流，并根据此字节流创建类的过程。装载过程成功的标志就是在方法区中成功创建了类所对应的 Class 对象。
2. 链接：指验证创建的类，并将其解析到 JVM 中使之能够被 JVM 执行。
3. 初始化：则是将标记为 static 的字段进行赋值，并且执行 static 标记的代码语句 。



## Java 内存模型与线程



### 为什么有 Java 内存模型

在执行任务时，CPU 会先将运算所需要使用到的数据复制到高速缓存中，让运算能够快速进行，当运算完成之后，再将缓存中的结果刷回（flush back）主内存，这样 CPU 就不用等待主内存的读写操作了。



每个处理器都有自己的高速缓存，同时又共同操作同一块主内存，当多个处理器同时操作主内存时，可能导致数据不一致，这就是是**缓存一致性**问题。



如果我们任由 CPU 优化或者编译器指令重排，那我们编写的 Java 代码最终执行效果可能会极大的出乎意料。为了解决这个问题，让 Java 代码在不同硬件、不同操作系统中，输出的结果达到一致，Java 虚拟机规范提出了一套机制——Java 内存模型。



### 什么是内存模型

内存模型是一套共享内存系统中多线程读写操作行为的规范，这套规范屏蔽了底层各种硬件和操作系统的内存访问差异，解决了 CPU 多级缓存、CPU 优化、指令重排等导致的内存访问问题，从而保证 Java 程序（尤其是多线程程序）在各种平台下对内存的访问效果一致。



在 Java 内存模型中，我们统一用工作内存（working memory）来当作 CPU 中寄存器或高速缓存的抽象。线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有工作内存（类比 CPU 中的寄存器或者高速缓存），本地工作内存中存储了该线程读/写共享变量的副本。



在这套规范中，有一个非常重要的规则——happens-before。



### happens-before 先行发生原则

1. **程序次序规则**：b 对 a 的结果有依赖，则不会发生指令重排优化。
2. **锁定规则**：无论是在单线程环境还是多线程环境，一个锁如果处于被锁定状态，那么必须先执行 unlock 操作后才能进行 lock 操作。
3. **volatile变量规则**：volatile 保证了线程可见性。通俗讲就是如果一个线程先写了一个 volatile 变量，然后另外一个线程去读这个变量，那么这个写操作一定是 happens-before 读操作的。
4. **传递规则**：如果操作 A happens-before 操作 B，而操作 B happens-before 操作 C，则操作 A 一定 happens-before 操作 C。 
5. 线程启动规则：Thread 对象的 start() 方法先行发生于此线程的每一个动作。假定线程 A 在执行过程中，通过执行 ThreadB.start() 来启动线程 B，那么线程 A 对共享变量的修改在线程 B 开始执行后确保对线程 B 可见。
6. 线程中断规则：对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测，直到中断事件的发生。
7. 线程终结规则：线程中所有的操作都发生在线程的终止检测之前，我们可以通过 Thread.join() 方法结束、Thread.isAlive() 的返回值等方法检测线程是否终止执行。假定线程 A 在执行的过程中，通过调用 ThreadB.join() 等待线程 B 终止，那么线程 B 在终止之前对共享变量的修改在线程 A 等待返回后可见。
8. 对象终结规则：一个对象的初始化完成发生在它的 finalize() 方法开始前。



### Java 内存模型应用

happens-before 原则非常重要，它是判断数据是否存在竞争、线程是否安全的主要依据，根据这个原则，我们能够解决在并发环境下操作之间是否可能存在冲突的所有问题。在此基础上，我们可以通过 Java 提供的一系列关键字，将我们自己实现的多线程操作“happens-before 化”。

- 使用 volatile 修饰 value

![](D:\project\blog_image\Ciqah16VdruAD-KcAABUSaeaBRs555.png)

- 使用synchronized关键字修饰操作
- 使用Lock加锁操作



## 总结

1. Java 内存模型的来源：主要是因为 CPU 缓存和指令重排等优化会造成多线程程序结果不可控。
2. Java 内存模型是什么：本质上它就是一套规范，在这套规范中有一条最重要的 happens-before 原则。
3. 最后介绍了 Java 内存模型的使用，其中简单介绍了两种方式：volatile 和 synchronized。



## 既生 Synchronized，何生 ReentrantLock



### synchronized

编译而成的字节码中会包含 monitorenter 和 monitorexit 这两个字节码指令。



字节码中有 1 个 monitorenter 和 2 个 monitorexit。这是因为虚拟机需要保证当异常发生时也能释放锁。因此 2 个 monitorexit 一个是代码正常执行结束后释放锁，一个是在代码执行异常时释放锁。



关于 monitorenter 和 monitorexit，可以理解为一把具体的锁。在这个锁中保存着两个比较重要的属性：计数器和指针。

1. 计数器代表当前线程一共访问了几次这把锁；
2. 指针指向持有这把锁的线程。



### ReentrantLock

ReentrantLock 与 synchronized 不同，当异常发生时 synchronized 会自动释放锁，但是 ReentrantLock 并不会自动释放锁。因此好的方式是将 unlock 操作放在 finally 代码块中，保证任何时候锁都能够被正常释放掉。 



#### 读写锁（ReentrantReadWriteLock）

在写操作时，读操作会被阻塞。在读操作时，写操作会被阻塞。但是允许多个同时读，不允许多个同时写。



## Java 线程优化 偏向锁，轻量级锁、重量级锁



[Java 线程优化 偏向锁，轻量级锁、重量级锁](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1863)

==TODO==



## 深入理解 AQS 和 CAS 原理



[深入理解 AQS 和 CAS 原理](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1864)

==TODO==



### 总结

本课时主要介绍了 Java 中锁的几种状态，其中偏向锁和轻量级锁都是通过自旋等技术避免真正的加锁，而重量级锁才是获取锁和释放锁，重量级锁通过对象内部的监视器（ObjectMonitor）实现，其本质是依赖于底层操作系统的 Mutex Lock 实现，操作系统实现线程之间的切换需要从用户态到内核态的切换，成本非常高。实际上Java对锁的优化还有”锁消除“，但是”锁消除“是基于Java对象逃逸分析的



### 总结

总体来说，AQS 是一套框架，在框架内部已经封装好了大部分同步需要的逻辑，在 AQS 内部维护了一个状态指示器 state 和一个等待队列 Node，而通过 state 的操作又分为两种：独占式和共享式，这就导致 AQS 有两种不同的实现：独占锁（ReentrantLock 等）和分享锁（CountDownLatch、读写锁等）。本课时主要从独占锁的角度深入分析了 AQS 的加锁和释放锁的流程。

理解 AQS 的原理对理解 JUC 包中其他组件实现的基础有帮助，并且理解其原理才能更好的扩展其功能。上层开发人员可以基于此框架基础上进行扩展实现适合不同场景、不同功能的锁。其中几个有可能需要子类同步器实现的方法如下。

- lock()。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回 true，失败则返回 false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回 true，失败则返回 false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回 true，否则返回 false。



## 线程池之刨根问底



[线程池之刨根问底](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1865)

==TODO==



## DVM 以及 ART 是如何对 JVM 进行优化的？



[DVM 以及 ART 是如何对 JVM 进行优化的？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1866)

==TODO==



分别通过 javac 命令将它们编译为 .class 文件。

javac Dex1.java  ->  Dex1.class
javac Dex2.java  ->  Dex2.class



然后通过以下命令将 Dex1.class 和 Dex2.class 打包到一个 jar 文件中。

jar cvf AllDex.jar Dex1.class Dex2.class



上述命令会在当前目录生成一个 AllDex.jar 文件。

最后使用 dx 命令将 AllDex.jar 进行优化，并生成 AllDex.dex 文件。



dx --dex --output AllDex.dex AllDex.jars



命令结束后，会在当前目录生成 AllDex.dex 文件。

正常情况下，我们无法通过反编译工具查看其源码，但是可以通过 Android SDK 中的工具 dexdump 查看其字节码：



dexdump -d -l plain AllDex.dex



