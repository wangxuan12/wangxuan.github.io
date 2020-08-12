## Android 是如何通过 Activity 进行交互的？



### 总结

总结了几个使用 startActivity 时可能会遇到的问题：

- taskAffinity 实现任务栈的调配；
- 通过 Binder 传递数据的限制；
- 多进程应用可能会造成的问题；
- 后台启动 Activity 的限制。

==TODO==



## 彻底掌握 Android touch 事件分发时序



[彻底掌握 Android touch 事件分发时序](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1868)

==TODO==



### 总结

 dispatchTouchEvent 的事件的流程机制，这一过程主要分 3 部分：

- 判断是否需要拦截 —> 主要是根据 onInterceptTouchEvent 方法的返回值来决定是否拦截；
- 在 DOWN 事件中将 touch 事件分发给子 View —> 这一过程如果有子 View 捕获消费了 touch 事件，会对 mFirstTouchTarget 进行赋值；
- 最后一步，DOWN、MOVE、UP 事件都会根据 mFirstTouchTarget 是否为 null，决定是自己处理 touch 事件，还是再次分发给子 View。

然后介绍了整个事件分发中的几个特殊的点。

- DOWN 事件的特殊之处：事件的起点；决定后续事件由谁来消费处理；
- mFirstTouchTarget 的作用：记录捕获消费 touch 事件的 View，是一个链表结构；
- CANCEL 事件的触发场景：当父视图先不拦截，然后在 MOVE 事件中重新拦截，此时子 View 会接收到一个 CANCEL 事件。



## Android 如何自定义 View？



[Android 如何自定义 View？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1869)

==TODO==



### 总结

本课时介绍了自定义 View 的几个知识点，要自定义一个控件主要包含几个方法。
onDraw：主要负责绘制 UI 元素；
onMeasure：主要负责测量自定义控件具体显示的宽高；
onLayout：主要是在自定义 ViewGroup 中复写，并实现子 View 的显示位置，并在其中介绍了自定义属性的使用方法。

[代码仓库 课时15]([https://github.com/McoyJiang/LagouAndroidShare/tree/master/course15_%E8%87%AA%E5%AE%9A%E4%B9%89View/LagouCustomizedView/app/src/main/java/material/danny_jiang/com/lagoucustomizedview/views](https://github.com/McoyJiang/LagouAndroidShare/tree/master/course15_自定义View/LagouCustomizedView/app/src/main/java/material/danny_jiang/com/lagoucustomizedview/views))



## 为什么 RecyclerView 可以完美替代 Listview？



[为什么 RecyclerView 可以完美替代 Listview？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1870)

==TODO==



### 总结

分析了 Android RecyclerView 源码中的 2 块核心实现：

- RecyclerView 是如何经过测量、布局，最终绘制到屏幕上，其中大部分工作是通过委托给 LayoutManager 来实现的。
- RecyclerView 的缓存复用机制，主要是通过内部类 Recycler 来实现。

谷歌 Android 团队对 RecyclerView 做了很多优化，导致 RecyclerView 最终的代码极其庞大。这也是为什么当 RecyclerView 出现问题的时候，排查问题的复杂度相对较高。理解 RecyclerView 的源码实现，有助于我们快速定位问题原因、拓展 RecyclerView 功能、提高分析 RecyclerView 性能问题的能力。



## Android OkHttp 全面详解



[Android OkHttp 全面详解](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1871)

==TODO==



### 总结

OkHttp 的源码实现：

- 首先 OkHttp 内部是一个门户模式，所有的下发工作都是通过一个门户 Dispatcher 来进行分发。
- 然后在网络请求阶段通过责任链模式，链式的调用各个拦截器的 intercept 方法。其中我重点介绍了 2 个比较重要的拦截器：CacheInterceptor 和 CallServerInterceptor。它们分别用来做请求缓存和执行网络请求操作。
- 最后我们在理解源码实现的基础上，对 OkHttp 的功能进行了一些扩展，实现了网络请求进度的实现。



## Android Bitmap 全面详解



[Android Bitmap 全面详解](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=67#/detail/pc?id=1872)

==TODO==



### 总结

Bitmap 开发中的几个常见问题：

- 一张图片被加载成 Bitmap 后实际占用内存是多大。
- 通过 Options.inBitmap 可以实现 Bitmap 的复用，但是有一定的限制。
- 当界面需要展示多张图片，尤其是在列表视图中，可以考虑使用 Bitmap 缓存。
- 如果需要展示的图片过大，可以考虑使用分片加载的策略

