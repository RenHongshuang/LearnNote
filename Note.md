# LearnNote

## Java 内存分配模型

JVM 将整个内存划分为了几块，分别如下所示：

- 1）、方法区（元空间）：存储类信息、常量、静态变量等。=> 所有线程共享
- 2）、虚拟机栈：存储局部变量表、操作数栈等。
- 3）、本地方法栈：不同与虚拟机栈为 Java 方法服务、它是为 Native 方法服务的。
- 4）、堆：内存最大的区域，每一个对象实际分配内存都是在堆上进行分配的，，而在虚拟机栈中分配的只是引用，这些引用会指向堆中真正存储的对象。此外，堆也是垃圾回收器（GC）所主要作用的区域，并且，内存泄漏也都是发生在这个区域。=> 所有线程共享
- 5）、程序计数器：存储当前线程执行目标方法执行到了第几行。

## Java 中的锁

1.syncnorized 四种状态

无锁

偏向锁 ：CAS操作写入ThreadId 和时间戳

轻量级锁：

重量级锁：

2.可重入锁

3.读写锁

4.可重入读写锁

5.同lock接口，condition 接口，locksupport 接口以及抽象同步器实现的自定义锁

## 3.dp的理解

160dp 在160dpi的设备上 显示160px
160dp 在240dpi的设备上 显示240px

字节跳动适配方案：

https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA

## ViewModel 

1.基于工厂模式创建,fragment的创建恢复过程相对更加复杂一些

2.存在哪的问题

ViewModel 缓存在ViewModelStore中的Hashmap中， 而ViewmodelStore 存在NonConfigurationInstance中，而NonConfigurationInstance 存在ActivityThread中.

3.ViewModel只有在ConfigurationChange flag 为true 并且Lifecycle state 是Destroyed的情况下才会不销毁



## Lifecycle 

![image.png](https://user-gold-cdn.xitu.io/2019/5/12/16aac16b003a7bff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![Lifecycle-Seq2.png](https://user-gold-cdn.xitu.io/2019/5/12/16aac16b4e1ae7af?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## LiveData

1.LiveData基于观察者模式.

2.LiveData能感知activity/fragment 声明周期变化，默认只有在active状态才能回调通知状态改变

3.setValue只能在主线程中调用，postValue可以在子线程中调用



## 什么是文件描述符

https://mp.weixin.qq.com/s/R3Z-xWDYdUdYtjrTir4OtA

Linux 中一切都可以看作文件，包括普通文件、链接文件、Socket 以及设备驱动等，对其进行相关操作时，都可能会创建对应的文件描述符。

文件描述符（file descriptor）是内核为了高效管理已被打开的文件所创建的索引，用于指代被打开的文件，对文件所有 I/O 操作相关的系统调用都需要通过文件描述符



## 热更新原理

分类：

**1.代码修复**
类加载方案 :利用mutidex 和类加载机制 将patch.dex 放到 elment数组的头部
Instant Run方案 ：通过ASM字节码操控框架 动态的改变类的行为
底层替换方案：nativie 层 替换ArtMethod结构体

**2.资源修复** 

instant run方案：
1.创建new AssetManager ,通过反射调用addAssetPath方法加载外部资源
2.将resourceManager或ActvityThread(取决于sdk >19) 中的assetmanager类型的mAssert字段引用替换成新建的AssertManager(hook技术)

**3.动态链接库修复** 
原理需要了解so加载过程原理
方法1.将so 补丁插入到NativeLibraryElement数组的前部，优先加载
方法2.调用system的load方法接管so的加载入口


## 插件化技术

1.Hook 技术
a.Hook 点：容易找到并且不易变化的对象 --> 静态变量或者单例 
b.hook方法：反射和代理 hook java层
动态代理：运行时通过反射动态的生成代理对象 java靠实现InvocationHander 接口 重写invoke方法

2.Hook Activity

创建代理类在activity中的attachBaseContext 替换 activity  或者 activity thread 中的Instrumention静态成员变量

3.插件化

**1.Activity  插件化**

a.Hook iActivityManager  

b.Hook Instrumentation 

 预先占坑的方式解决没有在Androidmanifest.xml声明的问题.骗过ams校验，，之后替换占坑activity

**2.service的 插件化**

Hook  IActivityManager

用代理Service占坑，在targetService已经加载的情况下 ，通过hook IActivityManager，让宿主启动service 都先启动代理service，然后代理service 通过反射ActivityManager拿到已经加载的target Service 并反射调用attach 方法。

**3.ContentProvider  插件化**

与2类似 .

a.加载插件

b.启动插件的contentProvider需要hook IContentProvider ,调用contentProvider 全都指向占坑的代理conten Provider

c.代理contentProvider 根据uri解析auth 用来匹配插件的content provider .然后启动它的oncreate 

**4.BroadcastReceiver 插件化**

统一将插件所有静态广播，转为动态注册的广播，具体方法是通过classloader 创建广播对象，然后调用宿主的registerReceive方法。

**5.资源的插件化**

两种方式

1.合并资源,将插件资源添加到宿主resources中，这种方案插件能访问宿主资源

a.调用ResourcesManager 的createResource方法，得到包含宿主的AssetManager

b.反射调用AssetManager的addAssertPath 添加插件Resource，得到新的Resource

c.通过Hook ResourcesManager替换

2.每个插件独立构造资源

创建AssetManager,然后反射调用它的addAssetPath方法将插件的Resource

//这块书里没说明白，具体实现需要看vituralapk的源码

**6.so的插件化**

跟热更新类似，合并宿主和插件的DexElement，然后通过反射替换



## Binder 通信

![1590376806481](C:\Users\e-HongShuang.Ren\AppData\Roaming\Typora\typora-user-images\1590376806481.png)

## 性能优化

参考:https://mp.weixin.qq.com/s/3QhMGVIcR1yW3xweJCa-9Q  ,json chao ,张绍文的课程

**1.包体积优化**

1.图片转成webp格式

2.用link检查去掉无用代码，

3.gradle中配置shinkResource去掉无用资源

4.去掉无用的lib.so .留下armapi-v8一般就可以

5.开启混淆

![1591065913671](C:\Users\e-HongShuang.Ren\AppData\Roaming\Typora\typora-user-images\1591065913671.png)

**2.卡顿优化**

1.避免主线线程做耗时操作，否则会影响16ms同步信号到时候doframe 丢帧

2.建立监控体系，简单的方案是基于looper机制，可以再dispatch message 的回调前后做时间统计 和 基于 Choreographer  回调函数 postFrameCallback 来监控

3.UI上做布局优化，减少嵌套，用约束布局，相对布局，用merge include标签， viewstub等

4.关键界面可以做优化系统xml到view对象调用过程，两个部分1.xml->view类，反射调用。 反射调用这个步骤可以用hongyang的方案(https://mp.weixin.qq.com/s/ceXsH06fUFa7y4lzi4uXzw)

5.避免自定义view onMeasure->onLayout->onDraw 过程中的耗时操作

6.recycleview 局部更新 ，分页方案中的diff util等.

**3.启动优化**

1.启动时候懒加载一些不常用的组件

2.启动一个intent service或者rxjava之类的优化耗时初始化，前提是对主线程不必须的

3.ReDex重排启动相关的class

4.字节跳动 mutidex方案，解决5.0以下系统的odex 等待问题，原理是先直接加载dex，启动之后再通过开后台进程做odex优化

5.建立监控体系，Aop插桩技术，native hook ，java hook技术

**4.内存优化**

###### **1.避免内存溢出,内存泄漏，内存抖动 工具LeakCanary ,MAT，profiler**

1. 资源型对象未关闭: Cursor,File
2. 注册对象未销毁: 广播，回调监听
3. 类的静态变量持有大数据对象
4. 非静态内部类的静态实例
5. Handler 临时性内存泄漏: 使用静态 + 弱引用，退出即销毁
6. 容器中的对象没清理造成的内存泄漏
7. WebView: 使用单独进程

###### 2.避免大对象占用

1. AutoBoxing(自动装箱): 能用小的坚决不用大的。
2. 内存复用
3. 使用最优的数据类型
4. 枚举类型: 使用注解枚举限制替换 Enum
5. 图片内存优化（这里可以从 Glide 等开源框架去说下它们是怎么设计的），避免一个项目中多个网络框架库，这样能避免一些缓存。
6. 基本数据类型如果不用修改的建议全部写成 static final,因为 它不需要进行初始化工作，直接打包到 dex 就可以直接使用，并不会在 类 中进行申请内存
7. 字符串拼接别用 +=，使用 StringBuffer 或 StringBuilder
8. 不要在 onMeause, onLayout, onDraw 中去刷新 UI
9. 尽量使用 C++ 代码转换 YUV 格式，别用 Java 代码转换 RGB 等格式，真的很占用内存

###### 3.优化bitmap 相关

1.使用统一的图片框家，

2.项目中集中约束统一的bitmap操作api方法,

3.加载到没存之前，提前或许图片尺寸，对超过屏幕尺寸的图片进行处理

**5.稳定性优化**

1.简历监控体系

- 通过实现 Thread.UncaughtExceptionHandler 接口来全局监控异常状态，发生 Crash 及时上传日志给后台，并且及时通过插件包修复。
- Native 线上通过 Bugly 框架实时监控程序异常状况，线下局域网使用 Google 开源的 breakpad 框架。发生异常就搜集日志上传服务器(这里要注意的是日志上传的性能问题，后面省电模块会说明)
- Aop 插桩技术，AspectJ ，native hook技术

2.主要是fatal 和 anr ,native crash.   通过日志分析足够，掌握厂商抓log的方法串号.

![1591066443851](C:\Users\e-HongShuang.Ren\AppData\Roaming\Typora\typora-user-images\1591066443851.png)

**6.耗电优化，主要是后台任务**

1.复杂计算native 处理

2.减少cpu唤醒，用workmanager job scheduler 等

3.TCP心跳时间加长，合并网络请求，比如日志上传等



**7.IO优化**

1.避免主线程做序列化操作

2.不要频繁使用sp在，尽量批量操作

3.数据库操作用使用事务

# Flutter
1.stateful的widget的build方法为什么会在state内部？
现在，我们回答之前提出的问题，为什么build()方法放在State（而不是StatefulWidget）中 ？这主要是为了提高开发的灵活性。如果将build()方法在StatefulWidget中则会有两个问题：

状态访问不便。

试想一下，如果我们的StatefulWidget有很多状态，而每次状态改变都要调用build方法，由于状态是保存在State中的，如果build方法在StatefulWidget中，那么build方法和状态分别在两个类中，那么构建时读取状态将会很不方便！试想一下，如果真的将build方法放在StatefulWidget中的话，由于构建用户界面过程需要依赖State，所以build方法将必须加一个State参数，大概是下面这样：

  Widget build(BuildContext context, State state){
      //state.counter
      ...
  }
这样的话就只能将State的所有状态声明为公开的状态，这样才能在State类外部访问状态！但是，将状态设置为公开后，状态将不再具有私密性，这就会导致对状态的修改将会变的不可控。但如果将build()方法放在State中的话，构建过程不仅可以直接访问状态，而且也无需公开私有状态，这会非常方便。

继承StatefulWidget不便。

例如，Flutter中有一个动画widget的基类AnimatedWidget，它继承自StatefulWidget类。AnimatedWidget中引入了一个抽象方法build(BuildContext context)，继承自AnimatedWidget的动画widget都要实现这个build方法。现在设想一下，如果StatefulWidget 类中已经有了一个build方法，正如上面所述，此时build方法需要接收一个state对象，这就意味着AnimatedWidget必须将自己的State对象(记为_animatedWidgetState)提供给其子类，因为子类需要在其build方法中调用父类的build方法，代码可能如下：

class MyAnimationWidget extends AnimatedWidget{
    @override
    Widget build(BuildContext context, State state){
      //由于子类要用到AnimatedWidget的状态对象_animatedWidgetState，
      //所以AnimatedWidget必须通过某种方式将其状态对象_animatedWidgetState
      //暴露给其子类   
      super.build(context, _animatedWidgetState)
    }
}
这样很显然是不合理的，因为

AnimatedWidget的状态对象是AnimatedWidget内部实现细节，不应该暴露给外部。
如果要将父类状态暴露给子类，那么必须得有一种传递机制，而做这一套传递机制是无意义的，因为父子类之间状态的传递和子类本身逻辑是无关的。
综上所述，可以发现，对于StatefulWidget，将build方法放在State中，可以给开发带来很大的灵活性。
