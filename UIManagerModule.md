# UIManagerModule

**UIManagerModule本身是个NativeModule**
这个类在每次UI转换时成UI更新时会用到。意味着每次渲染的时候没有中间状态
例如，一个js的应用更新控件A的背景视图成蓝色，b控件的宽度成100，这个
类都会用到，特别注意，这意味着所有单个转换的UI代码块更新都必须单个执行在UI线程，
执行成多个代码块可以允许平台的UI系统中断和渲染特殊的UI状态

为了实现这些操作，这个module的入队操作都是在NativeViewHierarchyManager的
转换结束后处理

CSSNode 为了允许布局和测量在一个非UI线程中，这个module也操作实时的CSSNodeDEPRECATED对象
对应于一个native视图，CSSNodeDEPRECATED也被允许通过他们的样式计算布局，计算的结果
x/y/width/height都是被作为一个布局的任务，将会被native层执行批处理，中间可能会创建
很多小对象在UIManageMode中，避免内存吃紧，所以用了对象池，如果视图没有改变不要批处理
结束的时候分发事件

# 初始化

对象创建的构造函数中会创建好 视图工具DisplayMetricsHolder、事件分发器、注册module中封装好的属性
、自定义的事件、各种控件的Manager

# Method

这个方法返回一个在视图层执行的对象、大多数方法直接由UIImplementation代理
`getUIImplementation`

这方法打算和FabricUIManager一起重用
`getViewManagerRegistry_DO_NOT_USE`

通常用于更新虚拟树上的值改变native层的属性达到动画的效果，直接是native层的更新，可能布局相关的数据不会被处理
你调用之前确保你知道你在做什么
`synchronouslyUpdateViewOnUIThread`


添加一个新的根节点，js可以返回一个带孩子节点的tag，来控制子view的添加移除，老的UIManagerModule中的才有，
新的Fabric模式中通过FabricUIManager方式创建，应该在getWidth()/getHeight()之后调用直接返回对象
`addRootView`

`createView`

`updateView`

js通过这个方法add/remove/removing/moving
`manageChildren`

`setChildren`

测量在视图屏幕上的位置
`measure`

测量在设备窗口上的位置，包括状态栏
`measureInWindow`

测量给定坐标的相对位置，返回的 x, y都是相对于根view的
`measureLayout`

从根节点找一个view，返回一个坐标，用于Element Inspector DevTool
`findSubviewIn`

`dispatchViewManagerCommand`

`dispatchCommand`

**标脏这个节点**
`invalidateNodeLayout`

**提交了UI改动相关的批处理在js->java的批处理执行完会被调用
，onBatchComplete()是Native和JS完成一次通讯后才会被调用**
`onBatchComplete`

`getEventDispatcher`

`addUIManagerEventListener`

`removeUIManagerEventListener`

通过传入的参数更新ReactShadowNode节点信息
`updateRootLayoutSpecs`

如果要想更新view的逻辑可以通过该方法插入一个代码块在UI执行前会执行
`prependUIBlock`

`receiveEvent`