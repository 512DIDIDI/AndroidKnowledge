### 建造者模式

* **为什么要用建造者模式(何时使用，解决了什么问题)**

  建造者模式优化了**调用者创建对象**的过程，**隐藏了创建对象的复杂逻辑**，**实现定制化创建对象**，在下面三种场景下，可以考虑使用建造者模式

  1. 优化多参数构造器
  2. 构建完成后，要求类对象属性值不可变(只能读不能写)，但又不希望通过构造器传入属性
  3. 构建过程中，属性之间有依赖关系或约束条件。

* **三种场景下的使用**

  ```kotlin
  /**
   * scene1：当一个类的主构造函数需要很多必填参数时
   *
   * 不引入建造者模式，直接使用 constructor 构造 会出现相当长的构造代码
   */
  class PoolConfig(
      val name: String,
      val time: String,
      val maxTotal: Int,
      val minTotal: Int,
      val minIdle: Int,
      val maxCount: Int,
      val minCount: Int
  )
  
  /**
   * 建造者模式的缺点也很明显：相同的属性得定义两次，代码量增加了很多很多
   */
  class ImprovePoolConfig private constructor(private val builder: Builder) {
      var name = builder.name!!
      var time = builder.time!!
      var maxTotal = builder.maxTotal!!
      var minTotal = builder.minTotal!!
      var minIdle = builder.minIdle!!
      var maxCount = builder.maxCount!!
      var minCount = builder.minCount!!
  
      class Builder {
          var name: String? = null
          var time: String? = null
          var maxTotal: Int? = null
          var minTotal: Int? = null
          var minIdle: Int? = null
          var maxCount: Int? = null
          var minCount: Int? = null
          fun build(): ImprovePoolConfig {
              name.checkNull { throw IllegalArgumentException("name can not be null") }
              time.checkNull { throw IllegalArgumentException("time can not be null") }
              maxTotal.checkNull { throw IllegalArgumentException("maxTotal can not be null") }
              minTotal.checkNull { throw IllegalArgumentException("minTotal can not be null") }
              minIdle.checkNull { throw IllegalArgumentException("minIdle can not be null") }
              maxCount.checkNull { throw IllegalArgumentException("maxCount can not be null") }
              minCount.checkNull { throw IllegalArgumentException("minCount can not be null") }
              return ImprovePoolConfig(this)
          }
      }
  }
  
  
  /**
   * scene2：当我们希望一个对象构造完成后，其属性就不可变，但又不希望通过构造函数传入属性。
   *
   * 考虑使用建造者模式，通过builder来传入属性构造对象，真正的对象，属性方法均为val 只有getter方法
   */
  class ValParam private constructor(builder:Builder){
  
      val name = builder.name
      val age = builder.age
      val gender = builder.gender
  
      class Builder{
          var name:String? = null
          var age:Int = 0
          var gender:Int = 0
          fun build() = ValParam(this)
      }
  }
  
  
  /**
   * scene3：当类的属性之间有一定的依赖关系或者约束条件，
   *         使用 setXXX 或 constructor 无法放置这些逻辑，
   *         建造者模式，在build构建方法中可以安排这些逻辑。
   * 在此例中，isRef 的 true|false 影响了 arg 的类别， 影响了 type arg是否能为空。
   */
  class ConstructorArg private constructor(builder: Builder) {
      var isRef = builder.isRef
          private set
      var type = builder.type
          private set
      var arg = builder.arg
          private set
  
      class Builder(val isRef: Boolean) {
          var type: Class<*>? = null
          var arg: Any? = null
  
          /**当构造器中有复杂逻辑时，而普通的new / set又无法实现此逻辑，采用构造者模式来实现。*/
          fun build(): ConstructorArg {
              if (isRef) {
                  if (arg !is String) {
                      throw IllegalArgumentException("when isRef=true,arg must be String")
                  }
              } else {
                  if (arg == null || type == null) {
                      throw IllegalArgumentException("when isRef=false,arg and type must init")
                  }
              }
              return ConstructorArg(this)
          }
      }
  }
  
  fun main(){
      PoolConfig("dididi","123",20,12,13,13,14)
      //优化多参数构造器
      ImprovePoolConfig.Builder()
          .apply {
              name = "dididi"
              time = "123"
              maxTotal = 20
              minTotal = 12
              maxCount = 13
              minCount = 13
              minIdle = 14
          }
          .build()
      //构建完成后，仅能获取对象的属性，不能再set
      ValParam.Builder()
          .apply {
              age = 2
          }
          .build()
          .age
      //构建过程中，属性之间有约束关系或依赖关系
      ConstructorArg.Builder(true)
          .apply {
              arg = "hello"
          }
          .build()
  }
  
  fun Any?.checkNull(block:()->Unit){
      if (this == null){
          block()
      }
  }
  ```

* **源码中有什么应用到了建造者模式**

  1. `StringBuilder`，各种`append`方法就是传入属性值，`toString`就是构建`String`字符串，类似场景1。
  2. `AlertDialog.Builder`，`setTitle`，`setMessage`等设置`dialog`的属性，`create`创建`AlertDialog`，类似场景1。