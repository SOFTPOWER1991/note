### GT 是什么？

> GT 腾讯移动品质中心开源的一个工具集，是一个可以直接在手机上运行的“集成调测环境”。

### GT 用来解决什么问题？
> 
> 1. 对APP进行快速的性能测试，包括：CPU、内存、流量、电量、帧率、流畅度等等
> 2. 开发日志的查看
> 3. Crash日志查看
> 4. 网络数据包的抓取
> 5. APP内部参数的调试
> 6. 真机代码耗时统计
> 

**最爽的地方：代码都是开源的。如果感觉GT提供的功能不够，还可以利用GT提供的基础API开发满足我们需求的功能。**

### 支持的平台：

GT支持iOS和Android两个手机平台，其中：

* Android版由一个可直接安装的`GT控制台APP` 和 `GT SDK`组成,GT控制台可以独立安装使用，SDK需嵌入到被调试的应用、并利用GT控制台进行信息展示和参数修改。

* iOS版是一个Framework包，需要嵌入到APP工程，编译出带GT的APP才能使用；iPhone、iPad都能支持（我没验证过）


### GT 的使用步骤：

1. 安装GT SDK：
	
	> * 将 `gt-sdk-3.1.0.jar` 以及 `jniLibs下的相应native`库引入到项目工程中
	> * 在Application中加入 `GTRController.init(getApplicationContext());`  就完成了GT SDK 的安装

2. 打开手机GT控制台，选择嵌入了GT SDK 的APP。
3. 启动嵌入了SDK的APP，进行一波操作之后，就可以看到相应的检测数据了。
4. 导出数据 data.js 在pc浏览器上可以生成一个详细的检测报告。【优测性能分析】

**没有后台，完全是本地**

使用文档如下:

//TODO: 使用文档  or  video 

1. API文档；
2. 使用文档，是搞一个文档，还是录制一个video。

测试报告的解读：

1. 基础性能说明；

https://github.com/Tencent/GT/blob/master/android/GT-%E5%9F%BA%E7%A1%80%E6%80%A7%E8%83%BD%E8%AF%B4%E6%98%8E.md

2. 布局检测原理以及规则

https://github.com/Tencent/GT/blob/master/android/GT-%E5%B8%83%E5%B1%80%E6%A3%80%E6%B5%8B%E5%8E%9F%E7%90%86%E5%8F%8A%E8%A7%84%E5%88%99.md

流畅度检测——View绘制深度相关知识——hook ViewGroup.dispatchDraw方法。

3. 流畅性检测原理以及规则

https://github.com/Tencent/GT/blob/master/android/GT-%E6%B5%81%E7%95%85%E6%80%A7%E6%A3%80%E6%B5%8B%E5%8E%9F%E7%90%86%E5%8F%8A%E8%A7%84%E5%88%99.md

4. 页面启动时长检测原理以及规则

https://github.com/Tencent/GT/blob/master/android/GT-%E9%A1%B5%E9%9D%A2%E5%90%AF%E5%8A%A8%E6%97%B6%E9%95%BF%E6%A3%80%E6%B5%8B%E5%8E%9F%E7%90%86%E5%8F%8A%E8%A7%84%E5%88%99.md




