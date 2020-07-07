# 第29章　CocoStudio动画编辑器

CocoStudio工具链中的动画编辑器主要是给美术人员编辑骨骼动画所用，动画编辑器的功能如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　编辑骨骼动画，以及逐帧动画。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　管理一个对象身上的多个动画。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　编辑帧事件。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　编辑骨骼的碰撞区域。

本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　骨骼动画的命名规范。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　骨骼动画框架和运行流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　使用动画编辑器。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　编写代码。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　骨骼动画与碰撞检测。

整个骨骼动画系统复杂度相对较高，学习本章内容需要足够的耐心。另外，动画编辑器是CocoStudio 1.x版本的四件套中唯一一个在CocoStudio升级至2.x后仍被使用的工具。

## 29.1　骨骼动画的命名规范

在这里需要以严格的命名规范来规范美术命名，以节省程序员和美术人员之间的沟通成本，需要规范的命名主要有4个。之所以将命名规范放在最前面，是为凸显其重要性。

在命名规范上，应该美术人员迁就程序员，而不是程序员迁就美术人员。有规律的东西，可被封装，可维护。如果在美术人员开工之前没有给出相关的命名规范，则看到的很可能是很多毫无意义的字母+数字（如aaa123、asdf888），以及一些韵母发错的拼音（hu lan ren），坚决不允许让这些恶心的命名进入到我们神圣的代码中，降低我们的代码质量！

### 29.1.1　骨骼的命名

骨骼的规划和搭建是美术人员的工作，但对于骨骼的命名，则需要遵循一定的规范，这样不仅易于维护，而且能够便于程序员编写代码。

例如一个类似CS的小游戏中，“爆头”会使角色直接死亡，在进行碰撞的时候，只需要检测当前碰撞的Bone名字是否为Head（可以定义为一个常量或者宏），而不需要检测各种纷乱的命名，下面是一些命名的参考：

Head头、Body身体、Shoulder肩膀、Arm手臂、Palm手掌、Leg腿、Sole脚掌 （后缀可用A、B、C表示多节）。

### 29.1.2　动画的命名

动画的命名用于规范动画对象之间的管理，命名的规范在播放骨骼动画时，也有诸多的好处，便于抽象和封装，下面是一些命名参考：

Idle待机、Move移动、Run冲刺、Attack打、Tick踢、Die死亡（如果有多种死亡动画，可以用后缀区分）、Skill放技能、Hit受击。

### 29.1.3　帧事件的命名

帧事件是由骨骼运行到某一帧的时候发出的事件，这个名字由程序定义。例如 EAttack攻击事件，在攻击动画播放到一半的时候，才发出这个事件。

### 29.1.4　整个骨骼动画对象的命名

这是文件的命名规范，文件名和骨骼对象名字需要一致，可以根据具体的需求来设计命名规则，如用Arm前缀来区分非骨骼动画文件，用对象的ID或名字作为文件名。

## 29.2　骨骼动画框架和运行流程

本节介绍骨骼动画的框架和运行流程，一静一动，让读者了解骨骼动画几个关键类之间的关系，对骨骼动画有整体的认识，以及骨骼动画的运行和渲染流程。

### 29.2.1　骨骼动画框架

骨骼动画框架结构如图29-1所示。可以分为3部分Armature、Bone和Armature Animation来看。Armature是需要由我们来创建，并且操作的对象，是一个骨骼动画的实例，Bone和Armature Animation在Armature之下。Bone是一块块的骨骼，Armature在初始化的时候，根据Armature Data（从.csb或.json文件解析出来）中的骨骼数据将其创建并组织起来，搭建成为一个骨架。而Armature Animation则是该Armature对应的骨骼动画，Armature在初始化的时候，根据Armature Data中的动画数据进行初始化，记录了每个动画、每一关键帧的骨骼运动（生成若干Tween对象），需要通过Armature使用Armature Animation来播放骨骼动画，骨骼动画将驱动骨架中的骨骼进行运动。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160716.jpeg)

图29-1　骨骼动画框架结构

Armature是一个骨骼动画对象，可以理解为一个有骨骼动画功能的节点，可以使用Armature播放骨骼动画，做一些简单的碰撞检测，多个Armature之间能以父子关系进行绑定（在动画编辑器中无法编辑嵌套骨骼动画，也就是父子Armature，只能由代码或第三方编辑器来实现，如cpp-tests中的TestArmatureNesting例子）。

例如，Armature1是一个法师，Armature2是一把法杖，这把法杖本身也有着骨骼动画，可以变身成各种武器，那么通过父子骨骼的绑定，可以把法杖绑定到法师的手上，也可以绑定到一个战士的手上，Armature主要的职责如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　骨骼的管理。Armature管理骨骼动画中的所有骨骼，使用一个string-Bone的Map容器来管理所有的骨骼，Map的key为骨骼名字（所以骨骼不能同名）。调用addBone方法可以将一个骨骼作为子骨骼添加到自己的骨骼中。调用getBone方法可以根据骨骼名获取对应的骨骼，调用getBoneDic方法可以获取整个骨骼容器。调用changeBoneParent方法可以改变骨骼的父骨骼。调用removeBone方法可以移除一块骨骼或递归移除一块骨骼及其下的子骨骼。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　播放骨骼动画。通过调用getAnimation方法获取ArmatureAnimation对象，调用其playWithIndex、play等方法，传入动画的下标、动画的名字来播放骨骼动画。调用playWithNames和playWithIndexes可以让多个动画按照指定的顺序播放。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　驱动骨骼更新。在Armature的update方法中会调用Animation的update来更新动画，以及_topBoneList容器中的所有骨骼的update。topBoneList容器存放位于骨骼框架最上层的根骨骼，它们的update将会应用一系列的矩阵计算，调用DisplayFactory::updateDisplay，并驱动所有子骨骼的更新。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　驱动骨骼渲染。在Armature的draw方法中，首先遍历所有的子节点。对Bone类型的子节点进行特殊处理，对非Bone类型的子节点直接调用visit。**在对Bone子节点的处理中，首先获取Bone的实际渲染节点DisplayRenderNode**，针对不同渲染类型的节点进行不同的渲染（支持Skin、Armature、Particle等类型）。注意，这里对Bone和非Bone子节点进行了不同的处理，非Bone子节点（以及其下的子节点）可以被正常地显示，而Bone子节点本身不会被显示。所以对Bone添加任何子节点都不会被显示出来，Bone需要依赖其渲染节点来显示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　碰撞检测的支持。提供getBoundingBox获取包围盒，以及物理的方法进行碰撞检测。

**骨骼Bone**作为骨骼动画中的最小单元，拥有非常重要的功能，**骨骼可以衔接骨骼，也可以衔接Armature对象**。每个Bone对象都有一个唯一的name，唯一的范围指的是在这个骨骼动画中（因为Armature是用骨骼的name为key来管理所有骨骼的）。每一个骨骼都会拥有一个显示列表，显示列表中存放着真正的渲染节点。通过切换显示列表可以实现换装、临时隐藏骨骼、播放骨骼的帧动画等功能。

骨骼作为一种显示对象，在Cocos2d-x中，每个显示对象都拥有通过矩阵计算，将自己显示到正确位置的职责，主要的职责如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　显示。Armature本身没有显示功能，整个动画的表现由骨骼来完成。换装功能也由骨骼来实现。Armature的帧动画是通过在显示列表中切换骨骼的当前显示对象来实现的，一个帧动画会由多个Skin组成，**Skin继承于Sprite**，每个Skin对应一帧的表现，通过切换Skin来切换下一帧。也就是说，一个拥有100帧的骨骼，在创建时就会生成100个Skin（过多的帧动画会占用额外的内存）。Bone的渲染节点也可以是一个Armature，但这种情况比较复杂，并且Cocos2d-x官方也没有提供示例，在后面的内容中将会具体介绍。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　碰撞检测。骨骼身上的ColliderDector提供了精准的骨骼形状可供碰撞检测。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　骨骼连接。在骨骼运动的时候，根据父骨骼的矩阵和Tween的数据进行运动，并提供矩阵信息给子骨骼。

Armature使用ArmatureAnimation来播放动画，ArmatureAnimation主要的职责如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　动画的播放、循环、暂停、恢复和停止。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　多个动画的顺序播放。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　管理和监听动画事件、帧事件的触发。

### 29.2.2　运行流程

要使用Armature，需要先将Armature的资源进行加载，这些资源包括.json文件、Plist文件、纹理等，资源的加载是由ArmatureDataManager完成的，需要手动调用ArmatureDataManager来加载资源，然后才是创建Armature对象并操作该对象，初始化和运行的流程如图29-2所示，下面将详细介绍这两个流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160717.jpeg)

图29-2　骨骼动画初始化和运行流程简图

### 29.2.3　初始化

Armature对象的初始化流程如下：

（1）如果是第一次使用，需要**先在ArmatureDataManager中加载骨骼动画文件**，将数据缓存到ArmatureDataManager中。

（2）创建Armature对象，此时会从ArmatureDataManager的Armature中初始化以下对象。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　初始化ArmatureAnimation对象并设置AnimationData。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　根据ArmatureData创建所有的骨骼对象。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　获取下标为0的MovementData（默认动画），从中取出每个骨骼默认动画的FrameData，并设置到骨骼上。将骨骼的当前显示状态切换到FrameData中的当前帧，也就是美术人员在动画编辑器中，编辑的第一个动画的第一帧。

（3）完成各种初始化之后，就可以对Armature对象进行操作。

### 29.2.4　运行和更新

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　获取Armature的ArmatureAnimation对象，调用Play方法播放指定动画。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　在ArmatureAnimation中更新MovementData为新动画的数据，停止播放当前动画（清除TweenList）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　根据MovementData中的MovementBoneData，遍历Armature的骨骼，将本动画没有使用的骨骼隐藏，让其他骨骼播放Tween，并将Tween添加到TweenList进行管理。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　到这里ArmatureAnimation准备好了自身以及Armature下所有骨骼的数据。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　在Armature被addChild到场景中时，Armature的update就会执行。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　在Armature的update中，每帧都会调用ArmatureAnimation以及最顶层骨骼的update方法来驱动整个骨骼框架进行更新。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　动画的更新，会对TweenList中的每个Tween对象进行更新，触发并处理动作事件**movementEvent**，以及处理帧事件**frameEvent**。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　Tween在更新中，负责在两个关键帧之间的平滑过渡，以及帧之间的切换和帧事件的触发。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160715.jpeg)　在骨骼的更新中，主要是将Tween的运动计算为矩阵，然后应用到显示对象上，并驱动所有子骨骼进行更新，这里体现了骨骼运动的先后关系。

## 29.3　使用动画编辑器

了解了骨骼动画系统的基本框架之后，再来了解一下动画编辑器的使用，本节将介绍使用动画编辑器来编辑骨骼动画，编辑帧动画，动画管理，帧事件编辑，以及碰撞区域编辑。

编辑骨骼动画包含两个模式，即“形体模式”和“动画模式”，如图29-3所示，“形体模式”用来编辑整个骨骼架构，“动画模式”则用来编辑骨骼运动。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160718.jpeg)

图29-3　动画编辑器工具栏

首先切换到“形体模式”下来创建骨骼，可以单击“创建骨骼”按钮进入创建骨骼模式，在屏幕上拉出若干骨骼，再单击“绑定骨骼”按钮进行绑定，也可以单击“创建连续骨骼”按钮来创建自动绑定的骨骼，再次单击当前创建骨骼模式的按钮，即可退出骨骼创建模式。“创建骨骼”和“创建连续骨骼”两种模式的区别是，**“创建连续骨骼”模式创建的骨骼会自动绑定上一个骨骼为父节点，而“创建骨骼”模式创建的骨骼之间没有任何联系**，如图29-4所示。在连续创建骨骼的时候，两个骨骼并不需要看上去真的连接在一起，两块骨骼之间可以分开一定的距离。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160719.jpeg)

图29-4　连续骨骼和骨骼

在“创建骨骼”模式下的操作应该是按下鼠标→拖动→松开，而不是**单击**→按下→拖动→松开。简而言之，拉出来的骨骼应该是像下方的骨骼一样，每次松开鼠标后都会创建一个骨骼，应该避免创建多余的无用骨骼，否则编辑的时候会带来一些困惑。

选中骨骼，可以在属性面板中编辑骨骼的属性，如图29-5所示，如果骨骼拉得不够长，想调整一下长度，或者调整缩放、位置、旋转，以及显示的图片效果，都可以在该面板中进行操作，可以将多个图片拖动到“渲染资源”栏下，方便在代码中进行换装操作，也方便在编辑器中切换显示，单击下拉列表框会出现多个图片可供选择。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160720.jpeg)

图29-5　骨骼属性面板

骨骼的名字需要在“对象结构”面板中（如图29-6所示）才能修改，双击面板中的名字进行修改或按F2键修改，或在右键菜单中修改。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160721.jpeg)

图29-6　“对象结构”面板

在该面板中还可以删除骨骼对象，也可以直接选中对应的骨骼按Delete键进行删除，注意要选中骨骼，而不是选中图片。

选中顶部工具栏中间的运动模块可以进入编辑模式，可以编辑位置、旋转和缩放效果，这里有一个反向动力学选项，开启该选项会进入反向动力学的编辑模式，可以理解为一种智能的编辑位移和旋转的模式。前向动力学和反向动力学是骨骼动画中两个重要的概念，前向动力学是父骨骼运动带动子骨骼运动，而反向动力学是**子骨骼运动来带动父骨骼**，反向动力学的计算成本比前向动力学高很多，如图29-7所示为反向动力学，这个模式仅是为了方便编辑。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160722.jpeg)

图29-7　反向动力学效果

按H键或单击工具栏中的“查看骨骼关系”按钮，弹出骨骼关系视图，如图29-8所示，可以在其中直观地预览整个骨骼框架（检查骨骼关系是否正确），并方便地调整骨骼架构进行绑定、解绑、删除等操作，单击“绑定骨骼”按钮，从父骨骼身上画线到子骨骼可以快速地绑定骨骼，单击工具栏上的“连续绑定骨骼”按钮，在父骨骼身上按下→拖动到子骨骼身上→松开也可以快速地绑定。连续创建子骨骼可以方便地创建链式骨骼，如果一个骨骼需要有多个子骨骼，则只能用手动绑定的方式（如一个人的身体需要绑定头、手、脚等子骨骼）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160723.jpeg)

图29-8　骨骼关系视图

在编辑骨骼的时候，不一定要给所有的骨骼绑定上显示对象，一些骨骼可以单纯地辅助显示对象骨骼更好地显示，以用来模拟一些看不见的事物或者力量，如绑在气球上的线。

接下来可以切换到“**动画模式**”来编辑动画，切换到“动画模式”后，整个布局会发生改变，“对象结构”面板消失，因为在动画模式下不能对骨骼框架进行修改了，只能对骨骼动画进行编辑，增加了“动作列表”和“动画帧”两个面板，“动作列表”面板可以管理骨骼对象的骨骼动画，可以进行命名、添加、删除和复制等操作，如图29-9所示。这里有一个细节是在代码中，可以通过下标来指定动画，下标和“动作列表”面板的索引是一致的，但“动作列表”面板中的动画不能调整顺序。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160724.jpeg)

图29-9　“动作列表”面板

选中要编辑的动画，即可在“动画帧”面板中进行编辑，如图29-10所示。编辑的方式是选中要编辑的骨骼，插入关键帧，然后在屏幕上对该骨骼进行位移、旋转、缩放等操作，也可以在该骨骼的“属性”面板中进行编辑，两帧会直接自动进行补间计算（Tween）。通过右边的帧曲线可以控制两帧过度的节奏，一般是匀速过度，通过选择曲线，可以调整先快后慢或者先慢后快等节奏。单击左上角的“播放”按钮可以预览整个动画，通过隐藏骨骼可以只关注当前编辑的局部骨骼动画效果。当骨骼过多时，骨骼列表编辑起来会比较痛苦，并且关键帧只能逐个骨骼添加，不能批量添加，如果以树状形式呈现骨骼列表，则会清晰很多。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160725.jpeg)

图29-10　编辑动画

在插入的关键帧中，骨骼属性面板增加了一些功能，如显示/隐藏、混合、帧层级，以及帧事件等属性，如图29-11所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160726.jpeg)

图29-11　骨骼属性面板

不能在动画中动态地销毁骨骼，但可以隐藏骨骼，如一个抛手榴弹的动作，可以在出手的瞬间添加一个关键帧将骨骼隐藏，然后设置帧事件，程序在该帧事件的回调中，可以获取手榴弹骨骼的位置、旋转等属性，然后在该位置创建出一个真正的手榴弹并开始飞行。

**编辑帧动画**有两种方法，第一种方法是将若干帧动画资源（其他工具导出的序列帧图片）拉到“动画帧”面板中的骨骼上，然后会弹出“序列帧间隔”对话框，如图29-12所示，可以在其中选择每隔多帧切换下一帧，然后自动插入关键帧。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160727.jpeg)

图29-12　“序列帧间隔”对话框

另一种方法是将序列帧图片**批量地**（即在资源面板中选中多张图片）拖到“属性面板”的渲染资源中，然后手动创建关键帧，在每个关键帧中设置当前显示的渲染资源，可以在下拉列表框中使用右键快捷菜单删除资源（序列帧的名字需要有一定的规律和顺序，如图29-13中的XXX01.png）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160728.jpeg)

图29-13　设置当前显示资源

在制作骨骼动画和帧动画时，有一个问题需要特别注意，即图片资源的对齐问题，如果前期没有规划好对齐问题，当后期美术资源量多的时候，则会是一个比较痛苦的事情。

当从Flash等工具导出序列帧图片时，要么保证**每张图片导出的尺寸都一样**（不占磁盘空间，但运行时会更占内存），要么导出的图片需要**对准一个锚点**（如尺寸不一，但每张图片的中心点都是一致的），这两个规则是为了解决播放动画时的颤抖效果，因为如果没有对齐图片，导致动画在播放的时候会忽上忽下，给人不稳定、不连贯的感觉，如图29-14所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160729.jpeg)

图29-14　没有对齐的两帧动画

在制作骨骼动画时，一个好习惯是将骨骼对象的脚底位置定位到原点上**（画布的中心点）**，如果对象需要站在地面上，则统一使脚底与原点对齐（如图29-15所示），这样可以方便程序员设置骨骼动画对象的位置，不需要每一个对象都在代码中特别调整其Y轴，使一系列不同的对象在Y轴一致的时候看上去是站在同一个地平线上而不是高低穿插着。在制作帧动画时，可以通过百分比或者固定位置来设置动画对象脚底的位置，然后在代码中统一设置锚点。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160730.jpeg)

图29-15　统一使脚底与原点对齐

选中骨骼，单击工具栏中的“资源编辑器”按钮，或者按下G键，弹出“资源编辑”面板，如图29-16所示。在其中可以编辑图片的形状用于碰撞检测，也可以编辑骨骼图片的锚点，面板上的工具栏提供了快速编辑形状的方法，当需要划分出一块碰撞区域的时候，可以在其中编辑碰撞区域的形状，当然，碰撞检测需要在代码中实现。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160731.jpeg)

图29-16　资源编辑

## 29.4　编写代码

### 29.4.1　加载骨骼资源

在使用骨骼动画之前，需要将骨骼动画的资源手动加载进来，这里支持CocoStudio导出的ExportJSON格式以及DragonBones 2.0导出的骨骼动画，并且有同步加载和异步加载两种方式，异步加载通过设置回调可以在资源加载完成后马上知道。示例代码如下：

```
//异步加载骨骼动画
ArmatureDataManager::getInstance()->addArmatureFileInfoAsync(
"armature/Dragon.png",
"armature/Dragon.plist",
"armature/Dragon.xml",
    this, schedule_selector(TestAsynchronousLoading::dataLoaded));
ArmatureDataManager::getInstance()->addArmatureFileInfoAsync(
    "armature/Cowboy.ExportJSON",
this,
schedule_selector(TestAsynchronousLoading::dataLoaded));
//直接加载骨骼动画
ArmatureDataManager::getInstance()->addArmatureFileInfo("armature/bear.
ExportJSON");
```

当加载ExprotJSON的时候，.json文件内部会有一个Plist列表，如果骨骼动画涉及很多个Plist，会自动将多个Plist加载进来，而老版本的Cocos2d-x没有这么智能，只能保证一个.json对应一个Plist，或者在对应多个Plist的时候，先手动将其他关联的Plist加进SpriteFrameCached。

### 29.4.2　创建和播放骨骼动画

传入骨骼动画的名字创建骨骼动画，代码如下：

```
Armature *armature = Armature::create("bear");
```

播放骨骼动画，支持下标和动作名的方式播放，代码如下：

```
armature->getAnimation()->playWithIndex(0);
armature->getAnimation()->play("Idle");
```

一般是骨骼动画的名字等于去除后缀的骨骼动画的文件名，如bear是通过去除.ExportJSON得到的，但是通过修改ExportJSON的文件名，可以直接用新的文件名加载骨骼动画，但.csb格式的骨骼动画的名字是存在.csb文件内部的，因此需要修改.csb格式的骨骼动画名，需要重新导出而不能直接修改。

### 29.4.3　监听事件

监听事件代码如下：

```
/*动画事件监听，可监听3种类型MovementEventType:
  START动画开始
  COMPLETE动画播放完成
  LOOP_COMPLETE循环播放完成（当每次循环播放完之后触发）
*/
armature->getAnimation()->setMovementEventCallFunc( CC_CALLBACK_0(
    TestAnimationEvent::animationEvent,
    this,
    std::placeholders::_1,
    std::placeholders::_2,
    std::placeholders::_3));
//回调函数中传入的3个函数分别是触发该事件的Armature对象、事件类型以及触发的骨骼动画名
void TestAnimationEvent::animationEvent(
    Armature *armature,
    MovementEventType movementType,
    const std::string& movementID)
//帧事件监听，监听动画帧中的事件，事件在动画编辑器中输入
armature->getAnimation()->setFrameEventCallFunc( CC_CALLBACK_0(
    TestFrameEvent::onFrameEvent,
    this,
    std::placeholders::_1,
    std::placeholders::_2,
    std::placeholders::_3,
    std::placeholders::_4));
//传入4个参数分别为发送事件的骨骼、事件ID、起始帧下标和当前帧下标
void TestFrameEvent::onFrameEvent(
cocostudio::Bone *bone,
    const std::string& evt,
    int originFrameIndex,
    int currentFrameIndex)
```

**注意：**代码中的std::placeholders是参数占位的意思，表示回调函数有多少个参数。

### 29.4.4　动态挂载骨骼

将要挂载的对象作为一个骨骼，绑定到骨骼动画当前的某个骨骼中，可以让该对象跟随骨骼动画运动。以下是TestParticleDisplay中的代码，创建了粒子，然后将粒子作为骨骼的显示对象，并切换骨骼的显示列表来显示粒子系统，通过名字查找的方式，动态挂载到Armature对象的骨骼上。示例代码如下：

```
ParticleSystem *p1 = CCParticleSystemQuad::create("Particles/SmallSun.plist");
ParticleSystem *p2 = CCParticleSystemQuad::create("Particles/SmallSun.plist");
//创建一个骨骼，切换到粒子显示，并进行一些设置
cocostudio::Bone *bone = cocostudio::Bone::create("p1");
bone->addDisplay(p1, 0);
bone->changeDisplayWithIndex(0, true);
bone->setIgnoreMovementBoneData(true);
bone->setLocalZOrder(100);
bone->setScale(1.2f);
//添加到骨骼动画对象的bady-a3骨骼下
armature->addBone(bone, "bady-a3");
bone = cocostudio::Bone::create("p2");
bone->addDisplay(p2, 0);
bone->changeDisplayWithIndex(0, true);
bone->setIgnoreMovementBoneData(true);
bone->setLocalZOrder(100);
bone->setScale(1.2f);
armature->addBone(bone, "bady-a30");
```

### 29.4.5　换装功能

一键换装是骨骼的功能，先添加到骨骼的显示列表中，然后再设置骨骼的显示，**forceChangeDisplay强制显示参数为false时，显示一帧便会切回原先的显示，因为骨骼动画中包含帧动画，帧动画会将骨骼切换回原先动画对应显示的帧**；为true时彻底切换为新显示。也可以在编辑器中预先把所有换装的资源拉到骨骼的显示资源列表中，然后在代码中直接切换为想要的显示，但这种方法一般用来做帧动画。示例代码如下：

```
//先用一个数组记录所有装备图片的SpriteFrameName
std::string weapon[] = {
      "weapon_f-sword.png",
      "weapon_f-sword2.png",
      "weapon_f-sword3.png",
      "weapon_f-sword4.png",
      "weapon_f-sword5.png",
      "weapon_f-knife.png",
      "weapon_f-hammer.png"
      };
//遍历这个数组，将装备添加到显示列表中
for (int i = 0; i < 7; i++)
{
      //根据SpriteFrameName创建Skin显示对象并添加到显示列表中
      Skin *skin = Skin::createWithSpriteFrameName(weapon[i].c_str());
      armature->getBone("weapon")->addDisplay(skin, i);
}
//“切换”显示的武器，本质上是换一帧图片，displayIndex的范围是0～6，对应addDisplay
的第二个参数
/*changeDisplayWithIndex的第二个参数是强制切换，如果选择false，则只会有一帧的效
果，动画就会替换回原先的显示*/
/*因为动画在播放时，每一帧都会去更新该骨骼应显示哪一张图片，所以非强制切换，下一帧会被
Animation自动更新回去*/
armature->getBone("weapon")->changeDisplayWithIndex(displayIndex, true);
```

### 29.4.6　骨骼动画的衔接

Cocos2d-x是支持多个骨骼动画进行衔接的，也就是前面说的，一个法师是骨骼动画对象，手持一支法杖也是骨骼动画对象。听起来好像并不难，而且还有相应的接口可以设置，但实际上要使用这个功能却比想象中要复杂得多。

首先作为子节点添加到Armature中并不能起到跟随骨骼的效果，其只会作为一个普通的子节点被添加。添加到某块骨骼下作为子节点，也不会有效果，因为Bone以及Bone的子节点都不会被渲染，渲染骨骼时会渲染的是显示列表中的DisplayRenderNode。Bone中有一个setChildArmature方法，看上去应该是设置衔接Armature的接口，然而，经过测试，这个接口并没有用。

那么如何实现骨骼动画的衔接呢？结合前面的动态挂载骨骼的实现，可得出一种可行的方法，就是创建一个空的骨骼，然后为该骨骼添加一个Armature作为显示节点，最后调用addBone将这块骨骼绑定到骨骼框架中（例如法师的手中）。示例代码如下：

```
//传入一个法师骨骼，创建一个法杖骨骼，并衔接到法师的手中
void takeStaff(Armature* mage)
{
	//创建法杖
	auto armatureStaff = Armature::create("Staff");
	//播放法杖的骨骼动画
	armatureStaff->getAnimation()->playWithIndex(0);
	//创建法杖骨骼
	Bone* childBone = Bone::create("MyChildBone");
	//添加法杖骨骼动画到骨骼的显示列表中，并切换显示
	childBone->addDisplay(armatureStaff, 0);
	childBone->changeDisplayWithIndex(0, true);
	//将骨骼添加到法师的右手中
	mage->addBone(childBone, "rightHand");
}
```

### 29.4.7　骨骼动画的渲染优化

在Cocos2d-x 3.0之前，可以使用BatchNode来优化Armature的渲染。而在Cocos2d-x 3.0之后，自动批次渲染也会帮我们合并Armature的DrawCall。但合并是有条件的，合并的前提可以参考第15章的内容。

为了使骨骼动画的渲染能够被优化，在编辑骨骼动画的过程中，应该尽量不使用不同图片的纹理（在同一图集内视为同一张图片），同时应统一所有骨骼的混合模式。

## 29.5　骨骼动画的碰撞检测

骨骼动画的碰撞检测有两种方法：一种是直接使用矩形检测；另一种是使用ColliderDetector进行碰撞检测。

在骨骼编辑器中，选择骨骼，打开“资源编辑”面板，可以框选骨骼图片对应的碰撞区域，绑定了碰撞区域的骨骼皮肤会生成ColliderDetector，在代码中可以用物理的方法或者手动碰撞检测的方法进行碰撞检测，骨骼绑定的碰撞区域会根据骨骼动画进行更新，如果骨骼对应多帧图片，每一帧图片都可以进行编辑，所以一块骨骼可能拥有很多个ColliderDetector。

### 29.5.1　直接矩形检测

通过获取Armature对象的**getBoundingBox**方法获取当前骨骼动画的包围盒，该方法是实时计算的，可以保证每次获取都是最新的包围盒（每次都会重新计算）。如果需要在循环中和多个矩形进行检测，最好先用一个变量保存包围盒，然后用这个临时变量进行检测，而不要在循环中重复调用getBoundingBox。获取Armature对象指定的骨骼，调用骨骼的getBoundingBox方法，可以判断是否与具体的某一个骨骼碰撞到，Armature的getBoneAtPoint也可以基于点检测碰撞到了哪个骨骼。

关于碰撞检测的坐标系转换，在做碰撞检测的时候，经常需要将一个对象的位置矩阵转换到另一个对象的节点坐标系下，然后再进行点或矩形相交的判断，在这里，两个getBoundingBox可以直接进行判断，如果这两个对象是在同一个父节点下的话；如果不是，则需要将其中一个Rect转换到同一个坐标系下，然后再进行判断。

```
//当两个骨骼对象在同一个parent下
if(armature1->getBoundingBox().intersectsRect(armature2->getBoundingBox
()))
{
    CCLog("armature1 and armature2 intersects");
}
//当骨骼对象和Sprite在同一个parent下
if(armature1->getBoundingBox().intersectsRect(sprite1->getBoundingBox()
))
{
    CCLog("armature1 and sprite1 intersects");
}
//当骨骼对象和Sprite在不同一个parent下（armature1和parent1是同级关系，且parent
没有缩放和旋转）
Rect rect = sprite2->getBoundingBox();
rect.origin += sprite2->getParent()->getPosition();
if(armature1->getBoundingBox().intersectsRect(rect))
 {
    CCLog("armature1 and sprite2 intersects");
 }
//如果parent存在缩放或旋转，那么可以这样来计算rect
Rect rect = sprite2->getBoundingBox();
rect = RectApplyAffineTransform(rect , sprite2->getParent()->getNodeToP
arentAffineTransform());
```

当对象进行缩放时（不管是骨骼还是精灵），BoundingBox也会跟着缩放，每次调用getBoundingBox都会进行矩阵操作，将当前的对象转换到父节点下。所以如果没有发生变化，可以将BoundingBox缓存起来，避免多余的计算。如果只是位置发生变化，可以将变化量叠加到BoundingBox的origin上，以节省运算资源。

### 29.5.2　ColliderDetector手动计算碰撞检测

可以使用ColliderDetector中的顶点列表自行计算碰撞检测，在TestColliderDetector例子中，根据骨骼中的ColliderDetector对象获取ColliderBody列表，再调用getCalculatedVertexList，用顶点构造一个矩形，然后进行矩形碰撞。getCalculatedVertexList的坐标都是经过转换的坐标，处于骨骼对象的父节点坐标系中（在例子中，骨骼对象的父节点是场景节点），可以与同级其他对象的BoundingBox进行碰撞检测。

下面这段代码是TestColliderDetector手动计算碰撞检测的核心代码，在TestColliderDetector::onEnter中创建了两个Armature和一个Sprite，每隔一段时间Sprite重置位置，然后运行一个向右移动的Action，在TestColliderDetector::update中，展示了如何获取ColliderDetector的顶点进行碰撞检测。

```
void TestColliderDetector::update(float delta)
{
  armature2->setVisible(true);
  Rect rect = bullet->getBoundingBox();
  const Map<std::string, cocostudio::Bone*>& map = armature2->getBoneDic();
  for(const auto& element : map)
  {
    //获取骨骼的ColliderDetector
    cocostudio::Bone *bone = element.second;
    ColliderDetector *detector = bone->getColliderDetector();
    if (!detector)
      continue;
    const cocos2d::Vector<ColliderBody*>& bodyList = detector-> get 
ColliderBodyList();
    //一个ColliderDetector中可能有多个形状，遍历所有的形状（形状被封装到Body中）
    for (const auto& object : bodyList)
    {
      //CalculatedVertexList的坐标是相对armature2的父节点的，所以可以直接与兄弟
	 节点bullet进行比较
      ColliderBody *body = static_cast<ColliderBody*>(object);
      const std::vector<Vec2> &vertexList = body->getCalculated VertexList();
      float minx = 0, miny = 0, maxx = 0, maxy = 0;
     size_t length = vertexList.size();
      for (size_t i = 0; i<length; i++)
      {
        Vec2 vertex = vertexList.at(i);
        if (i == 0)
        {
          minx = maxx = vertex.x;
          miny = maxy = vertex.y;
        }
        else
        {
          minx = vertex.x < minx ? vertex.x : minx;
          miny = vertex.y < miny ? vertex.y : miny;
          maxx = vertex.x > maxx ? vertex.x : maxx;
          maxy = vertex.y > maxy ? vertex.y : maxy;
        }
      }
      Rect temp = Rect(minx, miny, maxx - minx, maxy - miny);
      //构造一个AABB盒与armature2的包围盒做碰撞检测
      if (temp.intersectsRect(rect))
      {
        armature2->setVisible(false);
      }
    }
  }
}
```

### 29.5.3　ColliderDetector使用Box2d碰撞检测

使用ColliderDetector还可以借助物理引擎进行碰撞检测，但不建议使用该方法。首先整个骨骼动画对象身上的刚体默认是没有物理效果的，所有的东西都被设置为传感器，也就失去了力的作用（这其实可以理解，是为了避免争抢图片的操作权），因此只能作为一个检测的作用。如果需要非常高精度的检测，那么可以使用，如果要求不高，还是自己计算较好，最主要的原因是启用Box2D进行碰撞检测运行起来不容易，存在各种BUG（有可能是因为版本升级后，被预处理的代码没有及时更新，从而导致各种编译错误）。

TestColliderDetector首先在onEnter中创建一个带有物理属性的子弹，然后调用initWorld，在initWorld中，将全局重力设置为0，防止子弹和人物掉下去，将刚体类型设置为b2_kinematicBody也可以达到这个目的，然后注册碰撞监听器，每次碰撞都会回调ContactListener，在initWorld方法中初始化物理世界，并设置物理属性。

```
void TestColliderDetector::initWorld()
{
  //清除重力
  b2Vec2 noGravity(0, 0);
  world = new b2World(noGravity);
  world->SetAllowSleeping(true);
  //注册碰撞监听
  listener = new (std::nothrow) ContactListener();
  world->SetContactListener(listener);
  debugDraw = new (std::nothrow) GLESDebugDraw( PT_RATIO );
  world->SetDebugDraw(debugDraw);
  uint32 flags = 0;
  flags += b2Draw::e_shapeBit;
  debugDraw->SetFlags(flags);
  //创建刚体
  b2BodyDef bodyDef;
  bodyDef.type = b2_dynamicBody;
  b2Body *body = world->CreateBody(&bodyDef);
  //创建子弹的形状
  b2PolygonShape dynamicBox;
  dynamicBox.SetAsBox(.5f, .5f);
  //设置为传感器的Fixture只用于碰撞检测
  b2FixtureDef fixtureDef;
  fixtureDef.shape = &dynamicBox;
  fixtureDef.isSensor = true;
  body->CreateFixture(&fixtureDef);
  bullet->setB2Body(body);
  bullet->setPTMRatio(PT_RATIO);
  bullet->setPosition(-100, -100);
  //设置刚体到Armature身上，ColliderDetector会自动为其组装Fixture
  body = world->CreateBody(&bodyDef);
  armature2->setBody(body);
}
```

在ContactListener中监听碰撞，这里简化了一下，只使用一个IsCollide变量来判断碰撞。

```
class ContactListener : public b2ContactListener
{
ContactListener()
{
      sCollide=false;
}
  //开始接触
  virtual void BeginContact(b2Contact *contact)
  {
    IsCollide = true;
    B2_NOT_USED(contact);
  }
  //接触结束，两个对象分开
  virtual void EndContact(b2Contact *contact)
  {
    IsCollide = false;
    B2_NOT_USED(contact);
  }
public:
  bool IsCollide;
};
```

在update中需要更新物理世界，当发生碰撞时，隐藏armature2。

```
void TestColliderDetector::update(float delta)
{
  armature2->setVisible(true);
  world->Step(delta, 0, 0);
  if(listener->IsCollide)
  {
    bb->getArmature()->setVisible(false);
  }
}
```

脱离ColliderDetector使用物理碰撞，在不需要精确实时更新的形状，只需要一个大概的形状时，使用物理碰撞并且希望有物理的效果（取消传感器设置），则只需要删除所有骨骼的ColliderDetector（可以在编辑器中清除，也可以通过代码动态设置），然后调用Armature的setBody方法，将形状手动设置到刚体中，这时就不需考虑ColliderDetector的形状了。物理作用会在整个Armature上，但物理和骨骼动画本身是互不影响的，物理作用在Armature上会使其受到力的作用后移动、旋转，但整个骨骼动画对象作为一个整体，其内部播放的动画不会有任何影响。这时的Armature就相当于一个具有物理属性的Sprite了。

### 29.5.4　ColliderDetector的运行机制和结构

下面介绍ColliderDetector的运行机制和结构，以及Armature和物理引擎是如何协作的，这里使用Box2D检测来介绍这个过程，与Chipmuk的流程类似，但接口有些不同，这里不是重点因此不多做介绍。

怎样为Armature附加物理属性？物理应用到动画对象之上会什么样呢？骨骼动画的播放和刚体受力后的运动如何协调工作？

调用Armature的setBody方法，传入b2Body可以为Armature设置刚体对象，我们传入的刚体不需要创建Fixture和形状，ColliderDetector会自动创建整个对象的形状。首先将Body的data属性设置为this，也就是Armature本身，接下来遍历所有的骨骼（不管其是否显示），获取骨骼的显示列表（如换装的武器列表），遍历所有的显示对象，只要其设置了ColliderDetector，则为其设置Body，并且它们都用同一个Body。

```
void Armature::setBody(cpBody *body)
{
  if (_body == body)
  {
    return;
  }
  _body = body;
  _body->data = this;
  for (const auto& object : _children)
  {
    //遍历所有骨骼，不论是否显示
    if (Bone *bone = dynamic_cast<Bone *>(object))
    {
      auto displayList = bone->getDisplayManager()-> getDecorativeDis 	  	  playList();
      //遍历骨骼的整个显示列表，不管当前显示的是哪一个
      for (const auto& displayObject : displayList)
      {
        //只要有ColliderDetector，就把刚体设置进去
        auto detector = displayObject->getColliderDetector();
        if (detector != nullptr)
        {
          detector->setBody(body);
        }
      });
    }
  }
}
```

在ColliderDetector的setBody方法中，会遍历所有的ColliderBody ，在编辑模式下，每添加一个形状（多边形、四边形或圆形）就会增加一个ColliderBody ，形状的数据存放在ColliderBody 中（在初始化显示对象的时候，形状数据就被初始化到ColliderDetector中了，ColliderDetector根据这些数据创建ColliderBody 并添加到成员变量_colliderBodyList中）。这时用每个ColliderBody身上的数据来创建一个多边形，然后使用这个多边形转换到物理坐标，创建一个b2Fixture，并设置为传感器，将当前的骨骼设置到b2Fixture的UserData中，然后将b2Fixture组装到刚体Body中，并将b2Fixture交由ColliderBody 进行管理。如果ColliderBody已经存在b2Fixture，那么会销毁旧的b2Fixture。

```
void ColliderDetector::setBody(b2Body *pBody)
{
  _body = pBody;
  //遍历碰撞触发器所有的形状
  for(auto& object : _colliderBodyList)
  {
    ColliderBody *colliderBody = (ColliderBody *)object;
    //获取它们的顶点数据
    ContourData *contourData = colliderBody->getContourData();
    b2Vec2 *b2bv = new b2Vec2[contourData->vertexList.size()];
    //除以Box2d的转换系数PT_RATIO，转换为Box2d世界的坐标
    int i = 0;
    for(auto& v : contourData->vertexList)
    {
      b2bv[i].Set(v.x / PT_RATIO, v.y / PT_RATIO);
      i++;
    }
    //使用转换后的坐标初始化形状
    b2PolygonShape polygon;
    polygon.Set(b2bv, (int)contourData->vertexList.size());
    CC_SAFE_DELETE(b2bv);
    //设置好形状并指定为传感器，然后组装到刚体身上，并设置当前骨骼为Fixture的
	UserData
    b2FixtureDef fixtureDef;
    fixtureDef.shape = &polygon;
    fixtureDef.isSensor = true;
    b2Fixture *fixture = _body->CreateFixture(&fixtureDef);
    fixture->SetUserData(_bone);
    //将colliderBody的旧骨骼销毁，然后将新骨骼绑定到colliderBody中
    if (colliderBody->getB2Fixture() != nullptr)
    {
      //这里的销毁有个BUG，如果连续setBody两次传入不同的Body会销毁失败
      //因为旧的Fixture是旧的Body的，而在一开始就把旧的Body重置了，新的Body并没	 有该Fixture
      //在最后再设置_body = pBody，然后将新Fixture创建到pBody上，代码就比较严谨了
      _body->DestroyFixture(colliderBody->getB2Fixture());
    }
    colliderBody->setB2Fixture(fixture);
    colliderBody->getColliderFilter()->updateShape(fixture);
  }
}
```

这时会有一个刚体且只有一个刚体，刚体经过了所有骨骼的ColliderDetector初始化，被组装上了各种各样的形状，当播放动画的时候，刚体上的各种形状开始变化，并且跟随着骨骼变化而变化，播放动画的时候骨骼的位置、旋转、缩放等发生了变化，但刚体本身却没有任何位移、旋转缩放产生，而是形状在发生变化，动画产生的变化最后会传递到每块骨骼身上，并生成一个变化矩阵应用到骨骼对应的ColliderDetector上，通过ColliderDetector的updateTransform方法将变化应用到形状上，直接改变刚体的形状而不是驱动刚体运动。

```
void ColliderDetector::updateTransform(Mat4 &t)
{
  if (!_active)
  {
    return;
  }
  for(auto& object : _colliderBodyList)
  {
    ColliderBody *colliderBody = (ColliderBody *)object;
    ContourData *contourData = colliderBody->getContourData();
    //获取形状指针到shape
    b2PolygonShape *shape = nullptr;
    if (_body != nullptr)
    {
      shape = (b2PolygonShape *)colliderBody->getB2Fixture()->GetShape();
    }
    unsigned long num = contourData->vertexList.size();
    std::vector<cocos2d::Vec2> &vs = contourData->vertexList;
#if ENABLE_PHYSICS_SAVE_CALCULATED_VERTEX
    std::vector<cocos2d::Vec2> &cvs = colliderBody->_calculatedVertexList;
#endif
    //遍历顶点数据，重新计算其位置，并应用到shape上
    for (unsigned long i = 0; i < num; i++)
    {
      helpPoint.setPoint( vs.at(i).x, vs.at(i).y);
      helpPoint = PointApplyTransform(helpPoint, t);
#if ENABLE_PHYSICS_SAVE_CALCULATED_VERTEX
      cvs.at(i).x = helpPoint.x;
      cvs.at(i).y = helpPoint.y;
#endif
      //设置shape对应的顶点为新的位置
      if (shape != nullptr)
      {
        b2Vec2 &bv = shape->m_vertices[i];
        bv.Set(helpPoint.x / PT_RATIO, helpPoint.y / PT_RATIO);
      }
    }
  }
}
```

上面的代码中笔者去掉了关于Box2D和Chipmunk的预处理部分，保留了Box2D的内容，这样使得整体代码可读性高一些，ENABLE_PHYSICS_SAVE_ CALCULATED_VERTEX宏用于手动判断碰撞检测，当其开启的时候，每次更新的结果都会被设置到ColliderBody 的_calculatedVertexList中，调用ColliderBody 的getCalculatedVertexList可以返回用于碰撞检测的顶点列表，而在开启Box2D物理碰撞检测时，位置的更新则会被同步到shape的顶点列表中。Box2D和Chipmunk在这里是冲突的，但手动检测和物理检测是不冲突的。顶点计算的点数据是从ColliderBody的contourData中取出的，这个数据是初始化时保存的数据， ColliderDetector每次都使用原始数据来重新计算变化矩阵计算，而不是根据Shape中的顶点进行偏移。