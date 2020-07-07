[TOC]

## 渲染管线流程：

顶点变换（将每个顶点移动到正确的位置）—>

图元组装(将顶点连接成几何图元) —>

光栅化(将图元转换成一系列片段) —>

插值计算(插值计算会为每一个片段计算出颜色、纹理坐标等插值) —>

(插值计算的结果被传入片段着色器，由着色器来决定最终的颜色)—>

测试（着色后输出的片段将经过裁剪、Alpha、模板、深度4个测试，测试通不过则丢弃）

混合（通过测试后的片段会与当前帧缓冲区中的颜色进行混合计算，最后写入帧缓冲区中）



# 第**15**章　使用**Shader**——**GLSL**基础

对于很多初学者而言，Shader是神秘的，因为Shader工作在图形处理器的底层，并且往往涉及不少复杂的图形学算法，要掌握Shader还需要有一定的OpenGL基础，如果没有非常合适的入门资料，那么掌握Shader将会遇到非常多的困难，没有循序渐进地学习，那么碰到的问题将会令人难以寸进，掌握不了的知识，自然会觉得很难，但实际并不是这样的，相信读完本章后，读者会对Shader有一个深刻的理解。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Shader简介。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　图形渲染管线。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　GLSL基础语法。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在OpenGL中使用Shader。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　在Cocos2d-x中使用Shader。



### 15.1　Shader简介

首先简单介绍一下Shader是什么，Shader意为着色器，是图形渲染管线中的几个处理单元，也可以理解为图形渲染管线中的几处会被执行的代码，Shader主要分为**顶点Shader、片段Shader以及几何Shader**这3种。

那么顶点、片段和几何分别是什么呢？<u>**顶点**</u>非常好理解，如绘制一张图片，一般是一个由四个顶点组成的图形。

<u>**片段**</u>指的就是要绘制出来的图元的每一个**像素**，这种叫法一般指在渲染管线的着色阶段的像素，<u>片段和像素是有区别的</u>，例如在重叠的情况下，一个像素上可以渲染多个片段。

<u>**几何**</u>指的是要绘制的**形状**，同样的四个点，可以组成两个三角形、一个四边形或者两条线。

**图形渲染管线**指的是图形渲染的<u>流水线</u>，可以理解为**将图元进行渲染的一系列步骤**，管线有<u>传统的固定管线</u>和现在的<u>可编程管线</u>两种。

在<u>固定管线</u>中无法对渲染的流程进行任何干预，而在<u>可编程管线</u>中，可以通过Shader来控制渲染的流程，可以制作出更加炫酷的效果。

那么如何通过Shader来控制渲染的流程呢？首先需要了解Shader中可以实现怎样的操作，这需要对图形渲染管线有深入的了解。然后需要掌握编写Shader脚本的基础知识，以及如何将Shader脚本应用到游戏中。

在DirectX中，需要使用HLSL(High Level Shader Language)语言来编写Shader脚本，而在OpenGL中，使用GLSL（OpenGL Shading Language）语言来编写Shader脚本，两者语法不同，但功能大致类似。



### 15.2　图形渲染管线

掌握Shader的关键，在于对图形渲染管线的理解，而图形渲染管线中的两个不易理解的概念，一个是对逐顶点和逐片段的理解，另外一个是对插值的理解。本节会讲解Shader要做的是什么，而**GLSL基础语法**和**在OpenGL中使用Shader**这两节内容会讲解应该怎么做。

接下来详细介绍一下图形渲染管线，这个管线描述了顶点、纹理等数据，以及如何渲染到屏幕上的一个流程。OpenGL被设计为客户端与服务端的<u>CS模型</u>，**客户端为应用程序调用的OpenGL接口，运行在CPU上**，向服务端发送各种渲染请求。而**服务端则会通过图形渲染管线执行真正的渲染工作，运行在GPU上**。

在OpenGL应用程序中，会调用OpenGL相关的API，告诉OpenGL想要渲染什么东西，如何渲染，这些数据包括了顶点、顶点连接数据、纹理以及渲染时的相关状态。服务端在接收到这些请求执行渲染时，会交给图形渲染管线来执行，大致可以分为5个步骤，如图15-1所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112659.jpeg)

图15-1　大致的图形渲染管线流程

（1）准备好**<u>顶点数据和顶点连通性数据</u>**。<u>顶点数据</u>包含了**一系列顶点的位置、颜色、纹理坐标、法线等数据**，而<u>顶点连通性数据</u>则是描述顶点之间如何连接组合的数据，例如这边两个顶点是连成一条线，这边三个顶点组成一个三角形。

（2）获取到了顶点数据（包含颜色、位置、法线、纹理坐标等属性），将**每一个**顶点数据都进行了处理，这个阶段处理了顶点的位置变换，顶点光照计算，生成纹理坐标等操作，将处理完的顶点输出到下一步。【顶点处理，由 **顶点着色器** 处理】

（3）根据顶点数据和顶点连通性数据，将点连成线、线连成面。把零零散散的顶点**组装成一个一个的图元，并且将图元进行栅格化**，也就是处理成一个一个的<u>像素格子</u>，并且决定了每个像素格子的位置，这里也称之为<u>片段</u>。【图元装配与栅格（=像素）化，由 **几何处理器** 负责】

被确定的除了片段的位置之外，还可能包含片段的颜色，OpenGL会根据图元各个顶点的颜色，纹理坐标进行<u>插值</u>，得出每个片段的颜色插值。这个阶段会**对不在视口范围中的内容进行裁剪**。<u>背面剔除操作</u>也会在这个阶段执行。

（4）获取到了第三步输出的片段，并对每一个片段进行处理，可以对颜色进行处理，也可以丢弃片段，<u>雾化</u>也是在这个阶段执行的。【片段处理，由 **片段处理器** 负责】

（5）为即将呈现到屏幕上的内容进行最后的处理，对每个片段进行一系列测试：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　ScissorTest裁减测试，激活GL_SCISSOR_TEST时开启，判断是否在设定的裁减窗口中，如是则通过，否则不渲染当前片段。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　AlphaTest Alpha测试，激活GL_ALPHA_TEST时开启，判断当前片段的透明度是否符合glAlphaFunc设定的条件（如透明度大于0.5），如是则通过，否则不渲染当前片段。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　StencilTest模板测试，激活GL_STENCIL_TEST时开启，判断当前片段模板值与模板缓冲区中对应的模板值是否符合glStencilFunc()函数设定的条件，如是则通过，否则不渲染当前片段。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Depth Test深度测试，激活GL_DEPTH_TEST时开启，用于判断片段的前后遮挡关系，判断当前片段的深度值和深度缓冲区中对应的值是否符合glDepthFunc()函数设定的条件，如是则通过，否则不渲染当前片段。

关于以上4种测试，读者可以参考http://blog.csdn.net/crazyjumper/article/details/1968567，其中介绍的非常详细。

最后，如果测试成功的话（失败会被裁减掉），会**根据当前的混合模式将片段更新到帧缓冲区里对应的像素中**，当屏幕刷新时，缓冲区的内容会被渲染到屏幕上。如图15-2所示形象地描述了以上5步骤的流程。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112700.jpeg)

图15-2　形象的图形渲染管线流程

在上面的5个步骤中，步骤（2）由**顶点处理器**负责，顶点处理器可以调用实现的顶点着色器来处理每个顶点的位置变换，**将每个顶点移动到正确的位置**，这是这一阶段最重要的工作，因为我们只能在顶点着色器中设置每个顶点的位置。

此外，还可以预先计算一些变量，将这些变量传给片段着色器，从而提高渲染效率。例如，我们需要知道光源的方向，然后进行归一化的操作，并用于光照计算，假设将其放在片段着色器中执行，那么每处理一个像素都会执行一次该操作，而如果放在顶点着色器中，执行的次数就少了很多。

步骤（4）由**片段处理器**负责，片段处理器可以调用实现的片段着色器来处理每个片段的着色，**赋予每个片段正确的颜色值**，是这一阶段最重要的工作。

步骤（3）由**几何处理器**负责，几何处理器可以调用实现的几何着色器来组装图元，但几何着色器一般不会用到，因为一般不需要实现一套新的图元组装规则，而且顶点着色器和片段着色器也已经足够强大了。接下来详细介绍一下顶点处理器和片段处理器。



#### 15.2.1　顶点处理器

顶点处理器的职责是执行顶点着色器，OpenGL程序会将顶点数据传给顶点着色器，所谓顶点数据就是顶点的位置、颜色、法线、纹理坐标等数据。在OpenGL应用程序中使用glColor3f、glVertex3f等接口。通常会在顶点着色器中执行以下操作：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　使用模型视图矩阵和投影矩阵来对顶点的位置进行变换。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　对法线的变换和归一化操作。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　生成以及变换纹理坐标。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　逐顶点光照计算或为逐片段光照计算预先计算一些数值。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　颜色计算。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　定义插值变量，将插值变量进行插值计算后传递给每个片段。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　访问OpenGL的状态。

顶点着色器不仅仅局限于这些操作，同时也不必执行上述的全部操作，假如程序没有使用光照，则无须执行光照计算相关的操作。需要注意的是，**一旦使用了自定义的顶点Shader，那么顶点处理器的固定功能就会被替换**。如果在顶点着色器中只实现了光照计算，那么就不要指望顶点处理器的固定功能来完成纹理坐标的生成工作。

顶点着色器最重要的职责就是<u>计算位置</u>，所以**顶点着色器至少需要写一个变量gl_Position**，这是一个GLSL内置的位置变量，需要将变换后的顶点坐标赋予该变量。



#### 15.2.2　片段处理器

片段处理器的职责是运行片段着色器，片段着色器会接收到**经过插值计算后的结果**，如顶点坐标、颜色、纹理坐标、法线等。通常会在片段着色器中执行以下操作：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　逐像素计算颜色和纹理坐标。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　应用纹理（可以应用多重纹理）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　雾化计算。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　为逐像素的光照进行法线计算。

如同顶点处理器一样，**只要使用了自定义的片段着色器，所有固定功能将被取代**，所以不能使用片段着色器对片断进行材质计算，同时也不能使用原先的固定功能进行雾化计算。必须在片段着色器中实现需要的全部效果。

片段着色器可以有两种输出，一种是<u>将该片段抛弃</u>，另一种是<u>将片段的最终颜色写入gl_FragColor变量中</u>。

<u>片段着色器并不能访问帧缓冲区，所以混合操作只能在着色阶段之后执行</u>（混合操作可以将片段的颜色与当前帧缓冲区中对应位置的颜色进行混合）。片段着色器可以获取到片段的位置，但不能对位置进行修改。



#### 15.2.3　插值计算

这里**特别强调一下插值计算，因为这个概念非常重要**，看上去也很好理解，但实际上很容易忽略其中的一些细节。

简单回顾一下管线流程：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　首先是顶点变换，将每个顶点移动到正确的位置。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　接着是图元组装，将顶点连接成几何图元。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　然后是光栅化，将图元转换成一系列片段。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　然后是插值计算，插值计算会为每一个片段计算出颜色、纹理坐标等插值。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　插值计算的结果被传入片段着色器，由着色器来决定最终的颜色。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　着色后输出的片段将经过裁剪、Alpha、模板、深度4个测试，测试通不过则丢弃。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　通过测试后的片段会与当前帧缓冲区中的颜色进行混合计算，最后写入帧缓冲区中。

插值计算会根据整个图元的所有顶点的数据进行计算，一个三角形只有三个顶点，但却有很多的片段。如果三个顶点是不同的颜色，如图15-3所示，那么经过插值计算后的每一个片段，都有不同的颜色。纹理也类似，假设四个顶点分别对应纹理图片的四个角，那么中间所有的片段的纹理坐标，也是通过插值计算出来的。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112701.jpeg)

图15-3　顶点颜色不同的三角形

这里需要注意的是，插值的结果是在渲染管线中的插值计算操作帮我们计算出来的，但**需要计算哪些插值，却需要在着色器代码中指定**。

首先来看一下一条简单的线段的插值计算过程，一条线段由两个顶点组成，假设这两个顶点有着不同的颜色值，一黑一白两个颜色，在光栅化阶段中，这条线段生成了5个片段。那么在插值计算阶段，这5个片段接收到的颜色插值分别为(0.0, 0.0, 0.0)、(0.2, 0.2, 0.2) ... (1.0, 1.0, 1.0)。

如果两个顶点绑定了不同的纹理坐标，插值计算同样会为两个顶点中间的所有片段计算其纹理坐标的插值。

<u>那么插值计算阶段，如何知道要计算颜色插值还是纹理坐标插值呢？</u>这些都是由我们的着色器指定的，**当在顶点着色器中定义了一个varying变量，并且将顶点的颜色赋值给它时，那么在片段着色器中，接收到的这个varying变量就是一个经过插值计算的颜色**。

插值计算阶段并不关心计算的插值有何意义，在顶点着色器中所有的varying变量，都会被执行插值计算，每个片段都会接收到经过插值计算后的varying变量。如果将颜色值赋给varying变量，那么得到的就是颜色插值，将纹理坐标赋给varying变量，那么得到的就是纹理坐标插值。<u>顶点A到顶点B之间的所有片段接收到的插值，都是根据A和B的varying变量的值计算得来的。</u>

本节内容可能不容易理解，因为引用了一些15.3节才会介绍的知识，当读完本章内容，对插值计算存在疑问时（为什么这个片段可以正确地对应到纹理的颜色），再回顾本节内容，相信会有豁然开朗的感觉。



### 15.3　GLSL基础语法

接下来学习如何编写Shader脚本，首先需要掌握GLSL的基础语法，这种语法与我们熟悉的C/C++语言颇为相似，顶点着色器和片段着色器都是使用GLSL编写的。下面系统地介绍GLSL的语法。

#### 15.3.1　数据类型和变量



##### 1．基础类型

GLSL支持以下3种基本数据类型：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　float：浮点型，值为浮点数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　int：整型，值为整数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　bool：布尔型，值为true和false。

需要特别注意的是，假设有一个float变量，在着色器中将其赋值为1，在一些显卡上可能会崩溃！因为1是整数不是浮点数，所以需要将1.0赋给浮点型变量。可以这样使用基础变量：

```
int a,b;
float c = 3.0;
bool d = true;
```



##### 2．向量类型

向量是GLSL中常用的数据类型，3种基础数据数据类型分别对应3种向量：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　vec2、vec3、vec4：浮点型向量，可以存储2～4个浮点数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　ivec2、ivec3、ivec4：整型向量，可以存储2～4个整数。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　bvec2、bvec3、bvec4：布尔型向量，可以存储2～4个布尔值。

可以使用括号来初始化向量，例如：

```
vec3 a = vec3(1.0, 0.0, 3.0);
```

可以使用下标来访问向量不同维度的分量，也可以用x、y、z、w字段来访问向量的成员，使用r、g、b、a字段可以访问颜色向量，使用s、t、p、q字段可以访问纹理坐标向量。如果要访问向量的x、y、z3个字段，还可以连续使用字段。向量的操作还是非常灵活的，下面的代码演示了一些向量的操作。

```
vec2 a = vec2(1.0, 2.0);
vec3 b = vec3(3.0, 4.0, 5.0);
vec3 color = b.rgb;            //使用rgb来操作b的3个分量
float c = a[0];                //使用下标的方式来访问a的第一个分量
vec3 d = vec3(a, c);           //直接使用一个vec2变量和一个float变量初始化vec3
float e = float(d);            //d的x分量被赋给了e，y、z分量被丢弃
```



##### 3．矩阵类型

GLSL提供了2×2、3×3、4×4共3种矩阵类型，它们的类型名分别是mat2、mat3、mat4，矩阵的初始化和使用与向量类似。

此外GLSL还支持2～4×2～4的任意矩阵，如mat3×2、mat4×3等。



##### 4．采样器

GLSL提供了一组采样器，这是一种用于访问纹理的特殊类型，在读取纹理值时会用到，主要有下面几种采样器。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sampler1D：一维纹理采样器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sampler2D：二维纹理采样器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sampler3D：三维纹理采样器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　samplerCube：立方体映射纹理采样器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sampler1DShadow：一维阴影映射采样器。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　sampler2DShadow：二维阴影映射采样器。



##### 5．数组和结构体

在GLSL中，可以像C语言一样声明和访问数组，但是不能在声明时直接初始化数组，数组的下标从0开始，如下所示。

```
int a[3];
a[0] = 1;
```

GLSL还可以定义结构体，结构体的定义和初始化如下。

```
struct myst
{
    vec3 dir;
vec3 color;
};
myst st1;
myst st2 = myst(vec3(0.0, 0.0, 0.0), vec3(1.0, 1.0, 0.0));
st2.dir = vec3(0.0, 1.0, 0.0);
```



#### 15.3.2　操作符

GLSL的操作符和C/C++语言非常类似，具体如表15-1所示。

表15-1　GLSL操作符

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112702.jpeg)

由于GLSL中没有指针的概念，所以并不需要取值操作符&、解引用操作符*和指针操作符->。



#### 15.3.3　变量修饰符、统一变量和属性变量

变量修饰符可以放在变量类型之前，用于修饰变量，GLSL常用的有以下变量修饰符。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　const：编译时期的常量，不可被修改。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　attribute：属性变量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　uniform：统一变量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　varying：易变变量。

统一变量和属性变量都是**只读的全局变量**（不能在函数中定义），都是由OpenGL应用程序设置的变量。

属性变量是**针对每个顶点的数据**，所以只可以在顶点着色器中定义，一般会将顶点的颜色、位置、法线等顶点数据作为属性变量传递给顶点着色器。

统一变量是**针对整个图元的数据**，既可以在顶点着色器中使用，也可以在片段着色器中使用。可以将逝去的时间这样的变量设置到统一变量中。通过统一值传递应用程序中的变量给着色器，是实现各种效果时经常需要用到的手段。



#### 15.3.4　易变变量

varying易变变量是着色器中最神奇的变量，也最不容易理解。易变变量主要用于存储插值数据，需要**同时在顶点着色器和片段着色器中定义同一个易变变量**，在顶点着色器中写入易变变量的值，然后在片段着色器中获取易变变量的值（在片段着色器中，易变变量是只读的）。

易变变量最神奇的地方在于，当在顶点A和顶点B中写入了0和1时，在A和B中间的这个片段，得到的值会是0.5。在顶点着色器中写入的易变变量，到了片段着色器中，有可能变成其他的值。那么规律是什么呢？举一个简单的例子来说明。

假设现在有A和B两个顶点，它们连成了一条线，这条线被栅格化为10个片段。顶点着色器中定义了一个易变变量V，在处理顶点A和B时，将V分别赋值为1和0。

到了插值计算阶段，会对A和B两个顶点进行插值计算，这时候会获取它们的易变变量V（所有的易变变量都会在这时进行插值计算），插值算法会计算出每个片段的插值V（实际的插值算法更复杂），具体如下面的伪代码所示（下面这段伪代码只是为了帮助理解）。

```
插值 = abs(A.V - B.V) /片段数
AV = A.V
for (int i = 0; i <片段数; ++i)
{
    //在这里计算出了插值的结果，然后赋值到每个片段的易变变量V中
    片段[i].V = AV + i *插值
}
```

除了自定义的插值变量外，GLSL还内置了一些插值变量。GLSL的内置变量都是以gl_开头的，例如gl_Position和gl_FragColor。

读者可以参考http://my.oschina.net/sweetdark/blog/208024#OSC_h4_4这篇文章，其中总结了OpenGL超级宝典中列出的GLSL的各种内置变量。



#### 15.3.5　语句与函数

GLSL支持C/C++语言中的if-else、for、while、do-while语句，同时支持break和continue两个关键字。在片段着色器中，还可以使用discard语句来丢弃当前片段。

着色器是由函数组成的，**每个着色器都需要有一个main()函数，和C/C++语言一样，这是着色器程序的入口**。

此外用户还可以自定义函数，函数的定义与C/C++语言一样，主要由返回值、函数名、参数列表和函数体4部分组成。函数如果没有返回值可以将返回值的类型定义为void，返回值可以是任意类型，但不能是数组。

函数的参数有3种修饰符，分别是in、out、inout，in表示输入，out表示输出，inout表示既输入又输出。在没有指定修饰符的情况下默认为in。

函数允许重载，即函数名相同、参数列表不同的多个同名函数。下面是一个简单的函数示例。

```
bool areyouhappy(in float money)
{
    if(money < 10000000.0)
        return false;
    else
        return true;
}
```



#### 15.3.6　Shader简单示例

以下是两个简单的颜色Shader脚本，分别是一个顶点Shader和片段Shader的脚本，可以用in和out这两个修饰符来替换varying和attribute修饰符，uniform并不能使用in和out来修饰。

在顶点Shader中，可以用in表示从程序中输入的attribute变量，out表示要输出给片段Shader的varying变量。

在片段Shader中，可以用in表示顶点Shader输入的varying变量，out表示片段着色器最后的输出，一般是一个颜色。

```c
//顶点Shader
//输入顶点和颜色，将顶点位置直接设置到gl_Position变量中
//输出颜色插值到vVaryingColor中
attribute vec4 vVertex;
attribute vec4 vColor;
varying vec4 vVaryingColor;
void main()
{
    vVaryingColor = vColor;
    gl_Position = vVertex;
}

//片段Shader
//拿到经过插值后的vVaryingColor，将它设置到最终输出的gl_FragColor变量中
varying vec4 vVaryingColor;
void main()
{
    gl_FragColor = vVaryingColor;
}
```



### 15.4　在OpenGL中使用Shader

学习了GLSL的基础语法之后，就可以开始编写着色器了，在编写完着色器之后，需要将着色器应用到程序中使其生效，这需要调用一些OpenGL的API。本节要介绍的是如何在Cocos2d-x中使用Shader的预备知识。主要包括在OpenGL中使用Shader的流程和相关API，以及如何设置Shader脚本中的统一变量和属性变量。

对于没有OpenGL基础的同学，如果想要使用OpenGL来编写一个完整的示例，本节的内容是不够的，因为还需要掌握一些OpenGL的绘图方法，这里并不打算介绍额外的内容，如果要系统地学习OpenGL，读者可查阅《OpenGL超级宝典》一书。

#### 15.4.1　在OpenGL中创建Shader

在OpenGL中，存在Program和Shader两个概念，Program相当于当前渲染管线所使用的程序，是Shader的容器，可以挂载多个Shader。

而<u>每个Shader相当于一个C模块</u>，首先需要对Shader脚本进行编译，然后将编译好的Shader挂载到Program，在OpenGL的渲染中使用Program来使Shader生效，整个流程如图15-4所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112703.jpeg)

图15-4　使Shader生效的流程

可以将整个流程划分为创建Shader和创建Program两个子流程，创建Shader的流程如下。

（1）调用glCreateShader()方法创建一个Shader对象。

（2）调用glShaderSource()方法加载Shader脚本源码。

（3）调用glCompileShader()方法编译Shader脚本。

**glCompileShader**会调用显卡内置的HLSL编译器来编译Shader脚本，<u>当程序启动时，所有的Shader脚本都需要被编译才能使用</u>，虽然可以使用第三方的工具预先将Shader脚本编译为二进制文件，但因为OpenGL的标准只规定了HLSL的语法，对HLSL编译后的二进制文件格式并无规定，所以一次编译并不能在所有的设备上运行（正如同一份C/C++源码在Linux和Windows上编译后是不同的二进制文件格式，不能相互兼容）。

完成上述步骤后，就可以使用这个Shader了，创建好的Shader需要被挂载到Program中，创建Program的流程如下。

（1）调用glCreateProgram()方法创建一个Program对象。

（2）调用glAttachShader()方法将创建好的Shader进行挂载。

（3）调用glLinkProgram()方法执行链接操作。

（4）在需要使用Shader时，调用glUseProgram()方法应用当前Shader。

Shader的编译和链接与C/C++语言中的编译和链接非常相似，Shader的编译操作相当于将C/C++语言的源码编译成.o文件，而Program的链接操作则相当于将这些.o文件链接成一个可执行的程序。

我们可以创建多个Program，但**同时只可以激活一个Program**。创建完Program之后，只需要在调用OpenGL的绘制方法前使用glUseProgram即可应用当前的Shader。

如果激活的Program没有挂载片段Shader，那么片段Shader执行的结果是未定义的。如果激活一个不存在的Shader，那么所有Shader的执行结果都是未定义的。

接下来简单介绍一下流程中使用到的这些API。

```c
//创建一个Shader对象，并返回引用该对象的句柄——一个非0整数
//参数shaderType为Shader的类型，类型一般是GL_VERTEX_SHADER、
GL_FRAGMENT_SHADER或GL_GEOMETRY_SHADER，
GLuint glCreateShader(GLenum shaderType);

//加载源码到Shader中，这个操作会将Shader脚本代码复制到Shader对象中，多次调用时，上一次存储的脚本会被替换
//参数shader表示Shader对象的句柄，由glCreateShader()方法返回
//参数count表示string数组的长度
//参数string为Shader源码，这是一个字符串数组
//参数length为一个int数组，对应string参数这个字符串数组中每个字符串的长度，当这些字符串都是以’/0’结尾时，可以将这个参数置为NULL
void glShaderSource(GLuint shader, int count, const char **string, int
*length);

//编译存储于Shader中的代码，参数shader表示Shader对象的句柄
void glCompileShader(GLuint shader);

//创建一个Program对象，并返回引用该对象的句柄——一个非0整数
GLuint glCreateProgram();

//将一个已经编译好的Shader挂载到Program中
//参数program和shader分别表示Program对象和Shader对象的句柄
//一个Shader可以同时被挂载到多个Program对象中，但同一种类型的Shader，Program只能挂载一个
void glAttachShader(GLuint program, GLuint shader);

//对指定的Program对象执行链接操作，Program在链接成功之后才可以执行
//链接操作会将Program中的所有Uniform变量初始化为0
void glLinkProgram(GLuint program);

//激活指定的Program，接下来的绘制会使用指定的Program进行渲染
void glUseProgram(GLuint program);
```

关于更多API的信息，读者可以参考OpenGL官网的API文档，在这里可以找到最准确的API描述，网址为https://www.opengl.org/sdk/docs/man4/。

下面这段代码演示了上面这些接口的使用，useShader()函数接收两个参数，分别是顶点着色器和片段着色器的源码。

```c
void useShader(const char* vs, const char* fs)
{
    v = glCreateShader(GL_VERTEX_SHADER);
    f = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(v, 1, &vs, NULL);
    glShaderSource(f, 1, &fs, NULL);
    glCompileShader(v);
    glCompileShader(f);
    p = glCreateProgram();
    glAttachShader(p, v);
    glAttachShader(p, f);
    glLinkProgram(p);
    glUseProgram(p);
}
```



#### 15.4.2　属性变量

在Shader中，属性变量和统一变量需要由应用程序设置，Attribute属性变量用于传递顶点信息，而Uniform统一变量则用于传递用户自定义的变量。这两种变量在Shader中会被定义为全局变量，要在OpenGL中操作设置这两种变量，需要先获取它们的位置（相当于这个变量的地址），然后调用OpenGL相关的设置变量接口，为变量赋值。

##### 1．设置属性变量的接口

属性变量需要为每个顶点进行设置，属性变量可以在任何时刻更新，在顶点着色器中属性变量是只读的。因为属性变量包含的是顶点数据，所以在片断着色器中不能直接应用。首先需要获得变量在内存中的位置，这个信息只有在Program链接之后才可以获得。注意，对于某些驱动程序，在获得位置之前还必须调用glUseProgram()方法激活Program，以下接口可以获取指定属性的位置。

```c
//参数program为要操作的Program对象
//参数name为要获取的属性变量名
GLint glGetAttribLocation(GLuint program, char *name);
```

通过glGetAttribLocation()方法获取到属性的位置后，就可以**使用glVertexAttrib系列方法来为这个属性赋值**。glVertexAttrib系列函数会对应各种不同的属性变量类型，如浮点型、向量以及矩阵。

glVertexAttrib系列函数有一些共同的特性，如第一个参数固定为属性的位置（通过glGetAttribLocation()方法获得），在glVertexAttrib后首先会接一个数字，该数字表示参数的数量或数组的长度，数字的范围一般为1～4。

在数字之后会接一个类型字符，f为GLfloat浮点数、s为GLshort短整型、d为GLdouble浮点型，i为GLint整型，ui为GLuint无符号整型。类型字符并没有用b来表示布尔型，可以用操作整型的接口传入0和1来表示布尔。

如果在类型字符后面接一个v，函数后面的参数会是对应类型的数组，数字的含义表示数组的长度，没有接v时，函数后面会有对应数量的参数。具体如下。

```c
//设置1~4个参数的float类型数值
void glVertexAttrib1f(GLint location, GLfloat v0);
void glVertexAttrib2f(GLint location, GLfloat v0, GLfloat v1);
void glVertexAttrib3f(GLint location, GLfloat v0, GLfloat v1,GLfloat v2);
void glVertexAttrib4f(GLint location, GLfloat v0, GLfloat v1,GLfloat v2,
GLfloat v3);

//设置长度为1~4的float类型数组
GLint glVertexAttrib1fv(GLint location, GLfloat *v);
GLint glVertexAttrib2fv(GLint location, GLfloat *v);
GLint glVertexAttrib3fv(GLint location, GLfloat *v);
GLint glVertexAttrib4fv(GLint location, GLfloat *v);
```

上面的函数参数用数组形式或是分别指定多个参数的形式并没有太大区别，它们的关系就如OpenGL中的glColor3f()函数和glColor3fv()函数一样。



##### 2．在渲染时设置属性变量

在Program链接并激活之后可以获取属性变量的位置并在渲染时为其赋值，这里假设**在Shader脚本中定义了一个名为myattribute的属性变量**，首先使用glGetAttribLocation()函数从Program中获取该变量的位置：

```c
GLint local = glGetAttribLocation(p,"myattribute");
```

然后在执行渲染时可以为shader中的属性变量赋值，这里介绍两种为属性变量赋值的情况，第一种是当在glBegin()函数和glEnd()函数的中间，在使用glVertex系列函数生成顶点前，先调用glVertexAttrib系列函数进行赋值，接下来生成的顶点会绑定前面设置的属性变量。

```
glBegin(GL_TRIANGLE_STRIP);
    glVertexAttrib1f(local, 1.0f);
    glVertex2f(0.0f, 0.0f);
    glVertexAttrib1f(local, 2.0f);
    glVertex2f(0.0f, 1.0f);
    glVertexAttrib1f(local, 3.0f);
    glVertex2f(1.0f, 1.0f);
    glVertexAttrib1f(local, 4.0f);
    glVertex2f(1.0f, 0.0f);
glEnd();
```

第二种情况是当使用顶点数组时，批量为顶点数组中的每个顶点设置属性变量。这种情况比较麻烦，需要用到属性变量数组，首先需要调用glEnableVertexAttribArray()方法为指定位置的属性变量开启设置属性变量数组的功能。

```c
void glEnableVertexAttribArray(GLint local);
```

开启了这个功能之后，需要调用glVertexAttribPointer()方法，将属性变量的值批量传入，属性变量的数组和顶点数组是一一对应的。

```c
//参数local，属性变量的位置
//参数size，属性变量的分量数量，必须为1~4，如1为float、2~3为vec2~3
//参数type，属性类型，如GL_FLOAT
//参数normalized，是否对传入的值执行归一化操作
//参数stride，顶点数组中，两个顶点之间的步幅，0表示连续的顶点
//参数pointer，属性变量列表指针，与顶点数组中的顶点一一对应
void glVertexAttribPointer(GLint local, GLint size, GLenum type, GLboolean
normalized, GLsizei stride, const void *pointer);
```

下面这段代码演示了在使用顶点数组进行渲染时，为顶点数组中的每一个顶点绑定属性变量。调用glVertexAttribPointer()方法设置完属性信息之后，当调用glDrawArrays()、glDrawElements()等方法渲染时，属性变量和顶点才会真正进行绑定。

```c
//定义4个顶点
float vertices[8] = {
    0.0f, 0.0f,
    0.0f, 1.0f,
    1.0f, 1.0f,
    1.0f, 0.0f };
float myattributes[4] = {1.0f, 2.0f, 3.0f, 4.0f };
//获取一个已经成功链接的Program中的myattribute属性变量
GLint local = glGetAttribLocation(p, "myattribute");
//使用顶点数组
glEnableClientState(GL_VERTEX_ARRAY);
//启用顶点属性变量数组——必须先开启才能使用glVertexAttribPointer
glEnableVertexAttribArray(local);
//设置顶点数组
glVertexPointer(2, GL_FLOAT, 0, vertices);
//设置顶点属性数组
glVertexAttribPointer(local ,1, GL_FLOAT, GL_FALSE, 0, myattributes);
```



#### 15.4.3　统一变量

属性变量相当于每个顶点的私有只读变量，而Uniform统一变量则相当于整个Program的全局只读变量。统一变量的设置与属性变量一样，都是先获取变量的位置，然后调用相关的接口进行设置。统一变量在一个图元的绘制过程中是不会改变的，所以不能在glBegin()和glEnd()函数中间设置统一变量的值。

获取统一变量位置和设置统一变量值的接口，与属性变量的相关接口基本一致，只是将方法名中的Attrib或VertexAttrib替换成Uniform。例如，从glGetAttribLocation()方法变成glGetUniformLocation()方法，以及从glVertexAttrib1f()方法变成glUniform1f()方法。当在Shader中声明了一个Uniform，并使用了这个Uniform，调用glGetUniformLocation()方法时会返回它的位置，但假设声明了但却没有使用这个Uniform，那么glGetUniformLocation()方法会返回-1。

当需要将纹理作为Uniform传给着色器时，可以使用glUniform1i()和glUniform1iv()函数，将纹理的句柄设置进去。

统一变量还有一组针对矩阵类型的设置函数，它们的函数参数列表是相同的，返回值为void，只是函数名不同，例如glUniformMatrix2fv()函数表示2×2的矩阵，而glUniformMatrix2x3fv()函数表示2×3的矩阵。函数的参数列表如下。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　GLint location：统一变量的位置。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　GLsizei count：矩阵的数量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　GLboolean transpose：是否要将矩阵进行转置（这是矩阵操作的一个术语）。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　const GLfloat *value：矩阵的数值数组。

Uniform所支持的矩阵操作函数包含以下函数。

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix2fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix2x3fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix2x4fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix3fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix3x2fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix3x4fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix4fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix4x2fv()函数；
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　glUniformMatrix4x3fv()函数。

需要注意的是，使用这些函数进行设置之后，变量的值将一直保持，直到Program再次链接或我们手动设置了新的值，当Program重新链接时，所有统一变量的值都会被重置为0。

Uniform统一变量的设置比属性变量要轻松得多，因为不需要想办法绑定到每个顶点上，只需要在渲染之前进行设置就可以了。



#### 15.4.4　错误处理

Shader在编译或链接的时候一般比较容易出现错误，而编译和链接方法的返回值都是void，那么如何知道方法的调用是否成功呢？这就需要使用glGetShaderiv()和glGetProgramiv()这两个函数，来获取Shader和Program的状态，这两个函数的原型是一样的，都是传入指定的对象，以及要获取的状态枚举，并传入一个GLint指针，接收状态的值。

```
//查询GL_COMPILE_STATUS可以得到编译的结果，GL_TRUE表示成功，GL_FALSE表示失败
void glGetShaderiv(GLuint shader, GLenum pname, GLint *params);
//查询GL_LINK_STATUS可以得到链接的结果，GL_TRUE表示成功，GL_FALSE表示失败
void glGetProgramiv(GLuint program, GLenum pname, GLint *params);
```

如果发生了错误，错误日志会被保存到InfoLog中，可以调用glGetShaderInfoLog()和glGetProgramInfoLog()方法从中查询错误的相关信息。这个日志保存了最后一次操作的信息，如编译时的警告、错误，连接时发生的各种问题。但由于OpenGL没有对InfoLog制定规范，所以不同的驱动程序或硬件可能产生不同的日志信息，但这并不影响根据日志发现问题。

我们需要调用这两个方法，传入一个缓冲区用于接收日志信息，这两个方法的函数原型如下，第一个参数为Shader或Program的句柄，maxLength参数表示infoLog缓冲区的长度，length参数指针会输出实际复制到infoLog中的字节数，infoLog参数为用于接收日志信息字符串。

```c
void glGetShaderInfoLog(GLuint shader, GLsizei maxLength, GLsizei *length,
GLchar *infoLog);
void glGetProgramInfoLog(GLuint program, GLsizei maxLength, GLsizei *length,
GLchar *infoLog);
```

那么，InfoLog日志的长度是多少呢？需要分配多大的缓冲区来接收日志呢？使用glGetShaderiv()和glGetProgramiv()这两个函数，传入GL_INFO_LOG_LENGTH类型，可以获得日志的长度。

下面的函数演示了如何获取Shader的日志并将日志用printf()函数打印出来，Program的日志打印也可以参考此方法。

```c
void printShaderLog(GLuint shader)
{
    GLint shaderState;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &shaderState);
    if(shaderState == GL_TRUE)
    {
        return;
    }
   GLsizei bufferSize = 0;
   glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &bufferSize);
   if(bufferSize > 0)
   {
       GLchar* buffer = new char[bufferSize];
       glGetShaderInfoLog(shader, bufferSize, NULL, buffer);
       printf("%s", buffer);
       delete[] buffer;
   }
}
```



#### 15.4.5　清理工作

在前面调用glCreateShader()和glCreateProgram()方法后，不需要再使用的时候，需要使用glDeleteShader()方法和glDeleteProgram()进行释放。

当一个Shader被挂载到Program中时，glDeleteShader()方法是无法释放这个Shader的，只会将这个Shader标记为已删除，还需要将使用glDetachShader()方法将Shader从Program中卸载。以下是这3个函数的原型：

```c
void glDetachShader(GLuint program, GLuint shader);
void glDeleteShader(GLuint shader);
void glDeleteProgram(GLuint program);
```

当一个Program正在被使用时，glDeleteProgram()方法是无法释放这个Program的，只会将这个Program标记为已删除，当Program不再被使用时，Program才会被释放。当Program真正被释放时，所有挂载在上面的Shader会被自动卸载。



### 15.5　在Cocos2d-x中使用Shader

接下来介绍如何在Cocos2d-x中使用Shader，在Cocos2d-x中可以通过调用节点的setGLProgram()或setGLProgramState()方法来为一个显示对象设置Shader，在节点被渲染时，Cocos2d-x会使用设置的Shader进行渲染。

#### 15.5.1　Cocos2d-x的Shader架构

在介绍具体的使用方法之前，先了解一下Cocos2d-x的Shader架构。Cocos2d-x的Shader架构如图15-5所示，由GLProgram、GLProgramState、GLProgramCache、GLProgramStateCache组成。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112704.jpeg)

GLProgram是Cocos2d-x对Program的封装，提供了Shader的编译链接和应用等接口，以及Uniform和Attribute的设置，可以将GLProgram看作一个静态的对象，一个GLProgram对象可以被多个节点复用。

GLProgramState是Cocos2d-x对GLProgram的封装，提供了更加便捷的操作接口，可以将GLProgramState看作一个动态的对象，GLProgramState内部记录了Uniform和Attribute，如果说GLProgram是一个类，那么GLProgramState就是这个类的对象。**GLProgram对象只需要一个即可，但GLProgramState对象可以有多个**。

一般情况下，GLProgramState也是多个节点共用一个，例如，所有的Sprite默认都是共用同一个GLProgramState，因为它们所有的Uniform值都是一致的。**当需要有不同的Uniform值时，就需要为每个节点绑定一个GLProgramState**。

例如对两张图片应用一个灰化的Shader，该Shader有一个Uniform用于设置灰度，当期望为这两个Sprite设置不同的灰度时，就需要为它们设置两个不同的GLProgramState。

Cocos2d-x提供了GLProgramStateCache和GLProgramCache用于缓存GLProgramState和GLProgram。在GLProgram.h中定义了Cocos2d-x中所有内置Shader的名字，**在GLProgramCache的loadDefaultGLPrograms()方法中，将所有内置的Shader都创建了出来并缓存**。



#### 15.5.2　Cocos2d-x内置Shader规则

为Cocos2d-x编写的Shader脚本与一般的Shader脚本有些不同，因为在GLProgram的compileShader()方法中，会执行下面这段代码，这段代码会在Shader脚本的前面加入一系列Uniform变量。

<u>在添加Uniform之前，会根据当前的平台以及Shader的类型，在脚本的头部使用precision关键字来设定数值的精度</u>。总共有低精度lowp、中精度mediump、高精度highp可以选择，可以为浮点数或整数指定精度，精度越高效果越好，但消耗越大。另外，顶点着色器支持的精度要比片段着色器高，片段着色器的highp精度支持对于显卡而言，是可选的。

```c++
    const GLchar *sources[] = {
#if CC_TARGET_PLATFORM == CC_PLATFORM_WINRT
        (type == GL_VERTEX_SHADER ? "precision mediump float;\n precision
         mediump int;\n" : "precision mediump float;\n precision mediump
         int;\n"),
#elif (CC_TARGET_PLATFORM != CC_PLATFORM_WIN32 && CC_TARGET_PLATFORM !=
CC_PLATFORM_LINUX && CC_TARGET_PLATFORM != CC_PLATFORM_MAC)
        (type == GL_VERTEX_SHADER ? "precision highp float;\n precision highp
        int;\n" : "precision mediump float;\n precision mediump int;\n"),
#endif
        "uniform mat4 CC_PMatrix;\n"
        "uniform mat4 CC_MVMatrix;\n"
        "uniform mat4 CC_MVPMatrix;\n"
        "uniform mat3 CC_NormalMatrix;\n"
        "uniform vec4 CC_Time;\n"
        "uniform vec4 CC_SinTime;\n"
        "uniform vec4 CC_CosTime;\n"
        "uniform vec4 CC_Random01;\n"
        "uniform sampler2D CC_Texture0;\n"
    "uniform sampler2D CC_Texture1;\n"
    "uniform sampler2D CC_Texture2;\n"
    "uniform sampler2D CC_Texture3;\n"
    "//CC INCLUDES END\n\n",
    source,
};
```

上面的代码定义了很多CC_开头的Uniform，有矩阵、纹理等参数，这些是Cocos2d-x内置的Uniform，在Shader中可以直接使用。

<u>在初始化一个GLProgram时（编译成功后），Cocos2d-x会自动根据Shader是否使用了相应的Uniform变量来初始化_flags成员变量相对应的属性</u>。例如，当使用到了随机数，那么_flags变量的useRandom属性就会被设置为true。**在编译Shader时，GLSL的编译器会自动将声明了但实际没有使用到的Uniform移除**，所以如果在Shader脚本中没有使用CC_Random01这个Uniform值，那么用glGetUniformLocation()方法来获取CC_Random01的位置会返回-1，这时不需要对它进行赋值。

```c++
void GLProgram::updateUniforms()
{    _builtInUniforms[UNIFORM_AMBIENT_COLOR] = glGetUniformLocation (_program,
    UNIFORM_NAME_AMBIENT_COLOR);
    _builtInUniforms[UNIFORM_P_MATRIX] = glGetUniformLocation(_program,
    UNIFORM_NAME_P_MATRIX);
    _builtInUniforms[UNIFORM_MV_MATRIX] = glGetUniformLocation(_program,
    UNIFORM_NAME_MV_MATRIX);
    _builtInUniforms[UNIFORM_MVP_MATRIX] = glGetUniformLocation(_program,
    UNIFORM_NAME_MVP_MATRIX);
    _builtInUniforms[UNIFORM_NORMAL_MATRIX] = glGetUniformLocation (_program,
    UNIFORM_NAME_NORMAL_MATRIX);
    _builtInUniforms[UNIFORM_TIME]=glGetUniformLocation(_program,UNIFORM_
   NAME_TIME);
    _builtInUniforms[UNIFORM_SIN_TIME]=glGetUniformLocation(_program,UNIFORM_
    NAME_SIN_TIME);
    _builtInUniforms[UNIFORM_COS_TIME]= glGetUniformLocation(_program, UNIFORM_
    NAME_COS_TIME);
    _builtInUniforms[UNIFORM_RANDOM01] = glGetUniformLocation(_program,
    UNIFORM_NAME_RANDOM01);
    _builtInUniforms[UNIFORM_SAMPLER0] = glGetUniformLocation(_program,
    UNIFORM_NAME_SAMPLER0);
    _builtInUniforms[UNIFORM_SAMPLER1] = glGetUniformLocation(_program,
    UNIFORM_NAME_SAMPLER1);
    _builtInUniforms[UNIFORM_SAMPLER2] = glGetUniformLocation(_program,
    UNIFORM_NAME_SAMPLER2);
    _builtInUniforms[UNIFORM_SAMPLER3] = glGetUniformLocation(_program,
    UNIFORM_NAME_SAMPLER3);
    _flags.usesP = _builtInUniforms[UNIFORM_P_MATRIX] != -1;
    _flags.usesMV = _builtInUniforms[UNIFORM_MV_MATRIX] != -1;
    _flags.usesMVP = _builtInUniforms[UNIFORM_MVP_MATRIX] != -1;
    _flags.usesNormal = _builtInUniforms[UNIFORM_NORMAL_MATRIX] != -1;
    _flags.usesTime = (
                       _builtInUniforms[UNIFORM_TIME] != -1 ||
                       _builtInUniforms[UNIFORM_SIN_TIME] != -1 ||
                       _builtInUniforms[UNIFORM_COS_TIME] != -1
                       );
   _flags.usesRandom = _builtInUniforms[UNIFORM_RANDOM01] != -1;
   this->use();
   //Since sample most probably won't change, set it to 0,1,2,3 now.
   if(_builtInUniforms[UNIFORM_SAMPLER0] != -1)
      setUniformLocationWith1i(_builtInUniforms[UNIFORM_SAMPLER0], 0);
   if(_builtInUniforms[UNIFORM_SAMPLER1] != -1)
       setUniformLocationWith1i(_builtInUniforms[UNIFORM_SAMPLER1], 1);
   if(_builtInUniforms[UNIFORM_SAMPLER2] != -1)
       setUniformLocationWith1i(_builtInUniforms[UNIFORM_SAMPLER2], 2);
   if(_builtInUniforms[UNIFORM_SAMPLER3] != -1)
       setUniformLocationWith1i(_builtInUniforms[UNIFORM_SAMPLER3], 3);
}
```

那么这些Uniform会在什么时候被设置呢？它们的值又会是多少呢？如果是纹理Uniform，在一开始就会被设置为固定的值，在这里可以同时使用4个纹理，在渲染的时候，Cocos2d-x会将纹理绑定到指定的纹理采样器中，我们在Shader脚本中可以访问这些采样器，而采样器对应的纹理，由Cocos2d-x调用OpenGL的glBindTexture()等方法绑定，并不需要设置这个Uniform的值。

而其他的如矩阵、时间、法线、随机数等Uniform，是在运行的时候动态设置的，这些属性是实时变化的，在Cocos2d-x程序运行时，Cocos2d-x会调用setUniformsForBuiltins()函数自动设置它们（每进行一次渲染时），代码如下。

可以看到下面的代码是根据_flags对象中的一些变量来判断是否需要设置Uniform，这样可以提高效率，否则当不需要使用CC_Time等Uniform时，在每次渲染时调用sinf()和cosf()函数来计算时间再设置到CC_Time等Uniform中，效率会很低下。

```c++
void GLProgram::setUniformsForBuiltins(const Mat4 &matrixMV)
{
    auto& matrixP = _director->getMatrix(MATRIX_STACK_TYPE::MATRIX_STACK_
    PROJECTION);
    if(_flags.usesP)
        setUniformLocationWithMatrix4fv(_builtInUniforms[UNIFORM_P_MATRIX],
        matrixP.m, 1);
    if(_flags.usesMV)
        setUniformLocationWithMatrix4fv(_builtInUniforms[UNIFORM_MV_MATRIX],
        matrixMV.m, 1);
    if(_flags.usesMVP) {
        Mat4 matrixMVP = matrixP * matrixMV;
        setUniformLocationWithMatrix4fv(_builtInUniforms[UNIFORM_MVP_MATRIX],
        matrixMVP.m, 1);
    }
    if (_flags.usesNormal)
    {
        Mat4 mvInverse = matrixMV;
        mvInverse.m[12] = mvInverse.m[13] = mvInverse.m[14] = 0.0f;
        mvInverse.inverse();
        mvInverse.transpose();
        GLfloat normalMat[9];
        normalMat[0] = mvInverse.m[0];normalMat[1] = mvInverse.m[1];normal
        Mat[2] = mvInverse.m[2];
        normalMat[3] = mvInverse.m[4];normalMat[4] = mvInverse.m[5];normal
       Mat[5] = mvInverse.m[6];
       normalMat[6] = mvInverse.m[8];normalMat[7] = mvInverse.m[9];normal
       Mat[8] = mvInverse.m[10];
       setUniformLocationWithMatrix3fv(_builtInUniforms[UNIFORM_NORMAL_MATRIX],
       normalMat, 1);
   }
   if(_flags.usesTime) {
       //This doesn't give the most accurate global time value.
       //Cocos2D doesn't store a high precision time value, so this will
       have to do.
       //Getting Mach time per frame per shader using time could be extremely
       expensive.
       float time = _director->getTotalFrames() * _director->getAnimation
       Interval();
       setUniformLocationWith4f(_builtInUniforms[GLProgram::UNIFORM_TIME],
       time/10.0, time, time*2, time*4);
       setUniformLocationWith4f(_builtInUniforms[GLProgram::UNIFORM_SIN_TIME],
       time/8.0, time/4.0, time/2.0, sinf(time));
       setUniformLocationWith4f(_builtInUniforms[GLProgram::UNIFORM_COS_TIME],
       time/8.0, time/4.0, time/2.0, cosf(time));
   }
   if(_flags.usesRandom)
       setUniformLocationWith4f(_builtInUniforms[GLProgram::UNIFORM_RANDOM01],
       CCRANDOM_0_1(), CCRANDOM_0_1(), CCRANDOM_0_1(), CCRANDOM_0_1());
}
```

除了以上内置的Uniform外，Cocos2d-x还内置了一些Attribute，这些变量是需要在Shader脚本中手动输入的，主要是颜色、坐标、纹理坐标、法线等常用属性。它们被定义在CCGLProgram.cpp中，在Renderer进行渲染的时候，会调用**glVertexAttribPointer**()方法设置颜色、位置和纹理坐标Attribute，而法线Normal会在渲染3D物体时才进行设置。

```c++
const char* GLProgram::ATTRIBUTE_NAME_COLOR = "a_color";
const char* GLProgram::ATTRIBUTE_NAME_POSITION = "a_position";
const char* GLProgram::ATTRIBUTE_NAME_TEX_COORD = "a_texCoord";
const char* GLProgram::ATTRIBUTE_NAME_TEX_COORD1 = "a_texCoord1";
const char* GLProgram::ATTRIBUTE_NAME_TEX_COORD2 = "a_texCoord2";
const char* GLProgram::ATTRIBUTE_NAME_TEX_COORD3 = "a_texCoord3";
const char* GLProgram::ATTRIBUTE_NAME_NORMAL = "a_normal";
const char* GLProgram::ATTRIBUTE_NAME_BLEND_WEIGHT = "a_blendWeight";
const char* GLProgram::ATTRIBUTE_NAME_BLEND_INDEX = "a_blendIndex";
```



#### 15.5.3　编写Shader

接下来手动编写一个常用的灰化Shader，首先需要编写顶点Shader脚本，在顶点Shader脚本中只需要正确输出顶点位置即可。在顶点Shader中需要使用到a_position位置和a_texCoord纹理坐标属性。属性的名字是Cocos2d-x预定义好的，不能更改，相关名字可以参考下面这段代码。

```c++
attribute vec4 a_position;
attribute vec2 a_texCoord;
varying vec2 v_texCoord;
void main()
{
    gl_Position = CC_PMatrix * a_position;
    v_texCoord = a_texCoord;
}
```

灰化的算法非常简单，只需要将每一个像素的RGB分量进行平均计算即可，彩色图片经过灰化之后会变成灰白的图片，**灰白图片的特征就是每个像素的RGB三个分量的值均相等**，值越高颜色越白，反之颜色越黑。

经过灰化之后，图片的每个像素的灰度都是根据原图片的颜色进行计算的，常用的计算方法有3种，平均值、最大值或加权平均值，<u>平均值</u>指将RGB三个分量相加后取平均值，然后将平均值赋值给RGB。<u>最大值</u>则是取RGB中最大的值赋值给RGB，<u>加权平均值</u>则是将每个分量按照一定的比例计算，如R占20%、G占30%、B占50%，通过权值调整RGB颜色参与计算的比例（平均值相当于RGB的权值相等），最后将计算好的值累加并赋给RGB。这里使用加权平均值的方法来计算灰度。

在片段Shader中，用一个varying变量来接收纹理坐标的插值，并用一个Uniform来接收灰度的权值，本来可以直接在代码中写“死”权值，但为了更加灵活，同时也为了介绍一下Uniform的使用，这里就用Uniform来设置权值。

如果将权值“写死”，那么Shader脚本中就会出现类似texColor.r * 0.2f这样的代码，需要注意的是，**Shader中的浮点数和整数是不同的类型，如果在脚本中写的是1，那么它就是整数，如果希望它表示的是浮点数的1，那么就应该用1.0f，否则有的显卡会认为这是错误，从而导致Shader编译不通过，但有的显卡又可以通过编译**。

```c++
varying vec2 v_texCoord;
uniform vec4 u_grayParam;
void main(void)
{
vec4 texColor = texture2D(CC_Texture0, v_texCoord);
texColor.rgb = texColor.r * u_grayParam.r + texColor.g * u_grayParam.g +
texColor.b * u_grayParam.b;
    gl_FragColor = texColor;
}
```

texture2D方法传入指定的纹理采样器及纹理坐标，可以返回对应的颜色，我们将颜色进行计算之后赋值到gl_FragColor变量中，这是GLSL内置的输出变量，表示最终的片段颜色。

到这里就完成了Shader脚本的编写，将顶点和片段Shader脚本分别命名为gray.vert和gray.frag，然后放到资源目录下。



#### 15.5.4　使用Shader的步骤

最后在Cocos2d-x中应用Shader，可以在Cocos2d-x官方示例的cpp-empty-tests中的HelloWroldScene.cpp中添加少量的代码来实现灰化。

最直接的方式就是创建GLProgram，然后调用Node的setGLProgram()方法直接设置。但这样并不是很高效，因为每次都重新创建GLProgram是一个不小的消耗，涉及Shader脚本的编译和链接，所以需要使用GLProgramCache来提高效率。GLProgramCache中以一个字符串为Key，GLProgram对象为Value，缓存了GLProgram对象。在每次使用GLProgram之前，先从GLProgramCache中查找，如果找不到再进行创建，并添加到GLProgramCache中，我们需要为每种GLProgram取一个直观的名字，如这里将灰化Shader称之为MyGrayShader。

GLProgram提供了两种创建接口，createWithFilenames()方法通过传入顶点和片段Shader的文件名来创建GLProgram，而createWithByteArrays()方法则通过传入它们的源码字符串创建一个GLProgram。一般而言，将Shader脚本放在文件中，通过传入文件名的方式会更便于管理一些。但使用源码字符串则会更加高效，因为在创建的时候不必去磁盘中读取文件，大部分情况下也不会去修改已经完成的Shader脚本。

设置Uniform的代码非常简单，只需要传入要设置的Uniform的名字，以及要设置的值即可。在**不需要有不同的Uniform变量**时，可以使用GLProgramStateCache再进一步地优化，因为可以共用同一个GLProgramState。下面在HelloWroldScene.cpp中添加一个grayNode()方法，该方法的实现如下。

```c++
#include "renderer/CCGLProgramStateCache.h"
void grayNode(Node* node)
{
    GLProgram* program = GLProgramCache::getInstance()->getGLProgram
    ("MyGrayShader");
    if (nullptr == program)
    {
         program = GLProgram::createWithFilenames("gray.vert", "gray.frag");
         GLProgramCache::getInstance()->addGLProgram(program,"MyGrayShader");
    }
    GLProgramState* programState = GLProgramState::getOrCreateWithGLPro
    gram(program);
    programState->setUniformVec4("u_grayParam", Vec4(0.2f, 0.3f, 0.5f, 1.0f));
    node->setGLProgramState(programState);
}
```

接下来在HelloWorld::init()方法中，对背景图片进行灰化，在背景图片创建之后，添加一行grayNode()方法，传入sprite对象。

```c++
auto sprite = Sprite::create("HelloWorld.png");
grayNode(sprite);
```

上面的代码运行效果如图15-6所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624112705.jpeg)

图15-6　灰化背景图的效果

如果需要将灰化Shader撤销，恢复图片的颜色，那么只需要将原来的Shader设置回去即可，可以在设置灰化之前将原来的Shader记录下来。如果知道其原先的Shader也可以直接进行重置，例如Sprite对象默认的Shader是GLProgram::SHADER_NAME_ POSITION_TEXTURE_COLOR_NO_MVP，可以通过GLProgramState::getOrCreateWithGLProgramName()方法传入上面的字符串常量来获取它的GLProgramState。