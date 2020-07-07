# 第28章　CocoStudio场景编辑器

本章介绍CocoStudio场景编辑器的使用以及场景编辑器与组件系统、触发器的使用。场景编辑器的功能主要有两个：一个是整合资源，通过场景编辑器中的组件，将数据、UI、动画编辑器的资源整合成场景；另一个是通过触发器工具，编辑游戏的逻辑，驱动游戏。

官方定义中场景编辑器的使用者主要是策划人员，而触发器的这个思路是比较不错的，但对策划人员的要求略高，场景编辑器的理想使用状态应该是程序员编写好一切可能的触发器条件、动作和时机，策划人员创建大量的触发器，组织游戏中的所有逻辑。

但实际情况很可能导致程序员和策划人员之间的耦合性过大，浪费了大量时间在沟通上，并且触发器并不是万能的，在代码中总有不适合使用触发器的情况，CocoStudio自身的BUG，以及触发器解决不了的问题累积起来，可能会变成一个皮球，在策划人员和程序员之间踢来踢去。

笔者认为，策划人员主要是将需求描述清楚，不需把整个游戏所有的运行逻辑的细节做出来（场景编辑器也难以很好地完成这个工作），如果需求不可行或实现代价比较大，则和程序员进行沟通，打磨需求，使其可行。所以对于触发器工具，定位是更倾向于策划人员想法的验证工具，程序可以给予一定的辅助，策划人员有一些想法需要验证，通过场景编辑器进行验证，而不是先让程序员做出来看看，不好再推翻重做。

除了验证策划人员的想法之外，作为游戏的剧情编辑器来编辑一些剧情场景是一个不错的选择。作为一个“懒惰”的程序员，要想办法从一些简单重复的工作中解放出来。本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　使用场景编辑器。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　组件系统。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　触发器。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　加载场景。

## 28.1　使用场景编辑器

场景编辑器的功能有整合其他资源，如果说UI编辑器负责游戏内的所有UI，那么场景编辑器负责的就是游戏内的所有场景，也就是整个游戏。每个场景有对应的UI和动画，都可以通过组件挂载到场景中。

打开文件菜单，可以发现在新建菜单项中，可以选择创建游戏、UI、动画和场景，之后会弹出如图28-1所示对话框。设置好Cocos2d-x的路径，即可创建一个新的游戏了，在场景编辑器中，双击资源中的UI项目（XXX.xml.ui）和动画项目（XXX.xml.animation），可以快速地打开UI和动画编辑器，双击资源目录中的场景项目（XXX.xml.scene）可以打开场景，如果有多个场景，可以在这里切换要编辑的场景。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160600.jpeg)

图28-1　“新建游戏项目”对话框

在场景编辑器中，可以切换画布尺寸、运行游戏，以及导出低清资源。对场景进行编辑的方法是选择左边工具栏的各种组件，如图28-2所示，拖曳到画布上，也可以拖曳到对象结构中。一个对象可以拥有多个组件，但渲染组件只能拥有一个——精灵、骨骼、UI、地图、粒子都是渲染组件。将组件拖曳到已经存在的对象上，可以将该组件添加到该对象上，规则是同类型的组件不能重复。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160601.jpeg)

图28-2　组件列表和对象结构列表

选中场景中的对象，可以在对象属性面板（如图28-3所示）查看、修改它们的属性，所有的对象都有着常规属性，在这里可以设置对象的基础属性。除了常规属性之外，还可以看到对象身上的各种组件，可以点击在组件面板右上角的X按钮删除组件。通过在资源面板中拖曳的方式可以将图片、UI、骨骼、粒子、音效和数据文件等资源拖曳到对应组件的文件属性中来设置组件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160602.jpeg)

图28-3　对象属性面板

在文件菜单的导出项目→导出资源或按Ctrl + E快捷键可以导出整个项目资源，在指定路径下生成一个Resources目录，目录中分布着UI、动画、图片、场景、粒子和声音等目录，可以替换到程序的Resources目录中使用，如图28-4所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160603.jpeg)

图28-4　“导出项目”对话框

## 28.2　组件系统

组件系统是和节点树紧密结合的一个系统，是将所有的组件都抽象出来，然后让不同的组件实现不同的功能，用很灵活的方式进行组合、复用。每个功能只实现一次，实现之后，所有需要实现该功能的对象直接装上该组件即可。组件和组件之间相互独立，只完成自己需要实现的功能。

在Cocos2d-x中，节点树系统加上组件系统，和Unity引擎的设计非常接近，如图28-5所示，但就工具链和设计的实现而言，二者还有较大的差距。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160604.jpeg)

图28-5　节点的组件系统

Cocos2d-x的组件系统依赖于节点，是节点中的一个子系统，每个节点都有一个组件容器ComponentContainer，ComponentContainer管理着所有的组件Component，而Component记录了拥有该组件的节点Owner。Node上有各种操作组件的方法，它们都直接调用ComponentContainer，如图28-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160605.jpeg)

图28-6　Node、ComponentContainer和Component之间的关系

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　当添加一个Component的时候，Node会将Component添加到ComponentContainer中，然后设置Node为Component的Owner，并调用Component的onEnter。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　当移除一个Component的时候，ComponentContainer也会将其移除，并调用其onExit，然后将其Owner设置为nullptr。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　当Node执行update的时候，ComponentContainer会驱动所有Component的update方法。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　当Node被释放的时候，ComponentContainer会被释放并清空容器。

就目前看Component系统的实现较不严谨，例如，组件的onEnter和onExit可能不会成对出现，当直接移除节点而不手动删除组件时，组件的onExit并不会被调用到。将挂载在A上的组件再挂载到B上，而不先从A移除，那么A和B都会同时拥有这个组件。类似这样的问题还有一些，但相信终将会被解决，Cocos2d-x会向更好的方向前进。

接下来介绍一下Cocos2d-x中的各个组件，在CocoStudio目录下的components目录，如图28-7所示，可以看到如下代码，这里实现了场景编辑器左边组件面板中所有的组件，在这里，粒子、精灵、骨骼、UI和地图等组件被合并为ComRender渲染组件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160606.jpeg)

图28-7　components目录

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　渲染组件ComRender：渲染组件为节点提供了渲染功能，封装了各种可视化节点，将其作为节点的子节点显示。

初始化时根据数据的className属性决定初始化的是哪个渲染组件，并创建对应的显示节点。

在ComRender的onEnter方法中，会将渲染节点添加到节点下，这样节点就拥有了渲染能力。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　声音组件 ComAudio：声音组件为节点提供了操作声音的功能。

初始化时根据数据的className属性判断加载的是音效还是背景音乐，对声音进行预加载，如果是背景音乐，则立刻播放。

ComAudio提供了一些操作声音的接口，如播放、暂停等接口，节点需要操作声音时，只需要获取该组件进行操作即可。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　属性组件ComAttribute：属性组件为节点提供了一个数据容器，也是和数值编辑器沟通的重要桥梁。

初始化时根据指定的.json文件（由数值编辑器导出）以Key-Value的形式解析出来，存储到自身的map中，key为string，Value为cocos2d::Value。

ComAttribute提供了各种操作属性的接口以及存储属性的能力，实现了在编辑器中的自定义属性。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　控制器组件ComController：控制器组件是唯一一个没有在编辑器的组件面板中出现的组件，因为其没什么实际的功能，其功能就是逻辑的封装，在控制器中编写对象的逻辑，例如，玩家根据点击进行移动的逻辑，怪物自动巡逻的逻辑。控制器组件很好地将显示对象以及逻辑分离开了。

控制器组件继承于InputDeleagte，其集合了触摸、重力感应、按键等手机上的输入功能。在onEnter时，它会自作主张地帮Owner节点调用scheduleUpdate。

要使用控制器组件，就要继承一个控制器，如HeroController，然后编写相关逻辑，再挂载到Hero节点身上。

## 28.3　触发器

本节主要介绍触发器的使用、触发器的框架和运作流程以及如何扩展触发器。

### 28.3.1　触发器的使用

触发器的运行流程是如图28-8所示，当事件列表中任意一个事件触发时，执行条件列表的条件，如果所有条件都满足，则依次执行动作列表的所有动作。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160607.jpeg)

图28-8　触发器的运行流程

点击上方工具栏的触发器按钮，会弹出“触发器编辑”窗口，窗口分为3部分，如图28-9所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　触发器列表：管理触发器、新建、删除、改名等。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　逻辑面板：事件、条件、动作的编辑。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　属性面板：编辑当前选中的逻辑的具体属性。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160608.jpeg)

图28-9　“触发器编辑”窗口

触发器的操作流程是，先**新建触发器**，然后在事件下拉列表框中选择事件，可以选择多个事件，条件和动作也同此，选择指定的事件，可以在右侧看到对应的条件和动作参数。如果点击下拉列表框后没有显示事件，有可能是当前项目下的TriggerXML目录出现了异常。

这里有一个比较特殊的就是节点的Tag属性，该属性并不能手动修改，文本框不支持输入，这时需要**将对象结构面板中的对象拖曳到文本框中**，此时渲染节点标签就会更改为所选对象的Tag，所以在这里就无法操作一些动态创建的节点，需要对触发器进行扩展才能做到。其他的属性都可以正常编辑。

当多个动作同时执行时，会按照动作列表面板中的顺序来执行，对同一个属性进行操作的动作（如位置），会被最后一个动作覆盖。

当我们点击生成按钮生成触发器时，会**输出对应的代码文件**到一个Code目录中，如果需要在Cocos2d-x中使用触发器，需要将这部分自动生成的代码复制到项目中。触发器会生成2.x和3.x两个不同目录的代码，如图28-10所示，对应不同的Cocos2d-x引擎版本。而在触发器编辑器中编辑的参数，会保存在场景的JSON文件或者csb文件中，与场景一起导出。有了代码和参数，要让触发器在游戏中生效，还需要在**所有的触发事件处调用sendEvent来触发指定的事件**（看到这里是不是觉得这个触发器太智能了），如果程序员扩展了触发器，那么场景编辑器在为触发器生成代码时也会为程序员手动扩展的触发器生成代码。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160609.jpeg)

图28-10　触发器生成的代码目录

还有一点需要注意，就是有一些事件仅仅触发一次，如进入场景，退出场景等，而场景更新这样的事件每一帧都会触发。使用触发器很容易出现的一个问题是，在场景更新中添加一个延迟条件，当条件满足后执行动作。这时候会不停地重复执行这个动作，如果是播放骨骼动画动作，会造成动画播放卡住的效果。实际上是因为不停地开始播放又不停地被打断导致的。这种问题只能靠扩展触发器的条件和动作来解决。

### 28.3.2　触发器的代码框架以及运作流程

整个触发器的框架由TriggerObj和TriggerMng组成，另外由ObjectFactory辅助创建具体的条件BaseTriggerCondition和动作BaseTriggerAction（如图28-11所示）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160610.jpeg)

图28-11　触发器的框架

TriggerObj是一个触发器对象，包含了一个条件列表和一个动作列表，TriggerObj管理着这两个列表，TriggerMng是触发器的管理器，管理着场景中所有的触发器。

（1）在调用SceneReader::createNodeWithSceneFile的时候，会调用TriggerMng::parse传入触发器的数据，在TriggerMng中根据场景的触发器列表，创建对应的TriggerObj。

（2）在TriggerObj初始化的时候，根据场景文件（.json或.csb）对触发器的配置，初始化条件和动作列表，并在TriggerMng中注册监听对应的事件。

（3）当调用sendEvent触发事件的时候，TriggerMng将触发对应的触发器。

（4）触发器先检测是否所有条件对象的detect方法都返回true，如果没有条件，则继续执行。

（5）接下来触发器顺序执行所有的动作对象的done方法，在具体动作对象的done中执行动作的逻辑。

### 28.3.3　触发器的扩展

触发器的扩展包含编辑配置以及编辑代码两部分，通过编辑配置，可以让触发器的编辑窗口添加新的事件、条件、动作，并且可以生成代码，而具体条件和动作的实现，需要编写C++代码来实现。

当**打开触发器编辑器界面**时，会自动生成当前场景的触发器配置目录——TriggerXML目录，如图28-12所示，在目录下存放所有事件、动作、条件的触发器XML配置文件。在资源面板中刷新一下，可以看到TriggerXML目录（位于ccsprojs/XXXScene目录下）。**如果场景目录下没有这个目录，则触发器编辑器中不会有事件、动作、条件列表给你选择**。当CocoStduio的版本出现冲突的情况下，有可能生成TriggerXML目录失败（我们项目中的TriggerXML一般都是在文档中的CocoStudio/Smaples/Trigger目录下复制过来的）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160611.jpeg)

图28-12　TriggerXML目录

打开TriggerXML文件夹，再打开Event.xml，如图28-13所示，在EventList节点下就是触发器编辑界面中事件下拉框内的事件列表，节点下管理着多个Event节点，在这里添加一个Event节点，下拉列表就增加一项事件。Event节点需要编辑3个属性，**分别是ClassName类名、Name名称、CHName中文名称**，两个名称都用于在列表中显示，但CHName显示在中文环境下，Name显示在英文环境下。CocoStudio会根据类名生成事件ID，可以在这里添加一个事件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160612.jpeg)

图28-13　Event.xml文件

```
<Event ClassName="TestEvent" Name="TestEvent" CHName="测试事件（TestEvent）"/>
```

Condition.xml、Action.xml和Event.xml类似，在ConditionList、ActionList节点下，管理对应的条件和动作节点，属性也和事件一样，但Condition和Action分别对应一个目录，在目录下，以各自对应的ClassName命名的XML文件描述了该条件或动作的参数，该XML编辑出来的内容会在触发器编辑窗口右边的属性面板中出现，如图28-14所示，其XML的编写规则比Event.xml复杂得多。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160613.jpeg)

图28-14　Condition.xml文件

以图28-14中的条件为例，这个条件是“渲染节点是否显示”，触发器编辑器的显示如图28-15所示，有两个参数，一个是NodeTag，另一个是IsVisible，其是一个下拉框。这些选项是由一个个Item组成的，通过添加Item节点来添加想要的属性。每个Item节点都有5个属性，**Type表示控件类型**（下面会详细介绍），**Name表示英文名，CHName表示中文名，key用于在代码中索引到该属性（属性的标识），Default为默认值。**

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160614.jpeg)

图28-15　触发器编辑器

控件类型有以下几种：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　NodeTag节点Tag类型 ，显示为整型数字，接受。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　MultiNode可传入多个节点Tag的控件 。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　NodeCom组件控件，可传入节点身上的组件，这里表现为组件名字。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　TextBox文本框控件，可输入文本。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　DoubleUpDown Double文本框控件，支持上下微调按钮。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　IntegerUpDown整型文本框控件，支持上下微调按钮。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160559.jpeg)　ComboBox单选框控件，支持多选一，后接Childes节点，再接Child节点列表，每个Childe都有Name CHName显示属性，以及ID属性。

这里只增加一个条件，增加动作的方法与前面介绍的一样，不再赘述，这里增加一个已有的条件TimeElapsed，其是一个比较简单的条件，其名字虽然叫TestCondition，和TimeElapsed相比生成的类也不一样，但逻辑、参数都一样，我们添加一个条件项到Condition.xml中，然后在Condition目录下将TimeElapsed.xml复制一份，改名为TestCondition.xml。

```
<Condition ClassName="TestCondition" Name="TestCondition" CHName="测试条件
（TestCondition）"/>
```

这些属性会被导出到场景文件中，在触发器序列化的时候（读取数据），会将这些属性存放到触发器对象内部的变量中，然后在执行时使用它们，此时就完成了编辑器界面的扩展，关闭触发器窗口，再次打开后修改即生效。接下来要实现相关的代码，在Code目录中，打开2.x的代码目录，对里面的代码进行修改。

EventDef.h是一个定义了触发器事件枚举的头文件，修改完XML文件点击生成后可以发现，该文件后面增加了一个枚举，TRIGGEREVENT_TESTEVENT = 8，TESTEVENT是根据事件的ClassName转换为大写生成的。

cons.h和cons.cpp是条件对象的头文件和源文件，在头文件和源文件尾部自动生成了一些代码，根据配置文件自动生成了一个类。

```
class TestCondition : public cocos2d::extension::BaseTriggerCondition
{
    DECLARE_CLASS_INFO
public:
     TestCondition(void);
    virtual ～TestCondition(void);
    virtual bool init();
     virtual bool check();
    virtual void serialize(const rapidJSON::Value &val);
    virtual void removeAll();
};
```

在serialize方法中，获取属性数据，在check方法中实现条件判断逻辑，如果我们扩展了动作对象，会被生成到act.h和act.cpp中，在动作对象生成的类中，需要重写done方法来实现动作的逻辑。这里直接复制TimeElapsed 在头文件和源文件的代码，然后覆盖TestCondition类，并将类名调整回TestCondition类完成代码扩展。

在这里简单介绍一下serialize，以TimeElapsed的序列化为例，在传入的属性列表中，通过查询的方式，定位到指定的key，然后再取出指定的Value，并保存到自身的成员变量中。

```
void TimeElapsed::serialize(const rapidJSON::Value &val)
{
    int count = DICTOOL->getArrayCount_JSON(val, "dataitems");
    for (int i = 0; i < count; ++i)
    {
    		   const rapidJSON::Value &subDict = DICTOOL-> getSubDictionary_
 		   JSON(val, "dataitems", i);
         std::string key = DICTOOL->getStringValue_JSON(subDict, "key");
         if (key == "TotalTime")
         {
              _fTotalTime = DICTOOL->getFloatValue_JSON(subDict, "value");
         }
    }
}
```

扩展完之后，**直接使用扩展的触发器条件或动作会直接崩溃**（创建触发器后运行），因为**模拟器并没有被重新编译**，在创建触发器的时候会崩溃，那么要怎样才能在编辑器中看到新增触发器的效果呢？

按照官方说法，需要重新编译模拟器，而模拟器的代码是开源的，放在github.com上，网址为https://github.com/chukong/CocoStudioConnector。

名字叫做CocoStudio Connector，而不是CocoStudio Simulator，下载代码然后解压，解压后的Classes子目录如图28-16所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160615.jpeg)

图28-16　CocoStudio Connector的Classes目录

这份代码本身也是一个Cocos2d-x工程（一个复杂点的HelloCpp），git上提示使用2.2.2版本的引擎编译（设置头文件路径、库路径等过程略过），但笔者尝试之后编译失败了，最后使用了更早期版本的Cocos2d-x引擎才编译通过（应该是2.2.1）。如果使用其他版本的引擎，与CocoStudio可能会不兼容，这时需要将触发器的代码替换到trigger目录下，然后生成exe，然后**在CocoStudio中重新设置模拟器为我们编译通过的模拟器**，这样就可以在CocoStudio中看到扩展的触发器效果了。

在编写触发器条件或动作的代码时，可以将模拟器设置路径中的命令行参数，如图28-17所示设置到VS项目的命令行参数中，这样可以方便地在VS中直接运行模拟器，并使用编辑的场景进行测试，当然，场景中编辑的触发器也是会执行、可被调试的。这时候可以在场景编辑器中编辑新添加的条件和动作，保存后在VS中直接运行调试效果。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160616.jpeg)

图28-17　设置模拟器路径

设置图28-17中的命令行参数到“项目属性”→“调试”→“命令参数”中，如图28-18所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160617.jpeg)

图28-18　VS中的属性页

自定义触发器的条件和动作逻辑问题，以及编译问题这里不再多说，因为trigger里其他的条件和动作都可以参考，按照这些例子进行编码即可。而编译问题可以尝试其他版本的Cocos2d-x，也可以自己解决编译错误。这里需要注意的是，不要用我们生成的模拟器替换CocoStudio目录下的模拟器，而是要设置模拟器路径。因为我们生成的模拟器关联到CocoStudio目录下的DLL版本问题，所以这里需要**将模拟器的exe以及所需的DLL一起打包放到一个目录中**，并指定模拟器的路径。这里需要注意的另外一个问题就是，模拟器的路径不要有中文路径。

## 28.4　加载场景

在代码中加载场景，使用SceneReader的createNodeWithSceneFile方法，传入场景编辑器生成的.csb或.json文件，返回一个Node，将Node添加到场景中。SceneReader会保存最近一个加载场景的引用，通过getNodeByTag方法可以递归查找该场景下指定Tag的一个节点。

使用SceneReader需要注意的问题是，在同一个场景中多次加载场景需要慎重，在场景切换的时候，需要调用destroyInstance来清理资源（声音、Reader本身，TriggerMng等）。

如果使用了触发器，还需要将触发器的代码包含到项目中，再包含触发器生成的EventDef.h，在**对应的触发时机处编写代码、调用sendEvent触发具体事件、进入场景、离开场景、点击、场景更新等事件**，都要手动触发才行，当然，如果触发器没有监听这些事件，也可以只触发所关心的事件（这里可以参考Simulator的HelloWorldScene）。

编写代码时，不要忘记引入命名空间，而旧版本的引擎还需要添加链接库才行。当然，更不能忘了将场景编辑器生成的资源文件复制到游戏的Resources目录下。

```
#include "cocostudio/CocoStudio.h"
#include "TriggerCode/EventDef.h"
using namespace cocostudio;
void TriggerTest::onEnter()
{
    //加载场景文件，生成场景节点，并添加为当前的子节点
    _filePath = "scenetest/TriggerTest/TriggerTest.JSON";
    _rootNode = SceneReader::getInstance()->createNodeWithSceneFile (_file
    Path.c_str());
    addChild(_rootNode);
    //发送进入场景消息
    sendEvent(TRIGGEREVENT_ENTERSCENE);
}
void TriggerTest::onExit()
{
    //发送离开场景消息
    sendEvent(TRIGGEREVENT_LEAVESCENE);
    SceneReader::destroyInstance();
    SceneEditorTestLayer::onExit();
}
```