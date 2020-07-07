[TOC]

# 第**35**章　**Objective-C**与**C++**互调

开发iOS程序，难免会涉及一些iOS平台相关的代码需要编写，例如，在接入一些广告、分享、统计等SDK的时候，都涉及编写Objective-C代码，然后在C/C++中调用，Android下需要通过JNI技术来实现Java与C/C++的互调，其中细节繁多，非常复杂，而Objective-C与C/C++的调用则非常简单。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Objective-C基础语法。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Objective-C与C++混编。

### 35.1　Objective-C基础语法

#### 35.1.1　Objective-C的一些特性

首先简单快速地了解一下Objective-C的特性，要快速掌握Objective-C，需要了解Objective-C的message特性、函数调用、变量定义、字符串、id等概念。

##### 1．Objective-C的文件

Objective-C的文件有3种，分别是.h、.m和.mm，.h文件用于声明接口，.m和.mm文件用于实现接口，.m文件中的m是message的缩写，message是oc的主要特性。.m和.mm的区别在于.mm文件中可以直接使用C/C++的语法，而.m只支持Objective-C的语法。

##### 2．Objective-C与C/C++的相同点

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　每行语句都需要以分号 ; 结尾。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　支持C/C++的基础数据类型，如int、char、short、long、float、double、unsigned等，但在Objective-C中void关键仅可以用于修饰函数的返回值和参数（表示无参数）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　支持 //注释和 /* */ 注释。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　变量的命名规则和C/C++一致。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　循环语句、条件语句以及各种运算符的规则和C/C++一致。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Objective-C的self等同于C/C++的this，nil等同于C/C++的NULL。

##### 3．Objective-C的函数

Objective-C的函数与C/C++的函数有较大的区别，主要体现在函数定义和函数调用上，除了可以使用C/C++的格式来定义函数之外，Objective-C的函数定义格式如下。

```
- (返回值类型) 方法名:( 参数类型1 )参数名1
连接参数2:( 参数类型2)参数名2...
连接参数n:( 参数类型n)参数名n
{
    函数体
}
```

上面格式中的方法名、返回值类型、参数类型、参数名很容易理解，无须解释，但连接参数又是什么意思呢？连接参数可以使代码更容易阅读和调用，可以理解为参数的一个注释吧。

调用Objective-C函数的格式如下：

```
[obj dowhat]
```

表示调用obj对象的do what方法，也表示向obj发送do what消息。调用函数传入多个参数时，需要用**Enter键或者逗号**来区分这些参数。

##### 4．Objective-C的@与字符串

Objective-C中的@非常有特色，同时挺让人费解的，@到底是干什么的呢？@主要有以下作用。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　修饰字符串常量，将字符串转换为NSString对象，在Objective-C中，**NSString和char\*不可以混用**，例如@"hello world"。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　格式化输出字符串，%@类似C/C++中printf的%s，如NSLog(@"you say %@",@"hello world"。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　修饰Objective-C的保留关键字，如@interfere、@class、@end等关键字。

##### 5．其他

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在比较BOOL时，不要拿BOOL和YES比较，和NO比较更妥当，YES定义为1，NO定义为0，都是一个字节。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　id表示指向某个对象的指针，意为identifier，表示一种泛型，相当于C语言的void*指针。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　#import关键字等同于确认头文件只被包含一次的#include关键字。

#### 35.1.2　Objective-C的类

接下来看一下如何在Objective-C中编写一个类，首先需要一个头文件，在头文件中声明一个类。

```
@interface Cls: NSObject
{
StructA a;
int b;
}
- (void) setA: (StructA) a;
- (void) Fun;
@end
```

上面代码中使用@interfere声明了一个Cls类（接口），Circle接口继承于NSObject，它有a和b两个数据成员，还有setA和Fun两个方法，在类声明的最后以@end关键字结尾。

接下来在.m或.mm文件中实现，首先用@implementation关键字修饰Cls，表示这是Cls类的实现代码，**在这里可以定义那些头文件中没有声明的方法，相当于私有函数**，但Objective-C没有真正的私有函数，在调用方法的时候Objective-C会隐秘地将自己作为self参数传递进去，相当于C/C++的this指针，只要类存在该方法，那么我们在运行时发送该消息，这个方法还是会执行的。

```
@implementation Cls
- (void) SetA: (StructA) sa
{
    a = sa;
}
- (void) Fun
{
}
@end
```

在定义完类之后，就可以使用了，要实例化（创建）对象，需要发送new消息，代码如下：

```
id c = [Circle new];
[c Fun];
[c SetA: a];
```

关于Objective-C类的一些相关知识补充如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　一般的类都会继承自NSObject，继承NSObject之后可以使用Cocoa，其提供了大量有用的特性。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　可以重写父类的接口，在调用接口时，会自下而上地先从子类开始找，找到就执行，找不到就继续寻找父类的接口。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　通过super关键字可以调用父类，向父类发送消息。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　@class ClassA是前向引用，相当于C++两个互相包含的类，需要在声明之前加多一条前向声明。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在类的成员方法前面用“+”表示这个方法会创建一个表示该类的对象。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Objective-C的对象默认有alloc()、new()、dealloc()方法可用于创建对象。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Objective-C的对象使用了引用计数进行管理，retain和release会更新引用计数，与Cocos2d-x类似，此外使用了NSAutoreleasePool自动释放池来自动管理一些对象，与Cocos2d-x的AutoreleasePool类似。

### 35.2　Objective-C与C++混编

Objective-C调用C++很简单，在Objective-C只要将文件后缀设置为.mm，即可像在.cpp中一样使用C++的东西，当然，还需要包含对应的头文件。

<u>C++调用Objective-C稍微麻烦一些，需要通过.mm在中间封装一层，在.mm的头文件中提供给C++调用的接口，然后在.mm的代码实现中调用Objective-C的代码</u>。例如，在Cocos2d-x中弹出一个iOS的确定菜单，类似于Windows的MessageBox。

首先在MyMessageBox.h头文件中添加一个函数。

```
void ShowMessage(const char* pszTitle , const char* pszMsg );
```

然后在MyMessageBox.mm中实现对应的代码。

```
#include "MyMessageBox.h"
#include <stdarg.h>
#include <stdio.h>
#import <UIKit/UIAlert.h>
void MyMessageBox(const char* pszTitle , const char* pszMsg )
{
    NSString * title = (pszTitle) ? [NSString stringWithUTF8String :
    pszTitle] : nil;
    NSString * msg = (pszMsg) ? [NSString stringWithUTF8String : pszMsg] :
    nil;
    UIAlertView * messageBox = [[UIAlertView alloc] initWithTitle: title
                                                           message: msg
                                                          delegate: nil
                                                cancelButtonTitle: @"OK"
                                                otherButtonTitles: nil];
    [messageBox autorelease];
    [messageBox show];
}
```

在任何希望弹出消息框的Cocos2dx C++代码里，都可以像下面的代码这样调用。

```
#include "MyMessageBox.h"
...
MyMessageBox(“ShowMessage”, “This is a Message”);
...
```

需要注意的是，上面的MyMessageBox()函数中，涉及字符串的转换，需要**把C++的char\*转换成Objective-C认识的类型NSString才可以在Objective-C中调用。**转换的方法就是NSString的stringWithUTF8String()方法，将UTF-8的char*字符串传入会返回对应的NSString对象。