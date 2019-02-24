---
layout: post
title: Dabbling in Developing Chrome Extension
key: 20190217
tags: ChromeExtension
---
# 初试 Chrome Extension 开发

这两天试着接触了一下 Chrome Extension 的开发。

起因是一个朋友问我有没有办法在操作网站表单时减少重复点击工作，一开始我的想法是直接贴 JS 代码到 console 里，后来想了下，不如做成插件，一键实现功能，更方便省事。

然而前天的我并不知道怎么开发插件。于是网上找了 [官方教程](https://developer.chrome.com/extensions/getstarted) 开始学习。今天终于搞定了最基本的操作 DOM 的功能。

插件是用 JS 写的，写方法并不难，难的是了解它运行的模式和结构。

<!--more-->

配置文件 `manifest.json` 是一切的开始，其中要配置几个关键内容：`background`, `content`, `page_action`：

```json
"background": {
    "scripts": ["background.js"],
    "persistent": false
  },
"content_scripts": [
  {
    "matches": ["<all_urls>"],
    "js": ["content.js"]
  }
],
"page_action": {
  "default_popup": "popup.html",
  "default_icon": {
    "16": "images/get_started16.png",
    "32": "images/get_started32.png",
    "48": "images/get_started48.png",
    "128": "images/get_started128.png"
  }
}
```

其中：

- `background` 算是插件的后台程序，插件一运行就会执行它。
- `content_scripts` 是附加的内容脚本，相当于给当前页面增加一个额外的 JS 文件，任何页面刷出来，都会额外再执行它，所以它也是插件中唯一可以直接操作网站网页 DOM 的脚本，很重要。
- `page_action` 是插件在页面上可以执行的功能，配置中的 `popup.html` 就是我点击插件图标时弹出来的交互界面。`page_action` 也可以换成 `browser_action`，区别是：`page_action` 的 `popup.html` 需要通过脚本调用（一般把判断写在 `background` 中），如果没有调用，只会弹出插件的操作菜单，而 `browser_action` 直接就可以调出 `popup.html` 来，虽然方便，但其实不太灵活。

用执行流程描述一下：在某网站某页面，`background` 判断插件满足使用条件—— `page_action` 被激活——点击插件图标，弹出 `popup.html` 界面——再点击界面中某元素，让它用 `sendMessage` 方法通知 `content_scripts`——“我要操作 DOM 元素”——于是 `content_scripts` 开始执行操作。 

搞懂他们之间的关系和调用的方式，看看文档和例子，学着做一下，插件开发就可以算入门了。当然，离开发真正合格的插件还差得很远。有时间我会好好做一个——等我想出来要做的东西时。
