### 工厂模式

* **为什么要用工厂模式(何时使用，解决了什么问题)**

  当出现了大量的`if-else(when)`**创建复杂对象**时，可以考虑使用工厂模式来优化代码。

* **怎么实现简单工厂和工厂模式(工厂模式是如何演变过来的)**

  ```java
  /**
   * @author dididi(yechao)
   * @since 05/08/2020
   * @describe 工厂类模式的演变过程
   * 缺点：
   *      1. 降低代码阅读性
   *      2. 增加代码量
   * 优点：
   *      1. 提升代码复用性，抽离了独立的业务代码
   *      2. 隔离了创建对象的复杂性，只管调用，不需要了解如何实现。
   *      3. 控制代码复杂度，使类的职责更加单一化，每个类的代码更加简洁。
   *
   * 一点观点：
   *      1. 工厂模式并不能消除代码的复杂度，它只是转移了这个复杂度，
   *         或者转移到函数(抽离成为函数)，if-else在函数内部，
   *         或者转移到类(简单工厂)，if-else在简单工厂类内部，
   *         或者转移到创建工厂类对象(工厂)的过程中，if-else在工厂的简单工厂内部。
   *      2. 工厂模式是**对象**创建 / 策略者模式是**行为**构建
   */
  
  /**
   * step1：一般写法，根据不同的配置信息，使用if-else或者when来动态创建对象或构建行为(实际对象的职责也是为了构建行为)
   * 缺点：代码臃肿，不同业务功能代码耦合。
   */
  class ParserLoader1 {
      fun load(config: Config) {
          println("do something")
          val parser = when (config) {
              Config.XML -> XmlParser()
              Config.JSON -> JsonParser()
              Config.HTML -> HtmlParser()
              Config.BASE64 -> Base64Parser()
              Config.TXT -> TxtParser()
          }
          parser.parse()
          println("do something")
      }
  }
  
  /**
   * step2：可以将具有独立功能的代码抽离成为函数。
   * 改进点：抽离独立业务为函数。
   */
  class ParserLoader2 {
      fun load(config: Config) {
          println("do something")
          val parser = createParser(config)
          parser.parse()
          println("do something")
      }
  
      private fun createParser(config: Config):Parser {
          return when (config) {
              Config.XML -> XmlParser()
              Config.JSON -> JsonParser()
              Config.HTML -> HtmlParser()
              Config.BASE64 -> Base64Parser()
              Config.TXT -> TxtParser()
          }
      }
  }
  
  
  class ParserLoader3 {
      fun load(config: Config) {
          println("do something")
          val parser = Parser3Factory.createParse(config)
          parser.parse()
          println("do something")
      }
  }
  
  /**
   * step3：**引入简单工厂模式**，将独立业务抽离为单独的类。
   * 改进点：为了使类的职责更单一，我们将独立业务抽离为单独的类，让简单工厂负责帮我们创建对象或构建行为。
   * 注意点：简单工厂模式又叫做静态工厂方法模式，其创建对象的方法是静态方法。
   *        为什么？避免创建多余的工厂类实例，浪费内存。
   */
  object Parser3Factory {
      fun createParse(config: Config):Parser {
          return when (config) {
              Config.XML -> XmlParser()
              Config.JSON -> JsonParser()
              Config.HTML -> HtmlParser()
              Config.BASE64 -> Base64Parser()
              Config.TXT -> TxtParser()
          }
      }
  }
  
  class ParserLoader4 {
      fun load(config: Config) {
          println("do something")
          val parser = Parser4Factory.createParse(config)
          parser.parse()
          println("do something")
      }
  }
  
  /**
   * step4：引入缓存池，解决对象复用问题。
   * 改进点：引入了缓存池，减少每次调用都需要创建对象的开销。
   * 事实上，大部分代码改进到这里就可以了，但如果构建工厂的过程也非常复杂，可以考虑工厂模式。
   */
  object Parser4Factory {
  
      private val cache = hashMapOf<Config, Parser>()
  
      init {
          cache[Config.XML] = XmlParser()
          cache[Config.JSON] = JsonParser()
          cache[Config.HTML] = HtmlParser()
          cache[Config.BASE64] = Base64Parser()
          cache[Config.TXT] = TxtParser()
      }
  
      fun createParse(config: Config):Parser {
          return cache[config] ?: throw IllegalArgumentException("no such config")
      }
  }
  
  /**
   * step5：工厂模式引入，工厂1抽象接口+简单工厂2(创建不同的工厂1对象)
   * 改进点：如果工厂对象的创建过程复杂，可以考虑再抽离一层，给工厂对象抽离接口，用简单工厂创建工厂对象。
   */
  class ParserLoader5 {
      fun load(config: Config) {
          println("do something")
          /**map获取具体的工厂对象，再由该工厂对象调用具体方法返回对象。*/
          val parser = Parser5FactoryMap.createFactory(Config.XML).createParser()
          parser.parse()
          println("do something")
      }
  }
  
  /**工厂类接口*/
  interface AbstractParser5Factory {
      fun createParser():Parser
  }
  
  /**具体工厂类实现*/
  class XmlParser5Factory : AbstractParser5Factory {
      override fun createParser(): Parser {
          println("do something")
          return XmlParser()
      }
  }
  
  class JsonParser5Factory : AbstractParser5Factory {
      override fun createParser(): Parser {
          println("do something")
          return JsonParser()
      }
  }
  
  class HtmlParser5Factory : AbstractParser5Factory {
      override fun createParser(): Parser {
          println("do something")
          return HtmlParser()
      }
  }
  
  class Base64Parser5Factory : AbstractParser5Factory {
      override fun createParser(): Parser {
          println("do something")
          return Base64Parser()
      }
  }
  
  class TxtParser5Factory : AbstractParser5Factory {
      override fun createParser(): Parser {
          println("do something")
          return TxtParser()
      }
  }
  
  /**工厂的简单工厂类，创建工厂对象*/
  object Parser5FactoryMap {
  
      private val cacheFactory = hashMapOf<Config, AbstractParser5Factory>()
  
      init {
          cacheFactory[Config.XML] = XmlParser5Factory()
          cacheFactory[Config.JSON] = JsonParser5Factory()
          cacheFactory[Config.HTML] = HtmlParser5Factory()
          cacheFactory[Config.BASE64] = Base64Parser5Factory()
          cacheFactory[Config.TXT] = TxtParser5Factory()
      }
  
      fun createFactory(config: Config): AbstractParser5Factory {
          return cacheFactory[config] ?: throw NullPointerException("no such factory")
      }
  }
  
  enum class Config {
      XML,
      JSON,
      HTML,
      BASE64,
      TXT
  }
  
  abstract class Parser {
      abstract fun parse()
  }
  
  class XmlParser : Parser() {
      override fun parse() {
          println("parse xml")
      }
  }
  
  class JsonParser : Parser() {
      override fun parse() {
          println("parse json")
      }
  }
  
  class HtmlParser : Parser() {
      override fun parse() {
          println("parse html")
      }
  }
  
  class Base64Parser : Parser() {
      override fun parse() {
          println("parse base64")
      }
  }
  
  class TxtParser : Parser() {
      override fun parse() {
          println("parse txt")
      }
  }
  ```

* **源码中有哪些场景应用了工厂模式**

  1. `DateFormat.format`创建不同的`StringBuilder`对象

* **开发过程中有用过工厂模式吗**

  针对不同医院需要导出不同的excel表格，第一次的开发中使用了switch根据不同医院打印不同的excel表格，后来医院数量增加，决定重构代码，采用了**简单工厂（负责根据不同医院生成不同的打印策略对象）+策略者模式（打印excel表格的逻辑）+注解和反射（优化简单工厂缓存池的注入逻辑，不需要修改简单工厂的内部代码，靠注解来辨别策略类，靠反射来获取注解类实例）**来优化业务逻辑。

