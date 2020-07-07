# 第**20**章　**Cocos2d-x**原生**Lua**框架详解

使用Lua开发可以提升开发效率，降低成本（Lua比C++语言更容易学、用Lua程序员开发比用C++程序员开发费用低、Lua代码比C++代码更难出现问题），便于修改以及能够方便地进行热更新。

Cocos2d-x的原生Lua框架可以很方便地将Lua嵌入到Cocos2d-x程序中，并且在Lua中可以便捷地操作Cocos2d-x。引擎自带的lua-empty-test和lua-tests展示了如何使用Lua来开发Cocos2d-x。

对于熟悉Cocos2d-x的程序员，要使用Lua开发Cocos2d-x程序，就很有必要对Cocos2d-x的原生Lua框架有一个系统的了解，这样更能快速地上手。

本章将系统地介绍Cocos2d-x的原生Lua框架，如果读者只是想简单了解在Cocos2d-x中如何使用Lua，那么只需要阅读20.1和20.2节内容即可，如果希望对Cocos2d-x整个的Lua框架有一个深刻的认识，希望读者能读完本章。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Cocos2d-x原生Lua框架结构。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用Cocos2d-x原生Lua框架。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Cocos2d-x原生Lua框架运行流程。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用genbindings.py导出自定义类。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　扩展Cocos2d-x Lua。

### 20.1　Cocos2d-x原生Lua框架结构

Cocos2d-x的原生Lua框架被封装到了libluacocos2d中，主要由以下几个部分组成，下面详细介绍。

#### 20.1.1　Lua核心层

Lua核心层也就是Lua库，引擎使用的是5.1.4版本的LuaJIT，LuaJIT是一个高效版的Lua库，对于使用者而言，使用Lua和使用LuaJIT并没有太大的区别，只需要替换头文件和链接库，无须改动代码。

LuaJIT的优点是效率比Lua高，在浮点计算、循环、协程切换等方面有显著提升。但对于大量依赖C语言编写的函数程序，性能提升有限。另外，LuaJIT自身还包含了ffi、coco等Lua没有的库。**LuaJIT的缺点是只支持32位内存寻址，而无法在64位的系统上运行，而且不支持Lua5.2，也不够稳定**。但是在最新推出的Cocos2d-x 3.14版本中开始使用luajit 2.10 beta2，它已经开始支持64位的系统了。

#### 20.1.2　Lua脚本引擎

Cocos2d-x通过LuaEngine和LuaStack对Lua的API进行了封装，LuaStack封装了lua_State，而LuaEngine则是一个管理LuaStack的单例。Lua脚本引擎实现了以下功能。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　封装了Lua的API，并使用单例进行管理，使程序员可以更加方便地操作Lua。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　将Cocos2d-x引擎的API导出到Lua。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Cocos2d-x到Lua的消息转发，如触摸、重力感应、节点事件等消息。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Lua脚本的加载，包括按自定义搜索路径查找脚本、加载脚本后进行解密等工作。

另外，Cocos2d-x还提供了LuaBridge，可以在Lua中方便地调用Java和Objective-C代码，大大方便了接入Android和iOS平台的SDK。

#### 20.1.3　Cocos2d-x到Lua的转换层

转换层将Cocos2d-x引擎的C++代码导出给Lua，使程序员可以在Lua代码中使用Sprite、Label、Director等接口。在LuaStack的初始化中，会将这些API导入到Lua环境中。

转换层是独立的，可以分为两部分，第一部分是使用tolua++自动生成的C++代码，放在auto目录下。第二部分则是程序员手动扩展的C++代码，放在manual目录下。

使用tolua++可以批量地为C++中的类生成C++代码，执行生成的C++代码就可以在Lua中使用C++的类，而tolua++对于一些需要特殊处理的接口无法有效地导出，如使用了C++11新特性的一些代码，为了使Lua中能够使用这些接口，所以增加了手动扩展部分，在已导出的类中进行扩展。

Cocos2d-x还提供了genbindings.py，来帮助导出自定义的类，工具位于引擎的tools目录中，后面会介绍其具体的用法。

#### 20.1.4　Lua辅助层

Lua辅助层是Cocos2d-x自带的一些Lua代码（实际上这些代码基本来自于Quick框架），这些代码主要的作用是扩展Cocos2d-x核心功能、定义大量的枚举（因为Cocos2d-x的枚举并未被导出）、废弃函数的警告包装（使用废弃函数就会打印该函数已经废弃，请使用新的函数）、常用数据结构（如Vec2、Rect等）、Json、OpenGL等以及一些便于使用的接口。

其他主要还有xxtea的加密解密接口、以及socket通信的接口。这里不再详细介绍。

### 20.2　使用Cocos2d-x原生Lua框架

#### 20.2.1　在Cocos2d-x中调用Lua

可以在C++代码中方便地执行Lua脚本，在Lua中回调C++代码也是一件轻松的事情。在Cocos2d-x中调用Lua脚本需要有3个步骤：

（1）初始化Lua脚本引擎。

（2）将需要额外注册的C++API注册到Lua中。

（3）执行Lua脚本。

代码如下。

```c++
bool AppDelegate::applicationDidFinishLaunching()
{
    //初始化LuaEngine
LuaEngine* engine = LuaEngine::getInstance();
//将LuaEngine设置到ScriptEngineManager中
    ScriptEngineManager::getInstance()->setScriptEngine(engine);
    lua_State* L = engine->getLuaStack()->getLuaState();
    //注册额外的C++API——CocosDenshion
    lua_module_register(L);
    //下面这行代码被注释起来，因为它会导致ZeroBrane Studio在调试的时候无法找到上下文
    //engine->executeScriptFile("src/hello.lua");
    //执行src/hello.lua脚本
    engine->executeString("require 'src/hello.lua'");
    return true;
}
```

上面是Cocos2d-x自带例子中，lua_empty_test的初始化代码。在初始化LuaEngine的时候，lua_State会被创建，这是Lua的核心对象，是一个独立的Lua环境。同时，Cocos2d-x引擎的API也会被导入到Lua中。而lua_module_register()方法内部调用了register_cocosdenshion_module()方法，将Cocos2d-x音效库导出到Lua中。最后调用LuaEngine的executeString()方法执行了Lua代码。

#### 20.2.2　在Lua中操作Cocos2d-x

engine->executeString("require 'src/hello.lua'")会执行src目录下的hello.lua脚本，而在hello.lua中，使用了Cocos2d-x提供的API，创建了GLView，场景以及场景中的精灵、界面等，粗略演示了如何加载其他Lua脚本，如何监听点击事件、执行Action、使用定时器等内容。下面介绍一下相关的关键知识点，这里需要参考引擎lua_empty_test中的hello.lua来学习。

##### 1．脚本的执行

require可以执行一个脚本，但对同一个脚本require多次，该脚本的内容只会被执行一次。当程序员require一个脚本的时候，会首先判断该脚本是否已经被加载，如果是则直接返回，否则加载、解析并执行该脚本，脚本是被当作一个函数来执行的。

当一个脚本被执行时，会从上到下顺序执行，所以hello.lua文件会按照从上到下的顺序执行：添加res和src到搜索路径中，require cocos.init文件，注册一些变量或函数，调用main()函数执行逻辑。

require "cocos.init"会搜索cocos目录下的init.lua文件，但在src和res目录下并没有这些文件，那么这个文件位于哪里？又做了什么事情呢？

cocos目录的内容位于引擎下的cocos/scripting/lua-bindings/script目录中，lua_empty_ test注册了生成事件，在编译完成后，会将该目录下的内容复制到输出路径的src/cocos目录下，而lua_empty_test项目下的src目录也会被复制到输出路径的src\cocos下。

所以如果程序员自己创建了一个空项目，可以手动将这些脚本复制到项目的src\cocos目录下，或者res目录下，然后再手动指定路径（addSearchPath），脚本本身也是一种资源文件，并没有什么特别的。这些Lua脚本就是前面介绍到的Lua辅助层了，require "cocos.init"可以使这个辅助层生效，后面会对辅助层的脚本进行详细的介绍。

##### 2．使用xpcall

在hello.lua的最后一行，使用xpcall执行了main()函数，并传入了__G__ TRACKBACK__到xpcall中，使用xpcall会以保护模式调用函数，如果内部发生了错误，会回调传入的调试函数。

```
xpcall(main, __G__TRACKBACK__)
```

__G__TRACKBACK__打印了错误信息以及发生错误时的堆栈，这些信息可以很好地帮助程序员定位问题。

```c++
function __G__TRACKBACK__(msg)
    cclog("----------------------------------------")
    cclog("LUA ERROR: " .. tostring(msg) .. "\n")
    cclog(debug.traceback())
    cclog("----------------------------------------")
end
```

##### 3．collectgarbage()方法

在main()函数的开头调用了collectgarbage()方法，这是Lua提供的内存管理方法，该方法接收的第一个参数是选项字符串，下面的代码设置了setpause选项和setstepmul选项。

setpause选项可以设置Lua垃圾收集器周期间的等待时间，传入的参数小于100表示不等待，而当值为200时意味着在总使用内存达到原来的两倍时才开启新的垃圾回收周期。简而言之，该值越小，收集的周期越短，反之则越长。

setstepmul选项可以设置垃圾收集器相对内存分配的速度，设置的数值越大则收集的速度越快，传入的参数小于100时收集器工作非常缓慢，默认值为200，收集器将以内存分配器的两倍速运行。

```
collectgarbage("setpause", 100)
collectgarbage("setstepmul", 5000)
```

此外，collectgarbage()方法还支持以下选项。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　stop：停止垃圾收集器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　restart：重启垃圾收集器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　collect：立即执行一次垃圾收集。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　count：返回当前Lua占用的内存大小。

##### 4．初始化场景

接下来main()函数调用initGLView创建了窗口，并设置了分辨率和FPS。下面这段代码看上去非常熟悉，一般在AppDelegate中会完成这些任务。这里用到的Cocos2d-x API都是在cc这个命名空间下，Cocos2d-x在Lua中还有另外一个命名空间，叫作ccui。cc.Director:getInstance()相当于C++代码中的cocos2d::Director::getInstance()，使用起来非常接近。

```
local function initGLView()
    local director = cc.Director:getInstance()
    --初始化OpenGLView
    local glView = director:getOpenGLView()
    if nil == glView then
        glView = cc.GLViewImpl:create("Lua Empty Test")
        director:setOpenGLView(glView)
    end
    director:setOpenGLView(glView)
    glView:setDesignResolutionSize(480, 320, cc.ResolutionPolicy.NO_BORDER)
    --打开FPS显示开关
    director:setDisplayStats(true)
    --设置FPS
    director:setAnimationInterval(1.0 / 60)
end
```

##### 5．使用引擎

下面的代码简单演示了在Lua中如何操作Cocos2d-x，如创建一个Sprite，可以使用引擎导出的API来创建，并调用Sprite的成员方法来进行操作。

```
local function creatDog()
    local frameWidth = 105
    local frameHeight = 95
    -- 加载图片，并创建一个SpriteFrame
    local textureDog = cc.Director:getInstance():getTextureCache():
    addImage("dog.png")
    local rect = cc.rect(0, 0, frameWidth, frameHeight)
    local frame0 = cc.SpriteFrame:createWithTexture(textureDog, rect)
    rect = cc.rect(frameWidth, 0, frameWidth, frameHeight)
    local frame1 = cc.SpriteFrame:createWithTexture(textureDog, rect)
    -- 创建一个Dog Sprite对象，这里也可以直接传入文件名调用cc.Sprite:create来创
    建一个Sprite
    local spriteDog = cc.Sprite:createWithSpriteFrame(frame0)
    -- 比较特殊的一点是，Lua的对象可以随意地添加属性，就像一个table可以随意添加新的
    内容，下面这种写法直接设置了spriteDog对象的isPaused属性
    spriteDog.isPaused = false
    spriteDog:setPosition(origin.x, origin.y + visibleSize.height / 4 * 3)
    -- 创建动画，并让spriteDog执行这个Action，这样的事情在C++里做了无数遍
   local animation=cc.Animation:createWithSpriteFrames({frame0,frame1}, 0.5)
    local animate = cc.Animate:create(animation)
    spriteDog:runAction(cc.RepeatForever:create(animate))
    -- 定义一个移动函数，并将该函数注册到Schedule中，使其每一帧都执行，tick()函数是
    一个闭包，其直接引用了上一层函数的spriteDog对象，并不断更新对象的位置
    local function tick()
        if spriteDog.isPaused then return end
        local x, y = spriteDog:getPosition()
        if x > origin.x + visibleSize.width then
            x = origin.x
        else
            x = x + 1
        end
        spriteDog:setPositionX(x)
    end
    cc.Director:getInstance():getScheduler():scheduleScriptFunc(tick,0,false)
    return spriteDog
end
```

用table来模拟类使得Lua中的对象获得了动态增删成员变量和方法的能力，闭包的使用也使程序员在设置各种回调函数的时候更加方便灵活。

### 20.3　Cocos2d-x原生Lua框架运行流程

#### 20.3.1　LuaEngine初始化流程

在程序员第一次调用LuaEngine的getInstance()方法时，LuaEngine就会创建一个LuaStack，在LuaStack的初始化方法中，依次执行以下步骤。

（1）使用lua_open()方法创建lua_State，并打开Lua的标准库。

（2）重新注册了print()函数。

（3）将Cocos2d-x的API注册到Lua中。

（4）添加了自定义的Lua加载器。

这些步骤初始化了Lua环境，并将Cocos2d-x的API导入到Lua环境中，使程序员可以在Lua中使用Cocos2d-x。如果程序员开启了CC_USE_PHYSICS，那么会导入物理相关的API。另外还会为iOS和Android平台导出LuaBridge的接口。

```
bool LuaStack::init(void)
{
    //初始化Lua环境并打开标准库
    _state = lua_open();
    luaL_openlibs(_state);
    toluafix_open(_state);
    //注册print()函数到Lua中，这会覆盖lua标准库的print方法
    const luaL_reg global_functions [] = {
        {"print", lua_print},
        {"release_print",lua_release_print},
        {nullptr, nullptr}
    };
    luaL_register(_state, "_G", global_functions);
    //注册Cocos2d-x引擎的API到Lua环节中
    g_luaType.clear();
    register_all_cocos2dx(_state);
    tolua_opengl_open(_state);
    register_all_cocos2dx_manual(_state);
    register_all_cocos2dx_module_manual(_state);
    register_all_cocos2dx_math_manual(_state);
    register_all_cocos2dx_experimental(_state);
    register_all_cocos2dx_experimental_manual(_state);
    register_glnode_manual(_state);
    //导入物理引擎的API
#if CC_USE_PHYSICS
    register_all_cocos2dx_physics(_state);
    register_all_cocos2dx_physics_manual(_state);
#endif
    //导入iOS下用于操作Objective-C的API
#if (CC_TARGET_PLATFORM == CC_PLATFORM_IOS || CC_TARGET_PLATFORM == CC_
PLATFORM_MAC)
    LuaObjcBridge::luaopen_luaoc(_state);
#endif
//导入Android下用于操作Java的API
#if (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)
    LuaJavaBridge::luaopen_luaj(_state);
#endif
    //注册被废弃的Cocos2d-x API到Lua环境中
    register_all_cocos2dx_deprecated(_state);
    register_all_cocos2dx_manual_deprecated(_state);
    tolua_script_handler_mgr_open(_state);
    //添加Cocos2d-x的Lua加载器
    addLuaLoader(cocos2dx_lua_loader);
    return true;
}
```

在完成LuaEngine的初始化之后，可以根据需求再注册其他的模块，如可以注册以下模块。

##### 1．音效模块

cocosdenshion为完整的音效模块，而audioengine为实验中的新音效引擎。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_audioengine_module（音效，试验中）；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_cocosdenshion_module 。

##### 2．CocosBuilder模块

包含了CocosBuilder的API，如register_cocosbuilder_module。

##### 3．CocoStudio模块

包含了CocoStudio1.x和2.x版本的API，如register_cocostudio_module。

##### 4．Cocos2d-x扩展模块

对Cocos2d-x核心模块中的粒子、Control、AssetsManager等做了扩展，如register_extension_module。

##### 5．3D相关模块

分别包含了Cocos2d-x的3D部分、Navmesh自动寻路、3D物理引擎。具体包括：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_cocos3d_module；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_navmesh_module；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_physics3d_module。

##### 6．网络模块

包含了Lua框架中的网络模块，如register_network_module。

##### 7．Spine模块

命名空间为sp，包含了Spine骨骼动画，如register_spine_module。

##### 8．UI模块

命名空间为ccui，包含了Cocos2d-x的UI框架，一系列的Widget。具体包括：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_ui_moudle；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　register_all_cocos2dx_ui_manual。

#### 20.3.2　加载Lua脚本

addLuaLoader()方法将cocos2dx_lua_loader()函数添加到了Lua中的全局变量package下的loaders成员中，它们都是table类型的变量。每当程序员调用require()方法来加载一个脚本文件时，Lua会使用package下的loaders成员中的加载器来加载脚本。

默认Lua的文件搜索规则会在当前路径以及系统特定路径下搜索脚本，而通过cocos2dx_lua_loader()函数，可以使用程序员自己的搜索规则来查找脚本文件，以及实现对Lua脚本的加密和解密。

```
int cocos2dx_lua_loader(lua_State *L)
{
    //后缀为luac和lua
    static const std::string BYTECODE_FILE_EXT = ".luac";
    static const std::string NOT_BYTECODE_FILE_EXT = ".lua";
    //要加载的文件名会被传入，如require "cocos.init"会传入"cocos.init"，在这里
    先把文件名取出
    std::string filename(luaL_checkstring(L, 1));
    //先将文件的.lua和.luac后缀裁剪
    size_t pos = filename.rfind(BYTECODE_FILE_EXT);
    if (pos != std::string::npos)
    {
        filename = filename.substr(0, pos);
    }
    else
    {
        pos = filename.rfind(NOT_BYTECODE_FILE_EXT);
        if (pos == filename.length() - NOT_BYTECODE_FILE_EXT.length())
        {
            filename = filename.substr(0, pos);
        }
    }
    //将所有的'.'替换成'/'，这里的'.'并不包含文件名的后缀
    pos = filename.find_first_of(".");
    while (pos != std::string::npos)
    {
        filename.replace(pos, 1, "/");
        pos = filename.find_first_of(".");
    }
//先在package.path变量中搜索脚本
unsigned char* chunk = nullptr;
ssize_t chunkSize = 0;
std::string chunkName;
FileUtils* utils = FileUtils::getInstance();
//获取package.path变量
lua_getglobal(L, "package");
lua_getfield(L, -1, "path");
std::string searchpath(lua_tostring(L, -1));
lua_pop(L, 1);
size_t begin = 0;
size_t next = searchpath.find_first_of(";", 0);
//遍历path中的所有路径，结合文件名进行组装，最后调用getFileData()加载
//getFileData()方法中会根据FileUtils中的搜索路径到对应的地方读取文件
//如果是Android，getFileData()方法会从apk包中解压资源，这是Lua默认的加载器
无法做到的
//只要读取到文件，则退出while循环
do
{
    if (next == std::string::npos)
        next = searchpath.length();
    std::string prefix = searchpath.substr(begin, next);
    if (prefix[0] == '.' && prefix[1] == '/')
    {
        prefix = prefix.substr(2);
    }
    pos = prefix.find("?.lua");
    chunkName = prefix.substr(0, pos) + filename + BYTECODE_FILE_EXT;
    if (utils->isFileExist(chunkName))
    {
        chunk = utils->getFileData(chunkName.c_str(), "rb", &chunkSize);
        break;
    }
    else
    {
        chunkName = prefix.substr(0, pos) + filename + NOT_BYTECODE_
        FILE_EXT;
        if (utils->isFileExist(chunkName))
        {
            chunk = utils->getFileData(chunkName.c_str(), "rb", &chunk
            Size);
            break;
        }
    }
    begin = next + 1;
    next = searchpath.find_first_of(";", begin);
} while (begin < (int)searchpath.length());
//如果加载成功，则调用luaLoadBuffer()方法加载到Lua中
//如果设置了加密，会在luaLoadBuffer()方法中进行解密，并调用luaL_loadbuffer
//luaL_loadbuffer会将代码编译后作为一个函数放入栈中返回
//require方法所加载的代码会执行luaLoadBuffer()函数，并将函数的返回值返回，一
般是一个table
if (chunk)
{
       LuaStack* stack = LuaEngine::getInstance()->getLuaStack();
       stack->luaLoadBuffer(L, (char*)chunk, (int)chunkSize, chunkName.
       c_str());
       free(chunk);
   }
   else
   {
       CCLOG("can not get file data of %s", chunkName.c_str());
       return 0;
   }
   return 1;
}
```

#### 20.3.3　Cocos2d-x到Lua的事件分发

一般会在需要使用到脚本的地方调用脚本引擎来执行脚本，但引擎与脚本的交互远不止这些，点击事件、重力感应、按钮事件以及Node的自身常用的onEnter、onExit等回调，都是在C++这一端触发的，那么Lua如何接收到这些回调呢？

在初始完LuaEngine之后，将LuaEngine设置到了ScriptEngineManager中，这时LuaEngine就成为了ScriptEngineManager默认的脚本引擎了。在Cocos2d-x中捕获到一些消息的时候，会通过ScriptEngineManager发送事件通知到脚本。

在Cocos2d-x的代码涉及需要从引擎回调脚本的地方，都会判断CC_ENABLE_ SCRIPT_BINDING预定义是否开启，如果是则调用ScriptEngineManager单例的getScriptEngine()方法，获取当前的脚本引擎，并调用脚本引擎的sendEvent()方法，发送事件。

```
void Director::restartDirector()
{
    reset();
    initTextureCache();
    getScheduler()->scheduleUpdate(getActionManager(), Scheduler::PRIORITY_
    SYSTEM, false);
    PoolManager::getInstance()->getCurrentPool()->clear();
    //发送重启事件到脚本中
#if CC_ENABLE_SCRIPT_BINDING
    ScriptEvent scriptEvent(kRestartGame, NULL);
    ScriptEngineManager::getInstance()->getScriptEngine()->sendEvent(&script
    Event);
#endif
}
```

在LuaEngine的sendEvent()方法中，会根据当前的事件类型做不同的处理，最后回调到Lua脚本中。例如Touch事件的处理，首先通过获取到Lua脚本的句柄，这个句柄就是要回调的Lua函数，将点击的各种信息压入栈中，执行Lua函数，最后对栈进行清理。

```
int LuaEngine::handleTouchEvent(void* data)
{
    if (NULL == data)
        return 0;
    TouchScriptData*touchScriptData=static_cast<TouchScript Data*>(data);
    if (NULL == touchScriptData->nativeObject || NULL == touchScriptData->
    touch)
        return 0;
   int handler = ScriptHandlerMgr::getInstance()->getObjectHandler((void*)
   touchScriptData->nativeObject, ScriptHandlerMgr::HandlerType::TOUCHES);
   if (0 == handler)
       return 0;
   switch (touchScriptData->actionType)
   {
       case EventTouch::EventCode::BEGAN:
           _stack->pushString("began");
           break;
       case EventTouch::EventCode::MOVED:
           _stack->pushString("moved");
           break;
       case EventTouch::EventCode::ENDED:
           _stack->pushString("ended");
           break;
       case EventTouch::EventCode::CANCELLED:
           _stack->pushString("cancelled");
           break;
       default:
           return 0;
   }
   int ret = 0;
   Touch* touch = touchScriptData->touch;
   if (NULL != touch) {
       const cocos2d::Vec2 pt=Director::getInstance()->convertToGL(touch->
       getLocationInView());
       _stack->pushFloat(pt.x);
       _stack->pushFloat(pt.y);
       ret = _stack->executeFunctionByHandler(handler, 3);
   }
   _stack->clean();
   return ret;
}
```

当触摸事件触发时，在Cocos2d-x中程序员可以知道哪个节点处理了这个触摸事件，但程序员如何获取到在Lua中这个触摸的处理函数呢？通过ScriptHandlerMgr！所有在Lua中注册，需要让引擎回调的函数都经过了特殊的处理，这个特殊处理是在Cocos2d-x转Lua的manual代码中处理的。例如，在hello.lua中注册了触摸回调，代码如下。

```
local listener = cc.EventListenerTouchOneByOne:create()
listener:registerScriptHandler(onTouchBegan,cc.Handler.EVENT_TOUCH_BEGAN )
listener:registerScriptHandler(onTouchMoved,cc.Handler.EVENT_TOUCH_MOVED )
listener:registerScriptHandler(onTouchEnded,cc.Handler.EVENT_TOUCH_ENDED )
local eventDispatcher = layerFarm:getEventDispatcher()
eventDispatcher:addEventListenerWithSceneGraphPriority(listener,
layerFarm)
```

上面调用了EventListenerTouchOneByOne的registerScriptHandler()方法，但如果查看引擎的源码可以发现，EventListenerTouchOneByOne并没有这个方法，那么这个方法又是从哪来的呢？这个方法是在manual代码中扩展的，在lua_cocos2dx_manual.cpp中扩展了这个方法，代码如下。

```
static int tolua_cocos2dx_EventListenerTouchOneByOne_registerScriptHandler(lua_Sta
te* tolua_S)
{
   if (nullptr == tolua_S)
       return 0;
   int argc = 0;
   EventListenerTouchOneByOne* self = nullptr;
   self = static_cast<EventListenerTouchOneByOne*>(tolua_tousertype (tolua_S,
   1,0));
   argc = lua_gettop(tolua_S) - 1;
   if (argc == 2)
   {
       LUA_FUNCTION handler = toluafix_ref_function(tolua_S,2,0);
       ScriptHandlerMgr::HandlerType type=static_cast<ScriptHandlerMgr::
       HandlerType>((int)tolua_tonumber(tolua_S, 3, 0));
       switch (type)
       {
           case ScriptHandlerMgr::HandlerType::EVENT_TOUCH_BEGAN:
               {
                   ScriptHandlerMgr::getInstance()->addObjectHandler((void*)
                   self, handler, type);
                   self->onTouchBegan = [=](Touch* touch, Event* event){
                       LuaEventTouchData touchData(touch, event);
                       BasicScriptData data((void*)self,(void*)&touchData);
                       return LuaEngine::getInstance()->handleEvent(type,
                       (void*)&data);
                   };
               }
               break;
           case ScriptHandlerMgr::HandlerType::EVENT_TOUCH_MOVED:
               {
                   self->onTouchMoved = [=](Touch* touch, Event* event){
                       LuaEventTouchData touchData(touch, event);
                       BasicScriptData data((void*)self,(void*)&touchData);
                       LuaEngine::getInstance()->handleEvent(type,(void*)&data);
                   };
                   ScriptHandlerMgr::getInstance()->addObjectHandler((void*)
                   self, handler, type);
               }
               break;
           case ScriptHandlerMgr::HandlerType::EVENT_TOUCH_ENDED:
               {
                   self->onTouchEnded = [=](Touch* touch, Event* event){
                       LuaEventTouchData touchData(touch, event);
                       BasicScriptData data((void*)self,(void*)&touchData);
                       LuaEngine::getInstance()->handleEvent(type, (void*)
                       &data);
                   };
                   ScriptHandlerMgr::getInstance()->addObjectHandler((void*)
                   self, handler, type);
               }
               break;
           case ScriptHandlerMgr::HandlerType::EVENT_TOUCH_CANCELLED:
               {
                   self->onTouchCancelled = [=](Touch* touch, Event* event){
                       LuaEventTouchData touchData(touch, event);
                       BasicScriptData data((void*)self,(void*)&touchData);
                       LuaEngine::getInstance()->handleEvent(type, (void*)
                        &data);
                   };
                   ScriptHandlerMgr::getInstance()->addObjectHandler((void*)
                   self, handler, type);
               }
               break;
           default:
               break;
       }
       return 0;
   }
   luaL_error(tolua_S, "%s has wrong number of arguments: %d, was expecting
   %d\n","cc.EventListenerTouchOneByOne:registerScriptHandler",argc,2);
   return 0;
}
```

当registerScriptHandler()方法被调用时，会调用到上面的C++方法，传入EventListenerTouchOneByOne、Lua回调以及触摸类型。在这里会设置EventListenerTouchOneByOne的C++回调，这个C++回调调用了LuaEngine的handleEvent()方法，当触摸事件发生的时候回调这个方法，就会执行到指定的Lua函数。而就是在这里将Lua函数的句柄以及监听者添加到了ScriptHandlerMgr中。但其实没有ScriptHandlerMgr也可以实现，只需要在匿名函数中记录Lua脚本的句柄即可，Widget的addClickEventListener就将Lua脚本的句柄直接记录在了匿名函数中。这里统一将事件注册到了ScriptHandlerMgr中，可以防止一些对象被释放之后，对象的回调仍然被触发。

既然说到了脚本句柄的注册以及使用，如果是一个严谨的程序员，肯定会想要了解脚本句柄什么时候会从ScriptHandlerMgr中被移除。有以下两种时机：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在Lua脚本中手动调用反注册接口进行删除。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　与该脚本句柄绑定的Lua对象释放时会自动删除其所有脚本句柄。

除了触摸事件之外，Cocos2d-x还支持很多事件，所有支持的事件如下。

```
enum ScriptEventType
{
    //节点事件
    kNodeEvent = 0,
    //Menu点击事件
    kMenuClickedEvent,
    //函数回调事件
    kCallFuncEvent,
    //Schedule回调事件
    kScheduleEvent,
    //单点触摸回调事件
    kTouchEvent,
    //多点触摸回调事件
    kTouchesEvent,
    //键盘事件
    kKeypadEvent,
    //重力感应事件
    kAccelerometerEvent,
    //控件事件——Cocos2d-x扩展的CCControl系列控件
    kControlEvent,
    //通用事件，EditorBox、WebSocket等对象都使用了该事件
    kCommonEvent,
    //组件事件，由Component发出，用于执行组件的onEnter、onExit、update回调，JS
    脚本专用
    kComponentEvent,
    //游戏重启事件——由Director的restartDirector()方法发出
    kRestartGame
};
```

对于节点，Cocos2d-x还支持以下子事件，在节点执行如onEnter、onExit等回调的时候，可以通知Lua。

```
enum {
    kNodeOnEnter,
    kNodeOnExit,
    kNodeOnEnterTransitionDidFinish,
    kNodeOnExitTransitionDidStart,
    kNodeOnCleanup
};
```

在节点的onEnter回调中，如果开启了脚本支持预定义，则会执行以下的代码，在onExit、cleanup等回调中，也会执行这段代码，并传入不同的子事件名。后面会介绍如何在Lua中监听这些事件。

```
ScriptEngineManager::sendNodeEventToLua(this, kNodeOnEnter);
```

#### 20.3.4　Lua辅助层初始化流程

Lua辅助层的初始化流程非常简单，只是将所有需要引用到的脚本进require。而绝大部分的脚本功能都非常简单，只是定义了一些table来映射Cocos2d-x中的枚举而已，如前面用到的cc.Handler.EVENT_TOUCH_BEGAN。这一层的脚本文件看起来虽多，但关键的内容只有两个目录，就是cocos2d目录和framework目录。

##### 1．cocos2d目录下的关键脚本

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Cocos2d.lua定义了大量常用结构体的操作接口，如Vec2、Size、Rect等。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　functions.lua提供了大量公用方法。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　json.lua提供了JSON的编码和解码接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　luaj.lua提供了在Lua中操作Java的接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　luaoc.lua提供了在Lua中操作Objective-C的接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Opengl.lua提供了在Lua中操作OpenGL的接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　bitExtend.lua提供了在Lua中的二进制操作接口。

##### 2．framework目录

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　event.lua定义了一个类似EventDispatcher的消息机制。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　extends目录下对Cocos2d-x中的各种节点进行了扩展，使其使用起来更加方便。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　audio.lua提供了一个audio库，里面简单封装了Cocos2d-x的声音引擎。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　device.lua定义了一些与设备相关的变量和方法，如语言、平台、路径分隔符等。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　display.lua定义了一些显示相关的常用变量和方法，如屏幕尺寸、分辨率、常用颜色值等。

#### 20.3.5　Lua辅助层的实用工具

##### 1．class()函数

class()函数是functions.lua中定义的方法，可以使用该函数定义或继承一个类，是非常实用的一个方法，使用class()方法还可以继承Cocos2d-x的类。下面简单介绍一下class()方法的实现。

```
function class(classname, ...)
    -- 创建一个新的table作为类
    local cls = {__cname = classname}
    -- 如果有要继承的父类，这里允许多继承
    local supers = {...}
    for _, super in ipairs(supers) do
        local superType = type(super)
        if superType == "function" then
            -- 如果父类是一个函数，则将该函数设置到__create字段
            cls.__create = super
        elseif superType == "table" then
            if super[".isclass"] then
                cls.__create = function() return super:create() end
            else
                -- 如果父类是一个纯粹的Lua类，将其添加到子类的父类列表的尾部
                cls.__supers = cls.__supers or {}
                cls.__supers[#cls.__supers + 1] = super
                if not cls.super then
                    -- 设置第一个继承的父类为super属性
                    cls.super = super
                end
            end
        end
    end
    -- 设置__index元方法，当需要访问变量的时候，会依次从所有的父类中查找
    cls.__index = cls
    if not cls.__supers or #cls.__supers == 1 then
        setmetatable(cls, {__index = cls.super})
    else
        setmetatable(cls, {__index = function(_, key)
            local supers = cls.__supers
            for i = 1, #supers do
                local super = supers[i]
                if super[key] then return super[key] end
            end
        end})
    end
    --为实例添加一个默认的空构造函数，以避免该类没有定义构造函数（new的时候会调用到）
    if not cls.ctor then
        cls.ctor = function() end
    end
    -- 添加一个new函数到cls，相当于Cocos2d-x的create()方法
    cls.new = function(...)
        -- 创建一个实例，将其__index元方法设置为cls，最后调用其ctor构造函数并返回
        local instance
        if cls.__create then
            instance = cls.__create(...)
        else
            instance = {}
        end
        setmetatableindex(instance, cls)
        instance.class = cls
        instance:ctor(...)
        return instance
    end
    -- 添加一个create()函数到cls，create函数更符合使用习惯
    -- 下面第一个参数_是self参数，因为这里用不到，所以参数名直接使用_，这是一种惯用法
    cls.create = function(_, ...)
        return cls.new(...)
    end
    -- 注意，返回的cls是给我们调用new()方法的模板，不要直接使用
    return cls
end
```

使用class()函数定义类的方法与定义一个Lua模块的方法很类似，下面简单演示一下定义一个继承于Sprite的类。继承C++的类需要用一个函数封装返回其实例，因为每次创建一个新的对象时，都需要一个新的C++对象，所以需要通过函数的方式让class知道如何创建这个对象。

```
-- 定义了一个MySprite类，继承于Sprite
local MySprite = class("MySprite", function ()
    return cc.Sprite:create()
end)
-- 编写其构造函数，每一个实例被创建时都会执行ctor()函数
function MySprite:ctor()
    print("MySprite ctor")
end
-- 编写自定义的成员函数
function MySprite:myFun()
    print("myFun")
end
-- 返回MySprite
return MySprite
```

现在得到了一个类，接下来需要使用这个类来创建该类的对象，像下面这种写法是错误的：

```
local Sprite1 = require "MySprite"
local Sprite2 = require "MySprite"
```

Sprite1和Sprite2都是同一个对象，所以需要使用new()函数来创建新的实例，正确的使用方法如下。

```
local MySprite = require "MySprite"
local Sprite1 = MySprite.new()
local Sprite2 = MySprite.new()
```

##### 2．iskindof()函数

在Lua中想要知道一个类的类型并不是很容易，因为一般的类型都是table，但使用iskindof()函数可以判断一个类的类型，iskindof()函数会递归查找其父类，依次判断其类型。如果是C++的类，会调用tulua封装的方法来获取类名。

使用的方法如下，传入对象以及类名字符串，如果该对象是一个指定的类对象，则打印消息。需要注意的是，指定Cocos2d-x的类时，需要加上cc或ccui等命名空间。

```
if iskindof(spriteDog, "cc.Sprite") then
    print("is Sprite")
end
```

metatable的名字会被命名为类名，iskindof()函数通过获取metatable的名字来判断类名，如果是userdata，说明是C++对象，这里会调用tolua.getpeer来获取其metatable。获取到metatable之后，调用了iskindof_()函数来进行判断。

iskindof_()中会判断metatable的__index字段，如果是一个table，则获取其__cname来判断。此外，也会获取该metatable的__cname字段来判断。如果判断不成功，则会递归遍历其所有的父类进行判断。这里有两个比较特别的地方，第一个是iskindof_()函数的定义，这里是声明了一个iskindof_变量，然后才进行赋值，这是因为使用到了递归，在编译iskindof_()函数时发现需要调用iskindof_()函数，而此时这个函数还未完成，所以需要告诉Lua有这个变量。另外这里获取table的操作使用的是rawget()方法，该方法可以从table中直接获取table的值而不经过metatable。

```
local iskindof_
iskindof_ = function(cls, name)
    local __index = rawget(cls, "__index")
    if type(__index) == "table" and rawget(__index, "__cname") == name then
    return true end
    if rawget(cls, "__cname") == name then return true end
    local __supers = rawget(cls, "__supers")
    if not __supers then return false end
    for _, super in ipairs(__supers) do
        if iskindof_(super, name) then return true end
    end
    return false
end
function iskindof(obj, classname)
    local t = type(obj)
    if t ~= "table" and t ~= "userdata" then return false end
    local mt
    if t == "userdata" then
        if tolua.iskindof(obj, classname) then return true end
        mt = tolua.getpeer(obj)
    else
        mt = getmetatable(obj)
    end
    if mt then
        return iskindof_(mt, classname)
    end
    return false
end
```

##### 3．handler()函数

当定义了一个类，需要注册类的成员方法作为回调时，一般都需要创建一个闭包，因为类的成员方法要求的第一个参数就是self，不能直接将类的成员方法注册为回调。而handler()函数为此提供了方便，只需要传入类对象以及类的成员方法，即可自动返回一个闭包，这个闭包可以作为回调函数进行注册。使用方法如下。

```
local callback = handler(mysprite, mysprite.onclick)
button:addClickEventListener(callback)
```

由于这里是要传入回调的成员方法，而不是要调用成员方法，所以上面指定mysprite的成员方法是用.而不是用:。handler()函数返回的闭包会调用对象的方法，并将对象本身作为第一个参数传入，这符合Lua的面向对象规则。此外，handler()函数的参数会被完整地转发到回调的成员函数中。

```
function handler(obj, method)
    return function(...)
        return method(obj, ...)
    end
end
```

##### 4．NodeEx脚本

在framework下的extends目录中，是Lua辅助层对Cocos2d-x核心节点的一些扩展，目的是使程序员在Lua中能够更加方便地使用这些节点。NodeEx.lua是其中比较重要的一个脚本，其中提供了节点回调处理。

在C++中程序员一般习惯在节点的onEnter和onExit等回调中编写一些逻辑，但在Lua中，需要做一些特殊处理才能开启它们。通过调用Node的registerScriptHandler()方法，可以注册一个回调函数到节点中，在Node的onEnter()、onExi()t、onEnterTransitionFinish()、onExitTransitionStart()、cleanup()等方法被调用时，回调该函数。

在注册的该回调函数中，需要根据Cocos2d-x中传入的字符串来判断到底触发了哪个事件，然后再调用具体的函数进行处理。而NodeEx脚本大大简化了这个步骤，只需要调用Node的enableNodeEvents()方法，即可开启节点事件回调。enableNodeEvents()的实现如下。

```
function Node:enableNodeEvents()
    -- 放置重复注册
    if self.isNodeEventEnabled_ then
        return self
    end
    -- 注册了
    self:registerScriptHandler(function(state)
        if state == "enter" then
            self:onEnter_()
        elseif state == "exit" then
            self:onExit_()
        elseif state == "enterTransitionFinish" then
            self:onEnterTransitionFinish_()
        elseif state == "exitTransitionStart" then
            self:onExitTransitionStart_()
        elseif state == "cleanup" then
            self:onCleanup_()
        end
    end)
    self.isNodeEventEnabled_ = true
    return self
end
```

上面的函数return了self，是为了能够进行连续操作（类似连续赋值）。在回调中根据不同的事件触发了不同的方法，而在onXXX_方法的内部，会回调onXXX方法，例如，enter事件触发了onEnter_方法，在onEnter_方法内又回调了onEnter()方法，只要开启了节点事件回调，在类的定义中添加对应的回调方法即可。

但是为什么要这么麻烦？而不直接回调onEnter()方法呢？因为有时候程序员希望在调用onEnter()方法时执行某一个函数，但这个函数的名字不一定是onEnter()方法，所以NodeEx多提供了一个方法，让程序员来设置自定义的回调。使用节点的onNodeEvent()方法，传入要监听的节点事件名称，并指定回调，即可在节点事件触发时回调程序员指定的方法，当然其默认的方法（如onEnter()函数）仍然会执行。这是一个额外的扩展。

```
function Node:onNodeEvent(eventName, callback)
    if "enter" == eventName then
        self.onEnterCallback_ = callback
    elseif "exit" == eventName then
        self.onExitCallback_ = callback
    elseif "enterTransitionFinish" == eventName then
        self.onEnterTransitionFinishCallback_ = callback
    elseif "exitTransitionStart" == eventName then
        self.onExitTransitionStartCallback_ = callback
    elseif "cleanup" == eventName then
        self.onCleanupCallback_ = callback
    end
    self:enableNodeEvents()
end
```

### 20.4　使用genbindings.py导出自定义类

Cocos2d-x的**cocos2d-x/tools/tolua**目录中有一个genbindings.py脚本可以**批量导出**自定义的C++类到Lua中，但一般情况下不会用到，因为在开发的过程中，要么以C++代码为主，要么以Lua为主，如果没有主次的话，那么这两个模块之间的联系也应该尽量地越少越好。在这种情况下，我们并不会导出太多的类给到Lua这边，所以按照第18章中介绍的table与面向对象的方式，简单导出一个类即可，不必如此复杂。

导出C++的类到Lua中有很多第三方的库，而Cocos2d-x使用的是tolua++，通过编写tolua++的pkg配置文件，来定义要导出的每一个类的信息，这个步骤相当于用tolua++的规则将类的头文件重写成pkg文件，tolua++会根据这个文件以及类的cpp文件来生成C++代码文件，将生成的C++代码文件并入项目中，然后执行它们可以将类导出到Lua中。

如果直接使用tolua++，当需要批量地导出类时就会很麻烦，因为需要编写大量的pkg文件，来描述所有的类。例如，将Cocos2d-x引擎的API全部导出到Lua中，那将是一个累人的苦力活。所以基于tolua++，Cocos2d-x提供了genbindings.py来完成**批量导出**的工作。使用的步骤如下。

（1）编写要导出的C++的类。

（2）为这个类编写一个ini配置文件（一个ini可以对应多个类）。

（3）修改genbindings.py脚本，使其加载ini配置。

（4）执行genbindings.py脚本，使其生成将类导出到Lua的C++代码。

（5）将生成的C++代码添加到项目中，并执行注册方法。

#### 20.4.1　各个平台的环境搭建

在tolua目录中的README.mdown文件详细介绍了如何在Windows、Mac以及Linux下搭建环境。在使用genbindings.py之前，需要先将环境搭建好。

##### 1．Windows环境搭建

Windows环境搭建的步骤如下。

（1）安装android-ndk-r9b，并将NDK的路径设置到环境变量NDK_ROOT中。

（2）下载并安装python2.7.3（32位，地址为http://www.python.org/ftp/python/2.7.3/ python-2.7.3.msi）。

（3）将python的安装路径设置到环境变量PATH中（如C:\Python27目录）。

（4）下载并安装pyyaml（地址为http://pyyaml.org/download/pyyaml/PyYAML-3.10. win32-py2.7.exe）。

（5）下载pyCheetah，并将其解压到python安装路径下的Lib\site-packages目录下（地址为https://raw.github.com/dumganhar/my_old_cocos2d-x_backup/download/downloads/ Cheetah.zip）。

（6）最后就可以在cocos2d-x\tools\tolua目录下执行genbindings.py脚本生成代码了，生成的代码会被放在cocos\scripting\auto-generated\js-bindings目录下。

##### 2．Mac环境搭建

使用Homebrew（地址为http://brew.sh/）来安装python，打开Homebrew的网址，将语言选择为中文，按照网站提示步骤，在终端输入安装代码即可装好Homebrew（这是一个Mac下很不错的工具，可以方便地安装各种开发工具和库）。安装完Homebrew之后，就可以在终端输入brew install python来安装python了。由于OSX 10.9及以上版本内置了python 2.7，所以可以直接使用不需要安装。

接下来在终端执行3行代码自动安装pyYAML和Cheetah，先安装pip，再使用pip来安装其他两个工具。

```
sudo easy_install pip
sudo pip install PyYAML
sudo pip install Cheetah
```

然后下载NDK，地址为http://dl.google.com/android/ndk/android-ndk-r9b-darwin-x86_64.tar.bz2，并将NDK进行解压。

最后执行进入cocos2d-x\tools\tolua目录下执行genbindings.py脚本生成代码，在该目录下执行下面两行语句（注意将android-ndk-r9b-path替换为NDK解压后的路径）。

```
export NDK_ROOT=/android-ndk-r9b-path
./genbindings.py
```

##### 3．Linux环境搭建

Linux环境的搭建和Mac类似，这里使用的是Ubuntu版本，需要执行以下命令安装python、PyYAML和Cheetah。

```
sudo apt-get install python2.7
sudo apt-get install python-pip
sudo pip install PyYAML
sudo pip install Cheetah
```

接下来下载并解压NDK（android-ndk-r9b），最后执行导出NDK_ROOT环境变量以及生成脚本的命令（NDK地址为https://dl.google.com/android/ndk/android-ndk-r9b-linux-x 86_64.tar.bz2）。

```
export NDK_ROOT=/path/to/android-ndk-r9b
./genbindings.py
```

#### 20.4.2　编写要导出的C++的类

搭建完环境之后，先编写一个简单的类，首先这个类继承于cocos2d::Ref，类提供了两个方法，返回一个字符串以及一个vector容器，tulua++可以自动识别C++标准库的容器。头文件的内容大致如下。

```
#include <string>
#include <vector>
#include <cocos2d.h>
class CTest : public cocos2d::Ref
{
public:
    CTest();
    virtual ~CTest();
    std::string& getString() { return m_String; }
    std::vector<int> getIntArray() { return m_Array; }
private:
    std::vector<int> m_Array;
    std::string m_String;
};
```

然后在源文件中，实现CTest的构造函数，在构造函数中先添加一些值到两个成员变量中。代码如下。

```
CTest::CTest()
{
    m_String = "TestStr";
    m_Array.push_back(1);
    m_Array.push_back(2);
}
```

#### 20.4.3　编写ini配置文件

在tolua目录下有大量ini配置文件，这些配置文件用于描述要导出的类，因此需要仿照这些配置文件为要导出的类编写一个ini配置文件，ini配置文件的结构由段-键-值3部分组成，一个ini文件可以分为多个段，段的名称用中括号包裹起来并独占一行（这里只用到了一个段）。而键和值是以键值对的方式一一对应的，每个段可以有多个键值对，键不能重复，但键对应的值可以为空（可以简单将键和值的类型都视为字符串）。另外，ini使用#作为注释符号，相当于C/C++代码中的//以及Lua中的--。下面的代码简单地介绍了这些规则。

```
# 这是注释
[section 1]
key1 = value1
key2 = value2
key3 =
```

在编写配置文件的时候，还有一些额外的规则是genbindings.py规定的，必需要遵循这些规则。首先需要指定一个段名并且只有一个段。另外段中的键有各种含义，下面简单介绍几个比较重要的键的含义。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　prefix函数前缀：每个生成的C++方法都会加上这个前缀。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　target_namespace目标命名空间：在Lua中访问时需要加上这个空间，如Node的命名空间是cc。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　macro_judgement平台预处理：可以控制代码在哪些平台下生效，填空为所有平台都生效。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　headers：为要导出的C++类的头文件，可以用空格分隔开多个文件，也可以指定一个头文件，然后让这个头文件包含所有引用到的头文件。这个键主要用于帮助genbindings.py找到头文件。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　classes：为要导出的Lua类，因为头文件中可能包含了多个类，所以需要指定classes字段将要导出的类罗列出来。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　skip：为需要跳过的类方法，当导出了某个类但不希望导出该类的某个方法时指定。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　rename_functions：可以指定将某个类的某个方法以一个新的名字导出。

这些字段具体如何填写，可以参考cocos2dx.ini文件，在这里先复制一个cocos2dx.ini，改名为test.ini，然后将段名进行修改并对以下键值对进行修改。

```
[Test]
prefix = myTest
target_namespace =
macro_judgement =
# 指定导出文件的完整路径，这里使用了cocos目录作为相对路径
headers = %(cocosdir)s/tests/lua-empty-test/project/Classes/Test.h
# 导出Test类
classes = CTest
skip =
rename_functions =
```

请找到上面列举出来的键，并将它们的值进行设置。

#### 20.4.4　修改并执行genbindings.py

genbindings是一个python脚本，该脚本的功能是搜索指定的ini配置文件，根据这些配置调用bindings-generator来生成导出的C++代码。打开genbindings.py找到134行左右，可以发现以下代码。

```
# 这里指定的是输出目录，可以自己定义一个目录
output_dir = '%s/cocos/scripting/lua-bindings/auto' % project_root
# 这里指定的是要加载的ini配置文件，以及配置文件对应的段名，生成的C++文件的文件名等
cmd_args = {'cocos2dx.ini' : ('cocos2d-x', 'lua_cocos2dx_auto'), \
            'cocos2dx_extension.ini' : ('cocos2dx_extension', 'lua_cocos
            2dx_extension_auto'), \
            'cocos2dx_ui.ini' : ('cocos2dx_ui', 'lua_cocos2dx_ui_auto'), \
```

在这里程序员可以将cmd_args变量中的其他代码都注释，然后再插入一行代码来指定程序员自己的类。

```
cmd_args = { 'test.ini' : ('Test', 'lua_test_auto'), \
            #'cocos2dx.ini' : ('cocos2d-x', 'lua_cocos2dx_auto'), \
            #'cocos2dx_extension.ini' : ('cocos2dx_extension', 'lua_cocos
            2dx_extension_auto'), \
            #'cocos2dx_ui.ini' : ('cocos2dx_ui', 'lua_cocos2dx_ui_auto'), \
```

然后在命令行中执行这个脚本，生成cpp文件。

#### 20.4.5　注册并在Lua中使用

在执行完genbindings.py脚本之后，会在引擎的cocos\scripting\lua-bindings\auto目录下生成名为lua_test_auto的头文件和源文件，需要将这两个文件添加到C++项目中（可以手动复制到项目中并添加）。

在lua_test_auto.h中，tolua++生成了一个名为register_all_myTest()的注册函数，可以将CTest类注册到Lua中。在完成Cocos2d-x Lua框架的注册之后，需要手动调用一下这个函数。可以在官方的lua_empty_test中测试一下，代码如下（别忘了先包含头文件）：

```
bool AppDelegate::applicationDidFinishLaunching()
{
    LuaEngine* engine = LuaEngine::getInstance();
    ScriptEngineManager::getInstance()->setScriptEngine(engine);
    lua_State* L = engine->getLuaStack()->getLuaState();
    lua_module_register(L);
    //注册CTest类到Lua环境中
    register_all_myTest(L);
    engine->executeString("require 'src/hello.lua'");
    return true;
}
```

最后可以在Lua中使用CTest这个类了，首先创建一个CTest类，然后调用该类的两个get()方法，并将获取到的内容进行打印。genbindings.py生成的每一个类，tolua++都会自动为其生成名为new的构造方法，我们需要使用new()方法来创建这个类的实例。

```
-- 使用new()方法来创建一个test对象
local test = CTest.new()
-- tolua++将string对象转换成了Lua中的string
print(test:getString())
local arr = test:getIntArray()
-- tolua++将vector转换成了Table数组
for _,v in ipairs(arr) do
    print(v)
end
```

将上面的Lua脚本添加到lua_empty_test项目的src目录下的hello.lua文件尾部，然后运行程序，可以在控制台中看到如图20-1所示的输出。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200623121320.jpeg)

图20-1　运行程序后控制台的输出

### 20.5　扩展Cocos2d-x Lua

当程序员需要使用的某些Cocos2d-x的方法没有被导出到Lua时，需要手动地扩展一下Cocos2d-x。这和导出自定义类不大相同，因为这些方法是自动导出脚本无法正确导出的类，否则就会在genbindings.py导出的auto文件里面了。

扩展是基于原有内容的基础上进行，所以如果程序员需要扩展Spine，那么就需要先将Spine类导出到Lua，然后再扩展。默认的扩展一般在LuaEngine初始化时会被注册，否则需要先手动进行注册，在LuaEngine初始化完成后，可以调用register_spine_module()方法将Spine模块进行注册。

扩展需要两个步骤，首先实现要扩展的方法，然后将扩展方法注册到该类的table中。扩展方法的编写可以参考libluacocos2d项目的manual目录下的文件，下面会举例简单介绍，由于我们的类是用tolua++导出的，所以导出时需要使用tolua++的API。tolua++是将类导出为table，而扩展一个类实际上就是将新的方法添加到这个table中。

#### 20.5.1　编写扩展方法

```
//需要包含LuaBasicConversions头文件
#include "LuaBasicConversions.h"
//这里扩展Spine的SkeletonAnimation的一个方法，用于判断是否存在某动画
static int lua_extend_spine_existAnimation(lua_State* tolua_S)
{
    if (nullptr == tolua_S)
        return 0;
    int argc = 0;
    spine::SkeletonAnimation* cobj = nullptr;
    bool ok = true;
#if COCOS2D_DEBUG >= 1
    tolua_Error tolua_err;
#endif
    //这里的sp.SkeletonAnimation是在Lua中的类名，就如节点的类名为cc.Node一样
#if COCOS2D_DEBUG >= 1
    if (!tolua_isusertype(tolua_S, 1, "sp.SkeletonAnimation", 0, &tolua_
    err)) goto tolua_lerror;
#endif
    //取出第一个参数，也就是self
    cobj = (spine::SkeletonAnimation*)tolua_tousertype(tolua_S, 1, 0);
#if COCOS2D_DEBUG >= 1
    if (!cobj)
    {
        tolua_error(tolua_S, "invalid 'cobj' in function 'lua_extend_spine_
        existAnimation'", nullptr);
        return 0;
    }
#endif
    //判断参数数量是否正确
    argc = lua_gettop(tolua_S) - 1;
    if (argc == 1)
    {
        const char* arg;
        //Lua传入的第二个参数就是动画名，这里作为字符串取出
        std::string arg_tmp;
        ok &= luaval_to_std_string(tolua_S, 2, &arg_tmp, "sp.Skeleton
        Animation:existAnimation");
        arg = arg_tmp.c_str();
        if (!ok)
            return 0;
        //接下来进行判断是否有该函数，并将返回值push到Lua的栈中，然后返回
        spAnimationState* state = cobj->getState();
        if (state && state->data)
        {
            spSkeletonData* const data = state->data->skeletonData;
            if (data)
            {
                for (int i = 0; i < data->animationsCount; i++)
                {
                    if (0 == strcmp(data->animations[i]->name, arg))
                    {
                        lua_pushboolean(tolua_S, 1);
                        return 1;
                    }
                }
            }
        }
        //如果找不到该动画，则返回false
        lua_pushboolean(tolua_S, 0);
        return 1;
    }
    luaL_error(tolua_S, "%s has wrong number of arguments: %d, was expecting
    %d \n", "existAnimation", argc, 1);
    return 0;
#if COCOS2D_DEBUG >= 1
tolua_lerror:
    tolua_error(tolua_S, "#ferror in function 'lua_summoner_extend_spine_
    existAnimation'.", &tolua_err);
#endif
    return 0;
}
```

#### 20.5.2　注册到类中

在一张特殊的表LUA_REGISTRYINDEX里面，取出指定的类的table，然后调用tolua_function()方法，将我们的方法注册进这张表中。代码如下。

```
static void extendSpine(lua_State* L)
{
    lua_pushstring(L, "sp.SkeletonAnimation");
    lua_rawget(L, LUA_REGISTRYINDEX);
    if (lua_istable(L, -1))
    {
        tolua_function(L,                       "existAnimation",
lua_extend_spine_existAnimation);
    }
    lua_pop(L, 1);
}
```

### 20.6　lua-tests导读

lua-test中包含了丰富的示例代码，在读完本章内容之后简单阅读一下这些代码，可以帮助读者更深入地理解如何使用Lua来编写Cocos2d-x的程序，并起到很好的巩固作用。如果读者希望了解如何使用Cocos2d-x的某个功能，也可以在这里快速找到使用的代码。这里简单介绍几个比较重要的例子。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在NodeTest例子中可以了解到大部分的节点操作以及在Lua中使用节点时需要特别注意的地方，如创建、添加和删除节点，操作节点的各种属性，在节点中执行Action、Schedule，监听点击事件和节点事件等。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在AssetsManagerTest例子中可以了解如何在Lua中使用AssetsManager，在程序运行中动态更新资源。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在ByteCodeEncryptTest例子中可以了解如何加载并执行经过加密处理的Lua字节码。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在CaptureScreenTest例子中可以了解如何在Lua中对游戏进行截屏。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在LuaBridgeTest例子中可以了解如何在Lua中调用Java代码和Objective-C代码。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在OpenGLTest例子中可以了解如何在Lua中操作OpenGL（主要演示了一些Shader的使用，效果同cpp-tests中的ShaderTest）。

在DrawPrimitivesTest例子中可以了解如何在Lua中绘制基础的图元。