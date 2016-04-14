---
layout: post
title:  "react-native build android apk 问题续 - NDK相关"
date:   2016-04-14 11:47 +0800
categories: react-native build android-ndk
---

这篇只是记流水账，供自己回顾。

我所使用的环境:

os: OS X
react-native:  commit [#a8f4159](https://github.com/facebook/react-native/commit/a8f4159fc76420a536ad3d98e737744fa7a04f72)
android-ndk: r11c


经过[上篇]所说的尝试之后，我自己又手贱的想build一个apk装到手机上，其实就是想装一个官方的[UIExplorer]的demo到手机上面，看文档的时候可以方便对照效果。

构建的命令简单:
> ./gradlew :Examples:UIExplorer:android:app:assemblRelease
复制执行，报错，发现是我没有安装android-ndk。咦，上篇中没有装ndk的情况也成功运行了，现在怎么又要用ndk了呢..


### 尝试不Compile ReactAndroid

-----  先说下这一步的结果，我没有弄成功。出了java heap space的原因，居然是内存不足，不过我怀疑是因为各种版本问题。特别可能是JDK的版本原因，这里我后面会再提到。

对比之前用react-native init建立出来的项目Awesome Project和UIExplorer的 `android/app/build.gradl`文件的差异发现:

{% highlight groovy %}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.0.1'

    // 以处为Awesome Project的依赖
    // compile "com.facebook.react:react-native:0.20.+"
    // 以处为UIExplorer的依赖
    // Build React Native from source
    // compile project(':ReactAndroid')
}
{% endhighlight %}

然后我一看，明明可以直接用嘛，然后就直接把替换使用了Awesome Project的依赖，直接找现呈编译好的不是更方便么~

再次assembleRelease，结果悲剧:
`:Examples:UIExplorer:android:app:bundleReleaseJsAndAssets FAILED`
OutOfMemory错误， Error: java heap space

内存不足，其实是个挺误导的提示，我遇到这个错误的原因在于我用的是`JDK 1.6`, 只要升级JDK, 重新再跑一次，然后能成功生成apk, 散花~~~

但是，实际上剧本并不是按照这么走的，我当时没有意识到JDK版本问题，然后就恢复了`compile project(':ReactAndroid')`, 开始了NDK编译ReactAndroid的神奇 之旅。


###  Could not generate DH keypair

在ReactAndroid: `downloadJSCHeaders`步骤中
> javax.net.ssl.SSLException: java.lang.RuntimeException: Could not generate DH keypair

这个问题可能不是什么大问题，当时心烦意乱，直接打开`react-native/ReactAndroid/build.gradle`,

{% highlight groovy %}

task downloadJSCHeaders(type: Download) {
    def jscAPIBaseURL = 'https://svn.webkit.org/repository/webkit/!svn/bc/174650/trunk/Source/JavaScriptCore/API/'
    def jscHeaderFiles = ['JSBase.h', 'JSContextRef.h', 'JSObjectRef.h', 'JSRetainPtr.h', 'JSStringRef.h', 'JSValueRef.h', 'WebKitAvailability.h']
    def output = new File(downloadsDir, 'jsc')
    output.mkdirs()
    src(jscHeaderFiles.collect { headerName -> "$jscAPIBaseURL$headerName" })
    onlyIfNewer true
    overwrite false
    dest output
}

{% endhighlight %}

然后我果然的手动下载了这几个文件，再过这个问题。


### NDK toolchains not found

    make: /usr/local/Cellar/android-ndk/r11c/toolchains/arm-linux-androideabi-4.8/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-g++: No such file or directory

检查NDK配置确实无误，然后查看路径发现，确实文件不存在，但并不是说我没有`arm-linux-androidaabi-g++`, 是因为交叉编译工具链的版！本！不！对！

可能当时在写react-native时所用的版本是`r10e`, 估计是`r10`所用的toolchian version是`4.8`, 但是我所新装的`r11c`的工具链是`4.9`版本。

两个解决方法:

* 方法一： 修改jni/Application.mk
* 方法二： 装个老版本的ndk


这里说一下怎么改`Application.mk`, 其实我并不懂java世界的东西，找到这个方法是因为我`grep "4.8"`发现的...

打开`react-native/react-native/ReactAndroid/src/main/jni/Application.mk`, 将`NDK_TOOLCHAIN_VERSION`改为`4.9`即可。

> NDK_TOOLCHAIN_VERSION := 4.8

这次我仍然没有进入轻松模式，我在用这个方法的时候，因为当时还没有意识到JDK版本的问题，同样没有执行成功，所以我装了一个老版本的NDK。


### 安装r10e NDK

brew只能装`r11c`的版本，老的版本并没有办法直接用brew安装。

还好NDK只是一个包，我们可以手动下载, 官网上并没有老版本的下载地址，只给出了新版本的地址，还好根据新版本的地址可以把老版本的弄出来。可是checksum就无可查了。

http://dl.google.com/android/repository/android-ndk-r10e-darwin-x86_64.zip

后面有看到官网上的r10e下载网址的截图, 发现当时是`bin`包，并不是`zip`包，不知道是不是google避免版权问题才在官网上不显示历史版本下载的。

这之前的NDK r11c是通过`brew`安装的，新下的`r10e`我也想通过`brew switch`命令进行切换，这样不用因为切版本而去改`$ANDROID_NDK`目录。

将NDK-r10e.zip包解压到`/usr/local/Cellar/android-ndk/r10e`, 然后执行`brew switch android-ndk r10e`, brew会进行软链接过去，`$ANDROID_NDK`保持原样指向`ANDROID_NDK=/usr/local/opt/android-ndk`不用修改。

### java  illegal start of type

> react-native/react-native/ReactAndroid/src/main/java/com/facebook/csslayout/CSSNode.java:92: illegal start of type

在查看日志后才发现有这么一条`warning`之前我一直没有在意，就是说要用JDK 1.7的，我的环境是JDK 1.6，进行降级使用。

看到这个日志，对于已经花了好多时间，踩一堆坑都快绝望的找不到方法的我，于是抱着试一试的心态升级了JDK。

结果！！就是因为JDK版本的原因，卡了我好半天。直到现在写这篇流水账的时候，我才意识到，其实我都没有必要用NDK，只要开始的时候用最新版本的JDK后，这些问题都不会出现。。。


### node相关问题

{% highlight sh %}
Error: Cannot find module 'core-js/fn/array/values'

Execution failed for task ':Examples:UIExplorer:android:app:bundleReleaseJsAndAssets'.
> Process 'command 'node'' finished with non-zero exit value 1
{% endhighlight %}

不确定是不是最新版react-native新引进来依赖版本的问题，或者是npm install的时候少装了依赖，对于已经精疲力尽的我，没有在这个问题上花时间，我直接从master分支跳到了6天前的`v0.23.1`分支上。


#### 再遇 ‘node’ command not found

这个问题是老问题了，在[上篇]中也同样遇到了这个问题，只是我才发现当时的解决方法简直就是一点都不科学。

当时状态比较混乱的我，暴力地采用了一堆方法来试途解决这个问题。

直接跑到gradwl里修改CommandLine的参数, 从最原先在`node`, 而改成`nvm use stable; node`, 再改成`nvm run stable`, 还有直接找到`.nvm`目录，用`~/.nvm/versions/node/v5.0.0/bin/node`等等，乱试了一大通。

最后重新读了下react native [Getting Started] 文档，

> Install nvm with its setup instructions here. Then run nvm install node && nvm alias default node, which installs the latest version of Node.js and sets up your terminal so you can run it by typing node. With nvm you can install multiple versions of Node.js and easily switch between them.

真是自己把自己坑了！！何必那么复杂。`nvm alias default node`就能解决问题。
当时看这里的时候只看到了要用nvm, 知道自己已经在用着nvm就没有细读了。原来才发现是我自己使用nvm的方式不对，这么简单的问题绕了一么一大大圈。







[上篇]: {% post_url 2016-01-07-problems-when-setup-react-native-behind-proxy %}
[UIExplorer]: https://github.com/facebook/react-native/tree/master/Examples/UIExplorer
[Getting Started]: https://facebook.github.io/react-native/docs/getting-started.html

