# 第7章　调试Cocos2d-x

本章要介绍的并不是调试代码，而是对Cocos2d-x游戏内容的调试，能够在运行时观察和控制当前的场景结构和节点详情，可以提高调试效率。例如，不需要反复地修改代码、编译、重启来观察某个节点的位置设置是否合理。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　控制台调试。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用KxDebuger调试Cocos2d-x。

### 7.1　控制台调试

Cocos2d-x提供了控制台的方式可以调试Cocos2d-x的游戏内容，通过控制台可以在程序运行的时候暂停和恢复，查看当前场景下的节点详情，TextureCache中缓存的纹理，控制fps的开关等，还可以实现一些自定义的命令。

不论游戏是在PC、手机还是Pad上运行，都可以使用控制台进行调试。例如，我们希望知道某个节点是否成功地被添加到场景中，无须调整代码打印日志，可以直接在控制台输入scenegraph命令，即可查看当前场景下的节点详情。

##### 1．开启Console监听

要使用Console进行调试，只需要两个简单的步骤，首先是Console的开启，只需要在AppDelegate中添加以下两行代码即可，通过调用Console的listenOnTCP()方法，可以在指定的端口进行监听。

```
auto console = director->getConsole();
console->listenOnTCP(5678);
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120112.jpeg)**注意：**如果使用的端口已经被其他程序占用，则会绑定失败。

##### 2．连接Console

开启了Console的监听之后可以在命令行中使用Telent连接Cocos2d-x程序进行调试。如果是在本地调试，可以直接连接localhost或127.0.0.1；如果是在手机上，需要保证PC和手机之间的网络能够正常连接（如在同一个局域网下）。

如果是在Windows 7系统下，默认是没有开启Telent程序的，可以通过下面这几个简单的步骤来开启Telnet程序。选择“控制面板”→“程序”→“打开或关闭Windows功能”选项，在弹出的对话框中选择“Telnet客户端”，单击“确定”按钮，如图7-1所示。然后就可以在命令行中输入Telnet命令了。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120113.jpeg)

图7-1　开启Telnet

在Mac和Linux下可以直接使用Telnet命令来连接Cocos2d-x程序，如图7-2所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120114.jpeg)

图7-2　telnet localhost命令

##### 3．执行指令

在连接上Cocos2d-x程序之后，可以输入各种命令来进行调试，输入的方式是命令名+空格+参数，不同的命令所需的参数不同，输入help可以列出所有的命令以及其相关的命令说明。

##### 4．内置指令

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　allocator指令可以打印内存分配的诊断信息，需要ccConfig.h中将CC_ENABLE_ALLOCATOR_DIAGNOSTICS宏置为1，然后重新编译，才可以使用这个命令。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　config指令可以打印出程序的配置信息，例如，适配策略，设计分辨率的宽和高，是否显示FPS等。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　debugmsg指令可以查看当前是否接收调试信息，当附带参数on时可以开启，附带参数off时可以关闭，当开启接收调试信息时，Cocos2d-x程序中打印的日志都会发送到控制台上。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　director指令可以控制游戏的暂停、恢复以及结束，可以附带以下参数：pause（暂停）、resume（恢复）、end（结束）、stop（停止）、start（开始）。-h参数或help参数可以查看帮助。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　exit指令用于退出控制台。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　fileutils指令可以查看当前FileUtils中缓存的所有的绝对路径，flush参数可以清空缓存。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　fps指令加上on和off参数可以控制fps的显示。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　help指令可以查看帮助。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　projection指令可以查看当前的投影方式是3D还是2D，加上2D或3D参数可以改变投影方式。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　resolution指令可以查看当前的分辨率以及适配策略，加上宽度、高度、分辨率适配策略这3个参数，可以修改当前的设计分辨率以及适配策略。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　scenegraph指令可以查看当前场景的详细内容，包括所有节点的类型、Tag、父子关系，特殊内容等（如Label的文本内容，Sprite的纹理ID等），如图7-3所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120115.jpeg)

图7-3　查看场景

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　texture指令可以查看当前TextureCache中缓存的纹理详情。加上flush参数可以清空TextureCache中的缓存。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　touch指令可以模拟触摸，加上tap、x坐标、y坐标3个参数可以模拟在指定的坐标的点击，加上swipe、x1、y1、x2、y2可以模拟从x1、y1坐标拖动到x2、y2坐标。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　upload指令加上文件名和经过Base64编码的文件内容，可以将文件上传到Cocos2d-x所在的设备上。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　version指令可以查看当前Cocos2d-x的版本。

##### 5．自定义指令

Cocos2d-x内置的指令很难满足丰富的调试需求，所以Cocos2d-x提供了一种便捷的方式可以添加自定义指令来进行扩展，cpp-tests中的Console Test示例演示了如何添加自定义的指令，代码如下。

```
_console = Director::getInstance()->getConsole();
static struct Console::Command commands[] = {
   {"hello", "This is just a user generated command", [](int fd, const
   std::string& args) {
      const char msg[] = "how are you?\nArguments passed: ";
      send(fd, msg, sizeof(msg),0);
      send(fd, args.c_str(), args.length(),0);
      send(fd, "\n",1,0);
   }},
};
_console->addCommand(commands[0]);
```

首先需要获取Console对象，然后构造一个Console::Command对象，再调用Console的addCommand()方法注册添加这个对象，这样就可以在命令行中使用这个命令了。

Command对象由3部分组成，即命令的名字、命令的帮助提示（输入help指令时罗列的帮助信息）、命令的处理回调，该回调会传入一个fd，以及命令的参数字符串，可以调用send()方法将命令执行后的结果发送到控制台中。

如果希望延迟发送，或者在某个时机触发时才发送相关信息给控制台，我们可以使用lambda或者用变量将fd保存起来，然后在合适的时机再调用send()方法发送信息。也可以开启debugmsg，然后通过调用Console对象的log()方法来发送信息给控制台。

### 7.2　使用KxDebuger调试Cocos2d-x

虽然使用Console可以简单地调试Cocos2d-x的内容，但效率较低，而且步骤比较烦琐。如果能像Unity那样提供运行时的可视化调试方案，那么可以大大提高调试效率。

Cocos2d-x官方新出的Creator也类似Unity，可以对游戏内容进行调试，但并不支持调试C++开发的Cocos2d-x程序，仅支持JavaScript和Lua。因此笔者设计了一套简易的可视化调试工具KxDebuger用于调试Cocos2d-x，KxDebuger不仅可以调试PC上的程序，还可以远程调试移动设备上的程序，由于时间原因，目前的KxDebuger还不够完善，但以后笔者会花一些时间来进行维护，使其成为一个顺手的调试利器。

##### 1．使用KxDebuger

kxDebuger分为两部分，第一部分是嵌入Cocos2d-x程序的库，第二部分是GUI界面工具。KxDebuger库依赖于ProtocolBuffer和kxServer，前者是Google开发的一个协议库，后者是笔者开发的一个简易的网络库，可以直接将这两个库的代码包含到项目中，具体可以参考KxDebuger示例项目，读者可以在下载地址中找到它。添加好KxDebuger库之后只需要执行一行初始化代码即可使用KxDebuger库的客户端。

```
kxdebuger::KxDebuger::getInstance()->init();
```

在代码中初始化KxDebuger库之后，编译程序并启动Cocos2d-x程序，接下来就可以启动KxDebuger的GUI界面工具了，如图7-4所示。首先需要选择IP和端口，默认的端口是6666，可以在KxDebuger::init中设置指定的端口，如果是本机调试，可以选择127.0.0.1，如果需要在其他计算机或移动设备上调试，需要修改对应设备的IP地址。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120116.jpeg)

图7-4　KxDebuger启动界面

连接成功之后，GUI解密工具会切换到调试界面，如图7-5所示，我们可以看到左侧的场景树和右侧的节点属性面板，在属性面板中可以查看和修改节点的各种属性。

##### 2．KxDebuger功能简介

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调试节点树：通过右侧的树控件可以实时观察场景树，并执行刷新和删除、查看节点等操作。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调试节点：当选中节点之后，可以在右侧的属性面板中查看并修改节点的各种属性，也可以激活高亮该节点。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　单步调试：通过“调试”菜单下的快捷键可以暂停、恢复游戏，也可以逐帧调试游戏，这在捕获一些瞬间出现的动画问题时非常有用。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　自定义调试：是KxDebuger的高级功能，通过修改GUI界面工具，以及在KxDebuger中注册新的服务，可以调试自定义的内容，如对游戏的AI和特定的逻辑进行调试。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120117.jpeg)

图7-5　KxDebuger调试界面