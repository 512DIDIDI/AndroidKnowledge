### 代理模式

* **什么是代理模式(解决了什么问题)**

  代理：【代理方】代表【被代理对象】处理【问题】。

  翻译为程序语言，什么是_代表【被代理对象】处理问题_，答：组合【被代理对象】，在【被代理对象】需要处理的问题前后加上【代理方】需要做的事情。

  所以不管是静态代理(实现统一接口，组合【被代理对象】)，还是动态代理(实际也是组合【被代理对象】)，其实本质都一样。都是为了扩展【被代理对象】的**方法**，在【被代理对象】的行为前后加上【代理方】的逻辑。

* **静态代理与动态代理的差别与实现**

  ```kotlin
  /**
   * 静态代理解决了什么问题？
   * 答：
   *      1. 拓展被代理对象的方法。实际上就是通过实现被代理对象的接口，组合被代理对象的方式。重写其方法。
   *      2. 当被代理对象不是接口时，实际就是继承被代理类，选择性的重写其方法。
   * 有什么局限性？
   * 答：
   *      1. 当方法很多，并且每个方法的扩展逻辑一样，容易写出很多冗余代码。
   *      2. 当有很多需要被代理的对象，那么就需要实现n个接口，代码量十分恐怖。
   */
  class StaticUserCollectorProxy(private val userController: UserController):IUserController{
  
      private val metricsCollector = MetricsCollector()
  
      override fun writeFile(message: String) {
          //被代理对象的方法执行前的逻辑
          val startTimeStamp = System.currentTimeMillis()
          //执行被代理对象的方法
          userController.writeFile(message)
          //被代理对象的方法执行后的逻辑
          val endTimeStamp = System.currentTimeMillis()
          val responseTime = endTimeStamp - startTimeStamp
          val apiName = "${this.javaClass.name}:writeFile"
          metricsCollector.recordRequest(RequestInfo(apiName, responseTime, startTimeStamp))
      }
  
      override fun login(userName: String, password: String) {
          //被代理对象的方法执行前的逻辑
          val startTimeStamp = System.currentTimeMillis()
          //执行被代理对象的方法
          userController.login(userName, password)
          //被代理对象的方法执行后的逻辑
          val endTimeStamp = System.currentTimeMillis()
          val responseTime = endTimeStamp - startTimeStamp
          val apiName = "${this.javaClass.name}:login"
          metricsCollector.recordRequest(RequestInfo(apiName, responseTime, startTimeStamp))
      }
  
      override fun signUp(userName: String) {
          //被代理对象的方法执行前的逻辑
          val startTimeStamp = System.currentTimeMillis()
          //执行被代理对象的方法
          userController.signUp(userName)
          //被代理对象的方法执行后的逻辑
          val endTimeStamp = System.currentTimeMillis()
          val responseTime = endTimeStamp - startTimeStamp
          val apiName = "${this.javaClass.name}:signUp"
          metricsCollector.recordRequest(RequestInfo(apiName, responseTime, startTimeStamp))
      }
  }
  
  
  /**
   * 动态代理能够解决什么问题？
   * 答：批量为被代理对象(source)的**方法**扩展功能
   * 如果这些扩展功能是可以并且需要被复用的逻辑，并且可能会有大量的代理类需要这些拓展功能，
   * 可以使用动态代理模式，使用代理类替换被代理对象的方法。
   */
  class UserCollectorProxy(private val source: Any) : InvocationHandler {
  
      private val metricsCollector = MetricsCollector()
  
      fun createProxy(): Any {
          //Proxy.newProxyInstance返回的是接口或者Proxy的实例
          return Proxy.newProxyInstance(
              source.javaClass.classLoader,
              source.javaClass.interfaces,
              this
          )
      }
  
      override fun invoke(proxy: Any?, method: Method?, args: Array<out Any>?): Any? {
          //被代理对象的方法执行前的逻辑
          val startTimeStamp = System.currentTimeMillis()
          //注意 kotlin需要把数组展开为可变长参数，否则会出现类型不匹配
          //调用被代理对象的方法。
          val result = method?.invoke(source, *(args ?: emptyArray()))
          //被代理对象的方法执行后的逻辑
          val endTimeStamp = System.currentTimeMillis()
          val responseTime = endTimeStamp - startTimeStamp
          val apiName = "${source.javaClass.name}:${method?.name}"
          metricsCollector.recordRequest(RequestInfo(apiName, responseTime, startTimeStamp))
          return result
      }
  }
  
  /**
   * 被代理对象(被代理对象必须实现了接口或者就是接口本身)
   */
  open class UserController : IUserController {
  
      private val file = File("user.txt")
  
      override fun writeFile(message: String) {
          file.writeText("hello$message\n")
      }
  
      override fun login(userName: String, password: String) {
          file.appendText("login:${userName}\n")
      }
  
      override fun signUp(userName: String) {
          file.appendText("sign:${userName}\n")
      }
  }
  
  /**
   * 被代理对象的接口
   */
  interface IUserController {
      fun writeFile(message: String)
  
      fun login(userName:String,password:String)
  
      fun signUp(userName: String)
  }
  
  /**测试类*/
  fun main() {
      val staticProxy = StaticUserCollectorProxy(UserController())
      staticProxy.apply {
          writeFile(" haha")
          signUp("test")
          login("test","123")
      }
  
      val proxy = UserCollectorProxy(UserController())
      //使用IUserController代理UserController，拓展其方法。
      (proxy.createProxy() as IUserController).apply {
          //每一次调用方法，都会调用proxy里的invocationHandler.invoke()方法
          writeFile(" world")
          signUp("dididi")
          login("dididi","123")
      }
  }
  
  class MetricsCollector {
      fun recordRequest(requestInfo: RequestInfo) {
          println(requestInfo)
      }
  }
  
  data class RequestInfo(val apiName: String, val responseTime: Long, val startTimeStamp: Long)
  ```

* **源码中有哪些使用到了代理模式**

  1. `Retrofit`的`create`方法使用动态代理模式生成`ServiceApi`的代理类

