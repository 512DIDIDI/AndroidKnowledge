# View的绘制背景知识

1. **`Bitmap`**
   * 作用：像素位图，是以像素为单位组成的图像，像素点可根据不同的排列和颜色(A R G B)以构成图像。因此，bitmap要点之处不外乎像素排列方式，像素压缩编码格式。

   * `Bitmap`占用内存计算(byte) = 图片长度(像素) * 图片宽度(像素) * 每个像素点占用的字节数(byte)

   * 注意：改变图片宽高，像素存储方式，会影响占用内存大小，

     ​			但质量压缩，如`JPEG`压缩算法，只会减小文件大小，而解码为`Bitmap`加载到内存占用大小是不变的。

   * `Bitmap.Config`：描述像素的存储方式，通过改变A R G B每个像素点占用的字节数来压缩内存。
     
     * `ALPHA_8`：1byte，仅有alpha通道，不存储颜色信息。
     * `RGB_565`：2byte，仅有RGB通道，其中R占5位(32种R精度)，G占6位(64种G精度)，B占5位(32种B精度)，图片可能会偏绿，可使用`Dither`添加抖动处理，如果需要不透明位图，它比`ARGB_4444`的图片质量更高。
     * `ARGB_4444`：2byte，**不建议使用，在android4.4以上使用此配置会默认替换为`ARGB_8888`**，A R G B各占4位(16种精度)。
     * `ARGB_8888`：4byte，A R G B各占8位(256种精度)，是常见存储方式中图片质量最高的像素存储方式。
     * `RGBA_F16`：8byte，适用于广色域和HDR内容。
     * `HARDWARE`：当位图仅存储在图形内存中，如果位图的唯一操作是绘制到屏幕上，可以考虑使用它。
     
   * `Bitmap.CompressFormat`：以`JPEG`，`PNG`，`WEBP`进行图片质量压缩，在保持像素的前提下，通过改变图片位深及透明度来达到图片压缩的目的，因此**只会改变生成的文件大小，并不会改变解码为`Bitmap`加载到内存中占用的内存大小。**

   * `BitmapFactory`：根据文件，流或字节数组创建`Bitmap`

     * `Options`
       1. `inDensity`：图片本身的像素密度（其实就是图片资源所在的哪个密度文件夹下，如在`xxhdpi`下就是480，如果在`asstes`、网络图片或`sd`卡下，默认是160`dpi`）
       2. `inTargetDensity`：图片最终在`bitmap`里的像素密度，如果没有赋值，会将`inTargetDensity`设置成`inScreenDensity`
       3. `inScreenDensity`：手机本身的屏幕密度
       4. `inScaled`  = `TargetDensity`／`inDensity`

2. **`Canvas`**

   * 定义：`Canvas`，是一种绘制时的规则，是`Android`2D图形绘制的基础。

   * 作用：规定绘制内容时的规则 & 内容，绘制内容是根据`Canvas`的规则绘制在屏幕上的，`Canvas`只是绘制时的规则，实际内容是绘制在屏幕上的。

   * 其他：

     * 当调用`Canvas.save()`/`Canvas.restore()`会进行图层以及画布状态的出入栈。

     ![状态栈](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS05NGMxMGYwNzMxOTExYmVhLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

     * `Canvas`是由多个图层构成的

       > 1. 在画布上操作 = 在图层上操作
       > 2. 如无设置，绘制操作和画布操作是默认在默认图层上进行
       > 3. 在通常情况下，使用默认图层就可满足需求；若需要绘制复杂的内容（如地图），则需使用更多的图层
       > 4. 最终显示的结果 = 所有图层叠在一起的效果

     ![画布构成 - 图层](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS05YWFjOTYxOTBiYzBhNTMzLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)

3. **`Drawable`**

   * 作用：可绘制物体的抽象层面，其子类表示可以绘制在`Canvas`上的对象。设置背景，点击效果，形状，动画等时常会用到其子类。不过注意的是，他不像`View`，不能进行事件分发和响应用户操作，大多数都是可以通过`res/drawable`中`xml`进行配置的。

4. **其他要点**

   * 屏幕点密度`dpi`计算公式：手机对角线像素(Pixel) / 手机对角线长度(英寸 Inch)

   * `px`：一个像素点的长度

   * `dp`：在`dpi`=160时，1`px`的长度，最后根据实际设备的`dpi`，进行像素缩放。因此 `px` = `dp` * (`dpi` / 160)

   * `sp`：文本大小单位，与`dp`同理，默认情况下，等同于`dp`大小，仅能用于文字大小单位。

   * `res`资源目录对应关系：

     | 匹配的资源目录 |   屏幕点密度(dpi)    | 缩放比(对比`mdpi`) |
     | :------------: | :------------------: | :----------------: |
     |     `ldpi`     |         120          |        0.75        |
     |     `mdpi`     | 160(基准dpi，默认值) |        1.0         |
     |     `hdpi`     |         240          |        1.5         |
     |    `xhdpi`     |         320          |        2.0         |
     |    `xxhdpi`    |         480          |        3.0         |
     |   `xxxhdpi`    |         640          |        4.0         |
     |    `nodpi`     |         ALL          |         /          |
     |    `tvdpi`     |   ≈213dpi 适用于TV   |       ≈1.33        |

   * `drawble`与`mipmap`文件夹

     * 如果在`app`内使用资源，那么放在`drawable`或`mipmap`目录下，系统只会根据上表`dpi`加载匹配的资源目录下的资源。
     * 如果是在`launcher`中，那么放在`mipmap`下的资源，`launcher`会自动选择更加合适密度的资源。
     * 因此，应用图标`launcher icon`必须放在`mipmap`文件夹下，且必须保证不同密度的`icon`。
     * 其他资源随意放在哪个文件夹下，如果有不同密度的图那最好，但如果仅有一张图，优先放在高密度(如`xxhdpi`)文件夹下，可以避免 高密度设备 使用 放在低密度文件夹下的图片 放大宽高 造成的占用内存增加。
     * 如果是网络图片，assets文件夹底下或者手机本地文件夹加载的图片，都会默认使用`mdpi`也就是160`dpi`的像素密度，如果想改变其缩放比，必须同时调整`BitmapFactory.Options.inDensity`和`BitmapFactory.Options.inTargetDensity`的值才能起作用。
     * 例子：一个30 * 30的ARGB_8888图片放在 `hdpi`文件夹下，480`dpi`的设备访问这张图，那么首先30 * 30的图片会被放大成 30 * ( 3 / 2)  * 30 * ( 3 / 2) =  45 * 45 大小的图片 最终加载成`bitmap`占用内存的大小就是 45 * 45 * 4  = 8100(byte)

5. [绘制图形基础以及各api使用](https://github.com/512DIDIDI/ViewDemo)

