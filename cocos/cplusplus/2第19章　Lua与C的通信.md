# 第**19**章　**Lua**与**C**的通信

Lua是一门脚本语言，一般脚本语言离不开宿主，在使用Cocos2d-x开发的游戏中，Lua的宿主就是Cocos2d-x程序，Cocos2d-x是以C/C++语言为主的，而C/C++语言是Lua的好基友，Lua提供了非常便捷的方式让程序员在C/C++与Lua之间进行通信，这正是本章的主题。

虽然有更便捷的方法，如tolua++之类的第三方库可以自动地生成一些内容，可以让Lua方便地访问C++代码。但这里并不打算介绍它们（使用它们可能碰到更多的问题），这里只介绍最简单的原生方法，这样可以让读者对Lua和C++的交互能够有更清晰的理解。

当程序员需要根据C++代码自动生成一些可以让Lua执行到的中间代码时，一般是在Lua端需要大量访问C++代码的情况下，例如Lua需要可以访问Cocos2d-x中的大量API。而对于应用层，是不应该发生这种事情的，要么以C++代码为主，在少量的地方使用Lua代码；要么以Lua代码为主，C++部分的代码只应该负责很少的一部分功能。或者有若干个相互间较为独立的模块，Lua代码负责其中的一些模块，而C++代码负责另外的一些模块。以上这几种情况，Lua与C++的交集都不会太多。

如果有一个模块需要Lua代码和C++代码非常密切地协同完成，以至于需要使用第三方库批量地生成C++转Lua的代码来提高开发效率，那么就需要好好考虑一下，当前的设计是否合理了。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　准备工作。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　操作table。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　C/C++中调用Lua。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　注册C/C++函数给Lua调用。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　将C/C++的类传给Lua。

### 19.1　准备工作

Lua与C语言的通信主要包含了C/C++如何访问Lua，以及Lua如何访问C/C++两个方面，而它们都是在C/C++代码中实现的，Lua层并不需要特地去做什么。

通过调用Lua提供的API，可以在C/C++代码中直接执行Lua语句和Lua脚本文件、调用Lua的函数、获取Lua的全局变量等。

而对于Lua访问C/C++代码的实现，是通过Lua的API将C/C++代码的函数按照Lua指定的格式进行定义以及注册。将C/C++代码放在宿主程序中（Cocos2d-x程序就是），那么只需要在Lua脚本调用之前进行注册即可。此外，可以将C/C++的代码编译成一个动态链接库，然后在Lua中加载并使用。

#### 19.1.1　头文件与链接库

要使用Lua的API，Lua是一个第三方的库，所以包含头文件以及指定链接库则是必不可少的工作，但是这里需要特别注意一个问题，**如果使用的是C++语言，那么在包含lua.h等文件时需要用extern"C" { ... }将#include语句包裹在花括号内**，告知编译器这些文件是按照C语言的规则来编译链接而不是C++语言，如果不这么做，则会导致编译出错。大家可以自行搜索C++ extern C关键字，网上有很多文章对这个知识点进行了系统的总结。

```
extern "C" {
#include "lua.h"
#include "lualib.h"
#include "lauxlib.h"
}
```

lua.h、lualib.h以及lauxlib.h是需要包含的3个头文件，lua.h定义了Lua提供的基础函数，提供最基本的功能，在lua.h中的函数都以lua_为前缀（如lua_open、lua_pcall等）。

lualib.h定义了Lua标准库的加载函数，一个新的环境默认是没有载入标准库的，如string、table、io、math等库，除了用于加载指定库的luaopen_开头的函数外，lualib.h还提供了一个luaL_openlibs()方法用于一次性加载所有的标准库。

lauxlib.h定义了辅助库提供的函数，它们都以luaL_为前缀，这些方法是基于lua.h中提供的方法来操作Lua的，lauxlib.h提供了更加友好、方便的函数让程序员操作Lua。

#### 19.1.2　lua_State指针

Lua脚本是通过Lua虚拟机来执行的，在C/C++代码中需要通过一个lua_State指针来操作Lua，绝大部分的API都要求我们传入lua_State指针。那么如何获得lua_State指针呢？

如果编写一个动态链接库来给Lua调用，那么只需要让外部将lua_State指针传入即可，因为在这里lua_State指针并不由程序员所创建，在后面的动态链接库节（19.4.1）中会详细介绍。

如果是在宿主程序中，那么就需要创建Lua的环境。当调用lua_open()函数时，会创建一个新的Lua环境，并返回一个lua_State指针。在创建了一个新的环境之后，需要为它打开Lua的标准库（也可以不这么做），然后就可以使用这个新建的环境来完成各种工作。最后，当程序退出时，需要调用lua_close()函数将这个Lua环境关闭。下面的代码演示了这个流程。

```
int main()
{
    lua_State *L = lua_open();
    luaL_openlibs(L);
    //执行Lua脚本，在C/C++和Lua之间进行交互
    lua_close(L);
}
```

在Cocos2d-x中，只需要调用LuaEngine::getInstance()函数即可完成Lua环境的创建以及相关的初始化，这些内容在第20章中会详细介绍。当需要获取lua_State指针时，可以通过下面的代码来获取。

```
cocos2d::LuaEngine::getInstance()->getLuaStack()->getLuaState()
```

#### 19.1.3　堆栈

在介绍C/C++和Lua之间交互的各种技巧之前，需要先介绍一下堆栈，因为堆栈是它们通信的基础。Lua使用一个堆栈在Lua与其他语言之间传递数据，这个设计是非常简洁灵活的。

在Lua与其他语言交换数据的时候，主要的问题有两个，一个是数据类型的问题，还有一个是数据的内存管理问题。

例如，当要设置一个Table时，Table的Key和Value可能是各种各样的数据类型，如果按照传统的方式来提供API，为了满足所有的Key和Value的组合，需要提供8×8，也就是64个API才能满足需求。而使用栈来传递数据，则将数据类型与要实现的具体功能分离开了。

例如，将一个Lua的值保存在C/C++代码的变量中，Lua并不知道这个引用，所以当这个值在Lua中没有其他地方引用到时，可能被Lua认为是垃圾而清理掉，这时候在C/C++代码中访问这个值就会发生错误。而通过堆栈来传递数据，这个堆栈中的值是可以被Lua管理的，C/C++代码通过Lua的API访问到这个值。

**当调用Lua的API来调用一个Lua方法时，或者访问Lua的全局变量时，需要将传入的参数压入堆栈中，而调用之后返回的结果也会被放在堆栈中。**

这里的栈是一个后进先出的数据结构，对于Lua而言，其只会改变栈顶的部分，而对于C/C++，则可以对栈中的任意元素进行查询和删除，甚至可以在任意位置插入一个元素。

如果对栈的概念不熟悉，建议读者学习一下栈这个数据结构，这非常重要。一个叫做汉诺塔的游戏有利于对栈的理解，**如果不理解栈这个数据结构，对于本章的阅读会非常困难。**

#### 19.1.4　压入堆栈

以下API支持我们将不同类型的数据压入堆栈。

```
//压入一个nil值
void lua_pushnil (lua_State *L);
//压入一个布尔值
void lua_pushboolean (lua_State *L, int bool);
//压入一个数值
void lua_pushnumber (lua_State *L, double n);
//压入一个字符串，字符串的内容可以包含/0
void lua_pushlstring (lua_State *L, const char *s, size_t length);
//压入一个以/0结尾的字符串
void lua_pushstring (lua_State *L, const char *s);
//压入一个function，C的闭包——以int fun(lua_State*)为原型的函数，以及该函数的
up值数量
//先压入up值，再压入function。up值指闭包概念中，绑定到该函数的外部变量
//lua_pushcclosure在压入function之前，会先将up值弹出
void  (lua_pushcclosure) (lua_State *L, lua_CFunction fn, int n);
```

此外还可以压入table、userdata等类型，这些在后面介绍。

#### 19.1.5　访问堆栈

当调用完Lua的函数之后，可能产生多个返回值，这些返回值被存放在栈中，这时候我们就需要将栈中的元素取出来。

Lua的API需要传入一个索引来操作栈中对应的元素，1表示第一个入栈的元素，2表示第二个，依此类推。此外Lua还提供了负索引来方便我们进行操作，正索引从栈底开始，而负索引从栈顶开始，-1表示栈顶的元素，-2表示栈顶前面的一个元素，依此类推。

堆栈中的元素是Lua的值，这个值有可能是布尔值、数值、table，甚至函数。那么如何确定元素的类型呢？有两种方法，一种是使用lua_isXXX系列API来判断指定的值是否为指定的类型（XXX为类型名，如string、table、function等）。由于字符串和数值类型在Lua中可以相互转换，所以lua_isnumber和lua_isstring函数会判断该值是否能够被转换为number或string类型。lua_isXXX的函数原型如下。

```
//传入lua_State*和索引，判断成功返回1，失败返回0
int lua_isXXX(lua_State *L, int index)
```

此外还可以使用lua_type()函数来获取该值的类型，每种类型都对应一个常量，包含了LUA_TNIL、LUA_TBOOLEAN、LUA_TNUMBER、LUA_TSTRING、LUA_TTABLE、LUA_TFUNCTION、LUA_TUSERDATA以及LUA_TTHREAD。

```
//传入lua_State*和索引，返回元素的类型，索引无效则返回LUA_TNONE
int lua_type(lua_State *L, int index)
```

在知道了类型之后，就可以使用lua_toXXX系列API来获取指定的元素的值。大多数情况下并不根据类型判断来得知数据的类型，而是直接假定栈中的元素是某类型，一般在调用了一个Lua函数之后，会很清楚这个Lua函数会返回什么，但进行类型判断可以使代码更加安全（有时候，应该让问题暴露出来）。

```
//把指定的索引处的的Lua值转换为一个C语言中的boolean值，对于false和nil以及无
效的索引会返回0，其他情况返回1
int lua_toboolean (lua_State *L, int index);
//把指定索引处的Lua值转换为一个C语言函数
lua_CFunction lua_tocfunction (lua_State *L, int index);
//把指定索引处的Lua值转换为lua_Integer，一个有符号整数类型
lua_Integer lua_tointeger (lua_State *L, int idx);
//把给定索引处的Lua值转换为一个C字符串。如果len不为NULL，字符串长度会被输出到
*len中。如果值是一个数字，lua_tolstring会将堆栈中的值转换为一个字符串
const char *lua_tolstring (lua_State *L, int index, size_t *len);
//等价于lua_tolstring，而参数len设为NULL
const char *lua_tostring (lua_State *L, int index);
//把指定索引处的Lua值转换为lua_Number
lua_Number lua_tonumber (lua_State *L, int index);
//把指定索引处的值转换为一个Lua线程
lua_State *lua_tothread (lua_State *L, int index);
//如果指定索引处的值是userdata类型，返回它们的指针，否则返回NULL
void *lua_touserdata (lua_State *L, int index);
```

#### 19.1.6　堆栈的其他操作

除了入栈和访问栈中的元素，Lua还提供了其他的API以供操作堆栈，包含弹出、插入、删除、替换等操作，如下所示。

```
//查询栈中元素的数量
int  lua_gettop (lua_State *L);
//设置栈顶位置，如比当前栈中元素数量多则插入nil，如少则将多余的元素移除
void lua_settop (lua_State *L, int index);
//压入一个在栈中已经存在的值到栈顶，相当于copy一个元素到栈顶，index为要copy的元素
索引
void lua_pushvalue (lua_State *L, int index);
//移除指定索引的元素
void lua_remove (lua_State *L, int index);
//将当前栈顶的元素插入到指定的索引中
void lua_insert (lua_State *L, int index);
//将当前栈顶的元素与指定索引处的元素进行替换
void lua_replace (lua_State *L, int index);
//弹出栈顶的n个元素
#define lua_pop(L,n)  lua_settop(L, -(n)-1)
```

### 19.2　操作table

table是比较特殊的数据类型，不像布尔值和数值那些可以直接压入栈中，因为table的键值对可以存储任意类型的值，所以table的读和写需要进行特殊的处理。

#### 19.2.1　如何将table传入Lua

当需要将一个table传给Lua时，需要将table压入栈中，但table内部的结构比较复杂，如何将一个复杂的table传给Lua呢？首先需要使用lua_newtable在栈顶创建一个空的table，然后将table的Key和Value压入（可以嵌套table），最后将Value设置为table的Key字段的值即可。

如何将Key和Value压入，前面已经介绍过了，如果是字符串类型可以使用lua_pushstring方法入栈，数值类型y可以使用lua_pushnumber，关键是最后需要调用设置方法将栈中的Key和Value绑定到table中，Lua提供了以下API。

```
//idx表示table的索引，lua_settable()函数将栈中索引为-2的元素作为Key，-1的元素
作为Value，设置到指定的table中，最后将栈顶的2个元素弹出
void  lua_settable (lua_State *L, int idx);
//lua_rawset()函数功能同lua_settable()
void  lua_rawset (lua_State *L, int idx);
//idx表示Table的索引，lua_setfield()函数将参数字符串k作为Key，栈顶的元素作为
Vluae，设置到指定的table中，最后将栈顶的元素弹出
void  lua_setfield (lua_State *L, int idx, const char *k);
//lua_rawseti()函数功能同lua_setfield()，不同的是参数为整数n
void  lua_rawseti (lua_State *L, int idx, int n);
```

前面的API都要求在调用前，将要设置的Key或Value的值压入栈顶，在调用之后都会将Key和Value弹出。另外lua_settable()函数和lua_rawset()函数的区别在于，lua_rawset()函数并不会触发metatable，是直接对table进行修改。下面代码演示了如何将一个vector<int>的内容作为table传递给Lua。

```
void pushVecIntToArray(const std::vector<int>& v, lua_State* luaState)
{
    lua_newtable(luaState);
    for (unsigned int i = 0; i < v.size(); ++i)
    {
         lua_pushinteger(luaState, v[i]);
        //索引-2的位置为目标table，而i + 1是因为Lua的数组下标从1开始
         lua_rawseti(luaState, -2, i + 1);
    }
}
```

#### 19.2.2　如何获取Lua返回的table

当从Lua中得到一个table时，会被放在栈中，Lua提供了与设置table相对应的API来获取table中的内容。从table中获取的值也会被放到栈中。

```
//idx表示table的索引，lua_gettable()函数获取栈顶元素作为Key，获取到table中该
Key对应的Value，将栈顶的Key弹出，最后将获得的Value压入栈顶
void  lua_gettable (lua_State *L, int idx);
//lua_rawget()函数功能同lua_gettable()函数，区别在于该函数不会触发metatable
void  lua_rawget (lua_State *L, int idx);
//idx表示table的索引，lua_getfield()函数从table中获取指定字符串Key的Value，
并压入栈中
void  lua_getfield (lua_State *L, int idx, const char *k);
//idx表示Table的索引，lua_rawgeti()函数从Table中获取指定数值Key的Value，并压
入栈中
void  lua_rawgeti (lua_State *L, int idx, int n);
```

通过上面的API可以获取table中任意字段的Vlaue，但是在很多情况下需要遍历整个table，在Lua中可以用泛型for语句结合pairs和ipairs迭代函数来遍历，在C/C++代码中又该如何遍历呢？ipairs的实现比较简单，从1开始遍历，直到碰到一个值为nil就结束。这里假设table中的值都是字符串，然后遍历这个table，并将Key和Value打印出来，代码如下。

```
//传入的index为table的下标
void iterAndPrintArray(lua_State* L, int index)
{
    int i = 1;
    while (true)
    {
        //获取下标为i的Value，并放入栈中
        lua_rawgeti(L, index, i);
        if (lua_isnil(L, -1))
        {
            break;
        }
        //打印后pop结果
      printf("%d is %s", i++, lua_tostring(L, -1));
      lua_pop(L, 1);
   }
}
```

而pairs迭代就要复杂一些，需要使用lua_next()方法来迭代table中的所有元素，lua_next()方法需要传入lua_State指针，以及table的索引，它会将Key弹出，然后将该Key的下一个Key和Value依次压入栈中，当开始遍历时，需要先压入一个nil，以表示从第一个Key开始，每次调用lua_next()方法，都会根据栈顶的Key以及指定的table来获取下一个Key，如成功则返回0。这里遍历一个Key和Value都是字符串的table，并将它打印出来，代码如下。

```
//传入的index为table的下标
void iterAndPrintTable(lua_State* L, int index)
{
    lua_pushnil(L);
    //注意，index如果是负数，会因为前面的pushnil而发生变化，所以这里要传入正数索
    引，或考虑前面的pushnil
    while (lua_next(L, index) != 0)
    {
        //lua_next之后，nil被弹出，同时压入下一个key和value
        printf("key is %s, value is %s",lua_tostring(L,-2),lua_tostring(L, -1));
        //将栈顶的Value弹出，保留Key给下一次的lua_next()方法调用
        lua_pop(L, 1);
    }
}
```

### 19.3　C/C++中调用Lua

接下来看一下在C/C++代码中如何调用Lua，通过Lua的API可以执行一段Lua代码，一个Lua脚本文件或者一个Lua函数，前面我们已经知道了如何对栈进行操作，在这里就需要结合API来完成一些具体的功能。

执行Lua代码或Lua脚本文件，都需要先经过编译，编译完之后会将一个可以执行到这些代码的函数压入栈中，最后调用Lua的API来执行这个函数。

#### 19.3.1　执行Lua片段

首先来看如何在C/C++代码中执行一段Lua代码，通过luaL_loadstring()方法可以在C/C++代码中**加载一段Lua代码**，luaL_loadstring()方法接受一个lua_State指针以及一个字符串，该方法会将字符串进行编译，并将编译好的代码作为一个函数类型的值放到栈中，luaL_loadstring()方法会调用luaL_loadbuffer，而luaL_loadbuffer最终又是通过lua_load来编译代码的。

接下来需要调用lua_pcall()方法来执行这段代码，除了lua_pcall()方法之外，还有lua_call()方法可以用于执行Lua的代码。

```
void lua_call (lua_State *L, int nargs, int nresults);
```

lua_call()方法会调用栈中的一个函数，首先需要将函数入栈（luaL_loadstring()方法做了这个工作），接下来将参数挨个入栈，传入的参数数量需要作为nargs参数传给lua_call()方法，同时将期望函数返回值的数量作为nresults参数传入（如果nresults的值为LUA_MULTRET，则所有的返回值都会入栈）。

在函数执行完毕之后，函数以及函数的参数会从栈中弹出，并将返回值挨个入栈（按照返回顺序的先后），如果nresults的值不为LUA_MULTRET，当函数的返回值数量大于nresults时，多于的返回值会被丢弃，小于nresults时，则会用nil来补齐。当lua_call()方法发生错误时，会抛给上层处理，上层如果没有处理，那么程序就会崩溃。

```
int lua_pcall (lua_State *L, int nargs, int nresults, int errfunc);
```

lua_pcall()方法会在保护模式下来执行代码，其和lua_call()方法类似，但其可以**指定一个函数作为错误处理函数，errfunc为栈中的一个函数的索引**，由于函数调用会对栈进行操作，所以使用负索引将难以准确定位到该错误处理函数。函数执行成功时返回0，失败则返回错误码（在lua.h中定义）。

如果使用lua_pcall()方法执行的Lua代码中又调用了C语言代码中的方法，在这个C语言代码中的方法中又使用lua_call()方法执行了一段错误的Lua代码，那么lua_call()方法会将错误抛给上层处理，而这个错误最终会被lua_pcall()方法处理。但是如果直接调用lua_call()方法，那么就没有上层会处理发生的错误了，那么程序只有崩溃。完整的代码如下（头文件的包含代码，就不在这里给了）。

```
int main()
{
    //创建环境并打开标准库
    lua_State* L = lua_open();
    luaL_openlibs(L);
    int error = luaL_loadstring(L, "print('hello world')")
        || lua_pcall(L, 0, 0, 0);
    if (error)
    {
        //如果发生错误，错误信息会留在栈中
        printf("%s", lua_tostring(L, -1));
        lua_pop(L, 1);
    }
    lua_close(L);
}
```

#### 19.3.2　执行Lua脚本文件

执行一个脚本文件需要调用luaL_loadfile()方法来加载并编译，最后通过lua_pcall()或lua_call()方法来执行文件。luaL_loadfile()方法可以打开Lua脚本文件，也可以打开编译好的Lua文件。以下是调用脚本文件的示例代码，与执行Lua代码片段类似。

```
int main()
{
    //创建环境并打开标准库
    lua_State* L = lua_open();
    luaL_openlibs(L);
    int error = luaL_loadfile(L, "test.lua")
        || lua_pcall(L, 0, 0, 0);
    if (error)
   {
       //如果发生错误，错误信息会留在栈中
       printf("%s", lua_tostring(L, -1));
       lua_pop(L, 1);
   }
   lua_close(L);
}
```

Lua与C/C++语言有一个很大的区别，就是如一个C/C++项目中有很多源码文件，那么只要包含了头文件就可以使用相应的方法和类。而**Lua项目中所有的Lua文件，都必须被编译并执行后，才能使用里面的函数和变量**。如果需要使用另外一个Lua脚本中定义的函数，那么需要先确保这个脚本被执行，可以在要使用的Lua脚本中调用require()函数，或在一个入口脚本处require，也可以在C/C++代码中执行这个脚本。

当在Lua脚本中定义了一个全局函数，实际上是以函数名为变量名，将函数赋值给这个函数名变量，Lua使用一个名为_G的table来存储所有的全局变量。如果只是Load一个代码块或者脚本文件而没有执行，则并不会产生这么一个变量，自然也就不能使用这个函数。

所以，一个脚本至少需要被执行一次，这个脚本的内容才会生效。而不是将Lua脚本放到项目中，就可以使用这个脚本定义的方法。

#### 19.3.3　调用Lua函数

当需要调用一个Lua函数的时候，首先需要获得这个Lua函数，一般Lua函数会被存储在全局变量或者某个table中，所以第一步需要将函数获取到栈中，接下来压入函数的参数，并调用函数，最后获取函数的返回值，并清理函数的返回值。如果不去清理函数的返回值的话，其会一直存放在栈中。

前面的例子因为都是直接调用无参数、无返回值的函数，所以这里简单演示一下一个带参数和返回值的函数调用，基于上一个例子，这里在test.lua中定义了一个名为add()的函数，该函数需要传入两个数值，并返回相加后的数值，代码如下（添加于lua_close之前）。

```
//获取全局变量add并压入栈中，add()是test.lua中定义的函数
lua_getglobal(L, "add");
//压入参数
lua_pushinteger(L, 2);
lua_pushinteger(L, 3);
//lua_State，参数有2个，返回值有1个，不设置错误处理回调
lua_pcall(L, 2, 1, 0);
//取出栈顶的返回值并打印
printf("%d", lua_tointeger(L, -1));
//弹出返回值
lua_pop(L, 1);
```

### 19.4　注册C/C++函数给Lua调用

前面了解了C/C++层如何调用Lua，接下来介绍Lua如何调用C/C++层，主要有两种方式，第一种是在宿主程序中注册函数，然后在Lua中直接使用注册的函数。另外一种是直接用C/C++代码编写一个动态链接库，Lua通过require()函数将动态链接库导入后调用库中提供的函数。

任何在Lua中注册的函数必须有同样的原型，这个原型就是lua.h中的lua_CFunction，定义如下。

```
typedef int (*lua_CFunction) (lua_State *L);
```

在函数被Lua调用时，函数的参数会被依次传入栈中，使用lua_toxxx方法可以取出这些参数。在函数执行完之后，需要将返回值依次push到栈中，最后将返回值的数量返回，如果没有返回值，则返回0。下面的代码演示了提供给Lua使用的add()方法是如何定义的。

```
int add_for_lua ( lua_State* L)
{
    double a = lua_tonumber(L, 1);
    double b = lua_tonumber(L, 2);
    lua_pushnumber(L, a + b);
    return 1;
}
```

接下来需要告诉Lua有这么一个函数，这里使用lua_register()函数来告诉Lua，下面的代码注册了一个名为add()的函数到Lua中，当该函数被调用时，会执行add_for_lua()方法。在调用Lua脚本之前执行下面的代码，在Lua中就可以直接使用add()函数。

```
lua_register(L, "add", add_for_lua);
```

上面的代码等同于push一个C函数，然后将其设置为全局变量add的值。

```
lua_pushcfunction(L, add_for_lua);
lua_setglobal(L, "add");
```

动态链接库不同于宿主程序，不需要创建和维护lua_State，也没有main()函数，只是提供了一系列供外部调用的方法。首先需要使用lua_CFunction的原型来定义函数，接下来需要将这些函数导出给Lua。

在Lua中使用require()方法或loadlib()方法可以加载一个库，在Lua中，一个库可以理解为一个装载了N个函数的table，而加载函数被调用后会返回这个table。

当Lua企图加载一个动态链接库时，虚拟机会去调用该动态链接库的luaopen_libname()方法，这里的libname指的是库的名字，该函数的原型同lua_CFunction一致。

在luaopen_libname()方法中，需要调用luaL_openlib(L, "libname", 0)并返回1即可，如果定义的库名为mylib的话，代码如下。

```
int luaopen_mylib (lua_State *L)
{
    luaL_openlib(L, "mylib", mylib, 0);
    return 1;
}
```

luaLopenlib中传入了4个参数，lua_State*、库名字符串、库函数列表以及upvalue数量。最后一个参数一般填0就可以了，库函数列表是一个结构体数组，可以这样定义：

```
static const struct luaL_reg mylib [] =
{
    //Lua中的函数名对应C函数
    {"add", add_for_lua},
    {"sub", sub_for_lua},
    //最后要以NULL结尾
    {NULL, NULL}
};
```

函数、函数信息列表以及luaopen注册函数，最后将它们编译成动态链接库，在Lua中require进来或使用loadlib加载即可使用。

### 19.5　将C++的类传给Lua

C++的类和Lua中的类是不同的，那么如何将一个C++的类传递给Lua使用呢？在Lua中，类是一组变量和方法的集合，通过metatable来实现类，创建这个metatable相当于定义了一个类，而将这个metatable绑定到一个具体的对象（可以是table也可以是userdata）上，相当于实例化了一个类。

具体实现的步骤如下。

（1）首先定义一个C++的类，这一步是可以省略的，因为在Lua中可以作为一个类使用就可以了，而在C++中是不是也是一个类，并不重要，但此处的目的是为了让Lua能够使用C++的类，所以这里需要定义一个类。

```
class A
{
public:
    int getVar() { return var; }
    void setVar(int v) { var = v; }
private:
    int var;
};
```

（2）定义完类之后，需要将该类的方法用Lua可调用的方式进行简单的封装，例如将getXXX这样的方法用注册给Lua调用的函数原型进行包装，下面的代码会详细介绍如何封装。因为在Lua中调用成员方法是通过 : 来访问的，那么传入的第一个参数就会是这个对象本身。下面封装了4个函数：

```
//new一个A对象
int newA(lua_State* L)
{
    A* a = new A();
    pushClass(L, a, "A");
    return 1;
}
//delete一个A对象
int deleteA(lua_State* L)
{
    A* a = reinterpret_cast<A*>(toClass(L, -1, "A"));
    delete a;
    return 0;
}
//调用A对象的getVar()方法，并返回var到Lua
int getVar(lua_State* L)
{
    A* a = reinterpret_cast<A*>(toClass(L, -1, "A"));
    lua_pushinteger(L, a->getVar());
    return 1;
}
//调用A对象的setVar()方法，传入要设置的值
int setVar(lua_State* L)
{
    A* a = reinterpret_cast<A*>(toClass(L, -2, "A"));
    int var = lua_tointeger(L, -1);
    a->setVar(var);
    return 0;
}
```

（3）封装了一系列方法之后，就需要把这些方法整合到一个metatable中，也就是相当于这个类的定义。

```
//A的Lua成员方法列表
static const struct luaL_reg AFunc[] =
{
    { "getVar", getVar},
    { "setVar", setVar},
    { NULL, NULL }
};
//注册A这个类（metatable）
void regiestA(lua_State* L)
{
    //创建一个名为A的metatable
    luaL_newmetatable(L, "A");
    lua_pushstring(luaState, "__index");
    //pushes the metatable
    lua_pushvalue(luaState, -2);
    //设置metatable.__index = metatable
lua_settable(luaState, -3);
//将A的成员方法列表绑定到metatable中
//等同于设置metatable的getVar和setVar字段为对应的C函数
    luaL_openlib(luaState, NULL, AFunc, 0);
    //另外注册两个函数，供Lua创建和释放A的对象
    lua_register(luaState, "newA", newA);
    lua_register(luaState, "deleteA", deleteA);
}
```

（4）pushClass()方法将一个类压入栈中，而toClass()方法则是将栈中的一个元素转换成类，这两个方法的实现如下。

```
void pushClass(lua_State* l, void* p, const char* className)
{
//申请一块4个字节的内存，用这块内存来存放p的地址，所以需要用二级指针
//如果使用一级指针void* up，那么up = p这个操作只是改变了up指向的地址，而没有改变
udata的内容
//我们需要让udata记录下我们的p，而不是局部变量up
    void** up = reinterpret_cast<void**>(lua_newuserdata(l, sizeof(p)));
    *up = p;
    luaL_getmetatable(l, className);
    lua_setmetatable(l, -2);
}
void* toClass(lua_State* l, int index, const char* name)
{
   void** p = reinterpret_cast<void**>luaL_checkudata(l, index, name);
   return *p;
}
```

（5）我们只需要在初始化Lua环境后，调用regiestA，即可在Lua脚本中使用A这个类了，在Lua中可以这样来使用这个类。

```
a = newA()
a.setVar(123)
print(a.getVar())
deleteA(a)
```

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121313.jpeg)**注意：**关于userdata和lightuserdata的一些区别，userdata是一块由Lua分配管理的内存块，可以存储一个复杂的结构体，而lightuserdata则只是存储一个C指针。

上面这种简单的需求，使用lightuserdata会更合适，**但在一些比较复杂的情况下，lightuserdata会发生错误**，例如getVar返回的是另外一个类，而不是一个简单的数据类型，可能导致lightuserdata的metatable失效。

这里的失效包括所有的lightuserdata对象，而使用userdata则没有这个问题，可以将lua_newuserdata()方法替换为lua_pushlightuserdata()方法，直接将指针p传入。除此之外，luaL_checkudata对userdata有效而对lightuserdata无效。前面这些问题是在Cocos2d-x 3.6版本中进行测试的，笔者认为有可能与Cocos2d-x自带的Lua库有关。但不管怎样，使用userdata要更加稳定一些。