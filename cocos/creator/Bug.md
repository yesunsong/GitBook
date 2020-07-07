

# Editor.Project.path

警告'Editor.projectInfo.path' has been deprecated, please use 'Editor.Project.path'.



原因：

内部插件发出的警告，忽略就好.

---

# xcodebuild

编译ios项目报错：

> xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance



原因：

更新xcode后无法确定路径。一句命令搞定：

> sudo xcode-select --switch /Applications/Xcode.app（后面的地址直接打开程序把Xcode往这里拖即可)。



如果以上方法不行可以尝试如下方法。

2. 执行

> xcodebuild -showsdks
>

报错如下：

> xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance



3. 执行

> sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
>

切换到当前Xcode路径下



4. 执行

>  xcodebuild -showsdks
>

![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200628225359.png)



![img](https://gitee.com/nlpleaf/PicGo/raw/master/20200628183511.png)



5. 执行

>  xcrun --sdk iphoneos --show-sdk-path
>

结果

- [ ] > /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS11.2.sdk
  >



6. 运行脚本命令成功



---

# cc.Node无法继承

写了一个类继承自cc.Node，然后再添加到其他节点里。可以运行的起来，但不会显示。

原因是：cc.Node不能继承，坑

---

# 全局GlobalZOrder

目前我们不支持 setGlobalZOrder，setLocalZOrder 和 siblingIndex 都只是支持同级节点的关系，如果要跨层级，那就肯定要自己调整节点树了。

GlobalZOrder 的接口继承自 Cocos2d-js，不过目前在 Creator 中确实没有实现。没有实现的主要原因是这个功能的坑很深，不仅会增加引擎的损耗和逻辑复杂度，使用不当对用户游戏也会产生副作用，所以我们没有想好要不要实现这个功能，毕竟需要用到 GlobalZOrder 的需求总是可以通过别的方式实现。

你希望达到的功能可以通过别的方式实现，比如在 LightRedA 和 LightRedB 上单独添加黑色遮罩。或者修改 LightRedA 的颜色。或者建立两个图层，一个在 Mask 上，一个在 Mask 下，节点在两个层之间相互切换。【https://forum.cocos.org/t/bug-setglobalzorder/37611】

---

# 怎么恢复被删除了的预制体

* 只能通过svn等版本管理器或者其他备份恢复
* 如果是使用Webstorm有一个很好的功能，每个文件都有history记录，它会记录每一次修改，大概会存好一两周内容：
  ![img](https://forum.cocos.org/uploads/default/original/3X/7/6/761c39baceb5122c25384006daad3fe1e2478613.png)
  目录也可以查看history，会显示每个文件的修改、删除情况，可以与当前文件内容做对比。