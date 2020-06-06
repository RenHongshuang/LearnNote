# LearnNote

## Android启动流程

大概流程：

1.电源键->加载引导程序Bootloader到Ram

2.拉起kernel，kernel设置缓存，寄存器，计划列表，加载驱动，解析init.rc

3.启动init用户进程(pid =1)，初始化和启动属性服务以及

- 启动守护进程

  init进程会孵化出ueventd、logd、healthd、installd、adbd、lmkd等用户守护进程；

- 启动`servicemanager`(binder服务管家)、`bootanim`(开机动画)等重要服务

- init进程孵化出Zygote进程，Zygote进程是Android系统的第一个Java进程(即虚拟机进程。

- 监听进程退出信号，回收僵尸进程

4.zygote进程

- 启动Apprruntime,

  解析init.zygote.rc中的参数，创建AppRuntime并调用AppRuntime.start()方法；

- 启动虚拟机 并注册jni方法

  调用AndroidRuntime的startVM()方法创建虚拟机，再调用startReg()注册JNI函数；

- 通过JNI方式调用ZygoteInit.main()，第一次进入Java世界；

- registerZygoteSocket()建立socket通道，zygote作为通信的服务端，用于响应客户端请求；

- 预加载类和资源

  preload()预加载通用类、drawable和color资源、openGL以及共享库以及WebView，用于提高app启动效率；

- 启动systemServer进程

  zygote完毕大部分工作，接下来再通过startSystemServer()，fork得力帮手system_server进程，也是上层framework的运行载体。

- 等待ams创建应用进程

  zygote功成身退，调用runSelectLoop()，随时待命，当接收到请求创建新进程请求时立即唤醒并执行相应工作

5.System Server进程    

- 启动binder线程池用于与其他进程通信

- 启动各种服务，System Server负责启动和管理整个Java framework，包含ActivityManager，WindowManager，PackageManager，PowerManager等服务。

- 创建systemServiceMananger 管理各种服务的生命周期

6.Ams 启动Launcher等 应用进程 以及binder线程池

- 1.点击图标经历startactivity, 仪表类调用 通过AIDL与ams通信发起启动activity请求
- 2.Ams 检查校验
- 3.Ams 通过socket通信请求zygote 创建应用进程，之后会通过反射启动 ActivityThread##main 函数，创建消息循环

- 4.AMS 通过 aidl 告诉 ActivityThread##H 来反射启动创建Application 实例，并且依次执行 attachBaseContext 、onCreate 生命周期，以及后续的activity各个生命周期



## Retrofit中的注解和动态代理

大概流程:
 1.通过动态代理实例化接口
 2.每调用一个方法都会调用到动态代理里面的 InvocationHandler，所以通过这个方法可以解析方法上面的注解及值,参数注解及值。
 3.创建一个HashMap键是Method 值是ServiceMehtod(保存注解信息的类) 来保存在内存中，再次访问就可以省去解析时间

## **WebView 与 H5**

https://www.jianshu.com/p/345f4d8a5cfa

对于Android调用JS代码的方法有2种：

1. 通过`WebView`的`loadUrl（）` 

   ```java
   mWebView.loadUrl("javascript:callJS()");
   ```

2. 通过`WebView`的`evaluateJavascript（）` 

   ```java
       mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
           @Override
           public void onReceiveValue(String value) {
               //此处为 js 返回的结果
           }
       });
   }
   ```

**对于JS调用Android代码的方法有3种：**

1. 通过`WebView`的`addJavascriptInterface（）`进行对象映射
2. 通过 `WebViewClient` 的`shouldOverrideUrlLoading ()`方法回调拦截 url
3. 通过 `WebChromeClient` 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt（）`方法回调拦截JS对话框`alert()`、`confirm()`、`prompt（）` 消息

## JVM

[图文]: https://mp.weixin.qq.com/s/S1Jcm1YOyEPRZpvB0DlPDQ

![img](https://mmbiz.qpic.cn/mmbiz_png/JJRW7FxQRExGEvHCSlBlBDLibfHJXldiaZTnV09WEY15A2dFwdTaXIE6tPomW5Qr1t09fOMibUo47GM4OoKUP0flw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

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

1. 从IPC角度来说：Binder是Android中的一种跨进程通信方式，该通信方式在linux中没有，是Android独有；
2. 从Android Driver层：Binder还可以理解为一种虚拟的物理设备，它的设备驱动是/dev/binder；
3. 从Android Native层：Binder是创建Service Manager以及BpBinder/BBinder模型，搭建与binder驱动的桥梁；
4. 从Android Framework层：Binder是各种Manager（ActivityManager、WindowManager等）和相应xxxManagerService的桥梁；
5. 从Android APP层：Binder是客户端和服务端进行通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用的 Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。



![img](http://gityuan.com/images/binder/java_binder/java_binder.jpg)

![binder_ipc_process](http://gityuan.com/images/binder/binder_start_service/binder_ipc_process.jpg)

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

卡顿原因：

1.主线程做耗时操作，比如序列化，DB等

2.内存不足，导致系统频繁GC，

3.内存抖动，比如for循环做字符串+ 操作

4.布局不合理

解决方案：

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



