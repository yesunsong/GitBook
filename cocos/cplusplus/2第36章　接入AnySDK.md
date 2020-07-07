# 第**36**章　接入**AnySDK**

在使用Cocos2d-x开发跨平台的手机游戏时，各种第三方SDK的接入令人头疼，程序员经常需要接入统计、分享、推送、广告以及各大渠道的平台SDK，开发和维护成本都非常高。而触控官方的AnySDK则大大降低了接入SDK的成本。

AnySDK是Cocos官方推出的一套第三方SDK接入解决方案，其操作简单、功能强大并且完全免费，除了Cocos2d-x之外，还支持Android、iOS、Unity、HTML 5等平台和引擎。使用AnySDK可以让程序员很大程度上从烦琐的SDK接入中解放出来，专注于游戏内容的开发。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　AnySDK概述。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　接入AnySDK Android框架。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　接入AnySDK iOS框架。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　登录流程。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　支付流程。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　母包联调。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　打包工具

### 36.1　AnySDK概述

在学习如何接入AnySDK之前，先从整体上了解一下AnySDK，为什么要接入AnySDK，以及如何接入AnySDK。

#### 36.1.1　为什么要接入AnySDK

目前国内有大大小小上百家手游渠道，每一个渠道都会要求开发者在游戏中正确接入相应渠道的登录及支付SDK，才能通过渠道审核并上架，如图36-1所示。在接入SDK的过程中会有以下一些问题。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　由于每一家渠道SDK的设计不同，SDK里自带的资源文件、功能接口等都不一样。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　在同一份游戏代码中无法同时接入多个渠道的SDK，因此**开发者必须维护多套游戏代码项目来分别接入各家渠道SDK**。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　网络游戏的服务端也需要为每个渠道实现对应的登录和支付验证接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　除了需要对各个渠道进行复杂的设置之外，还需要手动编写Objective-C和JNI、Java代码。

根据AnySDK收集的数据显示，一个有经验的开发者平均接入一款渠道SDK需要耗费的时间（客户端接入+服务端对接）大概在两到三天之间，而如果是之前没有接入SDK经验的开发者，这个时间会增加两倍，因为有很多渠道的特殊需求及“潜规则”需要了解。

更严重的问题是，当游戏要接入上线的渠道非常多的时候，耗费的时间数量会成倍增长，也就是说开发者接入前一款SDK时所做的工作并不能减少接入下一款SDK需要的工作量。就算程序员完成接入了，后续的更新和维护工作也令人痛苦，如图36-1所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122359.jpeg)

图36-1　痛苦的开发者

接入SDK几乎是每个游戏都要做的工作，但凡是重复性的工作，必然可以通过工具、方法进行简化、省略。AnySDK就是一款为开发者加速接入第三方SDK的工具。

AnySDK可以帮开发者实现**只接入一次就可以批量打出所有渠道包**，并且不需要关心SDK的版本更新和处理因为渠道服务端接口变化造成的紧急重复更新工作。AnySDK提供了各个语言版本的框架，开发者只需选择自己熟悉的开发语言即可。

根据AnySDK的文档操作接入一次，花费的时间与手工接入一个渠道SDK的时间相同，然后就可以通过AnySDK提供的打包工具打包出包含不同SDK的渠道包。

#### 36.1.2　AnySDK架构简介

AnySDK的整体架构如图36-2所示，AnySDK主要由3大部分组成，AnySDK管理后台+AnySDK服务器、AnySDK Framework客户端框架以及AnySDK打包工具。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　AnySDK管理后台可以对开发者的游戏、渠道、支付、登录验证等信息进行管理，还可以创建和管理测试账号。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　AnySDK服务器是**游戏服务器与各个渠道服务器对接的一个中间层**，每个渠道的登录和支付验证协议都不相同，AnySDK服务器统一了协议。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　AnySDK Framework客户端框架是游戏程序和渠道SDK的中间层，与AnySDK服务器类似，在客户端的代码中只需要接入AnySDK Framework，就可以很方便地接入各个渠道SDK，无须修改客户端代码，开发者可以在Cocos2d-x、Android、iOS、Unity、HTML 5等平台和引擎中接入AnySDK Framework客户端框架，接入了AnySDK Framework客户端框架的客户端生成的程序包被称为母包，开发者会使用它来为每个渠道生成一个特定的渠道包。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　AnySDK打包工具是**快速将母包生成渠道包**的工具，每个渠道都会有一些配置，开发者需要在AnySDK打包工具中进行设置，如Icon角标、包后缀、支付回调、AppID和AppKey等，设置好渠道的配置信息之后，就可以一键生成渠道包。需要注意的是**iOS是输入一个工程，然后在工程中生成多个渠道的Xcode项目，开发者需要为每个渠道的子项目单独打包**。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122400.jpeg)

图36-2　AnySDK框架

#### 36.1.3　AnySDK快速接入指引

接下来简单介绍一下接入AnySDK的步骤。

##### 1．准备工作

首先需要注册一个AnySDK的账号，用于登录AnySDK打包工具以及AnySDK管理后台，网址为http://dev.anysdk.com/，管理后台的使用可以参考**http://docs.anysdk.com/开发者管理后台使用说明**。

注册完账号之后，需要下载安装并登录AnySDK客户端工具，如图36-3所示。

登录之后可以看到如图36-4所示的界面，左侧是安妮市场、打包工具等按钮，右侧是各类资讯信息，包括最新的渠道公告、行业资讯、AnySDK最新公告以及AnySDK美女主程的生活和工作详情等，主界面的最右侧还有程序员老黄历这个实用的隐藏功能。

开发者需要在打包工具中新建一个游戏，创建完游戏后可以得到游戏的AppKey、AppSecret以及PrivateKey，具体的步骤可以参考AnySDK的客户端使用手册**http://docs.anysdk.com/PackageTool**。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122401.jpeg)

图36-3　登录AnySDK客户端工具

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122402.jpeg)

图36-4　AnySDK客户端工具主界面

然后在安妮市场中下载AnySDK Framework为Cocos2d-x提供的C++框架，如图36-5所示，开发者需要为Android和iOS平台下载对应的C++框架（这里也提供了Unity、Lua、JS、Java版本的框架下载）。安妮市场除了可以下载AnySDK Framework之外，还提供了各种插件、SDK和工具下载。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122403.jpeg)

图36-5　安妮市场

##### 2．接入AnySDK Framework

完成准备工作之后，需要在Cocos2d-x项目中接入AnySDK Framework，Cocos2d-x 3.13之后的版本是内置了AnySDK，所以可以直接使用。

AnySDK Framework的接入大致可以分为两步，即环境搭建和代码接入，开发者需要为Android和iOS分别进行环境搭建（Cocos2d-x 3.13之后的版本已经搭建好了环境）。

完成环境搭建之后，需要在游戏逻辑中接入AnySDK，例如，单击“登录”按钮调用AnySDK的登录接口、单击“购买道具”时调用AnySDK支付接口，AnySDK Framework根据市场上存在的第三方SDK概括为6大系统，即用户系统、支付系统、广告系统、统计系统、社交系统、分享系统。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　用户系统接口：登录、注销、切换账号、平台中心、显示悬浮按钮、隐藏悬浮按钮、显示退出页、显示暂停页等渠道相关功能。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　支付系统接口：支付、获取订单号。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　广告系统接口：显示广告、隐藏广告。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　统计系统接口：开始会话、结束会话、设置会话时长、设置是否捕捉异常、异常错误信息报告、自定义事件等统计功能。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　社交系统接口：提交分数、显示排行榜、解锁成就、显示成就榜。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　分享系统接口：分享。

在完成AnySDK Framework的接入后，需要编译游戏项目生成母包，这些内容稍后会进行介绍。

##### 3．搭建服务器与前后端联调

如果开发者开发的是一个网络游戏，那么还需要为服务器接入AnySDK，Web服务器的接入非常简单，AnySDK提供了各种语言的Web服务器示例，在github上可以下载到，网址为https://github.com/AnySDK/Sample_Server，只需要参考Demo即可。但C/C++等服务器的接入则需要变通一下，后面会介绍到C/C++服务器如何接入AnySDK。

在前后端都完成AnySDK的接入后，可以使用测试账号对母包进行调试，将登录、支付等功能调通，可以参考AnySDK的文档，网址为http://docs.anysdk.com/Debug-User，稍后会对最核心的登录和支付流程进行详细介绍，AnySDK提供了母包测试的功能，开发者可以在AnySDK后台添加测试账号，使用母包来调试登录和支付功能。

##### 4．打包渠道包

将调试好的项目打包生成母包，可以使用AnySDK打包工具来批量生成渠道包。如何生成母包，在AnySDK的客户端使用手册中有详细介绍。

从前面的步骤来看，AnySDK的接入并不算轻松，其工作量与正常接入一个渠道的SDK差不多，但接入AnySDK之后，可以很轻松地接入其他渠道，只需要在打包工具中进行简单的配置即可完成一个渠道SDK的接入。虽然有些渠道需要进行一些特殊处理，但在AnySDK的官方文档中非常详尽地介绍了这些问题的处理。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　Android常见问题：http://docs.anysdk.com/SDKParams。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　iOS常见问题：http://docs.anysdk.com/IOS-SDKParams。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　H5常见问题：http://docs.anysdk.com/H5-SDKParams。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　打包常见问题：http://docs.anysdk.com/Client-faqs。

### 36.2　接入AnySDK Android框架

在安妮市场中下载C++（Android）框架之后，解压出来可以发现3个protocols目录，如图36-6所示，开发者需要根据自己项目的STL标准库类型来决定使用哪个目录的内容。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122404.jpeg)

图36-6　C++（Android）框架

在Android项目的Application.mk文件中可以找到开发者的项目当前的STL标准库类型，如APP_STL := gnustl_static表示当前项目以gnu静态库的方式引入STL标准库，需要使用protocols_gnustl_static目录下的框架。

AnySDK Android框架的接入包含两个部分，将AnySDK Framework导入到Cocos2d-x项目中，以及在Cocos2d-x项目中完成初始化。另外，AnySDK还提供了一个示例项目供开发者参考，网址为https://github.com/AnySDK/Sample_CPP_Cocos2dx。

#### 36.2.1　导入Android AnySDK Framework

C++（Android）框架包含了静态库和对应的头文件、jar包、C++头文件以及res资源目录，接下来了解一下如何导入这些文件。

##### 1．复制AnySDK Framework

在Cocos2d-x的android工程目录下面新建protocols目录，然后将开发者选择的对应版本框架protocols目录下的include文件夹和android文件夹复制到protocols目录下。然后将框架目录下的res文件夹中的所有资源文件拷贝到android项目对应的文件中。

##### 2．修改Android.mk文件配置framework编译选项

这一步是修改游戏工程中C++代码的NDK编译配置文件Android.mk，将AnySDK提供的framework库链接到游戏工程的库中。

（1）将protocols目录添加到NDK_MODULE_PATH环境变量中，在android.mk第一行LOCAL_PATH := $(call my-dir)下面新加一行代码如下：

```
LOCAL_PATH := $(call my-dir)
$(call import-add-path,$(LOCAL_PATH)/../)
```

（2）添加AnySDK framework静态库声明，在Android.mk文件的LOCAL_C_INCLUDES声明下面添加一行代码如下：

```
LOCAL_WHOLE_STATIC_LIBRARIES := PluginProtocolStatic
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122405.jpeg)**注意：**此处注意语法规则，如果工程原有mk文件中没有其他LOCAL_WHOLE_STATIC_ LIBRARIES声明，则添加上面的代码即可，如果mk文件中有其他的**LOCAL_WHOLE_STATIC_LIBRARIES声明，那么就需要加在原有声明之后，并且将:=修改为+=**。我们必须添加到LOCAL_WHOLE_STATIC_LIBRARIES中，而不能添加到**LOCAL_STATIC_LIBRARIES**，否则会导致AnySDK部分函数找不到。

（3）添加库路径声明代码，在Android.mk文件的最后一行添加以下代码，用于导入框架中android目录下的Android.mk文件。

```
$(call import-module,protocols/android)
```

##### 3．导入框架自带的ja

如果开发者是用Eclipse工具开发，右击Eclipse工程，在弹出的快捷菜单中选择Properties，然后选择Java Build Path，在面板上选择Libraries，单击Add JARs...将libPluginProtocol.jar引进游戏工程，如图36-7所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122406.jpeg)

图36-7　导入jar包

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122407.jpeg)**说明：****游戏工程的Android API最小支持10**。

如果使用cocos compile命令编译的话，需要将jar包放在libs里（部分版本是放在jars里）。

##### 4．配置AndroidManifest.xml添加框架需要的权限

此外，还需要在AndroidManifest.xml中添加框架所需要的权限。

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.RESTART_PACKAGES" />
<uses-permission
android:name="android.permission.KILL_BACKGROUND_PROCESSES" />
```

一般来说，即便不集成AnySDK Framework，大部分的项目也都会注册申请这些权限。

#### 36.2.2　初始化AnySDK Framework

导入了AnySDK Framework之后，还需要对AnySDK Framework进行初始化，之后才能使用AnySDK的接口。

##### 1．初始化JavaVM

开发者需要在游戏工程加载jni的时候为AnySDK framework设置JavaVM引用，在游戏的Android项目目录下的jni目录下找到main.cpp添加代码，首先需要导入头文件并声明命名空间。

```
#include "PluginJniHelper.h"
using namespace anysdk::framework ;
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122408.jpeg)**注意：**此处导入头文件时要根据项目设定的头文件定义路径来写，以保证编译时能成功找到相应头文件。

接下来需要添加设置JavaVM的代码，若此处已有其他引擎初始化JavaVM的代码，保留其代码并在后面添加PluginJniHelper::setJavaVM(vm); 即可。

```
PluginJniHelper::setJavaVM(vm); //add for plugin
```

Cocos2d-x 2.x版本(proj.android/jni/hellocpp/main.cpp)、Cocos2d-x 3.3r0之前版本的JavaVM初始化代码如下：

```
#include "PluginJniHelper.h"
#define  LOG_TAG    "main"
#define  LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,LOG_TAG,__VA_
ARGS__)
using namespace cocos2d;
using namespace anysdk::framework;
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
  JniHelper::setJavaVM(vm);
  PluginJniHelper::setJavaVM(vm);
  return JNI_VERSION_1_4;
}
```

Cocos2d-x 3.3rc0及以上版本的JavaVM初始化代码如下：

```
#include "PluginJniHelper.h"
#define  LOG_TAG    "main"
#define  LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,LOG_TAG,__VA_
ARGS__)
using namespace cocos2d;
using namespace anysdk::framework;
void cocos_android_app_init (JNIEnv* env, jobject thiz) {
    LOGD("cocos_android_app_init");
    AppDelegate *pAppDelegate = new AppDelegate();
    JavaVM* vm;
    env->GetJavaVM(&vm);
    PluginJniHelper::setJavaVM(vm);
}
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122409.jpeg)**说明：**因为setJavaVM需要在onCreate()方法之前，所以写在JNI_OnLoad里肯定没错。Cocos2d-x 3.3rc0及其以上版本的cocos_android_app_init是在onCreate()方法之前的，所以也可以写在这里。Cocos2d-3.x版本头文件需要写全路径，例如3.2版本#include "../../../../proj.android/protocols/android/PluginJniHelper.h"。

##### 2．在Java层初始化AnySDK Framework框架

首先找到游戏工程的主Activity，以cocos2d-x引擎游戏为例，主Activity即是继承了Cocos2dxActivity的MainActivity。然后在主Activity的onCreate()方法中新增如下代码来初始化AnySDK Framework：

```
import com.anysdk.framework.PluginWrapper;
public class MainActivity extends Activity{
    protected void onCreate(Bundle savedState)
    {
        super.onCreate(savedState);
        PluginWrapper.init(this); //for plugins
    }
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122409.jpeg)**说明：**AnySDK的回调函数默认是在主线程，**使用Cocos2d-x的话，可以在onCreate()方法中加上PluginWrapper.setGLSurfaceView(Cocos2dxGLSurfaceView.getInstance());将回调改成在GL线程里操作**，如不在GL线程里操作界面会有问题。

另外还需要重写Activity生命周期相关方法，代码如下：

```
@Override
protected void onDestroy() {
    PluginWrapper.onDestroy();
    super.onDestroy();
}
@Override
protected void onPause() {
    PluginWrapper.onPause();
    super.onPause();
}
@Override
protected void onResume() {
    PluginWrapper.onResume();
    super.onResume();
}
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data)
{
    PluginWrapper.onActivityResult(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
@Override
protected void onNewIntent(Intent intent) {
    PluginWrapper.onNewIntent(intent);
    super.onNewIntent(intent);
}
@Override
protected void onStop() {
    PluginWrapper.onStop();
    super.onStop();
}
@Override
protected void onRestart() {
    PluginWrapper.onRestart();
    super.onRestart();
}
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**说明：**在Cocos2d-x 3.0之后的版本中集成Cocos2dxActivity之后已经不需要手动实现Activity里的生命周期方法，因此**如果开发者发现主Activity没有这些方法，就需要自己去重写这个方法（直接复制上面的代码片段也可以），并且super.onCreate (savedState);应在PluginWrapper.init(this);之前调用，因为调用init()函数的时候需要调用C++函数，而so文件是在onCreate()调用时加载。**

##### 3．在C++层初始化AnySDK Framework框架

在C++层调用任何AnySDK Framework函数之前都需要调用init函数进行框架初始化，推荐在java层初始化完成之后通知C++层初始化框架，代码如下：

```
#include "AgentManager.h"
using namespace anysdk::framework;
std::string appKey = "BC26F841-OOOO-OOOO-OOOO-OOOOOOOOOOOO";
std::string appSecret = "1dff378a8f254ecOOOOOOOOOOOOO";
std::string privateKey = "696064B29E9A0OOOOOOOOOOOOO";
std::string oauthLoginServer = "http://oauth.anysdk.com/api/
OauthLoginDemo/Login.php";
AgentManager::getInstance()->init(appKey,appSecret,privateKey,oauthLogi
nServer);
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**说明：**appKey、appSecret、privateKey这3个参数**是在打包工具客户端创建游戏之后生成的游戏唯一参数**，可以在打包工具游戏管理界面查看到。

而oauthLoginServer参数是游戏服务提供的用来做登录验证转发的接口地址，在此处配置的接口地址仅用于sim sdk测试模式下（即直接运行母包时）做登录时框架请求的地址，而在正式打出渠道包的时候会被替换成相应渠道在打包工具中配置的地址参数。

##### 4．加载及卸载SDK插件

在初始化框架完成之后加载所有集成的SDK，代码如下：

```
AgentManager::getInstance()->loadAllPlugins();//对插件进行初始化,包括对各个
sdk的初始化
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**注意：**由于部分SDK在初始化时涉及SDK闪屏的操作，因此强烈建议在完成AnySDK Framework框架初始化后调用加载插件操作，代码如下：

```
import com.anysdk.framework.PluginWrapper;
public class MainActivity extends Activity{
    protected void onCreate(Bundle savedState){
    super.onCreate(savedState);
    PluginWrapper.init(this);             //for plugins
    wrapper.nativeInitPlugins();          //通过jni调用用初始化函数
}
void Java_com_anysdk_sample_wrapper_nativeInitPlugins(JNIEnv* env, jobject
thiz)
{
       AgentManager::getInstance()->loadAllPlugins();
}
```

当游戏不需要插件时，可对插件进行卸载。

```
AgentManager::getInstance()->unloadAllPlugins();//对插件进行卸载，需要卸载时
可调用
```

##### 5．代码混淆

如果要混淆java代码，请不要混淆联编的jar包中的类。可以添加以下类到proguard配置，排除在混淆之外。

```
-keep class com.anysdk.framework.** {*;}
-keep class com.anysdk.Util.SdkHttpListener {*;}
```

### 36.3　接入AnySDK iOS框架

在安妮市场中下载C++（iOS）框架之后，解压出来也可以发现3个protocols目录，如图36-8所示，开发者需要根据项目的C++标准库类型来决定使用哪个目录的内容。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122411.jpeg)

图36-8　C++（iOS）框架

在Xcode中查看项目的Build Settings中的C++ Standard Library选项，如图36-9所示。根据标准库的类型来选择对应的目录，如libc++对应protocols_libc++目录。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122412.jpeg)

图36-9　C++ Standard Library选项

AnySDK iOS框架的内容看上去比Android框架要简单，iOS框架的接入也包含两个部分，即将AnySDK Framework导入到Cocos2d-x项目中并进行相应的设置，以及在Cocos2d-x项目中完成初始化。

#### 36.3.1　导入AnySDK Framework

##### 1．删除多余的Target

目前打包工具只支持单个Target，若有多个Target会导致打包出来的工程文件缺失，无法正常运行，可在目标上右击，在弹出的快捷菜单中选择“删除”命令，如图36-10所示。

如果有其他Target所需要用到的同名.plist文件，也需要删除，否则打包后工程AnySDK文件夹下可能会缺失.plist文件，一般Cocos2d-x的Xcode项目会有ios和mac两个目录，如图36-11所示，这里可以删除mac目录下的info.plist文件，也可以将整个mac目录删除。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122413.jpeg)

36-10　删除多余的Target

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122414.jpeg)



图36-11　删除mac目录或Info.plist文件

##### 2．将图标转为Asset Catalog形式

在母工程中需要将Icon转为Asset Catalog形式，否则可能导致打包后图标替换错误或者闪屏替换出现问题。

打开项目设置的General选项，如图36-12所示，若按钮为此样式，单击该Use Asset Catalog按钮，Xcode会自动转化原有图标，生成一个Images.xcassets文件夹。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122415.jpeg)

图36-12　转换App图标为Asset Catalog形式

##### 3．在项目中引用libPluginProtocol

一般有两种方式可以引入一个库文件，添加文件到项目中或添加lib到项目中，两种方式都可以，除了导入静态库之外，还需要将静态库的include目录添加到头文件搜索路径中，开发者可以在项目设置的Search Paths下的User Header Search Paths中添加搜索路径。

添加文件的方式导入静态库，需要在项目上右击，在弹出的快捷菜单中单击Add Files to XXX，之后会弹出文件选择对话框，找到对应的protocols目录，选取并单击add，就完成了libPluginProtocol库的导入，如图36-13所示，注意在添加时要选中Create groups单选按钮。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122416.jpeg)

图36-13　添加libPluginProtocol库

请不要修改.a的文件名（不同框架下文件名可能为libPluginProtocol.a, libPluginProtocol_libc++.a或libPluginProtocol_libstdc++.a），否则打包时检测框架版本号时会报错。

另外一种方式是添加lib到项目中，在Xcode中选中项目，选择TARGETS→Build Phases→Link Binary With Libraries，单击“+”按钮，如图36-14所示，弹出一个界面，单击Add Other，之后会弹出文件选择对话框，找到libPluginProtocol.a，选取并单击open，就完成了libPluginProtocol库的导入。

##### 4．导入框架依赖库

libPluginProtocol需要依赖以下几个系统库：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　CFNetwork.framework；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　CoreFoundation.framework；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　MobileCoreServices.framework；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　SystemConfiguration.framework；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　libz.dylib(Xcode7:libz.tbd)。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122417.jpeg)

图36-14　导入静态库

##### 5．添加链接参数

打开项目工程配置，添加库的链接参数，在项目工程配置中，找到Linking中的Other Linker Flags，添加-ObjC参数，如图36-15所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122418.jpeg)

图36-15　添加链接参数

如果添加链接参数后编译报错，需要根据错误信息来解决，如图36-16所示的错误，可以通过导入系统库MediaPlayer.framework和GameController.framework解决。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122419.jpeg)

图36-16　编译出错

使用Cocos2d-x2.2.x版本编译时还可能报其他错误，解决方法可以参考AnySDK的官方文档http://docs.anysdk.com/CppTutorial。

#### 36.3.2　初始化AnySDK Framework

导入了AnySDK Framework之后，还需要对AnySDK Framework进行初始化才能使用AnySDK的接口。在iOS中初始化AnySDK Framework非常简单，只需要在项目启动时调用初始化AnySDK Framework的代码即可。

开发者可以在Cocos2d-x的Xcode项目的iOS目录下的AppController.mm文件中进行初始化，首先包含头文件以及引入命名空间。

```
#include "AgentManager.h"
using namespace anysdk::framework;
```

然后在didFinishLaunchingWithOptions方法中添加初始化的代码如下。

```
//获取AgentManager
AgentManager* agent = AgentManager::getInstance();
std::string appKey = "Your APPKEY";
std::string appSecret = "Your APPSECRET";
std::string privateKey = "Your PRIVITEKEY";
std::string oauthLoginServer = "http://oauth.anysdk.com/api/
OauthLoginDemo/Login.php";
//初始化Agent
agent->init(appKey, appSecret, privateKey, oauthLoginServer);
//加载插件
agent->loadAllPlugins();
```

这段代码与Android中C++层的初始化是一样的，开发者也可以将初始化统一放到C++的AppDelegate中，在C++中进行初始化。

### 36.4　登录流程

登录是所有网络游戏都必须接入的一个核心功能，这个功能需要客户端和服务端协同完成。接下来了解一下AnySDK的登录流程，以及客户端和服务端如何接入登录。

#### 36.4.1　登录流程简介

正常的登录流程只是客户端发送账号密码给服务器，服务器进行校验并将结果返回给客户端，但接入了SDK之后的流程会复杂一些，完整的登录流程如图36-17所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122420.jpeg)

36-17　登录流程

客户端、平台SDK服务器、AnySDK服务器以及游戏服务器参与了整个登录流程，并将该流程划分为8个步骤。

（1）游戏客户端调用AnySDK框架login接口，客户端弹出渠道SDK登录界面，输入账号密码后单击登录，相应渠道SDK内部向渠道平台服务器发起登录请求。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**说明：**要接到AnySDK初始化成功的回调之后，才可以调用AnySDK框架的登录接口。

（2）渠道SDK服务器校验平台用户登录成功后，返回的用户ID和授权码token，这里的用户ID和授权码相当于用户的账号和密码。

（3）AnySDK框架从渠道SDK中获取到用户ID和授权码，然后向游戏服务器发送请求去做用户登录信息验证（此步请求的接口地址就是开发者填写在打包工具渠道参数上的用户登录验证地址）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**说明：**以上3步都是由AnySDK Framework完成的，不需要开发者再写代码，这个流程开发者是无法干预的。

（4）游戏服务器接收到客户端请求过来的参数之后，将用户信息等参数再转发给AnySDK服务器（此步骤执行的操作在下面有提供PHP和Java语言的示例代码，开发者可以直接使用或者参考实现）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**说明：**由于每个渠道SDK登录得到的参数个数与内容都是不同的，因此游戏服务器要遍历接收到的请求中的所有参数并全部转发给AnySDK服务器，否则无法通过登录验证。

（5）AnySDK服务器接收到游戏服务器发送过来的请求数据后去向对应的渠道服务器进行用户登录验证。

（6）AnySDK服务器接收渠道服务器的验证结果并获得用户最终的token。

（7）AnySDK服务器将渠道服务器返回的验证结果转发给游戏服务器。

（8）游戏服务器再返回通知AnySDK框架登录验证结果，并可以返回一些游戏逻辑相关的数据给游戏客户端。

（9）AnySDK框架执行登录回调函数来通知游戏客户端是否登录成功。

#### 36.4.2　客户端接入登录

在客户端接入登录功能非常简单，在完成AnySDK Framework的接入后，只需要调用AnySDK用户系统的登录接口，然后设置登录回调来处理登录结果即可。具体可以参考AnySDK官方的用户系统接入手册http://docs.anysdk.com/Usersystem。

##### 1．SDK初始化回调

首先需要确保SDK初始化成功才可以调用AnySDK的登录函数，当开发者调用AgentManager的loadAllPlugins接口时，会自动对插件和SDK进行初始化，但SDK是否初始化成功，需要根据SDK初始化回调来确定。

要获取SDK的初始化回调就需要设计一个类继承于UserActionListener，并重写其onActionResult()方法，如AnySDK自带示例的PluginChannel类，开发者的各种操作AnySDK框架都会通过onActionResult()方法将操作结果通知给开发者，如SDK初始化、用户登录等操作。

```
class PluginChannel:public UserActionListener
{
public:
virtual void onActionResult(ProtocolUser* pPlugin, UserActionResultCode
code, const char* msg)
{
    switch(code)
    {
    case kInitSuccess://初始化SDK成功回调
         //SDK初始化成功，游戏相关处理
         break;
    case kInitFail://初始化SDK失败回调
         //SDK初始化失败，游戏相关处理
         break;
    }
}
}
```

接下来需要创建一个PluginChannel对象，并绑定到UserPlugin中，通过调用AgentManager的getUserPlugin()方法获取UserPlugin对象，然后调用UserPlugin对象的setActionListener()方法绑定，代码如下（这段代码是PluginChannel内部调用的，所以可以直接传入this）：

```
if(AgentManager::getInstance()->getUserPlugin())
{
    AgentManager::getInstance()->getUserPlugin()->setActionListener(this);
}
```

##### 2．调用登录

在SDK初始化成功之后，可以调用UserPlugin的login()方法来登录，开发者可以直接调用login()方法，也可以传入一个字符串map，将一些参数传递给游戏服务器。代码如下：

```
//调用用户系统登录功能
void PluginChannel::login()
{
    ProtocolUser* _pUser = AgentManager::getInstance()->getUserPlugin();
    if(!_pUser) return;
    map<string, string> info;
    info["server_id"] = "2";
    info["server_url"] = "http://xxx.xxx.xxx";
    info["key1"] = "value1";
    info["key2"] = "value2";
    _pUser->login(info);
}
```

登录参数可以传入一个map，可传入服务器id(server_id)、登录验证地址（server_url）和透传参数（任意key值）。

服务器ID：key为server_id，服务端收到的参数名为server_id，不传则默认为1。

登录验证地址：key为server_url，传入的地址将覆盖配置的登录验证地址。

透传参数：key任意（以上两个key除外），服务端收到的参数名为server_ext_for_login，是个JSON字符串。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**说明：**除了在代码中添加透传参数，另外还可以在**AnySDK客户端的渠道参数配置中配置登录验证透传参数，服务端收到的参数名为server_ext_for_client（代码中添加的透传参数会被放到server_ext_for_login中）。**

开发者可以在onActionResult()方法中获取登录结果，就像获取SDK初始化结果一样，如表36-1中列出了登录操作会触发的回调。

表36-1　登录回调信息

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122421.jpeg)

UserPlugin还提供了其他辅助接口，如bool isLogined()方法可用于判断是否已经登录。std::string getUserID()方法可用于获取用户ID。UserPlugin还提供了如账号切换、注销等接口，可以参考AnySDK的用户系统接入手册。

#### 36.4.3　服务端接入登录

AnySDK要求开发者提供一个Web服务器来实现登录验证，开发者需要将服务器登录验证的URL配置到打包工具中，也可以在客户端调用login()方法时放到map参数中传入。

##### 1．Web服务器的登录验证

Web服务器的功能非常简单，只是原样地将客户端登录验证请求转发给AnySDK服务器，然后将验证结果原样返回给客户端。AnySDK提供了各种语言的web服务器示例，参考网址为https://github.com/AnySDK/Sample_Server。

开发者可以直接使用AnySDK提供的服务器代码，然后根据自己的需求适当调整代码，如果开发者的游戏是单机游戏，还可以不架设自己的服务器，直接使用AnySDK登录验证地址http://oauth.anysdk.com/api/User/LoginOauth/。AnySDK还提供了服务端登录功能接入手册http://docs.anysdk.com/OauthLogin。

##### 2．C/C++服务器的登录验证

如果开发者的游戏服务器本身是Web服务器，那么直接使用AnySDK提供的服务器示例代码就可以了，但如果开发者使用的是C/C++实现的长连接服务器，那么就需要做一些额外的处理。

当客户端通过Web服务器的验证之后，要去连接C/C++服务器，这时候C/C++如何验证客户端是否已经登录了呢？这里有两种验证方法。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　第一种方式是在Web服务器验证通过之后生成一个密码，或者直接使用渠道SDK的token字段作为密码，将这个密码通过扩展字段告知客户端，并将密码写入数据库中。

客户端在登录C/C++服务器时提交用户ID和该密码进行验证，C/C++服务器将用户提交的密码与数据库中的密码进行对比校验。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　第二种方式是在Web服务器验证通过之后，将客户端登录请求的内容通过扩展字段返回给客户端，客户端将登录请求的内容发给C/C++服务器，C/C++服务器再次请求AnySDK服务器进行验证。相当于客户端的一次登录操作，开发者向AnySDK验证了两次，一次是Web服务器的验证，另一次是C/C++服务器的验证。

重复的验证看上去有些多余，那么是否可以只在C/C++服务器验证，把Web服务器去掉呢？从AnySDK的登录验证流程来看是不可以的，因为开发者调用了login之后，就只能等待AnySDK的登录回调，AnySDK框架会自动发起HTTP登录验证请求。

不论使用哪种方式进行验证，都需要通过扩展字段向客户端返回数据，AnySDK统一登录验证获取的数据统一以Json格式返回，其中包含status、data、common、ext这4个子域部分。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　status：用于表达验证请求成功与否。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　data：保存渠道平台返回的验证信息（此部分数据的格式根据不同渠道的实现不同而有所差异，有些渠道会返回较多的用户信息数据，有些渠道只会返回校验结果，AnySDK返回这些数据是为了能让游戏服务器获取到所有的原始验证信息，开发者可以使用这些数据，也可以直接忽略这部分数据，不会影响接入）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　common：包含渠道编号、渠道SDK标识，渠道返回的用户userId（渠道唯一），以及用户登录前选择的游戏服务器ID（此数据只有当客户端调用带serverId的重载登录函数时才会有值，否则默认为空），开发者请使用此部分统一数据作为用户数据。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122358.jpeg)　ext：默认为空，游戏服务器可以在ext域中存放游戏逻辑相关的数据（比如开发商服务器内部设定用户标识），**这些数据会在游戏客户端获取到登录成功回调时附带的msg信息里完整地拿到**，然后用来执行相应的游戏逻辑。

如果登录成功，AnySDK服务器会返回类似下面代码的一串Json字符串，开发者可以向ext字段添加任何信息来返回给客户端。

```
{
   "status":"ok",
   "data":
   {
        "id":"18135798",
        "name":"\u6b27\u9ea6\u560e\u5730",
        "avatar":"http:\/\/u1.qhimg.com\/qhimg\/quc\/48_48\/22\/02\/55\
        /220255dq9816.3eceac.jpg?f=6c449fbaaa093e52b4053e46170af079",
        "sex":"\u672a\u77e5",
        "area":"",
        "nick":""
   },
   "common":
   {
        "channel":"000023",
        "user_sdk":"360",
        "uid":"18135798",
        "server_id":"1",
        "plugin_id":"12"
   },
   "ext":""
}
```

### 36.5　支付流程

支付是游戏的一个重要功能，是网络游戏最主要的收入来源，支付SDK可以让玩家在游戏中充值购买游戏道具，这个功能也需要前后端协同完成。

#### 36.5.1　支付流程

支付流程是一个异步的流程，要注意的问题比登录流程多一些，登录流程出现问题的话，最多是玩家登录不了，而支付流程出问题则可能出现玩家充值了但程序没有发给道具，或者玩家一次充值程序重复发放了道具，所以支付流程的处理应该更加严谨。在游戏服务器和渠道SDK服务器之间，同样是通过AnySDK服务器来进行中转，接入AnySDK的整个支付流程如图36-18所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122422.jpeg)

图36-18　支付流程图

（1）游戏客户端调用AnySDK框架支付接口（payForProduct），AnySDK框架请求AnySDK服务器生成本次交易订单号。

（2）AnySDK服务器返回订单号给客户端，AnySDK框架获取到AnySDK服务器生成的支付订单号。

（3）AnySDK框架调用渠道SDK支付接口向渠道平台服务器请求支付。

（4）支付成功后，渠道支付SDK会返回支付成功通知AnySDK框架，框架再执行支付成功回调函数通知游戏客户端。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**注意：****这里SDK返回的成功回调只是指交易已经成功提交给渠道服务器，并不表示此笔订单已经支付成功**，有些渠道支付SDK是只要进入支付选择界面，SDK生成订单就会返回支付成功回调，即使玩家取消支付或者支付失败，因此订单的支付结果只能以**游戏服务器是否接收到渠道服务器的支付订单回调信息**为准，而不能以本地支付回调为准。

（5）渠道平台服务器完成订单支付后发送订单验证信息异步通知AnySDK服务器。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**注意：**这里需要开发者在渠道后台（即开发者获取SDK参数的页面）**将支付订单回调**

**地址配置为AnySDK提供的各渠道专用的支付回调地址**，回调地址可以在打包工具参数配置页面查看。

（6）AnySDK服务器对渠道服务器发送过来的订单信息做校验，并返回正确的响应。

（7）AnySDK服务器将订单支付结果信息推送到游戏服务器提供的支付订单回调地址。此地址在打包工具配置渠道参数界面配置。

（8）游戏服务器对AnySDK推送过来的信息做校验，只要游戏服务器完成本次通知处理，不论是确认订单有效发放道具或是订单无效丢弃，请务必返回正确的响应ok或OK，否则AnySDK会重复通知。

（9）游戏服务器验证支付通知并发放道具。

#### 36.5.2　客户端接入支付

由于整个支付流程是异步的，所以客户端的支付需要处理两部分，首先是请求支付，然后是道具发放的处理。当然，在使用支付插件之前，需要调用AgentManager的loadAllPlugins先初始化插件，跟用户插件一样。具体可以参考AnySDK官方的支付系统接入手册http://docs.anysdk.com/IAPSystem。

##### 1．请求支付

AnySDK框架支持多个支付插件，AnySDK客户端选择多少个支付，getIAPPlugin就能获取到多少个。一般情况只会选择一个支付插件，如果选择多个支付插件需要开发者自己提供相关界面完成多支付的逻辑展示。

```
std::map<std::string , ProtocolIAP*>* _pluginsIAPMap= AgentManager::
getInstance()->getIAPPlugin();
std::map<std::string , ProtocolIAP*>::iterator it = _pluginsIAPMap->
begin();
if(_pluginsIAPMap)
{
   if(_pluginsIAPMap->size() == 1)//只存在一种支付方式
   {
        (it->second)->payForProduct(productInfo);
   }
   else //多种支付方式
   {
   //开发者需要自己设计多支付方式的逻辑及UI
   }
}
```

调用插件的void payForProduct(TProductInfo info);方法可以发起支付请求，该方法要求传入一个TProductInfo结构体，TProductInfo实际上是一个map，记录了支付的一些相关的信息，开发者需要提供表36-2中的参数。

表36-2　支付参数

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122423.jpeg)

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122410.jpeg)**注意：**调用支付函数时需要传入的一些玩家信息参数（如角色名称、ID、等级）都是渠道强制需求（如UC、小米），并非AnySDK收集所用，如果开发者不填或者填假数据都会导致渠道上架无法通过。必传参数不能为空，若当前没有可用的值可以写任意值上去，个别渠道可能还需要添加其他参数，请参考常见问题中的渠道说明，根据渠道判断并添加上相应参数。

Product_Id是支付请求中最关键的一个参数，就是要购买的商品ID，开发者需要在渠道的开发者后台添加对应的商品，如果同一个商品在不同的渠道有着不同的商品ID，那么开发者可以在AnySDK的开发者后台添加对应的商品映射，然后在代码中使用统一的商品ID，具体操作可以参考AnySDK官方的商品映射文档http://docs.anysdk.com/ProductMapping。

##### 2．发放道具

在请求支付之前，需要设置好支付结果的回调，要处理支付回调就需要设计一个类继承于PayResultListener，并重写其onPayResult()方法，如AnySDK自带示例的PluginChannel类，AnySDK会调用onPayResult()方法传入支付的结果。

```
class PluginChannel:public PayResultListener
{
public:
    //支付回调
    virtual void onPayResult(PayResultCode ret, const char* msg,
    TProductInfo info)
    {
        //处理回调函数
        switch(code)
        {
        case kPaySuccess:                           //支付成功回调
        //支付成功后，游戏相关处理
             break;
        case kPayNetworkError:                      //支付网络出错回调
        case kPayCancel:                            //支付取消回调
        case kPayProductionInforIncomplete: //支付信息填写不完整回调
        case kPayFail:                              //支付失败回调
        //支付失败后，游戏相关处理
            break;
        /**
        * 新增加:正在进行中回调
        * 支付过程中若SDK没有回调结果，就认为支付正在进行中
        * 游戏开发商可让玩家去判断是否需要等待，若不等待则进行下一次的支付
        */
        case kPayNowPaying:
            break;
        }
    }
}
```

通过AgentManager的getIAPPlugin()方法获取IAP插件map，遍历所有的IAP插件，调用它们的setResultListener()方法将回调对象设置进去（下面这段代码是PluginChannel内部调用的，所以可以直接传入this）：

```
std::map<std::string , ProtocolIAP*>* _pluginsIAPMap= AgentManager::
getInstance()->getIAPPlugin();
std::map<std::string , ProtocolIAP*>::iterator iter;
for(iter = _pluginsIAPMap->begin(); iter != _pluginsIAPMap->end(); iter++)
{
    (iter->second)->setResultListener(this);
}
```

在支付流程图中道具是由游戏服务器发放的，道具的发放应该包含两个处理，把道具添加到数据库中，以及通知客户端获得了道具，由于支付流程图中的游戏服务器是一个Web服务器，而Web服务器要给客户端做主动推送还是比较麻烦的，所以建议由客户端主动查询游戏服务器，在客户端接收到支付成功之后，每隔1～2秒请求一下游戏服务器支付的结果，在完成本次支付之后才允许玩家进行下一次支付。

由于支付流程有可能由于服务器的问题导致一直没有发放道具，那么客户端可以做一个超时处理，例如超过1分钟，提示玩家支付超时，让玩家稍后重试或询问客服人员，调用支付插件的resetPayState方法重置支付状态。

#### 36.5.3　服务端接入支付

与登录类似，开发者需要提供一个支付回调地址给AnySDK，用于接收AnySDK返回的支付结果，在回调中开发者需要进行IP校验、验签、以及订单状态检查，这些判断都通过之后给玩家发放道具，最后返回字符串OK给AnySDK服务器。

IP校验可以避免白名单以外的IP发送假的支付结果给支付服务器，通过签名验证可以保证传入的支付结果是有效的，假冒的支付结果无法验签通过，这样双重验证就确保了支付结果的安全性。

AnySDK提供的服务器示例实现了除发放道具外的所有逻辑，开发者可以直接在AnySDK的服务器示例上添加发放道具的逻辑即可。

如果是客户端在支付之后定时向服务器查询，那么订单支付失败了也应该将失败的结果返回给客户端，然客户端可以结束这次支付。

除了考虑支付失败的问题，发放道具的逻辑还需要考虑AnySDK服务器重复回调的问题，在回复OK字符串给AnySDK服务器之前，AnySD服务器会以特定的时间间隔回调支付地址，有可能开发者正在处理该订单，AnySDK服务器又针对该订单回调支付地址，所以开发者需要将已经发放道具的订单号存到数据库中，每次回调先判断该订单是否已处理。

如果不希望数据库被已完成的订单塞满，可以将订单号写入高效的Redis数据库中，并设置2天的超时时间，因为AnySDK的通知间隔不会超过2天，这样开发者只需要判断当前的订单是否与2天内的已完成订单重复即可。

如果希望在C/C++服务器接入支付功能，流程也很简单，在发放完道具（写入游戏数据库）之后，将订单信息写入Redis数据库中，C/C++服务器收到客户端发起的查询请求后，从Reids数据库中查询该订单的支付结果，并将结果返回给客户端。

### 36.6　母包联调

接入登录和支付这两个核心功能之后，就可以对客户端和服务端进行联调了，可以将母包打包成渠道包再进行联调，也可以使用AnySDK的测试账号直接用母包进行联调。

要联调母包，首先需要在AnySDK的开发者后台创建测试账号，如图36-19所示，开发者还可以在这里编辑测试账号，修改测试账号的密码和余额，账号中的余额可以用来测试支付。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122424.jpeg)

图36-19　开发者后台

#### 36.6.1　调试登录

在手机上运行母包，在调用登录时可以看到AnySDK提供的测试登录界面，如图36-20所示，输入测试账号和密码可以测试登录功能。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122425.jpeg)

图36-20　测试登录

#### 36.6.2　调试支付

在登录之后，调用AnySDK的支付插件时也会弹出AnySDK提供的支付测试界面，如图36-21所示，通过测试账号的余额可以测试支付功能。支付之后可以在AnySDK开发者后台查看测试账号的余额。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122426.jpeg)

图36-21　测试支付

在调试时，客户端不需要调整任何代码，但服务端需要根据注释开启母包调试的代码才可以进行母包调试。

另外由于调试支付功能需要开发者提供一个外网的回调地址，如果开发者处于内网环境中，没有外网IP的话，在个人调试时，可以使用花生壳软件，通过端口映射让AnySDK能够访问到服务器。

如果开发的是单机游戏，AnySDK提供了一个伪游服地址（http://pay.anysdk.com/ callbacktest/devnull.php），该地址只是单纯的返回ok。由于有的SDK的客户端回调并不准确，会有并未充值却回调成功的情况，所以建议用户搭建服务器接收通知，客户端再去服务端查订单结果。

### 36.7　打包工具

AnySDK的打包工具功能强大，配置也较烦琐，在AnySDK的客户端使用手册中有详细介绍http://docs.anysdk.com/PackageTool。

文档中详细介绍了工具的使用和配置方法，以及常见问题，然而iOS的打包并没有说明，iOS不同于Android，Android的打包要求输入的是母包APK，输出的是渠道包APK，可以直接使用。但iOS要求输入的是母包的xcodeproj，输出的是渠道包的xcodeproj，如图36-22所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122427.jpeg)

图36-22　打包iOS

选择渠道之后进行打包，打包完成后会弹出如图36-23所示的界面，单击右侧的“打开工程”按钮会在Xcode中打开生成的渠道xcodeproj文件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122428.jpeg)

图36-23　打包iOS完成

打包之后AnySDK会在项目的目录下增加一个proj.ios_mac_anysdk目录，母包和渠道包的xcodeproj都在该目录下，每个渠道的xcodeproj文件名都会以渠道编号为后缀，例如，海马玩渠道生成的是XXX-500017.xcodeproj，如图36-24所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623122429.jpeg)

图36-24　AnySDK生成的项目目录

之后就只需要使用proj.ios_mac_anysdk目录下的母包项目即可，原先的proj.ios_mac目录可以不用了。之后打包IPA文件还需要在每个渠道项目上执行Xcode的打包操作，具体可以参考34.1节的内容。

关于打包工具还有一点需要**特别注意**的是，修改打包工具中的参数配置是会同步到AnySDK后台，并立即生效！同时对于之前已经发布的版本，也是会生效的。

笔者在使用的过程中出现了一个严重问题。在开发的游戏上线之后，希望搭建另外一套和正式环境一样的沙箱环境，当正式环境出了问题之后可以在沙箱环境中测试，定位问题。服务器搭建完成之后，笔者就打了一个沙箱环境的包，这个包在打包时将打包工具中的登录和支付参数配置为了沙箱环境的回调地址，修改之后**直接导致了线上版本中的玩家登录不了，以及充值没有发放道具**。因为这个参数配置并不是打到安装包里的，而是配置到了后台，正式环境下的包从后台获取配置，所以也会被影响，于是笔者在AnySDK中另外新建了一个XXX沙箱应用，专门用于沙箱调试。

### 36.8　小结

虽然看上去AnySDK的内容很多，但一旦上手了，后面带来的方便远大于学习这套工具时的麻烦，正所谓磨刀不误砍柴工，在前期花一些时间来学习了解AnySDK是明智的选择，而且AnySDK文档丰富，技术讨论群中又有技术人员积极解决问题，学习成本比想像中要低。

如果免费版本满足不了需求，那么可以购买企业版本可以得到7×24小时的技术支持，如果需要接入的渠道AnySDK不支持，还可以根据AnySDK的文档和示例自行开发插件，具体可以参考AnySDK的插件自助开发手册http://docs.anysdk.com/Sh-overview。

除了登录和支付，AnySDK还支持统计、分享、广告、社交、推送等SDK的统一接入，这些基本都不需要服务器接入，每个系统都会提供相应的插件供开发者使用，这些系统的使用方法在AnySDK的官方文档中有详细介绍http://docs.anysdk.com/。