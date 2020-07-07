# 第2章　使用Cocos2d-x

本章开始正式使用Cocos2d-x，这一章以HelloCpp为入口，从整体上一窥Cocos2d-x的架构，建立起Cocos2d-x的世界观，进入到Cocos2d-x的世界中，以解决面对Cocos2d-x不知如何下手的问题。在使用一种新技术之前，先从整体上简单了解这种技术，会让你少走一些弯路。

按照传统的做法，例如，直接使用GDI、OpenGL、DirectX，或者HGE这种轻量级的游戏引擎，无非就是这么几步：开始初始化创建一堆东西，然后一个主循环，在主循环里不停地调用update和draw来进行游戏逻辑的更新和渲染，直到游戏的退出条件成立，将游戏中创建的对象销毁，一个游戏的流程大致就是这样。我们需要把所有要显示的东西保存到容器中，在update中根据逻辑来操作它们，在draw中一个一个地渲染它们。

Cocos2d-x的底层基本也是这样的流程，但在使用的时候，是在这一层之上，Cocos2d-x的封装大大简化了所需要编写的代码，一般而言，我们不需要关注显示对象的draw部分，只需要关注逻辑即可，并且所有的显示对象会被场景树组织好，管理起来，不需要我们手动管理。本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/ed1dfab6f5b7e4d0fe42db1a32b09210)　Cocos2d-x世界。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/ed1dfab6f5b7e4d0fe42db1a32b09210)　分析HelloCpp。

## 2.1　Cocos2d-x世界

使用Cocos2d-x来开发游戏，要先了解几个问题，如图片如何显示，逻辑如何执行，如何获取输入，如图2-1所示简单描述了Cocos2d-x程序的结构，系统输入、场景树，以及Schedule驱动的场景更新，接下来更深入地了解一下相关的细节。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/88de91392a269aa8c07da6ecfbc80fe0)

图2-1　Cocos2d-x程序结构

### 2.1.1　显示图片

Cocos2d-x由许多的场景组成，运行时有且只有一个场景。场景是一种节点，Cocos2d-x有各种各样的节点可以挂载在场景上，例如Layer、Sprite、Label、MenuItem等，每种节点都有自己的能力，一个游戏场景由场景节点以及各种各样的节点组成，形成一颗场景树。Cocos2d-x会按照节点树的规则将整个场景的内容依次渲染出来。因此在Cocos2d-x中，需要显示一些东西在屏幕上，只需要创建一些显示节点，然后添加到场景树中即可。

每种节点都有自己的功能，Sprite是最常用的精灵节点，可以显示一张图片。Menu和MenuItem也是很常用的节点，Menu是一个菜单，并没有显示功能，用来接收单击消息，并传递给MenuItem，MenuItem是菜单中的菜单项，也拥有显示图片的能力，但主要作为按钮使用。Layer也属于一种容器节点，类似Menu，并没有显示功能。

**节点可以直接使用，也可以继承扩展**，在实际应用中，大量代码是编写在自己继承的自定义节点类型中，在节点中进行编码是非常轻松的一件事，这是一种推荐做法。关于节点，会在第6章中详细介绍。

### 2.1.2　执行逻辑

一般，Cocos2d-x的逻辑都是写在各种继承的Node中，常用的有3种编写逻辑的方法。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/ed1dfab6f5b7e4d0fe42db1a32b09210)　第1种是在节点自动回调方法内写逻辑，init节点被创建初始化时调用，onEnter节点被添加到场景时调用，onExit节点从场景中删除时调用，这3个常用的方法，是Cocos2d-x会自动回调的虚函数，只需要把逻辑写在这里，Cocos2d-x会在节点被进行处理时相应地回调它们。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/ed1dfab6f5b7e4d0fe42db1a32b09210)　第2种是使用调度器来定时执行逻辑，每个类都有一个update回调函数，只要调用scheduleUpdate就会自动注册每帧执行该类的update回调函数。在update回调函数中可以编写每帧都会被执行的代码，例如，判断玩家HP是否为0，或者移动等。调度器也支持自定义时间间隔、延迟和重复次数的定时回调。调度器所需要执行的回调，需要手动注册到调度器中才会被执行。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/ed1dfab6f5b7e4d0fe42db1a32b09210)　第3种是指定事件发生之后的回调，其主要是各种UI按钮的单击回调，也是需要手动指定UI相关的回调函数。与调度器不同的是，Cocos2d-x的GUI并没有一个统一的接口来规范，本章会简单介绍一下Menu和MenuItem，更多的输入交互将放到后面再介绍。

### 2.1.3　获取输入

输入是一个很大的概念，这里简化了说，从表面上看，将菜单项MenuItem添加到菜单Menu中，菜单项即可接受单击消息并执行单击回调，实际上所有的输入都被Cocos2d-x至少封装了两层：

第一层是平台相关的输入接口，封装在不同平台下，系统输入事件如何触发，如何回调，这一层在不同平台中有不同的代码来适应。

第二层是Cocos2d-x消息处理层，将不同平台的输入消息转换为Cocos2d-x内部的消息，在3.x之前，Cocos2d-x使用TouchDispatcher或其他的Dispatcher将事件转发到指定的Delegate中，Menu作为一个Delegate来接收单击消息，并调用被单击到的MenuItem的单击回调，3.x使用EventDispatcher将事件封装为一个Event，转发到监听该Event的EventListener中，Menu拥有单击输入相关的EventListener，绑定了自身的回调函数，在单击事件触发时，在回调函数中检测并执行被单击到的MenuItem的单击回调。

## 2.2　分析HelloCpp

HelloCpp是Cocos2d-x的一个入门例子，在第1章中已经将其运行起来了，下面先介绍一下HelloCpp目录结构，这也是所有Cocos2d-x程序的一个基本结构，如图2-2和图2-3所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/e8acb768ad2419b263058762415831c8)

图2-2　2.x目录

![img](https://gitee.com/nlpleaf/PicGo/raw/master/27779672bef94280060ee480b0bb674a)

图2-3　3.x目录

main.cpp和main.h是win32下的程序入口，不同平台有不同的入口，这部分代码只负责在当前平台下启动Cocos2d-x。想要详细了解win32下是如何启动的，只需要阅读**《Windows程序设计》**的第3章**窗口与消息**，照着写一个简单的窗口Application即可。基于Android和iOS，在最纯粹的环境下，写一个HelloWrold类型的程序运行起来，不是一件很难的事情，但能让程序员建立对这个平台的一个整体认识，帮助解决在这个平台下碰到的一些问题，例如，有些问题不清楚是Cocos2d-x的问题，还是系统的问题，那么**把代码简化，把问题简化，再来解决**，则可以免除很多干扰（也就是使用排除法）。

接下来是AppDelegate和HelloWorldScene，AppDelegate是Cocos2d-x内部的入口，从操作系统到Cocos2d-x，会先进入到AppDelegate。在引擎初始化完成之后，会调用AppDelegate的applicationDidFinishLaunching方法，在AppDelegate中执行游戏的初始化，设置分辨率并启动场景，如图2-4所示。Cocos2d-x在几个主要平台上的详细运行流程，会在第14章中详细介绍，本章只介绍一个精简流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/ab8a314ff038b002c6930b59923267df)

图2-4　启动流程

```
bool AppDelegate::applicationDidFinishLaunching()
{
    //初始化director
    auto director = Director::getInstance();
    auto glview = director->getOpenGLView();
    //初始化OpenGL视图
    if(!glview)
    {
        glview = GLViewImpl::create("Cpp Empty Test");
        director->setOpenGLView(glview);
    }
    //设置OpenGL视图到director中
   director->setOpenGLView(glview);
   //设置屏幕分辨率
  glview->setDesignResolutionSize(designResolutionSize.width,
  designResolutionSize.height,   ResolutionPolicy::NO_BORDER);
    //开启显示左下角的FPS状态信息
    director->setDisplayStats(true);
    //设置FPS帧率，默认的帧率就是1.0 / 60，也就是每秒执行60帧
    director->setAnimationInterval(1.0 / 60);
    //创建HelloWorld场景，这个对象会交由Cocos2d-x管理，不需要手动释放
    auto scene = HelloWorld::scene();
    //运行HelloWorld场景
    director->runWithScene(scene);
    return true;
}
```

AppDelegate调用HelloWorld::scene()创建了一个HelloWorld场景然后启动HelloWorld，在HelloWorld的init中，构建了整个场景所需要的内容。构建场景树的过程就是create节点，然后调用场景中节点的addChild方法，将创建的节点添加到节点树中。在HelloWorld场景中构建了一个菜单、菜单项、文本和图片。

```
//在init中构建节点
bool HelloWorld::init()
{
    //1. 先执行父类的init
    if ( !Layer::init() )
    {
        return false;
    }
    auto visibleSize = Director::getInstance()->getVisibleSize();
    auto origin = Director::getInstance()->getVisibleOrigin();
    //2. 关闭按钮是一个MenuItemImage，表示一个菜单项，非选中状态和选中状态分别是
    CloseNormal.png和CloseSelected.png，当它被单击的时候，会调用HelloWorld的
    menuCloseCallBack函数
    auto closeItem = MenuItemImage::create(
                                        "CloseNormal.png",
                                        "CloseSelected.png",
                                        CC_CALLBACK_1(HelloWorld::menuClose
                                        Callback, this));
    //设置关闭按钮的位置，其在窗口的右下角
    closeItem->setPosition(origin + Vec2(visibleSize) - Vec2(closeItem->
     getContentSize() / 2));
    //创建菜单，上面创建的菜单项需要添加到菜单中才有效
    auto menu = Menu::create(closeItem, NULL);
    menu->setPosition(Vec2::ZERO);
    this->addChild(menu, 1);
    //创建HelloWorld的文本，字体是Arial，字号是TITLE_FONT_SIZE.
    auto label = LabelTTF::create("Hello World", "Arial", TITLE_FONT_SIZE);
    //将文本放置到中间偏上的位置，并添加到场景中
    label->setPosition(origin.x + visibleSize.width/2,
                            origin.y + visibleSize.height - label->
                            getContentSize().height);
    this->addChild(label, 1);
    //添加背景到场景中并居中
    auto sprite = Sprite::create("HelloWorld.png");
    sprite->setPosition(Vec2(visibleSize / 2) + origin);
    this->addChild(sprite);
    return true;
}
```

在创建MenuItemImage时，指定了回调函数menuCloseCallback，它是HelloWorld的成员函数，函数原型如下所示，传入一个Ref*，单击事件的发送者也就是我们所单击的MenuItemImage。在回调函数中，执行Director的end方法让游戏结束。

注册回调时，Cocos2d-x 2.x使用的是menu_selector(HelloWorld::menuCloseCallback)，而Cocos2d-x 3.x则是CC_CALLBACK_1(HelloWorld::menuCloseCallback,this)，具体的规则会在第5章中讲述。

```
void HelloWorld::menuCloseCallback(Ref* sender)
{
    Director::getInstance()->end();
}
```

图2-4和前面的代码介绍了显示图片和输入回调等内容，都是一些静态的内容。接下来调整一下HelloWrold的代码，做一些有趣的事情，用我们的代码来控制场景的内容，让它们“动”起来。

下面实现这样的功能：当每次单击按钮的时候，按钮的位置往左边偏移一些，而在每次update中，文本的位置往下偏移一点。要实现这两个功能，可以在HelloWorld场景类中定义两个成员指针，来指向文本对象和按钮对象，然后在update和按钮回调中编写相关操作。在这里不定义新的变量，可以用Tag来获取它们。Tag查询以及名字查询，可以减少一些成员变量的声明，但在操作非常频繁的时候，还是有必要添加一个成员变量来存放节点指针。总之，**怎样简洁怎样做**。

我们需要在HelloWorld::init的代码中，将这两行代码添加到return true的前面，然后修改menuCloseCallBack中的代码，并添加一个update函数，（在头文件添加一行void update(float dt)）。这两行代码将label的标签设置为123，方便以后查找节点。另外调用了**scheduleUpdate**函数，让Cocos2d-x每一帧都执行update方法。

```
label->setTag(123);
this->scheduleUpdate();
```

将menuCloseCallback回调函数的代码修改如下，完成每次单击按钮都往左边偏移一点的功能。

```
//pSender是发送者本身，在这里这个消息的发送者就是按钮，而我们要设置其位置
//需要将其转化成CCNode指针，这里用dynamic_cast这种写法
//这种是C++标准写法，用于有继承关系的指针转化，比直接(CCNode*)转化更加安全
//因为dynamic_cast中间会进行类型检查，当pSender不能转化为CCNode*的时候，会返回
一个空指针
//在判断的时候，养成常量在左的习惯，会减少很多不必要的麻烦
void HelloWorld::menuCloseCallback(Ref* pSender)
{
    Node* node = dynamic_cast<Node*>(pSender);
    if (NULL != node)
    {
        node->setPositionX(node->getPositionX() - 10.0f);
    }
}
```

在HelloWrold.cpp中添加如下代码，先获取子节点，如果获取到，则设置其位置，注意这里用到了一个50.0f * dt，dt作为一个参数传入，表示上一帧逝去的时间，1.0表示一秒钟，帮助我们控制文本平滑地运动。50.0f * dt表示每秒50单位的速度，100.0f * dt则表示每秒100单位的速度，这个单位可以简单理解为像素，但其并不等于实际的像素，会受到分辨率的影响。

```
//这里直接通过getChildByTag()函数获取文本对象，并且设置其偏移位置
void HelloWorld::update(float dt)
{
    Node* node = this->getChildByTag(123);
    if (NULL != node)
    {
        node->setPositionY(node->getPositionY() - 50.0f * dt);
    }
}
```

运行代码，可以看到HelloWorld慢慢落下来，每次单击按钮的时候，按钮都往左边移动一点，Cocos2d-x提供了很多方便操作场景的方法，这一章先上一碟开胃小菜。

## 2.3　小结

通过本章的介绍，读者可以对Cocos2d-x的使用有一个大致的了解，知道Cocos2d-x最基础的游戏规则——代码大概应该怎样写。

本章还对Cocos2d-x的场景、节点、运行流程简单地介绍了一下，将Cocos2d-x的游戏世界简单地呈现出来，希望能帮助读者理解它！

最后，忘掉键盘与鼠标，Cocos2d-x是一个用于开发移动平台的游戏引擎，其很多功能，都是基于移动平台的，如想开发PC端的游戏，则有很多更好的选择，如OGRE和Irrlich。总之，干活之前，先得挑件趁手的兵器。