# 背景知识

1. **`Window`**

2. **`ViewRoot`(实际实现类是`android.view.ViewRootImpl`)**

   * 作用：`View`的顶层结构，实现了`View`与`WindowManager`的通信，完成了`View`的三大流程

   * 相关源码：

     ```java
     private void performTraversals() {
         ... ...
         //完成view的measure流程
         performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
         //完成view的layout流程
         performLayout(lp, mWidth, mHeight);
         //完成view的draw流程
         performDraw();
     }
     ```

3. **`DecorView`**

4. **`Activity`**



