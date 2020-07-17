# View的Measure流程

1. **主要思路**

   **遍历递归** （递的是`MeasureSpec` 归的是`measureWidth/Height`）

2. **主体函数**

   `View.measure()`，`View.onMeasure()`， `View.setMeasuredDimension()`

3. **`ViewGroup.LayoutParams`**

   * 作用：用来告诉parent view布局的样式

   * `public static final int MATCH_PARENT = -1;`等于父布局大小-padding

   * `public static final int WRAP_CONTENT = -2;`要求视图足够大以适应内容大小	

4. **`MeasureSpec`**

   * 作用：父布局对子布局的布局要求。因为子布局的测量是受限于父布局的大小宽高等，所以在测量子布局之前，必须知道父布局的布局要求，`measureSpec`的作用就是用来包装父布局传递到子布局的布局要求(size&mode)的一个容器。实际布局要求的描述是由32位的int值表示(前两位表示mode，后30位表示其测量大小)

   * `UNSPECIFIED`：父布局对子布局没有任何约束，子布局可以是任意大小

   * `EXACTLY`：父布局给定确定的值，子布局限定在此确定值内，比如父布局是`match_parent`或`20dp`

   * `AT_MOST`：子布局可以任意增大，直到其指定的大小，比如父布局是`wrap_content` 

   * 相关源码：

     ```java
     android/view/View.java
     sUseBrokenMakeMeasureSpec = targetSdkVersion <= Build.VERSION_CODES.JELLY_BEAN_MR1;
     public static class MeasureSpec {
         //移位位数位30位 因为
         private static final int MODE_SHIFT = 30;
         //0x3 16进制转为2进制为 11 右移30位为 110000...00
         //位掩码，用来与size和mode进行 & 运算 获取对应值
     	private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
         //0 右移30位 0000000...0000
         public static final int UNSPECIFIED = 0 << MODE_SHIFT;
         //1 右移30位 01000000...0000
         public static final int EXACTLY     = 1 << MODE_SHIFT;
         //2 右移30位 10000000...0000
         public static final int AT_MOST     = 2 << MODE_SHIFT;
         //生成MeasureSpec包装的32位int值
         public static int makeMeasureSpec(@IntRange(from = 0, to = (1 << MeasureSpec.MODE_SHIFT) - 1) int 										size,@MeasureSpecMode int mode) {
             //返回 前2位 为mode位 后30位 为 size布局大小 的int数值
         	if (sUseBrokenMakeMeasureSpec) {
                 //sdk<17时用相加的方法
             	return size + mode;
         	} else {
                 //sdk>17时，用位运算符
             	return (size & ~MODE_MASK) | (mode & MODE_MASK);
         	}
     	}
         //获取mode
         public static int getMode(int measureSpec) {
         	return (measureSpec & MODE_MASK);
     	}
         //获取size
         public static int getSize(int measureSpec) {
         	return (measureSpec & ~MODE_MASK);
         }
     }
     ```

5. **`measure(int widthMeasureSpec,int heightMeasureSpec)`**

   * 作用：执行测量的函数，公共逻辑部分，final修饰，不能重写。

   * 如何开始：`ViewRootImpl`调用`DecorView`的`measure`，并将 用32位int描述的布局要求向下传递。

   * 相关源码：

     ```java
     android/view/ViewRootImpl.java
     private void performTraversals() {
         ... ...
         //获取宽度布局要求
         int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
         //获取高度布局要求
     	int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
         performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
         ... ...
     }
     ```

     ```java
     android/view/ViewRootImpl.java
     //根据windowSize和LayoutParams计算MeasureSpec包装后的int值
     private static int getRootMeasureSpec(int windowSize, int rootDimension) {
         int measureSpec;
         switch (rootDimension) {
         case ViewGroup.LayoutParams.MATCH_PARENT:
             //window不可被调整，rootView被限定在window大小内
             measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
             break;
         case ViewGroup.LayoutParams.WRAP_CONTENT:
             //window可被调整为rootView的大小
             measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
             break;
         default:
             //window需要一个精确的值，rootView也是这个精确的值
             measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
             break;
         }
         //返回计算MeasureSpec包装后的int值
         return measureSpec;
     }
     ```

     ```java
     android/view/ViewRootImpl.java
     private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
         //执行DecorView.measure mView是在setView中赋值的，具体查看1KnowledgeBackground
         mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
     }
     ```

     ```java
     android/view/View.java
     public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
         ... ...
         onMeasure(widthMeasureSpec, heightMeasureSpec);
         ... ...
     }
     ```

6. **`onMeasure`**

   * 作用：真正执行测量的函数，开发者仅能重写该函数，实现自定义view的测量。

   * 默认测量策略：

     ```java
     android/view/View.java
     protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
         setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                 getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
     }
     
     protected int getSuggestedMinimumWidth() {
         //当view设置了minWidth或者background属性时，返回其属性的值，作为默认最小宽度
     	return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
     }
     
     public static int getDefaultSize(int size, int measureSpec) {
         int result = size;
         //获取父布局传过来的布局模式
         int specMode = MeasureSpec.getMode(measureSpec);
         //获取父布局传过来的布局大小
         int specSize = MeasureSpec.getSize(measureSpec);
         switch (specMode) {
         case MeasureSpec.UNSPECIFIED:
             //当且仅当父布局模式为UNSPECIFIED时，其宽高才会为getSuggestedMinimumWidth中返回的值
             result = size;
             break;
         case MeasureSpec.AT_MOST:
         case MeasureSpec.EXACTLY:
             //父布局模式为AT_MOST或EXACTLY时，默认都会以父布局的布局大小返回，也就是默认都是填充父布局。
             result = specSize;
             break;
         }
         return result;
     }
     ```

7. **`setMeasuredDimension(int measuredWidth, int measuredHeight)`**

   * 作用：完成测量的函数，并将`onMeasure`中测量的结果存储到`mMeasuredWidth`和`mMeasureHeight`

   * 相关源码：

     ```java
     android/view/View.java
     protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
        	... ...
         setMeasuredDimensionRaw(measuredWidth, measuredHeight);
     }
     
     private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
     	mMeasuredWidth = measuredWidth;
     	mMeasuredHeight = measuredHeight;
     }
     ```

8. **`getMeasuredHeight/Width`**

   * 作用：返回测量流程结束后的结果。

   * 相关源码：
   
     ```java
     android/view/View.java
     public static final int MEASURED_SIZE_MASK = 0x00ffffff;
     
     public final int getMeasuredWidth() {
         return mMeasuredWidth & MEASURED_SIZE_MASK;
     }
     
     public final int getMeasuredHeight() {
         return mMeasuredHeight & MEASURED_SIZE_MASK;
    }
     ```

9. **递归简要流程：(虚线是递流程 实线是归流程)**

   ​	![](https://upload-images.jianshu.io/upload_images/13218197-8684ab4491714eee.png?imageMogr2/auto-orient/strip|imageView2/2/w/766/format/webp)
   
10. **其他要点**

    * `getMeasuredHeight/Width`与`getHeight/Width`区别：

      ![示意图](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDQzNjUtMzNkYjdiOWFhM2FkNWMzMi5wbmc)

11. **系列文章**

    1. [View的背景知识](1KnowledgeBackground.md)
    2. [View的测量流程](2Measure.md)
    3. [View的布局流程](3Layout.md)
    4. [View的绘制背景知识](4DrawBackground.md)
    5. [View的绘制流程](5Draw.md)
    6. [View的三大绘制流程总结](6Conclusion.md)
    7. [View的事件分发机制](7Event.md)

