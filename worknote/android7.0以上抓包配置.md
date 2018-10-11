之前PHP的同学碰到了Android 7.0上无法抓包的问题，当时帮他处理了下。

今天前端同学碰到一个问题：Android 7.0以上无法抓包。顺手帮处理下，总结再次。

**当同一个问题反复出现的时候，当引起足够的重视。说明，这个问题一定是共性。那么，还会有再次碰到的时候。于是，我机智的把这个问题的解决方案总结了下来。下次再有人碰到这个问题，直接把这个文章扔给他就行。**

**来，照着做就行！！！**

开始之前需要简单介绍一下抓包相关的内容。

#### 什么是抓包？
> 不同主机之间的数据通信都是通过网络来进行传输，对那些在网络上传输的数据（发送、请求的数据）进行截获、编辑、转存等操作叫做抓包。抓包可以是抓取电脑端请求的数据，还可以抓取移动端（手机APP）的数据包。

#### 为什么要进行抓包？
> 通过对网络上传输的数据进行抓取，可以对其进行分析，对于软件的Debug很大的帮助。当然也可以通过抓取用户发送的涉及用户名和密码的数据包来获取用户的密码。

#### 抓包的技术原理是什么?
> 一般情况下，数据按照各种网络协议按照一定的格式在网络上进行传输，网络上传输的数据是以帧为单位，在对需要发送的数据进行包装的时候，会把数据的接收方、发送的的地址（MAC地址、IP地址等）一起进行包装并进行发送。根据发送方和接收方的地址，会有一条数据包的传输路径，在这条路径上，发送的数据包，会经过网络上很多台主机，标准的TCP/IP协议是这样处理的：当有数据经过主机时，主机会通过存放在数据包里面的地址来进行判断，这个数据包是否是发送自己的，如果不是发给自己的，主机就不会对它进行解析，简单的进行丢弃（转发）。如果是发送给自己的，那么主机就会对其进行解析和存储。

> 如果想要存储那些不是发送给自己的数据包，可以把网络适配卡设置为杂乱模式。这样它就会接收经过它的每一个数据包了。

回到同学们碰到的问题上来：

#### Android7.0 以上为什么无法进行抓包了？
> 直白点说：Android 7.0 以上Google对安全进行了更加严格的限制，不再信任用户安装的证书。

如何绕开Google的安全策略限制？大致流程是酱紫的: 
* 在Mac端，对Charles进行Proxy Settings...的配置 and SSL Proxy Settings...的配置；
* 在Mac端生成Charles Root Certificates（下面有用）
* 在将上一步生成的证书放在源码中；
* 将上一步生成的证书安装到手机上；

大致经过上面这几部就可以愉快的抓包了。

### 一、Mac端的Charles配置

**抓取Http的配置，设置Proxy Settings...**
> Proxy --> Proxy Settings --> Port，添加一个端口号：8888

**抓取Https的配置，设置SSL Proxying Settings...**

> Proxy --> SSL Proxying Settings --> "勾选 Enable SSL Proxying"，点击下方的add按钮，添加这样的配置：

Host是你要抓的域名或者ip（这里用通配符* ，表示抓取所有的Https请求），端口号port为443.

### 二、 下载手机上安装的SSL证书

**help--> SSL Proxying --> Save Charles Root Certificate...** ，然后选择目录，保存一个`charles-ssl-proxying-certificate.pem`的文件

### 三、将上一步下载的证书安装到手机上

将charles-ssl-proxying-certificate.pem，拷贝到手机上。找到它，然后安装上就可以了。

### 四、配置代理的IP 

Android手机的网络代理一般在设置页面，点击当前连着的WiFi，手动修改代理配置，然后将你的电脑ip和第一步中配置的端口号填写到手机上即可。

在Android 7.0以下这些操作就可以进行抓包了，Android 7.0以上你可能还需要一些额外的配置：

1. 在主工程目录下添加刚才下载的证书，路径为`app/res/raw`
2. 在主工程目录下添加文件夹，以及7.0对应的安全配置，路径为`app/res/xml`
3. 在AndroidMenifest.xml的Application节点下添加上一步添加的配置`android:networkSecurityConfig="@xml/network_security_config"`

安全配置文件network_security_config.xml内容如下：
```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">ethercap.com</domain>
        <trust-anchors>
            <certificates src="@raw/charlesrui"/>
        </trust-anchors>
    </domain-config>
</network-security-config>
```

* domain下的ethercap.com为要抓包的网站的域名
* certificates 为刚才放在raw下的证书

按照上面的步骤Step by Step 便可以成功拦截数据请求了。

**注意事项：**

1. 每个证书包含了电脑的信息，想要成功抓包需要手机连接生成证书的那个电脑才可以。

**参考文章：**

1. [Network Security configuration](https://developer.android.google.cn/training/articles/security-config)
2. https://blog.csdn.net/luochoudan/article/details/72801573
