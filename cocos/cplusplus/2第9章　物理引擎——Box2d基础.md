# 第9章　物理引擎——Box2d基础

Box2d（http://Box2d.org/）是一个轻量级的，用于**2D游戏的刚体模拟的物理引擎**。所谓刚体，可以理解为硬的东西，它的尺寸固定，可以忽略形变，**在刚体内部，点和点之间的距离不会变化**。例如，笔者手下的键盘和鼠标，旁边的杯子，这些不容易变形的东西就是刚体。

那么有什么不是刚体呢？例如，杯子里面的水、身上穿的衣服、手边的手纸……它们被称为流体以及布料。和刚体对应的是软体，不是软件，如泥巴、面团、橡胶。在物理模拟中，这些东西的模拟是最麻烦的，而最好模拟的就是刚体了，PhysX物理引擎可以很好地模拟，而Box2d只能用刚体来模拟这一切。

虽然是轻量级的东西，但内容还是相当丰富的，本章会介绍Box2d的基础知识，并简单介绍Box2d是如何工作的。

对于Box2d的基础知识，Box2d官方的用户手册结合官方的testbed示例覆盖了Box2d的所有功能点，并且最新的Box2d源码中包含了中文版本的用户手册，由Antkillerfarm网友翻译，翻译的质量还是相当不错的。在https://github.com/erincatto/Box2d这里可以下载Box2d的源码包，解压后在Box2d/Documentation路径下可以找到manual_Chinese.docx文件。**强烈建议**读者下载下来看一下。如果读者已经掌握了Box2d的基础知识，可以跳过本章。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　核心概念。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　工作流程。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　物理世界World。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Body和Shape。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　关节Joint。

### 9.1　核心概念

首先来了解一下Box2d的核心概念，以下是Box2d的关键组成部分。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　世界World，一个物理世界就是刚体、形状、约束等相互作用的集合。Box2d支持创建多个世界，但一般不需要这么做。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　刚体body，物理世界中的一个物理对象，一个刚体可以由多个不同的形状组成，刚体上任意两点之间的距离是固定的。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　形状shape，用于碰撞检测的2D几何形状。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　夹具fixture，可将形状固定到刚体之上，并为形状添加密度、摩擦、恢复等材质特性。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　约束constraint，约束用于限制刚体的自由度，也就是限制刚体的移动或旋转。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　接触约束contact constraint，一个防止刚体穿透，以及用于模拟摩擦和恢复的特殊约束，由Box2d自动创建。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　关节Joint，用于将多个刚体固定到一起的约束。例如，我们的脚通过膝关节将大腿和小腿进行固定和约束。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　关节限制Joint limit，限制了一个关节的运动范围，如大腿和小腿无法进行360°的旋转。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　关节马达Joint motor，关节马达可以按照关节的自由度来驱动所连接的刚体。

### 9.2　工作流程

这里直接从Box2d自带的HelloWorld示例开始，从Box2d官网下载的源码包中可以找到这个例子，这是一个没有图形显示的HelloWorld，正因为如此，呈现的内容更加精简，后面的内容会介绍怎样把Box2d的模拟效果显示在Cocos2d-x上面，现在先把渲染抛到脑后吧。首先需要了解一下最纯粹的Box2d。

Box2d的运行流程大致如图9-1所示，首先是初始化Box2d，然后可以执行各种操作，如创建对象，操作对象、在循环中更新物理世界，Box2d会自动执行碰撞处理，并触发碰撞监听。需要注意的是，创建对象等操作不能在碰撞监听回调中执行。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120610.jpeg)

图9-1　运作流程

Box2d的物理模拟全部发生在b2World这个类中，其表示整个物理世界，使用Box2d的第一步就是要把b2World创建出来，构造函数很简单，只有一个参数，表示这个世界的重力加速度，地球的重力加速度是-9.8f，一般可以取近似值-10.0f（月球上的重力为地球的1/6大约为1.63f）。Box2d用b2Vec2来描述一个向量，重力也是一个向量，其结构等于一个Vec2，下面这段代码创建了一个世界。

```
b2Vec2 gravity(0.0f, -9.8f);
b2World world(gravity);
```

有了世界之后，需要在世界里面添加各种各样的对象，这些对象不需要并且**不允许**使用new来创建，应该直接调用World的Create系列函数，由World来创建。

Box2d中使用了SOA（小对象分配器）来快速有效地使用内存（能够高效地创建大量50～300字节大小的对象），Box2d每创建一个对象，都需要先填充一个对象的描述结构（这点和PhysX非常相似，这里使用一个b2BodyDef结构来填充描述。

```
b2BodyDef groundBodyDef;
groundBodyDef.position.Set(0.0f, -10.0f);
b2Body* groundBody = world.CreateBody(&groundBodyDef);
```

有了一个b2Body对象之后，还需要描述这个对象的形状，Box2d提供一个b2PolygonShape对象来描述一个多边形，调用它的SetAsBox()函数传入宽和高可以得到一个四边形，调用它的Set()函数传入一个顶点数组，可以得到一个N边形，把这个形状直接设置到b2Body对象中。

```
//创建一个宽为50.0f，高为10.0f的四边形，并设置到groundBody
b2PolygonShape groundBox;
groundBox.SetAsBox(50.0f, 10.0f);
groundBody->CreateFixture(&groundBox, 0.0f);
```

上面设置了地板的形状，它的质量为0，默认是一个静态的物体，静态的物理在物理世界中不会移动，但在游戏中，还需要创建一些动态的物体。要创建动态刚体的话，需要更多的设置，首先需要设置b2BodyDef的type，默认为b2_staticBody，动态物体需要将它设置为b2_dynamicBody。动态物体的形状描述也需要更多的内容，如质量、摩擦力……Box2d提供b2FixtureDef结构体来描述这些内容。

```
//定义动态Body并创建
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;
bodyDef.position.Set(0.0f, 4.0f);
b2Body* body = world.CreateBody(&bodyDef);
//定义另外一个四边形
b2PolygonShape dynamicBox;
dynamicBox.SetAsBox(1.0f, 1.0f);
//定义动态Body的配置
b2FixtureDef fixtureDef;
fixtureDef.shape = &dynamicBox;
//动态对象的密度不为0
fixtureDef.density = 1.0f;
//设置摩擦力
fixtureDef.friction = 0.3f;
//将形状配置到Body
body->CreateFixture(&fixtureDef);
```

初始完一个World以及相关的对象之后，需要让整个世界动起来，这需要调用World对象的Step()方法，其需要几个参数，第一个是从上一次调用到现在的时间间隔，这个参数可以直接使用Cocos2d-x在Update()函数中传入的参数。一般情况下，在每一帧的Update()调用一次Step。Box2d当运行比较复杂的运算的时候，需要一定的迭代次数来保证模拟的精度，Step要求传入两个迭代次数，即需要传入速度迭代数和位置迭代数，Box2d给出的默认值是8和3，如果对精度要求比较高，可以适当地增加这两个值，该值越大，相应地性能越低。velocityIterations表示速度约束的遍历次数，positionIterations表示位置约束的遍历次数，可以根据游戏的具体情况来设置这两个值。

```
world.Step(dt, velocityIterations, positionIterations);
```

在游戏更新的过程中，会不断地创建新的对象到世界中，也会不断地删除一些旧的对象，不管怎样，在World被析构的时候，这个世界的一切，都会从内存中被释放。

### 9.3　物理世界World

Box2d的World代表了整个物理世界，我们需要做的只是一些配置——需要往世界里面添加一些怎样的对象，然后让它运行起来就可以了。在游戏循环中，每一帧进行一次模拟，然后把模拟的结果渲染出来。

我们可以动态地查询World中现在某些对象的状态，也可以查询现在某个范围内有哪些对象，这些其都能做到。World的函数很多，这里只简单介绍几个适合入门读者学习的函数。

```
///传入一个描述来创建一个Body，这个函数不会对描述对象有任何的引用，请放心地释放它
///这个函数在callback()函数中是被锁定的（无法在一次Step中动态创建或者销毁）
b2Body* CreateBody(const b2BodyDef* def);
///释放指定的Body
///这个操作会自动释放它的所有形状以及关节
///这个函数在callback()函数中是被锁定的（无法在一次Step中动态创建或者销毁）
void DestroyBody(b2Body* body);
///创建一个关节把两个Body联系在一起
///这个函数在回调的时候是被锁定的
b2Joint* CreateJoint(const b2JointDef* def);
///销毁一个关节，这可能造成这个关节连接的两个对象发生碰撞
///这个函数在callback()函数中是被锁定的（无法在一次Step中动态创建或者销毁）
void DestroyJoint(b2Joint* joint);
///在一个时间步内，进行物理模拟，碰撞检测，约束解决等....
///@param timeStep要模拟的总时间长度，以秒为单位
///@param velocityIterations速率约束求解器的迭代次数
///@param positionIterations位置约束求解器的迭代次数
void Step( float32 timeStep,
int32 velocityIterations,
int32 positionIterations);
///设置全局的重力加速度
void SetGravity(const b2Vec2& gravity);
///获取全局的重力加速度
b2Vec2 GetGravity() const;
```

### 9.4　Body和Shape

Body表示在物理世界中的一个对象，往往也对应游戏世界中的一个对象，物体具有位置、角度、质量、速度等属性，每个刚体都包含一个形状列表，这意味着**一个刚体可以由很多个形状组成**，如由两个圆形加一个长方形组成一个哑铃，刚体总共可以分为**静态、动态以及动力学**3种。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　静态刚体永远不会移动，不管其他刚体以多大的力量推动它，它总是岿然不动，也不会与其他静态刚体碰撞。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　动态刚体表示场景中可以运动的对象，可以和任何刚体发生碰撞。撞到动力学刚体和撞到静态刚体都会被弹开，撞到动态刚体，会互相作用，也就是可能互相弹开，或者其中一个撞飞另外一个。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　动力学刚体可以移动，**但不受任何力的影响**，动态刚体对其施加的力，或者ApplyForce对其施加的力都无效，和静态刚体不会发生碰撞，动力学刚体之间，也不会发生碰撞，但是**动力学刚体的运动，可以对动态刚体产生力的作用**。

#### 9.4.1　刚体的碰撞

在笔者的测试代码中，一个向上移动的动力学刚体，碰到动态刚体，会将动态刚体往上顶起，碰到其他的动力学刚体，会穿过，而碰到静态刚体，也是直接穿过！动力学刚体并不会和静态刚体发生碰撞！当地面上有个动态刚体时，这时候一个动力学刚体从正上方往下运动，会将动态刚体缓缓压入地面。

由于动力学刚体不受力或冲量的作用，所以只能通过Body->**SetLinearVelocity设置线性速度或者关节的马达驱动**，来使其移动。如图9-2～图9-4演示了动力学刚体与各种刚体碰撞时的表现。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120611.jpeg)

图9-2　动力学刚体和动力学刚体互相穿透

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120612.jpeg)

图9-3　动力学刚体和动态刚体互相碰撞

如表9-1总结了各种刚体的碰撞情况，如果读者觉得动力学刚体难以理解，可以想一想在超级玛丽等游戏中可以跳上去的那些来回移动的平台，它可以托着游戏者移动，但却可以不受游戏者的影响。现实生活中的电梯也勉强可以理解为动力学刚体。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120613.jpeg)

图9-4　动力学刚体和静态刚体互相穿透

表9-1　各种刚体的碰撞情况

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120614.jpeg)

在游戏中经常会碰到一些单面墙壁，例如，是男人就上100层这样的游戏中，游戏者从下往上跳时可以穿过平台，而从上往下掉落时却会被平台挡住，这就需要使用到Box2d的碰撞监听了。当监听到碰撞事件时，判断角色运动的方向是向上还是向下，根据判断的结果来决定是否禁用此次碰撞。

#### 9.4.2　创建刚体

要创建一个Body需要传入一个物体的定义，b2BodyDef是一个内容丰富的结构，其默认构造函数如下，一般在填充完刚体描述对象的type和position之后，就会调用World的CreateBody接口来创建一个新的Body。

```
b2BodyDef()
{
    userData = NULL;                 //默认没有额外数据
    position.Set(0.0f, 0.0f);        //默认的位置和旋转角度
    angle = 0.0f;
    linearVelocity.Set(0.0f, 0.0f);  //线性加速度和角速度等
    angularVelocity = 0.0f;
    linearDamping = 0.0f;
    angularDamping = 0.0f;
    allowSleep = true;
    awake = true;                    //一开始就是醒着的状态
    fixedRotation = false;
    bullet = false;                  //当物体需要在高速运动中进行碰撞，开启它
    type = b2_staticBody;            //默认是一个静态物体
    active = true;
    gravityScale = 1.0f;
}
```

刚体的大致结构如图9-5所示，每个刚体都会有其对应的一些形状，在Box2d中通过Fixture来描述这些形状，以及它们的密度、摩擦、弹性等。

使用Body的CreateFixture可以为刚体设置一个新的形状，也可以为刚体设置很多形状，通过Body的SetGravityScale还可以设置刚体的重力缩放，某些物体可以让其不受重力的影响。

物体的摩擦系数是一个0～1之间的数，0表示非常光滑，1表示非常粗糙。物体的弹性系数（恢复系数）也是一个0～1之间的数，0表示没有弹性，1表示没有能量损失的反弹。密度也是一个必须要设置的数据，假设将密度设置为0，物体仍然会往下掉落，但是没有任何重量的物体堆叠起来就会变成如图9-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120615.jpeg)

图9-5　刚体的组成

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120616.jpeg)

图9-6　密度为0的堆箱子

在Box2d里可以创建的形状包含Circle圆形、Edge边界线、Polygon多边形以及Chain链条。多边形是一个闭合的多边形，而Chain是一系列点，可以理解为一个不闭合的多边形。形状主要是用于物理模拟中的碰撞检测，在创建完Body之后，对Body进行设置。下面看一下如何创建多边形。

```
b2CircleShape circle;            //圆形
circle.m_radius = 1.0f;          //半径为1
b2EdgeShape edge;
edge.Set(b2Vec2(-100.0f, 0.0f),  //创建一条平行于x轴的边
    b2Vec2(100.0f, 0.0f))
b2PolygonShape polygon;          //创建一个封闭的多边形
polygon.SetAsBox(5.0f, 5.0f);    //创建一个长和宽都为10的正方形
b2ChainShape shape;              //链形
b2Vec2* arr = new b2Vec2[5];
arr[0] = b2Vec2(0.0f, 0.0f);
arr[1] = b2Vec2(10.0f, 10.0f);
arr[2] = b2Vec2(20.0f, 0.0f);
arr[3] = b2Vec2(30.0f, 10.0f);
arr[4] = b2Vec2(40.0f, 0.0f);    //数组描述了一个'W'形状
shape.CreateChain(arr, 5);       //传入一系列点来确定形状
delete[] arr;
```

在创建闭合多边形的时候，也可以使用Set()函数，像Chain一样，传入一个顶点数组来创建形状，在为Body创建Fixture时，将shape对象的指针赋值给b2FixtureDef对象的shape变量即可。

上面的Edge常用于创建世界的边界，其确实很适合做这个工作，而且做得要比Polygon要好。至于Chain，感觉像是一系列连续的Edge。

SetAsBox是个很好用的接口，可以节省代码，需要注意的一点是，**传入的参数是宽和高的一半，函数会以当前位置为中心，创建一个四边形**。

填充好Shape数据之后，需要把它赋给对应的Fixture，同时为Fixture填充弹性、摩擦力、质量之类的属性。最后调用Body的CreateFixture，绑定到刚体之上。

### 9.5　关节Joint

关节，或者叫连接器，是用来连接两个刚体的对象，关节是一个很形象的比喻，就像人们身上的关节，如膝关节。在我们身边还有很多这样的例子，如可以推开的门、眼镜的镜架、汽车的轮子，这些把两个物体连接在一起，又在一定范围内可以移动或者旋转的对象，都可以称之为关节，关节是一种约束对象，对连接的两个物体进行约束。Box2d提供了下面10种关节。

```
enum b2JointType
{
    e_unknownJoint,
    e_revoluteJoint,   //旋转关节
    e_prismaticJoint,  //平移关节
    e_distanceJoint,   //距离关节
    e_pulleyJoint,     //滑轮关节
    e_mouseJoint,      //鼠标关节
    e_gearJoint,       //齿轮关节
    e_wheelJoint,      //滚轮关节
    e_weldJoint,       //焊接关节
    e_frictionJoint,   //摩擦关节
    e_ropeJoint        //绳索关节
};
```

#### 9.5.1　使用关节

在详细介绍每个关节之前，先来看一下在代码中如何使用关节。创建一个关节的步骤和创建刚体十分相似，都是先填充一个关节定义结构，然后放到World中，由World来创建这个关节，每个关节都必须包含两个刚体。

对于不同的关节，需要在userData中设置关节所需的信息，World的CreateJoint返回一个b2Joint指针，但b2Joint只是一个抽象类，World返回的b2Joint实际是一个b2DistanceJoint或b2RevoluteJoint之类的对象。

```
struct b2JointDef
{
   b2JointDef()
   {
       type = e_unknownJoint;
       userData = NULL;
         bodyA = NULL;
         bodyB = NULL;
         collideConnected = false;
    }
    ///关节的类型
    b2JointType type;
    ///特殊数据
    void* userData;
    ///第一个刚体
    b2Body* bodyA;
    ///第二个刚体
    b2Body* bodyB;
    ///两个刚体是否会发生碰撞
    bool collideConnected;
};
```

除了上面的设置，关节还有3个通用的概念。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　锚点：与Cocos2d-x的锚点概念很相似，是运用于旋转移动计算的一个点，如图9-7所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120617.jpeg)

图9-7　锚点

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　限制：每个关节都有其限制条件，如旋转角度、移动距离等（车轮和轮杆可以进行360°的旋转，却不能移动，人的大腿和小腿大约只能允许180°以内的旋转，非正常情况不算在内）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　马达：用来描述关节运动，并在物理模拟中制造关节的运动，可以是旋转或者移动。

得到一个b2Joint对象之后，可以做哪些操作呢？可以获取关节的信息，可以控制整个关节，如重新描述它的马达，更新限制等。

#### 9.5.2　旋转关节RevoluteJoint

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120618.jpeg)

图9-8　旋转关节

如图9-8所示，旋转关节就像是一个点，把两个刚体联系起来，并且限制它们只能绕着这个点旋转。这个点也可以是在刚体形状以外的一个点。例如，电风扇的叶子都可以围绕着中心的公共点旋转。

使用旋转关节的步骤如下，首先填充一个b2RevoluteJointDef结构体，然后调用World的CreateJoint()函数即可，我们来看一下b2RevoluteJointDef结构体。

```
struct b2RevoluteJointDef : public b2JointDef
{
    ///使用一个世界坐标作为两个刚体的连接点，来初始化关节
    void Initialize(b2Body* bodyA, b2Body* bodyB, const b2Vec2& anchor);
    ///A对象的锚点，位于A对象的局部坐标系，是A的旋转点
    b2Vec2 localAnchorA;
    ///B对象的锚点，位于B对象的局部坐标系，是B的旋转点
    b2Vec2 localAnchorB;
    ///参照角度（一个旋转90°的关节，参照角度为0时，返回的旋转是90°，参照角度为
    10°时，返回100°）一般在设置连接器限制时，这个参数会比较有用。b2RevoluteJoint
    的GetJointAngle返回的是两个连接器的相对角度
    float32 referenceAngle;
    ///是否启用限制
    bool enableLimit;
    ///角度最低限制，逆时针方向
    float32 lowerAngle;
    ///角度最高限制，逆时针方向
    float32 upperAngle;
    ///是否启用马达
    bool enableMotor;
    ///马达的目标速度（受限于最大扭力）
    float32 motorSpeed;
    ///马达可以达到的最大扭力
    float32 maxMotorTorque;
};
```

下面的代码演示了如何创建一个旋转关节。

```
//添加旋转关节
b2RevoluteJointDef revdef;
revdef.Initialize(boxbd, circlebd, b2Vec2(0.0f, 10.0f));
//开启马达
revdef.enableMotor = true;
revdef.maxMotorTorque = 105;
revdef.motorSpeed = 30.14f;
m_world->CreateJoint(&revdef);
```

RevoluteJoint的Initialize()函数会使用两个Body的锚点作为旋转锚点，并且根据输入的第3个参数，在世界坐标系中，根据当前两个Body在世界坐标系的位置，作为两个物体的公共顶点。如图9-9和图9-10演示了基于指定公共点的旋转关节运动情况。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120619.jpeg)

图9-9　旋转关节1

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120620.jpeg)

图9-10　旋转关节2

关于马达和限制，在启用马达之后，**第二个物体**会围绕公共点开始旋转，马达会达到motorSpeed的旋转速度，这个旋转的扭力矩不会高于maxMotorTorque。**motorSpeed的单位是弧度**，而maxMotorTorque的单位是N/m，一个力的单位。给马达赋予一个正的速度，会让其按照逆时针的方向旋转，负的速度会让其顺时针旋转。如图9-11和图9-12演示了马达的开启情况，在创建关节时，这里分别将四边形和圆形作为第二个物体传入。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120621.jpeg)

图9-11　四边形马达

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120622.jpeg)

图9-12　圆形马达

启用限制可以限制旋转的角度，可以限制一个超过360°的值，例如3600°，马达在旋转10圈之后，达到限制值会停下来，而手动让其旋转，也无法突破其限制值。

#### 9.5.3　平移关节PrismaticJoint

平移关节可以将两个物体之间的限制在一个特定的方向上平移，就像滑动门和垂直电梯，滑动面只可以向左右两边移动，而垂直电梯一般只能上下移动，平移关节阻止了物体的相对旋转。

```
//添加平移关节
b2PrismaticJointDef pridef;
pridef.Initialize(boxbd,       circlebd,  b2Vec2(0.0f,  20.0f),  b2Vec2(1.0f,
0.0f));
m_world->CreateJoint(&pridef);
```

b2PrismaticJointDef的Initialize()函数通过传入两个刚体，以**及一个公共顶点，还有一个轴来初始化**。公共顶点与旋转关节的意义一样，都是世界坐标系下的顶点。传入的轴表示一个方向，用来限制两个刚体的相对移动和旋转，假设**传入的轴是(0, 0)，那么两个刚体的相对移动不受限制**，但两个物体不会产生相对旋转，它还会令马达和限制失效。上面的代码为圆和正方形创建了一个只能在x轴上移动的平移关节，运行效果如图9-13和图9-14所示，两个节点只能沿着**相对x轴**进行移动。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120623.jpeg)

图9-13　平移关节1

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120624.jpeg)

图9-14　平移关节2

#### 9.5.4　距离关节DistanceJoint

距离关节将两个物体之间的距离保持在一定范围内的关节。例如，用弹簧拴住两个物体，也可以像一根棍子一样，让两个物体的距离固定不变，如图9-15所示。

```
//添加距离关节
b2DistanceJointDef disdef;
disdef.Initialize(boxbd, circlebd,
    b2Vec2(2.0f, 10.0f), b2Vec2(-2.0f, 10.0f));
disdef.localAnchorA = b2Vec2(0.0f, 0.0f);
disdef.localAnchorB = b2Vec2(0.0f, 0.0f);
//震动频率
disdef.frequencyHz = 2.0f;
//阻尼率
disdef.dampingRatio = 0.0f;
m_world->CreateJoint(&disdef);
```

通过设置frequency频率和damping ratio可以获得柔软的效果，frequency表示震荡的频率，单位是Hz，相当于游戏刷新频率的一半，阻尼率的取值一般在0～1之间，震荡幅度从0～1由大变小，如图9-16是使用柔软的距离关节做出的网的效果，可以参考TestBed的Web示例。下面这个例子的frequency取值是2.0而damping ratio取值是0。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120625.jpeg)

图9-15　距离关节

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120626.jpeg)

图9-16　使用距离关节模拟网

#### 9.5.5　滑轮关节PulleyJoint

滑轮关节可以用来创建一个滑轮的效果，滑轮的两端连接着两个绳子，绳子绑着两个物体，当一个物体上升的时候，另一个物体就下降，如图9-17和图9-18可以很简单清晰地描述出这个效果。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120627.jpeg)

图9-17　滑轮关节1

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120628.jpeg)

图9-18　滑轮关节2

```
//添加滑轮关节
b2PulleyJointDef puldef;
puldef.Initialize(boxbd, circlebd,
b2Vec2(-2, 15), b2Vec2(2, 15),      //A和B滑轮悬挂点的世界坐标
b2Vec2(-2, 10), b2Vec2(2, 10),      //A和B滑轮挂载点的世界坐标
     0.1f);                         //滑轮线的阻尼率
m_world->CreateJoint(&puldef);
```

#### 9.5.6　鼠标关节MouseJoint

鼠标关节主要用于方便我们用鼠标来拖动物体，它会先确定物体上的一个点，对这个点施加力，使其向鼠标的位置移动。被拖曳的物体可以自由旋转，可以为鼠标关节设置最大力矩来决定力的大小，设置频率和阻尼率，来达到弹簧和减震器的效果。

要实现用鼠标关节来拖曳一个刚体的效果，首先需要先实现单击到刚体的判断，并不是添加了鼠标关节之后，这个刚体就可以被单击了。

因为Box2d并不关注单击事件的实现，点击判断这个功能需要开发者自己来实现，在Box2d里面实现这个功能很简单，可以参考Box2d TestBed里面的Test.cpp文件，里面继承了b2QueryCallback类，用于查询在一个包围盒中的刚体，实现了ReportFixture()方法，在ReportFixture()中，会传入一个b2Fixture，如果判断你的鼠标（或手指）在这个对象的范围内，那么返回false，并保存选中的Fixture，这时候会终止查询，否则返回true。

```
class QueryCallback : public b2QueryCallback
{
public:
   QueryCallback(const b2Vec2& point)
   {
       m_point = point;
       m_fixture = NULL;
   }
   bool ReportFixture(b2Fixture* fixture)
   {
       b2Body* body = fixture->GetBody();
       if (body->GetType() == b2_dynamicBody)
       {
          bool inside = fixture->TestPoint(m_point);
          if (inside)
          {
              m_fixture = fixture;
              //We are done, terminate the query.
              return false;
          }
       }
       //Continue the query.
       return true;
   }
   b2Vec2 m_point;
   b2Fixture* m_fixture;
};
```

在鼠标按下的时候，需要查询鼠标是否单击到动态刚体，（这里的MouseDown应该由TouchBegin系列函数调用）这时候用到了World的QueryAABB()函数来进行检测，首先在鼠标单击的地方，构造一个非常小的碰撞盒，然后执行查询，这时候位于这个碰撞盒之内的Fixture会被传入到查询对象中，在查询对象中进行一次过滤，只**查询动态刚体**，并且判断点是否在Fixture之内，这属于边界情况的过滤，刚体位于这个极小的碰撞盒之内，但单击的坐标点并没有落在这个Fixture之内。通过这层层的过滤，在单击到刚体之后，就可以开始创建鼠标关节了。

```
void Test::MouseDown(const b2Vec2& p)
{
   m_mouseWorld = p;
   if (m_mouseJoint != NULL)
   {
       return;
   }
   //Make a small box.
   b2AABB aabb;
   b2Vec2 d;
   d.Set(0.001f, 0.001f);
   aabb.lowerBound = p - d;
   aabb.upperBound = p + d;
   //Query the world for overlapping shapes.
   QueryCallback callback(p);
   m_world->QueryAABB(&callback, aabb);
   if (callback.m_fixture)
   {
       b2Body* body = callback.m_fixture->GetBody();
       b2MouseJointDef md;
       md.bodyA = m_groundBody;
       md.bodyB = body;
       md.target = p;
       md.maxForce = 1000.0f * body->GetMass();
       m_mouseJoint = (b2MouseJoint*)m_world->CreateJoint(&md);
       body->SetAwake(true);
   }
}
```

要创建一个鼠标关节很简单，将地面（或者其他静态刚体）设置为BodyA，然后将要拖动的刚体设置为BodyB，并设置要移动到的目标点，以及最大力矩，上面代码中是用1000乘以刚体的质量作为最大力矩，还需要将拖动的刚体的状态设置为Awake。

#### 9.5.7　齿轮关节GearJoint

齿轮关节用于模拟现实中的齿轮，可以用一堆形状来描述一个齿轮，然后用旋转关节让这个齿轮旋转，也可以起到这样一个效果，但并不高效，并且在排列齿轮牙齿的时候需要小心翼翼。有了齿轮关节，就可以直接用一个齿轮关节来实现一个齿轮，如图9-19所示。

齿轮关节和其他关节不一样的地方在于，**它是一个依赖其他关节的关节**。一般的关节只是依赖于关节连接着的两个刚体。通过一个关节的运动，来驱动另外一个关节。如果在删除齿轮关节之前，删除了其他关节，那么程序将会崩溃。在释放的时候，需要先释放齿轮关节，然后再释放挂载在齿轮关节上的其他关节。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120629.jpeg)

图9-19　齿轮关节

下面的代码将创建两个齿轮关节，以及两个旋转关节和一个平移关节，通过齿轮关节的旋转带动其他关节。

```
//第一个小圆
b2CircleShape circle1;
circle1.m_radius = 1.0f;
//第二个大圆
b2CircleShape circle2;
circle2.m_radius = 2.0f;
//长方形
b2PolygonShape box;
box.SetAsBox(0.5f, 5.0f);
//小圆刚体
b2BodyDef bd1;
bd1.type = b2_dynamicBody;
bd1.position.Set(-3.0f, 12.0f);
b2Body* body1 = m_world->CreateBody(&bd1);
body1->CreateFixture(&circle1, 5.0f);
//小圆和静态地面用旋转关节连接，小圆只可以旋转，不能移动
b2RevoluteJointDef jd1;
jd1.bodyA = ground;
jd1.bodyB = body1;
jd1.localAnchorA = ground->GetLocalPoint(bd1.position);
jd1.localAnchorB = body1->GetLocalPoint(bd1.position);
jd1.referenceAngle = body1->GetAngle() - ground->GetAngle();
m_joint1 = (b2RevoluteJoint*)m_world->CreateJoint(&jd1);
//大圆刚体
b2BodyDef bd2;
bd2.type = b2_dynamicBody;
bd2.position.Set(0.0f, 12.0f);
b2Body* body2 = m_world->CreateBody(&bd2);
body2->CreateFixture(&circle2, 5.0f);
//大圆和静态地面用旋转关节连接
b2RevoluteJointDef jd2;
jd2.Initialize(ground, body2, bd2.position);
m_joint2 = (b2RevoluteJoint*)m_world->CreateJoint(&jd2);
//长方形刚体
b2BodyDef bd3;
bd3.type = b2_dynamicBody;
bd3.position.Set(2.5f, 12.0f);
b2Body* body3 = m_world->CreateBody(&bd3);
body3->CreateFixture(&box, 5.0f);
//平移关节限制长方形刚体的移动
b2PrismaticJointDef jd3;
jd3.Initialize(ground, body3, bd3.position, b2Vec2(0.0f, 1.0f));
jd3.lowerTranslation = -5.0f;
jd3.upperTranslation = 5.0f;
jd3.enableLimit = true;
m_joint3 = (b2PrismaticJoint*)m_world->CreateJoint(&jd3);
//连接两个圆形的齿轮关节
//当一个小圆转动时，另一个小圆会跟着转动
//小圆连接的关节旋转一周，大圆连接的关节旋转1/2周
b2GearJointDef jd4;
jd4.bodyA = body1;
jd4.bodyB = body2;
jd4.joint1 = m_joint1;
jd4.joint2 = m_joint2;
jd4.ratio = circle2.m_radius / circle1.m_radius;
m_joint4 = (b2GearJoint*)m_world->CreateJoint(&jd4);
//连接大圆和长方形的齿轮关节
//当大圆所在的旋转关节转动时，会驱动平移关节跟着移动
//平移关节发生移动时，也会驱动旋转关节
b2GearJointDef jd5;
jd5.bodyA = body2;
jd5.bodyB = body3;
jd5.joint1 = m_joint2;
jd5.joint2 = m_joint3;
jd5.ratio = -1.0f / circle2.m_radius;
m_joint5 = (b2GearJoint*)m_world->CreateJoint(&jd5);
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120630.jpeg)

图9-20　齿轮关节

上面的代码运行结果如下，拖动任意一个刚体，会导致关节发生平移运动或者旋转运动，这时运动将会通过齿轮关节，传达到另外一个关节上，按照ratio参数的比例，来赋予另外一个关节运动的能量。在设置齿轮关节的时候，最好先在脑海中想清楚，齿轮动起来是怎样的。齿轮关节的创建本身很简单，上面如此冗长的代码，主要是创建了其他的关节。运行效果如图9-20所示。

#### 9.5.8　滚轮关节WheelJoint

滚轮关节用于模拟汽车的轮子，滚轮关节连接汽车和汽车轮子，汽车轮子可以滚动，滚轮关节还提供了一个弹簧效果，当发生车震的时候，汽车会上下震动，读者可以想像一下一辆越野车在颠簸的路上前进，轮子在转动，轮子和车身的距离不断地放大、缩小，如图9-21所示。

滚轮关节相当于一个旋转关节和一个带弹簧的距离关节组合而成，如图9-22所示的车轮子，当车子被甩起来的时候，轮子会被带出一些，而当车子被压下去的时候，轮子也会跟着陷下去。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120631.jpeg)

图9-21　滚轮关节

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120632.jpeg)

图9-22　使用滚轮关节模拟车轮

```
b2WheelJointDef jd;
//滚轮关节两个刚体可以移动的方向是y轴，也就是上下移动
b2Vec2 axis(0.0f, 1.0f);
//car表示车身，wheel1和wheel2分别表示车的前后轮子，这是一辆两轮车
jd.Initialize(m_car, m_wheel1, m_wheel1->GetPosition(), axis);
//顺时针方向旋转
jd.motorSpeed = -10.0f;
//这个轮子的马力更大一些
jd.maxMotorTorque = 20.0f;
jd.enableMotor = true;
jd.frequencyHz = 4.0f;
jd.dampingRatio = 0.7f;
m_spring1 = (b2WheelJoint*)m_world->CreateJoint(&jd);
//第二个轮子
jd.Initialize(m_car, m_wheel2, m_wheel2->GetPosition(), axis);
jd.motorSpeed = -10.0f;
jd.maxMotorTorque = 10.0f;
jd.enableMotor = false;
jd.frequencyHz = 4.0f;
jd.dampingRatio = 0.7f;
m_spring2 = (b2WheelJoint*)m_world->CreateJoint(&jd);
```

#### 9.5.9　焊接关节WeldJoint

焊接关节尝试限制两个刚体之间所有的相对运动，但是焊接关节并不是很稳定，效果看起来有些柔软，就像跳水运动员起跳时踩着的跳水板一样。用焊接关节将一大堆动态刚体（小四边形）依次连接起来，在受到力的作用下，摇摇晃晃，就是这种效果，如图9-23所示。

上面是由若干小四边形组成的一小块板子，每两个小四边形都用Weld焊接关节连接起来，可以设置dampingRatio来设置它们之间连接在一起的弹性，使用方法非常简单，设置好frequencyHz和dampingRatio，然后传入两个刚体，就可以直接创建焊接关节，Testbed的Cantilever很好地介绍了焊接关节的使用。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120633.jpeg)

图9-23　焊接关节

```
b2PolygonShape shape;
shape.SetAsBox(0.5f, 0.125f);
b2FixtureDef fd;
fd.shape = &shape;
fd.density = 20.0f;
//设置弹性
b2WeldJointDef jd;
jd.frequencyHz = 8.0f;
jd.dampingRatio = 0.7f;
//将所有物体连在一起
b2Body* prevBody = ground;
for (int32 i = 0; i < e_count; ++i)
{
    b2BodyDef bd;
    bd.type = b2_dynamicBody;
    bd.position.Set(5.5f + 1.0f * i, 10.0f);
    b2Body* body = m_world->CreateBody(&bd);
    body->CreateFixture(&fd);
    if (i > 0)
    {
//创建关节
        b2Vec2 anchor(5.0f + 1.0f * i, 10.0f);
        jd.Initialize(prevBody, body, anchor);
        m_world->CreateJoint(&jd);
    }
    prevBody = body;
}
```

#### 9.5.10　摩擦关节FrictionJoint

摩擦关节会制造“从上到下”的摩擦，提供了2D的角摩擦和平移摩擦，单从这句话来看，确实相当费解。想像一下，你拉着一辆车努力地向前冲，会受到一整辆车的摩擦力的影响。想像一下，你在天空中飞速翱翔，速度越快，会感觉到越强大的空气阻力，差不多就是这种感觉了，如图9-24所示。

使用了摩擦关节的小方块们，在被撞击到的时候，会缓缓移动，下面例子的重力加速度设置为0。

```
b2FrictionJointDef jd;
jd.localAnchorA.SetZero();
jd.localAnchorB.SetZero();
jd.bodyA = ground;
jd.bodyB = body;
jd.collideConnected = true;
jd.maxForce = mass * gravity;
jd.maxTorque = mass * radius * gravity;
m_world->CreateJoint(&jd);
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120634.jpeg)

图9-24　摩擦关节

通过设置maxForce来限制刚体移动的阻力，通过设置maxTorque来限制刚体旋转时的阻力，具体可以参考Testbed的ApplyForce示例，Testbed是Box2d官方源码的示例，相当于Box2d的cpp-tests（cpp-test是Cocos2d-x的示例集合，属于Cocos2d-x的常识）。

#### 9.5.11　绳索关节RopeJoint

绳索关节用来模拟绳子，绳子的两端绑着两个刚体，两个刚体可以在绳子限定的范围内自由地移动和旋转，与现实世界使用的绳子没什么两样，这是一根柔软的，拉不断的绳子，如图9-25所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623120635.jpeg)

图9-25　绳索关节

通过设置b2RopeJointDef的maxLength，可以限定绳子的长度，通过传入两个刚体，可以在这两个刚体身上系一条绳子。

最后，在删除的时候，必须先删除关节，之后才可以删除关节上的物体，否则程序会崩溃。