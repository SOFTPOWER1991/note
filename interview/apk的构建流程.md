App的开发完成后，需要打包并上线！我打包的过程一般都是用Android Studio提供的图形化工具进行打包。那么这个打包的原理是什么呢？

先看一下一个Android工程里面都涉及到哪些资源文件：

> 1. 资源文件，指res下的图片、xml文件
> 2. 代码文件：.aidl 、java文件 、c文件、c++文件
> 3. 引用的库：so文件，jar文件
> 4. 第三方module

这么多的文件，是如何打包的一个apk里面来的呢？下面进行一个探究，看Android 官方文档给的一个打包流程图：

![process_2x.png](https://github.com/SOFTPOWER1991/note/blob/master/raw/build-process_2x.png)

根据官方文档对这个流程的结束，我理解后做如下转述：

1. 编译器将我们自己的Android工程下的源码文件、资源文件、aidl文件 和 依赖的module库文件、AAR库、jar库进行编译，所有源码编译后输出DEX文件，所有其他内容转换为已编译资源。
2. 打包器将DEX文件和已编译资源以及keystore文件合并成单个的apk。
3. 打包器对apk进行签名。如果是调试版本的apk用debug keystore进行合并，如果是release版本的apk用release keystore进行合并。
4. 在生成最终的apk之前，使用zipalign工具对apk进行优化，减少在设备上运行时占用的内存。

整个过程结束后，我们就可以获得debug版本的apk 或者是release版本的apk了。

那么问题来了：

1. 在这个构建流程中用到的那些工具呢？比如：编译器是什么、打包器是什么？

在网上搜索资料，大部分的博客都会提到这个图片，但是我怎么也找不到这两个图片的来源。姑且按照这两张图片来看来解释上面的问题：

apk构建概览图：

![android_build.png](https://github.com/SOFTPOWER1991/note/blob/master/raw/android_build.png)

apk详细构建图：

![android_build_detail.png](https://github.com/SOFTPOWER1991/note/blob/master/raw/android_build_detail.png)

# 在这个构建流程中用到的那些工具呢？

在图中我看到了涉及到的工具有如下几个：`merger工具`、`aapt`、`AIDL工具`、`apkbuilder`、`javac`、`proguard`、`dex`、`Jarsigner`、`zipalign`.....

经过查找我发现这些工具位于sdk目录的`build-tools`下：

```
/Users/zhanggeng/android_tools/android_studio/android-sdk-macosx/build-tools/25.0.0
```

根据图中的流程，我来做如下解读：

## Merger、aapt(Android Asset Packaging Tool)工作流程

> 1. `Manifest Merger`将项目中的所有`Manifest`文件合并输出至`Merged Manifest`文件；
> 2. `Resources Merger`将项目中的所有`Resources`文件合并输出至`Merged Resources`文件；
> 3. `Assets Merger`将项目中的所有`Assets`文件合并输出至`Merged Assets`文件；
> 4. `aapt` 把这些`merged`文件合并后输出文件：`R.java` 和 `Compiled Resources`文件；

## AIDL转为Java文件的流程

> 这个过程将项目中用到的aidl文件转为Java文件

## javac编译.java文件为.class文件

> javac编译器将 .java 文件编译为.class文件

## ProGuard 代码优化过程

> ProGuard 会检测和移除封装应用中未使用的类、字段、方法和属性，包括自带代码库中的未使用项（这使其成为以变通方式解决 64k 引用限制的有用工具）。ProGuard 还可优化字节码，移除未使用的代码指令，以及用短名称混淆其余的类、字段和方法

## Dex 命令工作过程

> 将项目中所有的.class文件进行打包生成classes.dex文件

## apkbuilder 工作过程

> apkbuilder 将 classes.dex文件和 上面的`Compiled Resources`文件打包成一个未签名的`unSigned apk`

## JarSigner工作过程

> JarSigner，把提供的keystore 和 unSigned apk进行签名

## zipalign工作流程

> 将签名后的apk进行对齐处理。使用它的优点就是：减少程序运行时对内存的占用。


# 参考资源

1. [配置构建](https://developer.android.com/studio/build/index.html?hl=zh-cn#build-config)
2. [Android APK打包流程](http://shinelw.com/2016/04/27/android-make-apk/)
3. [Android APK 编译打包流程](https://segmentfault.com/a/1190000008071324)
4. [也说 Android Apk 打包](http://leenjewel.github.io/blog/2015/12/02/ye-shuo-android-apk-da-bao/)
5. [压缩代码和资源](https://developer.android.com/studio/build/shrink-code.html?hl=zh-cn)
6. [改善android性能工具篇【zipalign】](http://blog.csdn.net/zhixiang2010/article/details/17463095)
7. [zipalign](https://developer.android.com/studio/command-line/zipalign.html)



