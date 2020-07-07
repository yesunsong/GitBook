# 第17章　Schedule详解

定时器是游戏中常用的功能，如游戏的倒计时，每隔一段时间自动刷新怪物，根据玩家的在线时间自动增长经验等等，Cocos2d-x使用了Schedule来实现这一功能。

Schedule的核心职责是**按照设定的时间执行指定的回调**，Cocos2d-x中所有与计时相关的内容，最终都依赖于Schedule来实现，如Cocos2d-x中庞大的Action系统。

Schedule是一个任务调度器，允许在游戏中方便地添加各式各样的定时任务，除了提供添加和删除各种定时任务的功能之外，Schedule还支持优先级、暂停恢复，以及全局时间缩放等功能。在Cocos2d 3.0之后Schedule还提供了线程安全的回调。本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　Schedule接口简介。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　Schedule整体结构与运行流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　Schedule使用过程中的问题。

## 17.1　Schedule接口简介

Scheduler直接继承于Ref，使用Schedule的第一步是需要获取Ref的引用，Director中保存一个全局Schedule对象的指针，在Director初始化时创建，可以直接使用。Node在初始化的时候会自动将Director中的Schedule保存为成员变量，并且提供了一些简单的方法封装，以方便使用。

### 17.1.1　回调函数

使用Schedule的第二步就是编写一个回调，Schedule可以添加各种定时回调，但对于回调函数的原型是有要求的，目前Cocos2d-x 3.0支持**ccSchedulerFunc函数回调**和**SEL_SCHEDULE对象回调**两种回调方式。ccSchedulerFunc函数回调的定义如下：

```
typedef std::function<void(float)> ccSchedulerFunc;
```

cSchedulerFunc是一个function类型，这是C++11的新特性，对应的函数原型是void fun(float)。在Cocos2d-x 3.0之前，Schedule只支持SEL_SCHEDULE对象回调，定义如下：

```
typedef void (CCObject::*SEL_SCHEDULE)(float);
```

在Cocos2d-x 3.0之前的版本中，Schedule要求回调函数必须是一个CCObject的成员函数，且原型是void fun(float)（CCObject本身有一个update函数，符合该原型，在ScheduleUpdate方法中可以以这个方法为默认回调）。

Cocos2d-x 3.0之后使用了C++11的匿名函数，只需要是void fun(float) 这样的函数原型即可，不需要是Ref的成员函数，这为使用匿名函数提供了方便（虽然3.0后的Schedule也需要同时传入Target对象和回调方法，但Target主要作为一个关联记录，便于在某对象移除出场景的时候，一并移除其关联的定时回调）。

### 17.1.2　注册回调

有了Schedule对象和回调函数，第三步就是注册定时回调了，可以调用以下几个接口来完成该功能：

```
//注册一个关联到Target的函数回调callback，在delay秒后，以interval秒一次的频率，
执行repeat + 1次
//Target用于关联注册者，callback为回调函数对象
//interval为0的话，回调函数在每一帧都会被调用
//repeat 会重复执行这个函数repeat次，总次数等于 1次 + 重复次数，使
用 CC_REPEAT_FOREVER 可以不断地重复
//delay 表示延迟delay秒之后才开始计时，为0的话表示不延迟，直接开始计时
//如果paused为true，这个回调需要被resumed之后才开始正常执行
//key用于作为函数回调的标识，用于区分同一个Target对象所注册的不同函数回调
void schedule(const ccSchedulerFunc& callback, void *target, float interval, 
unsigned int repeat, float delay, bool paused, const std::string& key);
//调用schedule注册一个函数回调，每隔interval秒执行一次
//启动延迟delay为0，重复次数repeat为CC_REPEAT_FOREVER无限次数
void schedule(const ccSchedulerFunc& callback, void *target, float interval, 
bool paused, const std::string& key);
//注册一个对象回调，在delay秒后，以interval秒一次的频率执行repeat + 1次
//执行回调时，会调用target->selector(dt)
void schedule(SEL_SCHEDULE selector, Ref *target, float interval, unsigned 
int repeat, float delay, bool paused);
//注册一个对象回调，每隔interval秒执行一次
void schedule(SEL_SCHEDULE selector, Ref *target, float interval, bool 
paused);
//注册一个函数回调，以priority为优先级，每帧调用target->update(dt)
void scheduleUpdate(T *target, int priority, bool paused)
//注册一个脚本回调，传入要回调的脚本句柄，每隔interval调用一次
//返回一个回调ID，使用这个回调ID可以注销回调
unsigned int scheduleScriptFunc(unsigned int handler, float interval, bool 
paused);
```

关于注册回调在有几点要注意：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　对象回调SEL_SCHEDULE和函数回调ccSchedulerFunc的区别在于，对象回调是执行Target的成员函数，而函数回调只是执行函数，函数与Target没有直接关系，只用来记录注册者。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　注册的参数key是用于区别一个Target对象所注册的不同函数回调，对象回调是根据成员函数指针来判断是否重复。而函数回调之所以需要用key来区别，是因为函数回调可能是一个匿名函数，不方便判断两个匿名函数是否相等。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　如果一个回调已经被注册，再次调用schedule**只会更新其interval属性**，对象回调根据成员函数指针来判断是否重复，函数回调使用key来判断是否重复。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　scheduleUpdate是比较特殊的回调，是函数回调包裹着对象回调，以函数回调的方式注册，最后在函数回调中执行Target对象的成员函数。在Cocos2d-x 3.0之前，scheduleUpdate则是一个纯粹的对象回调。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　scheduleSelector、scheduleUpdateForTarget等注册函数在Cocos2d-x 3.0之后开始废弃，替换为schedule和scheduleUpdate。

### 17.1.3　注销回调

在需要注销schedule的回调时，可以调用以下方法：

```
//注销关联到target上，key相等的函数回调，
void unschedule(const std::string& key, void *target);
//注销关联到target对象上，指定函数的对象回调
void unschedule(SEL_SCHEDULE selector, Ref *target);
//注销target对象上的update回调
void unscheduleUpdate(void *target);
//注销所有关联到target对象上的回调，包括update回调
void unscheduleAllForTarget(void *target);
//注销所有的回调，慎用
void unscheduleAll(void);
//注销所有优先级低于minPriority的回调
//minPriority必须为kPriorityNonSystemMin或者更高的值
void unscheduleAllWithMinPriority(int minPriority);
//传入脚本回调ID，注销该回调
void unscheduleScriptEntry(unsigned int scheduleScriptEntryID);
```

### 17.1.4　时间缩放、暂停、恢复

Schedule还提供了时间缩放（加速播放或减速播放）、暂停与恢复等功能。

```
/获取当前的时间缩放
inline float getTimeScale()
/设置时间缩放，默认为1.0
//设置小于1.0可以营造出慢镜头的效果，0.5表示原来播放速度的一半
//设置大于1.0可以营造出快进效果，2.0表示原来播放速度的一倍
//设置时间缩放会影响到所有的action和schedule
inline void setTimeScale(float timeScale)
//暂停Target对象的所有回调，直到对象被resumed
void pauseTarget(void *target);
//恢复Target对象所有被暂停的回调
void resumeTarget(void *target);
//询问一个对象当前是否被暂停
 bool isTargetPaused(void *target);
//暂停所有的schedule回调，慎用
std::set<void*> pauseAllTargets();
//暂停所有优先级小于minPriority的对象
//minPriority必须为kPriorityNonSystemMin或者更高的值
std::set<void*> pauseAllTargetsWithMinPriority(int minPriority);
//恢复传入的Target对象列表，用于与pauseAllSelectors配对
void resumeTargets(const std::set<void*>& targetsToResume);
```

### 17.1.5　优先级

**只有update回调有优先级的概念**，其他普通的函数回调或对象回调都没有优先级的概念。在注册一个update回调时，可以设置其优先级，前面针对优先级的批量注销和暂停操作，也只适用于update回调。**优先级按照数值从小到大的顺序执行，值越小优先级越高**，PRIORITY_SYSTEM是一个非常小的负数，优先级很高，在使用Node的scheduleUpdate注册update回调时，使用的**默认优先级是0**。

### 17.1.6　线程安全

Schedule还提供了一个比较实用的功能performFunctionInCocosThread。该功能是Cocos2d-x 3.0之后新增的，用于线程安全。在其他线程中使用该方法可以安全地由主线程执行一些逻辑，因为当我们在其他线程中操作Cocos2d-x相关内容时，有可能会导致程序崩溃或出现一些渲染错误的问题。

```
//在Cocos主线程中执行function回调，当需要在其他线程中执行主线程相关的逻辑时调用
//例如使用第三方SDK进行登录操作，SDK在线程中返回登录结果
//这时就需要把结果转到主线程中来处理
void performFunctionInCocosThread( const std::function<void()> &function);
```

## 17.2　Schedule整体结构与运行流程

### 17.2.1　整体结构

Schedule的整体结构由Schedule和Timer组成。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　Timer负责将一个回调封装为一个对象，管理回调的计时、触发、保存回调的状态。一共有3种Timer：TimerTargetSelector封装了对象回调；TimerTargetCallback封装了函数回调；TimerScriptHandler封装了脚本回调。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　Schedule管理着大量的Timer，负责注册、注销，以及驱动执行回调等工作，是Timer的总调度室。

如图17-1所示，Schedule内部将回调规划为以下几类进行管理。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155634.jpeg)

图17-1　Schedule结构

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　update回调，使用3个优先级不同的链表_updatesNegList、_updates0List和_updatesPosList来管理遍历，再使用一个哈希容器_hashForUpdates来管理Target和update回调的联系。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　对象回调和函数回调，使用一个哈希容器_hashForTimers来统一管理回调。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　脚本回调，使用一个vector容器_scriptHandlerEntries来管理注册的脚本回调。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　执行回调，使用一个vector容器_functionsToPerform来管理来自其他线程注册的一次性执行回调。

### 17.2.2　外部流程

在Cocos2d-x中共用一个Schedule对象，该Schedule对象由Director创建并维护。在Director启动的时候，会创建一个Schedule对象，通过Director::setScheduler可以获取到。可以自己创建一个Schedule对象来管理局部的某些计时回调，但一般直接使用全局的Schedule。

```
bool Director::init(void)
{
    //省略其他代码
    _scheduler = new (std::nothrow) Scheduler();
}
```

Director在每一帧的主循环中，都会调用全局Schedule的update，来驱动Schedule内部的计时回调。

```
if (! _paused)
{
    _scheduler->update(_deltaTime);
    _eventDispatcher->dispatchEvent(_eventAfterUpdate);
}
```

当Director被销毁时（退出游戏的时候），全局的Schedule也会被销毁，同时清理Schedule自身的所有回调对象。

```
Director::～Director(void)
{
    CC_SAFE_RELEASE(_scheduler);
}
```

### 17.2.3　更新顺序

我们已经知道Schedule内部分为了哪几类回调，以及如何注册、注销这些回调，那么这几类回调之间的执行顺序如何？回调内部的执行顺序又如何呢？如果不了解这些细节，在一些要求严格的执行顺序的代码中，可能会出现一些难以察觉的BUG。下面通过图17-2来分析整个更新流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155635.jpeg)

图17-2　Schedule的更新

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　如果_timeScale不为1.0，将传入的时间乘以_timeScale进行缩放 。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　按照优先级数值<0，= 0，> 0的顺序，遍历Schedule内部的3个update回调链表，依次执行每个回调对象的callback函数 。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　遍历自定义函数对象（对象回调和函数回调）的哈希容器，每个Target会有一个Timers列表，遍历Timers列表的每个节点，并调用它们的update函数，如果该回调已被注销，则直接删除该Timers。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　遍历3个update回调链表，检查每个回调是否被注销，如果是则删除该回调对象 。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　如果使用了脚本，遍历执行所有脚本Timers的update，如果回调被注销，则删除对应的回调对象。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　如果执行回调列表不为空，则进行锁操作，并取出执行回调的内容，解锁后顺序执行回调。

### 17.2.4　注册与注销

Schedule注册与注销需要注意的隐藏细节，主要是安全性和调用时机的细节。

首先是安全性的问题，我们知道，Schedule的update是对回调列表的遍历，然后顺序执行，这时候会涉及在遍历过程中动态地注册和注销Schedule，那么我们的操作是否安全呢？在遍历的过程中动态地注册和注销都是安全的。对象回调、函数回调和脚本回调会在遍历的过程中被安全地移除，而update回调会在遍历完成之后，额外遍历一次，将标记为移除的回调移除（调用unschedule进行注销，如果该回调在这一帧未执行，那么该回调就不会被执行）。

最后是调用时机的问题，当注册一个回调时，什么时候会开始进行计时呢？不同的回调在不同的时机注册，开始计时的时机也不同。注册Schedule时，主要的时机有以下几个地方（要看最上层的调用）：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　Node回调中，如onEnter和onExit。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　单击回调中，如按钮的单击处理。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　计时回调中，在计时回调内部注册。

Node回调中注册添加的所有计时回调，都会在**下一帧**开始生效，而单击回调注册添加的所有计时回调，会在**当前帧**开始生效。计时回调中注册的计时回调的情况是最复杂的。不同的回调注册生效时机是不同的，首先根据不同回调的遍历时机，如果在update回调中注册了一个函数回调，那么本帧这个函数回调是会生效的。

计时回调间的互相注册遵循这样的规则——顺序先的注册顺序后的回调，本帧执行，顺序后的注册顺序先的回调，下一帧执行。而相同类型的回调注册，则看具体的回调：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　在脚本回调中注册，当前帧会生效，因为脚本回调是直接插入脚本列表尾部。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　在函数回调和对象回调中注册其他对象的回调，生效时间不确定，根据哈希遍历规则而定，而注册自身的回调，当前帧会生效。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707155633.jpeg)　在update回调中注册，如果要注册的回调优先级低于等于当前回调的优先级（数值更大），则该帧生效，否则下一帧生效。

隐藏的细节读者略微了解即可，实际用到的情况很少，如果需要严格依赖控制回调的生效时机，那么尽量不要在计时回调中动态注册计时回调，应该避开这个“坑”。

## 17.3　Schedule使用过程中的问题

一个很容易困扰我们的问题就是，为什么注册到Schedule中的函数没有被执行？这个问题有两种情况，**第一种是Node对象处于未激活状态，第二种是游戏暂停了**。

当节点被添加到场景里的时候，就会进入激活状态，而当节点从场景中删除的时候，就会进入非激活状态，甚至直接被销毁。Node中有一个成员变量_running来标识这个状态，该变量的改变主要在onEnter和onExit中，在游戏设计的时候经常会继承，然后重写这两个方法，并且忘记调用父类的onEnter和onExit，那么这个时候状态变量就没有被重置，而runAction、Schedule等一系列的函数，在注册Schedule时会根据该变量来判断回调是否暂停，默认是非激活状态，所以Schedule的回调不会被执行。

Cocos2d-x 3.0之后与3.0之前有一个不同点是，**与Schedule关联的节点Target变成一个没有retain的弱引用**，当在回调外部调用Target的release方法时，这个Target可能真的被释放了，那么回调中对Target的任何调用都会导致程序崩溃。如果按照正确的方法来使用Schedule，那么就不会出现这么糟糕的问题，在这里只需要知道，Schedule并不会为Target对象执行retain操作就可以了。

与在外部释放了Target相对的是忘记注销匿名函数，你需要将匿名函数关联到一个合适的对象身上，由Node来自动管理这些回调是最好的选择，否则需要记住自己注册了什么回调，并在不需要使用的时候手动注销它们。

为了验证Schedule的稳定性，笔者使用了各种方法来验证Schedule，例如，使用未初始化的指针来注册匿名函数，在各种情况下注册注销回调等，从验证的结果来看，Schedule的实现还是很健壮的。