作者：王宇龙
链接：https://www.zhihu.com/question/30922650/answer/49997182
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



Canvas是Java层构建的数据结构，是给View用的画布。ViewGroup会把自己的Canvas拆分给子View。View会在onDraw方法里将图形数据绘制在它获得的Canvas上。

而Surface是Native层构建的数据结构，是给SurfaceFlinger用的画布。它是直接被用来绘制到屏幕上的数据结构。

开发者一般所用的View都是在Canvas进行绘制，然后最顶层的View（通常是DecorView)的Canvas的数据信息会转换到一个Surface上。SurfaceFlinger会将各个应用窗口的Surface进行合成，然后绘制到屏幕上（实际上是一个Buffer，但一般开发者不用考虑这些，所以省略一些概念）。

那为什么会有一个SurfaceView呢？

这是因为View的测量（Measure），布局（Layout）以及绘制（Draw）的计算量比较大。计算完以后再从Canvas转换成Surface中数据，然后再绘制屏幕，这个流程比较耗时。对于常规的UI绘制不会有什么问题，但是像Camera的预览以及视频的播放这样的应用场景来说就不可接受了。
SurfaceView就是为了解决这个问题。SurfaceView内容不再是绘制在Canvas上，而是直接绘制在其持有的一个Surface上。由于省去了很多步骤，其绘制性能大大提高。而SurfaceView本身只是用来控制这个Surface的大小和位置而已。

不知道这样解释题主能明白否。