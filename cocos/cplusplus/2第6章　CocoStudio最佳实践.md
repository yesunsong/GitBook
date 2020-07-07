# 第6章　CocoStudio最佳实践

本章主要分享一下CocoStudio 2.0及以上版本使用的一些经验，让大家能够更高效地使用CocoStudio。本章主要介绍以下内容：

- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　高效创建CSB。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　异步加载CSB。
- ![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200621094533.jpeg)　高效播放CSB动画。

### 6.1　高效创建CSB

CocoStudio可以导出CSB或JSON格式的资源文件，在Cocos2d-x中使用CSLoader可以加载它们，正常情况下这两种格式所占的体积（打包之后），解析速度都是CSB格式会稍微好一些，但如果在CocoStudio中大量使用了**嵌套CSB**，那么这个CSB文件的加载会耗费很长的一段时间。

例如，在CocoStudio中制作一个背包界面，背包上的每一个格子使用的都是同一个背包格子CSB文件，CocoStudio中是允许复用CSB的。在CocoStudio项目中编辑时，场景、节点等文件是CSB格式（CocoStudio Design），在导出时可以导出为CSB格式（CocoStudio Binary）。假设拖曳了100个背包格子放到背包界面上，那么导出CSB时会导出一个背包界面的CSB，以及一个背包格子的CSB文件。如果导出的是JSON格式，那么这个JSON文件中会包含100个背包格子的详细信息，如节点结构、名字、位置、Tag等。CSB格式则只会保存一份背包格子的详细信息，在背包界面CSB文件中引用100次背包格子CSB文件。

这样来看，在嵌套的情况下，CSB格式的冗余程度要大大小于JSON格式，但如果测试一下，会发现加载这样的一个CSB要比加载JSON慢很多，可以说是效率极低。经过分析发现，这样一个CSB文件加载时的瓶颈主要不是在加载纹理上，而是在加载CSB文件上，CSLoader在加载这样一个商店界面CSB时，执行了101次的文件I/O操作，首先读取商店界面CSB文件进行解析，在解析过程中发现引用到了商店格子CSB文件，则对商店格子CSB文件进行读取，因为引用了100次，所以读取了100次。文件I/O对性能有很大的影响，如此频繁地执行文件I/O，对游戏的性能影响是很致命的。除了嵌套之外，如果需要用同一个CSB文件来创建多个对象，也会产生多次文件I/O。

在了解了CSLoader加载CSB资源的性能瓶颈之后，可以从多个方面来解决。

#### 6.1.1　简单方案

首先可以使用一些简单的方法来缓解这个问题：不使用嵌套的CSB，改使用JSON格式可以大大提高嵌套CSB资源的加载效率。另外对于加载完返回的Node，使用一个池子进行管理，不用的时候回收到池子中缓存起来，而不是直接释放，下次再需要使用时先从池子里找，找不到再去加载。这些方法只能起到缓解的作用，并不能彻底解决CSLoader加载资源的性能瓶颈，效果还需要根据实际的应用场景来看。

#### 6.1.2　缓存方案

该方案实现起来较简单，有更好的扩展性，并且可以彻底解决CSLoader加载资源的性能瓶颈，但会占用一些额外的内存，用来存储CSB文件的内容，不过CSB文件一般的体积都比较小，所以影响不大（严格来说，缓存方案占用的内存应该比克隆方案更少）。

如果使用的是CocoStudio 3.10以及以上的版本，可以使用CSLoader的新接口，传入Data对象来创建Node，这样需要加载多个相同的CSB文件时，可以先用FileUtils的getDataFromFile()方法将文件的内容读取到Data对象中，然后使用该Data对象来重复创建Node，也可以将Data对象管理起来，在任何时候都可以使用该对象来创建Node，或者释放该对象。对于3.10之前的版本，可以对CSLoader进行简单的修改，手动添加这个接口，改动并不大。使用这种方式需要自己手动编写一个CSB文件管理的类，然后使用其来管理CSB文件。

缓存方案的另一种实现方式则是修改FileUtils单例，我们的目标是通过修改FileUtils加载文件的接口，对CSB文件进行缓存，缓存的规则可以自己来制定，如小于1MB的CSB文件才进行缓存。除了CSB之外，任何我们会在短时间内重复加载的文件都可以进行缓存，可以根据我们的需求方便地进行调整，甚至文件的加密解密也可以放在这里实现。

由于FileUtils与平台相关，在不同的平台下有不同的子类实现，而且其子类的构造函数是私有的，我们无法通过继承重写的方法来重写其getDataFromFile()方法。而且FileUtils的单例指针是FileUtils内部的全局变量。这重重限制让我们无法做到在不修改引擎源码的情况下实现对FileUtils的扩展。所以只能修改FileUtils的源码。直接改动FileUtils的getDataFromFile接口是最简单的，首先需要为FileUtils定义一个成员变量来缓存Data对象。

```
std::map<std::string, Data> m_Cache;
```

然后重写getDataFromFile()方法，在getDataFromFile()方法中对CSB文件进行特殊处理，先判断是否有缓存，没有则调用getData()方法加载数据，缓存并返回。

```
Data FileUtils::getDataFromFile(const std::string& filename)
{
  if (".csb" == FileUtils::getFileExtension(filename))
   {
       if (m_Cache.find(filename) == m_Cache.end())
       {
          m_Cache[filename] = getData(filename, false);
       }
       return m_Cache[filename];
   }
   return getData(filename, false);
}
```

也可以根据一个变量来设置是否开启缓存功能，以及提供清除缓存的接口，还可以在这个基础上对自己加密后的文件进行解密。

#### 6.1.3　克隆方案

克隆方案也是一种可以彻底解决CSLoader加载资源性能瓶颈的方案，并且无须修改引擎的源码。克隆方案不仅可以解决CSLoader的性能瓶颈，在很多时候我们拥有了一个节点，希望将这个节点进行复制时，都可以使用克隆的方法。Cocos2d-x的Widget实现了clone()方法，但实现得并不是很好，很多东西没有被克隆，例如，在Widget下面添加一个Sprite节点，为Widget设置了分辨率适配规则，对于CocoStudio所携带的动画Action以及一些播放动画所需的扩展信息，这些都没有被克隆。下面这里提供一个克隆节点的方法，可以使用这个方法很好地克隆绝大部分的CSB节点，对于CSB中的粒子系统以及骨骼动画等节点并没有进行克隆（主要是因为没有用到），但根据下面的代码可以自己进行扩展，克隆它们。

```
#include "CsbTool.h"
//Cocos2d-x在不同的版本下会包含一些不同的扩展信息，用于播放CSB动画，这些信息需要被
克隆
#if (COCOS2D_VERSION >= 0x00031000)
#include "cocostudio/CCComExtensionData.h"
#else
#include "cocostudio/CCObjectExtensionData.h"
#endif
#include "cocostudio/CocoStudio.h"
#include "ui/CocosGUI.h"
USING_NS_CC;
using namespace cocostudio;
using namespace ui;
using namespace timeline;
//要克隆的节点类型，WidgetNode包含了所有的UI控件
enum NodeType
{
    WidgetNode,
    CsbNode,
    SpriteNode
};
//克隆扩展信息
void copyExtInfo(Node* src, Node* dst)
{
    if (src == nullptr || dst == nullptr)
    {
        return;
    }
#if (COCOS2D_VERSION >= 0x00031000)
    auto com = dynamic_cast<ComExtensionData*>(
        src->getComponent(ComExtensionData::COMPONENT_NAME));
    if (com)
    {
        ComExtensionData* extensionData = ComExtensionData::create();
        extensionData->setCustomProperty(com->getCustomProperty());
        extensionData->setActionTag(com->getActionTag());
        if (dst->getComponent(ComExtensionData::COMPONENT_NAME))
        {
            dst->removeComponent(ComExtensionData::COMPONENT_NAME);
        }
        dst->addComponent(extensionData);
    }
#else
    auto obj = src->getUserObject();
    if (obj != nullptr)
    {
        ObjectExtensionData* objExtData = dynamic_cast<ObjectExtensionData*>
        (obj);
        if (objExtData != nullptr)
        {
            auto newObjExtData = ObjectExtensionData::create();
            newObjExtData->setActionTag(objExtData->getActionTag());
            newObjExtData->setCustomProperty(objExtData->
            getCustomProperty());
            dst->setUserObject(newObjExtData);
        }
    }
#endif
    //复制Action
    int tag = src->getTag();
    if (tag != Action::INVALID_TAG)
    {
        auto action = dynamic_cast<ActionTimeline*>(src-> getActionByTag
        (src->getTag()));
        if (action)
        {
            dst->runAction(action->clone());
        }
    }
}
//克隆布局信息
void copyLayoutComponent(Node* src, Node* dst)
{
    if (src == nullptr || dst == nullptr)
    {
        return;
    }
    //检查是否有布局组件
    LayoutComponent * layout = dynamic_cast<LayoutComponent*>(src->
    getComponent(__LAYOUT_COMPONENT_NAME));
    if (layout != nullptr)
    {
        auto layoutComponent = ui::LayoutComponent::
        bindLayoutComponent(dst);
        layoutComponent->setPositionPercentXEnabled(layout->
        isPositionPercentXEnabled());
        layoutComponent->setPositionPercentYEnabled(layout->
        isPositionPercentYEnabled());
        layoutComponent->setPositionPercentX(layout->
        getPositionPercentX());
        layoutComponent->setPositionPercentY(layout->
        getPositionPercentY());
        layoutComponent->setPercentWidthEnabled(layout->
        isPercentWidthEnabled());
        layoutComponent->setPercentHeightEnabled(layout->
        isPercentHeightEnabled());
        layoutComponent->setPercentWidth(layout->getPercentWidth());
        layoutComponent->setPercentHeight(layout->getPercentHeight());
        layoutComponent->setStretchWidthEnabled(layout->
        isStretchWidthEnabled());
        layoutComponent->setStretchHeightEnabled(layout->
        isStretchHeightEnabled());
        layoutComponent->setHorizontalEdge(layout->getHorizontalEdge());
        layoutComponent->setVerticalEdge(layout->getVerticalEdge());
        layoutComponent->setTopMargin(layout->getTopMargin());
        layoutComponent->setBottomMargin(layout->getBottomMargin());
        layoutComponent->setLeftMargin(layout->getLeftMargin());
        layoutComponent->setRightMargin(layout->getRightMargin());
    }
}
NodeType getNodeType(Node* node)
{
    if (dynamic_cast<Widget*>(node) != nullptr)
    {
        return WidgetNode;
    }
    else if (dynamic_cast<Sprite*>(node) != nullptr)
    {
        return SpriteNode;
    }
    else
    {
        return CsbNode;
    }
}
Sprite* cloneSprite(Sprite* sp);
//递归克隆子节点，如果是继承于Widget，可以调用clone()方法进行克隆，但在CocoStudio
中，Widget下可以包含其他非Widget节点，这些节点是不会被克隆的，所以需要递归检查一下
void cloneChildren(Node* src, Node* dst)
{
    if (src == nullptr || dst == nullptr)
    {
        return;
    }
    for (auto& n : src->getChildren())
    {
        NodeType ntype = getNodeType(n);
        Node* child = nullptr;
        switch (ntype)
        {
        case WidgetNode:
            //如果父节点也是Widget，则该节点已经被复制了
            if (dynamic_cast<Widget*>(src) == nullptr)
            {
                child = dynamic_cast<Widget*>(n)->clone();
                dst->addChild(child);
            }
            else
            {
                //如果节点已经存在，找到该节点
                for (auto dchild : dst->getChildren())
                {
                    if (dchild->getTag() == n->getTag()
                        && dchild->getName() == n->getName())
                    {
                        child = dchild;
                        break;
                    }
                }
            }
            //对Widget的clone()方法没有克隆到的内容进行克隆
            if (dynamic_cast<Text*>(n) != nullptr)
            {
                auto srcText = dynamic_cast<Text*>(n);
                auto dstText = dynamic_cast<Text*>(child);
                if (srcText && dstText)
                {
                    dstText->setTextColor(srcText->getTextColor());
                }
            }
            child->setCascadeColorEnabled(n->isCascadeColorEnabled());
            child->setCascadeOpacityEnabled(n->
            isCascadeOpacityEnabled());
            copyLayoutComponent(n, child);
            cloneChildren(n, child);
            copyExtInfo(n, child);
            break;
        case CsbNode:
            child = CsbTool::cloneCsbNode(n);
            dst->addChild(child);
            break;
        case SpriteNode:
            child = cloneSprite(dynamic_cast<Sprite*>(n));
            dst->addChild(child);
            break;
        default:
            break;
        }
    }
}
//克隆Sprite
Sprite* cloneSprite(Sprite* sp)
{
    Sprite* newSprite = Sprite::create();
    newSprite->setName(sp->getName());
    newSprite->setTag(sp->getTag());
    newSprite->setPosition(sp->getPosition());
    newSprite->setVisible(sp->isVisible());
    newSprite->setAnchorPoint(sp->getAnchorPoint());
    newSprite->setLocalZOrder(sp->getLocalZOrder());
    newSprite->setRotationSkewX(sp->getRotationSkewX());
    newSprite->setRotationSkewY(sp->getRotationSkewY());
    newSprite->setTextureRect(sp->getTextureRect());
    newSprite->setTexture(sp->getTexture());
    newSprite->setSpriteFrame(sp->getSpriteFrame());
    newSprite->setBlendFunc(sp->getBlendFunc());
    newSprite->setScaleX(sp->getScaleX());
    newSprite->setScaleY(sp->getScaleY());
    newSprite->setFlippedX(sp->isFlippedX());
    newSprite->setFlippedY(sp->isFlippedY());
    newSprite->setContentSize(sp->getContentSize());
    newSprite->setOpacity(sp->getOpacity());
    newSprite->setColor(sp->getColor());
    newSprite->setCascadeColorEnabled(true);
    newSprite->setCascadeOpacityEnabled(true);
    copyLayoutComponent(sp, newSprite);
    cloneChildren(sp, newSprite);
    copyExtInfo(sp, newSprite);
    return newSprite;
}
//克隆CSB节点
Node* CsbTool::cloneCsbNode(Node* node)
{
    Node* newNode = Node::create();
    newNode->setName(node->getName());
    newNode->setTag(node->getTag());
    newNode->setPosition(node->getPosition());
    newNode->setScaleX(node->getScaleX());
    newNode->setScaleY(node->getScaleY());
    newNode->setAnchorPoint(node->getAnchorPoint());
    newNode->setLocalZOrder(node->getLocalZOrder());
    newNode->setVisible(node->isVisible());
    newNode->setOpacity(node->getOpacity());
    newNode->setColor(node->getColor());
    newNode->setCascadeColorEnabled(true);
    newNode->setCascadeOpacityEnabled(true);
    newNode->setContentSize(node->getContentSize());
    copyLayoutComponent(node, newNode);
    cloneChildren(node, newNode);
    copyExtInfo(node, newNode);
    return newNode;
}
```

### 6.2　异步加载CSB

即使使用了缓存的方案，首次加载CSB文件还是会阻塞一段时间，因为这里面还包含了纹理的加载，如果要加载的纹理比较大或者要加载多个纹理，或者要同时加载多个CSB文件，那么就会有比较明显的卡顿。如果能够将CSB文件进行异步加载，就可以很好地改善这个问题。CSLoader是不支持异步加载CSB的，如果将CSB文件的加载分为加载纹理和创建节点两部分，那么创建节点这部分是**无法做到线程安全的**！因为各种节点在创建时操作了各种单例对象，如从TextureCache中获取纹理，在EventDispatcher中注册触摸事件等。在子线程和主线程中同时操作这些资源，很可能导致程序崩溃或出现其他异常。

使用了缓存方案之后，主要的瓶颈在纹理加载这里，所以可以使用TextureCache的异步加载纹理的方法，将CSB所需的纹理进行异步加载，加载完之后再在主线程中执行创建节点的逻辑。接下来的问题就是如何知道每个CSB需要加载哪些纹理。可以通过一个简单的方法解析CSB文件，得到所需的纹理，但这个方法的效率不高，所以最好是通过另外一个简单的程序，生成一个配置表，在配置表中记录每个CSB文件所需的纹理列表，然后直接使用这个配置表。使用下面的方法可以递归找出一个CSB文件加载所需的全部纹理。

```
//传入CSB文件的Data，以及用于保存纹理文件名的set，查找单个CSB所引用的所有纹理
void CCsbLoader::searchTexturesByCsbFile(Data& data, set<string>& texSet)
{
    auto csparsebinary = GetCSParseBinary(data.getBytes());
    auto textures = csparsebinary->textures();
    int textureSize = csparsebinary->textures()->size();
    for (int i = 0; i < textureSize; ++i)
    {
        string plistFile = FileUtils::getInstance()->fullPathForFilename
        (textures->Get(i)->c_str());
        if (m_LoadingPlists.find(plistFile) != m_LoadingPlists.end()
            || SpriteFrameCache::getInstance()->isSpriteFramesWithFileLoaded
            (plistFile))
        {
            continue;
        }
        m_LoadingPlists.insert(plistFile);
        Data plistData = FileUtils::getInstance()->getDataFromFile
        (plistFile);
        if (plistData.isNull())
        {
            continue;
        }
        string textureFile;
        ValueMap dict = FileUtils::getInstance()->getValueMapFromData(
            reinterpret_cast<const char*> (plistData.getBytes()), plistData.
            getSize());
        if (dict.find("metadata") != dict.end())
        {
            ValueMap& metadataDict = dict["metadata"].asValueMap();
            textureFile = metadataDict["textureFileName"].asString();
        }
        if (!textureFile.empty())
        {
            //计算相对路径，将纹理的文件名对应到plist的路径下
            textureFile = FileUtils::getInstance()->fullPathFromRelativeFile
            (textureFile, plistFile);
        }
        else
        {
            //如果plist文件中没有纹理路径名，则尝试读取plist对应的.png
            textureFile = plistFile;
            //将xxxx.plist结尾的.plist移除，替换成.png
            textureFile = textureFile.erase(textureFile.find_last_of("."));
            textureFile = textureFile.append(".png");
        }
        //该纹理未被加载且没有在待加载列表中，则添加进texSet中
        if (Director::getInstance()->getTextureCache()->getTextureForKey
        (textureFile) == nullptr
         && m_LoadingTextures.find(textureFile) == m_ LoadingTextures.end())
      {
         m_LoadingTextures.insert(textureFile);
         texSet.insert(textureFile);
      }
   }
}
```

searchTexturesByCsbNodeTree()方法可以递归查找一个CSB节点的所有嵌套CSB文件所引用到的纹理，传入一个对象和一个set容器，CSB文件所引用到的纹理都会被存储到容器中。

```
void CCsbLoader::searchTexturesByCsbNodeTree(const flatbuffers::NodeTree*
tree, set<string>& texSet)
{
    //对所有的子节点做相同的处理
    auto children = tree->children();
    int size = children->size();
    for (int i = 0; i < size; ++i)
    {
         auto subNodeTree = children->Get(i);
         //对于CsbNode子节点，需要一并加载进来
         auto options = subNodeTree->options();
         std::string classname = subNodeTree->classname()->c_str();
         if (classname == "ProjectNode")
         {
             auto projectNodeOptions = (ProjectNodeOptions*)options->data();
             std::string filePath = FileUtils::getInstance()->
             fullPathForFilename(
                  projectNodeOptions->fileName()->c_str());
             //有此文件且未加载过该文件
             //如果已经搜索过，则没必要再搜索
             if (!filePath.empty()
                  && m_CsbNodes.find(filePath) == m_CsbNodes.end()
                  && m_CheckedCsb.find(filePath) == m_CheckedCsb.end())
             {
                  m_CheckedCsb.insert(filePath);
                  Data data = FileUtils::getInstance()->getDataFromFile
                  (filePath);
                  if (!data.isNull())
                  {
                      m_CsbFileCache[filePath] = data;
                      //找到这个CSB所引用的Png
                      searchTexturesByCsbFile(data, texSet);
                      auto csparsebinary = GetCSParseBinary(data.getBytes());
                      //对该CSB进行递归
                      searchTexturesByCsbNodeTree(csparsebinary->nodeTree(),
                      texSet);
                  }
             }
         }
         else
         {
             searchTexturesByCsbNodeTree(subNodeTree, texSet);
         }
    }
}
```

### 6.3　高效播放CSB动画

在使用CSLoader加载的节点时，可以让其执行一个ActionTimeline类型的Action，通过调用ActionTimeline的方法可以控制动画的播放和暂停等，一般情况下要播放一个CSB节点的动画时，都是先创建CSB文件对应的ActionTimeline，然后让CSB节点执行，最后调用ActionTimeline的play()方法播放动画。实际上这是一种低效的做法，因为CSB节点的ActionTimeline是不会停止的，也就是说我们只需要一个ActionTimeline就够了，而不是每播放一次动画创建一个。那么应该如何获取到CSB节点当前的ActionTimeline呢？ActionTimeline与其他的Action有两点最大的不同，除了ActionTimeline不会停止之外，ActionTimeline在执行的时候，Action的tag就会被设置为节点的tag。

所以正确的做法应该是这样的，先根据CSB节点的tag获取Action，并动态转换成ActionTimeline（在一些旧版本的引擎中，同一个CSB文件创建出来的多个节点对象的ActionTimeline对象是同一个，所以可能出现播放一个ActionTimeline的动画，所有CSB对象都执行了动画，可以升级引擎或使用ActionTimeline的clone方法解决），如果转换成功则使用这个ActionTimeline来播放动画，否则再使用CSB的路径创建一个ActionTimeline，让CSB节点执行这个Action。但如果在运行之后修改了CSB节点的Tag，或者将这个Action停止了，就无法正确播放动画了。

Cocos2d-x 3.10之前的版本是会自动执行ActionTimeline的，但由于在某些情况下会存在严重的内存泄漏，所以Cocos2d-x 3.10的代码中取消了根节点自动播放ActionTimeline的功能，但嵌套的CSB节点还是会自动播放ActionTimeline。CSLoader的内存泄漏很隐蔽，但危害很大，重现这个内存泄漏的BUG很简单，只需要在一个for循环中不断调用CSLoader创建节点，然后再直接调用release将创建的节点释放，就会产生内存泄漏了，如果加载的是比较复杂的CSB节点，更容易重现这个问题。查看程序占用的内存会发现，程序占用了很大的一块内存。

这个内存泄漏的原因是因为CSB节点在创建的时候自动执行的ActionTimeline，这时候ActionManager会对CSB节点有一个retain操作，增加了它的引用计数，而直接通过autorelease释放CSB节点，但并不会真正释放这个CSB节点，因为没有一个地方让ActionManager执行release的操作，如果在释放之前先执行一下CSB节点的cleanup()方法，就可以解决内存泄漏。