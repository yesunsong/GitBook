# 第**32**章　**Android**环境搭建

Cocos2d-x的Android环境搭建之烦琐一直为众人所诟病，而且由于使用了大量的工具，在环境搭建的过程中遇到的问题也是层出不穷，就算是经验丰富的程序员，在搭建Android环境时，也多多少少会碰到问题，但随着Cocos2d-x不断地完善，Android环境的搭建已经越来越轻松了。

通过本章的学习，读者可以了解搭建Cocos2d-x的Android环境的基本概念，总结环境搭建时容易碰到的问题，了解Eclipse和Android Studio两种环境搭建的方式，以及如何创建Android项目和打包。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　Android环境搭建基础。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　使用Eclipse打包。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　使用Android studio打包。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　新建Android项目。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　实用技巧。

### 32.1　Android环境搭建基础

Android环境的搭建看上去很烦琐，但是可以将其分解为简单的步骤，然后了解每个步骤的目的，这样就会简单很多。无论以何种方式、在哪个平台搭建Android环境，以下3个部分都是必须的。

#### 32.1.1　JDK和JRE

JDK是Java的SDK，JRE是Java的运行环境，对于一般的用户来说，安装JRE即可运行程序，而对于开发者则需要安装JDK来进行开发，**JDK本身包含了JRE**。

Android程序本身是一种Java程序，要开发Android程序，首先需要程序员能够开发Java程序。在搭建Android环境之前，必须把Java环境搭建好，而Java环境的搭建非常简单。

##### 1．安装JDK和JRE

首先在Java的官网下载JDK，网址为http://www.oracle.com/technetwork/java/javase/ downloads/index.html。下载之后运行安装程序，之后一直单击Next按钮即可完成安装，如图32-1所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123220.jpeg)

图32-1　安装JDK

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123221.jpeg)**注意：**安装路径中不要有中文，这是一个好习惯。

##### 2．设置环境变量

在安装完成之后，需要设置几个环境变量。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　JAVA_HOME：设置为JDK的安装路径。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　CLASS_PATH：设置为.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　Path：在尾部追加%JAVA_HOME%\bin;。

Windows设置环境变量的步骤如图32-2所示。

（1）右击桌面上“我的电脑”或“计算机”，在弹出的快捷菜单中选择“属性”命令。

（2）在弹出的窗口中选择“高级系统设置”。

（3）在弹出的“系统属性”对话框中单击“环境变量”按钮。

（4）双击已有的环境变量或单击“新建”按钮可以修改和添加环境变量。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123222.jpeg)

图32-2　设置环境变量

#### 32.1.2　关于ADK

ADK是Android的SDK，可用于开发Android程序，直接下载即可使用，但是需要使用另外一个工具来下载。

如使用Eclipse开发，可以用ADT来下载ADK，ADT为Android开发工具Android Develop Tools，相当于是Android的SDK大管家，可以用于管理各种版本的ADK，并提供一系列Android开发工具以供下载，如ADB、模拟器、日志打印工具等，如图32-3所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123223.jpeg)

图32-3　ADT管理ADK

如使用Android Studio开发可以直接在Android Studio中下载ADK，基本上ADT的所有功能Android Studio都有，如图32-4所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123224.jpeg)

图32-4　Android Studio管理ADK

#### 32.1.3　关于NDK

由于Cocos2d-x是使用C++开发的，为了使Cocos2d-x能够在Android上编译运行，需要使用NDK将Cocos2d-x编译为一个库供Android程序调用。读者可以在这个地址http://wear.techbrood.com/tools/sdk/ndk/根据自己的操作系统选择NDK，下载并解压即可。

### 32.2　使用Eclipse打包

搭建完基础环境之后，可以在Eclipse中编译Android程序了，首先需要下载Eclipse，可以在Eclipse的官网下载Eclipse，网址为http://www.eclipse.org/downloads/。

#### 32.2.1　打开Android项目

下载安装后运行Eclipse，然后选择“文件”→“菜单”→“新建”→“项目”命令，在打开的窗口中选择Android Project from Existing Code选项，打开一个已存在的Android项目，如图32-5所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123225.jpeg)

图32-5　打开Android项目

也可以打开Cocos2d-x自带的cpp-empty-test示例，找到cpp-empty-test下的proj.android目录并打开即可，如图32-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123226.jpeg)

图32-6　打开cpp-empty-test Android项目

#### 32.2.2　解决Java报错

在打开项目之后，很有可能发现项目中存在报错问题，如图32-7所示，此时需要把错误改正才可以继续。当然，没有错误的话就可以跳过这一步了。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123227.jpeg)

图32-7　Java报错

##### 1．引用Cocos2d-x包错误

Java报错有非常多的原因，图32-7中的错误位于导入org.cocos2dx.lib包时报错，错误信息是找不到这个包，这是一个很常见的错误。org.cocos2dx.lib封装了Cocos2d-x的Android底层框架，一般在Cocos2d-x项目中会引用到，当这个引用丢失时会报图32-7所示的错误。

解决这个问题的方法很简单，直接在Eclipse中打开Cocos2d-x引擎的Android项目即可（位于platform\android\java目录下），如图32-8所示，打开引擎的Android项目之后，CppEmptyTest项目的报错消失了，打开项目的属性→Android窗口，可以看到原本丢失引用的红色叉叉图标变成了绿色的对勾图标。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123228.jpeg)

图32-8　打开Cocos2d-x的Android包

除了打开项目之外，也可以将Cocos2d-x的Java代码复制到项目的src目录下，在Linked Resources里面修改src_common的路径到正确的路径（旧版本引擎），如图32-9所示，方法非常多。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123229.jpeg)

图32-9　添加Linked Resources

##### 2．JDK版本错误

另外一个导致Java代码报错的问题是JDK、JRE版本错误的问题，这种错误一般表现在正常的Java代码报错，例如JDK中某个对象的方法或属性错误，正常来说不会出现这个问题，但在安装了多个版本的JDK或JRE时比较容易出现这个问题。

这种情况下应该搜索报错信息来确定应该使用哪个版本的JDK，下载对应版本的JDK之后，重新配置JDK和JRE，如图32-10所示，可以在Preferences-Java-Compiler列表项的Compiler compliance level下拉列表框中选择对应的JDK版本，如果JDK和当前的JRE版本不一致，在窗口下方会有提示，单击提示的Configure链接可以进行配置。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123230.jpeg)

图32-10　修改JDK版本

可以在打开的配置界面添加对应的JRE目录，如图32-11所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123231.jpeg)

图32-11　设置JRE

设置完JRE之后，还需要对项目进行设置，在项目属性中，选择Java Build Path的Libraries选项卡，然后单击Add Library按钮，在弹出的窗口中添加JRE System Library，如图32-12所示，单击Next按钮，在弹出的窗口中选择Workspace default JRE即可。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123232.jpeg)

图32-12　为项目设置JRE

在设置好JRE之后，项目的ADK属性可能被重置，如果项目的ADK引用被重置了，只需要打开项目属性的Android界面，然后选择对应的ADK即可。

##### 3．ADK版本错误

当Java代码中调用ADK部分的代码报错时，一般是因为ADK版本错误，此时需要搜索报错的代码来确定对应的ADK，然后在ADT中下载指定的ADK，然后在项目属性的Android界面选择对应的ADK即可。

ADK的版本不对应还可能出现如图32-13所示的错误，此时可以在Application.mk文件中添加APP_PLATFORM :=android-XXX代码，指定Android的正确版本号，代码中的XXX表示Android的版本号。我们可以在ADT中看到它们的对应关系。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123233.jpeg)

图32-13　ADK版本错误

#### 32.2.3　设置NDK

##### 1．添加Builder

接下来需要添加一个Builder来编译C++代码，这里需要注意多个NDK Builder的冲突问题，如果项目原先已经有用于编译C++的Builder，那么可以直接在原先的基础上修改，如果没有就需要添加一个。添加的方法很简单，如图32-14所示，在项目属性窗口Builders项中，单击New按钮，弹出Builder的设置对话框。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123234.jpeg)

图32-14　添加Builder

在Main选项卡中单击Location下的Browse File System按钮，打开NDK路径下的ndk-build.cmd文件，单击Working Directory下的Browse Workspace.按钮，选择当前项目。

##### 2．设置NDK_MODULE_PATH

接下来还需要设置环境变量NDK_MODULE_PATH，NDK_MODULE_PATH主要用于搜索其他NDK模块（每个模块会对应一个Android.mk，通过这个Android.mk来编译指定的cpp文件生成模块），例如，需要搜索cocos下的Android模块，这里将NDK_MODULE_PATH设置为相对路径../../..;../../../cocos;../../../external;，如图32-15所示，主要根据Cocos2d-x引擎中Android.mk的存放位置决定，这里这样设置是因为笔者的项目位于引擎的相对路径下，当然也可以设置为绝对路径，或者使用环境变量。

##### 3．解决NDK编译报错

在使用NDK进行编译时，容易报以下几种错误。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123235.jpeg)

图32-15　设置NDK_MODULE_PATH

首先是找不到模块，如图32-16所示，我们的Android.mk引用了一个模块，Cocos2dx模块，而NDK没有找到该模块，所以报错，之前的版本是直接包含Cocos2dx模块的Android.mk，所以不会报错。而这里使用了import，错误信息提示我们，可以通过设置NDK_MODULE_PATH环境变量来解决错误，但这个环境变量比较容易设置错误。一般**一个Android.mk包含一个或多个模块**，当你要import Cocos2dx模块时，会在这个路径% NDK_MODULE_PATH\cocos2dx下寻找Android.mk文件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123236.jpeg)

图32-16　找不到模块

接下来是Builder冲突问题，当已经激活了一个默认的C++Builder时（一般只有在安装了CDT插件之后才会出现），可能会报以下错误，此时可以将默认的Builder修改为NDK Builder，也可以将默认的Builder禁用。

```
Cannot run program "bash": Launching failed
Error: Program "bash" not found in PATH
```

有时还可能遇到NDK版本问题，太久或者太新的NDK编译都有可能报错，或者引用到一些NDK没有提供的方法也有可能报错，建议使用NDKr9～NOK10b之间的版本来编译。当编译出错时，有些问题可以通过包含头文件来解决，例如图32-17所示的问题，可以通过包含<cstdio>来解决。有些问题则需要调整代码，例如这个版本的NDK不支持abs方法，可能需要自己实现一个，或者使用std::abs等，NDK的编译错误基本都需要根据实际的报错问题来解决，因此这里很难将所有问题进行概括。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123237.jpeg)

图32-17　NDK编译报错

当编译完成后有可能会出现一个找不到静态库的问题，如图32-18所示。该问题与NDK版本相关，R7以上的版本已经修复这个BUG了，把<NDK>\sources\cxx-stl\gnu-libstdc++\libs\armeabi\目录下的libgnustl_static.a复制到obj\local\armeabi\libgnustl_static.a目录下即可解决。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123238.jpeg)

图32-18　NDK编译后报错

#### 32.2.4　解决打包报错

在生成apk时，需要手动**将游戏的资源复制到assets目录下**，当资源文件的文件名存在中文或空格时，可能会导致打包失败，这时需要修改资源文件名。当资源文件中存在gz文件时，也可能导致打包资源失败，如图32-19所示，这时需要删除gz文件，或将该文件修改为其他格式。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123239.jpeg)

图32-19　资源打包失败

除了资源问题，还有一些其他问题也可能导致生成apk失败，在控制台会报Could not find HelloCpp.apk错误，如图32-20所示，出现该问题可以尝试用以下步骤来解决。

首先是有可能是没设置JRE，当Eclipse中没有设置JRE时，无法生成apk，在项目属性中选择Java Build Path→Libraries→Add library选项，设置JRE。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123240.jpeg)

图32-20　生成apk失败

然后还有可能是ADK的设置有误，先删掉原来的ADK再重新添加，然后选择工程→Android Tools→Fix Project Properties。

另外还有可能是ADT版本太低不兼容，或者是ADK Build-Tool没有安装，升级ADT或安装ADK Build-Tool后再解决该问题。

#### 32.2.5　解决运行报错

运行程序后闪退有以下几种常见的问题。

##### 1．资源问题导致的闪退

当忘记将res或Resource目录下的资源复制到Android项目的assets目录下时，访问该资源时会报错。

另外是**大小写的问题，Android是区分大小写的**，如果大小写不一致，也会导致加载资源报错。还有，如果**将文件分隔符/写成了\**，也会导致在Android平台下加载资源失败。

##### 2．so问题导致的崩溃

由于生成的so有问题导致的崩溃，可尝试重新生成so，也有可能是NDK的问题，因为NDK导致生成的so有问题，还有可能是生成的so引用了其他的so，但**apk没有找到对应的so**导致，如图32-21所示。比较常见的情况是C++代码编译出错了，但是程序员没注意，直接安装了没有包含so文件的apk。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123241.jpeg)

图32-21　找不到so导致的崩溃

##### 3．第三方库的ABI问题

ABI是Application Binary Interface的缩写，不同CPU架构的Android设备可以支持不同的ABI，常见的ABI有armeabi、armeabi-v7a、mips、x86等。只有使用支持的ABI编译出来的so才可以在对应的设备上运行，其中armeabi是兼容性最好的，但性能不是最佳，被所有的Android设备支持。第33章中会详细介绍ABI。

当程序引用了一些第三方的库，一般第三方库会提供多个ABI的动态链接库.so文件或静态库.a文件，如果程序同时支持了多个ABI，需要保证每个ABI的目录下都有对应的so，否则程序会由于找不到对应的库而崩溃。

例如，应用程序同时支持armeabi和armeabi-v7a，当需要添加一个新的so时，需要把armeabi和armeabi-v7a版本的so放到Android项目libs目录对应的目录下。

##### 4．模拟器版本过低

模拟器版本过低是一个在模拟器下比较容易出现的OpenGL问题，主要因为模拟器不支持OpenGL ES 2.0，如图32-22所示。创建Android 4.0以上的模拟器，并在创建时将Gpu emulation设置为true即可解决。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123242.jpeg)

图32-22　模拟器OpenGL报错

关于模拟器，可以尽量使用高版本的Android系统，因为高版本是兼容低版本应用的，并且使用高版本的Android系统的模拟器会相对快一些，如果需要反复调试，请不要关闭模拟器，因为模拟器的启动是很耗时间的，启动之后再次调试会直接进入调试状态，而如果调试完关闭模拟器，再次调试就需要重新启动，但建议还是在真机上调试比较好一些。

##### 5．so自动加载失败

so自动加载失败是笔者碰到的另外一个与so相关的棘手问题，笔者在接入fmod音效库时，按照正常的流程添加so，但在部分手机（Android 4.4以下）上出现加载失败的情况，如图32-23所示，报了Cannot load library : soinfo_link_image(linker.cpp:1652)错误，so文件放的位置以及文件本身都是没有问题的，并且Android 4.4以上的版本都可以加载，这个错误与Android的so自动加载机制有关，在Android 4.4以下的版本，我们需要在Java代码中手动加载。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123243.jpeg)

图32-23　动态加载so失败

手动加载的方式是在Activity中添加一个static作用域，并调用System.loadLibrary()函数将so文件手动加载，示例代码如下。

```
public class AppActivity extends Cocos2dxActivity {
    static
    {
         //手动加载so
        System.loadLibrary("fmod");
         System.loadLibrary("fmodstudio");
    }
```

### 32.3　使用Android Studio打包

在**Cocos2d-x-3.7rc0这个版本之后**支持了Android Studio的打包，Android Studio是一个不错的IDE，相比Eclipse各方面都有提升，这也是Android发展的一个趋势，但目前（到Cocos2d-x 3.12版本为止），Cocos2d-x对Android Studio的支持实际上只是一发烟雾弹，因为程序员**可以在不安装Android Studio的情况下使用Cocos2d-x的Android Studio工程进行编译打包**。

#### 32.3.1　cocos compile命令

Android Studio的打包非常简单，有两个前提条件，一个是Android基础环境搭建，也就是JDK、JRE、ADT、ADK、NDK等环境的搭建，另外一个是执行了Cocos2d-x引擎目录下的setup.py，正确设置了环境变量。如果一切设置正确，则可以在命令行使用cocos compile命令来编译打包APK。输入cocos compile –h命令可以查看帮助信息，cocos compile的帮助信息非常多，如图32-24是cocos compile支持的编译选项。

如图32-25是Android相关的参数，有时候当编译出现报错时，由于多个CPU同时编译，屏幕可能会输出错乱的信息，这种情况下可以添加-j1参数，指定只有一个CPU编译，这样就可以获取到正确的错误信息了。

另外，如果使用了Lua脚本，在编译时动态使用--lua-encrypt选项为Lua脚本进行加密也是一个很方便的功能。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123244.jpeg)

图32-24　cocos compile编译选项

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123245.jpeg)

图32-25　Android相关参数

#### 32.3.2　编译打包Android Studio

要编译android-studio项目，需要是**Cocos2d-x-3.7rc0以及之后的引擎才可以，**在命令行进入项目的目录，然后执行cocos compile -p android --android-studio命令，即可编译Android Studio项目，如图32-26所示（如果不希望使用Android Studio工程来编译，也可以把--android-studio参数去掉，这样会直接使用项目下的proj.android来进行打包）。

**初次编译会下载一些东西**，时间比较久，大概需要十几分钟的下载时间。编译完成后可以在项目目录的bin\android\debug下找到生成好的apk，如果需要发布release版本，则需要在编译时加上-m release选项，在最后会要求输入签名文件和相关密码来生成release版本，**release版本的性能可是比debug版本高出一大截呢**。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123213.jpeg)

图32-26　编译Android Studio

#### 32.3.3　打包遇到的问题

在使用ndk-r12b打包时，笔者遇到了relocation overflow in R_ARM_THM_CALL错误，如图32-27所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123246.jpeg)

图32-27　relocation overflow in R_ARM_THM_CALL错误

当程序员使用Thumb指令集编译时可能导致该错误，观察图32-28中的编译信息，可以发现是使用Thumb进行编译的，只要将Thumb指令集修改为ARM指令集即可。程序员可以在报错的Android.mk文件中添加上下面一行代码来修改指令集：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123247.jpeg)

图32-28　Thumb编译

```
LOCAL_ARM_MODE := arm
```

在Cocos2d-x中，本身就有该行代码，但是需要依赖自定义的USE_ARM_MODE变量来开启，程序员可以在Application.mk中加上该行代码来开启ARM模式。

```
USE_ARM_MODE := 1
```

修改完再编译，可以发现图32-29中，编译信息中的编译模式从Thumb切换到了ARM。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123248.jpeg)

图32-29　ARM编译

ARM处理支持两种通用指令集，分别是32位的ARM和16位的Thumb，Thumb可以看作是ARM的子集，Thumb模式编译出来的代码占用的存储空间要比ARM模式更加节省，只有ARM模式的60%～70%，但Thumb模式下需要执行的指令比ARM模式多30%～40%，在ARM模式下一条复杂指令可以完成的工作，Thumb模式下可能需要两三条指令，但由于Thumb指令简单，所以Thumb代码的功耗会更低一些。如果使用32位的存储器，ARM代码会更加高效，而在16位存储器下则是Thumb代码更高效，目前大部分Android设备都是32位的。

### 32.4　新建Android项目

在Cocos2d-x 2.1.3以及之前的版本，并没有一个很好的途径让程序员去创建一个Android项目，官方的方式是在Coco2d-x的根目录下找到create-android-project.bat或者create-android-project.sh来执行创建一个新的Android项目，需要程序员配置ADK、NDK、CYGWIN（Windows下）的环境变量，输入包名和项目名会在Cocos2d-x下创建新项目，但一般程序员不会用CYGWIN，因为太庞大了，所以也就不使用create-android-project脚本了，大部分是在Cocos2d-x自带的Android示例上进行修改。

在Cocos2d-x 2.1.4之后的版本开始使用Python脚本来统一创建新项目，3.x版本的Python脚本比2.x版本更加强大，可以使用cocos命令一键生成各个平台的项目，也可以使用cocos命令进行打包编译。

在使用cocos命令之前，需要做两件事情：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　安装Python 2.7。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123219.jpeg)　在命令行下执行引擎目录下的setup.py脚本进行初始化，如图32-30所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123249.jpeg)

图32-30　执行setup.py脚本

setup.py会要求程序员输入NDK_ROOT、ANDROID_SDK_ROOT（ADT的安装路径），以及ANT_ROOT等环境变量。

完成设置之后就可以使用cocos new命令来创建一个新的项目。输入cocos new -h可以查看创建项目的帮助信息，如图32-31所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123250.jpeg)

图32-31　cocos new帮助信息

帮助信息中还列出了每一个选项的详细介绍，如图32-32所示。

使用cocos new命令依次传入项目名、包名、使用的语言，可以创建一个新项目，如图32-33所示。进入项目目录即可看到各个平台的project目录。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123251.jpeg)

图32-32　cocos new参数介绍

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623123252.jpeg)

图32-33　创建新项目

### 32.5　实用技巧

#### 32.5.1　关于版本问题

由于打包所需要的工具比较多，各个工具之间的版本兼容性问题是一个关键点，出现了问题有些情况下还不容易定位，那么应该使用哪个版本的NDK呢？程序员如何定位当前出现的问题是工具的问题，还是项目自身的问题，或者是Cocos2d-x的问题呢？

程序员通过引擎目录下的README.md文件，可以了解到应该使用哪些版本的工具进行编译，在文件中可以找到Build Requirements和Runtime Requirements的相关信息，它们分别提出了编译和运行当前版本引擎所需的环境，以下是3.12版本的需求介绍。

```
Build Requirements
------------------
* Mac OS X 10.7+, Xcode 5.1+
* or Ubuntu 12.10+, CMake 2.6+
* or Windows 7+, VS 2013+
* Python 2.7.5
* NDK r11+ is required to build Android games
* Windows Phone/Store 8.1 VS 2013 Update 4+ or VS 2015
* Windows Phone/Store 10.0 VS 2015
* JRE or JDK 1.6+ is required for web publishing
Runtime Requirements
--------------------
  * iOS 6.0+ for iPhone / iPad games
  * Android 2.3.3+ for Android games
  * Windows 8.1 or Windows 10.0 for Windows Phone/Store 8.1 games
  * Windows 10.0 for Windows Phone/Store 10.0 games
  * OS X v10.6+ for Mac games
  * Windows 7+ for Win games
  * Modern browsers and IE 9+ for web games
```

例如，笔者使用NDK r9c版本来编译Cocos2d-x 3.12，在指定armeabi-v7a之后出现了莫名其妙的崩溃，一使用异步加载资源程序就闪退，通过阅读README.md文件后，发现当前版本引擎需要NDK r11及以上的版本，于是，笔者使用了最新的NDK r12编译出了可正常运行的apk。

如果要使用新的NDK来编译，除了修改环境变量NDK_ROOT之外，还需要重新启动计算机，我在Window 7下修改了该变量之后注销并重新登录计算机，发现编译时有部分代码仍然使用了旧的NDK来编译，但关机重启电脑之后就一切正常了，所以修改了NDK的环境变量之后，建议重启一下计算机，这样会稳妥一些。

#### 32.5.2　关于减少包体积

Cocos2d-x现在可以说是非常庞大了，增加了各种各样的功能，但大多数情况下程序员并不需要那么多东西，减少包体积的方法很简单，从Android.mk中去除引用，有时还需要添加一些预处理进行禁用，下面简单介绍一下。

例如，很多项目都没有使用到物理引擎，那么如何屏蔽掉物理引擎呢？首先可以修改引擎目录下的cocos目录中的Android.mk，在引用物理引擎的代码前添加#注释掉对物理引擎的引用，请留意下面加粗字体的代码：

```
LOCAL_STATIC_LIBRARIES := cocos_freetype2_static
LOCAL_STATIC_LIBRARIES += cocos_png_static
LOCAL_STATIC_LIBRARIES += cocos_jpeg_static
LOCAL_STATIC_LIBRARIES += cocos_tiff_static
LOCAL_STATIC_LIBRARIES += cocos_webp_static
# 首先注释掉静态库的引用
#LOCAL_STATIC_LIBRARIES += cocos_chipmunk_static
LOCAL_STATIC_LIBRARIES += cocos_zlib_static
#LOCAL_STATIC_LIBRARIES += recast_static#LOCAL_STATIC_LIBRARIES += bullet_static
LOCAL_WHOLE_STATIC_LIBRARIES := cocos2dxandroid_static
# define the macro to compile through support/zip_support/ioapi.c
LOCAL_CFLAGS   :=  -DUSE_FILE32API
LOCAL_CFLAGS   +=  -fexceptions
LOCAL_CPPFLAGS := -Wno-deprecated-declarations
LOCAL_EXPORT_CFLAGS   := -DUSE_FILE32API
LOCAL_EXPORT_CPPFLAGS := -Wno-deprecated-declarations
include $(BUILD_STATIC_LIBRARY)
#==============================================================
include $(CLEAR_VARS)
LOCAL_MODULE := cocos2dx_static
LOCAL_MODULE_FILENAME := libcocos2d
LOCAL_STATIC_LIBRARIES := cocostudio_static
LOCAL_STATIC_LIBRARIES += cocosbuilder_static
LOCAL_STATIC_LIBRARIES += cocos3d_static
LOCAL_STATIC_LIBRARIES += spine_static
LOCAL_STATIC_LIBRARIES += cocos_network_static
LOCAL_STATIC_LIBRARIES += audioengine_static
include $(BUILD_STATIC_LIBRARY)
#==============================================================
$(call import-module,freetype2/prebuilt/android)
$(call import-module,platform/android)
$(call import-module,png/prebuilt/android)
$(call import-module,zlib/prebuilt/android)
$(call import-module,jpeg/prebuilt/android)
$(call import-module,tiff/prebuilt/android)
$(call import-module,webp/prebuilt/android)
# 去除引入对应的NDK模块
#$(call import-module,chipmunk/prebuilt/android)
$(call import-module,3d)
$(call import-module,audio/android)
$(call import-module,editor-support/cocosbuilder)
$(call import-module,editor-support/cocostudio)
$(call import-module,editor-support/spine)
$(call import-module,network)
$(call import-module,ui)
$(call import-module,extensions)
#$(call import-module,Box2D)#$(call import-module,bullet)#$(call import-module,recast)
$(call import-module,curl/prebuilt/android)
$(call import-module,websockets/prebuilt/android)
$(call import-module,flatbuffers)
```

此外，还需要在Application.mk文件中添加禁用物理引擎的编译选项，否则会编译失败。

```
APP_CPPFLAGS := -DCC_ENABLE_CHIPMUNK_INTEGRATION=0
APP_CPPFLAGS := -DCC_ENABLE_BOX2D_INTEGRATION=0
APP_CPPFLAGS := -DCC_USE_3D_PHYSICS=0
APP_CPPFLAGS := -DCC_USE_PHYSICS=0
```

对于其他要禁用的功能，程序员可以通过搜索CC_USE来判断是否有相关的预处理需要进行屏蔽。