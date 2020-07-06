## view的测量流程

1. **主要思路**

   **递归** （递的是`MeasureSpec` 归的是`measureWidth/Height`）

2. **主体函数**

   `[View.measure]` `[View.onMeasure]` `[View.setMeasuredDimension]`

3. **`ViewGroup.LayoutParams`**

   * 作用：用来告诉parent view布局的样式

   * `public static final int MATCH_PARENT = -1;`等于父布局大小-padding

   * `public static final int WRAP_CONTENT = -2;`要求视图足够大以适应内容大小	

4. **`MeasureSpec`**

   * 作用：因为子布局的测量是受限于父布局的大小宽高等，所以在测量子布局之前，必须知道父布局的布局要求，`measureSpec`的作用就是用来包装父布局传递到子布局的布局要求(size&mode)的一个容器。实际布局要求的描述是由32位的int值表示(前两位表示mode，后30位表示其测量大小)
   * `UNSPECIFIED`：父布局对子布局没有任何约束，子布局可以是任意大小
   * `EXACTLY`：父布局给定确定的值，子布局限定在此确定值内，比如父布局是`match_parent`或`20dp`
   * `AT_MOST`：子布局可以任意增大，直到其指定的大小，比如父布局是`wrap_content`

5. **`measure(int widthMeasureSpec,int heightMeasureSpec)`**

   * 作用：执行测量的函数，公共逻辑部分，final修饰，不能重写。
   * 如何开始：父布局调用`child.measure`，并将用32位int描述的布局要求传递给子布局。

6. **`onMeasure`**

   * 作用：真正执行测量的函数，开发者仅能重写该函数，实现自定义view的测量。

   * 默认测量策略：

     ```java
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

   * 作用：完成测量的函数，将`onMeasure`中测量的结果存储到`mMeasuredWidth`和`mMeasureHeight`

8. **`getMeasuredHeight/Width`**

   * 源码：

     ```java
     public static final int MEASURED_SIZE_MASK = 0x00ffffff;
     
     public final int getMeasuredWidth() {
         return mMeasuredWidth & MEASURED_SIZE_MASK;
     }
     ```

   * 作用：返回测量流程结束后的结果。

9. **递归简要流程：(虚线是递流程 实线是归流程)**

   A.measure ---> A.onMeasure --> A.measureChild ——> A.setMeasureDimension
               _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ ↓  ↑________________________________________________________________________________
             ↓                                                                                                            ↑
   B.measure --> B.onMeasure --> B.measureChild ——> B.setMeasureDimension
              _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ ↓  ↑_____________________________________________________________________________________
            ↓                                                                                                             ↑
   C.measure --> C.onMeasure ———————————> C.setMeasureDimension

10. **其他要点**

    * `getMeasuredHeight/Width`与`getHeight/Width`区别：

      ![示意图](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy85NDQzNjUtMzNkYjdiOWFhM2FkNWMzMi5wbmc)

    * 

    
