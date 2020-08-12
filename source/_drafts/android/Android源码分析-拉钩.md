## startActivity 启动过程分析

[startActivity 启动过程分析](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1873)

==TODO==



### 总结

Activity 的启动在源码中的实现流程，这一过程主要涉及 3 个进程间的通信过程：

- 首先进程 A 通过 Binder 调用 AMS 的 startActivity 方法。
- 然后 AMS 通过一系列的计算构造目标 Intent，然后在 ActivityStack 与 ActivityStackSupervisor 中处理 Task 和 Activity 的入栈操作。
- 最后 AMS 通过 Binder 机制，调用目标进程中 ApplicationThread 的方法来创建并执行 Activity 生命周期方法，实际上 ApplicationThread 是 ActivityThread 的一个内部类，它的执行最终都调用到了 ActivityThread 中的相应方法。



## 底层剖析 Window 、Activity、 View 三者关系



[底层剖析 Window 、Activity、 View 三者关系](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1874)

==TODO==



### 总结

通过 setContentView 的流程，分析了 Activity、Window、View 之间的关系。整个过程 Activity 表面上参与度比较低，大部分 View 的添加操作都被封装到 Window 中实现。而 Activity 就相当于 Android 提供给开发人员的一个管理类，通过它能够更简单的实现 Window 和 View 的操作逻辑。

最后再简单列一下整个流程需要注意的点：

- 一个 Activity 中有一个 window，也就是 PhoneWindow 对象，在 PhoneWindow 中有一个 DecorView，在 setContentView 中会将 layout 填充到此 DecorView 中。
- 一个应用进程中只有一个 WindowManagerGlobal 对象，因为在 ViewRootImpl 中它是 static 静态类型。
- 每一个 PhoneWindow 对应一个 ViewRootImple 对象。
- WindowMangerGlobal 通过调用 ViewRootImpl 的 setView 方法，完成 window 的添加过程。
- ViewRootImpl 的 setView 方法中主要完成两件事情：View 渲染（requestLayout）以及接收触屏事件。



## Android 如何通过 View 进行渲染？



[Android 如何通过 View 进行渲染？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1875)

==TODO==



### 总结

主要介绍了 ViewRootImpl 是如何执行 View 的渲染操作的，其中核心方法在 performTraversals 方法中会按顺序执行 measure-layout-draw 操作。并顺带介绍了软件绘制和硬件加速的区别，最后介绍了 View 刷新的两种方式 Invalidate 和 postInvalidate。



## Android App 的安装过程



[Android App 的安装过程](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1876)

==TODO==



### 总结

首先Android点击APK安装，会启动PackageInstallerActivity，然后将APK信息传给PMS，PMS会做两件事，拷贝安装包和装载代码。在拷贝安装包过程中会开启Service来copyAPK,并且会检查apk安装路径，包的状态，最终以base.apk形式存在/data/app包名下。装载代码过程中，会继续解析apk，把清单文件内容存放于PMS，然后对apk进行签名校验，再执行dex2oat优化，安装成功后，更新应用设置权限，发送广播通知桌面显示APP图标，安装失败则删除安装包和各种缓存文件。



## 15 分钟彻底掌握 Handler



[15 分钟彻底掌握 Handler](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1877)

==TODO==



### 总结

- 应用启动是从 ActivityThread 的 main 开始的，先是执行了 Looper.prepare()，该方法先是 new 了一个 Looper 对象，在私有的构造方法中又创建了 MessageQueue 作为此 Looper 对象的成员变量，Looper 对象通过 ThreadLocal 绑定 MainThread 中；
- 当我们创建 Handler 子类对象时，在构造方法中通过 ThreadLocal 获取绑定的 Looper 对象，并获取此 Looper 对象的成员变量 MessageQueue 作为该 Handler 对象的成员变量；
- 在子线程中调用上一步创建的 Handler 子类对象的 sendMesage(msg) 方法时，在该方法中将 msg 的 target 属性设置为自己本身，同时调用成员变量 MessageQueue 对象的 enqueueMessag() 方法将 msg 放入 MessageQueue 中；
- 主线程创建好之后，会执行 Looper.loop() 方法，该方法中获取与线程绑定的 Looper 对象，继而获取该 Looper 对象的成员变量 MessageQueue 对象，并开启一个会阻塞（不占用资源）的死循环，只要 MessageQueue 中有 msg，就会获取该 msg，并执行 msg.target.dispatchMessage(msg) 方法（msg.target 即上一步引用的 handler 对象），此方法中调用了我们第二步创建 handler 子类对象时覆写的 handleMessage() 方法。