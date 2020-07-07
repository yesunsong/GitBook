# 第**22**章　**Quick**框架实践——**MVC**框架

MVC是一种常用的框架模式，MVC将程序划分为Model模型、View视图、Controller控制器三层，可以降低代码的耦合性，使代码易于维护，方便管理。

模型层负责数据逻辑处理，不论数据是从内存、数据库、网络、配置等方式获得，视图和控制器都无须关注。显示层负责显示逻辑，一份数据以饼状图还是树状图呈现给用户，取决于视图的实现。控制层负责处理与用户的交互逻辑，以及对模型和视图的控制，是衔接模型和视图的桥梁。

使用MVC有助于管理复杂的应用程序，我们可以专注在某个方面的开发，甚至可以由多个人来实现某个功能，MVC也使程序的测试变得更简单，因为模块间相对独立，方便单元测试。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　组件系统详解。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　ModelBase详解。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　MVC示例详解。

### 22.1　组件系统详解

首先需要了解一下Quick的组件系统，因为Quick有很多功能都是基于这套组件系统实现的，包括本章要介绍的MVC框架。

组件模式是非常不错的一个设计模式，可以将各种各样的功能封装为组件，然后组装到目标对象身上，非常灵活和方便。因为这些功能可能被各种各样的对象使用，而这些对象之间可能毫无关联，那么为了让这些对象都具有这样的功能，需要为每个对象实现一个这样的功能，这样就避免不了重复的代码。如果使用继承的方式，可以使它们共用一个功能，但显然并不是很合适，而组件模式则可以很好地实现它。

**组件系统的核心包含GameObject和Component两个类**，GameObject类在前面已经介绍过了，主要实现让对象拥有管理组件的能力，成为组件的载体。经过GameObject扩展过的对象，可以使用addComponent()、removeComponent()、getComponent()等方法操作组件。

Component类的实现很简单，在构造函数中初始化了组件的名字以及关联组件，名字主要用于区别组件，相当于组件的ID，而关联组件则是该组件所依赖的一些组件，当组件被添加到对象上时，会检查对象是否已经挂载了关联组件，如果没有则先添加关联组件。

Component类的核心功能是为**对象添加扩展方法**，通过exportMethods_方法将组件的方法导出到对象上，首先会判断对象是否拥有对应的字段，如果没有则为对象添加对应的函数。例如，通过exportMethods_()方法导出fun1()函数（一个具体组件的成员函数）到一个对象中，如果该对象存在fun1变量，则跳过，否则将该对象的fun1变量赋值为对应的函数。此外，Component类还提供了组件被添加和移除时的回调供程序员重载，分别是onBind_()和onUnbind()函数。

调用GameObject.extend()方法或cc()方法，传入一个对象，该对象即可获得管理组件的能力，在对象的addComponent()方法中，会从Registry单例中创建组件对象（所以使用组件之前要先在Registry中注册组件），将组件添加到components容器中进行管理，并回调组件的bind_()方法。移除时会相应地回调组件的unbind_()方法。

```
function target:addComponent(name)
    local component = Registry.newObject(name)
    self.components_[name] = component
    component:bind_(self)
    return component
end
```

注册组件到Registry的方法非常简单，在程序启动时执行Registry.add()方法，传入对应的类（使用require或import导入的table）以及类名即可。

#### 22.1.1　EventProtocol事件组件

事件组件是Quick中最基础的一个组件，用于实现一个简单的消息机制，该组件导出了以下方法。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　addEventListener()：注册监听回调，参数为监听的事件名、回调函数、tag标签，该函数会返回一个监听者的唯一ID。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　dispatchEvent()：触发事件，参数为事件名字符串。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　removeEventListener()：移除指定的监听者，参数为监听者的唯一ID（由addEventListener返回）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　removeEventListenersByTag()：根据Tag移除指定的监听者（所有的），参数为tag变量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　removeEventListenersByEvent()：移除监听指定事件的监听者，参数为事件名字符串。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　removeAllEventListenersForEvent()：移除监听指定事件的所有监听者，参数为事件名字符串。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　removeAllEventListeners()：移除所有的监听者。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　hasEventListener()：查询是否有监听者监听指定的事件，参数为事件名字符串。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　dumpAllEventListeners()：打印所有的监听者的详细信息，主要用于调试。

在EventProtocol的内部使用了一个listeners_变量来存储所有的监听者，listeners_变量是一个table，其结构如图22-1所示，它的key是事件名，value是一个监听者列表table，管理了多个监听者。监听者列表table的key为监听者的唯一ID，value为监听者的信息table，记录了监听者相关的信息，下标1对应监听者的回调，下标2则对应监听者的tag，tag变量默认为一个空的字符串，如果将一个对象作为tag传入，tag会被处理为空字符串，并将tag对象与回调函数用handler封装成闭包。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121349.jpeg)

图22-1　listeners_结构图

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121350.jpeg)**注意：**EventProtocol会将所有的事件名都转换成大写，所以监听一个open事件和监听一个OPEN事件实际上是同一个事件。

#### 22.1.2　StateMachine状态机组件

状态机组件是从github上的一个JavasCript版本移植而来的，状态机也是游戏中常用的一个功能，主要用于控制状态的切换。该组件导出了以下方法。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　setupState()：初始化状态，参数为一个复杂的配置table。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　isReady()：简单地判断当前状态是否为none。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　getState()：获取当前的状态。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　isState()：判断当前是否为某状态，参数为对应的状态名。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　canDoEvent()：判断当前状态是否可以切换到指定的状态，参数为对应的状态名。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　cannotDoEvent()：返回canDoEvent方法的相反值，参数为对应的状态名。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　isFinishedState()：判断当前状态是否为结束状态。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　doEventForce()：强制切换至某状态，参数为对应的状态名。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　doEvent()：切换至某状态，参数为对应的状态名。

状态机提供的功能非常简单，要灵活使用这个组件，需要掌握两个关键点，**初始化配置table和状态切换的相关回调**。初始化配置table构建了状态机所有的事件和状态，这个table描述了整个状态机的所有状态切换规则，有两个重要的概念，就是事件和状态，通过执行事件可以导致状态的切换。而状态机的回调机制能够让程序员通过设置回调来执行状态切换流程中的逻辑。

在setupState()方法中，需要传入一个复杂的配置table，该table的详细结构如下。

initial字段，该字段对应了状态机的初始信息，值可为table，也可以是一个字符串，table的格式如下，state为初始化状态，event为初始化默认执行的事件，defer为布尔值，用于判断是否推迟初始化。当initial字段为字符串时，StateMachine会自动将其转换成table，将字符串设置为state字段，event字段为"startup"，defer自动默认为nil（在判断时，nil等同于false）。

```
{ state = "foo", event = "setup", defer = true|false }
```

当defer为nil或false时，setupState会在完成初始化之后，自动切换到initial字段对应的初始状态，默认的状态为none。

terminal或final字段，该字段为字符串，意为状态机的结束状态，主要用于isFinishedState()函数的判断。

events字段，该字段为一个event数组，对应了状态机可以执行的事件列表，每个事件都是一个table，事件table包含3个字段：name字段为事件的名字，from字段为可以执行该事件的状态，可以为字符串或table（当有多个状态可以执行该事件时），to字段表示执行该事件后会切换的状态。该结构的示例代码如下：

```
cfg.events = {{name = "disable", from = {"normal", "pressed"}, to =
"disabled"},
 {name = "enable", from = {"disabled"}, to = "normal"},
 {name = "press", from = "normal",  to = "pressed"},
 {name = "release", from = "pressed", to = "normal"},}
```

状态机本身并没有一个状态列表或状态枚举，所有的状态都是通过events组织起来的，events相当于一个状态机的核心框架，所有事件的from和to字段将所有的状态串起来，状态机通过执行各种事件实现在各个状态间的切换。

callbacks字段，该字段对应一个回调table，在状态机执行各种事件时，会有相应的回调被触发，callbacks中记录了相应的事件被触发时的回调，通过状态机的回调规则命名。详细的回调规则如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onbeforeevent()方法：在执行任何事件前会执行该回调，并传入执行的事件名，如果该函数返回false，则该事件执行失败。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onafterevent()方法或onevent()方法：当任意事件被执行时会执行该回调，并传入执行的事件名。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onleavestate()方法：当状态机从任意状态离开时会执行该回调，并传入执行的事件名，如果该函数返回false，则该事件执行失败。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onenterstate()方法或onstate()方法：当状态机进入任意状态时会执行该回调。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onbeforeXXX()方法：在执行XXX事件前会执行该回调，如果该函数返回false，则该事件执行失败。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onafterXXX()方法：在XXX事件被执行时会执行该回调。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onleaveXXX()方法：当状态机从XXX状态离开时会执行该回调，如果该函数返回false，则该事件执行失败。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onenterXXX()方法：当状态机进入XXX状态时会执行该回调。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　onchangestate ()方法：当状态机发生状态切换时会执行该回调。

状态机的回调可以有几种分类的方法，按时机来分可以划分为状态切换前和切换后，按功能来分可以划分为普通回调和询问回调（可以控制状态切换流程），按目标来分可以划分为通用回调和特定回调（针对特定的事件或状态）。

这几种分类基本涵盖了所有的需求，询问回调可以通过返回false来拒绝本次状态的切换，onbefore()函数返回字符串asyn，还可以进入异步切换状态的逻辑。异步切换指的是从状态A切换到状态B需要一定的时间，不是立即切换完成的，状态机会为异步切换的事件创建两个回调函数成员，分别是transition()和cancel()，因为什么时候完成切换，是由具体的逻辑来决定的，当完成切换时，调用event.transition()可以完成切换，而调用event.cancel()则可以取消这次切换。在异步切换时，是无法再次切换其他状态的，除非使用doEventForce()方法强制执行。

上面介绍的回调中的XXX指的是特定的状态和事件，在实际使用中，可以将XXX替换为对应的状态名或事件名。例如，进入idle状态时，会回调onenteridle()函数，状态机会根据前缀+XXX的规则来组合函数名，如果找到对应的函数，就执行相应的函数。

如果是同状态切换（状态A切换到状态A），那么状态切换的相关回调不会被触发，因为不涉及状态的离开、进入以及切换，但事件回调会被触发，因为事件发生了。

如图22-2所示为状态机执行事件时的状态切换流程，以及在流程中相关回调的执行顺序和起到的作用。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121351.jpeg)

图22-2　执行事件EventC从状态StateA到状态StateB

### 22.2　ModelBase详解

ModelBase是Quick的MVC框架中的模型基类，所有的数据模型都要继承于它，本节会介绍ModelBase的功能以及如何使用ModelBase。模型是现实世界的抽象，ModuleBase的主要功能是描述一个模型的数据结构，初始化以及获取模型数据，并默认添加了事件组件。

ModelBase使用一个schema变量来描述模型由哪些数据组成，schema可以理解为模型的架构，其是一个table变量，table的key为数据名字符串，value为一个table数组，数组下标1为数据类型，下标2为该数据的默认值。

在ModelBase的构造函数中，会要求传入模型的初始属性table，table的key和value分别为数据名和数据的值，构造函数会调用ModelBase:setProperties()方法来初始化属性，该方法会根据schema进行过滤，如果schema定义了a和b两个属性，在初始化table中传入a和c两个属性，那么ModelBase只会初始化a，而c由于不在schema变量描述的数据中，所以不会被初始化。如果a对应的value为nil，ModelBase则会从schema中取出默认值为其赋值。在初始化完成之后，a和其对应的值会被设置到这个对象自身中，例如，假设初始化了modelA，那么可以用modelA.a来访问成功初始化的属性。

getProperties()方法可以获取当前模型的属性table，该方法要求传入两个参数，第一个是要获取的属性table数组，第二个是要过滤的属性table数组。也可以什么参数都不传，这种情况下会根据ModelBase的fields变量来获取，将所有要获取的属性和对应的值添加到一个table中，最后返回这个table。

默认添加的事件组件可以允许程序员通过监听的方法，实现属性变化时自动刷新显示对象这样的功能，在接下来的实例中会详细介绍到。

那么应该如何使用ModelBase呢？这里举一个最简单的例子，例如，要实现一个点的模型，这个模型包含了x和y这两个变量。定义时可以这样子定义：

```
local PointModel= class("PointModel", cc.mvc.ModelBase)
-- 描述模型的变量结构
PointModel.schema = clone(cc.mvc.ModelBase.schema)
PointModel.schema["x"]  = {"number", 0}
PointModel.schema["y"]  = {"number", 0}
function PointModel:ctor()
    -- 调用父类的构造函数
    PointModel.super.ctor({x = 15, y = 50})
end
```

这个模型有什么用呢？单独的一个模型并没有什么用处，我们要介绍的是MVC而不是M，因此需要让它们串起来协作完成任务，接下来的MVC示例详解，会介绍如何实际地使用它。

### 22.3　MVC示例详解

Quick提供了丰富的例子来演示Quick的各种功能，通过这些例子可以快速掌握Quick的使用。虽然前面已经系统了解了Quick框架，但想要熟练地使用，还欠缺一些实践操作。在这里简单介绍一下几个比较重要的例子，相信读者学习了这几个例子后，就可以很快找到使用Quick的感觉了（一旦理解并接受了一个框架的设定，那么这个框架使用起来就会非常顺手，编程语言也是一样）。

在学习示例之前，首先需要将示例运行起来，可以使用Quick 3.5或QuickPlayer来运行示例，QuickPlayer可直接运行，呈现在我们眼前的就是示例列表了，单击即可运行对应的示例。如果使用的是Quick 3.5，需要先执行引擎目录下的setup.py，然后进入quick目录下的samples目录，在这里可以看到各种示例目录，任意进入一个示例目录，执行启动脚本即可运行该示例，在Windows下执行debug_win.bat，而在Mac下执行debug_mac.sh，示例目录中的src目录下存放着该示例的实现脚本。使用QuickPlayer运行示例，也可以在相应的目录下找到示例源码。

本章将介绍Quick示例中的mvc示例，MVC示例是一个简单的小游戏，场景中有两个角色，可以看到这两个角色的属性，通过按钮可以控制角色发射子弹，子弹会移动，当子弹命中对方之后会修改对方的属性，这些都可以直观地在界面上呈现出来，示例运行的效果如图22-3所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121352.jpeg)

图22-3　Quick的MVC示例的运行效果

#### 22.3.1　代码结构简介

打开src目录下的app目录，除了MyApp.lua脚本和scenes目录之外，该项目还添加了models、controllers和views这3个目录。

models目录下存放这Actor.lua和Hero.lua两个模型脚本，模型脚本使用了EventProtocol和StateMachine这两个重要的组件来完成一些核心的工作，如控制状态切换，以及事件分发。Actor继承于cc.mvc.ModelBase，而Hero则继承于Actor。

controllers目录下是PlayDuelController.lua脚本，PlayDuelController继承于cc.Node，该脚本是一个控制器，实现玩家所有的控制功能，例如，单击fire按钮发射子弹的逻辑，以及单击REFRESH按钮的刷新逻辑，都是在这里实现的。在MVC模式中，controller即可从model层中获取数据，也可以操控view层的显示。

views目录下是HeroView.lua脚本，该脚本继承于cc.Node，实现了角色的显示功能，包含角色动作的播放，以及根据模型的数据更新血量等信息的显示。

#### 22.3.2　启动流程详解

大概了解了MVC有哪些内容之后，下面来看看该示例的启动流程，从main.lua到MyApp.lua，然后进入MainScene场景。在MainScene的构造函数中初始化了控制层和刷新按钮。

```
local PlayDuelController = import("..controllers.PlayDuelController")
local MainScene = class("MainScene", function()
    return display.newScene("MainScene")
end)
function MainScene:ctor()
    display.newColorLayer(cc.c4b(255, 255, 255, 255)):addTo(self)
    -- 添加一个控制器对象
    self:addChild(PlayDuelController.new())
    -- 添加刷新按钮，按钮单击时切换到一个新的MainScene
    cc.ui.UIPushButton.new("Button01.png", {scale9 = true})
        :setButtonSize(200, 80)
        :setButtonLabel(cc.ui.UILabel.new({text = "REFRESH"}))
        :onButtonPressed(function(event)
            event.target:setScale(1.1)
        end)
        :onButtonRelease(function(event)
            event.target:setScale(1.0)
        end)
        :onButtonClicked(function()
            app:enterScene("MainScene", nil, "flipy")
        end)
        :pos(display.cx, display.bottom + 100)
        :addTo(self)
end
```

上面的代码有几个有趣的地方，第一个是使用import()方法来导入脚本而不是require，import()是functions.lua中定义的一个方法，本质和require没什么区别，但import()方法实现了一些自动搜索的逻辑，而require则要求输入一个完整的路径（包括完整的相对路径）。

在创建刷新按钮时，可以看到这里使用了一些简短连续的函数调用，有UIPushButton的成员方法，也有shortcodes.lua中为Node扩展的简短方法，每次调用完都紧接着跟踪下一次调用，之所以可以这么做，是因为上面的每个函数在执行完之后，都将self变量返回，所以紧跟在后面的函数调用实际上是执行这个self对象的方法，这与C++中实现连续赋值是同样的道理。

接下来看一下PlayDuelController.lua脚本，PlayDuelController的构造函数中创建了两个Hero模型，并将它们设置到app中，app是一个单例对象，本例中的MyApp添加了setObject()、getObject()和isObjectExists()等方法。创建完模型之后又创建了对应的显示对象，并添加到views_中，然后创建了两个按钮，按钮的单击回调为PlayDuelController:fire()方法，该方法传入了射击者和射击目标两个模型。

```
function PlayDuelController:ctor()
    -- 创建模型
    if not app:isObjectExists("player") then
        -- player对象只有一个，不需要每次进入场景都创建
        local player = Hero.new({
            id = "player",
            nickname = "dualface",
            level = 1,
        })
        app:setObject("player", player)
        print("create player")
    end
    self.player = app:getObject("player")
self.enemy = Hero.new({
    id = "enemy",
    nickname = "lurenjia",
    level = 1,
})
self.views_ = {}
self.bullets_ = {}
-- 创建显示对象
self.views_[self.player] = app:createView("HeroView", self.player)
    :pos(display.cx - 300, display.cy)
    :addTo(self)
self.views_[self.enemy] = app:createView("HeroView", self.enemy)
    :pos(display.cx + 300, display.cy)
    :flipX(true)
    :addTo(self)
-- 创建开火按钮
cc.ui.UIPushButton.new("Button01.png", {scale9 = true})
    :setButtonSize(160, 80)
    :setButtonLabel(cc.ui.UILabel.new({text = "fire"}))
    :onButtonPressed(function(event)
        event.target:setScale(1.1)
    end)
    :onButtonRelease(function(event)
        event.target:setScale(1.0)
    end)
    :onButtonClicked(function()
        self:fire(self.player, self.enemy)
    end)
    :pos(display.cx - 300, display.bottom + 100)
    :addTo(self)
cc.ui.UIPushButton.new("Button02.png", {scale9 = true})
    :setButtonSize(160, 80)
    :setButtonLabel(cc.ui.UILabel.new({text = "fire"}))
    :onButtonPressed(function(event)
        event.target:setScale(1.1)
    end)
    :onButtonRelease(function(event)
        event.target:setScale(1.0)
    end)
    :onButtonClicked(function()
        self:fire(self.enemy, self.player)
    end)
    :pos(display.cx + 300, display.bottom + 100)
    :addTo(self)
-- 注册帧事件
self:addNodeEventListener(cc.NODE_ENTER_FRAME_EVENT, handler(self,
self.tick))
self:scheduleUpdate()
-- 在视图清理后，检查模型上注册的事件，看看是否存在内存泄漏
self:addNodeEventListener(cc.NODE_EVENT, function(event)
    if event.name == "exit" then
        self.player:getComponent("components.behavior.EventProtocol"):
        dumpAllEventListeners()
    end
end)
end
```

接下来分析一下Hero模型的初始化，Hero模型继承于Actor模型，Actor模型又继承于cc.mvc.ModelBase对象，ModelBase提供了属性的get()、set()方法，并添加了EventProtocol组件，该组件实现了一个简单的消息机制。

```
function ModelBase:ctor(properties)
    -- cc方法为self扩展了addComponent等方法
    cc(self):addComponent("components.behavior.EventProtocol"):exportMethods()
    -- 添加组件，并初始化属性
    self.isModelBase_ = true
    if type(properties) ~= "table" then properties = {} end
    self:setProperties(properties)
end
```

在Actor中，首先复制了父类的schema，schema是模型的结构，描述了模型有哪些数据变量，以及变量的类型和默认值。在构造函数中添加了状态机组件，该组件是一个私有变量，在Lua中可以使用下划线来声明“私有”变量，尽管该私有变量仍然可以被外部访问到。

接下来初始化了状态机的默认事件，这是一个table，table中记录了每个状态的名字，以及该名字可以由哪些状态切换而来，以及可以切换至何种状态。管理状态的切换是状态机的核心职责。紧接着又定义了一个table，该table定义了各种状态切换时的回调，当状态机发生状态切换时，会执行相应的回调。最后调用状态机的setupState()方法将它们设置到状态机中，并调用状态机的doEvent()方法启动状态机，实际上就是将当前状态切换到start状态。

```
Actor.schema = clone(cc.mvc.ModelBase.schema)
Actor.schema["nickname"] = {"string"} --字符串类型，没有默认值
Actor.schema["level"]    = {"number", 1} --数值类型，默认值 1
Actor.schema["hp"]       = {"number", 1}
function Actor:ctor(properties, events, callbacks)
    Actor.super.ctor(self, properties)
    -- 因为角色存在不同状态，所以这里为Actor绑定了状态机组件
    self:addComponent("components.behavior.StateMachine")
    -- 由于状态机仅供内部使用，所以不应该调用组件的exportMethods()方法，改为用内
    部属性保存状态机组件对象
    self.fsm__ = self:getComponent("components.behavior.StateMachine")
    -- 设定状态机的默认事件
    local defaultEvents = {
        -- 初始化后，角色处于idle状态
        {name = "start",  from = "none",    to = "idle" },
        -- 开火
        {name = "fire",   from = "idle",    to = "firing"},
        -- 开火冷却结束
        {name = "ready",  from = "firing",  to = "idle"},
        -- 角色被冰冻
        {name = "freeze", from = "idle",    to = "frozen"},
        -- 从冰冻状态恢复
        {name = "thaw",   form = "frozen",  to = "idle"},
        -- 角色在正常状态和冰冻状态下都可能被杀死
        {name = "kill",   from = {"idle", "frozen"}, to = "dead"},
        -- 复活
        {name = "relive", from = "dead",    to = "idle"},
    }
    -- 如果继承类提供了其他事件，则合并
    table.insertto(defaultEvents, checktable(events))
    -- 设定状态机的默认回调
    local defaultCallbacks = {
        onchangestate = handler(self, self.onChangeState_),
        onstart       = handler(self, self.onStart_),
        onfire        = handler(self, self.onFire_),
        onready       = handler(self, self.onReady_),
        onfreeze      = handler(self, self.onFreeze_),
        onthaw        = handler(self, self.onThaw_),
        onkill        = handler(self, self.onKill_),
        onrelive      = handler(self, self.onRelive_),
        onleavefiring = handler(self, self.onLeaveFiring_),
    }
    -- 如果继承类提供了其他回调，则合并
    table.merge(defaultCallbacks, checktable(callbacks))
    self.fsm__:setupState({
        events = defaultEvents,
        callbacks = defaultCallbacks
    })
    self.fsm__:doEvent("start") --启动状态机
end
```

Hero在Actor的基础上增加了exp属性，并添加了一个简单的升级逻辑。这里将Hero继承于Actor的目的，只是想演示一下模型的继承、扩展以及更多的模型使用方法，而非什么有深意的设计。

需要注意的是这里定义的事件以及schema，并非是Hero对象的变量，而是所有Hero对象共用的变量。

```
local Actor = import(".Actor")
local Hero = class("Hero", Actor)
Hero.EXP_CHANGED_EVENT = "EXP_CHANGED_EVENT"
Hero.LEVEL_UP_EVENT = "LEVEL_UP_EVENT"
Hero.schema = clone(Actor.schema)
Hero.schema["exp"] = {"number", 0}
-- 升到下一级需要的经验值
Hero.NEXT_LEVEL_EXP = 50
-- 增加经验值，并升级
function Hero:increaseEXP(exp)
    assert(not self:isDead(), string.format("hero %s:%s is dead, can't
    increase Exp", self:getId(), self:getNickname()))
    assert(exp > 0, "Hero:increaseEXP() - invalid exp")
    self.exp_ = self.exp_ + exp
    -- 简化的升级算法，每一个级别升级的经验值都是固定的
    while self.exp_ >= Hero.NEXT_LEVEL_EXP do
        self.level_ = self.level_ + 1
        self.exp_ = self.exp_ - Hero.NEXT_LEVEL_EXP
        self:setFullHp() -- 每次升级，HP都完全恢复
        self:dispatchEvent({name = Hero.LEVEL_UP_EVENT})
    end
    self:dispatchEvent({name = Hero.EXP_CHANGED_EVENT})
    return self
end
```

HeroView继承于Node，在构造函数中创建了3个显示节点，分别用于显示角色、角色状态信息、角色名字，并使用EventProxy监听了一些事件。监听的对象是传入的Hero模型，在Hero模型执行相应的事件时，会回调HeroView相应的回调方法。

这里比较有意思的是EventProxy，如果单独看EventProxy的源码对其用处感到疑惑的话，那么这几行简单的代码应该可以让读者体会到EventProxy的方便之处。因为我们每次注册的监听，都需要在该节点被移除时注销，而EventProxy最方便的地方就是可以自动注销这些监听。

```
local HeroView = class("HeroView", function()
    return display.newNode()
end)
function HeroView:ctor(hero)
    local cls = hero.class
    -- 通过代理注册事件的好处：可以方便地在视图删除时，清理所有通过该代理注册的事件
    -- 同时不影响目标对象上注册的其他事件
    -- EventProxy.new() 第一个参数是要注册事件的对象，第二个参数是绑定的视图
    -- 如果指定了第二个参数，那么在视图删除时，会自动清理注册的事件
    cc.EventProxy.new(hero, self)
        :addEventListener(cls.CHANGE_STATE_EVENT, handler(self, self.on
        StateChange_))
        :addEventListener(cls.KILL_EVENT, handler(self, self.onKill_))
        :addEventListener(cls.HP_CHANGED_EVENT, handler(self, self.upda
        teLabel_))
        :addEventListener(cls.EXP_CHANGED_EVENT, handler(self, self.update
        Label_))
    self.hero_ = hero
    self.sprite_ = display.newSprite():addTo(self)
    self.idLabel_ = cc.ui.UILabel.new({
            UILabelType = cc.ui.UILabel.LABEL_TYPE_TTF,
            text = string.format("%s:%s", hero:getId(), hero:getNickname()),
            size = 20,
            color = display.COLOR_BLACK,
        })
        :align(display.CENTER, 0, 100)
        :addTo(self)
    self.stateLabel_ = cc.ui.UILabel.new({
            UILabelType = cc.ui.UILabel.LABEL_TYPE_TTF,
            text = "",
            size = 20,
            color = display.COLOR_RED,
        })
        :align(display.CENTER, 0, 70)
        :addTo(self)
    self:updateSprite_(self.hero_:getState())
    self:updateLabel_()
end
```

#### 22.3.3　发射子弹

当按下发射按钮之后，会回调到PlayDuelController的fire()方法，这个回调在PlayDuelController的构造函数中，创建发射按钮时就已经设置好了。fire()函数传入了射击者和射击目标两个模型，首先通过射击者模型的canFire()方法判断射击者是否可以发射子弹，该方法执行了状态机的canDoEvent()方法，传入了"fire"状态，状态机会根据当前的状态来判断是否可以发射子弹，判断的规则正是在Actor构造函数中初始化的默认事件表。

如果可以发射子弹则执行发射者模型的fire()方法，并创建一个子弹Sprite，将子弹添加到bullets_列表中，并为子弹赋予一些初值，如攻击者、目标、移动速度、初始位置等。

```
function PlayDuelController:fire(attacker, target)
    if not attacker:canFire() then return end
    attacker:fire() -- 开火后，需要冷却一定时间才能再次开火
    -- 创建子弹图像，并设置起始位置和飞行方向
    local bullet = display.newSprite("#Bullet.png"):addTo(self)
    local view = self.views_[attacker]
    local x, y = view:getPosition()
    y = y + 12
    if view:isFlipX() then
        x = x - 44
        bullet.speed = -5
    else
        x = x + 44
        bullet.speed = 5
    end
    bullet:pos(x, y)
    bullet.attacker = attacker
    bullet.target = target
    self.bullets_[#self.bullets_ + 1] = bullet
    self:InterfaceTest()
end
```

Actor模型的fire()方法中执行了状态的doEvent()方法，先执行了fire事件，再执行ready事件，ready事件后面跟着一个延迟执行参数，也就是发射了之后等待一段时间才执行ready事件。这个参数会被传入到Actor的onLeveFiring_()方法中，在onLeveFiring_()方法中会根据这个参数，启动一个schedule，在等待一段时间后切换到下一个事件。

```
function Actor:fire()
    self.fsm__:doEvent("fire")
    self.fsm__:doEvent("ready", Actor.FIRE_COOLDOWN)
end
```

当状态切换时，状态机会触发onchangestate回调，并传入切换的状态，在Actor模型初始化时，将onchangestate回调设置为Actor的onChangeState_()方法，在该方法中通过dispatchEvent()方法触发了Actor.CHANGE_STATE_EVENT事件。

```
function Actor:onChangeState_(event)
    printf("actor %s:%s state change from %s to %s", self:getId(), self.
    nickname_, event.from, event.to)
    event = {name = Actor.CHANGE_STATE_EVENT, from = event.from, to = event. to}
    self:dispatchEvent(event)
end
```

HeroView将onStateChange_()方法注册为Actor.CHANGE_STATE_EVENT事件的回调。

```
function HeroView:onStateChange_(event)
    self:updateSprite_(self.hero_:getState())
end
```

HeroView的onStateChange_()方法调用updateSprite_()方法，并传入了Hero模型当前的状态，当将程序切换到firing状态时，firing状态会作为参数被传入到updateSprite_()方法中，updateSprite_()方法会根据状态切换当前显示的SpriteFrame，firing状态对应的图片是HeroFriring.png，这时候就可以看到角色做出了发射子弹的动作，而当状态切换回idle时，角色又会回到普通的待机动作。

```
function HeroView:updateSprite_(state)
    local frameName
    if state == "idle" then
        frameName = "HeroIdle.png"
    elseif state == "firing" then
        frameName = "HeroFiring.png"
    end
    if not frameName then return end
    self.sprite_:setSpriteFrame(display.newSpriteFrame(frameName))
end
```

#### 22.3.4　命中目标

当发射了一发子弹后，这发子弹就会开始移动，最后命中目标，并执行相应的命中逻辑。创建出来的子弹在经过一系列初始化之后，会被添加到PlayDuelController的bullets_列表中。PlayDuelController在构造函数中通过scheduleUpdate开启了update更新，并将tick()方法注册为帧节点事件的回调，update每帧会发送一个帧节点事件，从而驱动tick()方法时每一帧的执行。

在tick()方法中，遍历了bullets_数组，首先更新了每个子弹的坐标，然后处理了超出屏幕外的子弹。接下来判断子弹和目标的距离，调用PlayDuelController的hit()方法来执行命中判断的逻辑。

```
function PlayDuelController:tick(dt)
    for index = #self.bullets_, 1, -1 do
        local bullet = self.bullets_[index]
        local x, y = bullet:getPosition()
        x = x + bullet.speed
        bullet:setPositionX(x)
        if x < display.left - 100 or x > display.right + 100 then
            bullet:removeSelf()
            table.remove(self.bullets_, index)
        elseif bullet.target then
            local targetView = self.views_[bullet.target]
            local tx, ty = targetView:getPosition()
            if dist(x, y, tx, ty) <= 30 then
                if self:hit(bullet.attacker, bullet.target, bullet) then
                    bullet:removeSelf()
                    table.remove(self.bullets_, index)
                else
                    bullet.target = nil
                end
            end
        end
    end
end
```

PlayDuelController的hit()方法首先判断目标是否死亡（也是通过状态机），如果没有死亡，则调用attacker（也就是Hero模型）的hit()方法，如果没有命中目标，则会在角色头上飘出一个Miss图片，表示未命中。

```
function PlayDuelController:hit(attacker, target, bullet)
    if not target:isDead() then
        local damage = attacker:hit(target)
        if damage <= 0 then
            local miss = display.newSprite("#Miss.png")
                :pos(bullet:getPosition())
                :addTo(self, 1000)
            transition.moveBy(miss, {y = 100, time = 1.5, onComplete =
            function()
                miss:removeSelf()
            end})
        end
        return damage > 0
    else
        return false
    end
end
```

Actor的hit()方法进行了闪避和伤害计算，最后执行扣除血量的操作，当血量更新时，会触发HeroView的更新血量逻辑，而当血量被减少到0以下时，Hero模型会执行kill事件，切换到死亡状态，同时触发HeroView的onKill_回调，播放死亡动画。中间的事件转发流程同前面介绍的流程一致，这里就不重复介绍了。

```
function Actor:hit(enemy)
    assert(not self:isDead(), string.format("actor %s:%s is dead, can't
    change Hp", self:getId(), self:getNickname()))
    -- 简化算法：伤害 = 自己的攻击力 - 目标防御
    local damage = 0
    if math.random(1, 100) <= 80 then -- 命中率 80%
        local armor = 0
        if not enemy:isFrozen() then -- 如果目标被冰冻，则无视防御
            armor = enemy:getArmor()
        end
        damage = self:getAttack() - armor
        if damage <= 0 then damage = 1 end -- 只要命中，强制扣HP
    end
    -- 触发事件，damage <= 0可以视为 miss
    self:dispatchEvent({name = Actor.ATTACK_EVENT, enemy = enemy, damage =
    damage})
    if damage > 0 then
        -- 扣除目标HP，并触发事件
        enemy:decreaseHp(damage) -- 扣除目标Hp
        enemy:dispatchEvent({name = Actor.UNDER_ATTACK_EVENT, source = self,
        damage = damage})
    end
    return damage
end
```

### 22.4　小结

读完22.3节的示例代码后，本节简单点评一下这个例子，虽然从运行的效果上看22.3节例子非常简单，但代码阅读起来并不轻松，笔者认为整个例子可以变得更加简单一些，将模型刷新事件和状态更新事件的转发变得更加清晰一些。阅读这样的代码，需要先了解每个对象的职责，然后将其中的消息转发流程梳理清楚。

如果不使用MVC模式来实现22.3节的例子的话，那么代码至少可以减少50%，因为可以不用将程序分成3层，然后通过消息在这三层之间来回通信。要获取的数据以及执行的操作都可以直接执行，不需要绕一圈，这虽然会增加一定的耦合，但耦合真的是令人无法接受吗？有时候适当的耦合反而可以让代码更加简单，22.3节的例子只是演示了Quick的MVC框架的使用方法，并没有体现出MVC框架的优点，反而是让我们看到了一些缺点。

在这里对MVC框架做一个简单的分析，诚然MVC是一个不错的框架模式，否则不可能被应用得如此广泛，但使用一个框架的前提是需要它，它可以很好地解决问题，而不是这个框架很不错，很流行（系统学习某框架的情况除外）。

首先来看一下MVC的优点，主要的优点有耦合性低、可维护性高、方便管理，划分为3层是比较恰当的，每一层都有其明确的职责，每一层的接口明确，利于多人协作开发和维护（原本由1个程序员完成的工作可以划分为3个程序员同时执行），基于这些优点，MVC框架应用于大型的复杂软件工程是很合适的。

那么MVC有什么缺点呢？主要有不易理解、复杂度高、效率较低3点。对于简单的功能，严格按照MVC模式进行分层，会增加结构的复杂性，导致过多的更新操作，使整个流程更加复杂。所以MVC并不适用于结构和规模较小的程序（想象一下使用MVC模式来实现一个HelloWorld）。所以还是应该立足于需求，根据需求来划分出最合适的模块。

最后，除了MVC之外，还应该学习一下Quick的其他示例，以下几个示例有比较高的学习价值。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　UI示例，演示了Quick封装的UI框架，该框架提供了丰富的控件以及完整的布局方案。由于使用Lua开发的大部分工作都是和UI打交道，而这套UI框架可以提高开发效率，是一套纯脚本的UI框架，使用Sprite、Label等基础对象组装而成，不同于Cocos2d-x的Widget框架，也和CocosStudio没有任何关系。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　StateMachine示例，该示例演示了Quick的状态机组件，实际上这个组件的实用性还是很不错的，像Quick内部的很多控件都使用了该组件。Quick 3.5版本移除了该示例，在Quick 3.3版本或github上的版本才有该示例的代码。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Filters示例，该示例演示了Quick实现的丰富特效，这些特效是在C++层使用Shader实现的，Quick在Lua层提供了简单的接口让程序员能够方便地使用这些效果。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Coinflip示例，该示例是一个完整的游戏示例，可以扫除初学者不知如何上手的问题，读者也可以从中酝酿使用Quick的感觉。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Tests示例，该示例系统地演示了Quick框架的大部分功能，是一堆例子的集合，读者可以将该示例中所有例子都运行一遍，观看效果，简单了解一下。在需要使用到某部分功能的时候，可以找到对应的代码来参考。