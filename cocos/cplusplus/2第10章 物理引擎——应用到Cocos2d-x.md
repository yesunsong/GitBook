# 第10章　物理引擎——应用到Cocos2d-x

了解了Box2d的基础知识后，需要使用Box2d，并将它应用到Cocos2d-x中。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　物体的运动。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　碰撞检测。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Box2d的调试渲染。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在Cocos2d-x中使用Box2d。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Box2d的相关工具。

### 10.1　物体的运动

前面讲解了刚体以及关节，知道了它们在被动地受到重力或者马达的作用下，会产生相应的物理效果，那么如何主动来控制物体的运动呢？

对于关节，可以用设置马达的方式，马达来驱动关节运动，而对于刚体，可以通过**施加力或者冲量来使其移动，施加角力矩或者角冲量来使其旋转等**，听起来很厉害，但本质上都是一些比较简单的东西，放到代码中实现就更简单了。

#### 10.1.1　施加力和冲量

对一个物体施加力，可以通过下面两个函数来实现，ApplyForce第一个参数决定物体运动的方向，单位是牛顿，而第二个参数，决定物体受力点的位置，假设**受力点不在物体上，也是有效的**，效果如图10-1所示，蓝色的受力点在物体外部，当对其施加一个向上的力时，可以将受力点认为是在正方形的右上角，为物体施加力就像在推动一个物体一样。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120636.jpeg)

图10-1　施加力

```
///为物体的point点施加一个方向为force的力，point 是世界坐标
void ApplyForce(const b2Vec2& force, const b2Vec2& point);
///对物体的中心点施加一个方向为force的力
void ApplyForceToCenter(const b2Vec2& force);
```

对一个物体施加一个冲量可以用下面的ApplyLinearImpulse()函数来实现，第一个参数也是一个力的向量，这个向量的单位是kg·m/s。物体质量越大，运动的速度会越慢，而为物体施加一个冲量，等于直接赋给物体一个速度，在施加一个很大的冲量的时候，物体会一下子冲出去，然后在摩擦、重力、碰撞的影响下，慢慢衰减下来，就像突然使用了极品飞车的加速氮气一样。

```
///在point上施加向量为impulse的冲量，冲量的单位是kg·m/s
void ApplyLinearImpulse(const b2Vec2& impulse, const b2Vec2& point);
```

对比力和冲量，**对物体施加力需要通过力来摆脱物体的静摩擦力，然后慢慢加速，而冲量则是可以直接对物体赋予一个速度**。两种施力的力量参数的单位是不一样的，但受力点的意义一样，不同的受力点会在受力过程中，对物体产生不同的旋转。

#### 10.1.2　角力矩和角冲量

使用下面的函数可以为物体施加一个角力矩和角冲量，当传入一个正数时，物体会在逆时针方向上自转，而传入一个负数时，物体在顺时针方向上自转。传入的参数越大，并且物体质量越小，转动越快。从本质上讲，角力矩相当于上面的施加力，而角冲量则是直接赋予物体角速度。施加角力矩，物体会慢慢运动起来，就像掷链球一样，把链球拉起来，慢慢转圈。而施加角冲量，可以让物体瞬间高速旋转。在没有收到其他力的作用下，旋转的物体并不会产生移动。

```
///施加一个角力矩，让物体根据穿过质点的z轴进行旋转
void ApplyTorque(float32 torque);
///为物体施加一个角冲量，单位是kg·m/s
void ApplyAngularImpulse(float32 impulse);
```

### 10.2　碰撞检测

虽然说物体之间的碰撞Box2d已经很好地模拟出来了，但还是要了解一下Box2d的碰撞检测，因为在很多时候需要知道碰撞的具体信息。例如，怪物被子弹碰到了，除了被打飞，或者打倒，还需要做很多其他的操作，如把怪物删除，然后增加经验、金币之类的事情，这就需要用到**碰撞监听**。而在玩CS之类游戏的时候，当把友军伤害选项给屏蔽掉之后，我们发射的子弹只对敌军产生影响，这样的功能就涉及**碰撞过滤**，本节主要介绍这两项内容。

#### 10.2.1　碰撞监听

Box2d里面使用接触（Contact）来描述碰撞信息，在每次两个物体的AABB出现重叠的时候，Box2d会产生相应的触点，当物体的AABB分离的时候，又会把触点删除。使用World的GetContactList()函数可以获取当前世界所有的接触信息，通过这些接触信息，可以获取到接触的物体以及接触点等信息。但这并不是明智的做法，因为无法捕捉到所有的接触。例如在一次循环中，在很多力的作用下，两个物体短暂地接触了，之后又快速地分开，在接触列表中是不会找到这个接触对象的。并且在每一帧轮询这些接触对象本身也是一个冗余的运算，就像坐公交车，每个站台都问一句XX站到了没？明智的做法是使用接触监听，它会在到达XX站台时自动通知你。

使用监听需要实现一个监听者，叫作接触监听器，名字为b2ContactListener，位于b2WorldCallbacks.h内，可以实现它的几个接触相关的接口。

```
///在两个fixtures开始碰撞的时候回调（当它们开始重叠的时候，只会在step中调用）
virtual void BeginContact(b2Contact* contact) { B2_NOT_USED(contact); }
///在两个fixtures结束碰撞的时候回调（当它们分开的时候，物体被销毁的时候也会调用）
virtual void EndContact(b2Contact* contact) { B2_NOT_USED(contact); }
///在接触更新完成之后调用（碰撞检测发生之后，碰撞冲突处理之前）.此时可以禁用触点，实现
单面碰撞。例如，是男人就上100层这样的小游戏的楼梯，在玩家跳上去时不发生碰撞，在玩家掉
落的时候才碰撞。通过禁用触点可以避免此次碰撞处理，并且不会调用到接触处理完成的回调，但
是可能会在一小段时间内连续收到PreSolve回调
virtual void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
{
    B2_NOT_USED(contact);
    B2_NOT_USED(oldManifold);
}
///在接触被处理完之后调用，这时候物理模拟已经完成，在这里可以获得接触对象碰撞之后产生
的力量、旋转等信息
virtual void PostSolve(b2Contact* contact, const b2ContactImpulse* impulse)
{
    B2_NOT_USED(contact);
    B2_NOT_USED(impulse);
}
```

Box2d**不允许在碰撞回调中修改物理世界**，因为上面可能发生在一个step回调之中，当出现多个对象同时碰撞，会依次处理每两个对象之间的碰撞，而这时候会调用到回调函数，在回调函数中修改了对象，可能会导致其他对象的碰撞结果不正确，假设在回调中释放了对象，还有可能导致程序崩溃。如果需要删除或者做修改，可以将触点信息保存起来，在step完成之后，再来处理。

上面的回调中可以做的修改仅仅是禁用触点，来规避此次碰撞。例如超级玛丽游戏中的单面墙，可以通过判断碰撞的方向来决定是否规避此次碰撞，代码如下。

```
virtual void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
{
b2WorldManifold worldManifold;
contact->GetWorldManifold(&worldManifold);
if (worldManifold.normal.y > 0.5f)
{
    contact->SetEnabled(false);
}
}
```

在每次回调都会有一个b2Contact对象被传进来，描述了发生碰撞的两个物体的详细信息，通过GetFixtureA()函数和GetFixtureB()函数可以分别获取这两个物体，但是需要你自己判断这两个物体是什么，可以根据指针来判断，也可以设置UserData，根据UserData来判断。在冲突处理之前的PreSolve回调中，调用b2Contact对象的SetEnabled()函数可以设置是否禁用接触。

最后，在使用的时候，需要用new操作符创建一个这样的监听器对象，通过调用World->SetContactListener(myContactListener)，把它设置给Wold。

#### 10.2.2　碰撞过滤

在创建Fixture的时候，可以通过**设置过滤标识**，来控制Fixture之间的碰撞过滤，Fixture有两种过滤标识，每个标识都是一个16位的int16变量（相当于短整型），可以表示16种不同类型的碰撞。

```
b2Filter()
{
    categoryBits = 0x0001;  //类别标志位
    maskBits = 0xFFFF;      //遮罩标志位
    groupIndex = 0;         //分组索引
}
```

类别标志位定义了Fixture的类别，而遮罩标志位则定义了可以与之发生碰撞的类别，举个例子：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　我是一个公主a = 1。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　我是一个战士b = 2。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　我是一个英雄c = 4。

我是一个公主，那么我的**categoryBits** |= a，我只能和英雄发生碰撞，那么我的**maskBits** |= c，这是我的逻辑，那么这个碰撞过滤，还得看英雄愿不愿意和我碰一下，也就是英雄的**maskBits & a**是否等于0，用一句代码来解释，就是：

```
isCollid = A.maskBits & B.categoryBits != 0 && A.categoryBits & B.maskBits !=
0
```

在b2Filter的构造函数中，将categoryBits设置为1，而将maskBits设置为0xFFFF，表示我可以和所有的Fixture发生碰撞，而且所有的Fixture在创建的时候，categoryBits都是1。

那么**分组索引**是干什么的呢？其用来描述更加复杂的规则。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　当A和B的分组索引相同且为正数，则产生碰撞。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　当A和B的分组索引相同且为负数，则不产生碰撞。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在其他的情况下，都使用正常的类别/遮罩过滤规则。

简单概括就是，如果是大于0的相同组，则一定碰撞，如果是小于0的相同组，则一定不碰撞。

在使用上面的标志和组无法解决问题的时候，还可以通过触点过滤器来进行碰撞过滤，这有点类似于碰撞监听，继承一个b2ContactFilter对象，然后实现它的碰撞过滤方法，通过在World中调用SetContactFilter()函数，来传入触点过滤器，使其接受触点消息。

```
virtual bool ShouldCollide(b2Fixture* fixtureA, b2Fixture* fixtureB);
```

当这两个Fixture的AABB包围盒发生重叠的时候，就会调用这个回调，这时候它们可能还没有真正发生碰撞，可以通过自己设置的UserData来判断过滤，也可以根据Fixture的过滤标识结合自己定义的规则来判断过滤。如果允许它们发生碰撞，则返回true；否则返回false，让它们直接穿透，不做碰撞处理。另外，在游戏过程中也可以动态地修改Fixture的过滤标识，来达到想要的效果。

### 10.3　Box2d的调试渲染

Box2d只是一个物理引擎，**本身并不提供显示功能，只提供物理模拟的功能**，因此可以很方便地运用到那些2D游戏引擎当中，在Box2d引擎的代码中，提供了TestBed这个庞大的范例库。

在TestBed中，GLUT完成了物理场景的渲染，以及其他的一些输入和输出的工作，那么在Cocos2d-x中，对Box2d的TestBed略加修改就可以搬到TestCpp里面了。

Box2d虽然不支持渲染，但提供了一个**用于调试**的渲染接口，在libBox2d项目的Common目录下的b2Draw提供了绘制点、线、圆、多边形等的接口，根据传入的参数，使用OpenGL的接口绘制它们，cpp-tests的Box2DTestBed中，GLESRender实现了这个功能。

需要注意的一点是，TestBed渲染的内容是Debug Draw，而不是Game Draw，它仅仅只是一些点、线、圆而已（绘制的是调试内容，而不是游戏内容）。这套流程跟cpp-tests的Box2dTest不太一样。

不管是在Box2d的TestBed还是Cocos2d-x的Box2dTestBed，它们的b2Draw对象的实现都在很好的工作，并且以后应该也不会修改到这部分的代码，在这里介绍如何使用OpenGL来绘制点、线、圆之类并不是很恰当，所以关于DebugDraw的渲染，我们只介绍它们是如何工作的，而不讨论渲染的细节。

首先需要实现b2Draw接口，把它叫作GLESRender，并且需要有一个这个类的对象，通过World→SetDebugDraw(myDebugDraw)；然后将它赋给World，接下来，在每一帧模拟完物理世界之后，调用World→DrawDebugData()。

在每次调用DrawDebugData()函数的时候，Box2d都会根据当前世界的所有对象的形状等信息，调用b2Draw对象的绘图函数来渲染。如果希望在Cocos2d-x中显示调试信息，来帮助确定物理对象的显示与实际的物理形状是否一致，可以使用cpp-tests中Box2dTestBed示例的GLES-Render.h和GLES-Render.cpp文件，将GLESDebugDraw设置为物理世界的调试渲染对象。

### 10.4　在Cocos2d-x中使用Box2d

在Cocos2d-x中使用Box2d有两个要点，第一点是在Cocos2d-x中完成物理引擎的初始化和更新，第二点是将Cocos2d-x的渲染和Box2D的物理模拟结合起来。本节将由浅入深地介绍Box2d在Cocos2d-x中的应用，将物理引擎嵌入Cocos2d-x框架中，从对象物理模拟和Cocos2d-x渲染，到碰撞监听的回调，再到安全地删除刚体，释放物理世界。

#### 10.4.1　物理世界

首先需要有一个场景，将这个场景作为我们的物理世界，所以这个场景需要完成第一个任务，即在onEnter或init的时候，初始化物理世界，设置好重力。

```
b2Vec2 gravity;
gravity.Set(0.0f, -10.0f);
m_World = new b2World(gravity);
m_World->SetAllowSleeping(true);
m_World->SetContinuousPhysics(true);
```

在每一帧的update中，都要执行物理世界的更新，这只是很简单的几行代码。

```
int velocityIterations = 8;
int positionIterations = 1;
m_World->Step(dt, velocityIterations, positionIterations);
```

创建World，更新World，放在场景中是没有问题的，但是假设把这些物理相关的东西，封装到一个物理管理器里面作为一个单例，在Cocos2d-x中使用会更好一些。这样做的好处是，把物理框架从场景分离开，使物理框架的功能更独立一些，清晰一些，整体的耦合性小一些。并且在很多地方，可能需要使用物理引擎来做一些东西。假如需要先找到场景节点，然后获取它的World，那么这样做会使整体的耦合度变得很高，而如果封装到单例里面，这些操作就会变得“优雅”很多。

```
class CPhysicsManager
{
private:
    CPhysicsManager(void);
    virtual ~CPhysicsManager(void);
public:
    static CPhysicsManager* getInstance();
    //在一场游戏结束之后应该调用
    static void destory();
    //更新物理世界
    void update(float dt);
    inline bool isLocked()
    {
         if (NULL == m_World)
         {
             return false;
         }
         return m_World->IsLocked();
    }
    //获取世界
   inline b2World* getWorld()
   {
       return m_World;
   }
private:
   static CPhysicsManager* m_Instance;
   b2World* m_World;
   CPhysicsListener* m_ContactListener;
};
```

笔者是这样设计这个单例的，这个单例会负责维护两项，一个是我们的物理世界，在构造函数中初始化物理世界，另外一个是我们自定义的碰撞监听器，在绝大部分情况下总是需要它的。在构造函数中会初始化物理世界，而update()函数将会驱动物理世界的Step进行模拟。

```
CPhysicsManager::CPhysicsManager(void)
{
    //创建碰撞监听器
    m_ContactListener = new CPhysicsListener();
    //初始化物理世界
    b2Vec2 gravity;
    gravity.Set(0.0f, -10.0f);
    m_World = new b2World(gravity);
    //设置碰撞监听器
    m_World->SetContactListener(m_ContactListener);
    //允许刚体睡眠
    m_World->SetAllowSleeping(true);
    //激活连续碰撞检测
    m_World->SetContinuousPhysics(true);
}
```

最后在游戏场景初始化的时候，调用单例的初始化函数，在游戏场景退出的时候，调用单例的销毁函数，因为当玩家退回到主界面的时候，物理世界不应该继续模拟了，而当玩家进入游戏的时候，物理世界必须是一个新的世界，不能使用上一个关卡所遗留的数据来模拟。当然，需要在游戏场景节点的update()函数中调用CPhysicsManager的update，这里没有说释放，但应在析构函数中，把new出来的CPhysicsListener和b2World释放掉。

接下来还需要创建场景的边界，用一个包围盒把场景框住，在设置和Box2d相关的大小变量时，一般都要除以一个PTM_RATIO常量，这个常量一般是32，表示像素和物理单位米的比例，因为在设置物体大小的时候，按照现实世界的比例来设置是比较好的，假设没有这个参数，那就会变成1像素=1米，在大多数情况下，32像素=1米的比例能够更好地工作。当然这只是一个比例问题。让显示对象缩小到1/32或其他比例以适应物理对象，或者让物理对象放大32倍来适应显示对象，关键的地方在于物理对象的大小和显示对象的大小是否相等。

```
//定义包围盒
b2BodyDef groundBodyDef;
groundBodyDef.position.Set(0, 0); //bottom-left corner
//调用世界工厂的方法创建刚体
b2Body* groundBody = m_World->CreateBody(&groundBodyDef);
//定义包围盒的形状
b2EdgeShape groundBox;
//设置包围盒的底部
groundBox.Set(b2Vec2(VisibleRect::leftBottom().x/PTM_RATIO,
    VisibleRect::leftBottom().y/PTM_RATIO),
b2Vec2(VisibleRect::rightBottom().x/PTM_RATIO,
    VisibleRect::rightBottom().y/PTM_RATIO));
groundBody->CreateFixture(&groundBox,0);
//设置包围盒的顶部
groundBox.Set(b2Vec2(VisibleRect::leftTop().x/PTM_RATIO,
    VisibleRect::leftTop().y/PTM_RATIO),
b2Vec2(VisibleRect::rightTop().x/PTM_RATIO,
    VisibleRect::rightTop().y/PTM_RATIO));
groundBody->CreateFixture(&groundBox,0);
//设置包围盒的左边
groundBox.Set(b2Vec2(VisibleRect::leftTop().x/PTM_RATIO,
    VisibleRect::leftTop().y/PTM_RATIO),
b2Vec2(VisibleRect::leftBottom().x/PTM_RATIO,
    VisibleRect::leftBottom().y/PTM_RATIO));
groundBody->CreateFixture(&groundBox,0);
//设置包围盒的右边
groundBox.Set(b2Vec2(VisibleRect::rightBottom().x/PTM_RATIO,
    VisibleRect::rightBottom().y/PTM_RATIO),
b2Vec2(VisibleRect::rightTop().x/PTM_RATIO,
    VisibleRect::rightTop().y/PTM_RATIO));
groundBody->CreateFixture(&groundBox,0);
```

我们能且只能通过World的create()方法来创建刚体，如果直接用new或者malloc来创建刚体，那么创建的刚体将不在这个世界之内，也不会和物理世界有任何交集。上面创建包围盒的代码，应该写在场景中，因为这并不属于物理框架的内容，物理框架不会知道，创建的这个场景的地形长什么样的，有多大。

#### 10.4.2　物理Sprite

接下来要添加场景内的东西了，主要是把显示对象Sprite和b2Body结合起来，双继承也许会是一个好主意，但可能存在比较多的争议，将b2Body作为Sprite的一个成员变量已经可以比较好地工作了，为什么是b2Body作为Sprite的成员变量，而不是反过来呢？首先，编码的时候可能会频繁用到Sprite里面的东西，但是b2Body可能很少问津。另外，当Body被销毁的时候，Sprite可能需要继续存在于场景中，对于Body，只是需要使用其物理特性而已。

首先需要有一个继承于Sprite的类，因为需要为其添加一些成员变量，一个b2Body指针，在onEnter的时候初始化这个指针，可以在onExit或者析构函数中释放它，继承于Sprite的类先管其叫CPhysicsObject，可以在init中初始化物理刚体，这块在描述不同的CPhysicsObject时，可以根据需要重写这部分的代码。下面的一小段代码只是用来介绍，在Cocos2d-x中创建刚体的过程，实际上CPhysicsObject应该作为一个纯粹的物理对象基类来使用，不应该在这里添加创建刚体的代码，应该由子类来完成这个任务。

```
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;
bodyDef.position.Set(getPositionX() / PTM_RATIO, getPositionY() / PTM_
RATIO);
m_Body = world->CreateBody(&bodyDef);
b2PolygonShape dynamicBox;
Size sz = getContentSize();
//根据精灵图片的大小以及像素和米的比例，来设置包围盒
dynamicBox.SetAsBox(sz.width * 0.5f / PTM_RATIO, sz.height * 0.5f / PTM_
RATIO);
//设置好动态刚体的属性，然后配置给Body
b2FixtureDef fixtureDef;
fixtureDef.shape = &dynamicBox;
fixtureDef.density = 1.0f;
fixtureDef.friction = 0.3f;
m_Body->CreateFixture(&fixtureDef);
```

在onExit()函数中，析构或者任何想要删除刚体的时候，需要调用下面的代码来释放。

```
void CPhysicsObject::onExit()
{
   if (NULL != m_Body)
   {
       CPhysicsManager::getInstance()->getWorld()->DestroyBody(m_Body);
       m_Body = NULL;
   }
   Sprite::onExit();
}
```

有了刚体之后，需要注意一件事情，就是不要再调用这个刚体的setPosition()方法来改变刚体的位置，如果要设置，需要同时更新m_Body的位置属性，强制设置位置会导致物理模拟出错。

另外还应该做一个事情，就是**把m_Body的位置，旋转等属性同步到Sprite中**。在Node中有一个nodeToParentTransform()函数，用于返回一个描述节点当前的旋转和位置的矩阵，在Node中是根据当前节点的位置、锚点，以及旋转来计算这个矩阵的，在这里用m_pBody的位置和旋转来计算。

```
AffineTransform CPhysicsObject::nodeToParentTransform(void)
{
   if (NULL == m_Body)
   {
       return Sprite::nodeToParentTransform();
   }
   b2Vec2 pos = m_Body->GetPosition();
   float x = pos.x * PTM_RATIO;
   float y = pos.y * PTM_RATIO;
   if ( isIgnoreAnchorPointForPosition() ) {
       x += m_tAnchorPointInPoints.x;
       y += m_tAnchorPointInPoints.y;
   }
   //Make matrix
   float radians = m_Body->GetAngle();
   float c = cosf(radians);
   float s = sinf(radians);
   if( ! m_tAnchorPointInPoints.equals(CCPointZero) ){
       x += ((c * -m_tAnchorPointInPoints.x * m_fScaleX) + (-s * -m_
       tAnchorPointInPoints.y * m_fScaleY));
       y += ((s * -m_tAnchorPointInPoints.x * m_fScaleX) + (c * -m_
       tAnchorPointInPoints.y * m_fScaleY));
   }
   //Rot, Translate Matrix
   m_tTransform = AffineTransformMake( c * m_fScaleX, s * m_fScaleX,
   -s * m_fScaleY, c * m_fScaleY,
       x,    y );
   return m_tTransform;
}
```

nodeToParentTransform()函数在对象需要被重绘的时候调用，Cocos2d-x根据isDirty虚函数的返回值，来决定是否重绘。正常情况下，当玩家的位置、大小、旋转发生改变的时候，nodeToParentTransform()函数就会返回true，而在使用了Box2d的情况下，CPhysicsObject应该在物体的运动状态下，返回true，而在静止状态下，返回false，可以直接返回body的IsAwake()函数，当刚体醒着的时候更新，当刚体静止下来的时候，停止更新。

```
bool CPhysicsObject::isDirty()
{
   if (NULL != m_Body)
   {
       return m_Body->IsAwake();
   }
   return CCSprite::isDirty();
}
```

CPhysicsObject还需要重写一些接口，用于设置位置和旋转，因为在设置旋转和位置的时候，需要同步到物理世界，而在获取位置和旋转的时候，也需要从物理世界中获取。

```
const CCPoint& CPhysicsObject::getPosition()
{
   if (NULL == m_Body)
   {
       return CCSprite::getPosition();
   }
   b2Vec2 pos = m_Body->GetPosition();
   float x = pos.x * PTM_RATIO;
   float y = pos.y * PTM_RATIO;
   m_tPosition = ccp(x,y);
   return m_tPosition;
}
void CPhysicsObject::setPosition(const CCPoint &pos)
{
   if (NULL == m_Body)
   {
       return Sprite::setPosition(pos);
   }
   float angle = m_Body->GetAngle();
   m_Body->SetTransform(b2Vec2(pos.x / PTM_RATIO, pos.y / PTM_RATIO),
   angle);
}
float CPhysicsObject::getRotation()
{
   if (NULL == m_Body)
   {
       return Sprite::getRotation();
   }
   return CC_RADIANS_TO_DEGREES(m_Body->GetAngle());
}
void CPhysicsObject::setRotation(float fRotation)
{
   if (NULL == m_Body)
   {
       return Sprite::setRotation(fRotation);
   }
   else
   {
       b2Vec2 p = m_Body->GetPosition();
       float radians = CC_DEGREES_TO_RADIANS(fRotation);
       m_Body->SetTransform(p, radians);
   }
}
```

#### 10.4.3　碰撞处理

现在我们有了一个物理场景，以及物理节点，这个带有物理属性的节点可以正常显示，那么接下来还需要一个碰撞监听器，虽然这不是必须的，但在每次碰撞发生的时候，告诉节点，被碰了一下或者说跟谁碰到一起了是非常有用的。例如，愤怒的小鸟游戏，玩家发射出去的小鸟，不同的小鸟碰到不同的障碍，效果是不一样的，普通小鸟碰到冰块时穿透力很低，而蓝色小鸟碰到冰块时会有非常强的穿透力。黑色小鸟碰到障碍时会直接爆炸。这些都是由碰撞触发的，从而根据碰撞信息进行相对应的处理。

```
class CPhysicsListener :
    public b2ContactListener
{
public:
    CPhysicsListener(void);
    virtual ~CPhysicsListener(void);
    //当两个对象互相碰撞
    virtual void BeginContact(b2Contact* contact);
    //当两个对象碰撞结束
    virtual void EndContact(b2Contact* contact);
    //当两个对象准备进行物理模拟之前调用
    virtual void PreSolve(b2Contact* contact, const b2Manifold*
    oldManifold);
    //当两个对象完成了物理模拟之后调用
    virtual void PostSolve(b2Contact* contact, const b2ContactImpulse*
    impulse);
    //处理每一帧的物理碰撞事件
    void Execute();
private:
    std::set<CPhysicsObject*> m_PhysicsObjets;
};
```

想要处理好物理碰撞，那么就需要一个碰撞监听器，碰撞监听器的实现很简单，先写一个空的碰撞监听器，这个碰撞监听器的关键在于，如何把碰撞消息传递给物理节点，这需要通过一个Body获得一个PhysicsObject，那么最好的方法就是，将这个PhysicsObject放到Body的UserData中。因此在碰撞监听器中，需要重写4个碰撞回调函数，而在我们的物理节点基类中，也需要对应4个接口，来接收这4种碰撞消息。例如下面的代码。

```
void CPhysicsListener::PreSolve(b2Contact* contact, const b2Manifold*
oldManifold)
{
    //碰撞的第一个刚体如果是一个CPhysicObject（用userData判断），那么调用它的回调
    CPhysicsObject* objA = reinterpret_cast<CPhysicsObject*>
    (contact->GetFixtureA()
        ->GetBody()->GetUserData());
    if (NULL != objA)
    {
        objA->beforeSimulate(contact, oldManifold);
    }
    //接下来判断第二个刚体
    CPhysicsObject* objB = reinterpret_cast<CPhysicsObject*>
    (contact->GetFixtureB()
        ->GetBody()->GetUserData());
    if (NULL != objB)
    {
        objB->beforeSimulate(contact, oldManifold);
    }
}
```

其他几个接口的实现与其类似，都是简单地转发消息，但需要注意的一点是，不要在这些回调函数中改变刚体，因为这会对物理模拟造成影响，特别是不要删除刚体，否则可能导致程序崩溃。假设需要在碰撞发生的时候改变刚体，那么可以在碰撞发生的时候记录状态，在物理模拟完成之后，再进行改变。同样，删除刚体，也是需要在物理模拟完成之后再进行删除。

假设需要在碰撞的时候改变刚体的属性，例如，让一个物体在被碰到的时候破碎，如玻璃杯，或者是碰到某个物体之后变重，如海绵碰到水，这种情况下可以在监听器中增加一个Execute()方法，在World的Step执行之前或之后来执行该方法，在该方法中，将调用这一帧，所有触发碰撞的对象的一个方法，来执行这些操作，包括删除刚体。

```
void CPhysicsManager::update(float dt)
{
    //触发碰撞事件，交给监听者处理?
    m_ContactListener->Execute();
    int velocityIterations = 8;
    int positionIterations = 1;
    m_World->Step(dt, velocityIterations, positionIterations);
}
```

在每次事件触发的时候，对上面的代码小小改动一下，将PhysicsObject添加到一个容器中，缓存起来。

```
CPhysicsObject* objA = reinterpret_cast<CPhysicsObject*>
(contact->GetFixtureA()
    ->GetBody()->GetUserData());
if (NULL != objA)
{
    //添加到一个Set容器中
    m_PhysicsObjets.insert(objA);
    objA->beforeSimulate(contact, oldManifold);
}
```

然后在每一帧都会执行的Execute()方法中，遍历这一帧所有触发事件的对象，并调用它们的processOver()函数，在所有物理对象的processOver()函数中，可以根据当前的状态改变刚体或者销毁刚体。

```
for (set<CPhysicsObject*>::iterator iter = m_PhysicsObjets.begin(); iter !=
m_PhysicsObjets.end(); ++iter)
{
   CPhysicsObject* obj = *iter;
   obj->processOver();
}
m_PhysicsObjets.clear();
```

这里面有一个陷阱，可能导致一个对象被销毁之后，仍然触发其碰撞监听。这是非常可怕的一件事情，意味着程序很可能因此而崩溃！那就是在销毁一个刚体的时候，假设这个刚体正和其他对象发生了接触，那么这个时候，会有一个在Step之外的EndContact回调被触发，这是合理的，但很容易被忽视，并且产生BUG。正常的流程如图10-2所示，在一个Step中完成所有的触发，而图10-3演示了需要多个Step才能处理完一次接触的情况。

要解决这个BUG其实很简单，就是在释放刚体之前，先把刚体的UserData设置为NULL，这样回调流程就无法触发到PhysicsObject里面了。假设你的代码期望收到这个EndContact回调，那么需要在processOver()函数里面把好关，防止因为重复的processOver()函数调用，导致重复释放的问题，并且需要管理好UserData的引用计数。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120637.jpeg)

图10-2　一次Step完成

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120638.jpeg)

图10-3　需要多次Step

### 10.5　Box2d的相关工具

Box2d的编辑器不少，但功能大多比较简单，主要有PhysicsEditor、BoxCAD、Physics Body Editor、Vertex Helper等。编辑器主要可以用于编辑物理形状，这样就不需要在代码中使用大量的坐标编码来描述形状给Box2d，而是由编辑器来描述形状，我们只需要调用简单的接口，就可以把复杂的物理形状创建出来，大大减少了代码中的硬编码，同时也有利于后期物理形状的调整。

#### 10.5.1　PhysicsEditor介绍

PhysicsEditor与TexturePacker是同一个公司开发的，支持Windows和Mac，可以用来简单地编辑一些形状，然后导出Plist，其工具界面如图10-4所示，官方提供了一个简单的解析类供用户使用，虽然是收费软件，但仍可以免费使用，只不过有两个限制，即导出的Plist不能超过10个shape；每次导出要等5秒才能导出。这个工具用来编辑物体的物理形状，还是很不错的。在导出的时候，需要在导出界面的右上角选择导出格式，注意选择Box2D generic Plist选项导出。

使用PhysicsEditor进行编辑，一般的步骤如下。

（1）先把要编辑的物理对象的图片添加到PhysicsEditor中，如图10-5所示。

（2）为物理形状起一个名字，在**所有的物理形状中，这个名字必须是唯一的**，如图10-6所示。

（3）单击上方的形状按钮，添加形状到物理对象中，可以选择圆形、三角形、描点，然后调整添加的形状，最右边的两个按钮是对当前选择的图形进行镜像翻转，方便编辑一些对称形状，如图10-7所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120639.jpeg)

图10-4　PhysicsEditor编辑器

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120640.jpeg)

图10-5　添加图片

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120641.jpeg)

图10-6　为形状起名字

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120642.jpeg)

图10-7　工具栏

（4）选择抠图工具可以快速地勾勒出复杂的形状，其选项中，Tolerance（容差）值越高，顶点就越少，Alpha threshold（透明极限）值越高，抠图的区域（红色区域）就越精细。Trace mode（追踪模式）有Straight（直线模式）和Natural（自然模式）两个选项可以选择，直线模式的顶点会更少一些，自然模式会更精细一些。Frame mode（帧模式）适用于需要描述多个物理形状之间的相交或者集合，如图10-8所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120643.jpeg)

图10-8　抠图工具

（5）选择导出模式为Box2D generic (PLIST)尤为重要，因为这是要为Cocos2d-x、Box2d导出的形状文件，选择不同的导出模式会有不同的附加选项可供选择，如图10-9所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120644.jpeg)

图10-9　导出格式

（6）还需要填写锚点的值，一般都是将锚点设置为0.5,0.5，所以这里将Relative选项的值都填为0.5,0.5，然后为密度、弹性和摩擦力选项赋值，弹性和摩擦力的取值范围是0～1，而密度的单位应该是kg/m³，如图10-10所示。

（7）在属性面板的最下方还有一个多选菜单，左边的Cat.表示你自己的碰撞属性，右边的Mask表示你能与哪些物体碰撞，例如，这里你的碰撞属性是bit_0，则可以与所有对象碰撞，假设Mask的bit_0没有选中，那么这个物理对象不会和另外一个相同类型的物理对象发生碰撞，但可以和其他的物理对象发生碰撞。选中Cat.表示这个物理对象是什么，而选中Mask表示你能和什么物理对象发生碰撞（所有默认的物理对象的Cat.都是bit_0），如图10-11所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120645.jpeg)

图10-10　属性设置

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120646.jpeg)

图10-11　碰撞掩码设置

（8）最后，单击上方的Publish或Publish As按钮，就可以导出Plist文件了，Plist文件可以直接在Cocos2d-x中使用。在PhysicsEditor的安装目录下，Documentation目录中有如何使用PhysicsEditor的相关文档，Examples目录下有多种引擎使用PhysicsEditor的示例，Loaders目录下有多种引擎加载PhysicsEditor导出的Plist文件的加载器，Cocos2d-x只需要包含GB2ShapeCache-x.h和GB2ShapeCache-x.cpp文件即可。

#### 10.5.2　BoxCAD介绍

BoxCAD是一个在线编辑Box2d的网站，可以编辑各种形状和关节，然后播放演示，Dump Code按钮可以自动生成所编辑内容的AS代码，并下载下来，Box2d使用的各种语言代码都是比较相似的，可以简单修改一下，放到Cocos2d-x中（TestBed框架），如图10-12所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120647.jpeg)

图10-12　BoxCAD界面

#### 10.5.3　Physics Body Editor介绍

Physics Body Editor与PhysicsEditor相似，都是用Java实现的一个开源的编辑器，Java的东西都是跨平台的，但其只能导出JSON格式，虽然其界面看上去还不错，但目前还没有找到Cocos2d-x调用它的代码，如图10-13所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120648.jpeg)

图10-13　Physics Body Editor界面

#### 10.5.4　Vertex Helper介绍

Vertex Helper也是一个编辑形状的工具，但该工具是编辑完形状直接生成代码，生成的代码可以直接放到2dx代码中（BoxCAD是需要修改一下的），相对于BoxCAD，Vertex Helper并不支持编辑关节，也不支持物理模拟，并且只能在Mac下运行。

使用的时候需要先将参考图片拖进工具中，然后选择编辑模式，旋转Type和Style选项，接下来在图片上挨个单击，右下角的文本框中就会自动生成相对应的初始化代码了，如图10-14所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120649.jpeg)

图10-14　Vertex Helper界面

除此之外还有Mekanimo、PhysicsBench等工具，这里就不一一介绍了。