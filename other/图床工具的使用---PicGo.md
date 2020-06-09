---
typora-root-url: ..
---

# 图床工具的使用---PicGo

[![img](https://gitee.com/nlpleaf/PicGo/raw/master/d2002da1-0981-4540-be50-728fbf64b753.png)](https://www.jianshu.com/u/8e750c8f0fae)

[广州芦苇科技App](https://www.jianshu.com/u/8e750c8f0fae)关注

42019.01.15 16:51:49字数 1,303阅读 30,085

> 所谓图床工具，就是自动把本地图片转换成链接的一款工具，网络上有很多图床工具，就目前使用种类而言，PicGo 算得上一款比较优秀的图床工具。它是一款用 `Electron-vue` 开发的软件，可以支持微博，七牛云，腾讯云COS，又拍云，GitHub，阿里云OSS，SM.MS，imgur 等8种常用图床，功能强大，简单易用

我们到下面的链接下载最新版本，

https://github.com/Molunerfinn/PicGo/releases

注意：*mac* 系统选择 *dmg* 下载，*windwos* 选择 *.exe*系统，如果不是下载安装包，想看源码的话，可以选择 `git clone https://github.com/Molunerfinn/PicGo.git` 克隆到本地

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-3e994af556f91ac4.jpg)

image

安装后完成之后，平时会显示在桌面最上方，点击小图标，选择打开详细窗口，窗口看着还是非常整洁，美观

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-c6b73a86530641bf)

image

------

### GitHub

在此，我们就拿 *GitHub* 作为图床来说一下具体用法。由于在国内访问 *GitHub* 速度不是很快，不过如果你有特殊工具或者适应这样的速度的话，那就不妨使用一下，这是个不错的选择

首先登陆 *GitHub*，新建一个仓库或者也可以使用一个已有仓库

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-c7909f4e3fcbbf7f)

image

前几天 *GitHub* 上已经开放使用*私有库*，我们可以肆无忌惮的在上面存放私人信息～～（PS：想是这样想，不过私有库只有自己能够访问，因此图片上传上去之后是没法显示滴～👻）

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-402f67cb199e341f)

image

创建好后，需要在 GitHub 上生成一个 *token* 以便 PicGo 来操作我们的仓库，来到个人中心，选择 *Developer settings* 就能看到 *Personal access tokens*，我们在这里创建需要的 *token*

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-4f3b5ec620ad82c9)

image

点击 Generate new token 创建一个新 token，选择 repo，同时它会把包含其中的都会勾选上，我们勾选这些就可以了。然后拉到最下方点击绿色按钮，Generate token 即可。之后就会生成一个 *token* ，记得复制保存到其他地方，这个 *token* 只显示一次！！

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-f0c880d3bb7236d1)

image

------

### 配置 PicGo

打开 PicGo 面板，

- 仓库名格式为 `用户名/仓库名`
- 分支名：master
- token：上一个咱们创建的token

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-6267aa02a346dff8)

image

然后点击确定即可完成绑定，然后设置成默认图床

#### 使用

接着打开上传区，把需要的图片拉进去，这里有三种上传图片的方式：

- menubar 图标拖拽上传（仅支持 macOS）
- 主窗口拖拽或者选择图片上传
- 剪贴板图片（最常见的是截图）上传（支持自定义快捷键）

其中前两种都是可以明确获得文件名，而第三种无法获取文件名（因为剪贴板里有些图片比如截图根本就不存在文件名，这个问题后面会有解决方案🐡）

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-cacc481080cf629e)

image

接着上传完成之后，我们到相册就会看到上传成功后的照片了，另外在自己的仓库里，也能够看到上传的图片

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-39d26cd823799ead)

image

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-a8cf744910d6f980)

image

接着点击复制时，你会发现，它会给我们复制为 *Markdown* 的样式，如果你是在写 *Markdown* 的话，就不再需要再转化格式了🍀

------

### 其他功能

有时候在上传照片前，我们想重新命名这个图片名称，或者是在上传之后我们需要给图片重新命名，这款工具就可以快速满足我们的需要，支持编辑相册的图片信息，同时更好更方便的管理我们的图片。

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-c94e359ef72dccba)

image

工具默认上传前不做重命名，如果需要在上传前重新命名，我们可以到 PicGo设置中进行设置，把上传前重命名开关打开即可。

![img](E:\GitBook\other\图床工具的使用---PicGo.assets\15194389-58309d69ded7449a)

image

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-b4f5cd1e15769c20)

image

与此同时，除了单张上传外，其实还支持多张，批量上传，只需要选中多个文件，将其导入，或者拖入其中便可以了，这样再多的文件，也能够做到一次上传就能用的效果。

#### 支持查看当前上传的图床

如果你设置了除 *GitHub* 以外的图床，而不清楚当前上传到了哪个图床时，没关系，在上传区能够看到这个图床是哪一个

![img](https://gitee.com/nlpleaf/PicGo/raw/master/15194389-b2e9dbd146850f3d)

image

另外还支持开机自动启动，要是你觉得每次开机都要手动启动 PicGo 是一件麻烦的事情，不妨试试让他开机自启吧😄～

PS：忍不住吐槽一下，七牛云免费一年过后存放的图片链接就会过期，因此可以考虑换一个图床，选择一个更稳定持久的图床使用

------

今天的工具介绍就到这啦，如果你有什么好用的工具也可以在下方留言给我们推荐🐸