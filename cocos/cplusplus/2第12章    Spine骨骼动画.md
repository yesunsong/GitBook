# 第12章　Spine骨骼动画

Spine是一款针对游戏的2D骨骼动画编辑工具，骨骼动画相比普通的帧动画有着更加流畅的美术效果，能够大大降低游戏安装包的体积，并且制作流程更加高效简洁，动画的播放和切换更加流畅，能够方便地实现换装功能。在程序中还可以通过控制骨骼，来实现很多帧动画无法实现的功能。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Spine功能简介。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Spine结构。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用Spine。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Spine高级技巧。

### 12.1　Spine功能简介

相比CocoStudio（1.x版本）等免费开源的骨骼动画编辑器，Spine有着非常显著的优势，使用过Spine和其他2D骨骼动画编辑工具的美术人员，绝大部分都对Spine有非常高的评价，因为Spine除了强大的功能外，其软件界面非常简洁舒服（如图12-1所示），效果流畅，有很好的用户体验。一个用起来成熟的工具，开发效率自然更高。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120835.jpeg)

图12-1　Spine工具界面

除了大部分骨骼动画编辑工具支持的功能外，Spine还提供了以下实用的功能：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　曲线编辑器，通过调整两个关键帧之间的差值来实现更加自然的动画效果。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　网格Meshes、自由变形FFD、蒙皮Skinning等功能非常强大，可以轻松实现如**拉伸、挤压、弯曲、反弹**等普通矩形图片难以实现的功能，并大大提高了纹理贴图的空间使用率。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　反向动力学工具IK Posing，可以利用反向动力学便捷地调整骨骼动画。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Spine的边界框Bounding Boxes功能，可用于在游戏中实现碰撞检测和物理集成。

Spine支持Unity、Cocos2d-x、Cocos2d等游戏引擎，还支持ActionScript 3、C、C#、JS、Lua等语言。这款工具唯一的缺点就是贵，基础版每年需要支付69美元，专业版每年则需要支付289美元。但也正是有了可靠的收入，Spine才能不断地完善，做得更好。对于商业游戏而言，购买专业版带来的效率提升是很划算的。

本章并不打算介绍如何使用Spine来制作骨骼动画，这里只介绍关于Spine的最基础的内容，以及在程序中使用Spine的方法和技巧。Spine软件的使用，在其官网有详细的文档（大部分是英文的）以及视频（需要翻墙）介绍。网址如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　http://zh.esotericsoftware.com/spine-quickstart；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　http://zh.esotericsoftware.com/spine-getting-started；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　http://zh.esotericsoftware.com/spine-videos。

除了官网之外，泰然网中也有几篇不错的教程，很适合美术人员阅读。在很多Spine中文交流论坛中，也可以找到很多教程。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　http://www.tairan.com/archives/9980/；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　http://www.tairan.com/archives/9981/；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　http://www.tairan.com/archives/9982/。



### 12.2　Spine结构

学习如何在程序中使用Spine之前，先来了解一下Spine的结构，Spine骨骼动画的结构与一般的骨骼动画有些不同，如果不了解其结构，那么在使用Spine时将会碰到很多障碍。

Spine骨骼动画对象由Bone骨骼、Slot插槽、Attachment附件、Skin皮肤以及Animation动画等部分组成，如图12-2所示为Spine骨骼动画中各个部分的关系。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Bone骨骼：骨骼是Spine骨骼动画的基本组成元素，每块骨骼都有它的名字和长度，可以用于移动、旋转和缩放。骨骼之间存在父子关系，父骨骼的运动会带动其子骨骼。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Slot插槽：插槽主要用于关联附件，骨骼本身并没有实现显示功能，每块骨骼可以关联多个插槽，通过插槽所关联的附件来显示，而每个插槽可以关联多个附件，但**同一时间只能激活一个附件，只有被激活的附件才会显示出来**。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Attachment附件：附件主要用于显示，Spine支持的附件类型有Region附件（图片）、Mesh附件（网格）、BoundingBox附件（边界框）、SkinnedMesh附件（蒙皮网格）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120836.jpeg)

图12-2　Spine骨骼动画结构图

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Skin皮肤：皮肤的作用是重用骨骼和动画，通过一套新的附件，来切换骨骼动画的表现。例如官方例子中的哥布林（游戏中的一种怪物），哥布林包含了两个皮肤，男性哥布林和女性哥布林（如图12-3所示），使用同一套骨骼框架和动画，通过切换皮肤可以切换不同的哥布林形象。**每个皮肤会对应一系列插槽以及插槽中相对的附件，当应用一个皮肤时，在骨骼动画对象中，该皮肤对应的所有插槽以及插槽中相对的附件都会被替换为当前皮肤所对应的附件**，默认情况下是没有皮肤的。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120837.jpeg)

图12-3　切换骨骼皮肤

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Animation动画：播放动画可以让骨骼动画对象运动起来，动画包含了一系列的时间轴，即骨骼时间轴（描述骨骼如何运动）、插槽时间轴（描述了颜色以及附件的变化，可实现帧动画）、事件时间轴（记录了动画中触发的事件，以及事件携带的参数）、渲染顺序时间轴（描述了插槽渲染顺序的变化）。**一个骨骼动画对象可以拥有多个动画，并且也可以同时播放多个动画，通过动画混合技术**，例如一个角色拥有移动和射击两个动画，这两个动画可以被同时播放。



### 12.3　使用Spine

在了解了Spine骨骼动画对象之后，可以开始在Cocos2d-x中使用Spine，掌握了在Cocos2d-x中使用Spine的方法，在其他引擎或语言中使用Spine也会变得轻松。本节会介绍Spine最基础的用法，包括加载Spine、播放动画、监听动画回调、切换表现等。

在使用Spine之前，需要设置对应的头文件搜索路径，并指定libSpine.lib链接库，这是每使用一个新的库都必须执行的步骤（如果项目已经设置好了则不需要）。下面介绍的例子都是基于Cocos2d-x引擎自带的cpp-tests项目中的Spine示例。

#### 12.3.1　加载Spine

Spine可以导出JSON格式和skel二进制格式，~~目前Cocos2d-x中只支持JSON格式，~~Spine可以导出3种JSON格式，分别是JSON、JavaScript、Minimal（如图12-4所示），**在Cocos2d-x中对应可以加载的是JSON格式**，如果导出其他格式，Cocos2d-x在加载时会崩溃。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120838.jpeg)

图12-4　JSON格式

在导出选项的最下方，有一个Create atlas选项，当选中该选项时，会将当前骨骼动画用到的图片打包到一张图集中。

一般来说，Spine导出一般需要生成3个文件，数据文件.json或.skel，图集文件.atlas（相当于Plist图集），图片文件PNG。<u>Spine允许有**多个不同的骨骼动画对象来共用一个图集**</u>。另外需要注意的是，**图集文件和其对应的图片文件需要被放在同一个目录下**。

在Cocos2d-x中，要创建Spine骨骼动画，只需要包含spine/spine.h头文件（位于引擎目录下的cocos/editor-support目录中），然后调用SkeletonAnimation的createWithFile()静态方法，传入骨骼数据文件以及对应的图集文件即可。被创建的骨骼动画对象可以通过addChild()方法添加到游戏场景中。

SkeletonAnimation是Cocos2d-x中的Spine骨骼动画对象，SkeletonAnimation继承于SkeletonRenderer，而SkeletonRenderer继承于Node和BlendProtocol，在SkeletonRenderer中实现了自定义的骨骼动画渲染。

```c++
#include "spine/spine.h"
SkeletonAnimation *skeletonNode = SkeletonAnimation::createWithFile(
"spine/spineboy.json", "spine/spineboy.atlas");
addChild(skeletonNode );
```



#### 12.3.2　播放动画

SkeletonAnimation的setAnimation()和addAnimation()方法可以播放动画，在学习这两个方法之前，需要了解一下spTrackEntity，<u>spTrackEntity</u>可以用于执行动画，相当于是动画运行时的对象。

SkeletonAnimation可以拥有多条Track（可以译为轨道，这是一个下标从0开始的数组，也就是spTrackEntity的容器），<u>每一条Track在同一时间内可以执行一个动画，但一条Track可以对应多个动画，例如，下标为0的Track当前播放动画A，3秒后将播放动画B。</u>

setAnimation()函数可以设置指定下标的Track当前的动画，并返回该动画对应的spTrackEntity对象。addAnimation()函数则可以设定指定下标的Track在一段时间后会执行的动画，并返回该动画对应的spTrackEntity对象。

```c++
//下面是播放动画相关的接口
//设置指定Track当前播放的动画，trackIndex为Track下标，name为骨骼动画名，loop为是否循环
spTrackEntry* setAnimation (int trackIndex, const std::string& name, bool
loop);

//在指定的Track上添加一个待播放的动画，trackIndex为Track下标，name为骨骼动画名，loop为是否循环，delay为动画等待的时间，delay的单位为秒
spTrackEntry* addAnimation (int trackIndex, const std::string& name, bool
loop, float delay = 0);

//获取当前正在播放的spTrackEntry对象，trackIndex为Track下标
spTrackEntry* getCurrent (int trackIndex = 0);

//清除所有的Track，所有的动画都会被停止
void clearTracks ();

//清除指定的Track
void clearTrack (int trackIndex = 0);

//setTimeScale可以控制动画播放的速度，默认值为1.0f，该值越大则动画播放速度越快，反之则越慢
void setTimeScale(float scale);
float getTimeScale() const;
```

当在切换动画的时候，从动画A的当前帧直接跳到动画B的第一帧，这中间看上去会有些突兀，所以Spine提供了插值算法可以让两个动画的切换变得平滑流畅，在播放动画之前，可以调用SkeletonAnimation的setMix()方法来设置两个动画切换时的过渡时间。

```c++
//setMix()方法可以设置从动画A切换到动画B的插值过渡时间，fromAnimation为动画A，toAnimation为动画B，duration为过渡时间
void setMix(const std::string&fromAnimation, const std::string&
toAnimation, float duration);
```

在切换Spine动画的时候，有时候会碰到<u>**残影问题**</u>，也就是上一个动画中的一些动作会停留在屏幕上，这种情况只需要在每次切换动画前调用一下骨骼动画对象的setToSetupPose()方法即可。

如果<u>切换时出现播放了其他动画的帧，导致动画切换时出现闪烁的现象</u>，那么可以通过调用spAnimationState_apply()方法来处理，该方法会平滑地执行一帧，从而跳过闪烁的那一帧。注意调用前判断spTrackEntry*是否为空。

```c++
//去残影
setToSetupPose();
spTrackEntry* entry=setAnimation(trackIndex, name, loop);
//如果指定的name找不到，setAnimation失败，就会导致spAnimationState_apply 崩溃，所以加个判断
if(entry){
spAnimationState_apply(_state, _skeleton);
}
```



#### 12.3.3　动画回调

可以为SkeletonAnimation对象设置回调函数，Spine支持两类回调，一类是全局回调，另一类是绑定到指定spTrackEntity对象的回调。Spine会在动画开始、动画结束、动画完成（一次循环）、帧事件触发这4种情况下执行设置的回调，这4种回调对应的函数原型如下。

```c++
//动画开始回调
typedef std::function<void(int trackIndex)> StartListener;
//动画结束回调
typedef std::function<void(int trackIndex)> EndListener;
//动画完成回调
typedef std::function<void(int trackIndex, int loopCount)>
CompleteListener;
//帧事件回调
typedef std::function<void(int trackIndex, spEvent* event)> EventListener;
```

通过调用SkeletonAnimation的以下成员函数可以设置指定的回调函数。

```c++
//以下4个函数设置的是全局回调，每个Track的动画触发相应的事件都会执行这些回调
//设置动画开始回调
void setStartListener (const StartListener& listener);

//设置动画结束回调
void setEndListener (const EndListener& listener);

//设置动画完成回调
void setCompleteListener (const CompleteListener& listener);

//设置帧事件回调
void setEventListener (const EventListener& listener);

//以下4个函数设置的是指定spTrackEntry的回调，只有指定的spTrackEntry触发相应的事件才会被执行
//设置动画开始回调
void setTrackStartListener (spTrackEntry* entry, const StartListener& listener);

//设置动画结束回调
void setTrackEndListener (spTrackEntry* entry, const EndListener& listener);

//设置动画完成回调
void setTrackCompleteListener (spTrackEntry* entry, const CompleteListener& listener);

//设置帧事件回调
void setTrackEventListener (spTrackEntry* entry, const EventListener& listener);
```

帧事件可以携带3个不同类型的参数，分别是string、int以及float，这些参数可以在Spine编辑器中设置，设置好事件信息之后，可以在事件时间轴中触发事件。

在引擎自带的cpp-tests中的SpineTest示例中，可以找到SpineTestLayerNormal示例，该示例演示了前面介绍的各种播放动画相关接口的使用。

```c++
bool SpineTestLayerNormal::init () {
    if (!SpineTestLayer::init()) return false;
    //创建一个骨骼动画
    skeletonNode = SkeletonAnimation::createWithFile("spine/spineboy.json", "spine/spineboy.atlas", 0.6f);
    skeletonNode->setScale(0.5);
    //设置回调
    skeletonNode->setStartListener( [this] (int trackIndex) {
        spTrackEntry* entry = spAnimationState_getCurrent(skeletonNode->
        getState(), trackIndex);
        const char* animationName = (entry && entry->animation) ? entry->
        animation->name : 0;
        log("%d start: %s", trackIndex, animationName);
    });
  
    skeletonNode->setEndListener( [] (int trackIndex) {
        log("%d end", trackIndex);
    });
  
    skeletonNode->setCompleteListener( [] (int trackIndex, int loopCount)
{
        log("%d complete: %d", trackIndex, loopCount);
    });
  
    skeletonNode->setEventListener( [] (int trackIndex, spEvent* event) {
        log("%d event: %s, %d, %f, %s", trackIndex, event->data->name,
        event->intValue, event->floatValue, event->stringValue);
    });
  
    //设置动画插值，从walk切换到jump动画使用0.2秒的时间过渡
    skeletonNode->setMix("walk", "jump", 0.2f);
  
    skeletonNode->setMix("jump", "run", 0.2f);
  
//在下标为0的Track上循环播放walk动画，并在3秒之后切换到jump动画，在完成jump动画的播放之后，立即切换到run动画
skeletonNode->setAnimation(0, "walk", true);
  
spTrackEntry* jumpEntry = skeletonNode->addAnimation(0, "jump", false,3);
  
skeletonNode->addAnimation(0, "run", true);

  //设置jump动画的开始回调
skeletonNode->setTrackStartListener(jumpEntry, [] (int trackIndex) {
        log("jumped!");
});
  
//设置骨骼动画对象的位置，并添加到场景中
    Size windowSize = Director::getInstance()->getWinSize();
    skeletonNode->setPosition(Vec2(windowSize.width / 2, 20));
    addChild(skeletonNode);
    scheduleUpdate();
  
    //添加一个点击回调，在点击时切换调试骨骼信息的显示，以及动画播放的速度
    EventListenerTouchOneByOne* listener = EventListenerTouchOneByOne::
    create();
    listener->onTouchBegan = [this] (Touch* touch, Event* event) -> bool {
        if (!skeletonNode->getDebugBonesEnabled())
            skeletonNode->setDebugBonesEnabled(true);
        else if (skeletonNode->getTimeScale() == 1)
            skeletonNode->setTimeScale(0.3f);
        else
        {
            skeletonNode->setTimeScale(1);
            skeletonNode->setDebugBonesEnabled(false);
        }
        return true;
    };
    _eventDispatcher->addEventListenerWithSceneGraphPriority(listener,
    this);
    return true;
}
```

在我们点击屏幕时，会触发点击回调，在点击回调中设置了骨骼动画的播放速度，当调用setTimeScale()函数设置了时间缩放之后，调用addAnimation()函数添加进去的，在N秒后播放的动画也会受到影响，如将时间放慢一倍，那么原本设置3秒后执行的动画会等到6秒后才执行。

点击回调还开启、关闭了骨骼调试模式，在调试模式下可以看到骨骼动画对象的骨骼结构。



#### 12.3.4　显示控制

##### 1．切换皮肤

在了解了如何播放Spine骨骼动画之后，下面进一步地控制骨骼动画的显示，切换骨骼动画表现的常用做法就是设置皮肤，调用SkeletonAnimation骨骼动画对象的setSkin()方法，传入皮肤的名称，即可切换表现，当然，皮肤需要美工在制作Spine动画的时候设置。在SpineTest的SpineTestLayerFFD中，调用setSkin()方法时传入了goblin皮肤名，如果我们传入goblingirl皮肤名，则会显示成女性哥布林角色。



##### 2．切换附件

除了切换皮肤之外，还可以切换某个插槽中当前显示的附件来切换局部的表现，例如，当需要将哥布林手中的长矛切换为其他武器，在Spine默认的哥布林资源中，其左手拿着的武器对应的插槽（名为left hand item）中有着两个附件，分别是一把刀（名为dagger）和一把长矛（名为spear）。

调用SkeletonAnimation的setAttachment()方法，传入插槽的名字和要切换的附件的名字，即可完成这个切换。



##### 3．挂载物体到骨骼

在使用骨骼动画时，可能有这样的需求，就是在指定的骨骼上绑定一些额外的对象，例如，在哥布林的双脚上绑定两个粒子效果，在哥布林播放动画的时候，让粒子效果跟随哥布林的双脚移动。这个功能实现起来并不是很轻松，因为在获取哥布林脚部的骨骼对象之后，并不能直接将粒子对象作为子节点添加到哥布林脚部的骨骼对象上。

在获取哥布林脚部的骨骼对象之后，只能在每一帧中根据获取的骨骼坐标，来更新粒子对象的坐标，从而模拟出粒子跟随骨骼的效果（没错，这是官方推荐的做法）。<u>在骨骼中获取的坐标所在的坐标系，是以骨骼动画对象为原点的坐标系，也就是在骨骼动画的节点空间中的位置</u>，我们需要将粒子对象添加到骨骼动画上，作为骨骼对象的子节点，然后在每一帧的update中，获取骨骼的位置，然后更新到粒子对象中。

当然，如果要挂载在其他节点下也是可以的，只是需要转换一下坐标系，将骨骼对象中的节点坐标转换到其他目标节点的节点空间坐标系中。

这里将SpineTest的SpineTestLayerFFD例子的代码调整了一下，在代码中演示了上面介绍的切换皮肤、切换附件以及跟随骨骼等功能。程序运行效果如图12-5所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200628105238.jpeg)

图12-5　骨骼上绑定粒子对象

```c++
bool SpineTestLayerFFD::init () {
    if (!SpineTestLayer::init()) return false;
    //创建哥布林骨骼动画
    skeletonNode = SkeletonAnimation::createWithFile("spine/ goblins-ffd.json", "spine/goblins-ffd.atlas", 1.5f);
    skeletonNode->setAnimation(0, "walk", true);
  
    //设置皮肤
    //skeletonNode->setSkin("goblin");
    skeletonNode->setSkin("goblingirl");
  
    //切换手上的武器附件为刀子
    skeletonNode->setAttachment("left hand item", "dagger");
  
    //设置位置，并添加到场景中，将骨骼动画对象的TAG设置为100，方便获取
    skeletonNode->setScale(0.4f);
    Size windowSize = Director::getInstance()->getWinSize();
    skeletonNode->setPosition(Vec2(windowSize.width / 2, 50));
    addChild(skeletonNode, 0, 100);
  
    //创建两个粒子效果，添加到骨骼动画对象身上，TAG分别为111和222
    auto p1 = ParticleGalaxy::create();
    p1->setTexture(Director::getInstance()->getTextureCache()->
    addImage("Images/fire.png"));
    p1->setScale(0.2f);
    skeletonNode->addChild(p1, 0, 111);
  
    auto p2 = ParticleGalaxy::create();
    p2->setTexture(Director::getInstance()->getTextureCache()->
    addImage("Images/fire.png"));
  
    p2->setScale(0.2f);
    skeletonNode->addChild(p2, 0, 222);
    //开启update监听
    scheduleUpdate();
    return true;
}

void SpineTestLayerFFD::update (float deltaTime) {
    //每一帧都获取骨骼动画对象
    auto skeletonNode = dynamic_cast<SkeletonAnimation*>(getChildByTag(100));
    //找到两只脚对应的骨骼
    auto leftBone = skeletonNode->findBone("left foot");
    auto rightBone = skeletonNode->findBone("right foot");
  
    //设置粒子效果的位置
    skeletonNode->getChildByTag(111)->setPosition(leftBone->worldX,
    leftBone->worldY);
    skeletonNode->getChildByTag(222)->setPosition(rightBone->worldX,
    rightBone->worldY);
}
```

骨骼中除了坐标之外，还附带了旋转角度，缩放值以及矩阵等信息，来帮助正确地显示需要跟随的骨骼对象。



##### 4．翻转骨骼

除了上述操作之外，还经常会用到的一个操作就是翻转，但<u>Spine骨骼动画对象并没有提供flip相关的方法，当需要对骨骼对象进行翻转时，需要通过设置缩放的方式实现</u>。例如，希望实现x轴方向上的翻转，需要调用setScaleX将骨骼对象的x轴缩放值乘以-1，y轴方向上的旋转与x轴的旋转类似。



### 12.4　Spine高级技巧

本节介绍一些实用的高级技巧，包括混合动画、Spine缓存与异步加载Spine动画，以及优化Spine的一些技巧。

#### 12.4.1　混合动画

Spine的混合动画功能支持Spine骨骼动画对象同时播放多个骨骼动画，如一边移动一边瞄准射击，<u>要实现混合动画非常简单，只需要在两个不同Track播放不同的动画即可</u>。但需要注意的是<u>被混合的动画之间是不能互相冲突的</u>，例如移动和跑步这两个动画混合在一起，效果可能就比较糟糕了。

另外，<u>当播放混合动画之后，需要切换到某个独立的动画，例如，从一边移动一边射击的混合动画切换到死亡，那么应该先调用骨骼动画的clearTracks()方法将Track进行清除，再关闭混合动画。</u>

SpineTest中的SpineTestLayerRaptor演示了混合动画，该例子对应了一个骑着恐龙的枪手，枪手拔枪瞄准的动画与恐龙的移动动画进行了混合，运行该例子可以发现，在恐龙移动2秒之后，枪手拔出了手枪，拔枪动画与恐龙移动的动画同时播放，效果非常和谐。

```c++
bool SpineTestLayerRapor::init () {
    if (!SpineTestLayer::init()) return false;
    //创建骨骼动画，在下标为0的Track上播放恐龙移动的walk动画
    skeletonNode = SkeletonAnimation::createWithFile("spine/raptor.json",
    "spine/raptor.atlas", 0.5f);
    skeletonNode->setAnimation(0, "walk", true);
    //在下标为1的Track上先设置一个空动画，然后再添加一个2秒后执行的拔枪动画
    //如果不先设置一个空的动画，那么addAnimation添加的动画会被立即执行
    skeletonNode->setAnimation(1, "empty", false);
    skeletonNode->addAnimation(1, "gungrab", false, 2);
    skeletonNode->setScale(0.5);
    Size windowSize = Director::getInstance()->getWinSize();
    skeletonNode->setPosition(Vec2(windowSize.width / 2, 20));
    addChild(skeletonNode);
    scheduleUpdate();
   return true;
}
```



#### 12.4.2　缓存Spine骨骼动画

当Spine骨骼动画比较复杂的时候，在创建Spine动画时可以感觉到明显的卡顿，因为每次调用SkeletonAnimation的createWithFile()方法，都会去加载骨骼文件和图集文件并进行解析，对于复杂的Spine骨骼动画，这个解析的步骤也是挺耗时间的。所以可以**将Spine的骨骼动画数据预加载之后进行缓存**，在每次创建Spine骨骼动画时，根据缓存好的数据进行加载。这是一个非常实用的技巧！不过在Cocos2d-x 3.14之后，可以使用3.5版本的Spine工具编辑导出skel格式的骨骼，这个格式加载非常快。

我们可以在Spine的SkeletonRenderer::initWithFile()函数中找到Spine骨骼动画的初始化流程，该流程的步骤大致如下。

（1）调用spAtlas_createFromFile()函数创建spAtlas对象。

（2）调用spSkeletonJson_createWithLoader()函数传入spAtlas对象，创建spSkeletonJson对象。

（3）调用spSkeletonJson_readSkeletonDataFile()函数传入spSkeletonJson和骨骼文件名，创建spSkeletonData对象。

（4）调用spSkeletonJson_dispose()函数释放spSkeletonJson对象。

（5）调用setSkeletonData成员函数传入spSkeletonData对象，初始化spSkeleton骨骼对象。

（6）调用initialize成员函数初始化其余内容（shader、blend、batch等）。

上述步骤中，（1）～（4）步创建了spSkeletonData骨骼数据对象，可以直接使用该数据对象来创建SkeletonAnimation对象，通过调用SkeletonAnimation::createWithData()方法，传入骨骼数据对象，用这种方法来创建Spine骨骼动画对象的效率比SkeletonAnimation::createWithFile()方法要高很多。这种方法相当于省略了前面的4个步骤。

步骤（1）创建了Atlas图集对象，这时涉及了PNG图片的加载，图集对应的图片会通过TextureCache的addImage()方法加载，所以当重复创建图集对象时，并不会重复地加载纹理，因为纹理被TextureCache缓存了起来。但Atlas并没有被缓存，所以每次都会重新从磁盘中读取Atlas图集文件，并解析成spAtlas对象，这个过程会有一定的消耗。

步骤（2）会将spAtlas文件转换成spSkeletonJson对象，步骤（3）会根据spSkeletonJson对象以及骨骼动画文件创建骨骼数据，也就是spSkeletonData对象，这一步骤会加载硕大的骨骼文件，并进行复杂的解析计算，是骨骼动画加载流程的一个瓶颈。

步骤（4）将已经没用的spSkeletonJson对象释放，Spine中每一个用C函数create出来的对象，都需要调用对应的dispose()方法来释放内存。这里创建了3个对象，分别是spAtlas、spSkeletonJson以及spSkeletonData对象，其中spAtlas和spSkeletonData对象需要到最后才可以释放。这里的最后指的是所有使用该骨骼数据创建的Spine骨骼动画对象都已经被释放，且短时间内不需要再创建这个骨骼动画对象。一般是在切换场景的时候做这个工作。

下面的代码简单演示了如何使用spSkeletonData对象来创建多个骨骼动画，可以使用容器将spSkeletonData和spAtlas缓存起来，然后在适当的时候释放它们，下面的代码是连续的，也可以把它们进行简单的封装和管理，这里就不多介绍了。

```
//在开始时创建atlas与skeletonData用于后面创建骨骼动画
spAtlas* atlas = spAtlas_createFromFile(atlasFile.c_str(), 0);
spSkeletonJson* json = spSkeletonJson_create(atlas);
spSkeletonData* skeletonData = spSkeletonJson_readSkeletonDataFile(json,
skeletonDataFile.c_str());
spSkeletonJson_dispose(json);
//使用skeletonData创建多个骨骼动画
auto sp1 = SkeletonAnimation::createWithData(skeletonData);
addChild(sp1);
auto sp2 = SkeletonAnimation::createWithData(skeletonData);
addChild(sp2);
auto sp3 = SkeletonAnimation::createWithData(skeletonData);
addChild(sp3);
//当这些骨骼动画使用完之后会被删除
removeChild(sp1);
removeChild(sp2);
removeChild(sp3);
//在最后释放前面申请的内存
//如果删除时，还有骨骼动画对象引用这些骨骼数据，那么程序会崩溃
spSkeletonData_dispose(skeletonData);
spAtlas_dispose(atlas);
```

#### 12.4.3　异步加载Spine骨骼

前面介绍了如何预加载Spine的骨骼数据，在使用时直接根据骨骼数据来创建Spine骨骼动画对象。可以大大提高创建Spine骨骼动画对象的效率，但在预加载Spine时，仍然会有严重的卡顿现象，解决这种问题的最佳方法就是异步加载，本节将会给出一个安全可靠的Spine异步加载方案。

谈到异步加载，不得不提起线程安全，非常不幸的是，目前Cocos2d-x的TextureCache的addImageAsync()方法并不是线程安全的，在解析图片的时候，会调用Image类的initWithImageFileThreadSafe()方法来读取图片文件并进行解析。加载图片用的是FileUtils的getDataFromFile()方法，该方法会调用FileUtils的fullPathForFilename()方法，在这里有可能会触发_fullPathCache容器的insert操作（当第一次读该文件时会将文件的完整路径插入），当两个线程同时操作了这个容器，那么程序就有可能崩溃。

笔者认为这个线程安全的BUG应该是在后面优化的时候新增的BUG，但因为相对于整个图片异步加载而言，这段代码被两条线程同时执行的概率非常低，所以这个BUG被隐藏得非常深。

知道了问题所在，自然就有解决问题的方法，在不修改引擎源码的前提下（这种事情做了心里会感觉很难受），可以将要加载的所有文件名（写一个简单的程序，遍历资源目录，将资源目录下的所有文件的路径写入一个配置文件中，这并不麻烦），都在主线程中先调用fullPathForFilename()方法来让它插入到_fullPathCache容器中，然后在开始异步加载时，主线程不要加载**新的**文件（这个要求并不过分）。这样就可以保证在加载的时候不会对_fullPathCache容器执行insert操作，多个线程对一个容器执行find操作是安全的。

常用的做法还包括先将待加载的资源添加到一个**待加载**（即还没开始加载）的列表中（并执行fullPathForFilename()方法），然后调用一个开始加载的方法，来开启线程。

接下来介绍Spine线程的异步加载，幸运的是**Spine骨骼的解析是线程安全的**，所以可以很轻松地将这部分代码放到线程中，但**创建spAtlas的操作则不是线程安全的**（因为里面用到了create方法，该方法会往自动回收池中插入数据），所以可以将Spine的加载分为3个步骤，如图12-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120840.jpeg)

图12-6　Spine异步加载流程

具体步骤如下。

（1）调用TextureCache的addImageAsync()方法异步加载纹理。

（2）完成纹理加载后回调主线程中的方法，创建spAtlas对象。

（3）在自己的线程中，获取spAtlas对象，解析骨骼数据，创建spSkeletonData对象，并添加到容器中。

（4）只要通知到主线程，就可以使用线程加载好的骨骼数据对象。

接下来介绍Spine异步加载的代码实现，这里实现了一个简单的CSpineLoader类用于异步加载Spine，首先从整体上介绍一下类的功能和结构，方便理解接下来要介绍的异步加载流程。

首先定义了一个回调函数的原型，用于在完成Spine的加载后进行回调，另外还定义了一个简单的结构体，用于存储一些变量，方便在线程之间进行交互。

```
//Spine加载完成回调
typedef std::function<void(spSkeletonData*)> ResLoadedCallback;
//Spine加载信息——用于在线程间安全地传递数据
struct SpineLoadingInfo
{
    std::string JsonFile;
    spAtlas* Atlas;
    spSkeletonData* SkeletonData;
    ResLoadedCallback Callback;
};
```

接下来是CSpineLoader这个类，除了异步加载的功能，CSpineLoader类还提供了骨骼的缓存和管理功能，并定义了一些相关的成员变量，这些变量在异步加载的流程中发挥了重要的作用。

```
class CSpineLoader
{
public:
   CSpineLoader();
    virtual ~CSpineLoader();
    //预加载资源
    virtual bool loadSpineAsyn(const std::string& resName, const std::
    string& atlasName, const ResLoadedCallback& callback);
    //获取一个资源
    spSkeletonData* getSkeletonData(const std::string& resName);
    //移除一个资源
    virtual void removeRes(const std::string& resName);
    //清除所有资源
    virtual void clearRes();
private:
    //加载结束时调用
    void onFinish();
    //骨骼加载线程
    void skeletonThread();
    //开启骨骼加载线程
    void startSkeletonThread();
private:
    bool m_bThreadWorking;                         //线程是否工作
    int m_nMaxSpineCount;                          //最大加载数量
    int m_nSkeletonLoadingIndex;                   //当前解析完成的骨骼下标
    int m_nAtlasLoadingIndex;                      //当前解析完的图集下标
    std::thread* m_SkeletonThread;                 //骨骼解析线程
    SpineLoadingInfo* m_LoadingList;               //异步加载队列
std::set<std::string> m_LoadingSpine; 	//正在加载的Spine
std::map<std::string, spSkeletonData*> m_SpineCache;
                                                   //加载完成的Spine骨骼
};
```

接下来介绍具体加载流程的实现，首先需要调用CSpineLoader的loadSpineAsyn()方法，将要加载的Spine骨骼文件、Atlas图集文件以及加载完成的回调传入。如果Spine正在加载，那么loadSpineAsyn会直接返回false（更好的处理是将callback添加到一个队列中，在加载完成后一并处理，就如TextureCache的addImageAsync()方法一样的处理，但为了简化问题，这里不进行该处理）。如果Spine已经加载完成，那么会直接执行回调函数。

如果**当前加载队列**的数量大于等于MAX_LOADING_LIST常量（该常量默认为256，可自行修改），则直接返回失败。这里解释一下加载队列这个概念，当开始异步加载时，要异步加载的Spine信息需要进入到一个队列中，因为可以在一个for循环中加载N个Spine骨骼，骨骼解析线程同时只可以解析一个骨骼，那么其他的骨骼就需要在队列中排队等待解析。

当**队列中所有的骨骼都加载完成后**，这个队列会被重置，也就是说，MAX_LOADING_LIST常量指的是最多允许连续加载的Spine骨骼数量。例如，从A场景切换到B场景时，需要加载N个骨骼，这个N不能大于MAX_LOADING_LIST常量。而当从B场景切换到A场景时，需要加载M个骨骼，这个M同样不能大于MAX_LOADING_LIST常量。

```
bool CSpineLoader::loadSpineAsyn(const std::string& resName, const
std::string& atlasName, const ResLoadedCallback& callback)
{
    //如果该Spine已经在加载中，或者该Spine文件为空则返回失败
    string fullJsonPath = FileUtils::getInstance()->fullPathForFilename
   (resName);
   if (fullJsonPath.empty()
       || m_LoadingSpine.find(fullJsonPath) != m_LoadingSpine.end())
   {
       CCLOG("CSpineLoader::loadSpineAsyn %s is Loading or Loaded",
       resName.c_str());
       return false;
   }
   //如果已经加载完成
   if (m_SpineCache.find(fullJsonPath) != m_SpineCache.end())
   {
       if (callback != nullptr)
       {
           callback(m_SpineCache[fullJsonPath]);
       }
       return true;
   }
   //连续加载超过最大限制，则返回失败
   if (MAX_LOADING_LIST <= m_nMaxSpineCount)
   {
       CCLOG("CSpineLoader::loadSpineAsyn Load Too Many Spine");
       return false;
   }
   //根据Atlas文件路径生成同名的图片文件路径（例如a.atlas -> a.png）
   int pos = atlasName.find_last_of('.');
   string fullImgPath = atlasName.substr(0, pos + 1) + "png";
   fullImgPath = FileUtils::getInstance()->fullPathForFilename
   (fullImgPath);
   //插入加载中的队列
   m_LoadingSpine.insert(fullJsonPath);
   //m_nMaxSpineCount默认为0，表示要加载的Spine骨骼的数量
   ++m_nMaxSpineCount;
   string fullAtlas = FileUtils::getInstance()->fullPathForFilename
   (atlasName);
   //异步加载纹理，加载完成后将相关信息设置到队列中，并自增图集加载下标
   Director::getInstance()->getTextureCache()->addImageAsync
   (fullImgPath, [this, fullJsonPath, fullAtlas, callback](Texture2D* tex)
   {
       SpineLoadingInfo* spineInfo = &m_LoadingList[m_nAtlasLoadingIndex];
       spineInfo->Atlas = spAtlas_createFromFile(fullAtlas.c_str(), 0);
       assert(spineInfo->Atlas);
       spineInfo->Callback = callback;
       spineInfo->JsonFile = fullJsonPath;
       ++m_nAtlasLoadingIndex;
       //惰性开启线程
       startSkeletonThread();
   });
   return true;
}
```

在loadSpineAsyn()函数中，最后调用了addImageAsync()方法来异步加载纹理，纹理加载完成之后创建了spAtlas对象，因为spAtlas_createFromFile()方法并不是线程安全的（在线程中完成该步骤程序可能崩溃），因此将创建好的spAtlas对象设置到SpineLoadingInfo中，并将m_nAtlasLoadingIndex自增1。SpineLoadingInfo是从m_LoadingList中取出来的，这是我们的加载队列，是一个数组，通过偏移的方法，取出相对应下标的SpineLoadingInfo指针，并将数据设置进去。m_LoadingList在构造函数中就被创建了，在析构时释放。

在执行时，addImageAsync()方法的调用并不会保证顺序（当纹理已经被加载时，回调会立刻被执行），所以真正地添加队列，是在addImageAsync()方法的回调中执行。与一般的添加队列不同，这里的添加并没有像一般使用队列时，new一个元素动态添加进去，而是在一个已经分配好内存的数组中，直接设置数组元素的属性，这样是为了方便实现安全的**无锁队列**。

完成了Atlas的创建以及将相关信息添加到队列中后，调用startSkeletonThread()方法开启线程，这个方法会在第一次加载时创建线程，在加载完所有Spine时会结束线程，而当再次加载骨骼时，又会重新创建线程。

```
void CSpineLoader::startSkeletonThread()
{
    if (m_SkeletonThread == nullptr)
    {
        //开启线程
        m_bThreadWorking = true;
        m_SkeletonThread = new std::thread(&CSpineLoader::skeletonThread,
        this);
    }
}
```

SkeletonThread()为我们的线程函数，函数中是一个while循环，不断地判断m_nAtlasLoadingIndex是否大于m_nSkeletonLoadingIndex，由于这两个变量初始化都为0，当spAtlas创建完并设置到m_LoadingList队列之后，才将m_nAtlasLoadingIndex自增1。条件成立时说明有骨骼可以开始解析了，解析完的骨骼数据也被设置到m_LoadingList中，然后调用Schedule的performFunctionInCocosThread()方法，在**主线程中执行加载完成的处理**。当条件不成立时，线程进行短暂的sleep，让出时间片给其他线程执行。

```
void CSpineLoader::skeletonThread()
{
    while (m_bThreadWorking)
    {
        if (m_nAtlasLoadingIndex > m_nSkeletonLoadingIndex)
        {
            SpineLoadingInfo* spineInfo = &m_LoadingList[m_
            nSkeletonLoadingIndex];
            spSkeletonJson* json = spSkeletonJson_create(spineInfo->Atlas);
            spineInfo->SkeletonData = spSkeletonJson_readSkeletonDataFile(
                json, spineInfo->JsonFile.c_str());
            spSkeletonJson_dispose(json);
            ++m_nSkeletonLoadingIndex;
            Director::getInstance()->getScheduler()->
            performFunctionInCocosThread([this, spineInfo](){
                //在主线程中执行收尾工作
                m_LoadingSpine.erase(spineInfo->JsonFile);
                m_SpineCache[spineInfo->JsonFile] = spineInfo->
                SkeletonData;
                if (spineInfo->Callback)
                {
                  spineInfo->Callback(spineInfo->SkeletonData);
              }
              if (m_nSkeletonLoadingIndex >= m_nMaxSpineCount)
              {
                  onFinish();
              }
          });
          //超过最大限制，退出
          if (m_nSkeletonLoadingIndex >= MAX_LOADING_LIST)
          {
              break;
          }
      }
      else
      {
          this_thread::sleep_for(chrono::milliseconds(1));
      }
   }
}
```

加载完成有两个处理，第一个是针对该骨骼的，我们会将其从m_LoadingSpine中移除，但对m_LoadingList不做任何处理，将骨骼数据设置到m_SpineCache容器中进行管理，并回调设置的回调函数。第二个是针对队列中所有骨骼加载完成的处理，当所有骨骼都加载完成后，就会调用onFinish()方法结束线程，并重置队列。

```
void CSpineLoader::onFinish()
{
   if (m_SkeletonThread)
   {
       m_bThreadWorking = false;
       m_SkeletonThread->join();
       delete m_SkeletonThread;
       m_SkeletonThread = nullptr;
   }
   m_nAtlasLoadingIndex = 0;
   m_nSkeletonLoadingIndex = 0;
   m_nMaxSpineCount = 0;
   m_LoadingSpine.clear();
}
```

整个流程是简单清晰的，接下来总结一下用到的线程安全的技巧。首先为什么要用最原始的数组，而不是vector、queue等数据结构？因为在主线程中，会对这个容器进行添加操作，而在骨骼线程中，则会对它进行获取的操作，这并不是线程安全的，程序有可能崩溃。而使用固定数组，则没有动态添加这样的概念，但还需要通过其他手段来保证线程安全——m_nAtlasLoadingIndex和m_nSkeletonLoadingIndex。

当创建第一个Atlas时，在完成所有操作之前，m_nAtlasLoadingIndex一直是0，这个变量是只有主线程才会操作的，骨骼线程只会读取它来进行判断进度。当m_nAtlasLoadingIndex不大于m_nSkeletonLoadingIndex时，骨骼线程做不了任何事情，只能是等待。也就是说m_LoadingList中的第一个元素只有主线程会操作，当主线程操作完之后，交由骨骼线程使用，主线程不再操作。

因为线程在任何时刻、任何地方都有可能切换，所以当线程间操作同一个数据时，就很容易出现问题。这里通过两个进度下标，将线程间对同一个资源访问的时机进行了规划，以确保线程操作时不会有其他的线程操作，虽然没有使用锁，但却达到了线程安全的目的。

整个流程就像一条流水线，流水线A（主线程）将玩具的零件放入盒子中，然后将盒子交给流水线B（骨骼线程）。流水线B将零件进行组装，组装完成后又放回盒子，交给流水线C（还是主线程）。流水线C将盒子密封并装箱，结束整个流程。

流水线A将盒子交给流水线B，以及流水线B交给流水线C的操作，分别是通过增加m_nAtlasLoadingIndex和m_nSkeletonLoadingIndex来实现的，在流水线B交给流水线C的操作中，还可以选择使用lambda直接将骨骼和相关信息设置到匿名函数中，交由主线程执行，从而移除m_nSkeletonLoadingIndex变量。

下面的代码演示了如何操作，将SpineTest中的SpineTestLayerNormal示例修改如下，异步加载了两个骨骼，加载完之后创建并添加到场景中。

```
bool SpineTestLayerNormal::init () {
   if (!SpineTestLayer::init()) return false;
   CSpineLoader* loader = new CSpineLoader();
   ResLoadedCallback callback = [this](spSkeletonData* data){
       auto sk = SkeletonAnimation::createWithData(data);
       sk->setScale(0.5);
       sk->setPosition(100, 200);
       addChild(sk);
   };
   loader->loadSpineAsyn("spine/spineboy.json", "spine/spineboy.atlas",callback);
   loader->loadSpineAsyn("spine/goblins-ffd.json","spine/goblins-ffd.atlas",
   callback);
   return true;
}
```

#### 12.4.4　Spine的性能优化

Spine骨骼动画的性能本身还是不错的，但是在使用的时候，却有可能成为性能瓶颈，原因并不是Spine库的效率低下，主要的原因还是使用不当导致的。

通过应用前面介绍的缓存技术可以极大提高Spine骨骼动画创建的效率，但当屏幕上的骨骼动画对象过多的时候，仍然有可能导致游戏掉帧，因为大量Spine骨骼动画的更新计算也是颇为耗时的。

SpineTest中的SpineTestPerformanceLayer例子可以辅助做一些Spine的性能测试，该例子通过在屏幕上点击可以创Spine建骨骼动画对象，本例中创建的是Spine官方的哥布林角色。在示例中，在屏幕上创建了300个哥布林角色（如图12-7所示），游戏的帧率没有受到任何影响，哥布林的动画非常流畅。

虽然每个哥布林会带来一个DrawCall的消耗，但300多个DrawCall并没有对程序的性能造成影响。当哥布林的数量接近500个时，帧率才开始下降，在游戏中一般很少会创建这么多的骨骼，所以Spine的性能就不需要进行优化了吗？

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120841.jpeg)

图12-7　加载众多哥布林骨骼动画

答案当然是否定的，将SpineTestPerformanceLayer中的哥布林换成了另外一个骑士的骨骼动画（如图12-8所示），当添加了100个骑士到场景中时，游戏的帧率已经下降到49.5了。那么是什么导致的帧率下降呢？显然不是DrawCall，因为100个骑士的DrawCall远小于300多个哥布林的DrawCall。性能下降的原因在于大量骨骼、网格、时间轴的运算逻辑。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120842.jpeg)

图12-8　加载众多骑士骨骼动画

影响Spine骨骼动画性能的3大因素，分别是骨骼数量、网格数量以及时间轴数量。**这3个因素同样对Spine骨骼动画的加载速度有较大的影响**。在制作Spine的时候，只要能够严格控制这3个因素，就可以使游戏的运行效率控制在理想的状态下，如果没有任何规范控制，那么美工制作时可能会为了达到更好的效果，而无限制地添加骨骼、网格以及时间轴。下面给出Spine骨骼动画制作时，这3个因素的建议值。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Bones骨骼数量：建议值60以内，不建议超过80。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Mesh顶点数量：建议值400以内，不建议超过600。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Timelines时间轴数量：建议值80以内，不建议超过150。

在Spine动画编辑器右上角的Views菜单中，可以打开Metrics视图，在Metrics视图中，可以看到骨骼动画相关的参数。Timelines参数需要在Tree视图中的Animations下选择指定的动画后才可以看到，因为每个动画的Timelines都不同（如图12-9所示）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120843.jpeg)

图12-9　Metrics视图

前面给的建议值仅仅是一个建议，因为具体要根据游戏的实际情况而定，如果游戏非常简单，要渲染的骨骼动画对象也很少，那么完全可以放宽要求。如果游戏要求200个角色同屏战斗，那么要求就要更加严格，做到200个角色同屏播放动画不掉帧仍然不够，因为还需要算上战斗逻辑的消耗。

另外有些动画需要重点优化，那一定不会是大BOSS角色，而是那些出现得最频繁的小怪，对出现最频繁的角色做最大的优化，对于有多个动画的角色，优化的重点也应该放在出现最频繁的动画，如移动。

除了让美工制作出尽量高效的骨骼动画之外，还可以通过一些其他手段来优化游戏的运行效率，对于没有在屏幕上的对象，可以不让它执行任何动画。将不在屏幕上的Spine骨骼动画对象设置为隐藏并不能提升性能，因为Spine骨骼动画在隐藏的状态下，仍然会执行骨骼动画的更新计算，所以可以考虑将屏幕外的Spine骨骼动画停止，当Spine进入屏幕时恢复其骨骼动画，可以使用scheduleUpdate()方法和unscheduleUpdate()方法来实现动画停止和恢复的逻辑。