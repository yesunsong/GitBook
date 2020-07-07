# 第**16**章　使用**Shader**——常用特效

第15章了解了OpenGL的渲染管线、GLSL的基础语法以及如何在Cocos2d-x中编写Shader，本章了解一下如何使用Shader制作一些炫酷的效果。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　Blur模糊效果。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　OutLine描边效果。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　RGB、HSV与HSL效果。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　调整色相。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　流光效果。

### 16.1　Blur模糊效果

首先介绍testcpp的Shader-Sprite示例自带的模糊效果，模糊效果实现的思路是这样的，<u>在片段着色器中将当前像素的颜色值与周围像素的颜色值累加，然后取平均值</u>，这样就可以得到一个模糊的效果。如果希望控制模糊的程度，那么就需要引入**<u>权重</u>**的概念，<u>当前像素的权重最高，向周围的像素逐渐减弱</u>，这样可以控制当前像素的颜色不会和原色相差太多，其次结合**<u>控制模糊计算的区域</u>**，可以很好地控制模糊效果，模糊计算的区域越大，效果越模糊，反之则越清晰，效果如图16-1所示。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170414.jpeg)

图16-1　模糊效果

Shader的脚本如下所示，这里传入了resolution、blurRadius、sampleNum共3个Uniform，分别代表纹理的尺寸、模糊区域以及区域内的采样数。由于<u>纹理坐标的取值范围是0～1</u>，所以我们无法知道当前像素的周围像素对应的纹理坐标，因此需要传入图片的尺寸，通过尺寸可以确定图片的x和y轴分别有多少个像素，将1除以图片尺寸就可以得出每个像素的单位距离，如宽度为100像素的图片，在当前像素的x轴加上0.01就可以得到其右边像素的纹理坐标。

```c
#ifdef GL_ES
precision mediump float;
#endif

varying vec4 v_fragmentColor;
varying vec2 v_texCoord;

uniform vec2 resolution;
uniform float blurRadius;
uniform float sampleNum;
vec4 blur(vec2);
void main(void)
{
    //调用blur()函数计算模糊后的颜色值
    vec4 col = blur(v_texCoord);
    gl_FragColor = vec4(col) * v_fragmentColor;
}

vec4 blur(vec2 p)
{
    //如果模糊区域大于0且采样数大于1才进行模糊处理
    if (blurRadius > 0.0 && sampleNum > 1.0)
    {
        //计算出像素的单位制
        vec4 col = vec4(0);
        vec2 unit = 1.0 / resolution.xy;
        //范围除以采样数，得出要遍历的步长
        float r = blurRadius;
        float sampleStep = r / sampleNum;
        float count = 0.0;
        //以当前像素点为中心，遍历周围指定大小的矩形范围
        for(float x = -r; x < r; x += sampleStep)
        {
            for(float y = -r; y < r; y += sampleStep)
            {
                //计算权重，越靠边缘权重越小
                float weight = (r - abs(x)) * (r - abs(y));
                //所有的颜色值乘以对应的权重，然后累加起来并将权重累加到count变量中
                col += texture2D(CC_Texture0, p + vec2(x * unit.x, y * unit.y))
                * weight;
                count += weight;
            }
        }
        //将颜色累加除以权重，返回其平均值
        return col / count;
    }
    //默认返回原色
    return texture2D(CC_Texture0, p);
}
```

```c
#ifdef GL_ES
precision lowp float;
#endif

attribute vec2 a_texCoord;
attribute vec4 a_color;
attribute vec4 a_position;

varying vec4 v_fragmentColor;
varying vec2 v_texCoord;

void main()
{
    gl_Position = CC_PMatrix * a_position;
    v_texCoord = a_texCoord;
    v_fragmentColor = a_color;
}
```



### 16.2　OutLine描边效果

testcpp的Shader-Sprite示例中有一个简单的描边效果，这个描边效果实现得比较糟糕，首先会将图片原本应该空白的地方填充为黑色，如图16-2右图所示，稍后会介绍如何修复这个BUG，修复后的效果如图16-2左图所示。其次该描边效果无法很好地控制描边的粗细，如果需要大一些的描边，效果会变得非常糟糕。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170415.jpeg)

图16-2　描边效果

该描边效果实现的思路大致如下，首先累加上、下、左、右4个点的透明度，然后将累加的透明度乘以设定的阀值（该阀值可以控制描边颜色，但意义不大），并将结果乘以描边色作为要设置的颜色（黑色部分由于其上、下、左、右4个点的透明度加起来为0，乘以描边色之后得出的rgb就是vec3(0, 0, 0)黑色），最后根据当前颜色的透明度作为权值，来决定描边色和原色的比重，如果当前颜色的透明度为0，那么会完全使用原先的颜色，如果当前颜色的透明度为1，则会完全使用计算后的描边色。

Cocos2d-x示例的Shader脚本源码如下。

```c
varying vec2 v_texCoord;
varying vec4 v_fragmentColor;
uniform vec3 u_outlineColor;
uniform float u_threshold;
uniform float u_radius;
void main()
{
    float radius = u_radius;
    vec4 accum = vec4(0.0);
    vec4 normal = vec4(0.0);
    //获取当前颜色
    normal = texture2D(CC_Texture0, vec2(v_texCoord.x, v_texCoord.y));
    //累加上、下、左、右4个点的颜色
    accum += texture2D(CC_Texture0, vec2(v_texCoord.x - radius, v_texCoord.y
   - radius));
   accum += texture2D(CC_Texture0, vec2(v_texCoord.x + radius, v_texCoord.y
   - radius));
   accum += texture2D(CC_Texture0, vec2(v_texCoord.x + radius, v_texCoord.y
   + radius));
   accum += texture2D(CC_Texture0, vec2(v_texCoord.x - radius, v_texCoord.y
   + radius));
   //进行一些无实际意义的计算....
   accum *= u_threshold;
   accum.rgb = u_outlineColor * accum.a;
   accum.a = 1.0;
   //按照当前颜色的透明度，来决定当前颜色的颜色值
   normal = ( accum * (1.0 - normal.a)) + (normal * normal.a);
   gl_FragColor = v_fragmentColor * normal;
}
```

接下来对这个脚本进行一下优化，并解决其将背景的透明部分填充为黑色的BUG，调整后的代码如下。

```c
void main()
{
    float radius = u_radius;
    vec4 accum = vec4(0.0);
    vec4 normal = vec4(0.0);
    normal = texture2D(CC_Texture0, vec2(v_texCoord.x, v_texCoord.y));
    //将透明度的判断放到第一步，如果该点本身不透明，则直接使用原先的颜色即可，无须描边
    if(normal.a < 0.5)
    {
        //仍然是累加计算，如果希望描边更加平滑，可以计算其周围12个方向的颜色，甚至更多
        accum += texture2D(CC_Texture0, vec2(v_texCoord.x - radius, v_
        texCoord.y - radius));
        accum += texture2D(CC_Texture0, vec2(v_texCoord.x + radius, v_
        texCoord.y - radius));
        accum += texture2D(CC_Texture0, vec2(v_texCoord.x + radius, v_
        texCoord.y + radius));
        accum += texture2D(CC_Texture0, vec2(v_texCoord.x - radius, v_
        texCoord.y + radius));
        //如果计算后，周围有任何一个点是不透明的，则需要进行描边
        if(accum.a >= 0.1)
        {
            //设置accum为描边颜色
            accum.rgb = u_outlineColor;
            accum.a = 1.0;
            //根据原色normal的透明度来决定最终颜色
            normal = ( accum * (1.0 - normal.a)) + (normal * normal.a);
        }
    }
    gl_FragColor = v_fragmentColor * normal;
}
```

由于我们移除了u_threshold变量，所以需要在C++代码中将设置该Uniform变量的代码注释，另外通过增量一些判断，减少了一些无意义的计算。

前面介绍的OutLine描边效果存在一个问题，就是在图片自身的渲染过程中，对图片本身做的处理，因为图片本身存在一些空白的地方，这些空白也会进入片段着色器执行渲染，如果要渲染的图片的四周没有多余的空白，或者图片不带透明通道，或者要渲染的是一个3D模型，那么上面这种描边的效果就很糟糕了。接下来要介绍的是另外一种常用的描边思路——<u>对图片执行两次渲染，首先放大图片渲染一次，对此次渲染应用一个纯色的Shader，颜色就使用我们设定的描边色，如果希望效果更好一些，还可以加上一点模糊效果，第二次正常渲染图片</u>，这样就可以得到一个不错的描边效果了。

但这种方式应用于一些不是居中缩放的图片时效果也不是很理想，因为缩放可能会导致错位，从而看上去就不是描边的效果了，对于这种图片，我们只能在原图的上、下、左、右4个方向甚至8个方向偏移若干像素，然后渲染多次，从而达到较好的描边效果。



### 16.3　RGB、HSV与HSL效果

RGB是我们最熟悉的一种色彩模式，RGB使用三原色作为色彩的分量来表达所有的颜色，我们可以用一个正方体来描述RGB颜色空间，如图16-3所示。坐标系的X、Y、Z分别表示RGB颜色分量，取值范围为0～1。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170416.jpeg)

图16-3　描述RGB颜色空间的正方体

HSV和HSL都是基于RGB的色彩模式，由于RGB的描述方式不够直观，相对而言，用色彩、是否饱满以及明暗程度来描述一个颜色，要比直接告诉RGB三个分量的值直观得多。可以用一个倒锥体来描述HSV颜色空间（如图16-4所示），使用锥体和倒锥体组合来描述HSL颜色空间（如图16-5所示）。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170417.jpeg)

图16-4　描述HSV颜色空间的倒锥体

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170418.jpeg)



图16-5　描述HSL的椎体和倒锥体组合

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　H表示Hue色相，取值范围为0°～360°，对应模型中的圆形色环，每一度都对应一种颜色，HSV和HSL中关于色相的定义是一样的。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　S表示Saturation饱和度，取值范围为0～1，表示颜色的鲜艳程度，该值越大色彩越鲜艳。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　V表示Value明度，取值范围为0～1，在HSV中用于表示光的量。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　L表示Lightness亮度，取值范围为0～1，在HSL中用于表示白这种颜色的量。



### 16.4　调整色相

了解了HSV和HSL颜色空间有什么用呢？用处可多了，可以对色相、饱和度、明度进行调整，如果是RGB颜色模型，很难通过直接调整RGB的颜色值，来控制颜色的色相、饱和度、明度等属性。但如果先将RGB转换为HSV或HSL，则可以直接调整颜色的色相、饱和度、明度等属性，调整之后再转换回RGB颜色。

举一个实际例子，改变图片的颜色是一个很常见的需求，当我们有一个角色的时候，通过变色来生成其他类似的角色，可以大大节省图片资源。在Cocos2d-x中通过setColor可以设置节点的颜色，但设置完的效果往往不是想要的效果。而通过调整色相，则可以调整出不错的变色效果，如图16-6所示，我们希望通过调整颜色得到一个紫色的角色，图16-6中的3个角色，中间的角色是原图，左侧的角色调用了setColor设置紫色，而右侧的角色则使用了修改色相的方式。修改色相的效果要比调用setColor的效果好得多。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170419.jpeg)

图16-6　setColor与修改色相的效果对比

调整色相的片段Shader脚本如下，HSV、HSL与RGB的转换公式比较复杂，这里不做详细介绍。

```c
#ifdef GL_ES
precision lowp float;
#endif
//传入HSV偏移的Uniform
uniform vec3 u_hsv;
varying vec4 v_fragmentColor;
varying vec2 v_texCoord;
//将RGB转换为HSV
vec3 rgb2hsv(vec3 c)
{
    vec4 K = vec4(0.0, -1.0 / 3.0, 2.0 / 3.0, -1.0);
    vec4 p = mix(vec4(c.bg, K.wz), vec4(c.gb, K.xy), step(c.b, c.g));
    vec4 q = mix(vec4(p.xyw, c.r), vec4(c.r, p.yzx), step(p.x, c.r));
    float d = q.x - min(q.w, q.y);
    float e = 1.0e-10;
    return vec3(abs(q.z + (q.w - q.y) / (6.0 * d + e)), d / (q.x + e), q.x);
}
//将HSV转换为RGB
vec3 hsv2rgb(vec3 c)
{
    vec4 K = vec4(1.0, 2.0 / 3.0, 1.0 / 3.0, 3.0);
    vec3 p = abs(fract(c.xxx + K.xyz) * 6.0 - K.www);
    return c.z * mix(K.xxx, clamp(p - K.xxx, 0.0, 1.0), c.y);
}
void main()
{
    gl_FragColor = v_fragmentColor * texture2D(CC_Texture0, v_texCoord);
    //对透明度大于0.5的像素进行色相旋转
    if(gl_FragColor.a >= 0.5)
    {
        //获取当前RGB对应的HSV
        vec3 hsvcolor = rgb2hsv(gl_FragColor.rgb);
        //累加上传入的Uniform
        hsvcolor += u_hsv;
        //将修改后的HSV转换回RGB
        gl_FragColor.rgb = hsv2rgb(hsvcolor);
    }
}
```

下面的代码简单演示了如何应用色相旋转Shader，除了旋转之外，还可以将色相统一设置为指定的值，来实现如冰冻、中毒、灼烧等效果，只需要将上述脚本的hsvcolor += u_hsv调整为hsvcolor.x = u_hsv.x即可。下面的代码是应用在Spine骨骼动画之上，对于Sprite同样有效。但需要注意保持顶点Shader，Sprite和Spine的顶点Shader不同，如果要对Sprite设置色相，需要设置Sprite默认的顶点Shader，Sprite的顶点Shader为ccPositionTextureColor_noMVP_vert，可以将下面代码的ccPositionTextureColor_vert替换为ccPositionTextureColor_noMVP_vert，来设置Sprite。

```c++
auto glProgram = ShaderCache::getInstance()->getGLProgram("HSV");
if (glProgram == nullptr)
{
    auto fileUtiles = FileUtils::getInstance();
    auto fragmentFilePath = fileUtiles->fullPathForFilename ("shaders/HSV.frag");
    auto fragSource = fileUtiles->getStringFromFile(fragmentFilePath);
    glProgram = GLProgram::createWithByteArrays(ccPositionTextureColor_vert, fragSource.c_str());
    ShaderCache::getInstance()->addGLProgram(glProgram, "HSV");
}

SkeletonAnimation::createWithFile("role/12000/12000.json","role/12000/12000.atlas");
ani->setSkin("12002");
ani->setPosition(Director::getInstance()->getWinSize() * 0.5f);
parent->addChild(ani);
                                                             
SkeletonAnimation::createWithFile("role/12000/12000.json","role/12000/12000.atlas");
ani2->setSkin("12002");
ani2->setPosition(Director::getInstance()->getWinSize() * 0.5f);
ani2->setPositionX(ani2->getPositionX() - 100.0f);
ani2->setColor(Color3B::MAGENTA);
parent->addChild(ani2);
                                                             
SkeletonAnimation::createWithFile("role/12000/12000.json","role/12000/12000.atlas");
ani3->setSkin("12002");
ani3->setPosition(Director::getInstance()->getWinSize() * 0.5f);
ani3->setPositionX(ani3->getPositionX() + 100.0f);
parent->addChild(ani3);
ani3->setGLProgram(glProgram);
                                                             
auto vec = Vec3(0.7, 0, 0);
ani3->getGLProgramState()->setUniformVec3("u_hsv", vec);
```

在这里我们要传入色相偏转的Uniform，虽然色相的取值范围是0°～360°，但在这里的单位是1，所以取值范围是0～1。



### 16.5　流光效果

流光效果是笔者在实际项目中制作的一种效果，美工给出一张七彩的卡片框，需要让卡片上的颜色流转起来，这比单纯的七彩要炫酷得多，那么如何实现呢？仍然是使用色相旋转来实现，只不过前面介绍的是静态的色相旋转，而这里要使用的是动态的色相旋转，Shader脚本需要有一点小改动，另外由于是一个持续变化的Shader，所以这里封装了一个Action来实现这个效果，效果如图16-7所示，由于是动态的效果，所以这里给出了多张图以便观察在不同时刻下的颜色变化。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200624170420.jpeg)

图16-7　流光效果

流光效果的片段Shader脚本如下所示，与色相调整Shader的主要区别在于hsvcolor.x = mod(u_modTime * 0.25 + hsvcolor.x, 360.0);，这里使用了一个取模的方法，并且在外部会根据时间的变化来修改u_modTime的值，可以通过调整u_modTime的系数来控制颜色流动的速度。

```c++
#ifdef GL_ES
precision lowp float;
#endif
uniform float u_modTime;
varying vec4 v_fragmentColor;
varying vec2 v_texCoord;
vec3 rgb2hsv(vec3 c)
{
    vec4 K = vec4(0.0, -1.0 / 3.0, 2.0 / 3.0, -1.0);
    vec4 p = mix(vec4(c.bg, K.wz), vec4(c.gb, K.xy), step(c.b, c.g));
    vec4 q = mix(vec4(p.xyw, c.r), vec4(c.r, p.yzx), step(p.x, c.r));
    float d = q.x - min(q.w, q.y);
    float e = 1.0e-10;
    return vec3(abs(q.z + (q.w - q.y) / (6.0 * d + e)), d / (q.x + e), q.x);
}

vec3 hsv2rgb(vec3 c)
{
    vec4 K = vec4(1.0, 2.0 / 3.0, 1.0 / 3.0, 3.0);
    vec3 p = abs(fract(c.xxx + K.xyz) * 6.0 - K.www);
    return c.z * mix(K.xxx, clamp(p - K.xxx, 0.0, 1.0), c.y);
}

void main()
{
    gl_FragColor = v_fragmentColor * texture2D(CC_Texture0, v_texCoord);
    if(gl_FragColor.a >= 0.8)
    {
        vec3 hsvcolor = rgb2hsv(gl_FragColor.rgb);
        hsvcolor.x = mod(u_modTime * 0.25 + hsvcolor.x, 360.0);
        gl_FragColor.rgb = hsv2rgb(hsvcolor);
    }
}
```

流光效果的Action实现如下，首先是头文件，定义了若干成员变量，以及方法。

```c++
class CFluxayAction : public cocos2d::ActionInterval
{
public:
    CFluxayAction();
    virtual ~CFluxayAction();
    static CFluxayAction* create();
    virtual void startWithTarget(cocos2d::Node *target) override;
    virtual void step(float dt) override;
    virtual void fourceStop();
    virtual bool isDone() const { return false; }
    inline cocos2d::GLProgramState* getProgramState()
    {
        return m_OldProgramState;
    }
private:
    float m_fModTime;
    cocos2d::GLProgramState* m_FluxayProgramState;
    cocos2d::GLProgramState* m_OldProgramState;
    cocos2d::Node* m_Target;
};
```

startWithTarget、step和fourceStop是这个Action最核心的3个方法，在startWithTarget中创建了流光Shader，并记录了节点原先的Shader，将流光Shader设置为节点的当前Shader。fourceStop()方法会在Action结束时被调用，主要的工作是恢复节点为原先的Shader，而step()方法则会在每一帧被调用，修改u_modTime的Uniform值。

```c++
void CFluxayAction::startWithTarget(cocos2d::Node *target)
{
    if (target)
    {
        ActionInterval::startWithTarget(target);
        m_Target = target;
        m_Target->retain();
        //保存旧状态
        CC_SAFE_RELEASE_NULL(m_OldProgramState);
        m_OldProgramState = target->getGLProgramState();
        CC_SAFE_RETAIN(m_OldProgramState);
        //获取FluxayShader，获取不到就创建一个
        auto glProgram = ShaderCache::getInstance()->getGLProgram("Fluxay");
        if (glProgram == nullptr)
        {
            auto fileUtiles = FileUtils::getInstance();
            auto fragmentFilePath=fileUtiles->fullPathForFilename ("shaders/
            Fluxay.frag");
            auto fragSource = fileUtiles->getStringFromFile(fragmentFilePath);
            auto vertexFilePath = fileUtiles->fullPathForFilename("shaders/
            Fluxay.vert");
            auto vertSource = fileUtiles->getStringFromFile(vertexFilePath);
            glProgram = GLProgram::createWithByteArrays(vertSource.c_str(),
            fragSource.c_str());
            ShaderCache::getInstance()->addGLProgram(glProgram, "Fluxay");
        }
        //设置Shader到当前节点
        target->setGLProgram(glProgram);
        m_FluxayProgramState = target->getGLProgramState();
        CC_SAFE_RETAIN(m_FluxayProgramState);
    }
}
void CFluxayAction::step(float time)
{
    if (m_FluxayProgramState)
    {
        //累加时间并修改Uniform
        m_fModTime += Director::getInstance()->getAnimationInterval();
        m_FluxayProgramState->setUniformFloat("u_modTime", m_fModTime);
    }
}
void CFluxayAction::fourceStop()
{
    //恢复为原样
    if (m_OldProgramState && m_Target)
    {
        m_Target->setGLProgramState(m_OldProgramState);
        CC_SAFE_RELEASE_NULL(m_OldProgramState);
    }
    CC_SAFE_RELEASE_NULL(m_Target);
}
```

