---
layout: post
title:  "Class Method Autobinding in Babel 6"
date:   2016-01-16 00:50
categories: es6 babel autobinding react
---

## background

First tell you why I like to use the autobinding feature, here's the example from react offical blog [React v0.13.0 Beta 1]:  

You can check details on the link, in a short hand, we don't need to write `this.method = this.method.bind(this);` in the `constructor` function.

{% highlight js%}
class Counter extends React.Component {
  tick = () => {
    ...
  } // NO semi-colon here
  ...
}
{% endhighlight %}


The descripion in react offical blog [React on ES6+]:

> Luckily, by combining two ES6+ features – arrow functions and property initializers – opt-in binding to the component instance becomes a breeze:

{% highlight js%}
class PostInfo extends React.Component {
  handleOptionsButtonClick = (e) => {
    this.setState({showOptionsModal: true});
  }  // NO semi-colon here
}  
{% endhighlight %}


## semi-colon

As you can see, I added the `NO semi-colon here` comments into the two snippets. That's the story I wanna tell.

I tested the feature and it worked well in the Nov 2015, But now, I copy the snippet, but babel says `Syntax Error` to me.
And I didn't pay attention on it, so I switch to add `this.method = this.method.bind(this);` for every method I created, MANUALLY!
But as I write more codes, I can stand it any more, so I started to check why the autobinding feature doesn't work.

I found the diffrence between the test project and current project, test project I'm using `Babel 5`, but now in `Babel 6`.  I'm not sure it's the issue, just FYI.

by the help of [ZhangRGK], we found it that the autobinding feature seems not a standalone feature,
It combines `class properties` and `arrow function`.

As the [Proposal: ES Class Fields & Static Properties] shows, we need append a semi-colon to the class property definition statement.
{% highlight js%}
class ClassWithInits {
  myProp = 42;
}
{% endhighlight %}

furthemore, we change the value to arrow function, It is should be the autobinding feature we want.
{% highlight js%}
class ClassWithInits {
  myProp = () => 42;
}
{% endhighlight %}

DO NOT FORGET to append SEMI-COLON!!
As I tested, it works! I don't need to manually write binding statement anymore.


But actually, it seems NOT WORK LIKE THIS WAY. Since I found that the `class properties` feature is not included in the presets I added, `es2015` and `stage-0`.
As at least it works now, I don't wanna dive into it.


[React on ES6+]: http://babeljs.io/blog/2015/06/07/react-on-es6-plus/

[React v0.13.0 Beta 1]: http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#autobinding
[ZhangRGK]: http://blog.zhangrgk.ninja/
[Proposal: ES Class Fields & Static Properties]: https://github.com/jeffmo/es-class-fields-and-static-properties
