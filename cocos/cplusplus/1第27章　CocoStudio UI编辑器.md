# 第27章　CocoStudio UI编辑器

本章主要介绍CocoStudio UI编辑器的使用、资源导出，在代码中加载并使用编辑器导出的UI文件、UI的数据更新、UI编辑器的适配问题以及如何扩展CocoStudio UI编辑器及自定义控件。本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　使用UI编辑器。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　编写代码。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　分辨率适配。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　扩展UI编辑器。

## 27.1　使用UI编辑器

首先映入眼帘的是UI编辑器最上方的菜单栏，接下来是工具栏，工具栏中可以切换普通模式和动画模式，也可以调整画布的大小，剩下的按钮都是对控件进行对齐、布局、排版操作的选项（如图27-1所示）。**UI编辑器存在两种模式，即普通模式和动画模式，普通模式用于创建、调整控件；动画模式用于编辑UI动画**。默认会进入普通模式。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160508.jpeg)

图27-1　菜单栏和工具栏

创建新项目时，UI编辑器会创建一个画布，画布是我们操作的舞台，本质上是一个节点。CocoStudioUI编辑器支持多个画布，在画布列表面板中可以选择、新建、删除、复制、重命名画布，如图27-2所示。每个画布就是一个界面，而画布列表管理着游戏中的所有界面，这些界面共享着资源面板中Resources目录下的资源。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160509.jpeg)

图27-2　画布列表

UI编辑器最常用的操作就是**拖放控件**，这些控件在GUI框架系列章节有详细介绍，而在这里要做的就是把鼠标移动到控件上按下，然后把控件拖放到画布中，然后再设置它们的位置。可以通过一些比较麻烦的方法来扩展UI编辑器，扩展一些控件，如图27-3所示中最下方的CSCustomImageView控件（这个在后面介绍）。

当我们创建一个新的画布，或者新建一个项目，打开其默认创建的画布时，在“对象结构”面板中，可以看到默认创建了一个Panel_XXX节点，作为一个默认的容器层，并且无法删除，所有的控件都被添加到这个Panel之下，并且必须在该Panel之下（可以是孙节点，曾孙节点...），在对象结构面板中，可以组织节点的父子关系、添加新的控件、改名等。PageView在这里有一个特殊的功能添加子页面，如图27-4所示，该选项会自动添加一个容器层作为PageView下的一个Page。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160510.jpeg)

图27-3　控件列表

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160511.jpeg)

图27-4　“对象结构”面板

**拖放控件和调整控件位置是在普通模式下做的事情，而动画模式下可以编辑UI动画**。首先将编辑器切换至动画模式（单击工具栏的模式按钮），下方会出现“动作列表”和“关键帧”面板，可以先在“动作列表”中右击添加新的动画，然后选中需要编辑的控件右击，编辑动画，如图27-5所示。这时在关键帧面板中才会出现可编辑的动画对象。如果当前的动作列表为空，则会自动创建一个Animation0的动画，然后创建动画对象，如图27-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160512.jpeg)

图27-5　右键编辑窗口

接下来就可以编辑动画了。在“动作列表”面板中管理所有动画，在“关键帧”面板中添加帧，如图27-6所示，然后修改对象的属性，如位置、缩放、旋转和透明度等。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160513.jpeg)

图27-6　“动作列表”和“关键帧”面板

选中对象，在“属性”面板中可以查看和修改控件的属性，如图27-7所示，如果没有该面板，可以在最上方的窗口菜单中找到并打开。在动画编辑模式下，只有“常规”和“控件布局”中的部分属性能够参与动画，其他属性无法在动画中动态改变，这些属性分别是透明度、颜色混合、坐标、缩放、旋转。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160514.jpeg)

图27-7　属性面板

目前UI编辑器的动画暂时没有帧动画的功能。在编辑UI上的图片资源时，使用一贯的拖曳操作，将图片从资源目录下拖放到控件上。选择Custom模式可以调整控件的尺寸，以及设置九宫。

当选择Custom模式之后，会出现九宫格选项，选中该选项，然后编辑原点和尺寸就可以拉出九宫了，**X和Y表示要拉的原点，可理解为左下角的宽和高**，W和H表示中间区域的大小，这里的单位都是基于未拉伸的原图的像素大小，**W=原图的W-2×X，H = 原图的H-2×Y**。设置完之后切记**不能缩放图片，需要通过调整最上方的尺寸来调整大小**。

编辑完UI之后，可以导出UI，按Ctrl+E快捷键或者在文件菜单下选择“导出项目”选项，可以看到如图27-8所示界面，可以选择导出多个或者当前画布，还可以选择导出大图或小图，**大图指按照规则导出的Plist+PNG图集，小图则是每个资源都导出单份**。也可以选中“导出二进制数据文件”复选框，而不使用ExportJSON。选择导出大图之后，可以设置导出格式、排序方法、宽高限制等参数。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160515.jpeg)

图27-8　“导出项目”对话框

资源导出的规则在这里需要详细介绍一下，如图27-9所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160516.jpeg)

图27-9　导出资源规则

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　如果导出使用大图，编辑器会根据当前导出的画布，自动将所有单张的图片合并到图集中，如果图片太多，会创建多个图集。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　不管是导出使用大图还是小图，本身已经是Plist图集的资源，不会再被合并，仍然是以整个图集的方式被导出。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　多个画布中有重叠资源时，每个画布单独导出使用大图会导致资源冗余，应该单独导出使用小图，或者导出全部画布。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　尽量自己来规划图集，打包Plist，而不要过分依赖编辑器的自动合并图集功能。

## 27.2　编写代码

整个UI系统分为两大类UI，一类是控件，如按钮、图片、滚动条、文本框等，另一类是控件容器，如ScrollView、PageView、Layout等，专门用来存放控件。

其他书籍中会对这两类UI进行剖析，这里仅介绍和CocoStudio的对接。在开始编码之前，如果读者使用的版本比较旧，可能需要设置一下头文件的搜索路径，引入命名空间和指定链接库，因为有些版本的GUI是作为一个动态链接库分离出来的。

### 27.2.1　加载UI

我们使用GUIReader这个单例来加载UI，通过widgetFromJSONFile函数读取一个CocosGUI导出的ExportJSON文件，解析成为一个Widget返回，一般在进入某个场景，或者弹出某对话框的时候，创建一个Widget并添加到场景中。

在获取到Widget之后，需要手动绑定一些按钮的回调函数，可以用UIHelper提供的接口，来定位对应的按钮，然后调用按钮的addTouchEventListener或addClickEventListener方法来设置回调（对于一些特殊的UI，需要进行特殊的设置，如PageView和ScrollView）。

```
Widget* widget = GUIReader::getInstance()->widget FromJSONFile ("TestUI. 
ExportJSON");
addChild(widget);
Button* button= dynamic_cast<Button*>(Helper::seekWidgetByName(widget, 
"Button1"));
button->addTouchEventListener(CC_CALLBACK_2(UIButtonTest::touchEvent, this));
```

在回调函数中判断事件，实现对应的逻辑，回调传入点击事件触发的对象，以及是何种点击事件。

```
void UIButtonTest::touchEvent(Ref *pSender, Widget::TouchEventType type)
{
    switch (type)
    {
        case Widget::TouchEventType::BEGAN:
            break;
        case Widget::TouchEventType::MOVED:
            break;
        case Widget::TouchEventType::ENDED:
            break;
        case Widget::TouchEventType::CANCELED:
            break;
        default:
            break;
    }
}
```

### 27.2.2　UI动画

创建完UI之后，如果这个UI存在UI动画（在编辑器中编辑了UI动画），在加载UI文件时（GUIReader加载），UI动画会被添加到**ActionManagerEx**单例进行管理，可以获取**ActionManagerEx**的单例来播放UI动画，UI动画是以编辑器导出的文件名和动画名称为key，文件名包含后缀，所以传入的文件名需要是MyUI.csb或者MyUI.ExportJSON，并且不需要加目录前缀。

```
//播放UI动画，传入UI文件名和要播放的动画名
ActionObject* playActionByName(const char* JSONName,const char* 
actionName);
//播放UI动画，传入UI文件名和要播放的动画名，并指定播放完成后的回调
ActionObject* playActionByName(const char* JSONName,const char* actionName, 
cocos2d::CallFunc* func);
//释放所有UI动画对象
void releaseActions();
```

### 27.2.3　初始化UI

创建完UI之后，还需要对UI进行初始化，我们绑定了UI的回调函数，有时还需要将UI进行数据绑定。例如，一个玩家信息面板UI里可能有金币、等级、血量、攻击力等数据，在编辑阶段，程序员肯定是不知道用户的详细数据，这些数据往往需要从玩家的存档中取出，然后设置到UI中。

在创建该界面的时候，需要使用数据来填充界面（向玩家信息界面设置各种数据），**这时初始化的代码需要写在界面创建的地方**，这种做法称之为手动初始化。

如果是使用CocosBuilder的话，可以将一个UI绑定到一个类，这样很多操作都非常方便，**这时候初始化的代码是写在该UI所绑定的类中**，当创建这个UI时，UI会自己去取相应的数据初始化，这种属于自动初始化，是一劳永逸的做法。

这两种做法的区别是，当需要在另外一个地方打开同一个UI时，使用手动初始化的方法需要将初始化的代码复制一份，粘贴到需要打开UI的地方。而使用自动初始化的方法，只需要在另外的地方同样创建一个类，因为初始化的代码已经被封装到类中了。**当将代码进行第一次复制的时候，就应该考虑如何封装、复用代码，以消灭代码复制。**

```
//一些伪代码来介绍一下初始化
Widget* myUI = GUIReader::getInstance()->widgetFromJSONFile("XXX.UI");
Text* hpUI = ui::Helper::seekWidgetByName(myUI , "HPTEXT");
Text* mpUI = ui::Helper::seekWidgetByName(myUI , "MPTEXT");
hpUI->setText(data->getHpString());
mpUI->setText(data->getMpString());
```

CocoStudio也提供了类似的功能——扩展UI，通过添加新的UI控件来实现，但这种做法并不是很友好，新UI控件应该是以实现某种通用功能为目标，如WebView，而不是实现一个特定的功能，如初始化用户信息。这种做法的代价很大，需要在插件工程、CocoStudio以及项目中来回地“折腾”，代价远大于创建完成之后手动写代码。如果是功能型的控件，如单选框，作为一个扩展UI是不错的，但目前的扩展UI机制还很不成熟，功能不够强大（**例如，自定义控件不允许添加其他子控件这种规则性的需求是无法满足的**）。

优雅一些的做法是，创建一个新的类，继承于ControllerComponent，然后将这个类作为UI的组件添加到UI身上，把UI对应的逻辑转移到控制器组件身上，对UI数据进行填充、更新、都由该控制器来实现，而其他地方则不需要清楚地了解，如这个UI的内部结构是怎样一个树状组织，哪个控件叫什么名字，这些细节都被封装到控制器组件中。当有类似的UI时，还可以用同一个控制器组件来初始化其他UI，也可以用继承、封装等手段来复用这个控制器组件。

```
 //优雅做法的伪代码
//将上面的初始化代码封装到MyUIInitComponent的onEnter中
//只需要维护好MyUIInitComponent即可
Widget* myUI = GUIReader::getInstance()->widgetFromJSONFile("XXX.UI");
myUI->addComponent(MyUIInitComponent::create());
```

一般在UI初始化时，主要初始化的内容有：为一些Text设置文本，为一些ImageView之类的控件设置图片，或者显示/隐藏某些控件，这些需求可以进行一些简单的封装来简化的代码。例如设置文本，可以实现一个方法，传入一个Map对象，key和Value都是string对象，方法自动根据key集合seekWidgetByName来定位Text对象，并设置其文本内容为字符串的值，在方法内完成自动批量地初始化。

例如，对怪物卡片的信息有大量需求，这些可以由策划人员配置，然后程序读取到一个Map里，根据当前要查看的卡片来决定读取哪个配置文件，如果存在需要程序动态计算生成的文本内容，那么程序只需要修改动态生成的那一部分即可。这样的好处是，CocoStudio生成的内容直接和策划配表挂钩，如果调整的内容不需要程序动态计算，则不需要程序参与。

### 27.2.4　更新UI

UI的数据更新也是在游戏中经常碰到的，如玩家的分数、血量、金币、时间等，都是需要经常改变的UI，数据更新一般有3种方法，即手动更新、自动更新以及被动更新。

手动更新在初级程序员里面用的最多，因为最简单、直接，耦合性高。Cocos2d-x的节点树为手动操作提供了很大的方便，这种方式的操作一般有以下几个步骤，在数据发生改变的地方执行：查找节点（通过一长串的节点获取父节点，查找子节点的方法先定位到节点），强转类型（将节点强制转换为Label之类的节点），修改数据。既然使用引擎可以查找到节点，那么就一定有大量的人使用它。

手动更新存在的问题很明显，如果两个节点是复杂的“亲戚”关系，首先需要花一段时间在找亲戚上，这中间可能出现找错亲戚的情况，然后是多个地方调用的时候，需要在很多地方写类似的代码（如有很多个地方都有对玩家进行扣血的代码，扣完血后手动更新一下玩家的血量），当整个树状结构发生改变的时候，如中间插入或删除了一些节点，那么就会出现问题。当这个亲戚本身发生变化，如使用了另一种节点来表现（TTF改为了BMFont），那么同样会出现问题，这时会给代码的维护带来严峻的问题。如无法通过查找引用来找出所有修改的地方，一旦遗漏，就会隐藏一个崩溃的BUG。

```
//手动更新的伪代码
void XXX::BeAttack(Object attacker)
{
    //扣血
    m_HP -= attacker->getAttack();
((Text*)getParent()->getParent()->getChildByTag(10086))->setText(m_HP->t
oString());
}
```

那么手动更新就是一个坏方法吗？方法没有好坏，**只有合适不合适！当两个节点是父子节点关系，且调用的代码不多时，直接使用这种方法是合适的，**将操作代码封装到控制器组件中，只操作控制器组件，也是一个好主意，至少进行了一些封装，并且便于定位调用代码。作为一个有经验的程序员，应该在一开始就能预测到一些即将出现的变化，并选择适当的方法。当变化已经出现，而目前的方法存在比较大的隐患、耦合时，应该立即进行一次小的重构，用更合适的方法，使整个程序往更好的方向前进。

主动更新的耦合度是非常低的，因为把数据的更新完全掌握在自己手中，只需能够获取到数据即可。如时间的更新，可以是在控件内或控件的控制器组件中编写代码，在update中更新时间。例如玩家的血量，只要可以访问到玩家的数据（一般会作为全局数据供访问），可以在每次update检测数值是否改变，如果改变则主动刷新UI，这种方法的缺点是每帧刷新效率比较低，当出现大量的主动更新时，可能对性能造成影响。虽然以目前的硬件效率来讲，这种影响几乎可以忽略，另外当需要获取的数据无法获取到时，主动更新就不可取了。当需要更新的是少量对象而不是大量对象，并且要获取的数据有良好的接口可以获取到，触发数据修改的地方变数大时，使用主动更新是合适的。

```
//主动更新的伪代码
void XXUI::update(float dt)
{
    if(m_HP != data->Hp)
    {
         m_HP = data->HP;
          getChildByTag(10086))->setText(m_HP->toString());
     }
}
```

最后是被动更新，被动更新其实就是监听者模式，在这里可以使用EventDispathcer——EventListener，加上CustomEvent来实现被动更新，在创建UI的同时，向EventDispathcer中注册一个EventListener，在回调函数中编写刷新显示的代码，然后在需要刷新显示的地方，调用EventDispathcer发出对应的消息。被动更新的优势是耦合度最低，因为显示的逻辑和触发的逻辑通过消息机制分离开了，两者的变化互不影响对方，而且数据可以通过消息参数传递，也没有主动更新有可能获取不到数据的限制触发才执行逻辑，效率也高，在很多情况下使用都是合适的。那么被动更新、监听者模式就是最好的吗？**方法没有好坏，只有合适不合适！**首先在监听者模式下需要编写注册、注销、回调、触发4处代码，是一个稍微偏重的武器。在有些情况下手动更新会更好，如类内部的通信，可以直接调用方法，并且该功能内聚性很强，不会有外部需要触发，那么用消息机制就是“杀鸡用牛刀”了，一行代码就可以完成。如在消息触发的时候，监听者还没有被创建出来，那么这种情况下，监听者的主动更新会更合适一些。

```
//被动更新的伪代码，触发消息
void XXX::BeAttack(Object attacker)
{
    //扣血
    m_HP -= attacker->getAttack();
    Hpdata->Hp = m_HP;
    m_EventDispatcher.dispatchCustomEvent("playerBeAttack", Hpdata);
}
//被动更新，消息回调
void XXUI::refleshHpUI(EventCustom* data)
{
    HpData* pHp = (HpData*)data;
    getChildByTag(10086))->setText(pHp->Hp ->toString());
}
```

上面是对控件初始化和数据更新的一些看法。在使用GUI的时候，还会有这样的需求——动态生成控件，如背包UI，当打开背包的时候，背包里面的每一个部件都是动态创建出来的，可以用CocosGUI的扩展控件来实现一个背包控件，但前面说了这种方法不推荐，因为不是可以共用的UI。这时需要有一段代码来进行初始化，这段代码可以写在背包的控制器组件中，然后逐个地添加这些小控件（物品）。而这些小控件也可以由一些文字、图片组成，这种情况下需要在代码里面创建它们，编写坐标，将它们打包成一个Node，然后再添加到容器中。这种做法确实不是很好，如果碰到一些需要动态创建且内部结构复杂的程序，就会很麻烦了，并且写坐标的工作最好不要放在代码里面。

那么可以这样做，在CocoStudio的UI编辑器中，编辑好单个的控件然后导出，在需要批量添加到容器的地方，将控件加载进来，然后定位到具体的这个控件，因为UI编辑器默认会添加一个Panel，所以需要跳过它。将该控件复制然后重新初始化（对应的图片，文字），再添加到容器中。如果修改UI编辑器导出的.json文件，将Panel去掉应该可以直接得到这个控件，但后面调整时容易忘记去修改.json文件，所以还是建议在代码里手动定位。

## 27.3　分辨率适配

UI的分辨率适配非常重要，CocoStudio的UI编辑器提供了完美的适配方案，如图27-10所示，但也要使用得当才能完美适配，否则就会出现不适配的问题。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160517.jpeg)

图27-10　UI编辑器中的布局效果

如图27-11所示是在程序中加载这套UI的效果，可以看到整个UI的布局位置都发生了错位（相对于图27-10）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160518.jpeg)

图27-11　实际运行的布局效果

上面是CocoStudio UI编辑器的一个例子，以非自适应分辨率导出，加载时之所以会错位，主要是因为**AppDelegate**调用了**Director**的**setContentScaleFactor**方法（注释掉即可），将所有对象的位置进行了缩放而不缩放资源，如果在测试骨骼动画的时候发现骨骼动画散开了，那么也是这个原因。

CocoStudio的容器提供了4种控件布局方式，**分别是绝对布局、相对布局、线性横向、线性纵向布局**。绝对布局是以左下角为原点，不论分辨率如何变化，都按照绝对坐标的位置排列，例如图27-12所示，当分辨率发生变化时，绝对布局的控件相对左下角的位置不变。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160519.jpeg)

图27-12　绝对布局

相对布局是基于父节点或兄弟节点，以上下左右、左上、左下、右上、右下八个方向为停靠点的相对布局。选择了相对布局之后，选中子UI，可以发现在属性面板的控件布局选项中增加了一些选项（停靠、横向布局、纵向布局、边缘），首先可以选择要停靠的节点，然后选择停靠点，选择完停靠点之后，如停靠左上角，那么边缘属性的左和上生效，右和下被屏蔽，根据设置的属性会基于停靠节点的停靠点产生偏移，并且在分辨率变化的时候保持相对停靠，如图27-13和图27-14所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160520.jpeg)

图27-13　相对布局

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160521.jpeg)

图27-14　CocoStudio的相对布局设置

线性横向布局和线性纵向布局，相当于相对布局的简约版，横向布局去掉了上中下的相对设置功能，纵向布局去掉了左中右的相对设置功能，让布局只关注纵向或者横向的变化。

当导出UI的时候，如果选中了容器层的自适应分辨率，意味着Panel的大小会跟着窗口的大小变化，里面的UI同时会跟着变化，如果没有选中，则Panel会是一个固定不变的尺寸。需要注意的是在**Cocos2d-x**中存在两个大小，一个是**WinSize**，可以通过**Director**获取，是设置的设计分辨率；另一个是**FrameSize**，通过**GLView**获取，是设备的实际分辨率。自适应分辨率下的Panel跟随WinSize变化，而不是FrameSize。

FrameSize的作用更多是辅助进行分辨率适配，帮助计算出不同比例下的WinSize应该是多少，如果WinSize是一个固定值，那么会导致黑边或者拉伸。

WinSize是代码中所依赖的尺寸，在进行位置或大小计算的时候，可以放心地使用它，根据设置的分辨率策略，可能会出现黑边、拉伸或者裁剪，但基于WinSize的位置计算可以保证代码基本正确运行。

## 27.4　扩展UI编辑器

UI编辑器的扩展就目前而言不算成熟好用，并且目前只能基于Coco2d-x 2.x版本开发，这里简单介绍一下，想深入了解的读者可以看关于扩展编辑器的官方文档（当前的版本不建议使用），地址是https://github.com/chukong/cocos-docs/blob/master/manual/studio/ui-Widget-Expansion/zh.md。

UI编辑器的扩展可以简单划分为3个流程：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　编写插件——实现该控件以及该控件的Reader，并封装，实现CocoStudio编辑器约定的接口，编辑Swig脚本，最后生成Dll。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　使用插件——把Dll放到CocoStudio的插件目录下（CocoStudio版本），在编辑状态下使用插件，然后导出。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160507.jpeg)　代码调用——将控件和控件Reader的代码放到Cocos2d-x项目中，调用GUIReader的registerTypeAndCallBack注册控件，然后加载使用。