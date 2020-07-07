通过 CC_JSB 来判断是否为 native 环境（模拟器）。
		通过 cc.sys.isMobile 来判断是否为手机环境。



因为 CC_JSB 是一个预定义的宏，在编译时会利用这个宏进行分支和布尔运算的静态裁剪，能够获得更高的性能和更小的包体。





CC_EDITOR、CC_DEV等预设全局变量的说明在engine/predefine.js里。



错误信息的提示在 engine/DebugInfos.json里

