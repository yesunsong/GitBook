中文链接:https://blog.csdn.net/i_dovelemon/article/details/25798677

英文链接:https://www.gamedev.net/tutorials/_/technical/game-programming/understanding-component-entity-systems-r3013/

​	一般来说，我们实现游戏实体都是采用面向对象的方法进行编程。每一个实体都是一个对象，并且需要一个基于类的实例化系统，允许实体通过多态来扩展。但是，这样的方法，往往导致系统中出现大量的类，造成类爆炸的情况出现。随着新的实体出现，我们发现很难在类继承图中添加新的实体，特别是当这个实体需要很多不同类型的功能的时候。你可以看下下面的一个简单的类图继承。一个静态的敌人，并不能够很好的继承出来。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/9947c0af99e1e9204f46ad3dade17901)



​	为了解决这样的问题，游戏开发人员想出了通过组合而不是继承的方法来进行实体的构建。一个实体，就是一群组件的聚合，通过这样的方式，它具有以下面向对象方法所不具有的好处:

​	1.容易添加新的复杂的实体类型

​	2.容易定义新的实体数据

​	3.更加的高效率



​	下面是如何实现实体的一种方式。注意，这里的组件都是纯粹的数据，没有任何的方法，在下面会详细解释为什么这么做。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/9197e0310b894e1a78d040cab4af1c5f)



# 组件

​	一个组件可以使用C中的结构体来进行设计。它没有方法，只是用来存储一些数据，并不在它之上进行动作。一个经典的实现方式是，每一个不同的组件都继承至一个抽象的Componet类，通过这样的方法我们能够在运行时动态的添加组件，识别组件。每一个组件都描述了实体的某个属性特征。当他们单独存在的时候，实际上是没有任何意义的，但是当多个组件通过系统的方式组织在一起，就能够发挥强大的力量。我们可以使用空的组件来对实体进行标记，从而能够在运行时动态的识别它。



## 例子

- Position(x,y)
- Velocity(x,y)
- Physics(body)
- Sprite(images, animations)
- Heath(value)
- Character(name, level)
- Player(empty)



# 实体

​	一个实体指的是存在于你的游戏世界中的物体。实体在代码上就是一个组件的列表。由于实体的结构实在是太简单了，所以很多实现都没有专门的设计一个实体的数据结构。相反的，一个实体就是一个ID，所有组成这个实体的组件将会被这个ID给标记，从而明确的知道哪些组件是属于哪个实体的。如果你想的话，你可以在运行时，动态的将组件从实体中移除或者增加一个或多个你感兴趣的组件。比如说，如果玩家发出了一个冰系魔法，将敌人冻住，你只要简单的将它的速度组件移除，那么敌人就静止住了。



## 例子

- Rock(Position, Sprite)
- Crate(Position,Sprite,Health)
- Sign(Position,Sprite,Text)
- Ball(Position,Velocity,Physics,Sprite)
- Enemy(Position,Velocity,Sprite,Character,Input,AI)
- Player(Position,Velocity,Sprite,Character,Input,Player)



# 系统

​	注意，我在上面没有提到任何和游戏逻辑相关的话题。游戏逻辑是系统需要进行的工作。一个系统就是对所有相关联的组件记性操作，比如说，同一个实体的组件。举个例子，人物的移动系统可能会对位置(Position)，速度(Velocity)，碰撞(Collider)，和输入(Input)进行操作。每一个系统，都会在每一帧中按照逻辑上的顺序进行更新。如果要让一个角色跳起来，我们只要检测下Input中的keyJump按键是否被按下，如果是，那么系统就会查看下载Collider中是否有一个接触了地面，如果是，就将这个实体的Velocity的y速度设置一下，让这个物体跳起来。

​	由于系统只会对相关联的组件进行操作，所以组件就定义了一个实体所应该具有的行为。比如说，如果一个实体有一个Position组件，但是没有Velocity组件，那么我们就知道，这个物体是静止不动的，系统就不会对这个实体的Position组件进行操作了。当我们对这个实体增加了一个Velocity组件的时候，系统就会使用Velocity组件来对物体进行移动。这样的行为可以使用被标记的组件来进行，被标记的组件能够重复的使用在不同的上下文中。对一个实体，增加一个空的Player组件，将会为这个实体打上了Player的标签，那么PlayerControl系统，就会寻找带有这个标签的所有组件，然后使用Input中的数据，进行操作。



## 例子

- Movememt(Position, Velocity) - 将速度增加到位置上去
- Gravity (Velocity) - 使用重力来对速度进行加速
- Render(Position, Sprite) - 绘制精灵
- PlayerControl(Input, Player) - 更具Input中的数据控制Player
- BotControl(Input, AI) - 更具AI代理和输入的数据控制



# 结论

​	使用了CES系统之后，我们就可以避免使用大量的类了。实体就是你游戏中存在的物体，它隐式的使用一系列的组件进行定义，这些组件都是纯粹的数据，只有系统才能够操作他们。

​	我希望我已经成功的向您解释了什么是CES系统，并且你有欲望在自己接下来的项目中试试这个系统的效果。如果你有任何关于本片文章的疑问，很感谢你在后面留下你的问题。