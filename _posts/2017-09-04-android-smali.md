---
title: android-smali
layout: post
guid: urn:uuid:9dd2d361-e41a-4e96-9823-4d3d971f7897
tags:
  - 
---


# 一次Android逆向的演练和思考
由于项目的需求，作者自己尝试逆向了一个APP去学习和了解基本的业务逻辑，在逆向过程中学习到了很多知识，本文对这些知识重新梳理总结，并且讨论作为一个APP的如何防止自己的产品被恶意逆向，保护自己的核心数据和知识产权。

为了防止不必要的麻烦，本文所使用 **目标APP**(指将要被逆向分析的APP,下同) 来自 [android-crackme-challenge](https://github.com/reoky/android-crackme-challenge.git)
### 0x00 关于逆向

#### _什么是逆向工程？_
> 逆向工程，又称反向工程，是一种技术过程，即对一项目标产品进行逆向分析及研究，从而演绎并得出该产品的处理流程、组织结构、功能性能规格等设计要素，以制作出功能相近，但又不完全一样的产品。逆向工程源于商业及军事领域中的硬件分析。其主要目的是，在不能轻易获得必要的生产信息下，直接从成品的分析，推导出产品的设计原理。 (维基百科)
#### _Is this just for hackers?_

逆向一个应用程序并非是黑客专利，或者说逆向技术并非是做一些'坏'事情。
>如果你是一名开发者，在很多情况下，你需要hack应用或者向其中注入代码。

> * 首先，应用中有一些你感兴趣的布局或者动画效果，使用这些工具你可以拿到对应的XML资源文件。
> * 其次，你可以看看本人的应用是如何实现具体的业务逻辑的：这并不是抄袭他人的代码，而是通过学习可以可以提高你的编码水平或者得到一些有用的改进提示。
> * 最后, 测试自己应用程序的安全性
> *    逆向工程可能会被误认为是对知识产权的严重侵害，但是在实际应用上，反而可能会保护知识产权所有者。例如在集成电路领域，如果怀疑某公司侵犯知识产权，可以用逆向工程技术来寻找证据。

### 0x01 目标介绍

开源项目 android-crack-me 提供了 一组 android-app 提供逆向练习，我们选取其中一个(crackme-one.apk)进行逆向。

* eb0a1916c742af44b3da7a6b82768e6548cf0d88 [crackme-five.apk](https://www.dropbox.com/s/u8bct350lo41xku/crackme-five.apk)
* d7ea8959af715083531d7e4729cf264fffce66b4 [crackme-four.apk](https://www.dropbox.com/s/4vumdeu0vlepdcf/crackme-four.apk)
* dae637b287d4256a614b941249cfb26111d49005 [crackme-one.apk](https://www.dropbox.com/s/mrjnme2xiv45j4g/crackme-one.apk) <- 我们用这个
* 01236c76c078083d3d27eb3e46825f19c2af5ffa [crackme-three.apk](https://www.dropbox.com/s/3yll2fglndiohyx/crackme-three.apk)
* e917d98be93278363fd2f57207b4f03adf14a910 [crackme-two.apk](https://www.dropbox.com/s/zqomglbit48iw5s/crackme-two.apk)

安装后程序界面:

![app](http://outngm0go.bkt.clouddn.com/17-8-17/48025932.jpg)

### 0x02 工具介绍

在逆向过程中合理的使用工具会达到事半功倍的效果，由于逆向的过程中技术广度较大，需要各个方面的工具，所以下面只列举几个常见的普遍的工具。

#### aapt

如果你安装了Android SDK，那么你就有aapt了。事实上，“Android Asset Packaging Tool”是android 构建工具中的一部分，你可以在以下路径中找到它，例如：``ANDROID_SDK_HOME/build-tools/23.0.2``

aapt是编译过程使用的一个工具，他的主要工作有：
* 生成R.java。 
* 将资源,manifest等装入APK。 
* 编译class，并且将classes转成dex。 

关于编译流程可见：[官方文档](http://developer.android.com/sdk/installing/studio-build.html)

以下是在逆向过程中比较常见``aapt``的使用方法：

``~/Library/Android/sdk/build-tools/26.0.0/aapt list -v crackme-one.apk``
![aapt-listv](http://outngm0go.bkt.clouddn.com/17-8-17/55648236.jpg)

``~/Library/Android/sdk/build-tools/26.0.0/aapt dump badging crackme-one.apk``
![aapt-listv-dump](http://outngm0go.bkt.clouddn.com/17-8-17/67628208.jpg)

#### apktool

[主页](https://ibotpeaches.github.io/Apktool/)

APKTool是GOOGLE提供的APK编译工具，能够反编译及回编译apk，同时安装反编译系统apk所需要的framework-res框架，清理上次反编译文件夹等功能。它可以完整解包APK，解包后你可以看到 APK 里面的声明文件、布局文件、图片资源文件、由 dex 解包出来的 smali 文件、语言文件等。如果你要汉化、修改界面、修改代码的话，apktool 可以帮你一站式完成。

* 几乎原生的资源数据 （resources.arsc, classes.dex, 9.png. XMLs）
* 能够rebuilding 到 apk
* Smali debug (配合 [IdeaSmali](https://github.com/JesusFreke/smali/wiki/smalidea))

```
$ apktool d test.apk
I: Using Apktool 2.2.4 on test.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: 1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
$
$
$ apktool b test
I: Using Apktool 2.2.4 on test
I: Checking whether sources has changed...
I: Smaling smali folder into classes.dex...
I: Checking whether resources has changed...
I: Building resources...
I: Building apk file...
I: Copying unknown files/dir...
```

#### dex2jar

dex2jar是一个能操作Android的dalvik(.dex)文件格式和Java的(.class)的工具集合。包含以下几个功能
* dex-reader/writer：用于读写 DalvikExecutable (.dex) 文件格式. 包含一个简单的API(与ASM相似)；
* d2j-dex2jar：执行dex到class的文件格式转换；
* smali/baksmali：与smali工具功能一致，但是对中文更友好；

### 0x03 逆向过程
#### 1.APK整体分析

首先使用aapt对 crackme.apk 进行整体分析，以便对apk整体有个认识。

运行

``~/Library/Android/sdk/build-tools/26.0.0/aapt dump badging crackme-one.apk``

结果

```
基本信息 包名，各种sdkversion等
package: name='com.reoky.crackme.challengeone' versionCode='1' versionName='1.0' platformBuildVersionName=''
sdkVersion:'11'
targetSdkVersion:'20'
uses-permission: name='android.permission.VIBRATE'
application-label:'CrackMe One'
...省略部分无用代码...
启动的activity
launchable-activity: name='com.reoky.crackme.challengeone.activities.ChallengeActivity'  label='CrackMe One' icon=''
feature-group: label=''
uses-feature: name='android.hardware.faketouch'
uses-implied-feature: name='android.hardware.faketouch' reason='default feature for all apps'
main
supports-screens: 'small' 'normal' 'large' 'xlarge'
supports-any-density: 'true' 
```

#### 2.使用apktool 解包

运行    
``$ apktool d crackme-one.apk``

解包后的文件是这样的

![decompile](http://outngm0go.bkt.clouddn.com/17-8-17/45699172.jpg)

其中 apktool.xml 是该反编译项目的清单目录，如果想使用apktool打包一个app项目，那么就需要 apktool.xml文件存在
smali目录下，是所以得Java类的集合，由于一个smali文件只允许描述一个Java类，所以如果存在内部类,将会以 ``外部类$内部类.smali``的方式命名。

为了保证能继续分析下去，这里将通过几个Sample简单介绍Smali的用法，如果已经知道Smali相关的知识，可以选择[跳过](#tag0)
    
---
##### Smali语法简单介绍
Davlik字节码中，寄存器都是32位的，能够支持任何类型，64位类型（Long/Double）用2个寄存器表示；
Dalvik字节码有两种类型：原始类型；引用类型（包括对象和数组）

* 原始类型：

|签名|java中的类型|说明|
|:------:|:------:|:------:|
|V|void|只能用于返回值类型|
|Z|boolean||
|B|byte||
|S|short||
|C|char||
|I|int||
|J|long|64位|
|F|float||
|D|double|64位|
|[|数组| ``[I`` -> ``int[]``, ``[[Z`` -> ``boolean[][]``|
    
* 类声明:

   ``L`` + ``使用'/'分割的包名`` + ``类名`` + ``;``

    例如: ``Ljava/lang/String;`` , ``LHelloWorld;``

* 方法声明：
    
    ``类声明`` + ``->`` + ``方法名(参数表)`` + ``返回类型``
    
    例如： ``java/lang/Object;->toString()java/lang/String;``

* 字段声明:
   
    ``Lpackage/name/ObjectName;——>FieldName:Ljava/lang/String;``

* 寄存器声明:

    * 在dalvik的字节码中，寄存器总是32位，可以保存任何类型的值。 2个寄存器用于保存64位类型。

    * 有两种方法可以指定方法中有多少个寄存器可用。 ``.registers``指令指定方法中的 **寄存器总数**，而``.locals``指令指定方法中 **非参数寄存器** 的数量。 

    * 当调用一个方法时，该方法的参数被放入最后的n个寄存器中。如果一个方法有2个参数和5个寄存器（v0-v4），则这些参数将被放置到最后2个寄存器v3和v4中。

    * 非静态方法的第一个参数始终是该方法被调用的对象。


|local|param|其他|
|:---:|:---:|:---:|
|v0||第一个本地寄存器，如果该方法不是静态方法，那么v0是该方法被调用的对象|
|v1||第二个本地寄存器|
|v2|p0|第三个本地寄存器，同时也是第一个参数寄存器|
|v3|p1|第四个本地寄存器，同时也是第二个参数寄存器|
|v4|p2|第五个本地寄存器，同时也是第三个参数寄存器|

64位调用：

``LObject;->method(IZ)V;``

假设该方法没有声明额外的寄存器那么它的寄存器结构将会是：

|local|param|
|:---:|:---:|
|v0|``this``|
|v1|I|
|v2, v3|Z|



##### 我们先写Smali版本的Hello World快速熟悉Smali的语法

Java Hello World
```
public class HelloWorld {
      public static void main(String[] args) {
           System.out.println("Hello World!");
      }
}
```

Smali Hello world

```
.class public LHelloWorld;

.super Ljava/lang/Object;

.method public static main([Ljava/lang/String;)V

  .registers 2

  sget-object v0, Ljava/lang/System;->out:Ljava/io/PrintStream;

  const-string v1, "Hello World!"

  invoke-virtual {v0, v1}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V

  return-void

.end method

```

| 指令|解释|
|:---:|:--:|
|``sget-object v0, Ljava/lang/System;->out:Ljava/io/PrintStream;``| 将静态对象引用 ``out`` 赋值于 ``v0`` |
|``const-string v1, "hello world" `` | 将 ``v1``赋值为 ``hello world ``|
|``invoke-virtual {v0, v1}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V`` | 调用实例 ``v0``的方法并且将``v1``作为参数|

更多常见指令可以参考
[这里](https://source.android.com/devices/tech/dalvik/dalvik-bytecode)
    
---

<span id = "tag0"> 继续 </span> 逆向 crackme,
为了方便阅读，我们使用IdeaSmali插件，使用AndroidStudio阅读，他看起来想这个样子：
![ideaSmali](http://outngm0go.bkt.clouddn.com/17-8-17/60336741.jpg)

为了方便动态调试，我们生成一个可以断点断点调试版本的APK：

找到Manifest，中Application的节点，添加一个debuggale的属性，并把属性值设为``true``,

![](http://outngm0go.bkt.clouddn.com/17-8-17/32836211.jpg)

使用 ``apktool b``打包并签名安装到手机即可。

![](http://outngm0go.bkt.clouddn.com/17-8-17/37604251.jpg)
    

根据aapt的分析，我们知道启动的Activity是 ``ChallengeActivity``, 所以我们从 ``ChallengeActivity`` 开始
代码如下:
```
.class public Lcom/reoky/crackme/challengeone/activities/ChallengeActivity;
.super Landroid/support/v4/app/FragmentActivity;
.source "ChallengeActivity.java"

# interfaces
.implements Landroid/app/ActionBar$TabListener;

....

# virtual methods
.method public onCreate(Landroid/os/Bundle;)V
    .locals 4
    .param p1, "savedInstanceState"    # Landroid/os/Bundle;

    .prologue
    const/4 v3, 0x2

    ...
    invoke-virtual {v0, v1}, Landroid/support/v4/view/ViewPager;->setAdapter(Landroid/support/v4/view/PagerAdapter;)V

	...
    const-string v2, "Challenge"

    invoke-virtual {v1, v2}, Landroid/app/ActionBar$Tab;->setText(Ljava/lang/CharSequence;)Landroid/app/ActionBar$Tab;

    move-result-object v1

    invoke-virtual {v1, p0}, Landroid/app/ActionBar$Tab;->setTabListener(Landroid/app/ActionBar$TabListener;)Landroid/app/ActionBar$Tab;

    move-result-object v1

    invoke-virtual {v0, v1}, Landroid/app/ActionBar;->addTab(Landroid/app/ActionBar$Tab;)V

    .line 39
    iget-object v0, p0, Lcom/reoky/crackme/challengeone/activities/ChallengeActivity;->actionBar:Landroid/app/ActionBar;

    iget-object v1, p0, Lcom/reoky/crackme/challengeone/activities/ChallengeActivity;->actionBar:Landroid/app/ActionBar;

    invoke-virtual {v1}, Landroid/app/ActionBar;->newTab()Landroid/app/ActionBar$Tab;

    move-result-object v1

	...

.method public onTabSelected(Landroid/app/ActionBar$Tab;Landroid/app/FragmentTransaction;)V
    .locals 2
    .param p1, "tab"    # Landroid/app/ActionBar$Tab;
    .param p2, "fragmentTransaction"    # Landroid/app/FragmentTransaction;

    .prologue
    .line 48
    iget-object v0, p0, Lcom/reoky/crackme/challengeone/activities/ChallengeActivity;->mViewPager:Landroid/support/v4/view/ViewPager;

    invoke-virtual {p1}, Landroid/app/ActionBar$Tab;->getPosition()I

    move-result v1

    invoke-virtual {v0, v1}, Landroid/support/v4/view/ViewPager;->setCurrentItem(I)V

    .line 49
    return-void
.end method

```

这里代码逻辑很简单，在顶部创建了一个tabhost，然后通过切换tabhost不同Fragment，由于另外两个Fragment:Hit
和About没有什么研究意义，所以我们直接去找相应的Fragment : CllangeOneFragment，主要逻辑都在下面，我们一一分析：

```
    .line 32
    .local v3, "view":Landroid/view/View;
    const v4, 0x7f070043

    invoke-virtual {v3, v4}, Landroid/view/View;->findViewById(I)Landroid/view/View;

    move-result-object v0

    check-cast v0, Landroid/widget/Button;

    .line 33
    .local v0, "buttonCheck":Landroid/widget/Button;
    new-instance v4, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;

    invoke-direct {v4}, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;-><init>()V

    invoke-virtual {v0, v4}, Landroid/widget/Button;->setOnClickListener(Landroid/view/View$OnClickListener;)V

    .line 36
    const v4, 0x7f070044

    invoke-virtual {v3, v4}, Landroid/view/View;->findViewById(I)Landroid/view/View;

    move-result-object v1

    check-cast v1, Landroid/widget/Button;

    .line 37
    .local v1, "buttonWriteFile":Landroid/widget/Button;
    new-instance v4, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;

    invoke-direct {v4}, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;-><init>()V

    invoke-virtual {v1, v4}, Landroid/widget/Button;->setOnClickListener(Landroid/view/View$OnClickListener;)V

    .line 41
    invoke-virtual {v3}, Landroid/view/View;->getContext()Landroid/content/Context;

    move-result-object v4

    const-string v5, "ANSWER"

    invoke-virtual {v4, v5}, Landroid/content/Context;->getFileStreamPath(Ljava/lang/String;)Ljava/io/File;

    move-result-object v2

    .line 42
    .local v2, "file":Ljava/io/File;
    invoke-virtual {v2}, Ljava/io/File;->exists()Z

    move-result v4

    if-eqz v4, :cond_0

    .line 43
    const v4, 0x7f0a0018

    invoke-virtual {v1, v4}, Landroid/widget/Button;->setText(I)V

    .line 48
    :goto_0
    return-object v3

    .line 45
    :cond_0
    const v4, 0x7f0a001d

    invoke-virtual {v1, v4}, Landroid/widget/Button;->setText(I)

```
这里声明了个寄存器来绑定界面上的button，比如这个 
```
    .line 32
    .local v3, "view":Landroid/view/View;
    const v4, 0x7f070043

    invoke-virtual {v3, v4}, Landroid/view/View;->findViewById(I)Landroid/view/View;

    move-result-object v0

    check-cast v0, Landroid/widget/Button;

```
通过```FindViewByID```方法
将id ``0x7f0a001d``的button引用移动到v0，那么这个ID的button是我们界面中的哪一个呢?
我们首先到R.java文件里面反查：
![](http://outngm0go.bkt.clouddn.com/17-8-17/71902617.jpg) 
我们可以通过本地XML文件对比，或者使用实时Layout查看的工具(比如[这个](http://blog.csdn.net/xyz_lmn/article/details/14222975))，将界面和代码逻辑一一映射出来。

![](http://outngm0go.bkt.clouddn.com/17-8-17/89073766.jpg)
找到了id与实际Button的对应关系后继续往下看:


```

    .line 33
    .local v0, "buttonCheck":Landroid/widget/Button;
    new-instance v4, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;

    invoke-direct {v4}, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;-><init>()V

    invoke-virtual {v0, v4}, Landroid/widget/Button;->setOnClickListener(Landroid/view/View$OnClickListener;)V

```

将第一个按钮(Check)设定了一个``Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener``的实例，这是一个简单的Onclick事件,我们稍后分析，我们先沿着Fragment的逻辑继续看下去。


```
    check-cast v1, Landroid/widget/Button;

    .line 37
    .local v1, "buttonWriteFile":Landroid/widget/Button;
    new-instance v4, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;

    invoke-direct {v4}, Lcom/reoky/crackme/challengeone/listeners/ChallengeOneFragmentOnClickListener;-><init>()V

    invoke-virtual {v1, v4}, Landroid/widget/Button;->setOnClickListener(Landroid/view/View$OnClickListener;)V

    .line 41
    invoke-virtual {v3}, Landroid/view/View;->getContext()Landroid/content/Context;

    move-result-object v4
```

第二个按钮(wirte/delete file)和第一个逻辑相同，看来是在这个listener内部通过View的id来区分他们，并且运行不同的逻辑。

```
    invoke-virtual {v4, v5}, Landroid/content/Context;->getFileStreamPath(Ljava/lang/String;)Ljava/io/File;

    move-result-object v2

    .line 42
    .local v2, "file":Ljava/io/File;
    invoke-virtual {v2}, Ljava/io/File;->exists()Z

    move-result v4

    if-eqz v4, :cond_0

    .line 43
    const v4, 0x7f0a0018

    invoke-virtual {v1, v4}, Landroid/widget/Button;->setText(I)V

    .line 48
    :goto_0
    return-object v3

    .line 45
    :cond_0
    const v4, 0x7f0a001d

    invoke-virtual {v1, v4}, Landroid/widget/Button;->setText(I)V
```

这段定义了第二个按钮的Text是 ``0x7f0a0018`` 还是 ``0x7f0a001d``

![](http://outngm0go.bkt.clouddn.com/17-8-17/4092513.jpg)
![](http://outngm0go.bkt.clouddn.com/17-8-17/47140987.jpg)

---

接下来看``ChallengeOneFragmentOnClickListener``:
省略次要代码，看关键代码：
```
    .line 35
    .local v6, "textGuess":Landroid/widget/EditText;
    invoke-virtual {v6}, Landroid/widget/EditText;->getText()Landroid/text/Editable;

    move-result-object v8

    invoke-virtual {v8}, Ljava/lang/Object;->toString()Ljava/lang/String;

    move-result-object v8

    invoke-virtual {v8}, Ljava/lang/String;->toLowerCase()Ljava/lang/String;

    move-result-object v8

    const-string v9, "poorly-protected-secret"

    invoke-virtual {v8, v9}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result v8

    if-eqz v8, :cond_1
```

如果TextGuess的字符串等于 ``poorly-protected-secret``的结果为 ``false``  那么跳到 ``cond_1``, 跟进 ``cond_1`` 

```
    :cond_1
    invoke-virtual {v5}, Landroid/view/View;->getResources()Landroid/content/res/Resources;

    move-result-object v8

    const v9, 0x7f060004

    invoke-virtual {v8, v9}, Landroid/content/res/Resources;->getColor(I)I

    move-result v8

    invoke-virtual {v6, v8}, Landroid/widget/EditText;->setTextColor(I)V

    .line 42
    invoke-virtual {v5}, Landroid/view/View;->getContext()Landroid/content/Context;

    move-result-object v8

    const-string v9, "Sorry, that\'s not right.."
``` 
输出Toast提示失败,
所以说如果输入的为 ``poorly-protected-secret`` 将会输出正确：

![](http://outngm0go.bkt.clouddn.com/17-8-17/59972589.jpg)

#### 逻辑总结

APP里有一段提示语，大致思路是他会将密文创建到程序的空间里(applicaiotn sanbox)，然后我们的任务是从sanbox里将密文取出来获取秘钥，通过分析程序我们发现
其实APP并没有将密文放在sanbox里面，而只是单纯的空文件，真正的验证是一个和 ``poorly-protected-secret`` 明文串作比较。

#### 更多

我们也可以修改他的判断逻辑,例如:

```

    const-string v9, "poorly-protected-secret"

    invoke-virtual {v8, v9}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result v8

	#关键判断
    if-eqz v8, :cond_1

```

将 ``if-eqz v8, :cond_1`` 改为 ``if-nez v8, :cond_1`` 或者 ``nop`` 后重新打包。

![](http://outngm0go.bkt.clouddn.com/17-8-17/48799448.jpg)

如图所示。

### 0x04 过程中遇到的问题

* ##### 为什么不直接反编译成Java，而是Smali

1. 若想修改APK代码, 从流程的复杂程度

Smali:

    1. 使用'apktool d'将apk解包
    2. 编辑Smali
    3. 使用'apktool b'打包apk
    4. 签名

Java:  

    1. 将 apk 文件解压,获取dex文件。 
    2. 将 dex 转成 jar 文件。
    3. 将 jar 里面的 classes反编译成 Java。
    4. 编辑Java。
    5. 编译 Java。
    6. 利用dx工具将jar转成dex。
    7. 合并 apk文件。
    8. 签名。

    如果APK里有多个dex，2-6步将重复多次。

导入Android Studio:

先看Android编译流程

![](http://outngm0go.bkt.clouddn.com/17-8-18/59362026.jpg)

当我们从dex里转出Java文件后,这里面不仅包含我们的 ``Application Source code``(包含 ``3rd party Libraries``)，还包含编译时期生成的R.java以及Java Interfaces。

这里存在的问题在于：

项目代码:

    //xxActivity.java
    ....
    View view = findViewById(R.id.button);
    ....


    //R.java
    .....
      public static class id {
        .....
        public static final int button = 0x12345;
        .....
    ....

    //xx.xml

    ...
    <Button
        android:id="@+id/button"
    ...
经过aapt处理：
    
    //xxActivity.java
    ....

    !!被一次性替换成真实的值!!
    View view = findViewById(0x12345);
    ....


    //R.java
    .....
      public static class id {
        .....
        public static final int button = 0x12345;
        .....
    ....

    //xx.xml

    ...
    <Button
        android:id="@id/button"
    ...

由于R.java 在每次build过程都会生成，所以要保证在AndroidStudio正常运行，需要关掉R.java生成的部分，或者将源代码的整形替换成R.id.xxx的形式，这是一个十分复杂操作。

当保留反编译过来的R.java会产生如下错误。
![](http://outngm0go.bkt.clouddn.com/17-8-18/72909682.jpg)
    
    
综上描述 **所以如果只是读源代码可以使用Java，如果是修改建议使用 Smali 。**

### 0x05 总结

通过逆向练习我们发现，如果不加以保护手段，我们的代码，关键数据都很容易被获取。
#### 如何防止自己的APP被修改
再次之前，引用一段来自Stack Overflow论坛的一个回答 [原帖](https://stackoverflow.com/questions/13854425/how-to-avoid-reverse-engineering-of-an-apk-file)

> 1. How can I completely avoid reverse engineering of an Android APK? Is this possible?

AFAIK, **there is not any trick for complete avoidance of reverse engineering.**

 And also very well said by @inazaruk: Whatever you do to your code, a potential attacker is able to change it in any way she or he finds it feasible. You basically can't protect your application from being modified. And any protection you put in there can be disabled/removed.

以下有几个常见的方式可以提高逆向的难度，从而尽可能的防止数据泄露等问题。

* 混淆(obfuscation)

     混淆是指使用短又没有语义的名字重命名非入口类的类名，变量名，方法名。入口类的名字保持不变。
Android Build tools 集合中提供了可以混淆代码的工具：ProGuard。

* 反调试


    //防止被trace
    void anti_ptrace(void)
    {
        ptrace(PTRACE_TRACEME, 0, 0, 0)；
    }

    //这为了防止破解者在应用的AndroidManifest.xml中添加：android:debuggable=true属性   
    public static boolean isDebuggable(Context context) {
            return (context.getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0;
    }


* dex加壳
* 防止资源文件篡改
* 防止二次打包 
    
    hash校验
    ![](http://outngm0go.bkt.clouddn.com/17-8-18/2288754.jpg)

* so文件保护
* 内存保护
    防止从内存中dump dex

### 0xFF 参考
技术：
* [android reverse engineering 101 (part 1)](http://www.fasteque.com/android-reverse-engineering-101-part-1/)
* [逆向工程wiki](https://zh.wikipedia.org/wiki/逆向工程)
* [26款优秀的Android逆向工具](http://www.freebuf.com/sectool/111532.html)
* [Smali语法介绍](http://blog.csdn.net/singwhatiwanna/article/details/19019547)
* [Smali github wiki](https://github.com/JesusFreke/smali/wiki)
* [ProGuard](https://www.guardsquare.com/en/proguard)
* [ProGurad介绍](http://www.jianshu.com/p/60e82aafcfd0)
* [JDWP协议以及实现](https://www.ibm.com/developerworks/cn/java/j-lo-jpda3/index.html)
* [从NDK在非Root手机上的调试原理探讨Android的安全机制](http://blog.csdn.net/luoshengyang/article/details/17131835)
* [Android远程调试的探索与实现](https://tech.meituan.com/android-remote-debug.html)
* [Android逆向之反调试](Android逆向之反调试)

资源：
* [逆向工具集合0](http://unclechen.github.io/2016/09/07/Android%E5%8F%8D%E7%BC%96%E8%AF%91%E6%8A%80%E6%9C%AF%E6%80%BB%E7%BB%93/)
* [逆向工具集合1](http://www.freebuf.com/sectool/111532.html)
* [看雪论坛](http://bbs.pediy.com/)


