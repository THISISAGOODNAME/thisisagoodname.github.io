---
layout: post
title: "终于，es6 import"
description: "终于，es6 import"
category: html5
tags: [html5]
---

&nbsp; &nbsp; &nbsp; &nbsp;如果说chrome 58中webgl2终于release对于大家来说不痛不痒(毕竟大家都在等NXT还有GPUweb)，那Chrome 61 release稳定版带来的新特性一定会让你爽翻——Chrome终于支持了es6 import，在不考虑兼容性的情况下，终于可以测试阶段不用webpack和babel了(发布当然还是要打包的，livereload见仁见智吧)。在前端方面，chrome可能会迟到，但绝不会缺席。

<!-- more -->

* Table of Contents
{:toc}

&nbsp; &nbsp; &nbsp; &nbsp;相信大家对于ES6，babel之类都熟的不能再熟了，我也就单刀直入，直接讲浏览器中import的用法。

&nbsp; &nbsp; &nbsp; &nbsp;在html中script标签中直接使用的时候要注意，需要给script标签加上`type="module"`属性，比如

```html
<script type="module" src="module.js"></script>
<script type="module">
  // or an inline script
  import {helperMethod} from './providesHelperMethod.js';
  helperMethod();
</script>
```

```javascript
// providesHelperMethod.js
export function helperMethod() {
  console.info(`I'm helping!`);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;再说一些`<script type="module"></script>`重要的特性吧

- module是延迟的，之后document加载完成后才能执行(真棒)
- `import` 和 `export` 声明必须在最外层(ES6难道就不是吗。。。)
- 所有代码都在严格模式下执行，相当于自动加上了`'use strict';`(不能更赞)

&nbsp; &nbsp; &nbsp; &nbsp;最后添一个各个浏览器对于ES6的支持情况吧

<p data-height="475" data-theme-id="0" data-slug-hash="MmvdOM" data-default-tab="result" data-user="samthor" data-embed-version="2" data-pen-title="Modules High Water Mark table" class="codepen">See the Pen <a href="https://codepen.io/samthor/pen/MmvdOM/">Modules High Water Mark table</a> by Sam Thorogood (<a href="https://codepen.io/samthor">@samthor</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

&nbsp; &nbsp; &nbsp; &nbsp;Safari 10.1(WebKit2)遥遥领先，100%提前完成了ES6的支持，chrome和Firefox各96%但考虑shadow DOM的支持，算chrome胜出吧。这次微软96%的支持虽然垫底但也比IE时代表现好太多。还是要表扬的。

&nbsp; &nbsp; &nbsp; &nbsp;今年其实还有一个大变化，就是CSS Level 4，这个chromium支持很勤，但是chrome和webkit就慢得多了，也许是大家习惯了Less，CSS in JS，PostCSS之后，对于CSS的诉求真的不大了吧。