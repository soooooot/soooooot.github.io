---
layout: post
title:  "前端单元测试中的名词及库简单解释"
date:   2016-04-19 00:55 +0800
categories: test javascript unittest
---

在碰巧看到前端单元测试的东西时，经常会看到一堆如`Karma`, `Mocha`, `Jasmine`, `Chai`之类的库的名词，但完全不知道这些库都是干什么的，然后就有了这篇blog。

我作为一个不会写单元测试的无证程序员，一直倍感压力，正巧借这次千载难逢的遇到需要写单元测试的项目(心里面尽是悲凉...), 阅读一些基础的文章资料后写一个简单的整理，帮助像我这样的单元测试的小白可以稍为了解各大概，听别人吹牛逼的时候也知道在吹着个什么东西。

写单元测试一般需要到`Assertion``(断言库)`, `Test Framework``(测试框架)`, `Test runner`, 我们以这三个层级给现在常见的这些库稍微分一个类。 (当然有些库直接就带有三个功能了...
)

### `Assertion``(断言库)`
断言就是否符合`预期(expection)`，要是不符合的话，就抛出异常。
例： 
{% highlight js %}
expect(foo).to.be.a('string');
expect(foo).to.equal('bar');
expect(foo).to.have.length(3);
expect(tea).to.have.property('flavors').with.length(3);
{% endhighlight %}

常见的断言库提供的语法就是`expect`, `should`, `asset`分别算是代表三种风格，然而这些风格又经常会挣到背后的`TDD``BDD`的方法论，这里就不去提`TDD``BDD`了，反正我就知道经常炮嘴轰来轰去的，想了解的人自己去了解吧。

`TDD`:  Test Driven Development

`BDD`:  Behavior Driven Development

关于这三种风格`Chai`都有支持，也许就是`Chai`是比较流行的断言库的原因? 这三种风格可以直接到[Chai]的首页中即可以看到。


比较流行点的独立断言库

* [Chai] - star 2.8k 提供三种写法，可以根据自己的需求喜好选用 
* [should.js] - star 1k
* [expect.js] - star 1.3k
* Node.js 核心库 Assert 


### Test Framework 测试框架

测试框架一般都是提供对`Test suite`测试套件和`test case`测试用例的支持。最直接的就是你看到那些测试代码里面，很有可能都会有`describe`,`it`或者`test`, `it`之流的。

`describe`/`test`就是说这是一个测试套件，表示一组相关的单元测试用例的集合。

`it`就是单元的测试用例：

{% highlight js %}
describe("The 'toBe' matcher compares with ===", function() {
  it("and has a positive case", function() {
    expect(true).toBe(true);
  });
  it("and can have a negative case", function() {
    expect(false).not.toBe(true);
  });
}
{% endhighlight %}


在测试用例里面就会写上`断言`是否符合`期望`, 若不符合，断言库就会抛出异常，测试框架则会对异常进行记录。


其中`Mocha`,`Jasmine`是通过对全局变量注入了`describe``it`，而`Tape``AVA`等则会提供了`test``it`的函数，侵入式并没有那么强。

当然提供`describe``it`的语句只是很小的一部分，一般还会需要一些常用的方法。有些库自己提供了这些方法，也有一些可以借助于专门的Helper方法库（如 sinon.js)

写单元测试为一个原则就是要能够尽可能得到的**快速响应**而且**同样的输入不能出现不同的输出**（想不起这个名词怎么说了...)，所以对于`文件操作`, `数据库操作`, `RPC`, `API调用`等，都难以满足这个原则，所以在测试中就出现了`Test Doubles`测试代替物这一个概念，就是在不影响测试主体的情况下，进行一些代替替换操作，如：每次访问这个API都会返回测试代码预设上的结果。

请容我先一本正经的介绍一下, `Test Doubles`是一个抽象的概念，常见的实现方法有`Stub`, `Mock`,`Dummy`, `Fake`, `Spy`(spies， 别改成复数就不认识它了)等等。对于这些概念，您可以参考

* [Mocks, Fakes, Stubs and Dummies] 这篇中有对比图
* [Mocks Aren't Stubs] Martin Flower大叔的文章
* [IBM DeveloperWorks的文章] 这篇带了点简单的例子说明

为什么我不在这里对这几个概念直接做解释呢，因为我**花了一晚上读资料，看各个测试框架中的实现**，然后被绕得云里雾里的，我发现好多测试框架对这几个名词的使用都有所差别，有些测试框架也对这些方法进行了合并，所以我觉得上面这几种方法都是理论上的，可能会帮助你理解。但我们的目标不是没有助牙，哦，不对。不是去理解上面的那几名词是干什么的，而是能够更好的写单元测试才是我们的目标。 所以我觉得在学写单元测试的时候，还是**直接看测试框架的API**吧, 那样更加直接有效。

另，看到不少文章会说到`Mock`，如Mock方法, Mock数据, 实际上我觉得应该是代指`Test Double`。

比较流行的带有测试框架的库

* [Jasmine] -  应该是目前最流行的测试框架
* [Mocha] -  "feature-rich, making asynchronous testing simple and fun."
* [Tape] -  "tap-producing test harness for node and browsers"
* [TAP] -  Test Anything Protocol tools for node
* [AVA] -  "Futuristic test runner", 最近势头很好
* [Cucumber] -  它支持很多语言平台，并且他们的基本功能都相同
* QUnit -  "A JavaScript Unit Testing framework, used by jQuery."
* [protractor] - E2E test framework for Angular apps
* [jstest] - The cross-platform JavaScript test framework

另介绍方法库：

* [Sinnon.JS] - Standalone test spies, stubs and mocks for JavaScript.


### test runner
有了测试代码后，你总得需要把测试代码跑进来，然后你得管理测试结果，你需要测试报告(reporter), 你可以还需要和`CI`持续集成系统进行结合, 有些需要打开浏览器测试的你还要想办法在浏览器中运行测试代码。
做这些事的呢，就是test runner, 一般这些测试框架都带有了基本的test runner的功能，只是也有一些会单独专门的再测试框架之前单独做一层test runner
`Karma`就只是单纯的`test runner`, 它并不是测试框架。另外`Gulp`, `Grunt`也算是的test runner.

#### 高级测试(High-level) 与 低层测试 (Low-level)

* Low-level: 指不与其它交互，自己就完成运行的测试
* High-level: 与其它部分交互，e2e(End to End)测试(基本上是指，测试时需要调用浏览器)

#### 代理模式(proxy) 与 WebDriver
* 代理模式: 由test runner启动server, 浏览器向server请求页面，server这边注入js代码执行
* WebDriver: `WebDriver`协议是与浏览器通协的协议，它可以更好的调控浏览器，比如可以触发原生的DOM事件。


常见的test runner, 有一些可能不仅仅只是test runner.

* [Karma] -  "Spectacular Test Runner for JavaScript."
* [Selemium] -  它同时提供测试框架，断言库。支持WebDriver, 有java, ruby, python等API, 写过爬虫的同学应该比较熟悉它。
* [Mocha] -  它同时还是一套测试框架，很多node项目都用它。"making asynchronous testing simple and fun."
* [JsTestDriver] -  又名jsTD
* Grunt
* Gulp
* [Teaspoon] - 1.2k Teaspoon: Javascript test runner for Rails. 
* [Test'em] - 2.3k Test'em 'Scripts! A test runner that makes Javascript unit testing fun.




### headless browser
如果没有headless broswer, 我们的单元测试大多都只能对逻辑进行测试，而测试用例涉及到`DOM`时，可能就只能用`Test Double`, 而有时候我们测试的主体就是`DOM`, 显然`Test Double`也是不行的。
当然幸运的时，我们有`headless browser`。`headless browser`是什么呢, 写过一些由js动态生成内容的网页的crwaler的同学可能就对此很熟悉了。它能实现几乎除了图形界面以外浏览器能做的一切事情。

`PhantomJS`就是一款`headless browser`, 国内的同学就算没写过单元测试的应该也对`PhantomJS`这个名词特别熟悉吧，嗯？没有印象？回想一下你在`npm install`时，由于**网络原因**总会有一个叫`PhantomJS`的大约100M+的东西总是安装失败。对的，就是它。因为`PhantomJS`有整个`webkit`的内核，能做几乎除图形界面以外浏览器能做的所有事，还提供了js的API，所以长得也稍微胖了一些，`PhantomJS`还实现了截取屏幕图形的功能，对于一些需要人工测试的地方，若先用此功能直接生成screenshot, 测试人员也会确认起来也会方便很多。

常见的headless brower及其相关工具有:

* [PhantomJS] - PhantomJS is a headless **WebKit** scriptable with a JavaScript API.
* [SlimerJS] - A scriptable browser for Web developers  **Gecko**, 它并不算真的headless
* [CasperJS] - 封装库 "CasperJS is a navigation scripting & testing utility which uses headless browsers."




### 其它
* Coverage 测试覆盖率: 这个是怎么计算的我也不太了解，改天再去看看。
* CI 持续集成: 这块应该也是test runner的延伸部分



### references: 

* [测试框架 Mocha 实例教程] - 阮一峰的，非常推荐
* [Node.js 单元测试：我要写测试] - 淘宝前端团队 参考了不少
* [JavaScript Test Runner] - Karma作者的论文
* [karma 测试框架的前世今生] - 同样淘宝前端团队, 内容主要来自于提炼上文[JavaScript Test Runner]
* [Mock方法介绍] - Mock点的选取讲得不错
* [Mocks, Fakes, Stubs and Dummies]
* [Mocks Aren't Stubs]
* [IBM DeveloperWorks的文章]
* [Unit Testing Best Practices in AngularJS]


[测试框架 Mocha 实例教程]: http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html
[Unit Testing Best Practices in AngularJS]: http://andyshora.com/unit-testing-best-practices-angularjs.html

[JavaScript Test Runner]: https://raw.githubusercontent.com/karma-runner/karma/master/thesis.pdf
[Node.js 单元测试：我要写测试]: http://taobaofed.org/blog/2015/12/10/nodejs-unit-tests/
[karma 测试框架的前世今生]: http://taobaofed.org/blog/2016/01/08/karma-origin/

[Mock方法介绍]: http://baidutech.blog.51cto.com/4114344/743740/

[Mocks, Fakes, Stubs and Dummies]: http://xunitpatterns.com/Mocks,%20Fakes,%20Stubs%20and%20Dummies.html
[Mocks Aren't Stubs]: http://martinfowler.com/articles/mocksArentStubs.html
[IBM DeveloperWorks的文章]: http://www.ibm.com/developerworks/cn/java/j-lo-TestDoubles/




[PhantomJS]: http://phantomjs.org/
[SlimerJS]: http://www.slimerjs.org/
[CasperJS]: http://casperjs.org/

[Chai]: http://chaijs.com/
[should.js]: https://shouldjs.github.io/
[expect.js]: https://github.com/Automattic/expect.js


[Karma]: https://karma-runner.github.io
[Selemium]: www.seleniumhq.org
[Mocha]: https://mochajs.org/
[JsTestDriver]: https://code.google.com/archive/p/js-test-driver/
[Teaspoon]: https://github.com/modeset/teaspoon
[Test'em]: https://github.com/testem/testem


[Sinnon.Js]: http://sinonjs.org/

[Jasmine]: https://jasmine.github.io
[Tape]: https://github.com/substack/tape
[Cucumber]: https://cucumber.io/
[TAP]: http://www.node-tap.org/
[AVA]: https://github.com/sindresorhus/ava
[protractor]: http://www.protractortest.org/
[jstest]: http://jstest.jcoglan.com
