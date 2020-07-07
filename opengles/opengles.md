# OpenGL ES 2.0 渲染流程整理

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200704004958.png)

[_大猪](https://me.csdn.net/u013654125) 2018-03-26 15:11:41 ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200704004959.png) 1348 ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200704005000.png) 收藏 1

分类专栏： [opengl](https://blog.csdn.net/u013654125/category_6981639.html) 文章标签： [opengl](https://so.csdn.net/so/search/s.do?q=opengl&t=blog&o=vip&s=&l=&f=&viparticle=)[opengl es](https://so.csdn.net/so/search/s.do?q=opengl es&t=blog&o=vip&s=&l=&f=&viparticle=)

文章转载自：https://blog.csdn.net/xufeng0991/article/details/51958492



OpenGL渲染流程及渲染管线，OpenGL ES2.0的渲染管线如下图所示，阴影部分为可编程阶段。

![这里写图片描述](https://gitee.com/nlpleaf/PicGo/raw/master/20200704005001.png)

下面是对图中的每个过程的详细解释：

### 1 VBO/VAO（顶点缓冲区对象或顶点数组对象）

VBO/VAO是cpu提供给GPU的顶点信息，包括了顶点的位置、颜色、纹理坐标（用于纹理贴图）等顶点信息。 
VBO，全名Vertex Buffer Object。它是GPU里面的一块缓冲区，当我们需要传递数据的时候，可以先向GPU申请一块内存，然后往里面填充数据。最后，再通过调用glVertexAttribPointer把数据传递给Vertex Shader。 
VAO，全名为Vertex Array Object，它的作用主要是记录当前有哪些VBO，每个VBO里面绑定的是什么数据，还有每一个vertex attribute绑定的是哪一个VBO。

### 2 VertexShader（顶点着色器）

![这里写图片描述](https://gitee.com/nlpleaf/PicGo/raw/master/20200704005002.png)

顶点着色器是处理VBO/VAO提供的顶点信息的程序。 
VBO/VAO提供的每个顶点都执行一遍顶点着色器。 
Uniforms（一种变量类型）在每个顶点保持一致，Attribute每个顶点都不同（可以理解为输入顶点属性）。

顶点着色器的输入数据由下面组成：

- Attributes：使用顶点数组封装每个顶点的数据，一般用于每个顶点都各不相同的变量，如顶点位置、颜色等。
- Uniforms：顶点着色器使用的常量数据，不能被着色器修改，一般用于对同一组顶点组成的单个3D物体中所有顶点都相同的变量，如当前光源的位置。
- Samplers：这个是可选的，一种特殊的uniforms，表示顶点着色器使用的纹理。
- Shader program：顶点着色器的源码或可执行文件，描述了将对顶点执行的操作。

顶点着色器的输出：

- varying：在图元光栅化阶段，这些varying值为每个生成的片元进行计算，并将结果作为片元着色器的输入数据。从分配给每个顶点的原始varying值来为每个片元生成一个varying值的机制叫做插值。
- 另外，还有gl_postion、gl_FrontFacing和gl_PointSize。

顶点着色器可用于传统的基于顶点的操作，例如：基于矩阵变换位置，进行光照计算来生成每个顶点的颜色，生成或者变换纹理坐标。 
另外因为顶点着色器是由应用程序指定的，所以你可以用来进行任意自定义的顶点变换。

### 3 PrimitiveAssembly（图元装配）：

顶点着色器下一个阶段是图元装配，这个阶段，把顶点着色器输出的顶点组合成图元。图元（primitive）是一个能用opengl es绘图命令绘制的几何体，包括三角形、直线或者点精灵等几何对象，绘图命令指定了一组顶点属性，描述了图元的几何形状和图元类型。在图元装配阶段，这些着色器处理过的顶点被组装到一个个独立的几何图元中，例如三角形、线、点精灵。对于每个图元，必须确定它是否位于视椎体内(3维空间显示在屏幕上的可见区域)，如果图元部分在视椎体中，需要进行裁剪，如果图元全部在视椎体外，则直接丢弃图元。裁剪之后，顶点位置转换成了屏幕坐标。背面剔除操作也会执行，它根据图元是正面还是背面，如果是背面则丢弃该图元。经过裁剪和背面剔除操作后，就进入渲染流水线的下一个阶段：光栅化。

### 4 rasterization（光栅化）

![这里写图片描述](https://gitee.com/nlpleaf/PicGo/raw/master/20200704005003.png)

光栅化是将图元转化为一组二维片段的过程，然后，这些片段由片段着色器处理（片段着色器的输入）。这些二维片段代表着可在屏幕上绘制的像素。用于从分配给每个图元顶点的顶点着色器输出生成每个片段值的机制称作插值（Interpolation）。这句不是人话的话解释了一个问题，就是从cpu提供的分散的顶点信息是如何变成屏幕上密集的像素的，图元装配后顶点可以理解成变为图形，光栅化时可以根据图形的形状，插值出那个图形区域的像素（纹理坐标v_texCoord、颜色等信息）。注意，此时的像素并不是屏幕上的像素，是不带有颜色的。接下来的片段着色器完成上色的工作。总之，光栅化阶段把图元转换成片元集合，之后会提交给片元着色器处理，这些片元集合表示可以被绘制到屏幕的像素。

### 5 FragmentShader（片段着色器）

![这里写图片描述](https://gitee.com/nlpleaf/PicGo/raw/master/20200704005004.png)

片段着色器为片段（像素）上的操作实现了通用的可编程方法，光栅化输出的每个片段都执行一遍片段着色器，对光栅化阶段生成每个片段执行这个着色器，生成一个或多个（多重渲染）颜色值作为输出。 
片元着色器对片元实现了一种通用的可编程方法，它对光栅化阶段产生的每个片元进行操作，需要的输入数据如下：

- Varying variables：顶点着色器输出的varying变量经过光栅化插值计算后产生的作用于每个片元的值。
- Uniforms：片元着色器使用的常量数据
- Samplers：一种特殊的uniforms，表示片元着色器使用的纹理。
- Shader program：片元着色器的源码或可执行文件，描述了将对片元执行的操作。

片元着色器也可以丢弃片元或者为片元生成一个颜色值，保存到内置变量gl_FragColor。光栅化阶段产生的颜色、深度、模板和屏幕坐标(Xw, Yw)成为流水线中pre-fragment阶段(FragmentShader之后)的输入。

### ６Per-Fragment Operations(逐个片元操作阶段)

![这里写图片描述](https://gitee.com/nlpleaf/PicGo/raw/master/20200704005005.png) 
片元着色器之后就是逐个片元操作阶段，包括一系列的测试阶段。一个光栅化阶段产生的具有屏幕坐标(Xw, Yw)的片元，只能修改framebuffer(帧缓冲)中位置在(Xw, Yw)的像素。上图是Opengl es 2.0逐片元操作的过程：

- Pixel ownership test：像素所有权测试决定framebuffer中某一个(Xw, Yw)位置的像素是否属于当前Opengl ES的context，比如：如果一个Opengl ES帧缓冲窗口被其他窗口遮住了，窗口系统将决定被遮住的像素不属于当前Opengl ES的context，因此也就不会被显示。
- Scissor test：裁剪测试决定位置为(Xw, Yw)的片元是否位于裁剪矩形内，如果不在，则被丢弃。
- Stencil and depth tests：模板和深度测试传入片元的模板和深度值，决定是否丢弃片元。
- Blending：将新产生的片元颜色值和framebuffer中某个(Xw, Yw)位置存储的颜色值进行混合。
- Dithering：抖动可以用来最大限度的减少使用有限精度存储颜色值到framebuffer的工件。 
  逐片元操作之后，片元要么被丢弃，要么一个片元的颜色，深度或者模板值被写入到framebuffer的(Xw, Yw)位置，不过是否真的会写入还得依赖于write masks启用与否。write masks能更好的控制颜色、深度和模板值写入到合适的缓冲区。例如：颜色缓冲区中的write mask可以被设置成没有红色值写入到颜色缓冲区。另外，Opengl ES 2.0提供从framebuffer中获取像素的接口，不过需要记住的是像素只能从颜色缓冲区读回，深度和模板值不能读回。

> 参考： 
> OpenGL渲染流程 http://www.cnblogs.com/BigFeng/p/5068715.html 
> OpenGL ES 2.0渲染管线 http://codingnow.cn/opengles/1504.html (图片来源) 
> OpenGL ES 2.0可编程管道 http://www.cnblogs.com/listenheart/p/3292672.html 
> OpenGL ES 2.0编程基础 http://blog.csdn.net/iispring/article/details/7649628 
> OpenGL-渲染管线的流程(有图有真相) http://www.cnblogs.com/zhanglitong/p/3238989.html