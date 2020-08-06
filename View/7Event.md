# View的事件分发机制

1. **主要思路**

   **递归(责任链)将事件自顶向下传递**

2. **关键类**

   |                            相关类                            |                         作用                         |
   | :----------------------------------------------------------: | :--------------------------------------------------: |
   |          `InputEvent`（`KeyEvent` & `MotionEvent`）          |                     输入事件描述                     |
   |                        `InputManager`                        |                    系统输入管理器                    |
   |                        `ViewRootImpl`                        | 连接`WindowManager`与`DecorView`的纽带，控制事件分发 |
   |      `InputEventReceiver`（`WindowInputEventReceiver`）      |                 接收并消费或分发事件                 |
   | `InputStage`（`ViewPreImeInputStage` `EarlyPostImeInputStage` `ViewPostImeInputStage` `SyntheticInputStage` `AsyncInputStage` `NativePreImeInputStage` `ImeInputStage` `NativePostImeInputStage` ） |  事件分发的不同阶段，组成责任链，按顺序分发处理事件  |
   |                         `DecorView`                          |                                                      |
   |                          `Activity`                          |                                                      |
   |                           `Window`                           |                                                      |
   
3. `InputEvent`

   * 作用：