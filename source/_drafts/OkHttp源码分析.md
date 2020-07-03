# OkHttp源码分析

>OkHttp的核心：
>
>1. `OkHttpClient`：配置中心
>2. `Dispather`：进行线程的调度
>3. `RealCall`：进行实际的请求执行
>4. `RealInterceptorChain`：使用责任链模式，逐级调用拦截器，最终返回`Response`

## 基本使用

直接盗用了OkHttp官方的:chestnut:

```kotlin
val client = OkHttpClient()
val request = Request.Builder()
		.url(url)
      .build()
client.newCall(request).execute()
```



## OKHttp 请求的大致流程

先上图：

<img src="https://upload-images.jianshu.io/upload_images/1916953-fc6439af2bfefddc.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/685/format/webp" style="zoom:70%;" />

使用`OkHttp`进行请求的时候可以使用`execute` 或者 `enqueue`，这两者之间的区别就是`execute`会直接就进行请求，而`enqueue`会经过`Dispather`进行调度后在请求。



那首先先看看简单一点的，也就是下面这个:chestnut:

`client.newCall(request).execute()`



`newCall`需要一个Request的参数，而其返回值是一个`Call`：

```kotlin
override fun newCall(request: Request): Call = RealCall(this, request, forWebSocket = false)
```

可以看到里面创建的是一个`RealCall`对象，然后返回。



然后得到`RealCall`后，就会调用它的`execute`方法来获得响应结果：

```kotlin
  override fun execute(): Response {
    synchronized(this) {
      check(!executed) { "Already Executed" }
      executed = true
    }
    timeout.enter()
    callStart()
    try {
      //将RealCall添加进执行队列
      client.dispatcher.executed(this)
      //经过拦截器逐级执行，最后返回Response结果
      return getResponseWithInterceptorChain()
    } finally {
      client.dispatcher.finished(this)
    }
  }
```



`dispatcher`的`execute`：

```kotlin
  @Synchronized internal fun executed(call: RealCall) {
    //添加进执行队列
    runningSyncCalls.add(call)
  }
```



这就是一个OkHttp进行请求的大致流程了。

**总结一下就是：当使用`client.newCall(request)` 时我们创建了一个`ReallCall`对象，紧接着调用了`RealCall`的`execute()`方法，把`RealCall`添加进执行队列中，最后通过`getResponseWithInterceptorChain()`方法（其内部添加了一个个拦截器，然后采用责任链模式，逐级执行）最终得到了`Response`返回结果。**



## Dispatcher 调度器

首先来看看Dispatcher的一些属性：

```kotlin
//所能允许的最大请求数
@get:Synchronized var maxRequests = 64
//对于每个host所能允许的最大请求数
@get:Synchronized var maxRequestsPerHost = 5

@get:Synchronized
@get:JvmName("executorService") val executorService: ExecutorService
get() {
    if (executorServiceOrNull == null) {
        executorServiceOrNull = ThreadPoolExecutor(0, Int.MAX_VALUE, 60, TimeUnit.SECONDS,
                SynchronousQueue(), threadFactory("$okHttpName Dispatcher", false))
    }
    return executorServiceOrNull!!
}

//待处理的异步请求队列
private val readyAsyncCalls = ArrayDeque<AsyncCall>()

//正在执行的异步请求队列
private val runningAsyncCalls = ArrayDeque<AsyncCall>()

//正在执行的同步请求队列
private val runningSyncCalls = ArrayDeque<RealCall>()
```



之前只说了使用`execute`的情况，接下来就来说说`enqueue`的。

使用`enqueue`的时候会进行线程切换到后台，进行请求，使用方式：

`client.newCall(request).enqueue(callBack)`



`enqueue`会将传进去的`CallBack`参数包装成一个`AsyncCall`：

```kotlin
  override fun enqueue(responseCallback: Callback) {
    synchronized(this) {
      check(!executed) { "Already Executed" }
      executed = true
    }
    callStart()
    client.dispatcher.enqueue(AsyncCall(responseCallback))
  }
```

而且可以看到这里使用的是`dispatcher`的`enqueue`方法，不过我们先不管它。先看看`AsyncCall`是个啥玩意，看名字是一个异步的`Call`。

```kotlin
internal inner class AsyncCall(
    private val responseCallback: Callback
  ) : Runnable
```

果不其然，它实现了`Runnable`接口，那自然而然的也就会有`run`方法了。



那就来看看`run`方法都干了些啥

```kotlin
    override fun run() {
      threadName("OkHttp ${redactedUrl()}") {
        var signalledCallback = false
        timeout.enter()
        try {
          //经过拦截器逐级执行，最后返回Response结果
          val response = getResponseWithInterceptorChain()
          signalledCallback = true
          //请求成功的回调结果
          responseCallback.onResponse(this@RealCall, response)
        } catch (e: IOException) {
          if (signalledCallback) {
            // Do not signal the callback twice!
            Platform.get().log("Callback failure for ${toLoggableString()}", Platform.INFO, e)
          } else {
            //请求失败的回调结果
            responseCallback.onFailure(this@RealCall, e)
          }
        } catch (t: Throwable) {
          cancel()
          if (!signalledCallback) {
            val canceledException = IOException("canceled due to $t")
            canceledException.addSuppressed(t)
            responseCallback.onFailure(this@RealCall, canceledException)
          }
          throw t
        } finally {
          client.dispatcher.finished(this)
        }
      }
    }
  }
```

这个方法有点长，不过我们只关心最主要的，其中`getResponseWithInterceptorChain()`，和`execute()`中的一样返回`response`。接下来`responseCallback.onResponse`和`responseCallback.onFailure`这两个方法实在太熟悉了，这不就是我们天天写的`CallBack`回调！



这下`enqueue`的流程也结束了。不过发觉没有，`enqueue`比`execute`还少了一步。那就是并没有看到将其加入到**执行队列中**，为什么`execute`需要加入到执行队列而`enqueue`不需要呢？难道有什么py交易？当然不是，还记得上面提到的`dispatcher`么，没错其实加入队列操作`dispatcher`做了，那到底是不是这样呢？那就让我们那看看把。毕竟真相只有一个。



回到之前的`client.dispatcher.enqueue(AsyncCall(responseCallback))`方法，让我们看看`dispatcher.enqueue`有什么秘密。

```kotlin
  internal fun enqueue(call: AsyncCall) {
    synchronized(this) {
      //将call加入待处理队列
      readyAsyncCalls.add(call)

      // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
      // the same host.
      if (!call.call.forWebSocket) {
        val existingCall = findExistingCallWithHost(call.host)
        if (existingCall != null) call.reuseCallsPerHostFrom(existingCall)
      }
    }
    //去执行call
    promoteAndExecute()
  }
```

可以看到它主要做了两部分工作，先将其加入待处理队列，然后去执行。至于为什么要先加入到待处理队列，我们之后再说。那接下来就来看看`promoteAndExecute`做了什么。

```kotlin
  private fun promoteAndExecute(): Boolean {
    this.assertThreadDoesntHoldLock()

    val executableCalls = mutableListOf<AsyncCall>()
    val isRunning: Boolean
    synchronized(this) {
      val i = readyAsyncCalls.iterator()
      while (i.hasNext()) {
        val asyncCall = i.next()
			//正在执行请求数量大于了所允许的最大请求数，那么就不将其从待处理队列移到执行队列。
        if (runningAsyncCalls.size >= this.maxRequests) break // Max capacity.
        //同样，这里是每个host的最大请求不能大于所允许的最大值。
        if (asyncCall.callsPerHost.get() >= this.maxRequestsPerHost) continue // Host max capacity.

        i.remove()
        asyncCall.callsPerHost.incrementAndGet()
        executableCalls.add(asyncCall)
        //将call加入到执行队列
        runningAsyncCalls.add(asyncCall)
      }
      isRunning = runningCallsCount() > 0
    }
```

可以看到最后也是加入到了执行队列中，这就将`execute`和`enqueue`对应起来了。



下面来通过一个时序图，再来对比一下`execute`和`enqueue`两种实现方法的不同：

<img src="https://user-gold-cdn.xitu.io/2018/10/19/1668c5c04f04eab2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" style="zoom:70%; background-color: #f4f5f5;" />

## OkHttpClient 配置中心

对于OkHttpClient我们主要关注一下它的配置属性

```kotlin
  //调度器，调度后台线程发起请求。
  @get:JvmName("dispatcher") val dispatcher: Dispatcher = builder.dispatcher
  //连接池，对链接进行管理和复用，和线程池差不多。
  @get:JvmName("connectionPool") val connectionPool: ConnectionPool = builder.connectionPool

  //拦截器
  @get:JvmName("interceptors") val interceptors: List<Interceptor> =
      builder.interceptors.toImmutableList()

  @get:JvmName("networkInterceptors") val networkInterceptors: List<Interceptor> =
      builder.networkInterceptors.toImmutableList()
  //对连接的发起，连接的返回等一系列事件监听。
  @get:JvmName("eventListenerFactory") val eventListenerFactory: EventListener.Factory =
      builder.eventListenerFactory
  //对连接失败和请求失败的重试开关。
  @get:JvmName("retryOnConnectionFailure") val retryOnConnectionFailure: Boolean =
      builder.retryOnConnectionFailure
  //用于自动重新认证，比如用来做token的刷新。
  @get:JvmName("authenticator") val authenticator: Authenticator = builder.authenticator
  //当需要重定向时，是否自动follow
  @get:JvmName("followRedirects") val followRedirects: Boolean = builder.followRedirects
  //当需要重定向，并且还会切换协议的时候（比如https切换成http），是否自动follow
  @get:JvmName("followSslRedirects") val followSslRedirects: Boolean = builder.followSslRedirects
  //饼干罐，用于存储cookie，OkHttp默认没有实现。
  @get:JvmName("cookieJar") val cookieJar: CookieJar = builder.cookieJar
  //缓存
  @get:JvmName("cache") val cache: Cache? = builder.cache
  //域名服务
  @get:JvmName("dns") val dns: Dns = builder.dns
  //使用代理服务器
  @get:JvmName("proxy") val proxy: Proxy? = builder.proxy

  @get:JvmName("proxySelector") val proxySelector: ProxySelector =
      when {
        // Defer calls to ProxySelector.getDefault() because it can throw a SecurityException.
        builder.proxy != null -> NullProxySelector
        else -> builder.proxySelector ?: ProxySelector.getDefault() ?: NullProxySelector
      }

  @get:JvmName("proxyAuthenticator") val proxyAuthenticator: Authenticator =
      builder.proxyAuthenticator
  //使用socket进行http连接
  @get:JvmName("socketFactory") val socketFactory: SocketFactory = builder.socketFactory
  
  private val sslSocketFactoryOrNull: SSLSocketFactory?
  //使用加密过的socket进行http连接
  @get:JvmName("sslSocketFactory") val sslSocketFactory: SSLSocketFactory
    get() = sslSocketFactoryOrNull ?: throw IllegalStateException("CLEARTEXT-only client")
  //证书验证器
  @get:JvmName("x509TrustManager") val x509TrustManager: X509TrustManager?
  //存放有tls版本和所支持的加密套件（包括非对称加密，对称加密和hash算法）
  @get:JvmName("connectionSpecs") val connectionSpecs: List<ConnectionSpec> =
      builder.connectionSpecs
  //支持的协议版本，Http/1.1 Http/2.0等
  @get:JvmName("protocols") val protocols: List<Protocol> = builder.protocols
  //证书验证相关，对域名进行验证
  @get:JvmName("hostnameVerifier") val hostnameVerifier: HostnameVerifier = builder.hostnameVerifier
  //证书验证相关，可以自验证证书
  @get:JvmName("certificatePinner") val certificatePinner: CertificatePinner
  //证书验证相关，证书验证的实际操作者
  @get:JvmName("certificateChainCleaner") val certificateChainCleaner: CertificateChainCleaner?

  /**
   * Default call timeout (in milliseconds). By default there is no timeout for complete calls, but
   * there is for the connect, write, and read actions within a call.
   */
  @get:JvmName("callTimeoutMillis") val callTimeoutMillis: Int = builder.callTimeout

  /** Default connect timeout (in milliseconds). The default is 10 seconds. */
  @get:JvmName("connectTimeoutMillis") val connectTimeoutMillis: Int = builder.connectTimeout

  /** Default read timeout (in milliseconds). The default is 10 seconds. */
  @get:JvmName("readTimeoutMillis") val readTimeoutMillis: Int = builder.readTimeout

  /** Default write timeout (in milliseconds). The default is 10 seconds. */
  @get:JvmName("writeTimeoutMillis") val writeTimeoutMillis: Int = builder.writeTimeout

  //发送心跳连接，来保持连接存活，用于webSocekt和Http2
  @get:JvmName("pingIntervalMillis") val pingIntervalMillis: Int = builder.pingInterval
```



## RealCall 请求执行者

之前`execute`和`enqueue`最终都会执行`getResponseWithInterceptorChain`方法，而这两个都是RealCall的方法。其中`enqueue`最终会执行到`AsyncCall`的`run`方法，而`AsyncCall`其实是RealCall的子类，也就是说`getResponseWithInterceptorChain`其实是`RealCall`中的方法。

那我们来看看`getResponseWithInterceptorChain`到底做了什么把。

```kotlin
  internal fun getResponseWithInterceptorChain(): Response {
    //添加各种拦截器
    val interceptors = mutableListOf<Interceptor>()
    //这是供使用者使用的拦截器
    interceptors += client.interceptors
    //重试和重定向拦截器
    interceptors += RetryAndFollowUpInterceptor(client)
    //发送时，对reques的header和body进行拼接，得到响应后，对Response进行解析。
    interceptors += BridgeInterceptor(client.cookieJar)
    //缓存拦截器，会根据一些策略判断缓存是否过期，没有过期则直接返回。
    interceptors += CacheInterceptor(client.cache)
    //和tcp和ssl连接相关
    interceptors += ConnectInterceptor
    if (!forWebSocket) {
      interceptors += client.networkInterceptors
    }
    //最终的http请求发起者。
    interceptors += CallServerInterceptor(forWebSocket)

    //这里构造了一个责任链
    val chain = RealInterceptorChain(
        call = this,
        interceptors = interceptors,
        index = 0,
        exchange = null,
        request = originalRequest,
        connectTimeoutMillis = client.connectTimeoutMillis,
        readTimeoutMillis = client.readTimeoutMillis,
        writeTimeoutMillis = client.writeTimeoutMillis
    )
		
    try {
      ...
      //从第一个拦截器开始逐级执行。
      val response = chain.proceed(originalRequest)
      return response
    } catch (e: IOException) {
        ...
    } finally {
        ...
    }
  }
```

可以看到`getResponseWithInterceptorChain`的工作就是，把各种拦截器都组合成一个list，然后使用责任链模式，将list放入到`RealInterceptorChain`中，并开始逐级执行。



## RealInterceptorChain

`RealInterceptorChain`是拦截器能够逐级执行的实际管理者。



来看看它的核心代码：

```kotlin
  override fun proceed(request: Request): Response {
    //index是每个拦截器的下标，这里就是进行一个边界判断
    check(index < interceptors.size)
		...
    //这里来获取下一个拦截器
    val next = copy(index = index + 1, request = request)
    val interceptor = interceptors[index]

    @Suppress("USELESS_ELVIS")
    //调用这个拦截器的intercept方法
    val response = interceptor.intercept(next) ?: throw NullPointerException(
        "interceptor $interceptor returned null")
		...
    //最后返回response
    return response
  }
```

这就是责任链调度的核心方法，其实现**类似递归**。



其实到这里就差不多了，接下来就简单看看几个拦截器实际执行的intercept方法，其实不看都行。

每个拦截器的intercept执行方法，其关键是找到`realChain.proceed`方法，在这个方法之前的都是进行前置处理，也就是发送前的处理。在此方法后的都是后置处理，也就是得到响应后的处理。



接着来看看OkHttp责任链的执行流程：

<img src="https://user-gold-cdn.xitu.io/2018/10/19/1668c5c6363ea20f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" style="zoom:70%; background-color: #f4f5f5;" />



### RetryAndFollowUpInterceptor

负责在请求失败时的重试，以及重定向的 ⾃动后续请求。它的存在，可以让重试和重定向对于开发者是⽆感知的

```kotlin
	override fun intercept(chain: Interceptor.Chain): Response {
    ...
    //可以看到是一个死循环
    while (true) {
      ...//进行前置处理
        try {
          //交给下一个拦截器处理
          response = realChain.proceed(request)
          newExchangeFinder = true
          //这里两个catch都是失败后，需要重试或者重定向的处理
        } catch (e: RouteException) {
          // The attempt to connect via a route failed. The request will not have been sent.
          if (!recover(e.lastConnectException, call, request, requestSendStarted = false)) {
            throw e.firstConnectException.withSuppressed(recoveredFailures)
          } else {
            recoveredFailures += e.firstConnectException
          }
          newExchangeFinder = false
          continue
        } catch (e: IOException) {
          // An attempt to communicate with a server failed. The request may have been sent.
          if (!recover(e, call, request, requestSendStarted = e !is ConnectionShutdownException)) {
            throw e.withSuppressed(recoveredFailures)
          } else {
            recoveredFailures += e
          }
          newExchangeFinder = false
          continue
        }
        
       	...
        //满足条件后就跳出循环，返回Response
        if (followUp == null) {
          if (exchange != null && exchange.isDuplex) {
            call.timeoutEarlyExit()
          }
          closeActiveExchange = false
          return response
        }

        val followUpBody = followUp.body
        if (followUpBody != null && followUpBody.isOneShot()) {
          closeActiveExchange = false
          return response
        }
        request = followUp
        priorResponse = response
      } finally {
        call.exitNetworkInterceptorExchange(closeActiveExchange)
      }
    }
  }
```



### BridgeInterceptor

主要是发送时，对reques的header和body进行拼接，得到响应后，对Response进行解析。

```kotlin
  override fun intercept(chain: Interceptor.Chain): Response {
    val userRequest = chain.request()
    val requestBuilder = userRequest.newBuilder()
		...
     //下面都是类似的请求拼接流程
    if (userRequest.header("Host") == null) {
      requestBuilder.header("Host", userRequest.url.toHostHeader())
    }
		...
    //把处理交给下一个拦截器
    val networkResponse = chain.proceed(requestBuilder.build())

     //下面就是得到响应后做一些解析工作
    cookieJar.receiveHeaders(userRequest.url, networkResponse.headers)

    val responseBuilder = networkResponse.newBuilder()
        .request(userRequest)

    if (transparentGzip &&
        "gzip".equals(networkResponse.header("Content-Encoding"), ignoreCase = true) &&
        networkResponse.promisesBody()) {
      val responseBody = networkResponse.body
      if (responseBody != null) {
        val gzipSource = GzipSource(responseBody.source())
        val strippedHeaders = networkResponse.headers.newBuilder()
            .removeAll("Content-Encoding")
            .removeAll("Content-Length")
            .build()
        responseBuilder.headers(strippedHeaders)
        val contentType = networkResponse.header("Content-Type")
        responseBuilder.body(RealResponseBody(contentType, -1L, gzipSource.buffer()))
      }
    }

    return responseBuilder.build()
  }
```



### CacheInterceptor

缓存拦截器，会根据一些策略判断缓存是否过期，没有过期则直接返回。

```kotlin
	override fun intercept(chain: Interceptor.Chain): Response {
		
      ...
     //这里就是缓存处理判断的地方
    val strategy = CacheStrategy.Factory(now, chain.request(), cacheCandidate).compute()
    
      ...
        
    try {
       //交给下一个拦截器处理
      networkResponse = chain.proceed(networkRequest)
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        cacheCandidate.body?.closeQuietly()
      }
    }

    // 判断是否需要更新缓存
    if (cacheResponse != null) {
      if (networkResponse?.code == HTTP_NOT_MODIFIED) {
        val response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers, networkResponse.headers))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis)
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis)
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build()

        networkResponse.body!!.close()

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache!!.trackConditionalCacheHit()
        cache.update(cacheResponse, response)
        return response.also {
          listener.cacheHit(call, it)
        }
      } else {
        cacheResponse.body?.closeQuietly()
      }
    }
		...
    return response
  }
```



### ConnectInterceptor

tcp和ssl连接相关

```kotlin
  override fun intercept(chain: Interceptor.Chain): Response {
    val realChain = chain as RealInterceptorChain
    val exchange = realChain.call.initExchange(chain)
    val connectedChain = realChain.copy(exchange = exchange)
    return connectedChain.proceed(realChain.request)
  }
```



### CallServerInterceptor

代码就不贴了，最后这个拦截器的工作就是发送Http包和接受响应了。



## 总结

到这里就正在的结束了，最后再简单的总结一下：

**当使用`client.newCall(request)` 时我们创建了一个`ReallCall`对象，紧接着调用了`RealCall`的`execute()`方法，把`RealCall`添加进执行队列中，最后通过`getResponseWithInterceptorChain()`方法（其内部使用通过拦截器组成的责任链，通过 用户自定的拦截器、重试(重定向)、桥接、缓存、建立tcp和tls连接 和 建立Http连接）最终得到了`Response`返回结果。而`enqueue`只是多了个通过使用dispatcher将其调度到后台线程进行执行而已。**



## 参考

1. [OkHttp:4.7.2源码](https://github.com/square/okhttp)
2. [OKHttp源码解析](https://www.jianshu.com/p/27c1554b7fee)
3. [Andriod 网络框架 OkHttp 源码解析](https://juejin.im/post/5bc89fbc5188255c713cb8a5#heading-10)

