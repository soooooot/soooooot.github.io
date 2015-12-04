---
layout: post
title:  "webpack with a browserify packaged library annotator.js"
date:   2015-12-04 21:41:54 +0800
categories: webpack
---


The first time I read the [install - annotator.js doc][annotator-install], the **2.0+** version, it is said that there's an `npm package` way to  work with.

```
We also publish an annotator package to npm. This package is not particularly useful in a Node.js context, but can be used by browserify or webpack. Please see the documentation for these packages for more information on using them.
```

I spent about 3 minutes to do it with `npm package`, but it failed. Actually, It's my first time to work with `webpack`, and also the first time to manage `front-end` library with `npm` instead of `bower`. So I decided to dive into it later, and I just switch to the `built package` way which is to use the released minified `annotator.min.js`.

The way I fetch the minified file is to download it manually in github released page, I can't find a built or packaged version in the npm package.

Several days later, I started to trying to work it with `npm package`, and I will describe the procedures.


I installed the annotator.js with npm. Since the default annotator version is 1.x version, so I need to specify the version `2.0.0-alpha.3`. Actually the first time I installed a wrong version, and it toke me about 10 minutes to find out the issue. So sad...

{% highlight sh %}
npm install annotator@2.0.0-alpha.3 -S
{% endhighlight %}

I imported the annotator with require command in my webpack entry file.
{% highlight js %}
var annotator = require('annotator');
{% endhighlight %}

It seems that everything is done, and I started to compile. It failed as I wished. 

{% highlight sh %}
ERROR in ./~/annotator/css/annotator.css
Module parse failed: /package-dir/node_modules/annotator/css/annotator.css Line 4: Unexpected token .
You may need an appropriate loader to handle this file type.
| -------------------------------------------------------------------- */
|
| .annotator-notice,
| .annotator-filter *,
| .annotator-widget * {
 @ ./~/annotator/browser.js 5:10-40

{% endhighlight %}

It seems that my webpack did not recognize the `.css` file. All right, webpack is not so smart to compat browserify without any configuration in the situation. So I think maybe I need to go over the annotator.js source code to know how they manage it with browserify, and I need to copy that in webpack way. 

There's a big issue for me, it is.... I don't know how to use browserify. I hope it will not too hard to understand.

Luckily, I found the snippet in the annotator.js project `browser.js` file.

{% highlight js %}
// Inject Annotator CSS
var insertCss = require('insert-css');
var css = require('./css/annotator.css');
insertCss(css);
{% endhighlight %}

**I realized that the `require` command is parsed by webpack instead of browserify.** After I read the `insert-css` library docs, I add the `loader` rule to webpack.config file.

{% highlight js %}
{
  text: /\.css$/,
  loader: "raw-loader",
  include: /node_modules\/annotator\/css/
}
{% endhighlight %}

And I compiled again, I got 0 error, it's a good news. And I open the browser to test, I still got an error in console.

```
GET http://localhost:9900/img/annotator-icon-sprite.png?embed 404 (File not found)
```

I missed something in the annotator broswerify configrations. 

Go back to annotator.js source code, `package.json`.

{% highlight js %}
{
    "browserify": {
        "transform": [
          "./tools/cssify"
        ]
    }
}
{% endhighlight %}

And also the `./tools/cssify.js` file. briefly, the file imports `enhance-css`, `clean-css` npm library to handle the `.css` file to make the file as base64 string instead of indenpendent file. All I need to do it copy it in webpack way.

After I kept trying to change the webpack config, I find out the config works. I changed the `.css` rule from `raw-loader` to `css-loader`, and I add a `url-loader` for image files.

**Here's the final solution. the webpack config loader fules.**

{% highlight js %}

[
  {
    text: /\.css$/,
    loader: "css-loader",
    include: /node_modules\/annotator\/css/
  },
  {
    text: /\.png/,
    loader: "url-loader",
    include: /node_modules\/annotator\/img/
  }
]

{% endhighlight %}

**Tell you the truth, It just works, But I don't know how it works in detail.**

Just a few minutes ago, during I am writing the blog. I think I know why the solution works. 

Let's go back to read the annotator.js `browser.js` snippet.
{% highlight js %}
// Inject Annotator CSS
// var insertCss = require('insert-css');
var css = require('./css/annotator.css');
// insertCss(css);
{% endhighlight %}

I **guess** the `insert-css` does not work in webpack way. The webpack has the ability to handler `.css` file all by himself. And luckyly, `insert-css` library has the fault tolerance ability.  I comment out the two insertCss related lines to test, and **the guess, it turn out to wrong.** 

So till now, I still don't know how it works. 

And also, I don't know it's right or not to read the library source code and copy it in webpack way. What about if I need to use 10+ browserify library?

Last, thanks william for giving me the advise to add the webpack loader rules.


[annotator-install]: http://docs.annotatorjs.org/en/latest/installing.html
