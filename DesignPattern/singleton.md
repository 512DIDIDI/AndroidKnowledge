### 单例模式

* **为什么要用单例模式(何时使用，解决了什么问题)**：

  这个类**只允许创建一个对象或者实例**，例如以下两个场景

  1. 处理资源访问冲突，例如日志，多线程同时写入日志，为了防止日志被覆盖，应该设计为单例模式
  2. 全局共享类，例如配置文件类，最终配置文件加载到内存中，应该只存在一份实例。

* **怎么实现单例以及各种模式的优缺点**

  ```
  /**
   * @author dididi(叶超)
   * @email yc512yc@163.com
   * @since 02/05/2020
   * @describe 单例模式(回字有几种写法)
   * 总结一下：
   *      一般情况下，直接使用饿汉object就行
   *      除非真的有延迟初始化的需求，可以选择双重校验或静态内部类。
   */
  
  /**
   * 饿汉式
   * kotlin的object本身就会创建一个INSTANCE的实例，
   * 访问object中的方法会通过INSTANCE来调用，因此，object xxx{} 就是kotlin中的饿汉单例实现
   *
   * 优点：
   *      1. 创建方式简单，在kotlin中仅需使用object关键字就可以
   * 缺点：
   *      1. 不支持延迟初始化带来的问题(在使用类引用时就创建单例对象，造成占用内存多，初始化耗时长等)
   *
   * 一点观点：
   *      饿汉单例模式，虽然不支持延迟初始化，会带来其问题，但按照fail-first(有问题早暴露)的设计原则，
   *      如果实例化过程中会出现OOM情况，那应该让其尽早出现，尽早解决。而如果初始化耗时长，
   *      也应该在开屏页面直接初始化，而不是在用户使用过程中初始化，避免出现卡顿和ANR现象。
   */
  object HungrySingleton {
      fun test() {}
  }
  
  /**
   * 懒汉式1
   * 缺点：
   *      1. 没有考虑并发情况，并发会创造多个实例风险
   * 优点：
   *      1. 延迟初始化(当需要获取单例对象时，才创建单例对象，一般来说就是调用getInstance时才创建对象)
   *         避免使用类引用时就创建单例对象。
   *      2. 避免实例占用内存多
   */
  class LazySingleton private constructor() {
      companion object {
          private var instance: LazySingleton? = null
          fun getInstance() = instance ?: LazySingleton()
      }
  
      fun test(){}
  }
  
  /**
   * 懒汉式2
   * 改进点：
   *      1. 考虑到懒汉式会创建多个实例的风险，所以需要给getInstance加锁
   * 缺点：
   *      1. 由于给getInstance方法加了锁，因此出现高并发状况时，就算已经有了单例对象，
   *         也要阻塞等别的线程获取完毕后，才能调用getInstance方法
   */
  class LazySynchronizedSingleton private constructor() {
      companion object {
          private var synchronizedSingleton: LazySynchronizedSingleton? = null
  
          //这种做法将getInstance方法直接锁住了 虽然避免了创造多个实例的风险，但当已经有实例时，也要等待耗时，是一种很差劲的写法
          @Synchronized
          fun getInstance() = synchronizedSingleton ?: LazySynchronizedSingleton()
      }
  
      fun test(){}
  }
  
  
  /**
   * 双重校验锁模式
   * 在有些jvm上会出现错误(jdk1.5以下)
   * 首先要明白在JVM创建新的对象时，主要要经过三步。
   *
   * 1.分配内存
   *
   * 2.初始化构造器
   *
   * 3.将对象指向分配的内存的地址
   *
   * 这种顺序在上述双重加锁的方式是没有问题的，
   * 因为这种情况下JVM是完成了整个对象的构造才将内存的地址交给了对象。
   * 但是如果2和3步骤是相反的（2和3可能是相反的是因为JVM会针对字节码进行调优，
   * 而其中的一项调优便是调整指令的执行顺序），就会出现问题了。
   * 因为这时将会先将内存地址赋给对象，针对上述的双重加锁，
   * 就是说先将分配好的内存地址指给synchronizedSingleton，
   * 然后再进行初始化构造器，这时候后面的线程去请求getInstance方法时，
   * 会认为synchronizedSingleton对象已经实例化了，直接返回一个引用。
   * 如果在初始化构造器之前，这个线程使用了synchronizedSingleton，就会产生莫名的错误。
   *
   * 可以加上 [Volatile] 关键字来禁止jvm自动的指令重排序优化
   */
  class DoubleCheckSingleton private constructor() {
      companion object {
          @Volatile
          private var instance: DoubleCheckSingleton? = null
  
          //只在实例未初始化时才同步锁
          fun getInstance() =
              //todo: 注意 这里是一把类锁 synchronized(SynchronizedSingleton::class)，这样能使所有SynchronizedSingleton对象共享同一把锁
              instance ?: synchronized(DoubleCheckSingleton::class) {
              //同步块中也需要判断是否为空，两个线程同时进入第一个判断，a线程创建好后，b线程应该要返回a线程创建的单例
              instance ?: DoubleCheckSingleton()
          }
      }
  
      fun test(){}
  }
  
  /**
   * 静态内部类实现单例
   * 由JVM的类加载机制来确保只有唯一一个实例以及延迟初始化
   * 类的初始化触发条件：
   *      1. 创建类的实例
   *      2. 访问类的静态变量/静态方法(除了常量)
   *      3. 反射
   *      4. 初始化一个类，如果发现其父类未初始化，先从父类开始初始化
   *      5. 虚拟机启动时，定义了main()方法的那个类先开始初始化
   * todo:不清楚kotlin静态内部单例是否如此实现，这样的写法会多出object内的一个INSTANCE对象
   */
  class InnerClassSingleton private constructor(){
  
      companion object{
          fun getInstance() = Inner.instance
      }
  
      private object Inner{
          val instance = InnerClassSingleton()
      }
  
      fun test(){}
  }
  
  /**
   * 枚举类实现单例
   * 实质上内部实现跟饿汉模式一样
   */
  enum class EnumSingleton{
      INSTANCE;
  
      fun test(){}
  }
  ```

* **源码中有哪些场景使用了单例模式**

  1. `AccessibilityManager`用了饿汉模式创建单例对象。
  
* **开发过程中有用过单例模式吗**

  以前写demo用过，主要是配置信息的读写（为了保证其只有一份配置信息），例如后台地址，`Iconfont`的字体库等等配置信息。




