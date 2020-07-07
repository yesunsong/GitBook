# iOS 12.4 无法真机测试 Could not locate device support files （Could not find Developer Disk Image）

> iOS 升级到11之后，你会发现无法进行真机测试了。这种情况我在iOS 10.0更新的时候也遇到过。原因是Xcode 的DeviceSupport里面缺少了对应iOS系统版本的SDK。所以你可以选择将Xcode更新到最新版本。但是Xcode的更新速度很慢，快的时候一两个小时，慢的时候可能要一两天。而从网盘里面下载Xcode更是不可行，教训我们已经见识过了。
> 另外一个办法就是，不是缺iOS SDK的支持文件吗，我们直接把缺的文件导进去不就可以了吗？

### 步骤如下：

- 打开Finder
- 找到应用程序文件夹
- 在里面找到XCode
- 点击XCode，右键，显示包内容
- Contents-->Developer-->Platforms-->iPhoneOS.platform-->DeviceSupport
  /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport<
- 然后你就能看到你的Xcode支持的真机测试的一些系统型号对应的文件

- 把你找到的 对应手机系统版本文件夹拖进去就可以了。



# 可用的链接

13.5

链接:https://pan.baidu.com/s/1KYEQO1rBPSaAr2bYobGOvA  密码:dl86

12.3

链接:https://pan.baidu.com/s/1teH7geg7n_TfIRut3xHVrQ  密码:yvxx

12.4

链接:https://pan.baidu.com/s/1Puk1yNjBBfyKMBXP5_okzg  密码:ngez

12.4  最新版

链接:https://pan.baidu.com/s/1TUNcgplIUSzckHykcw3UDw  密码:0t0k

13.0  13.1

链接:https://pan.baidu.com/s/1NJMeOqNnTDne-Nj4Jso13A  密码:lzaz


链接：https://www.jianshu.com/p/5a6a5e34bf85