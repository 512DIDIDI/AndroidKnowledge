# 背景知识

1. **`Activity`**

   * 作用：负责生命周期管理与事件处理，每个`Activity`组合了一个`Window`，实际视图控制是交由`Winodw`管理ui排版。是为了满足多窗口管理和傻瓜式视图管理的需要而诞生的。

   * 起源：在`Activity`中的`onCreate()`方法中会调用`setContentView()`加载自定义视图，实际调用的是`Window`的`setContentView`方法

   * 相关源码：

     ```java
     android/app/Activity.java
     public void setContentView(@LayoutRes int layoutResID) {
         //调用mWindow.setContentView()
         getWindow().setContentView(layoutResID);
         initWindowDecorActionBar();
     }
     ```

     ```java
     android/app/Activity.java
     final void attach(... ...){
     	... ...
         //实例化mWindow
     	mWindow = new PhoneWindow(this, window, activityConfigCallback);
         //实现window的接口，键盘触摸 ui策略等
     	mWindow.setWindowControllerCallback(this);
     	mWindow.setCallback(this);
     	mWindow.setOnWindowDismissedCallback(this);
     	... ...
     }
     ```

2. **`Window`**

   * 作用：唯一实现类`PhoneWindow`，创建了顶层视图`DecorView`，`DecorView`作为父布局用来加载`Activity`中`setContentView()`方法传入的`layoutRes`。是为了管理ui排版，视图控制而诞生的。

   * 相关源码：

     ```java
     com/android/internal/policy/PhoneWindow.java
     @Override
     public void setContentView(int layoutResID) {
         if (mContentParent == null) {
             //创建DecorView，并实例化DecorView布局内的ViewGroup控件mContentParent
             installDecor();
         } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
             mContentParent.removeAllViews();
         }
     
         if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
             final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                     getContext());
             transitionTo(newScene);
         } else {
             //将mContentParent作为父控件，加载layoutResID
             mLayoutInflater.inflate(layoutResID, mContentParent);
         }
     	... ...
     }
     ```

     ```java
     com/android/internal/policy/PhoneWindow.java
     private void installDecor() {
         ... ...
         if (mDecor == null) {
             //创建DecorView
             mDecor = generateDecor(-1);
           	... ...
         }
         ... ...
         if (mContentParent == null) {
             //实例化mContentParent
         	mContentParent = generateLayout(mDecor);
             ... ...
         }
     }
     ```

     ```java
     com/android/internal/policy/PhoneWindow.java
     protected DecorView generateDecor(int featureId) {
         ... ...
         //创建DecorView
         return new DecorView(context, featureId, this, getAttributes());
     }
     ```

     ```java
     com/android/internal/policy/PhoneWindow.java
     public static final int ID_ANDROID_CONTENT = com.android.internal.R.id.content;
     protected ViewGroup generateLayout(DecorView decor) {
         ... ...
         int layoutResource;
         if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
             layoutResource = R.layout.screen_swipe_dismiss;
             ... ...
         } 
         ... ...
         else {
         	layoutResource = R.layout.screen_simple;
         }
         ... ...
         //加载DecorView布局
         mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
         //从DecorView中获取id为content的控件
     	ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
         ... ...
         //将contentParent返回
         return contentParent;
     }
     ```

     ```java
     android/view/Window.java
     public <T extends View> T findViewById(@IdRes int id) {
         //返回DecorView中给定id的视图控件
         return getDecorView().findViewById(id);
     }
     ```

     ```xml
     frameworks/base/core/res/res/layout/screen_simple.xml
     <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
     	android:layout_width="match_parent"
     	android:layout_height="match_parent"
     	android:fitsSystemWindows="true"
     	android:orientation="vertical">
     	<ViewStub android:id="@+id/action_mode_bar_stub"
     		android:inflatedId="@+id/action_mode_bar"
     		android:layout="@layout/action_mode_bar"
     		android:layout_width="match_parent"
     		android:layout_height="wrap_content"
     		android:theme="?attr/actionBarTheme" />
     
     	<FrameLayout
     		android:id="@android:id/content"
     		android:layout_width="match_parent"
     		android:layout_height="match_parent"
     		android:foregroundInsidePadding="false"
     		android:foregroundGravity="fill_horizontal|top"
     		android:foreground="?android:attr/windowContentOverlay" />
     </LinearLayout>
     ```

3. **`DecorView`**

   * 作用：继承自`FrameLayout`，是Android视图树的根视图，`View`层的事件都会先经过`DecorView`，再分发到下面的`View`

   * 相关源码：

     ```java
     com/android/internal/policy/DecorView.java
     void onResourcesLoaded(LayoutInflater inflater, int layoutResource) {
         mDecorCaptionView = createDecorCaptionView(inflater);
         //加载PhoneWindow生成的layoutResource
         final View root = inflater.inflate(layoutResource, null);
         if (mDecorCaptionView != null) {
             if (mDecorCaptionView.getParent() == null) {
                 addView(mDecorCaptionView,
                         new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
             }
             mDecorCaptionView.addView(root,
                     new ViewGroup.MarginLayoutParams(MATCH_PARENT, MATCH_PARENT));
         } else {
     		//添加到DecorView中
             addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
         }
         mContentRoot = (ViewGroup) root;
     }
     ```

4. **`WindowManager`**

   * 作用：在`Activity`生命周期`onResume`之后，绑定`DecorView`与`ViewRootImpl`。

   * 相关源码：

     ```java
     android/app/ActivityThread.java
     public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
             String reason) {
        	... ...
         //执行activity的onResume方法
         final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);
         ... ...
        	final Activity a = r.activity;
        	if (r.window == null && !a.mFinished && willBeVisible) {
             //获取activity的window实例
     		r.window = r.activity.getWindow();
             //获取window中实例化的DecorView
     		View decor = r.window.getDecorView();
     		decor.setVisibility(View.INVISIBLE);
             //获取windowManager,获取的是WindowManager的子类，WindowManagerImpl
     		ViewManager wm = a.getWindowManager();
     		WindowManager.LayoutParams l = r.window.getAttributes();
     		a.mDecor = decor;
     		if (a.mVisibleFromClient) {
     			if (!a.mWindowAdded) {
     				a.mWindowAdded = true;
                     //windowManager调用addView添加DecorView
     				wm.addView(decor, l);
     			}
                 ... ...
     		}
     	}
         ... ...
     }
     ```

     ```java
     android/view/WindowManagerImpl.java
     public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
         mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
     }
     ```

     ```java
     android/view/WindowManagerGlobal.java
     public void addView(View view, ViewGroup.LayoutParams params,
             Display display, Window parentWindow) {
         ViewRootImpl root;
         //实例化ViewRootImpl
         root = new ViewRootImpl(view.getContext(), display);
         //保存decorView对象
         mViews.add(view);
         //保存viewRootImpl对象
         mRoots.add(root);
         //保存decorView的layout params
         mParams.add(wparams);
         try {
             //绑定ViewRootImpl与DecorView
             root.setView(view, wparams, panelParentView);
         }... ...
     }
     ```

5. **`ViewRoot`(实际实现类是`android.view.ViewRootImpl`)**

   * 作用

     1. 连接`DecorView`与`WindowManager`，也可以说是`Window`与`DecorView`的纽带
     2. 通过`DecorView`完成`View`的三大流程绘制
     3. 用责任链模式，向`DecorView`分发用户的`InputEvent`事件

   * 注意：在`Activity.onResume()`之后，`ViewRootImpl.setView`中将`DecorView`传入，并在之后执行`requestLayout()`通过`DecorView`递归执行`View`的三大流程，所以`View`的三大流程都是在`Activity.onResume`之后。

   * 相关源码：

     ```java
     android/view/ViewRootImpl.java
     final IWindowSession mWindowSession;
     
     public ViewRootImpl(Context context, Display display) {
         //WindowManagerGlobal获取IWindowSession
         mWindowSession = WindowManagerGlobal.getWindowSession();
     }
     
     public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
         synchronized (this) {
             if (mView == null) {
                 //这个mView就是DecorView，view的三大绘制流程均是通过mView来递归的
                 mView = view;
                 ... ...
                 //view的三大绘制流程
                 requestLayout();
                 //创建InputChannel
                 mInputChannel = new InputChannel();
                 //wms根据当前的window创建了SocketPair用于跨进程通信，并对传入的mInputChannel进行了注册
                 //此后ViewRootImpl中的mInputChannel就指向了正确的InputChannel
                 //client端与server端就能进行双向通信了
                 res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
            				 	getHostVisibility(), mDisplay.getDisplayId(), mTmpFrame,
             				mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
             				mAttachInfo.mOutsets, mAttachInfo.mDisplayCutout, mInputChannel,
             				mTempInsets);
                 //创建WindowInputEventReceiver，处理事件分发
                 mInputEventReceiver = new WindowInputEventReceiver(mInputChannel,
                               Looper.myLooper());
                 //组装InputStage责任链
                 mSyntheticInputStage = new SyntheticInputStage();
                 InputStage viewPostImeStage = new ViewPostImeInputStage(mSyntheticInputStage);
                 InputStage nativePostImeStage = new NativePostImeInputStage(viewPostImeStage,
                             "aq:native-post-ime:" + counterSuffix);
                 InputStage earlyPostImeStage = new EarlyPostImeInputStage(nativePostImeStage);
                 InputStage imeStage = new ImeInputStage(earlyPostImeStage,
                             "aq:ime:" + counterSuffix);
                 InputStage viewPreImeStage = new ViewPreImeInputStage(imeStage);
                 InputStage nativePreImeStage = new NativePreImeInputStage(viewPreImeStage,
                             "aq:native-pre-ime:" + counterSuffix);
                 mFirstInputStage = nativePreImeStage;
                 mFirstPostImeInputStage = earlyPostImeStage;
             }
         }
     }
     ```
     
     * `View`的三大绘制流程入口
     
     ```java
     android/view/ViewRootImpl.java
     public void requestLayout() {
         if (!mHandlingLayoutInLayoutRequest) {
             checkThread();
             mLayoutRequested = true;
             //这里执行view的绘制流程
             scheduleTraversals();
         }
     }
     
     final class TraversalRunnable implements Runnable {
     	@Override
     	public void run() {
     		doTraversal();
     	}
     }

     final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
     
     void scheduleTraversals() {
     	if (!mTraversalScheduled) {
             ... ...
     		mChoreographer.postCallback(
     			Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
             ... ...
     	}
     }
     
     void doTraversal() {
     	if (mTraversalScheduled) {
     		... ...
     		performTraversals();
             ... ...
     	}
     }
     
     private void performTraversals() {
         ... ...
         //measure流程
         performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
         //layout流程
         performLayout(lp, mWidth, mHeight);
         //draw流程
         performDraw();
     }
     
     private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
     	try {
             //执行DecorView的mesaure流程
     		mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
     	} finally {
     		Trace.traceEnd(Trace.TRACE_TAG_VIEW);
     	}
     }
     
     private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
                 int desiredWindowHeight) {
         final View host = mView;
         try {
             //执行DecorView的layout流程
     		host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
         }
     }
     
     private void performDraw() {
         ... ...
         try {
     		boolean canUseAsync = draw(fullRedrawNeeded);
         }
         ... ...
     }
     
     private boolean draw(boolean fullRedrawNeeded) {
         ... ...
         if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset,
             scalingRequired, dirty, surfaceInsets)) {
         	return false;
     	}
         ... ...
     }
     
     private boolean drawSoftware(Surface surface, AttachInfo attachInfo, int xoff, int yoff,
             boolean scalingRequired, Rect dirty, Rect surfaceInsets) {
         ... ...
         //执行DecorView的draw流程
         mView.draw(canvas);
         ... ...
     }
     ```
     

6. **综上**

   1. 创建`DecorView`的流程：
      * `Activity`在`attach`时，会创建`PhoneWindow`实例`mWindow`;
      * `Activity`在`onCreate`中调用`setContentView`方法时，会间接调用`mWindow.setContentView`;
      * `PhoneWindow`在`setContentView`中会创建`DecorView`的实例，并将`Activity`传入的布局加载到`DecorView`中`id`为`content`的`mContentParent`中;
      * `DecorView`是android视图树的根节点，最底层的`View`，在`PhoneWindow`创建布局时`generateLayout`会根据不同类型的主题创建布局，其中会有一个`id`为`content`的`ViewGroup`，`Activity`中传入的布局文件就是以`content`为父布局;
   2.  建立`DecorView`与`WindowManager`的联系并最终绘制显示的流程：
      * `ActivityThread`在调用`handleResumeActivity`，会执行`Activity`的`onResume`方法，随后，会创建`ViewRootImpl`的实例，获取`Activity`的`Window`实例，并获取`Window`中的`DecorView`实例，使用`WindowManager`，将`ViewRootImpl`与`DecorView`绑定;
      * `ViewRootImpl`获取到`DecorView`的实例后，会持有该引用，并分步调用`mView`的`measure`,`layout`,`draw`方法。
   3. 通过责任链模式，向`DecorView`分发用户的`InputEvent`事件(具体细节查看[View的事件分发机制](7Event.md))
      * 创建`InputChannel`，并通过`Binder`在`SystemServer`进程中完成`InputChannel`的注册。
      * 创建`WindowInputEventReceiver`来处理事件分发。
      * 组装`InputStage`责任链，负责不同`InputEvent`事件的处理。
   
7. **系列文章**

   1. [View的背景知识](1KnowledgeBackground.md)
   2. [View的测量流程](2Measure.md)
   3. [View的布局流程](3Layout.md)
   4. [View的绘制背景知识](4DrawBackground.md)
   5. [View的绘制流程](5Draw.md)
   6. [View的三大绘制流程总结](6Conclusion.md)
   7. [View的事件分发机制](7Event.md)



