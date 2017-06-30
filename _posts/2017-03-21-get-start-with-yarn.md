---
layout: post
title: "优化CSS文件引入策略"
description: "优化CSS文件引入策略"
tags: [memo]
---

使用`link`标签引入的CSS文件，或者使用`@import`引入的CSS文件会阻止浏览器对网页的渲染，浏览器在这些样式文件下载完毕前不会渲染任何东西从而延迟网页渲染，我们称之为*阻碍渲染CSS（Render blocking stylesheets）*。但有些场景下，我们并不需要浏览器，比如说当我们首次渲染并不需要某个CSS文件时，或者CSS文件实在太大，我们更关心如何能更快地展现网页而不是更早渲染出漂亮的页面时，我们就需要采取一些策略来阻止浏览器帮我们的倒忙。

## 阻碍渲染CSS

- 如背景所述，阻碍渲染CSS会延迟浏览器渲染网页，只有所有的CSS文件都加载完毕后，浏览器才会开始渲染网页
- CSS文件越大，下载所需时间会越长，延迟网页渲染的时间也越长
- CSS文件越多，下载所需时间会越长，延迟网页渲染的时间也越长
- 延迟网页渲染的时间越长，网页加载的时间也会越长（实际上和感觉上都是这样）

## 



[GitHub Project: loadCSS](https://github.com/filamentgroup/loadCSS)
loadCSS

A function for loading CSS asynchronously [c]2017 @scottjehl, @zachleat Filament Group, Inc. Licensed MIT

Why loadCSS?

Referencing CSS stylesheets with link[rel=stylesheet] or @import causes browsers to delay page rendering while a stylesheet loads. When loading stylesheets that are not critical to the initial rendering of a page, this blocking behavior is undesirable. The new <link rel="preload"> standard enables us to load stylesheets asynchronously, without blocking rendering, and loadCSS provides a JavaScript polyfill for that feature to allow it to work across browsers, as well as providing its own JavaScript method for loading stylesheets.

Latest release: https://github.com/filamentgroup/loadCSS/releases
NPM: https://www.npmjs.com/package/fg-loadcss
Recommended loadCSS Usage

LoadCSS is designed for loading CSS that is not critical to the initial rendering of the page, and desirable to load in an asynchronous manner. (For the critical CSS rules, we recommend either inlining that CSS in a style element, or referencing it externally and server-pushing it using http/2. Read more here)

The standard markup pattern for loading files asynchronously is: <link rel="preload"> (W3C Spec). We recommend using this markup pattern to reference your non-critical CSS files. loadCSS and its rel=preload polyfill are designed to enable this markup to work in browsers that don't yet support this feature.

For each CSS file you'd like to load asynchronously, use a link element like this:

<link rel="preload" href="path/to/mystylesheet.css" as="style">
In browsers that support it, the rel=preload attribute will cause the browser to fetch the stylesheet, but it will not apply the CSS once it is loaded (it merely fetches it). To address this, we recommend using an onload attribute on the link that will do that for us as soon as the CSS finishes loading.

<link rel="preload" href="path/to/mystylesheet.css" as="style" onload="this.rel='stylesheet'">
This step requires JavaScript to be enabled, so we recommend including an ordinary reference to your stylesheet inside a noscript element as a fallback.

<link rel="preload" href="path/to/mystylesheet.css" as="style" onload="this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="path/to/mystylesheet.css"></noscript>
After linking to your asynchronous stylesheet(s) this way, include the loadCSS script, as well as the loadCSS rel=preload polyfill script in your page. These can be inlined or linked and http/2-pushed if possible. Here's how they would look inlined in the page:

<link rel="preload" href="path/to/mystylesheet.css" as="style" onload="this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="path/to/mystylesheet.css"></noscript>
<script>
/*! loadCSS. [c]2017 Filament Group, Inc. MIT License */
(function(){ ... }());
/*! loadCSS rel=preload polyfill. [c]2017 Filament Group, Inc. MIT License */
(function(){ ... }());
</script>
These scripts will automatically detect if a browser supports rel=preload. In browsers that natively support rel=preload, these scripts will do nothing, allowing the browser to load and apply the asynchronous CSS (note the onload attribute above, which is there to set the link's rel attribute to stylesheet once it finishes loading in browsers that support rel=preload). In browsers that do not support rel=preload, they will find CSS files referenced this way in the DOM and load and apply them asynchronously using the loadCSS function.

Note: regardless of whether the browser supports rel=preload or not, a CSS file will be referenced from the same location in the source order as your original link element. Keep this in mind, as you may want to place the link in a particular location in your head element so that the CSS loads with an expected cascade order. Also, any media attribute value on the original link element will be retained when the polyfill is in play.

You can view a demo of this rel=preload pattern here: https://master-origin-loadcss.fgview.com/test/preload.html

Manual CSS loading with loadCSS

The loadCSS.js file exposes a global loadCSS function that you can call to load CSS files programmatically, when needed.

loadCSS( "path/to/mystylesheet.css" );
The code above will insert a new CSS stylesheet link after the last stylesheet or script that it finds in the page, and the function will return a reference to that link element, should you want to reference it later in your script. Multiple calls to loadCSS will reference CSS files in the order they are called, but keep in mind that they may finish loading in a different order than they were called.

Function API

If you're calling loadCSS manually (without the rel=preload pattern, the function has 3 optional arguments.

before: By default, loadCSS attempts to inject the stylesheet link after all CSS and JS in the page. However, if you desire a more specific location in your document, such as before a particular stylesheet link, you can use the before argument to specify a particular element to use as an insertion point. Your stylesheet will be inserted before the element you specify. For example, here's how that can be done by simply applying an id attribute to your script.
... <script id="loadcss"> // load a CSS file just before the script element containing this code loadCSS( "path/to/mystylesheet.css", document.getElementById("loadcss") ); </script> ... ```
media: You can optionally pass a string to the media argument to set the media="" of the stylesheet - the default value is all.
Using with onload

Onload event support for link elements is spotty in some browsers, so if you need to add an onload callback, include onloadCSS function on your page and use the onloadCSS function:

var stylesheet = loadCSS( "path/to/mystylesheet.css" );
onloadCSS( stylesheet, function() {
	console.log( "Stylesheet has loaded." );
});
Browser Support

loadCSS attempts to load a css file asynchronously in any JavaScript-capable browser. However, some older browsers such as Internet Explorer 8 and older will block rendering while the stylesheet is loading. This merely means that the stylesheet will load as if you referenced it with an ordinary link element.

Contributions and bug fixes

Both are very much appreciated - especially bug fixes. As for contributions, the goals of this project are to keep things very simple and utilitarian, so if we don't accept a feature addition, it's not necessarily because it's a bad idea. It just may not meet the goals of the project. Thanks!