### 装饰者模式

* **什么是装饰者模式（解决了什么问题）**

  装饰：对某样事物进行点缀，或增加装饰品。

  举个例子，奶茶加珍珠，本质上还是奶茶，但加了珍珠作为装饰品，增强了奶茶的口感。珍珠奶茶加布丁，可以在上面的基础上再加布丁，仍然是奶茶，增加了布丁，增强了奶茶的口感。

  上面例子有三点是装饰者模式与其他模式最大的区别：

  1. 奶茶加了珍珠仍然是奶茶。
  2. 奶茶增加珍珠或布丁，增强了奶茶的口感。
  3. 在加了珍珠的奶茶上，还能继续加布丁，再次增强奶茶的口感。

  转为程序语言，装饰者模式，通过继承表明事物的本质，通过组合增强方法，通过构造器传入事物，方便嵌套。

* **代码实现**

  ```kotlin
  abstract class InputStream{
      abstract fun read()
      abstract fun skip(n:Long)
      abstract fun available()
      abstract fun close()
  }
  
  class FileInputStream : InputStream(){
      override fun read() {
          println("file read")
      }
  
      override fun skip(n: Long) {
          println("file skip")
      }
  
      override fun available() {
          println("file available")
      }
  
      override fun close() {
          println("file close")
      }
  }
  
  /**
   * 装饰者模式
   * 构造方法传入抽象类
   * 并组合一个抽象类实例，
   */
  class BufferedInputStream(private val ins:InputStream):InputStream(){
      override fun read() {
          /**在抽象类实例方法前，增加自己的功能，增强方法*/
          println("read from buffered")
          ins.read()
      }
  
      override fun skip(n: Long) {
          ins.skip(n)
      }
  
      override fun available() {
          ins.available()
      }
  
      override fun close() {
          ins.close()
      }
  }
  
  fun main(){
      val inputStream:InputStream = FileInputStream()
      val bufferedInputStream = BufferedInputStream(inputStream)
      bufferedInputStream.read()
  }
  ```

* **源码中的应用场景**

  `java io`，`InputStream`，`BufferedInputStream`为`InputStream`增加了缓存。

* **装饰者模式与代理模式的区别**

  装饰者模式侧重于**动态添加功能**（使用者需要明确添加什么功能），代理模式侧重于**信息隐藏**（使用者不用知道这个过程是怎么样的）。