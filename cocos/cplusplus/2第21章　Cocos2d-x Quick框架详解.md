# 第**21**章　**Cocos2d-x Quick**框架详解

本章可以让读者快速地了解并熟悉Quick框架，掌握Quick框架的正确用法和一些使用技巧，并掌握Quick框架的运行流程和一些重要功能的实现，了解Quick和Cocos2d-x以及其原生框架的区别。虽然Cocos2d-x官方已经不怎么维护Quick了，但Quick作为一个受欢迎的框架，是值得学习了解的。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Quick简介。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Quick框架结构。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用Quick。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Quick运行流程分析。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Quick脚本框架详解。

### 21.1　Quick简介

什么是Quick？Quick是一个基于Cocos2d-x的Lua框架，与Cocos2d-x原生的Lua框架相比，Quick框架更加完善、易用、强大。

Quick和Cocos2d-x的原生Lua框架又有什么联系和区别呢？Quick依赖于Cocos2d-x的原生Lua框架，主要是在此基础上进行了**扩展以及修改**。另外，原生Lua框架的使用更偏向于面向过程，而Quick则更偏向于面向对象，这一点在浏览这两个框架的示例代码时可以明显察觉到。

为什么要使用Quick？Quick作为Lua开发的首选，首先一个很简单的原因就是因为其非常流行，用户多，并且被Cocos2d-x官方所收编。Quick本身是很精简的一个轻量级的框架，使用方便，易于学习，有丰富扩展和强大的模拟器。这些优点让程序员使用Quick进行开发能够大大提高开发效率，降低开发成本。

但是，如果项目是以C++代码为主，只在一些地方少量地使用了Lua时，则并没有使用Quick的必要，直接使用原生的Lua框架会更方便一些，使用一个工具是因为其可以恰当地解决问题，而不是因为这个工具流行就用它，盲目跟风并不可取。

### 21.2　Quick框架结构

Quick到底对Cocos2d-x的原生Lua框架做了怎样的扩展和修改呢？原生Lua框架由Lua内核、Lua脚本引擎、Lua转换层、Lua辅助层以及其他的一些扩展组成。除了Lua辅助层中的Lua脚本之外，其他的C++代码被封装为了libluacocos2d库。

整个Quick框架由Lua内核、Lua脚本引擎、Lua转换层、**Quick核心层**、**Quick脚本框架**，以及丰富的Quick扩展组成，另外再加上方便易用的Quick模拟器。

首先介绍一下Quick核心层，Quick在此基础上对Lua转换层添加了一些手动扩展，主要是对cc.Node添加的一些扩展，扩展主要针对节点的触摸功能。这些代码被放到了引擎下的external\lua\quick目录下，使用VisualStudio打开Cocos2d-x可以在libluacocos2d项目的quick筛选器下快速找到这些代码，代码量并不多，只有5个头文件和5个源文件。这几个文件提供了Quick的C++支持，**只需要包含这10个文件，就可以使原生的Lua框架能够使用Quick**（当然，还需要调用一下register_all_quick_manual()方法）。

Quick核心层主要提供了节点触摸的简单接口，如setTouchEnabled()、setTouchSwallowEnabled()、setTouchMode()等常用的方法，并为此提供了一套相应的节点-触摸管理机制。setTouchEnabled()方法除了注册点击监听之外，Quick核心层会将节点添加到LuaNodeManager中进行管理，节点需要执行cleanup方法才会清除LuaNodeManager中对应的Node，如果一个节点被释放了，但却没有执行cleanup()方法，例如执行了这样的代码removeAllChildrenWithCleanup(false)，最后当节点被释放的时候，节点在EventDispatcher中所注册的点击事件会被注销，但并不会从LuaNodeManager中被清理，当我们再次点击时，LuaNodeManager试图操作这个已经被释放的节点，此时程序就会崩溃。所以，**每个注册了点击监听的节点，都应该保证其cleanup()方法能够被执行**。

Quick脚本框架**替换了原生Lua框架中的Lua辅助层**，使用了部分Lua辅助层的代码，并进行了重构。在提供了原有Lua辅助层的功能的前提下，优化了Node、Action、UI等模块的接口，封装了大量易用的UI控件，搭建了基于mvc模型的框架。

Quick扩展库包含了Json、Sqlite、Zlib等库，以及文件操作、MD5加密、Base64加密、filters特效、网络、iOS的IAP支付等功能。扩展库的代码位于引擎目录的cocos\quick_libs目录下。

Quick模拟器可以很方便地在Mac和Windows下运行使用Quick开发的程序，模拟器本身也是一个Cocos2d-x程序。除了可以方便地调试程序之外，Quick模拟器还提供了大量示例代码的预览，如图21-1所示，以及项目的创建、打包、项目列表等功能，这些功能视Quick模拟器的版本而定，后面会介绍到。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121326.jpeg)

图21-1　Quick示例预览界面

### 21.3　使用Quick

#### 21.3.1　创建Quick项目

有很多途径可以创建Quick项目，并且每一种途径创建的Quick项目都不一样，有时太多的选择比没有选择更加令人纠结。下面简单点评一下这几种途径。

##### 1．使用Cocos引擎

严格来说，**Cocos引擎创建的Lua项目并非基于Quick框架，而是基于原生Lua框架**。它只是吸收了Quick框架的模拟器以及MVC框架，功能上相对于完整的Quick框架要少很多（Cocos引擎目前已**不提供下载**，只能通过下载Cocos2d-x源码，并使用python脚本进行创建）。

使用Cocos引擎创建Lua项目，这种方式非常**方便快捷**，只需要在Cocos引擎（Cocos最新的统一入口）中创建一个Cocos项目，然后将语言选择为Lua即可，如图21-2所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121327.jpeg)

图21-2　使用Cocos引擎创建Lua项目

Cocos会自动为程序员创建一个模拟器程序，并在项目目录下生成一个src目录，src目录下包含了原生Lua框架的脚本，Quick的MVC框架，以及默认的场景脚本。

在项目的frameworks\runtime-src目录下可以找到模拟器程序的项目文件，在这里可以添加一些自定义的C++文件到项目中，例如添加一些自定义的C++类，然后导出给Lua使用。

不过这种方式创建的Lua项目**难以调试Cocos2d-x引擎的源码**，同时也找不到Quick示例项目的代码。难以调试Cocos2d-x引擎源码的原因是在Cocos中手动创建的项目，是链接到已经编译好的lib文件中，并且Cocos引擎没有提供这些源码（没有源文件，自然无法调试源码），而在最新版本的框架中，已经可以找到引擎的项目文件了。在Cocos引擎安装目录的frameworks目录下，找到对应的Cocos2d-x引擎版本，这里使用的是3.8版本，在引擎的build目录下，可以找到引擎的项目文件。

##### 2．使用Quick 3.5

使用Quick 3.5来创建Quick项目，可以在Cocos商店中获取Quick 3.5，这是当前Quick最新的**正式版本**，下载完Quick 3.5之后将其解压。解压之后可以在引擎目录下找到quick目录，这里放了Quick的framework以及丰富的示例代码。Quick 3.5对应Cocos2d-x 3.5版本的引擎，在这里需要使用命令行创建方式来创建Quick项目，在安装了python之后，**执行引擎目录下的setup.py**，然后可以使用cocos命令来创建、编译、运行项目，具体的使用方法可以参考引擎目录下的README.md文档。下面的代码演示了使用cocos new命令来创建lua项目。

```
cocos new MyGame -p com.your_company.mygame -l lua -d NEW_PROJECTS_DIR
```

cocos命令后面的new表示创建一个新项目，MyGame为项目的名字，而-p后面跟着的是包的名字，-l后面跟着的是语言，这里使用了Lua，默认的语言是cpp。-d指定了项目的输出路径，这里需要将NEW_PROJECTS_DIR替换为程序员自己的路径。

##### 3．使用Quick-Player

使用Quick-Player来创建Quick项目应该说是最华丽的方式了，Quick-Player是一个模拟器，但与Cocos自动创建的模拟器有较大的区别，Quick-Player相当于是Quick的入口，有两个途径可以获得Quick-Player，当然，这两个Quick-Player并不一样。

第一种是下载Quick 3.3版本，可以在https://github.com/dualface/v3quick里下载。第二是从github下载最新的Quick版本，下载地址是https://github.com/chukong/quick-cocos2d-x。下载后解压，**并执行解压目录下的setup脚本**（setup_win.bat或setup_mac.sh）。从github上下载的Quick还包含了清晰的介绍文档，在解压后的目录中可以找到README.html文件，从其中可以了解到一些有用的信息，值得阅读一下！但从下载的源码上来看，github上最新版本的Quick对应的Cocos2d-x版本是2.2.6，与当前最新的3.9版本相差甚远，而且Cocos2d-x 4.0版本也将呼之欲出，所以github上的Quick版本有些过时了。

在player目录下可以找到Quick-Player，运行后可以看到如图21-3所示的界面（这是最新的Quick版本，与Quick 3.3版本的界面不同）。在这个界面中可以创建Quick项目，但这里笔者使用最新版本的Quick-Player碰到过创建不了的情况，这种情况下还可以选择使用命令行创建。在bin目录下执行create_project脚本即可创建项目，执行create_project -h可以查看帮助，输入以下命令可以创建一个Quick项目。

```
create_project -p com.quick2dx.samples.hello
```

create_project命令支持以下参数。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-h：显示帮助信息。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-p：包的名字。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-o：项目的输出路径（默认为当前路径+最后的包名）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-r：屏幕朝向（默认为竖屏portrait，横屏为landscape）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-np：不创建平台相关的项目文件。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-op：只创建项目文件。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-f：覆盖式的创建。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-c：根据配置文件加载。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-q：静默创建（不打印任何输出）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　-t：指定模板路径。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121328.jpeg)

图21-3　Quick-Player主界面

##### 4．手动创建

最后一种方式就是在一个C++项目中，手动添加对Quick的支持，一般来说，只需要**将Quick核心层的10个C++文件添加到程序员的项目中，并进行注册**，然后将Quick的脚本框架添加到项目中，并指定一个搜索路径即可。可以将一个空的Quick程序的src目录复制过来，然后在此基础上进行修改。如果需要使用到Quick扩展库的内容，就将quick_libs库添加到项目中，并调用其注册函数（用于将C++代码导出到Lua环境中）。

这种方式的缺点是略显麻烦，因为有较多的手动操作（相对于其他方式），并且**没有模拟器支持**，在做多分辨率调试的时候麻烦一些。但使用这种方式拥有较高的主动权，并且可以很方便地调试Cocos2d-x引擎，所以这种方式更加适合熟悉Cocos2d-x和Quick的程序员。

#### 21.3.2　第一个Quick程序

当创建了一个Quick项目时（这里以Quick-Player创建的项目为例），项目中包含了Lua脚本目录，在脚本目录中，通常会包含main.lua、config.lua以及一个名为app的目录。main.lua是项目的入口脚本，用于初始化并启动程序。config.lua是项目的配置脚本，用于配置程序的屏幕、分辨率等。

app目录下存放着该项目的脚本文件，包含了游戏的视图、场景、模型、控制层等脚本。下面是一个MyApp.lua脚本，以及一个views目录（也有可能是scenes目录，视创建的方式以及Quick版本而定），views目录下放着场景脚本MainScene.lua。这里的MyApp.lua相当于Cocos2d-x中所熟悉的AppDelegate，而MainScene.lua相当于HelloWorldScene。

打开MainScene.lua脚本，可以看到一个名为MainScene的类，其继承于Cocos2d-x的Scene对象。在构造函数ctor()中创建了一个TTFLabel对象，设置其文本内容为Hello, World，并添加到了MainScene中。在这里可以添加各种元素到MainScene场景中。

```
local MainScene = class("MainScene", function()
-- 使用display.newScene()方法来创建场景，这个方法会创建一个Scene，并开启其节点事
件监听
    return display.newScene("MainScene")
end)
-- 构造函数中使用ui.newTTFLabel()方法创建文本标签Hello，World
function MainScene:ctor()
    ui.newTTFLabel({text = "Hello, World", size = 64, align = ui.TEXT_
    ALIGN_CENTER})
        :pos(display.cx, display.cy)
        :addTo(self)
end
-- 场景切换进来时会回调
function MainScene:onEnter()
end
-- 场景切换出去时会回调
function MainScene:onExit()
end
return MainScene
```

在项目的目录中，可以找到项目在各个平台下的工程文件，如Windows下是Visual Studio工程文件、Mac下是Xcode工程文件，打开当前平台的工程文件，然后编译运行。程序运行的效果如图21-4所示（在Windows下运行，如果运行失败，应检查一下是不是忘记执行初始化脚本）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121329.jpeg)

图21-4　Hello World程序的运行效果

#### 21.3.3　开发工具

了解了如何创建Quick项目，以及创建后的Quick项目结构、脚本之后，还需要了解开发Quick程序所需要的工具。这里简单介绍3种开发工具，分别是Cocos IDE、Visual Studio + Babe Lua、Sublime Text + QuickXDev。

##### 1．Cocos IDE开发工具

Cocos IDE是Cocos2d-x官方提供的Lua开发环境，其界面如图21-5所示。这里虽然介绍该开发工具，但并不推荐使用。Cocos IDE提供了代码高亮和补齐，以及调试功能。但因为运行效率比较低，稳定性和兼容性都比较差，较容易出现卡死、崩溃等现象，而且官方对其的维护并不积极，所以不建议使用。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121330.jpeg)

图21-5　Cocos IDE界面

##### 2．Visual Studio + Babe Lua开发工具

在Windows下一般使用Visual Studio进行开发，Visual Studio有很多非常不错的插件，如Visual Assist、Visual SVN等。使用Babe Lua插件，可以在Visual Studio上很方便地开发Lua程序。Babe Lua是一款非常小巧的插件，提供了语法高亮、跳转到定义处、断点调试等功能。

安装Babe Lua之前先要确保计算机上安装了Visual Studio 2012及以上版本，然后下载并安装Babe Lua插件（地址为https://babelua.codeplex.com/releases）。安装完成之后可以在Visual Studio主界面的菜单上找到Lua菜单项。

安装完Babe Lua之后，还需要新建一个Lua项目，在主菜单上选择Lua-New Lua Project，会出现创建Lua项目的对话框，如图21-6所示，需要填写的5个参数分别如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Lua scripts folder：Lua的脚本目录，该目录下所有的脚本文件会被加入项目。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Lua exe path：可执行程序的路径，这里应该选择模拟器的路径。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Working path：工作路径，这个路径需要配置成项目路径，用于查找res资源目录。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Command line：启动程序时传入的参数，可以不填。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Lua project name：Lua项目的名字。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121331.jpeg)

图21-6　Visual Studio创建Lua项目时的对话框

假设当前的解决方案下已经有了一个C++项目，Lua部分只是该项目的一部分，那么还是需要创建一个Lua项目，可以将它们视为两个不同的项目。如果要调试运行程序，那么需要将Lua项目设置为启动项目，然后按快捷键F5启动调试。在Lua菜单下选择Run Without Debugging或按快捷键Ctrl+4可以直接启动（这种模式下不会触发断点）。

Babe Lua的代码高亮和跳转到定义处这两个功能用起来并不是很强大，高亮功能只高亮了少量代码，而跳转到定义处只能跳转到一些全局以及同一个文件中的方法和变量的定义处，无法跳转到类对象的成员方法的定义处，跟Visual Assist X插件相比还有很大的差距，但还算比较稳定。

另外，如果需要高亮Quick框架的源码，那么还需要下载Quick的自动补全词库（下载地址为http://pan.baidu.com/s/1sjmC169），下载后解压到“我的文档\BabeLua\Completion”目录下，并重启Visual Studio。除了下载词库之外，将Quick框架的脚本包含到项目中会更加方便，可以在项目上的右键快捷菜单中选择添加-Exiting Folder，来选择Quick脚本框架的目录。

http://www.cocoachina.com/bbs/read.php?tid=205043中对Babe Lua进行了非常详细的介绍，有兴趣的读者可以参考一下。

##### 3．Sublime Text + QuickXDev开发工具

Sublime Text是一个轻量级、跨平台的文本编辑器，该软件的文字和界面风格让人感觉非常舒服，并且有着丰富的插件，很适合用于编写Lua这样灵活的脚本。

选择“首选项”→“插件控制”→“安装插件”，在弹出的插件列表中输入quickX，可以找到QuickXDev插件，如图21-7所示，单击搜索结果会自动下载安装该插件。安装完成之后可以在“首选项”→“插件设置”中找到QuickXDev菜单项，接下来编写Quick代码时就会有自动的提示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121332.jpeg)

图21-7　在Sublime Text的插件列表中找到QuickXDev插件

QuickXDev还提供了跳转到定义处的功能，可以在要查看的标识符上右击，在弹出的菜单中选择Goto Definition，或按Ctrl+Shift+G快捷键。在使用这个功能之前需要先对插件进行设置。选择“首选项”→“插件设置”→QuickXDev-Settings-User，这时会打开一个新的文件，需要在文件中输入以下内容，将quick_cocos2dx_root对应的路径替换成Quick框架的根目录即可，注意，Windows下的路径需要将\替换成\\，当我们跳转到Quick源码时，QuickXDev会在我们填写的目录下寻找**quick\framework目录**。

```
{
   "quick_cocos2dx_root":"C:\\lua\\quick-cocos2d-x",
   "author":"baoye"
}
```

使用Sublime Text除了可以打开一个文件之外，还可以打开整个目录，在打开的目录上右击，弹出的菜单中会出现QuickXDev提供的菜单项，如图21-8所示，可以实现新建Lua脚本、创建项目、编译脚本、建立用户自定义标识等功能，建立用户自定义标识可以让程序员在调用自定义的函数和类时，也提供代码补全以及跳转到定义处的功能。从功能上来看，Visual Studio + BabeLua的组合更加强大一些，而从视觉效果来看Sublime Text+Quick XDev则看上去更加舒服一些，并且支持跨平台。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121333.jpeg)

图21-8　QuickXDev提供的菜单项

还有一些其他不错的选择，如LuaEditor、LuaStudio，这里就不多做介绍了，有兴趣的读者可以自行了解。

### 21.4　Quick运行流程分析

通过前面章节介绍，我们创建了一个Quick项目，并且使其成功地运行起来，接下来会详细地分析一下Quick程序的运行流程。

#### 21.4.1　初始化流程

Quick程序本身也是一个Cocos2d-x程序，但比普通的Cocos2d-x程序多了一些额外的内容，如界面上方的菜单项，Quick程序启动时会初始化这些菜单项并接收命令行配置，如分辨率、屏幕尺寸等，这些工作是平台相关的，也并非Quick程序的核心流程。

在完成Quick模拟器的初始化之后，会启动Cocos2d-x，所以我们可以以熟悉的AppDelegate为入口进行分析，启动Cocos2d-x时会调用到AppDelegate的applicationDidFinishLaunching()方法。不同版本的Quick在这里执行的逻辑有较大的差异。主要有两种不同的方式，一种是直接手动注册脚本引擎，并调用入口脚本，这是实际打包到手机上会使用的方式。另外一种是启动Lua的RuntimeEngine，RuntimeEngine是v3版本之后封装的一个单例，主要用于支持模拟器和Code-IDE。

但无论是哪一种方式，都会初始化脚本引擎，并将相应的C++方法导出到Lua中，最后调用Lua的入口脚本，一般这个入口脚本名为main.lua，会被放到项目的src或scripts目录下，也可以修改入口脚本。入口脚本中通过执行以下代码启动了Quick的脚本框架，就如C++代码中的启动时，在main()函数中调用AppDelegate的run()方法一样，但main.lua中的run()方法执行的逻辑则简单得多。

```
require("app.MyApp").new():run()
```

上面的代码会在脚本文件的搜索路径中查找app目录，并寻找app目录下名为MyApp.lua的脚本，然后执行该脚本，该脚本会返回一个类（一般是一个table），然后调用new()方法，创建出这个类的实例，并执行其run()方法。

app.MyApp是Quick项目自动生成的脚本，在require这个脚本的时候会被执行，定义一个名为MyApp的类并返回。同时该脚本require了config和framework.init脚本。config是项目的配置脚本，配置了一些如调试模式、屏幕尺寸、分辨率适配等变量。framework.init脚本则会根据相关的配置来初始化整个Quick脚本框架，稍后会对整个Quick脚本框架做一个详细的分析。

```
require("config")
require("framework.init")
```

在完成框架的初始化之后，才开始MyApp的定义，它继承于cc.mvc.AppBase，这是Quick框架中的MVC框架的应用基类。在构造函数中调用了父类的构造函数，并传入self。在run()方法中首先添加了一个资源的搜索路径，然后调用了self:enterScene()方法，进入到MainScene场景中。enterScene()是AppBase所定义的方法，其实现为调用Director的replaceScene()方法来进行场景切换。

```
-- MyApp继承于cc.mvc.AppBase
local MyApp = class("MyApp", cc.mvc.AppBase)
function MyApp:ctor()
    MyApp.super.ctor(self)
end
-- 启动App，进入MainScene场景
function MyApp:run()
    CCFileUtils:sharedFileUtils():addSearchPath("res/")
    self:enterScene("MainScene")
end
return MyApp
```

#### 21.4.2　MVC框架运行流程

MyApp继承于cc.mvc.AppBase，并且调用了其enterScene()方法进入了MainScene场景，那么cc.mvc.AppBase是什么时候初始化的？enterScene()方法背后又是如何执行的呢？

在framework.init中，会调用cc.init，也就是cc目录下的init.lua脚本，cc目录下存放着Quick框架扩展的基础类和组件。在cc.init中会初始化这些类和组件，其中就包括cc.mvc模块，cc.mvc模块的AppBase的定义和构造函数如下。

```
-- 定义了一个AppBase类，不继承于任何父类
local AppBase = class("AppBase")
AppBase.APP_ENTER_BACKGROUND_EVENT = "APP_ENTER_BACKGROUND_EVENT"
AppBase.APP_ENTER_FOREGROUND_EVENT = "APP_ENTER_FOREGROUND_EVENT"
function AppBase:ctor(appName, packageRoot)
    -- 添加了一个事件协议组件
    cc(self):addComponent("components.behavior.EventProtocol"):exportMethods()
    -- 设置应用程序的名字（标题）以及app目录，默认为app，也可以传入第二个参数设置自
    定义的目录
    self.name = appName
    self.packageRoot = packageRoot or "app"
    -- 这里添加了两个事件监听，程序从前台切换到后台会回调self.onEnterBackground()
    方法
    local eventDispatcher = cc.Director:getInstance():getEventDispatcher()
    local customListenerBg = cc.EventListenerCustom:create(AppBase.APP_
    ENTER_BACKGROUND_EVENT,
                                handler(self, self.onEnterBackground))
    eventDispatcher:addEventListenerWithFixedPriority(customListenerBg,1)
-- 程序从后台切回前台会回调self.onEnterForeground()方法
    local customListenerFg = cc.EventListenerCustom:create(AppBase.APP_
    ENTER_FOREGROUND_EVENT,
                                handler(self, self.onEnterForeground))
    eventDispatcher:addEventListenerWithFixedPriority(customListenerFg,1)
    self.snapshots_ = {}
    -- 设置全局变量app
    app = self
end
```

在构造函数中AppBase设置了self的name和packageRoot属性，添加了EventProtocol组件，以及前后台切换的事件回调，并设置全局变量app为self。

在Quick启动时，MyApp调用了enterScene()方法来切换场景，enterScene()方法会接收5个参数，也可以只传入1个参数。第1个参数是场景名字，AppBase会加载在self.packageRoot下的**scenes目录**中对应的脚本，self.packageRoot默认的值是app。第2个参数是一个table，作为初始化目标场景的参数。其余的3个参数会被传入到display.replaceScene()方法中，用于控制场景切换的效果。除了enterScene()方法外，AppBase还提供了类似的createView()方法，会加载self.packageRoot下的views目录中对应的界面脚本。

在display.replaceScene()方法和display.newScene()方法等方法中，为scene对象调用了setNodeEventEnabled()和setAutoCleanupEnabled()方法，这两个方法分别位于Cocos2d-x扩展模块的NodeEx.lua和SceneEx.lua中，setNodeEventEnabled()方法会让Scene的onEnter和onExit回调生效，而setAutoCleanupEnabled()方法则允许场景切换的时候自动清理一些纹理，通过调用场景的markAutoCleanupImage()方法，传入一个图片的名字，在场景退出时会自动清理这个图片对应的纹理或SpriteFrame。

```
function AppBase:enterScene(sceneName, args, transitionType, time, more)
    local scenePackageName = self.packageRoot .. ".scenes." .. sceneName
    local sceneClass = require(scenePackageName)
    local scene = sceneClass.new(unpack(checktable(args)))
    display.replaceScene(scene, transitionType, time, more)
end
```

Quick的MVC框架并没有定义View层和Control层的基类，也没有严格定义MVC框架的使用流程，所以这个MVC框架整体看上去比较空洞，只提供了AppBase以及Model这两个类。但在实际应用Quick时，我们还是可以很好地写出基于MVC模式的代码，而不必纠结于代码是否严格地遵循MVC模式来实现，我们要实现的是功能而不是设计模式，纠结于设计模式的话就是本末倒置了。

### 21.5　Quick脚本框架详解

本节将指引读者快速地掌握Quick脚本框架，前面几节中简单介绍了Quick框架的结构，环境的搭建和项目的创建，并剖析了Quick程序的基本运行流程，所以本节会对Quick的脚本源码做一个简单的介绍，以及阅读的指引。

Quick的源码写得很整洁，有着清晰的注释，可读性极强，阅读Quick的源码不但可以更加深入地了解Quick，对于初学者而言还可以从中学习到良好的编码风格。

Quick脚本框架主要实现了3个目标，一是通过脚本的封装提供了更加简洁易用的接口，二是封装了Quick框架的各种扩展功能，三是提供了基于MVC模型的框架。

#### 21.5.1　Quick脚本框架整体结构

首先需要认识一下Quick脚本框架的整体结构，Quick脚本框架是由Cocos2d-x原生Lua框架的Cocos脚本目录重构而来，所以两者会有部分重合的代码。进入到Quick脚本框架的framework目录，可以看到一堆的脚本文件以及4个目录，首先从这一堆脚本文件中挑选出值得一阅的脚本。脚本中的具体实现这里便不细说了，因为脚本中的注释之详细实属罕见。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　display.lua显示模块，提供创建各种显示对象、截屏的接口，以及显示相关的常量，比原生框架的display.lua提供了更多的功能。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　shortcodes.lua对Node扩展了很多简短的代码，如addTo、moveTo等方便的接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　functions.lua定义了大量常用的全局函数，如class()、handler()等方法，以及对I/O、table、math等标准库进行了扩展。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　device.lua设备模块，提供了各种与设备相关的接口，如弹出信息框、输入框、打开网页、获取系统语言、系统平台、设备唯一ID等。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　json.lua Json模块，提供了JSON格式的文件编码和解码接口，依赖于Quick扩展中的cjson库。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　audio.lua音效模块，提供了对SimpleAudioEngine的封装。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　crypto.lua加密解密模块，提供了MD5、Base64、AES256等算法的加密和解密功能，依赖于Quick扩展中的crypto库。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　network.lua网络模块，提供了网络状态检查、WiFi网络检查、HTTP请求等功能，依赖于Quick扩展中的network库。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　luaj.lua提供了在Android系统下与Java代码进行交互的功能。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　luaoc.lua提供了在iOS系统下与Objective-C代码进行交互的功能。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　transition.lua动画模块，对各种Action效果进行了简单的封装。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　scheduler.lua调度器模块，对Schedule进行了简单的封装。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　filter.lua依赖于Quick扩展中的filters库。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　ui.lua提供了创建Quick框架自定义UI的方法，但这些方法目前已经被废弃了。

上面是一些有意义的脚本文件的简介，其中**display.lua、shortcodes.lua、functions.lua** 这3个脚本在Quick框架中被使用得最频繁，因为它们提供了大量的方法能够使代码更加简短，从而提高编码效率。

接下来介绍一下在framework目录下的4个目录，cc目录为Quick框架的基础模块，cocos2dx目录为Cocos2d-x的扩展模块，该模块相当于对原生Lua框架的Cocos脚本框架的梳理，包含了大量常量的定义以及Node、Action、Sprite等类的扩展，deprecated目录下存放了一些废弃代码，platform目录下存放了一些平台相关的脚本。这里关键介绍cc目录和cocos2dx目录。

#### 21.5.2　Quick框架基础模块

framework的cc目录下是Quick框架基础模块，虽然Quick的代码风格不错，但当这份代码被N个人维护过之后，就不是那么回事了。cc目录的目录结构如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　components目录，定义了组件类，以及事件组件、状态机组件、布局组件、拖曳组件等功能组件。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　mvc目录，定义了Quick的mvc框架，但该目录下只有AppBase和ModelBase的实现。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　net目录，定义了一个TcpSocket类，用于TCP通信。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sdk目录，存放了接入平台sdk相关的脚本，该目录下只有一个内购脚本。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　ui目录，该目录下封装了大量Quick自定义的控件，这些控件不同于Cocos2d-x的UI框架。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　uiloader目录，定义了CocoStudio 2.x和1.6的加载器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　utils目录，定义了一些工具类和工具函数，如字节转换、计时器等功能，但并不常用。

其中components、mvc和ui这3个目录是基础模块中最常用的模块，这里的components不同于Cocos2d-x框架中的Component，这里是纯Lua的组件，而ui目录下的控件，也和Cocos2d-x的Widget没有任何关系，所以千万不要将它们混为一谈。后面会介绍相应的示例，在示例中可以了解到如何使用它们。

#### 21.5.3　Quick脚本框架初始化流程

最后来分析一下Quick脚本框架的初始化流程，可以将Quick脚本框架的初始化流程分为两部分来看，第一部分是整个框架的初始化，第二部分是Quick框架基础模块，也就是cc目录内部的初始化。在Quick程序启动时会执行MyApp.lua文件，在这里会调用require("framework.init")初始化Quick脚本框架。在该脚本中会依次require以下脚本，debug.lua、functions.lua、cocos2dx.lua、device.lua、transition.lua、display.lua、filter.lua、audio.lua、network.lua、crypto.lua、json.lua。其中cocos2dx.lua会require cocos2dx目录下的脚本。

根据当前平台自动require在platform目录下相应的脚本，如果是iOS系统，会require luaoc.lua脚本，如果是Android系统则会require luaj.lua脚本。

如果在config.lua中定义LOAD_DEPRECATED_API为true则会require ui.lua脚本，该变量默认值为false。如果定义LOAD_SHORTCODES_API为true则会require shortcodes.lua脚本，该变量默认为true。

在初始完框架大部分模块之后，init.lua会require cc目录下的init.lua脚本，来初始化Quick框架基础模块。首先cc.init会依次执行cc目录下的Registry.lua、GameObject.lua、EventProxy.lua脚本以及components目录下的Component.lua脚本。

Registry可以理解为一个单例类，其作用是将类和对应的名字进行缓存，可以通过指定的名字来创建对应类的对象，Registry还提供了如添加、删除、判断等管理功能。

GameObject也可以理解为一个单例，其只有一个extend()方法，extend()方法中对传入的对象进行了扩展，使其拥有了添加、删除、获取、查询组件的功能，并为其添加了一个组件table变量。

EventProxy是一个事件代理类，用于代理添加和删除监听回调，EventProxy的构造函数要求传入两个节点，分别是被监听节点以及监听节点，假设它们的名字分别为target和listener，listener可以监听target的事件，真正的监听、触发等核心功能的实现是在事件组件EventProtocol.lua中实现的。如果listener监听了target的事件，在listener被移除的时候没有移除该监听，当事件被触发时将会出现BUG，程序员很容易因为忘记移除而导致这样的BUG，所以EventProxy很好地解决了这个问题，它将所有listener对target的监听记录下来，然后当listener被移除时，会自动注销监听的回调。

在导入了这几个脚本之后，components目录下的所有组件都被导入并添加到Registry中，然后创建了一个元表，**该元表的__call字段被赋值为一个函数，该函数会将传入的第二个参数传入到GameObject.extend()方法中进行扩展。最后这个元表被设置给了cc这个table**。

```
local GameObject = cc.GameObject
local ccmt = {}
ccmt.__call = function(self, target)
    if target then
        return GameObject.extend(target)
    end
    printError("cc() - invalid target")
end
setmetatable(cc, ccmt)
```

如果读者在前面阅读代码的过程中，发现有像cc(xxx)这样写的代码，肯定会疑惑，cc不是一个table吗？怎么还可以这样？因为这里设置了元表，当调用cc(xxx)时，cc会作为第一个参数传入元表的__call字段对应的函数中，而xxx会作为第二个参数被传入，然后__call字段对应的函数通过调用GameObject的extend()方法，为参数xxx扩展组件相关的方法（这个技巧秀得很漂亮，让人眼花缭乱，但如果需要调用cc()来扩展一个对象时，直接调用GameObject.extend()方法更加直观）。

最后cc.init会调用mvc.init()方法、ui.init()方法以及uiloader.ini()方法初始化mvc、ui以及uiloader模块，完成Quick框架基础模块的初始化。关于组件、MVC模块如何使用，会在第22章节中详细介绍。