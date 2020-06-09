# PicGo + Gitee(码云)实现markdown图床

[![img](https://upload.jianshu.io/users/upload_avatars/20524049/838152de-7c23-4e15-84c6-e70404631110.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/jpg)](https://www.jianshu.com/u/f81ef1ee8360)

[LeonG7](https://www.jianshu.com/u/f81ef1ee8360)关注

https://www.jianshu.com/p/b69950a49ae2



0.8582020.01.13 23:35:27字数 1,840阅读 5,165

**前言**：深感在线博客的编辑器坑太多了，文档丢失、必须联网、可移植性太差，所以开始寻找可替代的方案。

##### [markdown](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2Fmarkdown%2F3245829%3Ffr%3Daladdin)是一门易于上手能帮助作者专心写作的文档编辑语言，它的好处太多了，建议想自己动手做笔记写博客的朋友都可以学一学，10分钟上手（我昨天晚上还不会用，今天就开始用它写博客了。。足以证明它是真的很简单）

##### [Tpyora](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.typora.io%2F)是一款优雅的markdown编辑器，也推荐给大家，至于安装和配置，比安装word还简单，就不赘述了

但是，这都不是重点，重点是咱们写博客的时候，总是需要**插入图片**的，图片存在本地的话上传到博客网站去就没法显示了，就算一个图一个图的复制粘贴上去，想移植到其他的博客网站，图就会失效，我们就需要图床

> 图床是干什么的？ 图床就是一个便于在博文中插入在线图片连接的个人图片仓库。设置图床之后，在自己博客中插入的图片链接就可以随时随地在线预览了，并且不会因为任何意外原因无法查看，除非自己亲自删除。

##### 神奇的PicGo就是为了解决这个问题诞生的，它可以将图片上传到指定的图床上，然后返回markdown链接，直接粘贴到你的文档中，就搞定啦

问题又来了，网上推荐七牛云阿里云都是要租赁服务器的，太麻烦还要钱，微博现在挂链接又很厉害。大部分人选择用github，但是github虽好却是国外的网站，速度终究比不上国内网站，研究了小半天，终于发现完美的解决方案

## 最终决定使用PicGo + 国内的github - [码云](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee%2Fcom)来实现markdown图床

废话说到这里，开始进入正题

------

## 1. 安装

- PicGo
- picgo-plugin-gitee-uploader插件

#### 首先打开[picgo官网](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMolunerfinn%2FPicGo)，下载安装包

![img](https://upload-images.jianshu.io/upload_images/20524049-c3183c5e6eae9024?imageMogr2/auto-orient/strip|imageView2/2/w/838/format/png)

~~速度还挺快~~，没关系，PicGo2.1.2的安装包下面链接自取

#### 安装之后打开主界面

![img](https://upload-images.jianshu.io/upload_images/20524049-e6194df4e0b60535?imageMogr2/auto-orient/strip|imageView2/2/w/1017/format/png)

#### 选择最底下的插件设置，搜索**gitee**

![img](https://upload-images.jianshu.io/upload_images/20524049-8ebcd31954e4c76c?imageMogr2/auto-orient/strip|imageView2/2/w/1017/format/png)

#### 点击右边的gitee-uploader 1.1.2开始安装

> 这里注意一下，必须要先安装[node.js](https://links.jianshu.com/go?to=https%3A%2F%2Fnodejs.org%2Fen%2F)才能安装插件，没装的自己装一下，然后重启就行。

这个地方有两个插件，我试了一遍，两个都能用，大家看心情选择，先说一下右边这个**gitee-uploader 1.1.2**，用不了的同学就选左边那个，我都会讲一遍配置

------

## 2. 建立gitee（码云）图床库

注册码云的方法很简单，网站引导都是中文，不多说了，我们直接建立自己的图床库。

#### 点击右上角的+号，新建仓库

![img](https://upload-images.jianshu.io/upload_images/20524049-4c77956a7edd6bf0?imageMogr2/auto-orient/strip|imageView2/2/w/313/format/png)

新建仓库的要点如下：

1. ##### 输入一个仓库名称

2. ##### 其次将仓库设为公开

3. ##### 勾选使用Readme文件初始化这个仓库

**这个选项勾上，这样码云会自动给你的仓库建立master分支，这点很重要!!!** 我因为这点折腾了很久，因为使用github做图床picgo好像会自动帮你生成master分支，而picgo里的gitee插件不会帮你自动生成分支。

![img](https://upload-images.jianshu.io/upload_images/20524049-092faaf928f931c2?imageMogr2/auto-orient/strip|imageView2/2/w/730/format/png)

点击创建进入下一步

------

## 3. 配置PicGo

安装了**gitee-uploader 1.1.2**插件之后，我们开始配置插件

#### 配置插件的要点如下：

![img](https://upload-images.jianshu.io/upload_images/20524049-a496e0ff6a5661e5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1002/format/png)

- repo：用户名/仓库名称，比如我自己的仓库leonG7/blogImage，也可以直接复制仓库的url

  ![img](https://upload-images.jianshu.io/upload_images/20524049-a70d8f5ef5ea003a.png?imageMogr2/auto-orient/strip|imageView2/2/w/529/format/png)

- branch：分支，这里写上master
- token：填入码云的私人令牌
- path：路径，一般写上img
- customPath：提交消息，这一项和下一项customURL都不用填。在提交到码云后，会显示提交消息，插件默认提交的是 `Upload 图片名 by picGo - 时间`

#### 这个token怎么获取，下面登录进自己的码云

1. 点击头像，进入设置

![img](https://upload-images.jianshu.io/upload_images/20524049-dbd08214e8df9dce.png?imageMogr2/auto-orient/strip|imageView2/2/w/137/format/png)

1. 找到右边安全设置里面的私人令牌

![img](https://upload-images.jianshu.io/upload_images/20524049-28faf2cd395d8279.png?imageMogr2/auto-orient/strip|imageView2/2/w/226/format/png)

1. 点击`生成新令牌`，把**projects**这一项勾上，其他的不用勾，然后提交

   ![img](https://upload-images.jianshu.io/upload_images/20524049-a3c9591596a1218d?imageMogr2/auto-orient/strip|imageView2/2/w/530/format/png)

   

   

   

   这里需要验证一下密码，验证密码之后会出来一串数字，这一串数字就是你的token，将这串数字复制到刚才的配置里面去。

   ![img](https://upload-images.jianshu.io/upload_images/20524049-3aa34ee5507a7d1d?imageMogr2/auto-orient/strip|imageView2/2/w/458/format/png)

   > 注意：这个令牌只会明文显示一次，建议在配置插件的时候再来生成令牌，直接复制进去，搞丢了又要重新生成一个。

#### 现在保存你刚才的配置，然后将它设置为默认图床，大功告成。

还有一个插件**gitee 1.2.2-beta**，功能差不多，`刚才那个能用的话就不需要用这个`，配置的内容有点差别，简单说一下：

![img](https://upload-images.jianshu.io/upload_images/20524049-79e2bde41590795f?imageMogr2/auto-orient/strip|imageView2/2/w/1013/format/png)

- url：图床网站，这里写码云的主页 [https://gitee.com](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com)

- owner：所有者，写上你的码云账号名，如果你不知道你的账号名，进入你刚才的仓库，浏览器url里面有

  ![img](https://upload-images.jianshu.io/upload_images/20524049-483889ad27b26339?imageMogr2/auto-orient/strip|imageView2/2/w/574/format/png)

- repo：仓库名称，只要写上仓库名称就行，比如我自己的仓库blogImage

- path：写上路径，一般是img，**这几个项都不用加“ / “符号**

- token：刚才你获取的个人令牌，两个插件是通用的，如果你用了另一个再来用这个，它会自动读取另一个插件的部分配置，不用重新申请

- message：不用填

------

## 4. 测试

随便选一张图片上传（picgo也支持剪贴板上传，截图工具推荐win10的*Snipaste*神器！），试试看

### 超级快有木有！比github快很多，0.1秒上传，而且导入到你的markdown编辑器里面也是秒识别你的图片内容，而如果是github图床上传太慢不说可能还会出现下面这样识别不出来的问题！

![img](https://upload-images.jianshu.io/upload_images/20524049-3ca12d7d72c86451?imageMogr2/auto-orient/strip|imageView2/2/w/916/format/png)

github链接因为网速原因可能识别不出

#### 上传之后默认复制链接，直接粘贴到你的markdown编辑器里，就可以愉快的进行写作了！

最后推荐一下我的博客写作套件**Typora + PicGo + Snipaste**，Typora写文档，Snipaste一键截图，PicGo一键上传图片返回链接。

## 5. 用到的软件

- PicGo
- node.js

[云盘自取](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F171HgeDOv0ScN4UxVeI4hEg) 提取码: **4fc6**

###### 这么给力还不点个赞吗？

------

> 最近typora新出了功能，**一键上传到picgo**，具体教程我也写出来啦 ，新出的功能难免bug很多，所以加了一些错误排查。

[手把手教你用Typora自动上传到picgo图床【教程与排坑】](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2FdisILLL%2Farticle%2Fdetails%2F104944710)

顺便推销一下自己的博客,对机器学习感兴趣的朋友可以看一看~[LeonG是什么意思](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2FdisILLL)



![特色](https://gitee.com/nlpleaf/PicGo/raw/master/20200609183628.png)



![斯特](https://gitee.com/nlpleaf/PicGo/raw/master/20200609183659.png)



![士大夫](https://gitee.com/nlpleaf/PicGo/raw/master/20200609183756.png)



![第三方](https://gitee.com/nlpleaf/PicGo/raw/master/20200609183757.png)