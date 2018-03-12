安装包越大，用户下载的门槛就越高，特别是在移动网络情况下，用户在下载应用时，对安装包大小的要求更高。

1. 下载门槛高；
2. 占用带宽成本；
3. 安装包越小用户越容易下载体验



1. 代码混淆
2. 资源优化：使用Android Lint 删除冗余资源
3. 资源文件最少化
4. 图片优化
5. 避免重复功能的库（代码规范）
6. 使用WebP图片格式
7. 插件化

# APK瘦身——师出有名

apk是 Android package

不妨这样来想这个问题——如果我的apk过大将会造成什么样的后果？

1. 摆在我面前两个选择，一个100M 一个10M的apk，我肯定会选择10M的。apk提及越大下载话费的流量就会越多，省流量，哪有那么多流量供你更新apk呢。
2. 一个好几百兆的游戏，比如王者荣耀一个游戏500M，下载时间都要话费好久，apk将会带来很长的安装时间。将会给用户带来很长的下载，安装等待时间，这个得~~~等。
3. 用户手机磁盘有限，如果肆无忌惮的占用用户磁盘，打开应用列表一看，一个apk占了好几百兆一定灰常不美丽。

# apk构建原理

这里参考Google官方文档：https://developer.android.com/studio/build/index.html?hl=zh-cn#build-config

工程图：//TODO: 此处应该要有图

单反是个Android工程一定会包含如下内容：

代码文件、图片、布局文件、各种resource资源、native .so库、引用的AAR、jar包

如何将这些文件组装成一个可以再用户手机上运行的Android应用呢？

典型的Android应用模块的构建流程：

其中涉及到两个家伙：

Compilers: 编译器将Android工程源代码文件转换成DEX文件（其中包括可以运行在Android设备上的字节码：//TODO: DEX文件是什么？）,将其他内容转换为已编译的资源。

> 被Compilers编译的内容包括：源代码、资源文件、AIDL文件，和依赖的第三方库的jar，AAR，library modules

apk packager : 

> apk打包器把DEX文件和已编译资源合并成单个apk。需要使用keystore 签名文件进行签名之后，才能部署到设备上（为什么需要签名才能部署到手机上）？

apk打包器使用调试或发布keystore签署您的apk。

在生成最终apk之前，打包器会使用zipalign工具对应用进行优化，减少在设备上运行时的内存占用。

构建流程结束，我们就会获得一个可以进行部署、测试的apk。

===

http://jayfeng.com/2016/03/01/Android-APP%E7%BB%88%E6%9E%81%E7%98%A6%E8%BA%AB%E6%8C%87%E5%8D%97/

classes.dex ：

1. 代码混淆

2. 删掉没用的代码：Android Studio--> Inspect Code
3. 一个apk尽量只有一套图片，建议放在xhdpi下
4. 图片质量，使用webp图片格式
5. 微信提供的资源文件混淆工具对资源文件做混淆AndResGuard  https://github.com/shwenzhang/AndResGuard/blob/master/README.zh-cn.md
6. 使用tinypng对图片资源进行压缩：https://tinypng.com/


lib库文件：

只提供针对主流架构的支持，比如ARM，对于mips和x86架构可以考虑不支持。


res资源：

资源的动态下发：比如APP中的聊天表情图片，可能有几屏的表情：
> 1. 如果你不发表情，那么这100个图片表情，打包进APk的时候，会白白占用资源；
> 2. 字使用的时候进行请求

专区图片：动态下发



===

借助Google play对不同版本，不同分辨率的手机提供不同的安装包 




=====
https://developer.android.com/studio/build/shrink-code.html?hl=zh-cn#shrink-code

代码压缩：ProGuard

资源压缩：Gradle，android提供的插件

**压缩代码：**

android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}

亮点需要注意的：

1. 开启minifyEnabled ,
2. 编写Proguard 文件
3. 保留特定的代码：--keep public class MyClass   https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html

**压缩资源**

资源压缩器只与代码压缩协同工作。代码压缩器移除所有未使用的代码后，资源压缩器便可确定应用仍然使用的资源。

要启动资源压缩，在build.gradle文件中将shrinkResources属性设置为true

android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}

因此这两个需要配合使用，需要配合开启。

自定义要保留的资源：

在项目中创建一个包含<resources>标记的xml文件，并在tools.keep中指定每个要保留的资源。在 tools:discard 属性中指定每个要舍弃的资源。这两个属性都接受逗号分隔的资源名称列表。您可以使用星号字符作为通配符。

<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
    tools:discard="@layout/unused2" />

移除未使用的备用资源：

可以利用 APK 拆分为不同设备构建不同的 APK，自定义在 APK 中包括的屏幕密度或 ABI 资源。
apk拆分：https://developer.android.com/studio/build/configure-apk-splits.html?hl=zh-cn


第4： 删除未使用的备用，无用的预言资源：

ndroid {
    defaultConfig {
        resConfigs "zh"
    }
}

5. 使用tinypng有损压缩

Android打包的时候本身对png进行无损压缩，所以使用像tinypng这样的有损压缩很有必要，Tinypng使用智能有损压缩技术，以尽量少的失真换来图片大小的锐减，效果非常好，强烈推荐。

tinypng的官方网站：http://tinypng.com/

6. 使用jpg格式的图片：

如果对于非透明的大图，jpg将会比png的大小有显著的优势，虽然不是绝对的，但是通常会减小到一半都不止。
在启动页，活动页等之类的大图展示区采用jpg将是非常明智的选择。

第7条：使用webp格式

webp支持透明度，压缩比比jpg更高但显示效果却不输于jpg，官方评测quality参数等于75均衡最佳。 相对于jpg、png，webp作为一种新的图片格式，限于android的支持情况暂时还没用在手机端广泛应用起来。从Android 4.0+开始原生支持，但是不支持包含透明度，直到Android 4.2.1+才支持显示含透明度的webp，使用的时候要特别注意。 官方介绍：https://developers.google.com/speed/webp/docs/precompiled

https://zhuanlan.zhihu.com/p/23648251

第8条：

删除lib包下不用的资源

第9条：使用微信资源压缩打包工具

微信资源压缩打包工具通过短资源名称，采用７zip对APP进行极致压缩实现减小APP的目标，效果非常的好，强烈推荐。
详情参考：Android资源混淆工具使用说明
原理介绍：安装包立减1M–微信Android资源混淆打包工具
建议开启7zip，注意白名单的配置，否则会导致有些资源找不到，官方已经发布AndResGuard到gradle中了，非常方便：

第10条：使用shape背景：

有些图片是可以用代码画出来的。shape

第11条：支持插件化

把APP的一部分分离出来，需要的时候再下载。

第12条：Facebook的redex优化字节码

redex是facebook发布的一款android字节码的优化工具，需要按照说明文档自行配置一下。

https://github.com/facebook/redex

表格

http://jayfeng.com/2016/03/01/Android-APP%E7%BB%88%E6%9E%81%E7%98%A6%E8%BA%AB%E6%8C%87%E5%8D%97/

https://github.com/OpenDevTeam/OpenBox/blob/master/topic/%5BAndroid%E6%8A%80%E6%9C%AF%E4%B8%93%E9%A2%98%5DAPK%E7%98%A6%E8%BA%AB%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B0%B1%E5%A4%9F%E4%BA%86.md

第12条：

使用VectorDrawable 和 SVG 图片来替换原有图片

====

1. 使用一套资源：

	对于大多数APP来说，只需要去一套设计图就足够了，取720的放置在xhdpi下

2. 开启minifyEnabled 混淆代码

 	在gradle使用minifyEnabled进行Proguard混淆的配置，可大大减小APP大小
 	
 	android {
	    buildTypes {
	        release {
	            minifyEnabled true
	        }
	    }
   }
   
   proguard 链接：https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html
   
   


======

如何来衡量前后两次优化的结果：

1. https://developer.android.com/studio/build/apk-analyzer.html


相关链接：
https://github.com/OpenDevTeam/OpenBox/blob/master/topic/%5BAndroid%E6%8A%80%E6%9C%AF%E4%B8%93%E9%A2%98%5DAPK%E7%98%A6%E8%BA%AB%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E6%96%87%E7%AB%A0%E5%B0%B1%E5%A4%9F%E4%BA%86.md


https://github.com/facebook/redex

https://zhuanlan.zhihu.com/p/23648251

https://zhuanlan.zhihu.com/p/20006066






