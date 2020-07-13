# View的Layout流程

1. **主要思路**

   **ViewGroup的遍历**（从`DecorView`开始遍历子`View`，执行`layout`）

2. **主体函数**

   `View.layout()`，`View.onLayout()`，`View.setFrame()`

3. **`layout(int l, int t, int r, int b)`**

   * 作用：为自身及其子`View`分配大小与位置

   * 如何开始：`ViewRootImpl`在`performTravesals`中调用`DecorView.layout()`

   * 相关源码：

     ```java
     android/view/ViewRootImpl.java
     private void performTraversals() {
         performLayout(lp, mWidth, mHeight);
     }
     private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
             int desiredWindowHeight) {
         final View host = mView;
         //执行DecorView的layout方法，传入的是measuredWidth,measuredHeight 因此得先measure，后layout
         host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
     }
     ```
     
     ```java
     android/view/View.java
     public void layout(int l, int t, int r, int b) {
         //判断是否需要重新测量
         if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
             onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
             mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
         }
         //存储之前的l t r b信息
         int oldL = mLeft;
     	int oldT = mTop;
     	int oldB = mBottom;
     	int oldR = mRight;
         //确定布局是否发生了变化，并存储新的l t r b
     	boolean changed = isLayoutModeOptical(mParent) ?
             	setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
        	if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
             //执行布局相应的onLayout()
            	onLayout(changed, l, t, r, b);
             ListenerInfo li = mListenerInfo;
         	if (li != null && li.mOnLayoutChangeListeners != null) {
             	ArrayList<OnLayoutChangeListener> listenersCopy =
                     (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
             	int numListeners = listenersCopy.size();
             	for (int i = 0; i < numListeners; ++i) {
                 	//当布局发生变化时，通知观察者布局变化通知
                 	listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
             	}
         	}
         }
     }
     ```

4. **`setFrame(int left, int top, int right, int bottom)`**

   * 作用：存储传入的`left`，`top`，`right`，`bottom`信息 并确定布局是否发生变化，如果发生变化，返回true

   * 相关源码：

     ```java
     android/view/View.java
     protected boolean setFrame(int left, int top, int right, int bottom) {
         boolean changed = false;
         //如果传入的位置信息跟之前的位置信息不同
         if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
             //则说明布局发生改变
     		changed = true;
             //重新存储位置信息
             //这里注意一点，mLeft,mTop,mRight,mBottom都是相对位置，不是坐标系的绝对位置
             //是相对于父view的位置大小
             mLeft = left;
     		mTop = top;
     		mRight = right;
     		mBottom = bottom;
         }
         //返回布局是否发生改变
         return changed;
     }
     ```

5. **`onLayout(boolean changed, int left, int top, int right, int bottom)`**

   * 作用：确定子`View`的位置与大小，所以，如果是`View`，`onLayout`是空实现，而如果是`ViewGroup`，`onLayout`方法是抽象，必须有具体的子类根据不同策略来实现。

   * 相关源码：

     ```java
     android/view/View.java
     //空实现
     protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
     }
     ```

     ```java
     android/view/ViewGroup.java
     //抽象方法
     protected abstract void onLayout(boolean changed,
                 int l, int t, int r, int b);
     ```

6. `getWidth()/getHeight()`

   * 作用：`setFrame()`存储布局传入的信息，`View`的宽高，也是最终的宽高。

   * 相关源码：

     ```java
     public final int getWidth() {
         //根据布局传入的right - left得到宽
         return mRight - mLeft;
     }
     public final int getHeight() {
         //根据布局传入的bottom - top得到高
         return mBottom - mTop;
     }
     ```

     

7. **例子**

   * 以`LinearLayout`来举例`onLayout`在`ViewGroup`中是如何实现的

   * 相关源码：

     ```java
     android/widget/LinearLayout.java
     protected void onLayout(boolean changed, int l, int t, int r, int b) {
         //根据LinearLayout的方向来决定子view的布局
         if (mOrientation == VERTICAL) {
             layoutVertical(l, t, r, b);
         } else {
             layoutHorizontal(l, t, r, b);
         }
     }
     
     void layoutVertical(int left, int top, int right, int bottom) {
         //获取子view个数
         final int count = getVirtualChildCount();
         //遍历子view
         for (int i = 0; i < count; i++) {
         	//这里我们不关注子view的left top right bottom是如何计算的 主要关注整体布局流程
         	setChildFrame(child, childLeft, childTop + getLocationOffset(child),
             	childWidth, childHeight);
         }
     }
     
     private void setChildFrame(View child, int left, int top, int width, int height) {
         //执行child.layout，继续完成子类的布局设置
         child.layout(left, top, left + width, top + height);
     }
     ```

8. **简要流程**

   ​		![](https://upload-images.jianshu.io/upload_images/13218197-0a23ec6a57f1eb88.png?imageMogr2/auto-orient/strip|imageView2/2/w/1028/format/webp)

9. **其他要点**

   * `getWidth/getHeight`是在`setFrame()`中得到的，也就是在`layout`流程后才能获取到，也是`View`的最终宽高。
   * `layout`流程中，用到的`left`，`top`，`right`，`bottom`都是相对于父`View`的位置，不是坐标系的绝对位置。