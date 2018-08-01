前情概要：

`Kotlin`: Android 开发的官方指定编程语言，与2017年被Google扶正上位；
> Kotlin最近我把他引入了我们的项目开发中；

`ARouter`: Alibaba 开源的页面路由框架，其中一个功能是：支持多模块工程使用，以解决不同页面之间的跳转问题；
> ARouter作为组件化开发中，不同模块沟通的桥梁；


记录今天碰到的一个问题：

> 今天做一个需求：人脉拓展。
> 两个module : IM 、Search。
> 
> 发起跳转请求的页面位于：IM module中； 
>目标地址位于Search包中。

> IM Module已经支持Kotlin，而Search 中的人脉拓展也使用了Kotlin进行开发。

按照以往的路由跳转配置之后，以为可以进行跳转了，试验结果————跪了。

百思不得其姐，啊啊啊！！！

怀疑的问题：

1. ARouter不支持Kotlin？ **想到是这个原因，我菊花一紧，不由全身哆嗦**
2. 我配置的没问题，代码有问题？**检查代码，发现没啥不对的**
3. Kotlin中使用ARouter，我配置的问题？

想到第三种可能，遂，移步ARouter的GitHub主页：https://github.com/alibaba/ARouter。

从上拉倒下，整个网页扫了一眼，快速定位到了`Kotlin`，然后到了一下描述：

> **完全支持Kotlin以及混编(配置见文末 其他#5)**
> 
> **Kotlin项目中的配置方式**
> 
> **Kotlin类中的字段无法注入如何解决？**

瞬间——紧紧的菊花松开了，基本方向已经定位。

按照文档配置：

```
// 可以参考 module-kotlin 模块中的写法
apply plugin: 'kotlin-kapt'

kapt {
    arguments {
        arg("moduleName", project.getName())
    }
}

dependencies {
    compile 'com.alibaba:arouter-api:x.x.x'
    kapt 'com.alibaba:arouter-compiler:x.x.x'
    ...
}
```

碰到的问题：

1. 之前是集成了apt插件的，再次集成kapt的时候，爆出编译错误，解决方案：**删掉原来的apt，改用kapt**
2. 在Kotlin源码文件中不能使用原有的Java形式的路由注解，得改用`@Router(path = "/xxxx/xxxx")` 的形式

后续要做的事儿：

> 升级ARouter框架的版本

