---
layout: post
title:  "搭建react-native安卓环境遇到的问题"
date:   2016-01-07 14:14
categories: react-native shadowsocks proxy proxychains privoxy terminal nvm 终端
---

### 环境
OS X / shadowsocksX (并非路由器翻墙)



### proxy问题
我使用的是shadowsocksX客户端，对于绝大多数的桌面应用，它工作得很好。但是，在终端下(至少我用的上iTerm2)，它并不能直接通过代理。

这里有两个方法在terminal下走shadowsocks的代理
* [privoxy -- socks5 to http]
    该方法是启动一个新的进程，监听新的端口(8118)，将socks5转成http proxy, 然后再使用**大多数**应用支持的http_proxy环境变量
* [proxychains]
    在每个命令前加上proxychains前缀： `proxychains4 curl -v https://www.twitter.com`
    值得一提的是，需要你的`curl`/`wget`**版本够新**去支持该特性

可以使用`curl --version`查看curl的版本，实测默认的`7.43.0`并不能工作,`7.45.0`可以工作。（把我坑了半天，把了好久才发现原来是这个原因）

当然，这样并不能完全工作。比如说`react-native run-android`时，`gradle`的操作就并不能直接的走设置过的代理，需要我们对`gradle`额外进行配置。

因为`gradle`的代理似乎只支持http/https，见[gradle-proxy-docs]，所以我开启了`privoxy`服务。
编辑`ProejctRoot/android/gradle.properties`文件，将下列内容配置上去。

{% highlight sh %}
systemProp.http.proxyHost=localhost
systemProp.http.proxyPort=8118

systemProp.https.proxyHost=localhost
systemProp.https.proxyPort=8118
{% endhighlight %}

另，可通过`cd android && ./gradlew installDebug --debug` 或`--info`, `--stacktrace`这样来调度查看其间遇到的问题。
我在安装时候还遇到了`Adnroid SDK Build Tools`版本不对的问题，目前它要指定的`23.0.1`才能工作。

再另，我再`brew install android-sdk`的时候，曾尝试过[brew安装本地文件]的方法，但是并未成功。
方法简要：就是把目标文件下好，放在`brew --cache`的目录上，但我没有操作成功，权限设置都和其它缓存文件一样。

### packager.sh 'node' command not found
因为是我使用了nvm，所以在未启用nvm的情况下，并不能找到node命令。
在此处参考了下github上面的issue，在`react-native`的脚本文件里hack了一下。

在`node_modules/react-native/packager/packager.sh`, 并在前面加上如下内容 
{% highlight sh%}
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
nvm use stable
{% endhighlight %}
此处根据你的nvm的安装情况进行修改。


[privoxy -- socks5 to http]: http://yanghui.name/blog/2015/07/19/make-all-command-through-proxy/
[proxychains]: http://cxh.me/2015/01/30/use-shadowsocks-in-terminal/
[gradle-proxy-doc]: https://docs.gradle.org/current/userguide/build_environment.html
[brew安装本地文件]: http://mygeekdaddy.net/2014/12/05/how-to-install-a-local-file-in-homebrew/
