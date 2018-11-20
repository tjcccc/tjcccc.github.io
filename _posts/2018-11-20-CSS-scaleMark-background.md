---
layout: post
title: Use CSS background for Scale Mark
key: 20181120
tags: CSS linear-gradient
---
# 用 CSS background 实现刻度线的呈现

　　有的时候，我们需要在网页中的进度条或某种度量计上呈现一条条的刻度线。例如这种：

![scale mark demo](/assets/images/blog/scale-mark-demo.png)

简单的实现方式，大致有两种：一是用图片做背景，横向平铺线条图片；二是给每一块刻度区域平铺一个元素，然后用边线实现。身为一个“环保主义者”，这两种方式都不能让我满意。在看了 Lea Verou 的 CSS SECRETS 后，我受到了启发——可以用渐变背景的方式去实现。

<!--more-->

　　原理很简单。最简单的颜色渐变是颜色 A 过渡到颜色 B，那么，如果将颜色 A 设置成透明色，将颜色 B 设置成刻度线颜色，不就可以搞出刻度线了吗：

```CSS
div {
  background: linear-gradient(to right, transparent 99px, #fff 1px);
  background-size: 100px 100%;
}
```

在以上例子中，我用 `background-size` 设定刻度区间（背景）宽度为 100px，其中透明色我给它 99px 宽，线条色（白）我给它 1px 宽，这样从透明色到线条色的渐变就会失去过渡效果，从而实现了 100px 宽的区间里只有最后 1px 是线条——刻度线就这样出来了。用 `repeating-linear-gradient` 同样可以实现，而且不需要设置 `background-size`，如下所示：

```CSS
div {
  background: repeating-linear-gradient(
    90deg,
    transparent,
    transparent 99px,
    #fff,
    #fff 100px);
}
```

这个样式表示第一段渐变色从开始到 99px 都是透明色，第二段渐变色从 99px 到 100px 都是白色，之后按此设定循环。

　　详细的代码可参考 [CodePen demo](https://codepen.io/tjcccc/pen/GwMzVE)，关于 `linear-gradient` 和 `repeating-linear-gradient` 的用法可参考 [linear-gradient - CSS：层叠样式表 \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient) 和 [repeating-linear-gradient - CSS：层叠样式表 \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/repeating-linear-gradient)。