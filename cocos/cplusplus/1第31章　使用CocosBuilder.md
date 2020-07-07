# 第31章　使用CocosBuilder

CocosBuilder是一个第三方的开发工具，可以用于设置游戏界面及动画。CocosBuilder 3.0还支持直接编辑JS脚本，以及直接运行游戏。不过CocosBuilder只有Mac版本，所以无法在Windows下使用。

之所以介绍CocosBuilder，因为是一个不错的工具。在CocoStudio出现之前（甚至出现后很长的一段时间内），是程序员编辑场景、界面的最佳选择，并且CocoStudio后面新增的一些绑定类、绑定回调等有用的特性，也是参照了CocosBuilder的特性，但可惜的是，目前CocosBuilder已经停止维护了。本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　CocosBuilder简介。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　编写代码加载CCBI。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　播放动画。

## 31.1　CocosBuilder简介

### 31.1.1　CocosBuilder界面简介

Mac下的程序启动之后，可以分为主界面和菜单两部分，先来了解一下CocosBuilder的界面。CocosBuilder界面的整体结构如图31-1所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160901.jpeg)

图31-1　CocosBuilder界面

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　左上角的按钮提供了编辑区域的缩放功能，以及单击模式和拖曳模式的切换。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　右上角的按钮用于在CocosBuilder中创建控件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　左边的项目树对应CocosBuilder项目文件所在的文件夹路径，在Finder中添加的资源文件都可以在Project视图下找到。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　右边的属性面板用于编辑单个对象的属性（一般是节点），如位置、图片、旋转等。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　正下方的时间轴面板可以编辑动画，也可以找到当前场景中所有的节点，可以编辑节点之间的父子关系。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　中间的编辑视图可以很直观地进行场景编辑。

移动鼠标到屏幕的最上方可以发现CocosBuilder的菜单栏，如图31-2所示。CocoBuilder的菜单栏包含文件、编辑、对象、视图、动画、窗口、帮助等菜单。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160902.jpeg)

图31-2　CocosBuilder菜单栏

### 31.1.2　工作流程

接下来看一下CocosBuilder的工作流程，如图31-3所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160903.jpeg)

图31-3　工作流程

（1）在CocosBuilder中编辑界面（可以设计各种动画及自定义类、绑定回调函数等）。

（2）将CocosBuilder项目发布，生成ccbi文件。

（3）将.ccbi文件和相关的图片资源导入项目中，与普通的资源一样。

（4）如果在CocosBuilder中添加了自定义类（Custom class），需要在项目中为自定义类编写代码。

（5）在需要创建界面的地方调用CocosBuilder库的函数创建它们，并添加到游戏场景中。

### 31.1.3　常见问题

下面是一些在使用CocosBuilder的过程中比较常见的问题。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　顺序问题，修改Default Timeline面板的节点先后顺序，可以决定显示的先后关系，其实也是添加节点的先后顺序，并没有ZOrder。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　菜单CMenu只可以添加CCMenuItemImage对象为子节点。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　**View Resolution可以选择不同的分辨率进行编辑**，这个功能很有用，因为可以直接看到当前编辑的场景，在不同的分辨率下有怎样的表现。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　添加子节点，选中一个节点，再选中“控件”面板即可添加为子节点，也可以在“时间轴”面板中，将子节点拖曳到父节点上自动成为其子节点，也可以从父节点中拖出来。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　CCSprite9Scale是九宫精灵，适用于拉伸按钮，但在代码方面，一个CCSprite9Scale需要创建9个CCSprite，如果只是想添加一个CCSprite，请不要使用CCSprite9Scale。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　CMD和+ 快捷键可以放大编辑视图，CMD和-快捷键可以缩小编辑视图，双手指拖动可以平移编辑视图。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Menu和Control是不同的控件，相比Cocos2d-x的CCMenu和CCControlButton、Control控件更强大一些，但大部分的按钮都是选择Menu，因为大部分按钮的需求都很简单，简单的需求就用简单的方法来做。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　当程序运行后找不到图片资源时，一定是没有将图片资源添加到XCode的资源目录下，这个操作需要在XCode的项目面板中手动添加图片资源才可以。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　版本问题导致创建的Node为空。Cocos2d-x 2.0.4-3对应CocosBuilder 2.1。另外，**CocosBuilder 3.0需要下载完整源码，编译出cocosplayer才可以发布ccbi文件**，且有BUG。在Cocos2d-x中对应CCLayerLoader的解析是isxxx属性字符串，而CocosBuilder 3.0生成的却是xxx属性字符串，前面没有is，所以在属性检查的时候会崩溃。这个BUG应该是CocosBuilder的BUG，因为Cocos2d-x中所有版本的解析类都是isxxx，但是CocosBuilder 3.0却把这个前缀去掉了。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　从CocosBuilder 1.0到CocosBuilder 2.1的几个版本（笔者只尝试了2.1，1.0似乎也是如此）生成的.ccbi文件版本号都是3，而Cocos2d-x 2.0.2之前引擎要求的ccbi版本号都是2，版本不对应，所以不可用！而Cocos2d-x 2.0.4版本是3，可用！**CocosBuilder 3.0需要对应Cocos2d-x 2.1及以后的版本**。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　千万别忘了**先保存再发布**。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　不要在资源文件或文件夹名中加上空格，那样XCode读取资源会失败，必须把空格去掉，但去掉之后会发现图片丢失的问题，所有的图片都丢失了。这时可以用文本编辑器快速查找替换.ccb中的空格，这样效率更高，比手动重新选择图片快N倍。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　CCScale9Sprite显示错位的问题，这个问题是因为CCScale9Sprite忽略了锚点而导致，**选中ignore anchor point即可解决问题**（这是CocosBuilder的一个BUG，最新版本不知是否修复了该错误。有一个美术人员将场景中五十多个节点都用CCSprite9Scale，加载到场景中时全部错位了，一个一个修改非常浪费时间，可打开CocosBuilder的.ccb文件，这是一个XML文件，启用快速替换功能，可快速将所有的节点改过来）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　图片路径问题，不要选中Flatten paths when publishing复选框，**该复选框会将资源的路径去掉，只保留文件名，如果选中需要将.ccbi文件删除再重新发布**，否则资源名不会改变。之所以有这个复选框，是因为当将所有的UI图片打包成一张图集的时候，只有图片名称而没有路径，所以如果将UI图片打成图集，可以选中该复选框，如图31-4所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160904.jpeg)

图31-4　Flatten paths when publishing复选框

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　在新建一个场景、层、节点的时候（如图31-5所示），可以选择Root object type为CCLayer，并设置其支持的各种分辨率。而其他情况下，可以选择一个CCNode或者CCSprite来设计一个弹出对话框或一个角色。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160905.jpeg)

图31-5　创建新文档

## 31.2　编写代码加载ccbi

介绍完CocosBuilder编辑器的使用之后再介绍一下如何编写代码，让我们的程序加载CocosBuilder生成的.ccbi了。

使用CocosBuilder导出的.ccbi，本质上就是根据CocosBuilder的描述，生成一个一模一样的节点。每个.ccbi都对应一个节点（这里指根节点），这个根节点下可能会有很多的子节点。

### 31.2.1　创建普通节点

根据.ccbi创建一个基础的节点，步骤如下。

（1）准备工作，包含头文件：

```
#include "cocos-ext.h"
```

（2）在源文件中声明命名空间：

```
USING_NS_CC_EXT;
```

（3）初始化CCB相应的库：

```
CCNodeLoaderLibrary *lib = CCNodeLoaderLibrary::sharedCCNodeLoader Library();
CCBReader *reader = new CCBReader(lib);
```

（4）传入对应的.ccbi文件，创建Node：

```
CCNode* pNode =reader->readNodeGraphFromFile("testNode.ccbi");
```

（5）最后千万不要忘了释放reader：

```
reader->release();
```

现在我们有了一个基础的节点，但在很多情况下需要将CocosBuilder中编辑的节点绑定到某个类上，例如，当需要为拖放上去的按钮添加一个回调函数的时候，以及需要为这个节点编写代码的时候，都需要将节点绑定到一个类上。

### 31.2.2　绑定到类

接下来介绍在CocosBuilder中将节点绑定到类的步骤。要将节点绑定到类需要做3件事情：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　需要先把这个类写出来。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　需要为这个类写一个加载器。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　将加载器注册到CCNodeLoaderLibrary中，以便于创建节点。

对于这个类的要求很简单，直接或间接继承于CCNode，并且需要有一个静态的create方法。例如，笔者写了CUIAccountBox类，用来弹出一个结算对话框，需要在该类的函数声明中加上下面这行代码：

```
public:
    CCB_STATIC_NEW_AUTORELEASE_OBJECT_METHOD(CUIAccountBox, create);
```

类的加载器是比较简单的，基本上每个类的加载器的代码都差不多，只是名字不一样：

```
class CUIAccountBoxLoader : public cocos2d::extension::CCNodeLoader
{
public:
CCB_STATIC_NEW_AUTORELEASE_OBJECT_METHOD(CUIAccountBoxLoader, loader);
protected:
CCB_VIRTUAL_NEW_AUTORELEASE_CREATECCNODE_METHOD(CUIAccountBox);
};
```

上面的**Loader继承于CCNodeLoader**，并且用CCB的宏定义了两个方法——loader静态方法，返回一个CUIAccountBoxLoader及CreateNode方法。

CUIAccountBoxLoader会调用CUIAccountBox的create方法。**假设CUIAccountBox没有create方法**（注意大小写），那么编译会出错。

最后就可以创建自定义的节点对象了。创建方法与创建一个Cocos2d-x节点类似，调用CCBReader的readNodeGraphFromFile方法，传入要加载的ccbi文件来创建自定义的节点对象。

```
CCNodeLoaderLibrary *lib = CCNodeLoaderLibrary::sharedCCNodeLoader Library();
lib->registerCCNodeLoader("UIAccountBox", 
CUIAccountBoxLoader::loader());
CCBReader *reader = new CCBReader(lib);
CUIAccountBox* box = dynamic_cast<CUIAccountBox*>(reader->readNodeGraph 
FromFile("ui/account.ccbi"));
reader->release();
```

需要注意的代码是第2行和第5行。第2行代码绑定了**UIAccountBox类**，将CUIAccountBoxLoader的加载方法注册进去。一旦读取到某个节点绑定了UIAccountBox类，就会调用CUIAccountBoxLoader来创建这个节点。

第4行代码在读取节点的时候做了一个动态转换，这个时候可以使用box的任何成员方法，当然也可以不转换，把它当做Node来使用。

### 31.2.3　绑定回调函数

接下来了解一下如何调用按钮的回调方法，在CocosBuilder中经常会摆放一些按钮，这时候往往需要为按钮设置单击的回调函数。

在设置完回调函数之后，需要在绑定的类中实现该回调函数。需要继承一个CCBSelectorResolver，其会有两个接口来实现。需要实现onResolveCCBCCMenuItemSelector接口来绑定/注册MenuItem的单击回调方法，实现onResolveCCBCCControlSelector接口来绑定/注册CCControl的回调。

```
virtual cocos2d::SEL_MenuHandler onResolveCCBCCMenuItemSelector (cocos2d:: 
CCObject * pTarget, cocos2d:: CCString * pSelectorName);
virtual cocos2d::extension::SEL_CCControlHandler onResolveCCBCCControl 
Selector(cocos2d::CCObject * pTarget, cocos2d::CCString * pSelectorName);
```

例如，在CocosBuilder中将某个按钮的回调绑定到CUIAccountBox函数的ButtonClick方法中，那么需要在这里将ButtonClick绑定到实际的回调函数中：

```
SEL_MenuHandler CUIAccountBox::onResolveCCBCCMenuItemSelector(CCObject * 
pTarget, CCString * pSelectorName)
{
   CCB_SELECTORRESOLVER_CCMENUITEM_GLUE(this, "ButtonClick", CUIAccountBox::
   onButtonClick);
   return NULL;
}
```

这里使用了CCB_SELECTORRESOLVER_CCMENUITEM_GLUE宏来绑定一个回调函数的标识，到一个真正的回调函数中。

这里将ButtonClick这个字符串对应到了CUIAccountBox的onButtonClick方法中，而这个onButtonClick方法就是一个正常的MenuItem单击回调方法。

最后分享一点经验：当我们在写一个类的Loader的时候，往往会去继承CCNodeLoader。但是，当自定义的类是一个Sprite或者MenuItem类时，**必须继承CCSprite Loader或者CCMenuItemLoader类才可以**，否则创建出来的对象不会执行Sprite或MenuItem的初始化。

### 31.2.4　绑定成员变量

绑定成员变量的本质是将一个节点作为另一个节点的成员变量。

例如，用CocosBuilder来编辑这样的一个UI，界面的层次非常复杂，有三五层父子节点层次在那里。假设需要获取一个子节点的子节点的子节点，那么可以想象代码是什么样的。这样的代码是死代码，只要节点层次稍微变动一下，代码就会失效。

上面是根据Cocos2d-x的节点机制，来定位到节点树下的某个节点。要避免这种情况，一般会将这个隐藏很深的子节点直接放到最上层，这样可以直接使用getChild方法来获取这个节点。当确实需要将其藏得很深的时候，可以直接使用成员变量来获取这个节点，这样就无须调用一堆巨长无比的getChildByTag方法一层一层地查找隐藏在深处的节点。

在代码里创建节点时，可以在节点创建的时候设置为某个类的成员变量，而在CocosBuilder中，可以这么做：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　让节点继承于CCBMemberVariableAssigner，这里的节点指的是拥有成员变量的节点，如A是B的成员变量，可以B->A.xxx()。这个需要继承于CCBMemberVariable Assigner的节点就是B节点。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　重写绑定成员变量的接口。

```
virtual bool onAssignCCBMemberVariable(cocos2d::CCObject* pTarget, const 
char* pMemberVariableName, cocos2d::CCNode* pNode);
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　在onAssignCCBMemberVariable中绑定成员变量。

```
CCB_MEMBERVARIABLEASSIGNER_GLUE(this, "TimeText", CCLabelTTF*, this-> 
m_TimeText);
return true;
```

在这里将一个文本控件绑定到CUIAccountBox的成员变量m_TimeText中，而在CocosBuilder中填写的变量名为TimeText，是一个CCLabelTTF类型的指针。

### 31.2.5　初始化流程

我们习惯在init函数中对成员变量进行初始化、设置，在这里我们无法在init函数中对前面绑定的m_TimeText变量进行初始化。因为在init中成员变量还未被绑定，具体可以看以下流程。

（1）构造函数，按照节点数从上到下进行构造，每个构造函数调用完，紧接着调用init。

（2）对成员变量、回调函数等进行绑定。

（3）调用CCNodeLoaderListener::onNodeLoaded，在类以及其对应的子节点全部创建完毕后，会回调类的onNodeLoaded。

（4）此时可以获得一个创建成功的节点，对其进行操作，可以调用其他一些初始化方法对在init中未能初始化的变量进行初始化。

（5）一般，创建完成会将节点添加到场景中，此时，在onEnter里也可以进行这个初始化。

结合上面的流程，当需要对自定义的类进行一些初始化时，可以直接放在init中，假设需要初始化绑定的成员变量可以继承CCNodeLoaderListener，在onNodeLoaded方法中初始化它们。

假设这个初始化需要更多的操作（如需要其他的ccbi加载完成之后才能执行），可以在节点创建完成之后，在合适的时机调用另外的一个initXXX方法来初始化或执行一些代码。

最后，可以选择在onEnter函数中来写这些代码。当然，不要忘记调用父类的onEnter方法。

### 31.2.6　在Cocos2d-x 3.x版本中使用

由于Cocos2d-x 3.x版本的代码发生了较大的变化，所以在方法、接口、命名方面都需要做相应的调整。具体可以参考1.4节中的约定的内容，以及TestCpp中ExtensionsTest下的CocosBuilderTest的示例代码，在其中可以找到当前版本的写法。

## 31.3　播放动画

在CocosBuilder中既可以编辑骨骼动画也可以编辑逐帧动画。CocosBuilder的的逐帧动画是在骨骼动画的基础上稍加变化而成的，该功能虽然说比较强大，但实际用到的不多。

在制作动画的时候，相对于使用CocosBuilder和CocosStudio，美术人员更倾向于使用Flash或者Spine等工具来编辑动画，主要是体验、习惯、以及制作效率等方面的问题。

本节并不介绍在Cocos2d-x里如何使用Flash制作动画。关于骨骼动画，可以参考**第**29章的内容，本章的主题还是CocosBuilder。

播放动画这里分为两个部分，一部分是如何在CocosBuilder中编辑动画，另一部分是如何在代码中进行调用，播放CocosBuilder导出的动画。

### 31.3.1　编辑动画

CocosBuilder动画基于时间轴，每个时间轴表示一段动画，时间轴有名字和时间、是否自动播放等属性。

首先需要在Animation下添加一条时间轴，选择Animation菜单，然后选择Edit Timelines选项，如图31-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160906.jpeg)

图31-6　动画菜单

在弹出的对话框中单击左下角的“+”按钮创建一个时间轴，可以设置时间轴的名字及总时间，如图31-7所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160907.jpeg)

图31-7　添加时间轴

创建好时间轴之后，可以在最下面的编辑面板中进行编辑，如图31-8所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160908.jpeg)

图31-8　编辑动画

编辑动画的步骤如下：

（1）选择要编辑的节点。

（2）拖动时间轴顶部的箭头，选择当前要编辑的位置。

（3）选择Animation菜单，插入关键帧。

（4）在关键帧中对节点的属性进行调整。

（5）重复步骤（2），直到动画编辑完成。

当插入关键帧的时候可以选择如图31-9所示的7种类型的关键帧。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160909.jpeg)

图31-9　关键帧

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Visible：允许在时间轴中动态设置节点在哪一帧隐藏，哪一帧显示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Position：允许设置从某一帧开始，到某一帧结束的位移。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Scale：允许设置从某一帧开始，到某一帧结束的缩放。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Rotation：允许设置从某一帧开始，到某一帧结束的旋转。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Sprite Frame：允许设置任意帧的图片，可以用来做逐帧动画。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Opacity：允许设置从某一帧开始，到某一帧结束的透明度。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160900.jpeg)　Color：允许设置从某一帧开始，到某一帧结束的颜色。

操作的细节是在每插入一个关键帧的时候，在右侧的属性编辑面板中，对节点对应的属性进行修改。

Visible和Sprite Frame帧是不会形成过渡动画的，也就是每两个关键帧之间不会有紫色的进度条，而对于其他属性，每两个关键帧之间会自动生成过渡动画。

例如，在时间轴开始的位置添加一个旋转关键帧，旋转为0°，时间轴结束的位置添加一个旋转为360°的旋转关键帧，播放动画时自然会从0°慢慢旋转到360°，而设置逐帧动画，只会在某个关键帧变化为下一帧的图片，并没有过渡效果。

### 31.3.2　编写代码

接下来看一下如何在代码中调用动画，可以理解为每一条时间轴表示一个动画，而每一条时间轴都有一个名字，可以通过这个名字来播放动画。

需要注意的是：假设这个动画是多个节点参与其中的，那么播放动画时所有参与的节点都会执行动画，并不是在节点1里播放动画，就只有节点1播放该动画。

我们的动画都在CCBReader的AnimationManager中保管着，所以要播放动画的两个步骤是获取AnimationManager，然后调用AnimationManager的相关接口来播放动画。

通过CCBReader的readNodeGraphFromFile，传入一个CCBAnimationManager的二级指针，可以获取CCBAnimationManager对象。在加载完节点之后，调用CCBReader的getAnimationManager也可以获得CCBAnimationManager对象。

在获取到CCBAnimationManager对象之后，需要将其保存为自己的一个成员变量，并且调用retain方法（在释放CCBReader的时候，release方法会被调用一次），以便于在任何时候使用它。

```
CCBAnimationManager *actionManager = NULL;
CCNodeLoaderLibrary * ccNodeLoaderLibrary = CCNodeLoaderLibrary:: newD 
efaultCCNodeLoaderLibrary();
ccNodeLoaderLibrary->registerCCNodeLoader("TestHeaderLayer", Test Header 
LayerLoader::loader());
ccNodeLoaderLibrary->registerCCNodeLoader("TestAnimationsLayer", Anima 
tionsTestLayerLoader::loader());
cocos2d::extension::CCBReader * ccbReader = new cocos2d:: extension:: 
CCBReader(ccNodeLoaderLibrary);
ccbReader->autorelease();
CCNode *animationsTest = ccbReader->readNodeGraphFromFile ("ccb/ccb/ 
TestAnimations.ccbi", this, &actionManager);
((AnimationsTestLayer*)animationsTest)->setAnimationManager(action 
Manager);
```

上面这段代码介绍了获取CCBAnimationManager并保存为成员变量的过程。

运行一个动画非常简单，只需要调用CCBAnimationManager对象的runAnimations即可，传入要播放的动画名称或ID，可以传入每一帧的时间间隔，下面的代码以0.3秒的间隔播放Idle动画。

```
mAnimationManager->runAnimations("Idle", 0.3f);
```