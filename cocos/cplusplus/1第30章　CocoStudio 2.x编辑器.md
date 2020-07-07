# 第30章　CocoStudio 2.x编辑器

CocoStudio从1.6到2.x的升级，对整体界面进行了重构，2.x只保留了UI编辑器的功能，去掉了骨骼动画编辑器、场景编辑器以及数据编辑器，并且将CocoStudio整合进了Cocos引擎中，Cocos引擎会为其创建的每一个项目生成各个平台的项目文件及通用的资源目录，并生成对应的CocoStudio项目。本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　使用Cocos引擎。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　使用CocoStudio 2.x。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　动画功能。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　分辨率适配。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　高级特性。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　其他特性。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　在Windows下调试引擎。

## 30.1　使用Cocos引擎

当前最新版本的Cocos引擎为2.2.8，下载地址为http://www.cocos.com/download/。

在第26章中简单介绍了Cocos引擎与Cocos2d-x之间的版本对应关系，可以根据Cocos2d-x版本来选择Cocos引擎的版本。

重构后的CocoStudio 2.x现在可Mac和Windows两大平台同步更新。另外，CocoStudio 2.x版本可以导入CocoStudio 1.6版本的项目文件，但是会存在一些丢失的情况。

我们需要使用Cocos引擎来创建项目，Cocos引擎的启动界面左边包含了若干选项，如项目、教程、商店、下载、反馈等。

如果需要创建一个完整项目，需要先到商店界面下载Framework，也就是Cocos2d-x引擎的内核，然后就可以创建项目了。单击项目面板的“新建项目”按钮，在弹出的窗口中选择Cocos项目，如图30-1所示，填写一些必要信息之后就可以完成项目的创建了。需要注意的是**使用Cocos引擎创建的项目有一个弊端，就是默认无法调试到引擎源码**，因为是直接使用编译好的lib、dll及头文件，没有源文件。另外一个需要注意的问题就是**在Windows下无法打包iOS程序包，必须在Mac下才可以。**

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160805.jpeg)

图30-1　使用Cocos引擎创建项目

在创建项目时也可以选择示例，创建一些CocoStudio的官方例子来学习CocoStudio的用法。这里有登录、关卡选择、背包、菜单、主场景等多个示例，如图30-2所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160806.jpeg)

图30-2　创建Cocos示例项目

## 30.2　使用CocoStudio 2.x

### 30.2.1　CocoStudio 2.x界面介绍

在使用CocoStudio 2.x之前先来简单了解一下CocoStudio的界面结构，如图30-3所示，主要包含菜单栏、工具栏、“控件”面板、“画布”面板、“属性”面板、“资源”面板、“对象”面板和“动画”面板等。下面简单了解一下各面板的功能。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160807.jpeg)

图30-3　CocoStudio 2.x主界面

菜单栏包含“文件”（项目和文件的创建、保存、关闭及打开，还有资源的导入及退出CocoStudio）、“编辑”（撤销与偏好设置）、“项目”（运行、发布与打包以及相关设置）、窗口（各个面板窗口的开启和关闭）、“语言”（切换中英文）和“帮助”等菜单。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　菜单栏下的工具栏如图30-4所示，其功能很多，包含快速新建文件，分辨率与横竖屏设置，运行、发布与打包的快捷键，以及各种对齐和排列方式（选择多个节点可以激活对齐与排列的小图标），单击工具栏最右边的小手按钮和箭头按钮，可以切换编辑画布时的拖曳模式和选取模式，按住键盘上的空格键可以快速切换到拖曳模式。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160808.jpeg)

图30-4　工具栏

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　“画布”面板是CocoStudio中最常用的面板，在该面板中可以编辑对象的位置、旋转、缩放及锚点，在“画布”面板上可以同时编辑多个画布，画布上显示的内容都可以在游戏中显示。在2.x中，“对象”面板罗列了各种对象，包含基础对象（TiledMap、Node、粒子、声音和Sprite）、控件（按钮、文本、滚动条和复选框等）、容器（列表、ScrollView和PageView等）、自定义控件（目前只有Armature）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　“动画”面板可以用来编辑对象的动画、节点树的结构以及查看播放动画。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　“资源”面板可以用来管理项目中所有的**资源文件**，包含图片、骨骼、图集、声音、粒子、CocoStudio文件等。可以实现资源的导入和删除等。需要注意的是，资源面板中的目录结构与磁盘中的目录结构是对应的，如果在磁盘中删除了一个图片，已经引用这个资源的控件会显示资源丢失样式，同时资源面板中的图片文件会被标记为红色。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　“属性”面板是当选中画布中的一个对象时，属性面板会出现该对象对应的属性，此时在属性面板中可以编辑属性，包含基础属性（位置、尺寸、资源、九宫和颜色等）与高级属性（回调函数、类、帧事件和用户数据等）。

### 30.2.2　使用CocoStudio 2.x

接下来了解一下CocoStudio 2.x的使用流程，首先进入CocoStudio项目中（30.1节中创建的项目）。然后选择新建文件，可以选择新建场景、图层、节点、PLIST合图以及3D场景，如图30-5所示。这里选择场景文件，然后填写文件的名称，单击“新建”按钮之后，在资源面板中会生成对应的csd文件（CocoStudio Design），csd文件可以调整目录（文件的默认路径为创建时，资源面板中的当前目录），多个csd文件之间可以相互嵌套。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160809.jpeg)

图30-5　“新建文件”对话框

那么这里新建的几种文件有什么区别呢？

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　场景文件用于编辑场景，一般需要嵌套其他csd文件，场景的大小视项目分辨率而定，根节点为Node。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　图层文件与场景文件类似，但图层有固定的大小并且根节点为LayerRGBA。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　节点文件用于编辑一个节点，可以嵌套其他csd文件，根节点为Node。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　合图文件等同于TexturePacker导出的Plist图集，文件的后缀为csi。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160804.jpeg)　3D场景文件用于编辑3D场景，可以添加相机、节点、3D例子和模型等。

新建完文件之后，可以在左侧的“对象”面板中拖曳各种对象到画布中，从而添加各种对象到画布中。例如，添加一个图片控件，在添加之后还需要设置控件的显示图片，这时需要在“资源”面板中导入图片资源文件（在“资源”面板中利用右键菜单导入，或者在文件菜单中导入，也可以直接添加到对应的磁盘目录中，然后刷新“资源”面板）。

导入资源之后，可以在“资源”面板中拖曳图片资源至图片控件的“属性”面板的图片资源属性中，此时图片控件将更新显示的图片。

调整好画布的内容之后，可以选择发布资源，选择项目菜单下的“发布与打包”命令，在弹出的对话框中进行发布，也可以单击工具栏中的“发布”按钮来发布。

此外，也可以选择在模拟器中运行查看当前编辑的内容。默认会发布到CocoStudio项目上层目录的res目录下，用到的.csd文件都会输出.csb文件（CocoStudio Binary），也可以选择输出.json文件，而.csi文件会输出成Plist文件。

### 30.2.3　在程序中加载.csb文件

res目录也是程序项目的资源目录，可以在程序的初始化场景中，加载CocoStudio发布的.csb文件。在CocoStudio目录的上层可以找到程序的项目文件，上层的目录结构与创建项目时所使用的语言相关。不同的语言会生成不同的目录结构。这里以C++项目为例，在对应平台的proj目录下打开相应的项目文件。

在HelloWroldScene.cpp中，可以在HelloWrold的init方法中，使用CSLoader的静态方法createNode来加载.csb文件，将返回的节点添加到场景中即可显示。

```
//包含指定的头文件
#include "cocostudio/CocoStudio.h"
#include "ui/CocosGUI.h"
//引用命名空间
using namespace cocostudio::timeline;
bool HelloWorld::init()
{
  //初始化父类
  if ( !Layer::init() )
  {
    return false;
  }
  //使用CSLoader的静态方法createNode来加载.csb文件
  auto rootNode = CSLoader::createNode("MainScene.csb");
  addChild(rootNode);
  return true;
}
```

## 30.3　动画功能

### 30.3.1　在CocoStudio中编辑动画

在CocoStudio 2.x中，可以编辑简单的骨骼动画和序列帧动画。CocoStudio 1.6的动画编辑器拥有强大的动画编辑功能，但CocoStudio 2.x中并没有实现这些功能，只是提供了一些折中的方法来兼容1.6的动画编辑器。下面详细介绍CocoStudio 2.x的动画编辑功能。

当要编辑动画的时候，需要在“动画”面板中选定对象列表中的对象，然后在时间轴中插入帧。插入两个帧之后，在两个帧中调整对象的位置、缩放、倾斜等属性，两个帧之间会自动填充补间动画，如图30-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160810.jpeg)

图30-6　动画编辑器

目前只能调整位置、缩放与倾斜这3种属性，其他属性调整无效，后续可能可以调整更多的属性。拖动时间轴中的当前轴，可以查看动画。

通过左上角的“动画播放”按钮，可以在播放动画、循环播放、多个动画间进行切换，以及逐帧查看动画。

对象列表可以选定当前要编辑的对象，也可以从画布中选择，然后对这个对象进行动画编辑。在CocoStudio中无法设置对象的ZOrder，但是可以在对象列表中，通过拖曳来调整对象显示的先后顺序，也可以在这里调整父子节点关系。

当需要编辑多个动画的时候，就需要用到动画列表，如图30-7所示。在CocoStudio2.x中，**所有的动画都被放在一条时间轴上**。也就是说，如果有攻击、移动、死亡这3个动画，只要播放，则3个动画会连续播放。在最初的CocoStudio2.0中甚至都没有动画列表的概念，如果需要单独播放攻击动画，需要记住攻击动画是从第几帧到第几帧，然后在代码中播放从第几帧到第几帧的动画。这样比较麻烦，所以后面的版本升级并引入了动画列表的概念。在动画列表中简单记录每个动画的名字以及其起始帧和结束帧，单击“动画播放”按钮右边的“管理动画列表”按钮，可以打开“管理动画列表”对话框，在这里可以添加和删除动画，并设置动画的起始和结束帧。这些工作与动画编辑无关，只是另外手动记录一份列表，也就是说，你记录的动画列表有可能与实际播放的动画根本不是一回事。添加了动画列表之后，在播放按钮下会出现对应的下拉列表框，通过选择不同的下拉列表框，可以切换要播放的动画。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160811.jpeg)

图30-7　管理动画列表

自动记录帧是一个比较有意思的功能，选中“自动记录帧”复选框，然后选择当前时间轴，再对画布中的对象进行操作，再选择新的时间轴，然后再操作，如此反复，可以快速地连续编辑动画。

在CocoStudio中，序列帧动画的制作比较麻烦，一般需要手动添加帧，然后在每一帧中修改对象的图片资源，这种做法相当浪费时间，所以CocoStudio 2.x提供了自动创建序列帧动画的功能。在资源面板中，同时选中多个图片资源（按住Ctrl或者Shift键），右击，在弹出的快捷菜单中选择“创建序列帧动画”命令，会自动创建一个带序列帧动画的Sprite对象，如图30-8所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160812.jpeg)

图30-8　创建序列帧动画

### 30.3.2　在程序中播放动画

那么在代码中如何播放动画呢？我们的动画信息都保存在导出的.csb文件中，要播放动画的时候，需要调用CSLoader的createTimeline方法先将动画创建出来。在createTimeline中需要传入对应的.csb文件，该方法会返回一个ActionTimeline指针，接下来需要调用ActionTimeline的一些方法，来决定要播放哪段动画。ActionTimeline本身是一个Action，所以在最后需要让CSLoader创建出来的Node执行这个Action，这样才会有动画被播放出来。示例代码如下：

```
bool HelloWorld::init()
{
  //初始化父类
  if ( !Layer::init() )
  {
    return false;
  }
  //使用CSLoader的静态方法createNode来加载.csb文件
  auto rootNode = CSLoader::createNode("MainScene.csb");
  addChild(rootNode);
  //创建ActionTimeline对象，并调用play方法播放动画列表中的动画
  auto act = CSLoader::createTimeline("MainScene.csb");
  act->play("MyAnimation", true);
  //最后需要让rootNode执行Action
  rootNode->runAction(act);
  return true;
}
```

我们知道，CocoStudio中只有一个时间轴，如果有两段不冲突的动画，也就是逻辑上可以同时播放的动画，那么是否可以让这两段动画同时播放呢？在CocoStudio中将它们编辑到同一个动画中并不是很靠谱，首先这两个动画是互相独立的，并且播放的时机并不确定，有可能是动画A播放了50%，动画B才播放，并且由于这两个动画都是由指定的事件触发的，所以根本不知道对方什么时候会播放，这时可以通过两个ActionTimeline来完成这个功能。在需要播放动画的时候创建一个ActionTimeline对象，这是一个Action，然后调用runAction方法将这个Action传入。当然，这中间可能会有冲突。示例代码如下：

```
//创建ActionTimeline对象，并调用play方法播放动画列表中的动画
auto act1 = CSLoader::createTimeline("MainScene.csb");
act1->play("MyAnimation1", true);
//让rootNode执行Action
rootNode->runAction(act1);
//创建ActionTimeline对象，并调用play方法播放动画列表中的动画
auto act2 = CSLoader::createTimeline("MainScene.csb");
act2->play("MyAnimation2", true);
//让rootNode执行Action
rootNode->runAction(act2);
```

## 30.4　分辨率适配

CocoStudio 2.x的分辨率适配思路虽然同1.x版本相同，但使用方法却是截然不同。由于同时存在两种思路相同但操作方式不同的分辨率适配方法，所以很容易被误导。这里简单介绍一下CocoStudio 2.x下的分辨率适配操作，以及如何使其在我们的程序中生效。

在CocoStudio 2.x的场景、图层和节点csd文件中，可以对所有的节点设置其相对父节点的相对布局或百分比位置，另外还可以设置**UI控件**相对父节点的尺寸适配。具体在节点的属性面板中的位置与尺寸下可以进行设置，所有的设置都可以通过右侧的预览视图观察到预期效果，如图30-9所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160813.jpeg)

图30-9　布局设置

如果编辑的是场景文件，那么通过调整当前场景的分辨率可以看到在不同分辨率下的效果。

#### **1．布局**

相对布局可以通过选中图中上、下、左、右的4个图钉按钮来设置。图钉旁的数字表示节点对应边距离父节点对应边的距离，也就是4个方向的对齐。如果期望节点向右上角对齐，那么只需要选中右方和上方的图钉即可。

当选中了图钉后，对应的输入框会被激活为可输入的状态。这时我们可以调整节点的相对位置，例如选中上方的图钉，然后输入0，会发现节点的上边紧贴在了父节点的上边，如图30-10所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160814.jpeg)

图30-10　设置布局

这4个图钉整合了CocoStudio 1.6中的4种不同的布局方式：**绝对布局、相对布局、线性横向和线性纵向布局**。它们简单直接，而且更加灵活。

#### **2．百分比位置**

在属性面板的位置与尺寸的坐标选项中有两种模式：像素模式及相对父容器的百分比。百分比模式将会根据**父容器的尺寸**结合所设置的百分比来决定位置。如图30-11所示，可以分别对X轴和Y轴设置不同的模式。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160815.jpeg)

图30-11　坐标模式

百分比模式和节点布局设置是相互影响的。如果激活了X轴的百分比模式，那么布局的左、右图钉会被同时激活，但无法控制图钉对应的参数，因为它将受百分比的值控制。例如，我们设置了X轴的百分比值为50，那么无论父节点如何变化，该节点都会处于父节点水平方向的中间位置。

如果在百分比模式下取消对应的图钉，那么百分比模式就会自动切换成像素模式。而当我们同时选中左和右的图钉后，节点的X轴会自动切换成百分比模式。所以说相对布局与百分比是互斥的。

#### **3．等比缩放**

对于CocoStudio中的UI控件，还可以通过设置让其尺寸根据父节点的尺寸变化。也就是说，当控件的父节点变大或变小时，控件的尺寸可以跟着等比例地变大或变小。通过选中4个图钉中间的两条线，可以让控件**尺寸**的宽和高随父节点的**尺寸**等比例缩放。如图30-12所示，我们选中了中间的横线，那么当父节点的宽度发生变化的时候，该节点的宽度将随之变化。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160816.jpeg)

图30-12　尺寸适配

#### **4．在程序中生效**

在CocoStudio 2.x中编辑一个场景或层的时候（CocoStudio 2.x的节点不存在这个问题，因为节点的尺寸为0），当设置好各种适配之后，将生成的CSB加载到场景中，就会发现分辨率适配规则看上去并没有生效，但如果在模拟器中运行，我们的布局在不同的分辨率下则是有效的。

这里需要设置其尺寸并调用ui::Helper::doLayout()，传入CSB节点来刷新该CSB节点的布局使其生效。代码如下：

```
Size frameSize = Director::getInstance()->getVisibleSize();
//设置场景或层的尺寸为当前分辨率的尺寸
node->setContentSize(frameSize);
//刷新布局
ui::Helper::doLayout(node);
auto scene = Scene::create();
scene->addChild(node);
Director::getInstance()->replaceScene(scene);
```

由于是递归刷新，所以也可以直接传入当前场景的根节点。不过传入需要刷新布局的节点会更合适一些。

#### **5．不生效的原因**

CocoStudio 2.x的布局规则都是需要根据父节点的尺寸来计算的。我们从CSB中创建节点的时候，是根据编辑时的尺寸进行布局计算的。

Layer的尺寸在编辑时是固定的。如果需要让Layer适配整个屏幕，那么就需要设置Layer的尺寸，然后通过ui::Helper的doLayout方法来重新进行布局计算。doLayout会递归遍历某节点下的所有节点，并进行布局计算。

Scene的尺寸在初始化的时候就会被设置为当前实际的分辨率。那么为什么CocoStudio编辑的场景的分辨率适配没能生效呢？因为CocoStudio 2.x编辑的场景实际输出的是一个Node而不是一个Scene，所以我们还是需要手动设置其尺寸，并执行doLayout方法来刷新布局。

## 30.5　高级特性

CocoStudio 2.0.5版本开始，增加了回调特性的功能，使用这个功能可以绑定节点到自定义的类型中，类似CocosBuilder的绑定类功能。这是非常实用的一个功能，可以方便CocoStudio和程序之间的交互，在编辑场景或编辑UI的时候非常有用。例如要在一个场景中摆放好敌人、障碍物等各种对象，那么这些对象在程序中就很可能是一些自定义的类，因为这些对象自身是有一些逻辑的，比如当角色碰到场景中的星星，星星会播放特效，然后消失并添加玩家的分数。如果不使用回调特性绑定类，那么使用起来就相当不方便了，需要通过各种get方法来获得这些对象，然后将这些对象管理起来，在另外一个类里操作这些对象。如果使用了自定义的类，那么在CocoStudio中创建星星的时候可以通过绑定一个星星类，在星星类中实现相应的逻辑。也就是说，**绑定自定义类可以允许在CocoStudio中编辑一些拥有特殊功能的对象**，也可以用来编辑像超级玛丽关卡这样的地图。例如使用CocoStudio编辑一些动态的UI，如背包等UI，我们无法在编辑的时候知道玩家的背包实际上是怎样的那么可以在程序中实现一个通用的Grid类，在Grid类的初始化中，根据玩家的背包数据来填充这个背包容器。

在编辑时注意只有根节点才可以绑定自定义的类！那么问题就来了，如果需要在一个场景中摆放多个自定义的星星怎么办呢？只需要将星星做成一个单独的节点文件，然后在场景中嵌入多个星星的.csd文件即可。

接下来看一下如何使用CocoStudio的回调特性。首先选择根节点对象，然后在“属性”面板中，选择“高级属性”，如图30-13所示，在“回调特性”下的“自定义类”文本框中输入自定义的类名，这里输入MyClass。注意，如果输入的类名与代码中注册的类名不一致，例如多了个空格之类的，那么CSLoader的createNode方法就会返回一个空指针，上层直接使用时会报错。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160817.jpeg)

图30-13　“属性”面板

为了在代码中加载绑定了自定义类的.csb文件，需要先实现自定义的类，类名不一定要与自定义类输入框中输入的类名一致，如类定义为CYourClass都是可以的，因为这里不是根据类名来识别。首先可以添加一个自定义的类CMyClass，这个类名并不重要，类必须继承于Node或Node的子类，并且必须有create方法，可以使用CREATE_FUNC宏快速添加create方法。只要满足这两点条件即可。另外，在onEnter回调中会弹出一个对话框I'm CMyClass，以此来检测自定义的类是否生效了。

```
#include <cocos2d.h>
//必须继承于Node或Node的子类
class CMyClass
  : public cocos2d::Node
{
public:
  CMyClass() { }
  ～CMyClass() { }
  //需要有create方法
  CREATE_FUNC(CMyClass);
  //在onEnter的时候弹出对话框
  virtual void onEnter()
  {
    Node::onEnter();
    cocos2d::MessageBox("I'm CMyClass", "Hello");
  }
};
```

接下来还需要实现一个专门创建CMyClass的工厂类CMyClassReader，该类名字没有硬性规定一定要叫什么，我们需要继承于cocostudio::NodeReader，并添加对应的单例方法，实现createNodeWithFlatBuffers虚函数，在虚函数中需要创建一个自定义的Node，也就是CMyClass，然后调用setPropsWithFlatBuffers来设置Node的属性，最后将Node返回。

```
#include "cocostudio/WidgetReader/NodeReader/NodeReader.h"
class CMyClassReader
  : public cocostudio::NodeReader
{
public:
  CMyClassReader() {};
  ～CMyClassReader() {};
  //单例方法
  static CMyClassReader* getInstance();
  static void purge();
  //创建类
  cocos2d::Node* createNodeWithFlatBuffers(const flatbuffers::Table* 
nodeOptions)
  {
    CMyClass* node = CMyClass::create();
    setPropsWithFlatBuffers(node, nodeOptions);
    return node;
  }
private:
  static CMyClassReader* m_Instance;
};
```

由于声明了单例对象，所以还需要在源文件中实现对应的方法并定义单例成员变量，否则会报未定义的错误。

```
//定义单例成员变量
CMyClassReader* CMyClassReader::m_Instance = NULL;
//单例方法
CMyClassReader* CMyClassReader::getInstance()
{
  if (NULL == m_Instance)
  {
    m_Instance = new CMyClassReader();
  }
  return m_Instance;
}
//单例销毁方法
void CMyClassReader::purge()
{
  CC_SAFE_RELEASE_NULL(m_Instance);
}
```

最后，在使用CSLoader进行createNode之前，需要将CMyClassReader注册到CSLoader中，调用CSLoader的registReaderObject方法，传入注册的类名，以及CMyClassReader的getInstance函数指针即可。需要特别注意的是，**传入的类名为编辑器中编辑的类名，并且后面需加上Reader**。如在编辑器中编辑的类名为MyClass，那么需要注册的名字是MyClassReader，而不是MyClass。

```
bool HelloWorld::init()
{
  //初始化父类
  if ( !Layer::init() )
  {
    return false;
}
CSLoader::getInstance()->registReaderObject(
"MyClassReader",
(ObjectFactory::Instance)CMyClassReader::getInstance);
  auto rootNode = CSLoader::createNode("MainScene.csb");
  addChild(rootNode);
  return true;
｝
```

编译并运行代码，在程序启动时就可以看到如图30-14所示的对话框。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160818.jpeg)

图30-14　I’m CMyClass对话框

此外，所有的UI控件都拥有单击回调的回调特性，这是一个比较鸡肋的特性，当设置了回调之后，单击该UI控件时，**会回调当前CocoStudio根节点的一个回调方法**，当然，前提是根节点必须是自定义的类。如果根节点是普通的节点，那么单击之后不会有事情发生。

可以在场景中添加一个UI控件，这里添加一个ImageView控件。然后在“属性”面板的“高级属性”下找到“回调特性”，在“回调方法”下拉列表中，选择Click或Touch，对应UI的Click回调和Touch回调，然后在旁边的文本框中输入事件名字（事件名后面将会介绍），如图30-15所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160819.jpeg)

图30-15　“属性”面板“回调方法”

接下来需要在代码中处理这个单击事件，在CSLoader加载节点的时候，如果加载到有回调方法的UI控件（也就是Widget），会将当前加载的Node（根节点）转换为WidgetCallBackHandlerProtocol对象，如果转换成功，再调用WidgetCallBackHandlerProtocol对应的方法，来获取回调函数，并设置为当前UI控件的点击回调。当单击的时候，UI控件对应的单击回调会被触发，此时会执行根节点对象返回的回调方法，阅读完下面的代码就比较好理解了。

首先需要将自定义类继承于WidgetCallBackHandlerProtocol，WidgetCallBack HandlerProtocol是一个用于处理控件回调的接口类，包含3个方法，用于返回3种不同的回调，默认都是返回空。

```
namespace cocostudio {
  class CC_STUDIO_DLL WidgetCallBackHandlerProtocol
  {
  public:
    WidgetCallBackHandlerProtocol() {};
    virtual ～WidgetCallBackHandlerProtocol() {};
    //返回一个触摸回调
    virtual cocos2d::ui::Widget::ccWidgetTouchCallback
    onLocateTouchCallback(const std::string &callBackName){ return
 	nullptr; };
    //返回一个点击回调
    virtual cocos2d::ui::Widget::ccWidgetClickCallback
    onLocateClickCallback(const std::string &callBackName){ return
 	nullptr; };
    //返回一个事件回调，当前版本的CocoStudio暂不支持
    virtual cocos2d::ui::Widget::ccWidgetEventCallback
    onLocateEventCallback(const std::string &callBackName){ return
 	nullptr; };
  };
}
```

这里将CMyClass继承于WidgetCallBackHandlerProtocol，注意，这里是一个多继承并且需要包含相应的头文件。我们在CocoStudio中编辑的是单击事件，所以可以只重写onLocateClickCallback函数。该函数会返回一个ccWidgetClickCallback对象，ccWidgetClickCallback 对象的定义为std::function<void(Ref*)>，也就是对应void fun(Ref* pSender)函数原型。这里可以返回一个原型相同的成员函数（需要用CC_CALLBACK_XX等宏来包装，如返回成员函数onClicK，则需要return CC_CALLBACK_1(CMyClass:: onClicK, this);），也可以直接返回一个lambda函数。

```
#include <cocos2d.h>
#include "cocostudio/WidgetCallBackHandlerProtocol.h"
//使用了多继承
class CMyClass
  : public cocos2d::Node
  , public cocostudio::WidgetCallBackHandlerProtocol
{
public:
  CMyClass();
  ～CMyClass();
  CREATE_FUNC(CMyClass);
  //实现了WidgetCallBackHandlerProtocol的虚函数，返回单击回调
  virtual cocos2d::ui::Widget::ccWidgetClickCallback
  onLocateClickCallback(const std::string &callBackName)
  {
    //传入的callBackName为在CocoStudio中编辑的回调名字
    if (callBackName == "TestClick")
    {
      return[](Ref* psender)->void
      {
        cocos2d::MessageBox("TestClick CallBack is Called", "Hello");
      };
    }
    return nullptr;
  };
};
```

此外，还需要在CocoStudio中确保UI控件属性面板中，“基础属性”→“常规”→“交互性”，是处于选中状态，否则不会监听单击事件。接下来发布资源并运行程序，单击添加的图片，弹出如图30-16所示的对话框。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160820.jpeg)

图30-16　Hello对话框

当在CocoStudio中注册了一个回调事件，而这个csd文件又被嵌套在另外一个.csd文件中，如果子.csb文件没有绑定自定义类，或者绑定的类没有继承协议类WidgetCallBackHandlerProtocol，那么在子节点中设置的回调方法，会在根csd所绑定的自定义类中返回（前提是根.csd绑定了继承于WidgetCallBackHandlerProtocol的类），这需要Cocos2d-x 3.5之后的版本（3.4的版本存在BUG）。

## 30.6　其他特性

#### **1．帧事件**

在场景中对象的高级属性面板中，可以编辑帧特性下的帧事件。帧事件顾名思义是在动画播放到某一帧的时候，触发一个事件，在程序中监听这个事件，当事件触发时，调用相应的回调。

要添加帧事件，首先需要选中动画面板中的自动记录帧这个选项，如图30-17所示。然后在需要触发帧事件的节点中，在其属性面板的高级属性标签页的帧特性中，输入帧事件的名称，然后按Enter键，如图30-18所示。这时我们可以缓慢播放动画，或手动拉动当前时间轴。可以发现，当动画播放到了指定帧时，节点属性的帧事件属性会发生改变。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160821.jpeg)

图30-17　自动记录帧

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160822.jpeg)

图30-18　编辑帧事件

选中了自动记录帧时，当节点属性发生变化时会自动在时间轴面板中记录一帧，否则需要在时间轴面板中手动添加一帧，然后记录修改。

接下来可以在代码中监听这个帧事件。因为这个帧事件是在动画的某一帧，由某个节点触发的，所以这里需要先使用导出的.csb文件创建一个动画，也就是ActionTimeline对象。调用其setFrameEventCallFunc方法可以设置帧事件触发时的回调。最后由节点执行这个动画，等待帧事件的触发即可。TestCpp中的ActionTimelineTestScene中演示了如何使用，代码如下：

```
void TestTimelineFrameEvent::onEnter()
{
    ActionTimelineBaseTest::onEnter();
    //创建并播放动画
    Node* node = CSLoader::createNode("ActionTimeline/DemoPlayer.csb");
    ActionTimeline* action = CSLoader::createTimeline("ActionTimeline/
    DemoPlayer.csb");
    node->runAction(action);
    action->gotoFrameAndPlay(0);
    node->setScale(0.2f);
    node->setPosition(150,100);
    addChild(node);
    //注册帧事件
    action->setFrameEventCallFunc(CC_CALLBACK_1(TestTimelineFrame
     Event::onFrameEvent, this));
}
//帧事件的回调
void TestTimelineFrameEvent::onFrameEvent(Frame* frame)
{
    //强转为EventFrame
    EventFrame* evnt = dynamic_cast<EventFrame*>(frame);
    if(!evnt)
        return;
    //获取事件名
    std::string str = evnt->getEvent();
    //getNode将返回执行动画的节点
    if (str == "changeColor")
    {
        evnt->getNode()->setColor(Color3B(0,0,0));
    }
    else if(str == "endChangeColor")
    {
        evnt->getNode()->setColor(Color3B(255,255,255));
    }
}
```

#### **2．自定义数据**

在场景中对象的高级属性面板中还可以编辑用户数据，输入自定义的数据。这里主要是输入字符串。自定义数据的作用类似于Tag，但字符串可以保存更多的数据。然而，在目前的CocosStudio中，仍然不能使用这一特性，所以请直接忽略它，无须纠结如何获得在CocosStudio中编辑的用户数据，因为目前的CocosStudio在加载Node时不会去解析这个属性。

#### **3．骨骼动画**

严格来说，CocoStudio 2.x对骨骼动画的支持并不好。我们期望的骨骼动画对象是一个Armature对象，而不是一个普通的Node对象。因为Armature对象拥有非常强大的功能。那么使用了CocoStudio 2.x之后，应该如何使用骨骼动画呢？如果不需要在CocoStudio 2.x中编辑它，那么就直接使用CocoStudio 1.6导出的.csb骨骼文件；如果需要在CocoStudio2.x中放置它，将它拖放到场景中，则可以使用CocoStudio 2.x中的Armature对象，然后导入CocoStudio 1.6的.json骨骼文件。最后一种方法是将CocoStudio 1.6的骨骼动画导入，转为CocoStudio 2.x的普通动画，但这会丢失很多内容。

CocoStudio 2.3.2 Beta版本开始支持编辑骨骼动画，在新建文件时可以发现新增了骨骼动画的选项，如图30-19所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160823.jpeg)

图30-19　创建骨骼动画

还可以导入CocoStudio 1.6的骨骼动画，通过文件→导入→导入1.6版本项目...导入到CocoStudio中，将会生成新的骨骼动画，该骨骼动画保留了骨骼关系及动画列表等特性，如图30-20所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160824.jpeg)

图30-20　导入1.6的骨骼动画

需要注意的是，新的骨骼动画文件不是一个Armature对象，而是一个SkeletonNode对象，它需要**Cocos2d-x 3.7.1版本以上**的引擎才能够解析。

SkeletonNode的定义位于引擎目录下的cocos\editor-support\cocostudio\ActionTimeline目录中的CCSkeletonNode.h和CCSkeletonNode.cpp文件中。在TestCpp的TestActionTimelineSkeleton示例中演示了SkeletonNode的使用。与Armature的功能类似，但代码简洁很多。

## 30.7　在Windows下调试引擎

在Windows下使用Cocos创建的项目并不能调试Cocos2d-x的代码，非常不方便，例如，当需要单步调试的时候无法跟进Cocos2d-x的代码，或者程序在Cocos2d-x内部崩溃的时候无法定位到崩溃处的堆栈，如图30-21所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160825.jpeg)

图30-21　调试

首先Cocos引擎自身的Cocos Framework没有工程文件，也没有源码，有的只是一堆的头文件以及编译好的lib和dll文件，那么怎样才能调试到Cocos2d-x的源码呢？首先需要下载对应版本的Cocos2d-x引擎源码并进行编译。这个很好理解，要进入cpp源码中，而Cocos Framework并没有引擎的cpp源码，那么自然需要下载一份版本对应的源码。接下来需要将头文件的搜索路径、库路径，以及链接的库都指向最新的源码中，然后修改项目的一些属性，这些操作有些麻烦，具体步骤如下：

（1）下载Cocos2d-x的引擎源码 http://www.cocos.com/download/cocos2d-x/，解压至一个纯英文路径下，对解决方案进行编译。

（2）在项目属性→C/C++→常规→附加包含目录下，将指向引擎的包含路径修改为下载引擎的路径（可直接修改COCOS_FRAMEWORKS或COCOS_X_ROOT环境变量）。

（3）将Cocos2d-x引擎编译生成的dll、lib、pdb文件复制至项目的输出目录，即proj.win32\Debug.win32目录，该目录视项目的$(OutDir)环境变量而定。

（4）在main.cpp中移除链接libcocos2d_201x.lib的源码，因为这些lib文件是Cocos引擎自带的并没有包含调试信息，而且需要链接的是下载的Cocos2d-x引擎编译出来的代码。

```
#if _MSC_VER > 1700
#pragma comment(lib,"libcocos2d_2013.lib")
#pragma comment(lib,"libbox2d_2013.lib")
#pragma comment(lib,"libSpine_2013.lib")
#else
#pragma comment(lib,"libcocos2d_2012.lib")
#pragma comment(lib,"libbox2d_2012.lib")
#pragma comment(lib,"libSpine_2012.lib")
#endif
```

（5）在项目属性→链接器→输入→附加依赖项下，添加项目的链接库为libcocos2d.lib、glew32.lib、因为libcocos2d_2013.lib将glew32.lib一起链接进来了，所以链接了libcocos2d_2013.lib就不需要再链接glew32.lib了，但原始的libcocos2d.lib并没有链接它。

（6）将项目属性——C/C++——代码生成——运行库，从多线程DLL(/MD)修改为多线程调试DLL(MDD)。Cocos引擎生成的DEBUG项目实际上使用的是Release的设置。

（7）删除项目属性中生成事件→预链接生成事件，以及命令行中的内容，避免额外的麻烦。

（8）重新编译项目，再次调试即可进入Cocos引擎源码了。