---
layout: post
title: Priority CSSism Design
key: 20160704
tags: CSS
---
# CSS 纯简设计理念

　　随着对 CSS 的理解和熟练，我现在写样式时会不自觉地去考虑三个问题：

> 1. 能否不用图片？
> 2. 能否不用 JavaScript？
> 3. 能否用最精简的结构？

这也成了我的一个 CSS 设计理念——尽量用最纯粹的CSS语句和最精简的 HTML + CSS 结构去解决样式问题。

　　以一个简单的例子说明一下：

> http://codepen.io/tjcccc/pen/PzmBpb/
> ![set up-w124](http://o9iywg0nm.bkt.clouddn.com/2016-07-03-Screen Shot 2016-07-03 at 08.37.45.png)

这是一个按钮。当鼠标移上去时，其按钮面会变暗；当鼠标点击时，会模拟出”按下去“的交互效果。

　　示例中可看到按钮面上有两个色块。我最初学 CSS 时可能会给他加一张图片让它横向平铺，或者加一个 div 层让它叠加呈现。现在，其实就是一句 background 语句的事。点击效果，以前可能会用 jQuery 去实现，现在我会用伪类样式加CSS动画轻松搞定。详见 CSS：

```css
/* 在 codepan 中省去了 -webkit- 和 -moz- 等前缀。 */
.btn-style {
  width: 120px;
  height: 40px;
  font-size: 16px;
  font-weight: bold;
  color: #1bb4f3;
  border-radius: 10px;
  background: linear-gradient(to bottom, #fff 50%, #efefef 50%);
  border: 4px solid #1bb4f3;
  box-shadow: 0 4px 0 0 #1bb4f3;
  cursor: pointer;
  &:hover {
    background: linear-gradient(to bottom, #efefef 50%, #dcdcdc 50%);
  }
  &:active {
    animation: clickBtn 0.2s;
  }
}
@keyframes clickBtn {
  to {
    margin-top: 4px;
    box-shadow: none;
  }
}
```

可看到：

1. 「background: linear-gradient(to bottom, #fff 50%, #efefef 50%);」——这句是线性渐变背景样式。「to bottom」表示从顶部开始垂直渐变到底部，「#fff」是起始色，「#efefef」是结束色，前一个「50%」表示有 50% 的「#fff」部分是实色，后一个「50%」表示有 (100% - 50%) 的「#efefef」部分是实色，这样一来剩下的渐变色部分就是 0%，恰好形成了两个纯色块的拼接样式。
2. 伪类元素「:active」表示元素在活动状态（被点击）时的样式。在「:active」中，我加了一个CSS动画（animation）效果，让按钮的阴影逐渐变为无，同时用「margin-top」使其位置往下偏移原阴影高度的距离，这样就有了「按下去」的交互效果——有了 animation，JS 可以从很多交互舞台上退场了。
3. 这段 CSS 我是用预处理器 [SCSS](https://sass-lang.com) 写的，其中「&:hover」的「&」表示所在样式本身，等于「btn-style:hover」。掌握一门 CSS 预处理器能提高你的工作效率，应属必备技能。

　　最后看其 HTML 内容：

```html
<button class="btn-style">
  Click
</button>
```

很简单，语义化标签，一层结构。由此，三个问题也都解决了：没用图片；没用JS；结构不复杂。

　　纯粹简洁的CSS，并非用来装逼，它对项目开发有直接的好处：

1. 不用图片——减少流量，加快页面加载。一张图片的文件大小往往比整套 CSS 样式文件还大。
2. 不用 JS ——根据 MSDN 的说法，CSS 动画有更好的[呈现性能](https://msdn.microsoft.com/zh-cn/library/jj680076(v=vs.85).aspx)。 CSS 动画和 JS 动画的优劣可参考这篇问答[《CSS3动画和js动画各有什么优劣》](https://segmentfault.com/q/1010000000645415)。我的想法是：让 JS 去处理只有 JS 能处理的事，减少不必要的脚本加载。
3. 简化结构——配合语义化的 HTML，减少不必要的样式结构。尽量减少构成样式的元素。