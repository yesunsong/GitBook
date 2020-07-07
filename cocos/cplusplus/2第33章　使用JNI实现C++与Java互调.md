# 第**33**章　使用**JNI**实现**C++**与**Java**互调

Cocos2d-x是用C++语言开发的引擎，而Android下使用的开发语言是Java，Cocos2d-x之所以能够在Android下运行，这归功于JNI技术，JNI是Java Native Interface的缩写，意为Java原生接口，这是一种可以让Native代码调用Java代码，以及让Java代码能够调用其他Native代码的技术。JNI技术是通过NDK（Native Development Kit）实现的，用以将C\C++代码编译成Native代码。

这里的Native代码指的就是C/C++代码，在Android平台下，Cocos2d-x还实现了一层Java底层框架，用于让Cocos2d-x适应Android平台。在Android平台的开发过程中，有时难免会需要调用一些Android系统的平台接口或一些使用Java编写的第三方库，或者使用Java来实现某些功能，在很多情况下可能需要在C++中调用这些Java代码，或者在Java代码中调用C++，这时就需要使用JNI来实现C++与Java的互调。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Android基本概念。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Hello JNI项目。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　编写JNI的C++代码。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Java调用C++。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在C++程序中使用Java。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在Cocos2d-x中实现Java和C++的互调。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Android.mk和Application.mk详解。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　ABI详解。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调试JNI代码。

### 33.1　Android基本概念

要开发Android游戏，需要简单了解Android开发的一些基本概念，如Activity、Intent、资源与R.java、AndroidManifest.xml。

#### 33.1.1　Activity简介

Activity是活动，相当于一个窗口，一个Android程序可以有多个Activity，在AndroidManifest.xml中配置为MAIN的Activity也相当于main()函数，程序员进行Android开发一般会继承一个Activity，在里面创建自己的界面。如图33-1演示了Activity的生命周期，Activity一共有4种状态、7个重要回调、3个周期。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122021.jpeg)

图33-1　Activity流程图

4种状态分别是Running活动状态、Paused暂停状态、Stopped停止状态和Killed死亡状态。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Running活动状态：Activity启动后处于屏幕的最前端，此时处于可见并可与用户交互的活动状态。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Paused暂停状态：当Activity被另一个透明或对话框样式的Activity覆盖时的状态，**此时其仍然可见，但已经失去了焦点，不可与用户交互**。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Stopped停止状态：当Activity不可见时的状态，如按Home键或被另一个Activity完全遮挡住。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Killed死亡状态：当一个Activity没有启动，或被程序员调用finish()函数强制结束，或者由于内存或程序自身原因崩溃时，处于死亡状态。

Activity的7个重要回调分别是onCreate()、onStart()、onResume()、onPause()、onRestart()、onStop()、onDestroy()。它们会在Activity与各种状态之间切换时调用，如图33-1所示。

Activity的3个生命周期分别如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　完整周期：从Activity启动的onCreate()到Activity结束的onDestroy()。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　可视周期：Activity由可见的onStart()到不可见的onStop()。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　可操作周期：Activity由可操作的onResume()到不可操作的onPause()。

#### 33.1.2　Intent简介

Intent表示意图，例如，程序员希望从这个Activity切换到另外一个Activity，这就是一个意图，这里简单理解为窗口切换的一个中介吧。当然，**在Cocos2d-x中的场景切换是不经过这一层的**，当程序员从游戏场景切换到一个全屏的广告时，一般是通过一个Intent。

#### 33.1.3　资源与R.java

R.java位于Android项目的gen目录下，是Android自动生成的一个类，用来索引资源，**在res目录下添加的任何资源都会在这里生成一个索引**，好处是不容易写错资源名字，导致找不到资源，因为所有的资源都会在输入R.之后显示在提示列表中。

R.java类也经常出问题，在引用错误的第三方库，或者重复引用的时候，都有可能让R.java失踪，这时候R.xxx下面就会显示红色的波浪线。

#### 33.1.4　AndroidManifest.xml简介

AndroidManifest.xml是Android应用程序的XML配置文件，配置了程序有哪些Activity，哪个是入口，需要哪些权限等，以及设置横竖屏、开启debug模式等，都需要修改这个文件。

### 33.2　Hello JNI项目

#### 33.2.1　使用Eclipse创建项目

在这里先用Eclipse创建一个Hello JNI的Android工程，使用Android 2.2的版本（Android 1.5以上都是可以的），**包的名称设置为com.hellojni**，然后确定生成，如图33-2所示。

创建完之后，打开res\layout\main.xml，在里面的TextView节点下添加一行android:id="@+id/mytext"，添加这个ID是为了后面可以直接操作这个文本框，因为我们需要通过ID来获取这个文本框对象。在Android中，可以直接编辑Layout下面的XML文件来控制程序的布局，Eclipse也提供可视化的编辑器来生成这些XML布局文件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122022.jpeg)

图33-2　使用Eclipse创建项目

```
<TextView
        android:id="@+id/mytext"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/hello" />
```

接下来简单调整一下src目录下自动生成的代码，加粗部分是笔者自己添加的代码。

```
//先导入TextView的包
import android.widget.TextView;
public class HelloJNIActivity extends Activity {
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        oncallbackInt(13213);oncallbackStr("宝爷威武");
    }
    private void oncallbackInt(int i){//传递整数的函数TextView v = (TextView)(this.findViewById(R.id.mytext));v.append("\n" + i);}private void oncallbackStr(String str){//传递字符串的函数TextView v = (TextView)(this.findViewById(R.id.mytext));v.append("\n" + str);}
}
```

编译运行可以看到如图33-3所示的内容，Hello World，HelloJNIActivity是例子默认输出的文本，后面那段“霸气”的文本就是笔者添加的了，在onCreate()回调的时候调用setContentView()函数设置布局为main.xml，这时会根据layout/main.xml布局文件来初始化布局，然后调用了两个笔者自己添加进去的函数，得到了图33-3所示的结果。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122023.jpeg)

图33-3　运行HelloJNI

布局文件可以手动编辑，也可以直接用Eclipse提供的工具进行可视化的编辑，非常方便。

#### 33.2.2　使用Android Studio创建项目

笔者使用的是当前最新的Android Studio版本2.1.1，在开发Android应用的时候，Android Studio的开发体验还是挺不错的，但也碰到了一些问题，下面会简单分享一下。

首先创建一个新项目，在菜单上选择File→New→New Project命令创建新项目，在弹出的对话框中填写**公司名为hellojni.com、项目名为HelloJNI**以及存储的位置，单击Next按钮。

在弹出的对话框中选择ADK版本，使用默认版本即可，再单击Next按钮，然后可以看到一个模板选择的对话框，如图33-4所示。选择一个空的Activity（默认选项），单击Next按钮会要求输入Activity的名字，以及选择是否生成Layout文件，默认即可，最后单击Finish按钮完成项目的创建。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122024.jpeg)

图33-4　Android Studio创建新项目

接下来开始创建工程，如果是第一次使用Android Studio的话，这里有可能由于更新Gradle而卡在building gradle project info界面，此时可以下载安装最新的Gradle，也可以通过翻墙软件让Android Studio自动下载。

如果读者使用的JDK不是1.8或以上版本，那么项目可能会报一个错误，让读者升级到JDK 1.8，笔者这里提示的错误是compileDebugJavaWithJavac.compileSdkVersion 'android-24' requires JDK 1.8 or later to compile。读者可以安装1.8或降低ADK的版本来解决该问题，降低ADK的版本需要在**build.gradle文件**中同时修改compileSdkVersion为23、buildToolsVersion为23.0.1、targetSdkVersion为23，以及dependencies中的compile选项为com.android.support:appcompat-v7:23.1.0，如图33-5所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122025.jpeg)

图33-5　降低ADK版本

接下来打开左侧项目栏中res/layout下的activity_main.xml，会出现如图33-6所示的界面，可以在Design设计模式和Text文本模式间进行切换，图33-6处于文本模式中，插入如下代码，添加一个TextView到布局文件中。在设计模式下还可以手动修改这些控件。

```
<TextView
    android:id="@+id/mytext"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content" />
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122026.jpeg)

图33-6　文本模式编辑布局

在MainActivity.java中添加同样的代码，运行后可以看到如图33-7所示的界面。这里笔者是在虚拟机上运行的，需要先安装Android虚拟机，安装完之后可能会提示还需要开启VT功能，这时候需要重启计算机，进入BIOS模式手动开启VT功能。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122027.jpeg)

图33-7　Android Studio虚拟机运行HelloJNI

### 33.3　编写JNI的C++代码

前面介绍了一个简单的Android程序，接下来要实现在Java中调用C/C++函数，在C/C++中回调Java的函数以及传参，下面会分别介绍在Eclipse下和Android Studio下如何操作。

#### 33.3.1　为Eclipse添加Android.mk和hello.c

首先在Android项目下新建一个jni文件夹，然后在jni文件夹下面添加一个hello.c文件和一个Android.mk文件，然后填写以下内容，将hello.c文件编译到myjni库里。

Android.mk文件如下。

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE    := myjni
LOCAL_SRC_FILES := hello.c
LOCAL_LDLIBS    := -llog
include $(BUILD_SHARED_LIBRARY)
```

上面的Android.mk文件会将hello.c编译成so，供Android调用，下面是hello.c文件的内容，其中，前面的LOGI和LOGW两个宏是用于在Android环境下打印日志用的，可以在Logcat窗口查看程序输出的日志。

```
#include <jni.h>
#include <android/log.h>
#define LOGI(...) ((void)__android_log_print(ANDROID_LOG_INFO, "native-
activity", __VA_ARGS__))
#define LOGW(...) ((void)__android_log_print(ANDROID_LOG_WARN, "native-
activity", __VA_ARGS__))
 JNIEXPORT void JNICALL Java_com_hellojni_HelloJNIActivity_callJNIInt
( JNIEnv* env, jobject obj , jint i)
 {
     //找到java中的类
     jclass cls = (*env)->FindClass(env, "com/hellojni/HelloJNIActivity");
     //再找类中的方法
     jmethodID mid = (*env)->GetMethodID(env, cls, "oncallbackInt", "(I)V");
     if (mid == NULL)
     {
         LOGI("int error");
         return;
     }
     //打印接收到的数据
     LOGI("from java int: %d",i);
     //回调Java中的方法
     (*env)->CallVoidMethod(env, obj, mid ,i);
 }
 JNIEXPORT void JNICALL Java_com_hellojni_HelloJNIActivity_callJNIString
( JNIEnv* env, jobject obj , jstring s)
 {
     //找到Java中的类
     jclass cls = (*env)->FindClass(env, "com/hellojni/HelloJNIActivity");
     //再找类中的方法
     jmethodID mid = (*env)->GetMethodID(env, cls, "oncallbackStr",
     "(Ljava/lang/String;)V");
     if (mid == NULL)
     {
         LOGI("string error");
         return;
     }
     const char *ch;
     //获取由Java传过来的字符串
     ch = (*env)->GetStringUTFChars(env, s, NULL);
     //打印
     LOGI("from java string: %s",ch);
     (*env)->ReleaseStringUTFChars(env, s, ch);
     //回调Java中的方法
     (*env)->CallVoidMethod(env, obj, mid ,(*env)->NewStringUTF(env,s));
 }
```

#### 33.3.2　在Android Studio下编写JNI

在Android下不需要编写Android.mk文件，只需要创建JNI目录，编写C/C++代码，设置build.gradle和gradle.properties。

##### 1．添加JNI目录

可以在项目视图上右击打开菜单，选择New→Folder→JNI Folder命令来创建一个JNI目录，如图33-8所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122028.jpeg)

图33-8　创建JNI目录

##### 2．添加C/C++代码

创建完JNI目录之后可以在JNI目录下创建一个hello.c文件，然后输入下面的代码。需要注意的是该段代码与Eclipse下的代码有些区别，如果直接复制Eclipse下的代码会报错，其跟Eclipse下的hello.c文件主要有以下区别。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　添加了#define NULL 0，因为这里编译表示NULL未定义。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　由于这里的包名和Eclipse下不同，多了一个.hellojni，所以函数名和FindClass()方法传入的字符串也要对应地添加上hellojni。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　去掉了callJNIString()方法中打印UTF字符串的代码，因为运行时报了字符格式的错误。

```
#include <jni.h>
#include <android/log.h>
#define LOGI(...) ((void)__android_log_print(ANDROID_LOG_INFO, "native-
activity", __VA_ARGS__))
#define LOGW(...) ((void)__android_log_print(ANDROID_LOG_WARN, "native-
activity", __VA_ARGS__))
#define NULL 0
 JNIEXPORT void JNICALL Java_com_hellojni_hellojni_HelloJNIActivity_
 callJNIInt( JNIEnv* env, jobject obj , jint i)
 {
     //找到Java中的类
     jclass cls = (*env)->FindClass(env, "com/hellojni/hellojni/
     HelloJNIActivity");
     //再找类中的方法
     jmethodID mid = (*env)->GetMethodID(env, cls, "oncallbackInt", "(I)V");
     if (mid == NULL)
     {
        LOGI("int error");
        return;
    }
    //打印接收到的数据
    LOGI("from java int: %d",i);
    //回调Java中的方法
    (*env)->CallVoidMethod(env, obj, mid ,i);
}
JNIEXPORT void JNICALL Java_com_hellojni_hellojni_HelloJNIActivity_
callJNIString( JNIEnv* env, jobject obj , jstring s)
{
    //找到Java中的类
    jclass cls = (*env)->FindClass(env, "com/hellojni/hellojni/
    HelloJNIActivity");
    //再找类中的方法
    jmethodID mid = (*env)->GetMethodID(env, cls, "oncallbackStr",
    "(Ljava/lang/String;)V");
    if (mid == NULL)
    {
        LOGI("string error");
        return;
    }
    //回调Java中的方法
    (*env)->CallVoidMethod(env, obj, mid , s);
}
```

如果先编写Java代码，可以使用更便捷的方法来创建上面的hello.c文件。在进入命令行使用javah -jni包名+类名可以生成一个头文件，将Java代码中声明的native()方法，自动按照指定的格式生成对应的C/C++方法，然后把头文件中的代码复制到hello.c中，就不用去写冗长的函数名了。

##### 3．设置build.gradle和gradle.properties

接下来需要手动在项目的app目录下的build.gradle中添加代码，如图33-9所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122029.jpeg)

图33-9　配置build.gradle

在android的defaultConfig下添加如下所示的NDK配置代码，可以配置要如何编译JNI目录下的代码。

```
ndk {
    moduleName "myjni"
    ldLibs "log", "z", "m"
    adbfilters "armeabi", "armeabi-v7a", "x86"
｝
```

如果在此时编译，会报如图33-10所示的错误，让我们在gradle.properties文件中添加一行android.useDeprecatedNdk=true选项，此时可以直接单击错误提示中的链接，Android Studio会自动在gradle.properties文件中加上这行代码（真是方便）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122030.jpeg)

图33-10　配置gradle.properties

网上有一种说法是Android Studio中假设只有一个C/C++文件会编译报错，需要添加一个空的C/C++文件，但笔者使用Android Studio 2.1.1时并没有发现该问题。

#### 33.3.3　C++的函数原型

需要注意的是前面函数的声明有一个标准格式：

```
JNIEXPOR 返回值JNICALL Java_包名_Java类名_函数名(JNIEnv* , jobject, 自定义参
数...)
```

例如，前面代码中的Java_com_hellojni_HelloJNIActivity_callJNIString方法，包名是com.hellojni.Hello，类名是HelloJNIActivity，函数名为callJNIString，由于“.”是C++中的关键字，所以需要使用下画线“_”来替换“.”，所以这些“.”都是通过下画线“_”连接的。

函数需要对应到Java中的native函数，并不是凭空写的，相当于要在Java中声明，然后在C/C++中定义，然后才可以在Java中使用。稍后会介绍如何在Java中声明。

这里的返回值是**Native类型**的，假设返回一个整数，那么void对应的修改为jint类型，字符串的返回值对应的是jstring，对应的类型可以参照图33-11所示。

图33-11所示的是基础类型，如图33-12是基础类型对应的数组类型。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122031.jpeg)

图33-11　JNI与Java类型对照表

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122032.jpeg)



图33-12　JNI与Java数组类型对照表

#### 33.3.4　C++调用Java

在定义完函数原型之后，我们的Java就可以找到对应的C++函数了，接下来在C++中应该如何调用Java的函数呢？

JNIEnv的FindClass可以找到Java中的类，调用GetMethodID可以获取里面的方法，CallVoidMethod可以调用Java中的方法，关于JNI提供的方法，可以参考NDK platform目录下的include/jni.h。

程序员在查找类的时候，是通过一个类似目录结构的字符串来查找的，这个**目录结构对应Java的命名空间，也就是包，最后是类名**。

在查找函数的时候，需要传入两个字符串，第一个字符串用于描述函数的名称，第二个字符串用于描述函数的原型，描述原型的字符串书写规则如下。

```
(参数列表;)返回值类型
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122033.jpeg)

图33-13　类型签名字符串

这里的参数列表和返回值类型可以参照图33-13所示，假设是传入一个int，返回一个long，那么这串字符串就是(I;)J，而如是传入String就有点不一样了，**String在Java中并不是一个基础类型而是一个类，传入类对象，需要用L加上包名/类名，String所在的包是java.lang所以String就是Ljava/lang/String**，传入自定义的类也是如此。

找到函数之后，调用CallVoidMethod()函数传入找到的函数ID以及参数，JNI就会调用对应的Java函数了，有一点需要注意的是，在传字符串的时候，Java传过来以及程序员要传给Java的都是UTF编码的格式，所以在取出Java传入的字符串时，要用JNIENV的GetStringUTFChars转换一下，将jstring转成char*，但注意用完之后要**调用ReleaseStringUTFChars()函数释放内存**。

NewStringUTF()函数创建出来的jstring，是在Java虚拟机的堆空间分配的内存，由Java虚拟机的内存管理机制管理，在不需要用到这个jstring的时候，可以**用JNIEnv对象的DeleteLocalRef()函数来释放**，类似Cocos2d-x的release操作，这样会减少对象的引用计数。

表33-1　JNI中的创建和释放函数

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122034.jpeg)

参照表33-1，所有Get开头的函数，一般都有对应的Release操作，而所有New开头的函数，一般都是对应DeleteLocalRef()函数来进行释放。

本节涉及的内容比较多，对于C++代码的编写主要涉及两部分的内容，即函数原型如何定义，以及在C++中调用Java，对于C++调用Java的部分，后面的内容中还会详细介绍。

### 33.4　Java调用C++

##### 1．修改Java代码

相比起C++代码的编写，Java的调用就简单很多了，我们在hello.c中实现了两个原生方法，那么要在Java中使用，只需要两个步骤，一是声明这个原生方法，二是加载我们编写的JNI模块。

```
private native void callJNIString(String str);
private native void callJNIInt(int i);
```

Hello.c根据JNI的规则，在对应的命名空间类下实现了两个方法，然后直接在HelloJNIActivity中声明这两个方法即可，类似使用第三方库的时候include一个头文件。

```
static
{
    //加载本地库
    System.loadLibrary("myjni");
}
```

在HelloJNIActivity中添加上面的代码，就会在启动的时候加载我们编写的静态库，在Android.mk中将模块命名为myjni，这会生成一个libmyjni.so文件，可以直接加载该文件。

当然，代码还需要调整的就是，调用我们的原生方法试试看，把在onCreate()函数中添加的两行代码修改如下：

```
callJNIInt(13213);
callJNIString("宝爷威武");
```

一切代码都写好之后还需要编译，编译有两种方法，一种是直接进入cygwin手动用ndk-build来编译在jni目录下的代码；另一种是直接在Eclipse下配置，但需要NDK r7以上的版本，然后直接编译。

##### 2．在Eclipse下运行

如图33-14所示，新建一个Builder，然后选择ndk-build.cmd来编译HelloJNI项目，最后单击运行可以看到代码在编译，如图33-15所示，如果编译出错的话也可以直接看到。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122035.jpeg)

图33-14　创建Builder

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122036.jpeg)

图33-15　运行结果

可以看到，显示的结果与之前调用的一样，但是这次调用经过了Java——C++——Java，绕了一大圈。

##### 3．在Android Studio下运行

在Android Studio下运行时，直接单击运行即可，无须设置Builder，读者可自己操作一下，这里不再详细介绍。

### 33.5　在C++程序中使用Java

前面的例子虽然也有C++调用Android的部分，但并不是一个完整的流程，它是基于Java调用C++传入的环境JNIEnv指针，假设在纯粹的C++环境下，应该如何调用Java呢？C++调用Java的静态函数和成员函数又有什么区别呢？如何实例化一个对象？如何操作这个对象的属性？本节会完整地介绍这些内容。

这次我们写的不是Android程序了，而是用C++写一个exe控制台程序，然后在该程序里调用Java，前面的例子是从Android启动，然后调用C++，最后C++回调Java的例子，本节的例子是直接启动exe，调用Java编译而成的class文件。

首先来编写一个Java文件，实现几个简单的函数，一个静态函数，然后传入两个字符串，把它们拼接在一起返回，然后编写一个成员函数，传入两个数字，返回两个数字的和，另外还加上一个String类型的属性name。**注意这里并没有打Java包（就是放到类似com/xxx/ooo的目录下，然后再Package com.xxx.ooo）。**

```
public class Test {
public String name;
    //返回两个字符串相加的静态函数
    public static String strAdd(String str1, String str2) {
         return str1 + str2 + "!";
    }
    //返回两个整数相加的成员函数
    public int intAdd(int int1, int int2) {
         return int1 + int2;
    }
}
```

Java代码写好了，然后需要将其编译成class文件，在Eclipse里写完后会自动被编译成class文件，并被放在Java项目的bin\classes目录下。如读者使用Eclipse的话，写完后保存，然后把class文件复制过来就可以了。也可以直接用命令行编译，将Java文件进行编译，如图33-16所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122037.jpeg)

图33-16　执行javac编译

使用javap命令可以查看编译的class文件，输入**javap -s -private类名**可以显示class文件的信息，如图33-17所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122038.jpeg)

图33-17　执行javap命令

其中，-s表示显示类的签名，-private表示显示类的全部方法，下面的信息对于用C++来调用Java是很有用的，可以让我们不必去记住麻烦的签名对照表，每个函数的签名，都在该函数后面的Signature之后，只要把这段代码复制到程序里面就可以了。下面是strAdd方法和strInt方法的签名。

```
strAdd:(Ljava/langString;Ljava/lang/String;)Ljava/lang/String;strInt:(II)I
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122039.jpeg)**注意：**strAdd中，每个类最后都要有一个“;”来标识这个类结束了。

**最后将编译出来的class文件放到工作目录下**，笔者这里的工作目录是ProjectDir目录，也就是如图33-18所示的一个目录。

编译完Java程序后，接下来就要用C++来调用Java程序了，使用C++直接调用Java需要以下几步。

（1）初始化Java虚拟机。

（2）获取class对象。

（3）获取方法对象。

（4）调用Java方法。

（5）销毁Java虚拟机。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122040.jpeg)

图33-18　TestJni目录

OK，接下来用Visual Studio新建一个C++控制台项目，命名为TestJni，然后在项目里面添加一个main.cpp，注意是cpp，如果是.c的话，那么要写的代码会有些不同，原因在后面解释吧。我们需要对项目进行一些简单的设置，以便于能够使用JNI并且顺利地编译成功然后运行。

首先是头文件路径的设置，如果这一步没有做，在编译main.cpp的时候会编译失败，这里需要设置的路径是JDK的路径\include，JDK的路径/include/win32，如图33-19所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122041.jpeg)

图33-19　设置包含目录

然后需要设置链接库以及链接库的路径，如果这一步没有做，在编译完代码将所有的代码链接成可执行程序的时候会报错，因为程序使用了JNI的东西，所以需要把它链接到程序中，需要设置附加库目录为JDK的路径/lib，如图33-20所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122042.jpeg)

图33-20　设置附加库目录

接下来是链接库，需要添加jvm.lib到附加库依赖项中，如图33-21所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122043.jpeg)

图33-21　附加依赖项

最后，需要将JRE Java运行环境添加到我们的工作路径中，如果这一步没有做，在运行exe的时候会提示找不到jvm.dll，这个dll就在JRE目录/bin/client文件夹中，在“调试”页面的“环境”选项后添加path=你的jvm.dll所在的路径即可，如图33-22所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122044.jpeg)

图33-22　设置环境

设置完项目，接下来可以开始编写代码了，下面的代码可以直接运行，请读者仔细查看这段代码，虽然代码注释已经解释得很清楚了，但还是要特别注意**每个操作调用的函数是不一样的**，特别是静态函数和成员函数。

```
#include <jni.h>
#include <string.h>
#include <stdio.h>
int main(void)
{
    JavaVMOption options[1];
    JNIEnv *env;
    JavaVM *jvm;
    JavaVMInitArgs jvmArgs;
    jclass cls;
    jmethodID mid;
    jfieldID fid;
    jobject obj;
    //设置class目录
    options[0].optionString = "-Djava.class.path=.";
    memset(&jvmArgs, 0, sizeof(jvmArgs));
    //JNI的版本
    jvmArgs.version = JNI_VERSION_1_6;
    //option数组的大小
    jvmArgs.nOptions = 1;
    jvmArgs.options = options;
    //启动虚拟机
    if (JNI_CreateJavaVM(&jvm, (void**)&env, &jvmArgs) != JNI_ERR)
    {
         //先获得class对象
         cls = env->FindClass("Test");
         if (cls != 0)
         {
              //获取方法ID，通过方法名和签名调用静态方法
              mid = env->GetStaticMethodID(cls, "strAdd",
    "(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;");
              if (mid != 0)
              {
                   //调用strAdd静态方法，传入两个BAO，它会返回BAOBAO！
                   const char* name = "BAO";
                   jstring jname = env->NewStringUTF(name);
                   //这里调用的是StaticObjectMethod,静态方法
                   jstring ret = (jstring)env->CallStaticObjectMethod(
                        cls, mid, jname, jname);
                   //将函数的返回值转成char*然后再打印出来
                   const char* str = env->GetStringUTFChars(ret, 0);
                   printf("strAdd: %s\n", str);
                   //释放GetStringUTFChars返回的字符串
                   env->ReleaseStringUTFChars(ret, str);
                   //释放NewStringUTF()函数创建的字符串
                   env->DeleteLocalRef(jname);
              }
              //调用默认构造函数<init>来new一个对象
              mid = env->GetMethodID(cls, "<init>", "()V");
              obj = env->NewObject(cls, mid);
              if (obj == 0)
              {
                   printf("Error: Create Test failed!\n");
              }
              //调用成员函数intAdd，它的签名是(II)I，后面没有跟分号哦
              mid = env->GetMethodID(cls, "intAdd", "(II)I");
              if (mid != 0)
              {
                   //调用成员函数intAdd返回1和9的和
                   //这里调用的是ObjectMethod
                   int sum = (int)env->CallObjectMethod(obj, mid, 1, 9);
                   printf("intAdd: %d\n", sum);
              }
              //通过属性名和签名获取属性ID，然后设置属性值
              //这里获取Test对象的name变量，它是一个String类型的变量
              fid = env->GetFieldID(cls, "name", "Ljava/lang/String;");
              if (fid != 0)
              {
                   const char* name = "baoye";
                   jstring arg = env->NewStringUTF(name);
                       env->SetObjectField(obj, fid, arg);
                       env->DeleteLocalRef(arg);
                 }
                 //将属性值取出来
                 fid = env->GetFieldID(cls, "name", "Ljava/lang/String;");
                 if (fid != 0)
                 {
                       jstring name = (jstring)env->GetObjectField(obj, fid);
                       const char* str = env->GetStringUTFChars(name, 0);
                       printf("Test.name is %s\n", str);
                       env->ReleaseStringUTFChars(name, str);
                 }
                 if (obj != 0)
                 {
                       env->DeleteLocalRef(obj);
                 }
             }
             //释放JVM
             jvm->DestroyJavaVM();
           }
       else
       {
             printf("Error: Create JVM Faile!\n");
       }
       return 0;
    }
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122045.jpeg)

图33-23　TestJni运行结果

程序的运行结果如图33-23所示。

最后来梳理一下这段C++代码的步骤。

（1）设置好参数。

（2）创建JVM虚拟机。

（3）找到了我们的Test类。

（4）调用静态函数strAdd传入两个BAO。

（5）创建Test对象。

（6）调用Test对象的成员函数intAdd传入1，9。

（7）获取Test对象的成员变量name，设置为baoye。

到这里还有一个问题没说，就是**这段代码在C语言环境下编译运行，编译会出错**！在C语言环境下编译会得到一系列的编译错误，XXX的左侧必须指向结构/联合，在所有JNIEnv指针和JavaVM指针调用成员函数的地方会提示这个错误，这是因为**C语言和C++语言对结构体的解析不同导致的，在C++语言中是类，而在C语言中则是纯粹的结构体，**在C语言中编译需要将所有env->xxx(env, ......)改为(*env)->xxx(......)。

### 33.6　在Cocos2d-x中实现Java和C++的互调

使用Cocos2d-x开发Android时，在Cocos2d-x里调用Java代码的情况可能会更多一些，例如，使用了某些Android平台的第三方库，以及91平台的SDK、新浪微博的SDK和一些广告平台的SDK，本节将简单介绍一下如何在Cocos2d-x中使用JNI。

#### 33.6.1　Cocos2d-x的JNI初始化

Cocos2d-x为程序员准备好了一个JNI环境指针，在Cocos2d-x中所有对Java的调用，都要通过其来执行。首先来看Cocos2d-x JNI环境的初始化，这里以官方tests示例中的cpp-empty-test为例。

首先Android程序的入口在Java层，并不在C++层，Android的入口位于AndroidManifest.xml文件中，标记为主Activity的Activity类，这个类一般位于proj.android（或proj.android-studio）目录下的src目录下。在src目录下可以找到一串根据创建项目时的包名生成的目录，例如org.cocos2dx.cpp_empty_test。这个包名会生成org\cocos2dx\cpp_ empty_test目录，在这个目录下默认会生成一个AppActivityjava。入口的Activity类就定义在这个Java文件中。

Cocos2d-x的Activity一般继承于Cocos2dxActivity，负责在Android环境下初始化Cocos2d-x，在主Activity中一般会调用System.loadLibrary来加载程序员用NDK编译出来的游戏库。

在Cocos2d-x引擎的cocos\platform\android目录下的javaactivity-android.cpp文件中，程序员可以找到下面这段代码（原本是在项目的jni目录下的main.cpp中）。

```
JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
    JniHelper::setJavaVM(vm);
    cocos_android_app_init(JniHelper::getEnv());
    return JNI_VERSION_1_4;
}
```

当主Activity调用loadLibrary将游戏库加载进去的时候，就会执行JNI_OnLoad()回调，在这里通过JniHelper的setJavaVM()函数，将Android系统传入的Java虚拟机保存起来，方便后面在Cocos2d-x中调用Java方法。

#### 33.6.2　在Cocos2d-x中调用Java

有了JNI环境之后，想在Cocos2d-x中调用Java就很方便了，下面来看一下如何在Cocos2d-x中调用Java。

Cocos2d-x将JNI环境封装了一层，称之为JniHelper，位于引擎的cocos/platform/android/jni路径下，最关键的函数是**getJavaVM()**，因为有了它就可以调用Java了，此外JniHelper的**callStaticXXXMethod**系列接口比起直接调用getJavaVM()要节省不少的代码量，所以可以直接使用，其中的XXX为Java静态函数返回值的类型，如果返回int则XXX替换为Int，没有返回值则XXX替换为Void。关于callStaticXXXMethod系列接口的使用，读者可以直接看一下Cocos2d-x自己是怎样使用它们的。

例如，在引擎的cocos\platform\android目录下的CCApplication-android.cpp文件中的setAnimationInterval()函数。

```
void Application::setAnimationInterval(float interval) {
    JniHelper::callStaticVoidMethod("org/cocos2dx/lib/Cocos2dxRenderer",
    "setAnimationInterval", interval);
}
```

直接调用JniHelper的callStaticVoidMethod系列接口，传入要调用的类以及类的**静态方法**，并传入参数。非常的方便！

#### 33.6.3　在Java中调用Cocos2d-x

在Java中调用Cocos2d-x，原理也同前面介绍的在Java中调用C++一样。在编写C++代码之前，先要确定要在哪里调用C++代码，因为函数名需要以调用处的Java代码的包名和类名作为前缀，拼写规则为前面所讲的：

```
JNIEXPORT 返回值JNICALL Java包名_Java类名_函数名(JNIEnv* , jobject, 自定义参
数...)
```

这个函数也需要在头文件处声明一下，在函数体内实现我们的逻辑，剩下就是Java的任务了，C++部分代码的编写和33.6.1节的例子类似，根据需求写函数即可，但要注意函数声明的拼写，Java部分的代码会更简单一些。

首先**不需要调用System.LoadLibrary来加载我们的so**，我们的代码会被统一编译到游戏的so里，如果想编译成另外一个So然后加载也是可以的，接下来还是在对应的Java代码中声明native()函数然后调用即可。

### 33.7　Android.mk和Application.mk详解

在程序员开发Cocos2d-x游戏的时候，未必会写一些Java代码来与C++互相调用，但一定会涉及修改Android.mk，至少程序员需要把新添加的C++文件添加到要编译的源文件列表中，本节将介绍Android.mk，如何添加源文件，如何添加第三方库。

#### 33.7.1　认识Android.mk

Android.mk是一个用描述你要编译的代码的文件，本质上和普通的MakeFile没什么区别，都是告诉编译器如何编译整个工程的源码，要编译哪些文件，编译的顺序，编译的规则，选项等。

Android.mk中广泛运用到了模块的概念，**一个模块一般表示一个静态库或动态库**，例如，Cocos2dx引擎本身是一个模块，而它的扩展库extensions、音乐库CocosDenshion也是一个模块。

程序员自身编写的游戏代码也是一个模块，而描述一个模块如何生成以及模块之间的关系，就是Android.mk的任务了。一般一个Android.mk表示一个模块，但也可以在一个Android.mk文件中描述几个模块。

要了解Android.mk，需要从Android.mk的Hello World开始，打开你的Ndk目录，在samples目录下找到hello-jni，在其jni目录下可以找到Android.mk，打开这个文件，内容如下。

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE    := hello-jni
LOCAL_SRC_FILES := hello-jni.c
include $(BUILD_SHARED_LIBRARY)
```

Android.mk**必须以LOCAL_PATH变量定义开头**，用于定位源文件，$(call xxx)这个写法会调用xxx函数，my-dir函数会返回当前这个文件本身所在的目录。

include操作将包含一个Makefile文件，CLEAR_VARS是系统的一个特殊的Makefile文件，它会清除许多LOCAL_XXX变量，因为这些是全局变量在上一个Makefile中的赋值，可能会导致这里的编译出现意外的结果。

LOCAL_MODULE用于标识当前编译的模块，这个模块名必须是唯一的，并且不能包含任何空格，模块生成的库会被自动添加上lib前缀和so后缀。

LOCAL_SRC_FILES变量包含一个C或C++源文件的列表，它们会被加入编译。

在文件的最后包含了一个BUILD_SHARED_LIBRARY的Makefile，它会将这个模块生成一个动态库。

#### 33.7.2　用Android.mk添加源文件以及指定头文件目录

当程序员要开始编写Android.mk的时候，有经验的可能会先找到TestCpp里的Android.mk文件，看它是怎么写的，然后参考着写一个，接下来会找到一个超长的Android.mk文件，里面密密麻麻地写了TestCpp中所有参与编译的源文件，而我们的游戏至少也有几十个源文件，而一个一个地写进去是否太麻烦呢？

程序员希望指定一个目录，然后该目录下的所有源文件都参与编译，那么如何让Android.mk自动搜索某个目录下的所有源文件，而不用程序员自己手动把目录下的源文件逐个添加到LOCAL_SRC_FILES中。

```
FILE_LIST := $(wildcard $(LOCAL_PATH)/../../Classes/*.cpp)
LOCAL_SRC_FILES += $(FILE_LIST:$(LOCAL_PATH)/%=%)
```

上面的代码用基于Androd.mk所在路径的相对路径指定了Classes文件夹下所有的cpp文件，可以多次使用这两行代码来指定多个路径。

#### 33.7.3　在Android.mk中引用其他模块（第三方库）

在Android.mk中可以很轻松地引用其他模块，例如，引用Cocos2d-x模块、XML模块，其实这里的模块与在Visual Studio中的引用外部lib库是一个概念，只是这里引用的模块都是使用NDK生成的模块。

##### 1．引用外部模块的步骤

引用一个外部模块需要以下几步。

（1）首先需要可以找到这个模块，通过NDK_MODULE_PATH或import-add-path方法可以设置模块的搜索路径。

第一种方式是在你的NDK_MODULE_PATH中添加要导入的Android.mk所在的路径。NDK_MODULE_PATH的路径不可以包含空格，使用分号“;”为分隔符，多个路径会顺序搜索。

第二种方式为在Android.mk调用import-add-path方法，在Android.mk中动态设置搜索路径，笔者个人比较推荐使用这种方法，调用的代码如下。

```
$(call import-add-path,$(LOCAL_PATH)/../../../cocos2d)
$(call import-add-path,$(LOCAL_PATH)/../../../cocos2d/external)
$(call import-add-path,$(LOCAL_PATH)/../../../cocos2d/cocos)
```

（2）在程序员自己的Android.mk的结尾处添加引用模块的指令，module-path是要引用的模块所在的路径，例如Cocos2d-x：

```
$(call import-module, module-path)
```

（3）在Android.mk中将其添加到本地包含的静态库或者动态库中。

```
LOCAL_STATIC_LIBRARIES += module-name （本模块依赖于静态库module-name）
LOCAL_SHARED_LIBRARIES += module-name （本模块依赖于动态库module-name）
```

##### 2．添加自定义的模块示例

例如，现在写一个动态库，然后在程序的Android.mk中导入该库。首先创建一个目录叫作MyModule，在目录下添加一个Add.cpp文件，在上面写上一个简单的函数。

```
int Add(int a, int b)
{
return a + b;
}
```

再添加一个Add.h文件，声明该函数。

```
int Add(int a, int b);
```

最后在该目录下添加一个Android.mk文件。

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := MyAdd
LOCAL_SRC_FILES := Add.cpp
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)
include $(BUILD_SHARED_LIBRARY)
```

确定MyModule所在的路径在NDK_MODULE_PATH中，例如，我们的MyModule的绝对路径为F:\MyGame\TestNdk\MyModule，那么NDK_MODULE_PATH需要包含F:\MyGame\TestNdk这个路径，然后在自己的Android.mk中这样引入MyAdd模块：

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := chipmunk_static
LOCAL_MODULE_FILENAME := libchipmunk
//这里省略加入源文件，以及头文件路径的指定
LOCAL_SRC_FILES := ......
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include/......
LOCAL_C_INCLUDES := $(LOCAL_PATH)/include/......
//在编译之前，指定你的模块依赖于MyAdd模块
LOCAL_SHARED_LIBRARIES += MyAdd
//编译成so
include $(BUILD_SHARED_LIBRARY)
//最后写入import命令，这里是MyModule，是Android.mk基于NDK_MODULE_PATH的相对
路径，我们的模块名字不叫MyModule，叫MyAdd，但是MyModule路径下的Android.mk描述
了这个模块
$(call import-module, MyModule)
```

#### 33.7.4　Android.mk的一些变量和函数

常用系统脚本如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　CLEAR_VARS：一个能够清除大部分的LOCAL_XXX变量的脚本，必须在一个新模块开始之前包含这个脚本。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　BUILD_SHARED_LIBRARY：一个能够将模块编译成动态链接库的脚本，必须先定义LOCAL_MODULE和LOCAL_SRC_FILES变量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　BUILD_STATIC_LIBRARY：一个能够将模块编译成静态链接库的脚本，其他同上。

常用函数如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　my-dir()函数：返回最近包含的Makefile的路径，一般是当前Android.mk的目录。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　all-subdir-makefiles()函数：返回my-dir目录下所有的Android.mk列表，递归搜索所有子目录。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　this-makefile()函数：返回当前makefile的路径。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　import-module()函数：根据名称导入另一个模块的Android.mk，在NDK_MODULE_PATH的目录中搜索。

常用系统变量如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_PATH：当前文件的路径，需要在模块开头的地方定义。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_MODULE：当前模块的名字，必须是唯一且不包含任何空格。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_SRC_FILES：组成模块的源文件列表。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_C_INCLUDES：头文件include的搜索路径。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_CFLAGS：编译器参数的选项。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_STATIC_LIBRARIES：需要被链接进该模块的静态库模块的列表。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　LOCAL_SHARED_LIBRARIES：该模块在运行时依赖的动态模块列表。

#### 33.7.5　关于Application.mk

Application.mk用于向NDK描述你的应用程序，一个应用程序是由若干个模块组成的，在Application.mk中可以设置以下变量。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_PROJECT_PATH：应用程序的路径，把Application.mk放到项目的jni目录下，可以不设置这个变量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_MODULES：选择要构建的Android.mk，假设不填这个，NDK**只构建当前目录下的Android.mk**，以及Android.mk中引用到的模块，多个模块之间需要用空格区分开，NDK会自动查找它们的依赖项。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_OPTIM：可以定义为release或debug，debug模式会更易于调试，默认是release模式。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_CFLAGS：可以设置程序中所有Android.mk编译的选项，如-G、-I、-L等选项，这个设置将被应用于所有的Android.mk。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_CPPFLAGS：该选项和APP_CFLAGS相似，但其只对参与编译的cpp文件有效。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_ABI：为了支持基于ARMv7的设备的软浮点单元指令，会设置为armeabi-v7a，为了同时支持ARMv5TE和ARMv7，可以将APP_ABI设置为armeabi armeabi-v7a。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_PLATFORM：用于描述需要编译的平台，这里需要根据下载的ADK来填写，如果这里填写错误的话，有可能导致Cocos2d-x的Android项目报错。打开NDK目录，在platforms目录下，就是可以填写的所有平台，如Android-9，每个Android平台都会对应一个ADK版本，这两个需要正确地对应。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　APP_STL：用于指定如何使用标准C++的库，system表示使用Android系统默认的C++运行时库，stlport_shared表示使用NDK提供的动态STLPort库，stlport_static表示使用NDK提供的静态STLPort库。

### 33.8　ABI详解

ABI是Application Binary Interface的缩写，意为应用程序二进制接口，它定义了二进制文件如何运行在相应的平台上，执行何种指令集。这是一个很关键但却很容易被忽视的知识点，在Android开发中会碰到很多种ABI，不同CPU架构的设备需要对应不同的ABI。

ABI与我们编译的so文件息息相关，使用不同的ABI会编译出不同的so文件，如果希望在指定的设备上使用我们的so，就必须提供使用该设备支持的ABI编译出来的so。

#### 33.8.1　常见的ABI

接下来了解一下常见的ABI。常见的ABI主要有armeabi、armeabi-v7a、armeabi-v8a、mips、mips_64、x86、x86_64等，http://www.eepw.com.cn/article/268232.htm中的文章对比了ARM、X86和MIPS这3种主流芯片的架构。

在Android开发中使用最多的是ARM，下面简单介绍一下ARM平台上的几种ABI。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　armeabi：通用性最好，支持所有ARM设备，支持软浮点运算（不支持硬浮点运算）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　armeabi-v7a：支持ARMv7设备、硬件浮点运算以及FPU指令。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　armeabi-v8a：armeabi-v7a的升级版，同时支持64位和32位程序。

#### 33.8.2　如何选择ABI

当我们要支持一种ABI，就需要将这种ABI对应的so放到程序安装包中对应的目录下，当程序运行时，会按照最适合当前设备的ABI去对应的目录下查找，例如，程序可能先去查找armeabi-v7a目录，找不到再去查找armeabi目录，都找不到就报错（注意这里找的是目录）。如果我们的程序支持所有的ABI，那么不论在什么设备上，都可以获得最佳的性能，但这也意味着需要将所有ABI对应的so打包到程序安装包中，大大增加了安装包的体积。

那么这么多的ABI，应该如何选择呢，armeabi是最通用的ABI，兼容所有的Android手机，但性能略差一些。armeabi-v7a比armeabi有显著的性能提升，而且目前支持armeabi-v7a的设备是最多的，如果程序同时要求性能和兼容性，那么至少在APK中同时支持armeabi和armeabi-v7a。

如果程序对性能要求不高，那么只使用armeabi即可，如果希望同时兼顾安装包的体积以及程序运行的性能，那么可以为不同的设备提供多个安装包。如果我们的项目是SDK项目，那么应该提供支持全平台ABI的so。

#### 33.8.3　如何生成对应ABI的so

当程序员决定要支持某一种ABI时，在生成so的时候，就需要做相应的配置，可以在Application.mk文件中设置APP_ABI变量来添加想要支持的ABI。

```
APP_ABI:=  "armeabi", "armeabi-v7a", "x86"
```

如果是在Android Studio下，可以通过修改app目录下的build.gradle，在android→defaultConfig→ndk下的adbfilters添加对应的ABI。

```
ndk {
    moduleName "myjni"
    ldLibs "log", "z", "m"
    adbfilters "armeabi", "armeabi-v7a", "x86"
｝
```

在配置完成之后，编译so时会依次编译多个版本的so，并生成到APK中对应的ABI目录下。如果使用了第三方的so，这里有一点需要注意的地方，假设应用程序支持了armeabi和armeabi-v7a两种ABI，那么应该同时引入第三方so的armeabi和armeabi-v7a版本，因为**不同ABI的so是不能混合使用的**！

因为程序运行时，是先确定libs目录下要使用哪一个ABI目录，然后其他ABI目录就被忽视了，假设没有将第三方so的armeabi-v7a版本放进去，那么在运行时就会找不到对应的这个so，从而崩溃。所以**要么所有用到的so都必须支持某个ABI，否则就不要支持这个ABI**！另外在编译so时，应该尽量选择更低一些的ADK版本，以获得更好的兼容性。

### 33.9　调试JNI代码

Android Studio的一个强大功能就是可以调试JNI代码，虽然调试起来并不是很方便，但可以断点调试已经很不错了，虽然据说Eclipse也可以使用GDB调试JNI代码，但之前笔者花了不少时间进行各种尝试都没能成功，出现的各种问题太多了，因此把调试环境搭好BUG早就解决了。

#### 33.9.1　Android Studio JNI调试环境搭建

Android Studio的调试功能使用起来就很方便（相对于Eclipse），如果是首次调试，需要下载LLDB插件，可以直接在Android Studio的SDK Tools下载（File菜单→Settings→Appearance&Behavior→System Settings→Android SDK→LLDB 2.1），如图33-24所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122046.jpeg)

图33-24　在SDK Tools中下载LLDB

安装完LLDB之后，断点调试只需要3步即可。

##### 1．切换为Android Native模式

只要**在项目的build.gradle中的android——defaultConfig下添加正确的NDK构建选项**，Android Studio就会**自动**在项目的运行配置中添加app-native模式以供选择，默认是app模式，将调试模式切换为app-native，如图33-25所示。

读者如果还没有安装LLDB，那么单击调试时会出现如图33-26所示的界面，提示C++ debugger package is missing or incompatible，意为C++调试工具出错或找不到，单击右边的Fix按钮可以自动下载LLDB进行修复。

##### 2．激活Jni Debuggable

接下来还需要为当前项目开启Jni Debuggable选项，如果没有开启，会看到如图33-27所示的报错：Build type isn’t JNI debuggable。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122047.jpeg)

图33-25　切换为Android Native模式

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122048.jpeg)

图33-26　LLDB丢失错误

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122049.jpeg)



图33-27　Build type isn’t JNI debuggable报错

此时可以手动在项目的build.gradle中的android→buildTypes→debug下，将jniDebuggable设置为true，也可以在File菜单中选择Project Structure，在弹出的对话框中选择Build Types选项卡，将debuge模式下的Jni Debuggable下拉列表选项框切换为true，如图33-28所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122050.jpeg)

图33-28　设置Jni Debuggable

##### 3．启动调试

接下来就可以在C/C++文件中打断点了，如果直接单击运行项目的运行按钮，是不会命中断点的，还需要单击运行按钮旁边的调试按钮来启动调试，如图33-29所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122051.jpeg)

图33-29　单击调试按钮

单击完调试按钮之后，手机会进入Waiting For Debugger状态，Android Studio的Console界面会输出启动调试的相关信息，如图33-30所示。稍等片刻就会发现程序停在了我们设置的断点处。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122052.jpeg)

图33-30　等待调试

#### 33.9.2　Android Studio的断点调试

知道了如何断点之后，下面来了解一下如何在Android Studio中进行调试，如图33-31所示。首先下方的Debugger窗口中有Frames子窗口，可以用于切换堆栈。Variables窗口可用于查看变量，LLDB窗口可以执行LLDB调试指令（GDB的升级版）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122053.jpeg)

图33-31　调试窗口

在调试窗口上的右键菜单中提供了一些便捷的调试操作，例如Evaluate Expression执行表达式，可以在打开的Evaluate Expression对话框中输入一些表达式，如在这里输入变量i，按Enter键后会输出表达式执行后的结果，这里会打印出i的值，如图33-32所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122054.jpeg)

图33-32　Evaluate Expression对话框

另外还有一个Watches窗口，可以用于添加监视变量，可以在Watches窗口中添加当前的临时变量i，如图33-33所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122055.jpeg)

图33-33　Watches窗口

最后简单介绍一些常用的Android Studio的调试快捷键。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　F7：Step Into单步执行（进入函数）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　F8：Step Over单步执行（不进入函数）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Shift + F8：Step Out跳出（执行完当前函数）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　F9：Resume Program继续执行（直到下一个断点）。