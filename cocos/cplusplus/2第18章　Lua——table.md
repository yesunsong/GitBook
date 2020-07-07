# 第**18**章　**Lua**——**table**

table是Lua中最为重要的数据类型，要用好Lua必须熟练掌握好table。table是一种key-value的关联容器，key和value都可以存储任意类型的值或对象，如number、string、function、userdata、table等。

基于table，可以以一种简单、统一、高效的方式来模拟数组、链表、队列、集合、图等复杂的数据结构。Lua中的面向对象、模块和包等概念也是使用table来实现的。在Lua中，table既不是值也不是变量，而是对象。程序运行过程中动态创建的对象，在程序中仅持有对table的引用。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用table。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　元表metatable。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　packages介绍。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　面向对象。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　table库。

### 18.1　使用table

#### 18.1.1　创建table

使用**构造表达式**可以创建一个table，例如：

```
--创建一个table，并将其引用存储到a中
a = {}
```

在构造表达式中，还可以初始化数组或键值对。

```
--下标1、2、3的值为字符a、b、c
--Lua的下标默认是从1开始而不是0
a = { 'a', 'b', 'c'}
--下标'r'、'g'、'b'的值为数字0、255、0
color = {r=0, g=255, b=0}
```

上面的两种初始化方式也可以混合使用，例如：

```
--键值对方式与列表方式并不冲突，列表方式的元素会依次从1开始
--arr['x'] -> 1
--arr[1] -> ‘a’
arr = { x=1, y=2, 'a', 'b', 'c'}
```

那么在混合初始化时，如果键值对与列表元素的key产生冲突，会出现怎样的结果呢？

```
--下标1=1会被列表初始化的'a'覆盖
arr = { [1]=1, y=2, 'a', 'b', 'c'}
```

那么这个初始化是按照先后顺序进行赋值的吗？不对，在混合初始化的时候，不论元素的先后，Lua都是**统一先初始化键值对，再初始化顺序元素**。在混合初始化的时候，判断每个顺序元素的下标时，应该无视所有的键值对元素，然后将顺序元素从1开始依次计算顺序。

如果期望下标从0开始怎么办呢？

```
--实际上[0]只对'a'生效
a = { [0] = 'a', 'b', 'c'}
```

如果希望下标从100开始，那么将代码中的[0]改为[100]只会让a的key为100，对b和c没有任何影响。

#### 18.1.2　访问table

table有两种访问方式，下标操作和点操作，点操作只是Lua提供的一种**语法糖**（让你的代码简短一些，比喻为给你一颗糖吃）。通过这两种操作可以取出table中的元素，代码如下。

```
--先初始化一个table
a = { x=1, y=2, 'a', 'b'}
print(a[1])       -->打印a
print(a[2])       -->打印b
print(a.x)        -->打印1
```

需要特别注意的是，字符串key和数字key是不同的，如a[1] 指的是数字下标，a['1']指的是字符串下标，是两个不同的key。

另外存在一些语法糖失效的情况，如a.1这种写法是通不过编译的。

#### 18.1.3　修改table

在创建了一个table之后，访问table中不存在的字段则会得到nil，通过赋值语句可以为其赋值，增加新的字段或修改已有的字段，例如：

```
a.z = 3
a[3] = 'c'
```

一个table中可以同时存储各种类型的key和value，这是非常灵活的。通过将字段置为nil，可以删除table的某个字段。

#### 18.1.4　删除table

如何删除table呢？可以将要删除的table设置为nil，但其不一定会被删除，因为我们持有的只是table的一个引用，这个table可能在其他的地方还有引用。当一个table再没有其他地方引用到时，Lua的垃圾回收机制会自动将其清理。

#### 18.1.5　遍历table

在Lua中遍历table的常用方法主要有两种，使用ipairs迭代方法顺序遍历数组（下标从1开始），以及使用pairs迭代方法遍历所有键值对。代码如下所示：

```
--注意，这里构建的table下标1、2、3都是有内容的
--但是4是nil，5为d，另外还插入一对key-vluae x = 1
t = {'a', 'b', x = 1, [3] = 'c', [5] = 'd'}
-- 依次打印出a、b、c的3个值
    for k,v in ipairs(t) do
    print( v)
end
--打印了所有内容a b c 1 d
for k,v in pairs(t) do
    print( v)
end
```

此外，还可以使用#t来获取table的大小，得到的大小会是3，而使用table库的方法table.maxn(t)得到的大小则是5。

### 18.2　元表metatable

metatable（元表）是一种特殊的table，通过metatable，可以修改一个值的行为，有点类似于C++中的重载。在Lua中，每个值都有一套预定义的操作集合，如数字可以互相加减，字符串可以进行连接等。这些行为都被定义在metatable中。

可以理解为metatable就是一个值的方法列表。**通过修改一个值的metatable，可以改变其行为**，例如，假设期望实现两个table相加的操作，可以修改这两个table的metatable，在metatable中实现相加的行为。

要实现这个功能，首先需要定义一个metatable，其与普通的table一样，但是需要定义相加方法的实现，代码如下：

```
mt = {}
mt.__add = function(t1, t2)
    local result = {}
    --将两个table合并到result中
    for k, v in pairs(t1) do result[k] = v end
    for k, v in pairs(t2) do result[k] = v end
    --返回结果
    return result
end
```

上面的代码定义了一个名为mt的table，并且将它的__add字段设置为一个函数，在这个函数中实现了将两个table合并为一个新的table并返回的功能。接下来需要将这两个普通的table进行相加操作。

```
local t1 = {x = 1, y = 2, z = 3}
local t2 = {a = 1, b = 2, c = 3}
```

为了使它们支持相加操作，需要将前面定义了相加操作的mt设置为它们的metatable，调用Lua的setmetatable()方法来进行设置。

```
--设置mt为t1和t2的元表
setmetatable(t1, mt)
setmetatable(t2, mt)
```

最后将它们相加，并将相加后的结果打印出来。

```
local t3 = t2 + t1
for k,v in pairs(t3) do
    print(k)
end
```

打印相加返回结果的key，输出了x y z a b c这6个元素。

Lua中每个值都有一个元表，但只有table和userdata这两种对象可以有各自独立的元表，其他类型的值则是全局共享一个元表。**在Lua中，只可以设置table的元表**，其他类型的元表只能在C语言代码中设置。Lua在创建一个新的table时，并不会为其创建元表，需要我们自己设置。

#### 18.2.1　元方法

前面例子中设置__add的方法，称之为元方法。当Lua试图将两个table进行相加操作时，**会先检查两者之一是否有元表**。如果有的话，则判断元表中是否有名为__add的字段，如果有就调用该字段对应的值，也就是元方法。

元方法并没有要求一定要返回什么，例如，在前面的例子中，将__add的元方法实现为将第二个table的内容合并到第一个table中不做返回，也是可以的。但直接用t1 + t2这样的写法编译并不会通过，可以用一个变量来接收相加之后的结果，但按照Lua的语法，这个变量会等于nil，因为函数并没有返回（有疑惑的读者可参考Lua函数的相关内容）。

#### 18.2.2　算术、关系与连接元方法

除了__add之外，Lua还提供了其他具有特定含义的字段，具体如表18-1所示。

表18-1　特定含义的字段

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121220.jpeg)

需要说明的是表18-1**中这些字段对应的元方法，都是会传入两个参数，而__unm除外**，因为其对应的取相反数-操作符是一个一元操作符，如local a = -1。

等于、小于、小于等于这3个关系操作符的元方法涵盖了6种关系判断操作，另外3个没有在表18-1中的操作符是不等于、大于等于、大于，这3个操作符会调用前面3个操作符的元方法，并用not进行取反，例如，不等于会调用等于的元方法进行判断，最后将结果取反并返回。

关系类的元方法在对不同的类型或具有不同元方法的对象进行比较时，Lua会报错，但等于比较不会报错，其会返回false。**只有被比较的双方共享同一个元方法时，才会执行等于比较的元方法**。

#### 18.2.3　特殊的元方法

除了上面的元方法之外，Lua还支持以下特殊的元方法，具体如表18-2所示。

表18-2　特殊的元方法

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121221.jpeg)

表18-2中几个特殊的元方法需要介绍一下，其中，__tostring主要用于将对象转换成字符串时调用，例如，可以将一个table的__tostring元方法设置为打印table中的所有元素，那么在调用print()进行输出的时候，就会打印出table中的所有元素。

**__metatable主要用于保护对象的metatable不被修改**，setmetatable()方法可以设置元表，而getmetatable()方法可以获取元表，如果对象元表的__metatable字段有值的话，调用该对象的setmetatable将会报错。而getmetatable()方法会先判断metatable是否存在__metatable字段，如果是则直接返回metatable的__metatable字段，否则返回metatable（当然，最开始肯定是要判断metatable是否为nil）。具体的操作代码如下。

```
mt = {}
--给__metatable设置一个字符串
mt.__metatable = "don't modify metatable"
--创建一个表并设置metatable
local t1 = {x = 1, y = 2, z = 3}
setmetatable(t1, mt)
--再次修改表的metatable，运行到此处Lua将会报错
--cannot change a protected metatable
setmetatable(t1, {})
--会打印出don't modify metatable
print(getmetatable(t1))
```

__index元方法和__newindex元方法是与表的访问相关的元方法，__index元方法在**访问一个table中不存在的字段**时会被调用，如果没有该元方法，那么Lua就会返回nil。

而__newindex元方法在**对一个table中不存在的字段进行赋值**时会被调用。当要读或者写的字段不存在时，Lua才会去判断元表中的__index()和__newindex()方法，如果没有该元方法，那么Lua就会为该table创建这个字段。例如：

```
t = {}
--访问了t中不存在的字段a，会调用元表的__index元方法
print(t.a)
--对t中不存在的字段a进行赋值，会调用元表的__newindex元方法
t.a = 1
```

#### 18.2.4　__index元方法

下面来看看如何使用__index元方法，首先可以为元表的__index字段设置为一个函数，函数接受两个参数，table和key，table就是我们所访问的table，而key就是该table中不存在的字段名。例如，假设期望访问不存在的字段时返回的默认值是0而不是nil，那么可以这样来使用__index元方法。

```
--创建一个元表
local mt = {}
mt.__index = function (t, k)
    return 0
end
--创建一个空的表，设置元表，并访问一个不存在的字段
local t = {}
setmetatable(t, mt)
print(t.a)
```

除了设置函数之外，还可以为__index设置一个table。例如，有一个怪物table，table中记录了很多属性，如攻击力等，假设需要基于这个怪物表来扩展一个精英怪物表，精英怪物表的很多属性都跟怪物表一样，只是在这个怪物表上扩展了一些字段，那么就可以将怪物表设置为精英怪物表的__index元方法（这个时候叫元方法并不是很恰当，但叫元表肯定是错误的）。

```
--创建怪物表
monster = { attack = 10, def = 5, speed = 1}
--创建一个元表
local mt = {}
--设置monster表到__index中
mt.__index = monster
--创建一个精英怪物表
eliteMonster = { power = 5 }
--设置元表到eliteMonster中
setmetatable(eliteMonster, mt)
print(eliteMonster.attack)
```

使用rawget()函数来访问表中的字段可以绕过对__index元方法的，直接访问表中的字段。

#### 18.2.5　__newindex元方法

__newindex的元方法接受3个参数，table、key以及要设置的value。通过__newindex元方法可以控制table中的字段的只读或只写，禁止添加一些字段，监控字段的数值改变（所有字段的值放到另外一个table中）等。例如：

```
--另外一个表
otherTabel = {}
--创建一个元表
local mt = {}
mt.__newindex = function (t, k, v)
    otherTabel[k] = v
end
--创建一个空的表，设置元表
local t = {}
setmetatable(t, mt)
t.a = 1
print(otherTabel.a)
```

使用rawset()函数来设置表中的字段可以绕过对__newindex元方法，直接设置到表中。__newindex和__index结合起来，只要运用得当，可以实现很多奇妙想法。

#### 18.2.6　__mode元方法

__mode元方法（弱引用表）主要用于标识表中的元素是不是弱引用，如果对Cocos2d-x的引用技术比较熟悉的话，这句话就很容易理解。

当将table A作为value存入另外一个table B中，那么这时候B就引用了A，相当于B对A进行了retain，称之为强引用，当将A置为nil之后，如果没有将A从B中清除，那么由于引用计数不为0，会导致A始终不会被释放。

这样可能带来一些由于管理不当导致的内存泄漏问题，或者是程序员忘记将其置为nil，然后让Lua的垃圾回收机制去释放它。弱引用表帮程序员解决了这个问题，当程序员不希望去做这样的管理时，可以将其设置为弱表。

程序员需要将元表的__mode元方法设置为一个字符串，当字符串包含'k'时，table的key就是弱引用的，而当字符串包含'v'时，table的value就是弱引用。'k'和'v'可以同时存在，key和value同时是弱引用。具体参考下面的例子：

```
--创建一个元表，__mode为v
mt = {}
mt.__mode = "v"
--创建空表t，并设置元表
t = {}
setmetatable(t, mt)
--将一个空的表tv设置为t的value
tv = {}
t[1] = tv;
--此时打印出来t的长度为1
print(#t)
--将tv置为nil，再调用Lua的垃圾回收方法
tv = nil
collectgarbage()
--此时打印出来t的长度为0
print(#t)
```

可以看到，Lua做了一件非常漂亮的事情，当将tv置为nil并强制垃圾回收时，作为弱表t的值，也从t中自动删除了。

这在C++中是很容易出问题的，例如，程序员创建了一个子弹列表，在子弹爆炸之后，就需要将子弹从子弹列表中删除，而这个时候往往上层正在遍历子弹列表，为了保证程序不会崩溃，往往需要进行一些如延迟删除之类的特殊处理，但弱引用表可以很简洁地实现这个功能。另外需要说明一下，如果代码中没有调用collectgarbage()方法，打印出来的长度仍然为1。

### 18.3　packages介绍

table可以用来模拟其他语言的命名空间以及库。由于table本身是第一类值（first class value），所以用table来模拟库相比其他语言的库机制要灵活得多。

在Lua中要使用一个第三方的库是很简单的，这个第三方库可以是一个Lua模块，也可以是一个编译好的动态链接库，程序员只需要调用require来加载这个库，然后就可以使用了，例如下：

```
--加载mod模块、调用mod模块的foo()方法
require "mod"
mod.foo()
--将mod重命名为更简短的m
local m = require "mod"
m.foo()
--直接取mod的foo()方法
require "mod"
local f = mod.foo
f()
```

#### 18.3.1　require()方法

require()方法用于加载一个模块，通过require“模块名”来加载模块，该方法会返回由该模块函数组成的table，并定义一个包含该table的全局变量（这些实际上是模块做的，而不是require()方法做的）。require()方法的实现大致如下：

```
function require(name)
    --如果未加载该模块，则进行加载
    if not package.loaded[name] then
         local loader = findloader(name)
         if loader == nil then
              error("unable to load module " .. name)
         end
         --先标记为已加载，以避免模块互相包含时导致死循环
         package.loaded[name] = true;
         local res = loader(name)
         if res then
              package.loaded[name] = res
         end
    end
    --返回包
    return package.loaded[name]
end
```

require()方法在加载同一个文件的时候，并不会进行重复加载，而是将它缓存起来，当再次require该文件时直接返回。

当require()方法第一次加载某模块时，如果找到该模块的Lua文件，则通过loadfile来加载文件，如果找到的是一个库文件，则通过loadlib来加载文件。**此时只是加载了文件，并没有执行代码**。为了运行代码，require()方法会**以模块名为参数来调用这些代码**。最后，如果代码有返回值，则将返回值存储到package.loaded中并返回。

Lua文件的搜索路径存放在package.path中，而库文件的搜索路径存放在package.cpath中，在require时，Lua会根据模块名在搜索路径中进行搜索。搜索路径中存在多个路径，以分号;分隔开。而路径中的?符号则会被替换成模块名。

```
.\?.lua;C:\Program Files (x86)\LuaStudio\lua\?.lua;C:\Program Files (x86)\
LuaStudio\lua\?\init.lua;C:\Program Files (x86)\LuaStudio\?. lua;C:\
Program Files (x86)\LuaStudio\?\init.lua
```

如果需要添加一个新的搜索路径，可以在path或cpath变量后面追加一个分号，并跟上要添加的路径。例如：

```
package.path = package.path .. ";/src"
```

当需要使用一个位于复杂的相对路径下的一个模块时，应该怎样写可以保证正确地被加载呢？可以使用如下方法来进行测试，mod.lua定义了一个mod表，提供了一个fun()方法，该文件放在一个复杂的路径下，可以使用下面的方法来指定这个模块。

```
--将mod.lua放到当前路径下的mymod/m/目录中，然后进行访问
local m1 = require "mymod/m/mod"
m1.fun()
--推荐使用.来替换/，因为.是平台无关的
local m2 = require "mymod.m.mod"
m2.fun()
```

如果该文件放在其他路径下，也可以通过相对路径来访问到。

```
--如果把整个mymod目录转移到上上层目录中，那么需要用..来指定上一层目录
local m3 = require "../../mymod/m/mod"
m3.fun()
--使用.替换/，如果是../则直接去掉/
local m4 = require "....mymod.m.mod"
m4.fun()
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121222.jpeg)**注意：**如果使用了LuaStudio等工具直接进行调试，可能会报找不到模块的错误，因为直接使用这种工具调试，当前路径并不是当前Lua文件的路径，而是exe启动时的路径。所以需要将模块放到与当前路径（不是当前文件路径）相对的路径下，或者设置package.path，或者重新整理出当前的路径到目标模块文件的相对路径。

#### 18.3.2　编写模块

接下来看一下如何编写模块，首先需要创建一个table，然后定义一些方法供外部调用，下面这种function mod.fun的写法，等同于定义一个function，然后将其设置到table的fun字段中。

```
mod = {}
function mod.fun()
   print("fun")
end
return mod
```

最后需要将模块table返回，如果没有返回值则会返回package.loaded[modename]的当前值。另外，还需要将模块名定义为一个全局变量。

在编写模块的时候，需要注意模块内的方法互调、模块内私有方法、模块名修改等问题。

首先如果在mod的fun1中调用mod的fun2方法，在调用时需要加上mod.作为前缀。当模块名从mod修改为oldmod时，那么所有的函数名以及调用函数的地方都需要修改。那么一个简单的方法是在内部为table定义一个内部的名字，这样外部名字被修改时对模块内部不会有任何影响。

```
local mod = {}
oldmod = mod
function mod.fun()
   print("fun")
end
return oldmod
```

上面的代码将mod修改为了oldmod。这里还有另外一种更通用的方法，就是接收外部传入的模块名，然后直接使用外部的模块名，也就是这个Lua文件的名字。例如：

```
modname = ...
_G[modname] = mod
```

### 18.4　面向对象

在Lua中可以使用table来实现面向对象，那么就要考虑几个问题：如何定义一个类，如何实例化一个类的对象，如何实现private()方法以及变量，如何实现继承与多态。

#### 18.4.1　定义类

首先，通过将某一类别的对象的共同特征抽象出来，也就是它们的属性和方法，然后将这些属性和方法进行封装，就得到了一个类（抽象能力是程序员的关键能力，这是一种分析、总结、提炼本质的能力）。那么在Lua中定义一个类也就是将属性和方法封装（或者说存储）到Table中。例如，下面定义了一个简单的类，称为Man，Man有一个age属性以及一个say()方法。

```
Man = { age = 0}
function Man.say()
 print("man say my age is " .. Man.age)
end
```

在定义Man.say()方法的时候，实际上是定义了一个函数say()，并保存到table中。那么在say()函数中是不可以直接操作Man的成员变量或者成员函数的，也就是说，我们往Table里面放了一个函数，这个函数如果需要访问Table中的其他元素，只能通过这个Table，也就是Man.age而不能是age。

那么上面的方法存在一个问题，如果将Man这个Table赋值给变量a，并将Man置为nil，则此时调用a.say()就访问不到Man.age了。我们知道将Man赋值给a并不会创建一个新的Table，只是a会得到这个Table的一个引用，而Man也只是这个Table的一个名字而已，真正的Table存储在Lua底层，而不是存储在Man这个变量中。所以当将Man置为nil之后，say()函数再去调用Man.age就会报错，因为Man这个变量已经是nil了，但是a.age是可以的。

```
a = Man
Man = nil
--报全局变量Man为nil的错误attempt to index a nil value (global 'Man')
a.say()
```

此外，每个对象都应该是一个独立的Table，有着独立的成员变量，就像每个Sprite都应该有自己的位置属性，而不是共用一个属性。所以为了解决这两个问题，可以通过将每个对象自身的Table也一起传入，代码如下：

```
function Man.say(me)
 print("man say my age is " .. me.age)
end
a = Man
Man = nil
a.say(a)
```

由于这样写比较麻烦，所以Lua提供了一个语法糖，也就是 : 的写法，Man:say会自动在前面添加一个self变量，而调用a:say时，会自动将a传入，简化了代码的编写。

```
function Man:say()
 print("man say my age is " .. self.age)
end
a = Man
Man = nil
a:say(a)
```

#### 18.4.2　实例化

接下来看一下如何实例化一个类，前面的a = Man并没有创建一个新的对象，那么Lua中创建一个类的对象也很简单，深复制一个Table就可以了。将类的Table设置为对象的metatable也可以。例如：

```
--复制所有的方法和属性到对象上
function Man:new()
 local instance = {}
 for k,v in pairs(self) do
  instance[k] = v
 end
 return instance
end
--设置Man为对象的metatable
function Man:new()
 local instance = {}
 setmetatable(instance, self)
 self.__index = self
 return instance
end
```

使用这两种方法创建的对象都是一个独立的对象，有自己的成员变量，但metatable显然会更好一些，因为没有占用多余的内存。两种方式在执行下面的代码时，都会输出正确的结果。深复制的table的逻辑很清晰，而设置metatable方法创建出来的对象，在设置age的时候，会直接在对象的table中插入age字段。如果没有对age进行赋值，那么在访问age时会触发metatable的__index字段，从Man中取出age变量。对say()方法的调用也是一样的，会直接取metatable中的say()方法（回顾一下metatable的__index和__newindex）。

```
lilei = Man:new()
hanmeimei = Man:new()
--注意这里用的是 . 号
lilei.age = 15
hanmeimei.age = 14
-- 注意这里用的是 : 号
lilei:say()
hanmeimei:say()
```

#### 18.4.3　继承

关于继承，我们定义一个Boy类，Boy继承于Man，Boy首先是一个Man对象，所以其metatable是Man，而Boy对象的metatable又是Boy，所以嵌套的metatable就实现了类的继承，当访问一个属性或方法时，首先会在Boy对象的table里面找，如找不到则找它的metatable，也就是Boy这个类，如果再找不到，就通过Boy类的metatable继续找，也就是Man。所以这里可以在Boy中选择性地重写方法，代码如下：

```
Boy = Man:new()
function Boy:say()
 print("boy say my age is " .. self.age)
end
function Boy:new()
 local instance = {}
 setmetatable(instance, self)
 self.__index = self
 return instance
end
jim = Boy:new()
jim:say()
```

最后是关于private和public的话题，建议的做法是通过规则来标记，例如_say，前面加了下画线的表示私有，虽然实际上私有方法也可以被外部调用到，但如果程序员不希望对象的某些方法或属性被访问，那么就不要去访问就可以了。

如果真的要实现private方法，在Lua中，可以通过两个表来实现这种功能。一个表保存内部状态，另外一个表提供操作接口，只将提供了外部接口的表提供给用户使用即可。关于面向对象部分的内容，在介绍Quick以及Cocos原生的Lua框架时，还会有更多的介绍。

### 18.5　table库

Lua的table库常用的主要有insert()、remove()、sort()、concat()这几个方法，其他诸如getn()、setn()、maxn()等长度相关的方法在Lua升级到5.2之后被移除了（Cocos2d-x目前使用的版本是Lua 5.1），并新增了pack()/unpack()、move()等方法。

#### 18.5.1　插入

insert()可以插入一个元素到table中的指定下标，并将后面的元素往后移动。函数原型有以下两种：

```
--在数组的最后插入value
insert(table, value)
--在数组的index位置插入value，并将index后的元素往后移动
insert(table, index, value)
删除
```

remove()方法可以将table中指定index的元素删除，并将后面的元素往前移动，函数原型如下：

```
remove(table, index)
```

#### 18.5.2　排序

sort()方法可以将table中的value按照指定的方式进行排序（由小到大），第一种用法是直接传入要排序的table，Lua默认会通过调用其小于操作符来进行比较并排序。

```
--一个随意顺序的table
a = {5, 1, 2, 4, 3}
--进行默认的排序，从小到大进行排序
table.sort(a)
--依次打印出1 2 3 4 5，这里的_在Cocos中是一种常用写法，并不是特殊的语法，而是指我
不关心这个值
for _,v in pairs(a) do print(v) end
```

接下来介绍sort()方法的高级用法，通过传入一个比较函数，可以按照自己的期望进行排序。比较函数接收两个元素，当第一个元素应该在前时，返回true，否则返回false。这里使用一个匿名函数，将这个数组进行逆序排序，最大的值在前面，代码如下：

```
table.sort(a, function (v1, v2) return v1 > v2 end )
```

concat()方法并不是将两个table合并为一个，而是将table中的value依次连接起来，作为一个字符串并返回。该方法主要是在需要将大量字符串进行拼凑时，提供更灵活的写法以及更高的效率。table的concat()方法要比字符串的连接操作符..高效很多。..操作符每次都会产生一个新的字符串，而concat()方法只会产生一个最终的结果字符串。concat()方法有4个参数，分别是table、分隔符（也就是要添加到每两个值中间的字符串）、从哪个下标开始，以及哪个下标结束。除了第一个参数，其他参数都是可选的。

#### 18.5.3　pack()和unpack()方法

pack()和unpack()方法是Lua 5.2新增的两个方法，pack()方法通过参数列表构建一个table数组，并设置一个n字段来记录数组的长度，最后返回该数组。而unpack()方法会依次返回一个table数组的所有值。注意，这里说的table数组，也就是从下标1开始到第一个nil的内容，如下所示。

```
--从下标1开始为a b c d，n = 4，共5个元素
a = table.pack( 'a', 'b', 'c', 'd' )
--依次打印a b c d，在同一行中输出
print(table.unpack(a))
```

#### 18.5.4　table长度

接下来是3个关于table长度的方法，getn()和maxn()方法的函数原型都是传入一个table，返回一个int，而setn()方法要求传入一个table以及要设置的长度。

setn()方法可以设置数组的长度，但在Lua 5.1版本中使用会报'setn' is obsolete的错误。

getn()方法可以返回数组的长度，这里的数组指的是从下标1开始，到第一个nil中间的这段内容，结果等同于#操作符。

maxn()方法可以返回table的完整大小，也就是所有的key-value键值对的总数，但在Lua 5.2以上的版本中恐怕需要手动遍历。

table的长度涉及一个有趣的问题，在一般的情况下，如要获取一个table的长度，有两种方法，一种是遍历，另外一种就是用一个变量来记录table的大小，在每次添加或删除的时候，修改这个变量。显然每次取table长度都进行遍历的话，如果操作频繁，对于比较大的table而言，效率上是非常糟糕的，因此就需要用一个变量来记录。那么将这个变量放在哪里是一个问题，如果放在table本身，那么就污染了这个table，而将这个变量放在另外一个table中，将table设置为弱引用table，那么所使用的table就会比较干净了。

Lua协调了这两者，setn()方法和getn()方法操作会先判断table中是否有n这个字段，如果没有，才去查找另外一个隐藏的table，笔者估计是以table的地址为key，数字n为value的一个table。当这两个操作都失败时，getn会进行一次从1开始直到碰到nil值的遍历，并返回遍历的结果。

那么给n字段赋予长度的意义有什么好处呢？我们知道Lua的table是会动态增长的，如果我们一开始就知道，这个table需要存放1000个元素，通过n来告诉Lua，直接分配长度为1000的数组，在一定程度上可以提高效率。

然而上面说的这些，目前并没有什么用处，只是一些有趣的历史问题。在Lua的升级之后，摒弃了这看似有用但实际用途有限的功能，也摒弃了可能带来一丝性能提升的优化，让Lua变得更加简洁。