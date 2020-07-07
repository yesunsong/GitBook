# 第21章　Menu和MenuItem详解

Cocos2d-x中最简单、常用的交互方法就是使用Menu，使用Menu和MenuItem可以方便地创建我们的按钮，本章将详细介绍Menu，本章主要介绍以下内容：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　使用Menu和MenuItem。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　结构框架与执行流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　MenuItem详解。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　实现TabView。

## 21.1　使用Menu和MenuItem

Menu和MenuItem的使用非常简单，可以先创建一系列的MenuItem，最后用其创建一个Menu。**也可以先将Menu创建出来，然后将MenuItem逐个地添加到Menu中**。

```
//创建一个具体的MenuItem——MenuItemImage
//指定按钮正常状态下和被选中状态下的图片，并传入点击回调函数
auto closeItem = MenuItemImage::create(
                                    "CloseNormal.png",
                                    "CloseSelected.png",
                                    CC_CALLBACK_1(HelloWorld::menuCloseCal	                                 lback,this));
//用于create一个Menu对象，不要忘了在最后加上NULL
auto menu = Menu::create(closeItem, NULL);
this->addChild(menu, 1);
//直接addChild到Menu下也是可以的
auto item2 = closeItem->clone();
menu->addChild(item2);
```

需要注意的是位置，如果不设置Menu的位置直接添加到场景，**默认Menu的位置会被设置为屏幕中心**（init的时候会调用setPosition，将位置设置为winSize的一半）。而MenuItem的位置会相对父节点的位置，也就是说直接添加一个MenuItem到Menu中，会出现在屏幕的中间。如果设置了Menu的位置，MenuItem会根据设置的位置来调整。

## 21.2　结构框架与执行流程

Menu和MenuItem的类结构如图21-1所示，Menu要求所有的子节点必须是MenuItem对象或其子类对象，Menu管理着若干MenuItem，主要管理MenuItem的状态、布局、回调等，而MenuItem只负责如何显示，以及绑定点击回调。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160030.jpeg)

图21-1　Menu和MenuItem的类图

### 21.2.1　Menu详解

Menu的setEnabled和setVisible方法都可以禁用、启用菜单，但不一样的是setEnable直接注册、注销触摸监听，当setEnable为false时，只是触摸失效。而Visible是在运行中判断过滤，Visible为false则Menu不会显示出来。以下是Menu关于菜单项布局排列的一些方法：

```
//把所有子菜单项垂直排列，默认间隔为5
void alignItemsVertically();
//把所有子菜单项垂直排列，每两个菜单项间隔padding单位
void alignItemsVerticallyWithPadding(float padding);
//把所有子菜单项水平排列，默认间隔为5
void alignItemsHorizontally();
//把所有子菜单项水平排列，每两个菜单项间隔padding单位
void alignItemsHorizontallyWithPadding(float padding);
//按列来排列子菜单项
void alignItemsInColumns(int columns, va_list args);
//按行来排列子菜单项
void alignItemsInRows(int rows, va_list args);
```

其中alignItemsInColumns和alignItemsInRows在此特别说明一下，按行和按列排列不需要设置MenuItem的位置，行列的排列自动以Menu所在的位置为中心进行排列，菜单项之间根据每个菜单项的大小以及窗口的大小进行排列。需要注意的是，**传入的参数总和必须等于菜单项的数量**。下面看一下如何使用：

```
auto menu = Menu::create();
menu->setPosition(visibleSize * 0.5);
addChild(menu, 1);
//依次添加九个菜单
for (int i = 0; i < 9; ++i)
{
    auto item = MenuItemImage::create(
        "CloseNormal.png",
        "CloseSelected.png",
        CC_CALLBACK_1(HelloWorld::menuCloseCallback, this));
    menu->addChild(item);
}
//分为3列，第1、第2列放4个菜单，第3列放1个菜单
menu->alignItemsInColumns(4, 4, 1, NULL);
```

运行效果如图21-2所示，第1列和第2列的位置是按照窗口的宽度除以4来平均排列的，第3列则直接居中。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160031.jpeg)

图21-2　Menu的alignItemsInColumns方法

### 21.2.2　MenuItem详解

MenuItem作为一个基类封装了很多功能，但不直接使用，而是使用其子类，MenuItem主要封装了以下功能：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　菜单项外部包围盒计算——rect方法。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　禁用和启用菜单项——setEnabled。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　菜单项的选中状态——selected相关方法。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　菜单项的点击回调——ccMenuCallback成员以及activate方法。

### 21.2.3　执行流程

MenuItem并不监听触摸消息，所有的触摸统一由Menu处理，在手指按下时，Menu先判断自身是否可见（包括所有的父节点），再遍历_children列表，如果子节点isVisible和isEnabled都为true，再进一步判断是否在点击范围内，如果是则调用Item的selected方法，并设置为当前选中Item。

当手指移动时，会不断检测是否碰到了新的MenuItem，如果是则调用当前Item的unselected方法，并调用新Item的selected方法。将当前选中的Item设置为新Item。如果手指移动出当前Item的范围，或在移动的过程中，Item被禁用或者隐藏，同样会调用Item的unselected方法，并将当前Item置为NULL。

当手指松开时，如果当前Item不为NULL，依次回调当前Item的unselected方法和activate方法，在activate方法中执行Item的点击回调。如果接收到的是Cancel消息，则只会调用Item的unselected方法。

一般，在MenuItem的selected方法中，会切换到选中状态，并替换选中时的显示内容。在unselected方法中，恢复到正常状态，而在activate方法中执行点击回调。

## 21.3　MenuItem详解

Cocos2d-x提供丰富的MenuItem供开发者使用，这里可以根据其表现和功能大致分为3类，文本菜单项、图片菜单项以及特殊菜单项。下面具体介绍。

### 21.3.1　MenuItemLabel详解

MenuItemLabel继承于MenuItem，通过将一个Label节点添加为子节点来显示文字，提供以下功能。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　设置Label节点——setLabel。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　设置显示的文字内容——setString。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　设置菜单项禁用时的文字颜色（默认为灰色）——setDisabledColor。

MenuItemLabel的create方法如下：

```
//使用一个LabelTTF或LabelBMFont节点，以及回调函数来创建MenuItemLabel
static MenuItemLabel * create(Node*label, const ccMenuCallback& callback);
//使用一个LabelTTF或LabelBMFont节点，以及空的回调函数来创建MenuItemLabel
static MenuItemLabel* create(Node *label);
```

### 21.3.2　MenuItemAtlasFont详解

MenuItemAtlasFont继承于MenuItemLabel，自动使用LabelAtlas来创建MenuItemAtlasFont，create方法如下：

```
//根据字符串value，图片文件charMapFile，单字符的宽和高，起始字符创建一个
LabelAtlas，并以空的回调来创建MenuItemAtlasFont
static MenuItemAtlasFont* create(const std::string& value, const 
std::string& charMapFile, int itemWidth, int itemHeight, char startCharMap);
//根据字符串value，图片文件charMapFile，单字符的宽和高，起始字符创建一个
LabelAtlas，以及指定的回调来创建MenuItemAtlasFont
static MenuItemAtlasFont* create(const std::string& value, const 
std::string& charMapFile, int itemWidth, int itemHeight, char startCharMap, 
const ccMenuCallback& callback);
```

### 21.3.3　MenuItemFont详解

MenuItemFont继承于MenuItemLabel，通过Label::createWithSystemFont创建的Label对象来创建MenuItemFont，提供以下功能：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　设置字体大小——setFontSize。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　设置字体名字——setFontName。

MenuItemFont的create方法如下：

```
//使用字符串和空的回调来创建MenuItemFont
static MenuItemFont * create(const std::string& value = "");
//使用字符串和callback回调来创建MenuItemFont
static MenuItemFont * create(const std::string& value, const ccMenuCallback& 
callback);
```

### 21.3.4　MenuItemSprite详解

MenuItemSprite继承于MenuItem，MenuItemSprite使用Sprite来显示图片，使用3个Sprite对象分别来表示菜单项的正常状态、选中状态以及禁用状态。在状态改变的时候切换Sprite进行显示，MenuItemSprite的create方法如下：

```
//传入正常状态，选中状态、禁用状态的Sprite节点，以及空的回调函数来创建
MenuItemSprite
static MenuItemSprite * create(Node* normalSprite, Node* selectedSprite, 
Node* disabledSprite = nullptr);
//传入正常状态、选中状态的Sprite节点，以及callback回调函数来创建MenuItemSprite
static MenuItemSprite * create(Node* normalSprite, Node* selectedSprite, 
const ccMenuCallback& callback);
//传入正常状态，选中状态以及禁用状态的Sprite节点，以及callback回调函数来创建MenuItemSprite
static MenuItemSprite * create(Node* normalSprite, Node* selectedSprite, 
Node* disabledSprite, const ccMenuCallback& callback);
```

### 21.3.5　MenuItemImage详解

MenuItemImage继承于MenuItemSprite，在此基础上封装了一些简单的create方法来方便使用，MenuItemImage的create方法如下：

```
//创建一个空的MenuItemImage
static MenuItemImage* create();
//传入正常状态、选中状态的图片，以及空的回调函数来创建MenuItemImage，内部根据图片自
动创建Sprite
static MenuItemImage* create(const std::string& normalImage, const 
std::string& selectedImage);
//传入正常状态、选中状态、禁用状态的图片，以及空的回调函数来创建MenuItemImage，内部
根据图片自动创建Sprite
static MenuItemImage* create(const std::string& normalImage, const 
std::string& selectedImage, const std::string& disabledImage);
//传入正常状态、选中状态的图片，以及callback回调函数来创建MenuItemImage，内部根据
图片自动创建Sprite
static MenuItemImage* create(const std::string&normalImage, const std:: 
string&selectedImage, const ccMenuCallback& callback);
//传入正常状态，选中状态、禁用状态的图片，以及callback回调函数来创建MenuItemImage，
内部根据图片自动创建Sprite
static MenuItemImage* create(const std::string&normalImage, const std:: 
string&selectedImage, const std::string&disabledImage, const ccMenuCallback& 
callback);
```

### 21.3.6　MenuItemToggle详解

MenuItemToggle是一个特殊的菜单项，继承于MenuItem，本身没有显示功能，是一个MenuItem的容器，通过其他MenuItem来进行切换显示。MenuItemToggle可以添加很多个菜单项，但只显示一个，每次单击菜单都会切换下一个菜单项来显示，依次循环显示菜单项。当一个菜单项需要在多种状态间切换时，MenuItemToggle是一个不错的选择，其功能如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　添加一个子菜单项——addSubItem。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　获得当前选中的子菜单项（默认为第一个）——getSelectedItem。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　切换到指定的子菜单项显示——setSelectedIndex。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　获取所有的子菜单项——getSubItems。

MenuItemToggle的create方法如下：

```
//创建一个空的MenuItemToggle对象
static MenuItemToggle* create();
//创建只有一个按钮的MenuItemToggle对象
static MenuItemToggle* create(MenuItem *item);
//根据menuItems列表和callback回调来创建MenuItemToggle对象
static MenuItemToggle * createWithCallback(const ccMenuCallback& callback, const Vector<MenuItem*>& menuItems);
//根据callback回调和若干MenuItem对象，来创建MenuItemToggle对象
static MenuItemToggle* createWithCallback(const ccMenuCallback& callback, MenuItem* item, ...)
```

**注意：**MenuItemToggle并不理会添加的是subItems的回调函数，每次单击只回调自己的callback。

## 21.4　实现TabView

在游戏中经常会用到类似TabView控件，例如图21-3所示的控件，多个按钮点击切换，被选中的按钮处于高亮状态，而之前处于高亮状态的按钮取消高亮状态。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160032.jpeg)

图21-3　TabView控件

这样的控件Menu并不支持，要实现一个这样的控件也不难，但是有没有简单一些的方法，用最少的代码来实现该功能呢？只要使用MenuItemImage就可以很简单地实现上述效果，需求如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　当MenuItemImage被点中时，处于高亮状态。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　当MenuItemImage被点中时，将处于高亮状态下的按钮取消高亮。

实现思路如下：

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　在MenuItemImage的点击回调中，调用setNormalImage，将NormalImage设置为被选中的高亮图片。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200707160029.jpeg)　获取处于高亮状态下的MenuItemImage，调用setNormalImage，将NormalImage设置为正常显示的图片。

如果每个Item使用各自的回调，那么可以通过Tag来获取高亮菜单项，将高亮对象的Tag设置为1，取消高亮的对象的Tag设置为设1。如果Tag本身有其他用途，那么可以定义一个成员变量来保存当前高亮的Item对象，并在后续的切换中维护这个变量。