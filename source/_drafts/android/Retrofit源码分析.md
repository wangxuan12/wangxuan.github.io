## 基本使用

直接使用官方的:chestnut:

### 创建HTTP API接口

```kotlin
interface GitHubService {
    @GET("users/{user}/repos")
    fun listRepos(@Path("user") user: String): Call<List<Repo>>
}
```



### 使用Retrofit生成接口实例

```kotlin
val retrofit : Retrofit = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .build()
val testService : GitHubService = retrofit.create(GitHubService::class.java)
```



### 调用实例方法，生成Call对象

```kotlin
val repos : Call<List<Repo>> = testService.listRepos("octocat")
```



### 发起请求

```kotlin
repos.execute()
//或者
repos.enqueue(callback)
```



## Retrofit源码

`Retrofit.Builder().build()` 里面的callFactory也就是实际请求发起者其实就是OkHttpClient

```kotlin
public Retrofit build() {
      ...
    	//创建一个OkHttpClient并将其复制给callFactory
      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }
}
```



`retrofit.create()`

内部使用了动态代理，也就是create传进来的接口会在 运行期生成具体实例。调用的Service接口方法，最后都会调用`InvocationHandler`的`invoke`方法。

```kotlin
  public <T> T create(final Class<T> service) {
    validateServiceInterface(service);
    return (T)
        Proxy.newProxyInstance(
            service.getClassLoader(),
            new Class<?>[] {service},
            new InvocationHandler() {
              private final Platform platform = Platform.get();
              private final Object[] emptyArgs = new Object[0];

              @Override
              public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
                  throws Throwable {
                
               	//这里判断如果是Object的方法，那就啥都不做。
                if (method.getDeclaringClass() == Object.class) {
                  return method.invoke(this, args);
                }
                args = args != null ? args : emptyArgs;
                //如果是有默认实现的方法，也不处理，所以关键是else的部分。
                return platform.isDefaultMethod(method)
                    ? platform.invokeDefaultMethod(method, service, proxy, args)
                    : loadServiceMethod(method).invoke(args);
              }
            });
  }
```



### loadServiceMethod

`loadServiceMethod(method)`这个方法会返回一个ServiceMethod对象，而ServiceMethod其实就是Retrofit解析Service接口原方法（返回值类型，参数类型，方法注解，参数注解等)生成的。

```kotlin
  private final Map<Method, ServiceMethod<?>> serviceMethodCache = new ConcurrentHashMap<>();

  ServiceMethod<?> loadServiceMethod(Method method) {
    //复用之前使用过的ServiceMethod
    ServiceMethod<?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        //判断方法是否符合规范，并且对方法的各种信息进行解析，最后生成ServiceMethod对象。
        result = ServiceMethod.parseAnnotations(this, method);
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```



`loadServiceMethod(method).invoke(args)`这行后面还调用了一个invoke方法，也就是调用的ServiceMethod中的`invoke`。

但是ServiceMethod并没有实现，实现的是它的子类HttpServiceMethod。

```kotlin
  final @Nullable ReturnT invoke(Object[] args) {
    //生成了一个OkHttpCall
    Call<ResponseT> call = new OkHttpCall<>(requestFactory, args, callFactory, responseConverter);
    //这里这个adapt使用CallAdapter将OkHttpCall进行转换。也是在这里会调用OkHttpCall发送实际的网络请求
    return adapt(call, args);
  }
```

