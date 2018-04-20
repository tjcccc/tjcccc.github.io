---
layout: post
title: Use Font-Awsome in Angular Way
key: 20180420
tags: Angular Font-Awesome
---
# 以 Angular 的姿势打开 Font-Awesome

## 环境

- Angular: v5.2.9
- Font-Awesome: v5.0.10
- angular-fontawesome: v0.1.0-9

## 无须再用传统的 Web Font 方式

　　以前习惯于 Font-Awesome 的传统方式：页面底部引用一个 font-awesome.min.css 文件，然后在页面中使用 `<i class="fa xxx"></i>` 放置图标——这在 Angular 里依然可行，不过这并不 Angularish ——我们其实可以用 Angular 模块组件那种方式去实现。写此文时，官网还没有正式上线 Package for Angular, 不过在官方 GitHub 上已经有相关[文档教程](https://github.com/FortAwesome/angular-fontawesome/blob/master/README.md)了，本文以下内容基本遵循该官方文档。

## 安装 Package

　　npm 方式：

```shell
$ npm install @fortawesome/fontawesome-svg-core --save
$ npm install @fortawesome/free-solid-svg-icons --save
$ npm install @fortawesome/angular-fontawesome --save
```

其中「free-solid-svg-icons」是经典样式，其他还有「regular」和「light」可选：

```shell
$ npm install @fortawesome/free-brands-svg-icons --save
$ npm install @fortawesome/free-regular-svg-icons --save
```

## 在 app.module.ts 中导入基本模块

```ts
// ...
import { FontAwesomeModule } from '@fortawesome/angular-fontawesome';

@NgModule({
  // ...
  imports: [
    // ...
    FontAwesomeModule
  ],
  // ...
})
// ...
```

　　导入后便无须在其他组件中重复导入了。这是以下使用图标方式的基础。

## 按需使用方式一

　　在 component 里导入你所需要的图标：

```ts
// ...
import { faCoffee } from '@fortawesome/free-solid-svg-icons';

//...
export class AppComponent {
  //...
  myIcon = faCoffee;
}
```

注意这里导入的图标名字要加 fa 前缀，并使用 camelCase 命名法。导入后，你便可以在 html 模板中用以下方式使用图标：

```html
<fa-icon [icon]="coffee"></fa-icon>
```

注意在 html 模板中要直接使用图标名。图标可在官网[图标库](https://fontawesome.com/icons)查询。

## 按需使用方式二

　　第二种按需使用的方式是使用 library, 使用 library 后你就不用再在 component 中导入图标了，一切都在 app.module.ts 中完成：

```ts
import { FontAwesomeModule } from '@fortawesome/angular-fontawesome';
import { library } from '@fortawesome/fontawesome-svg-core';
```

有了 library 后，接着再添加你需要用的图标：

```ts
import { FontAwesomeModule } from '@fortawesome/angular-fontawesome';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faCoffee } from '@fortawesome/free-solid-svg-icons';
```

然后把图标加入到 library 里：

```ts
// import ...
library.add(faCoffee);
// NgModule({...
```

这样你就可以在 html 模板中直接使用了。

## 全套导入

　　对于一般规模的网站，我还是推荐将图标全部导入，想用什么就用什么，比查找名字一个一个导入方便。全套导入的方式就是用图标包的别称代替图标名：

```ts
// Single:
import { faCoffee } from '@fortawesome/free-solid-svg-icons';
// All:
import { fas } from '@fortawesome/free-solid-svg-icons';
```

其中「fas」的「s」代表的是「free-solid-svg-icons」的「solid」。以此类推，其他样式的导入是：

```ts
import { far } from '@fortawesome/free-regular-svg-icons';
import { fab } from '@fortawesome/free-brands-svg-icons';
```

然后在 library 中添加即可：

```ts
library.add(fas);
// or
library.add(fas, far);
```

添加之后，你就可以在 html 中任意使用图标了。

## 在 html 模板中的写法

　　之前的方式：

```html
<fa-icon [icon]="coffee"></fa-icon>
// or
<fa-icon icon="coffee"></fa-icon>
```

其实是一种简便写法。它默认使用了 fas 样式的图标，如果要 far 或 fab，你需要这样写：

```html
<fa-icon [icon]="['fas', 'coffee']"></fa-icon>
```

将样式包别称作为前缀填入数组第一个元素。我推荐这种精确的写法。

## 图标基本特效

　　Font-Awesome 还有很多很棒的图标特效——可以通过 html 的标签属性实现。这里直接复制文档中一些基础的用法：

### 旋转与脉搏式转动:

```html
<fa-icon [icon]="['fas', 'spinner']" [spin]="true"></fa-icon>
<fa-icon [icon]="['fas', 'spinner']" [pulse]="true"></fa-icon>
```

### 固定宽度：

```html
<fa-icon [icon]="['fas', 'coffee']" [fixedWidth]="true"></fa-icon>
```

### 边框:

```html
<fa-icon [icon]="['fas', 'coffee']" [border]="true"></fa-icon>
```

### 翻转:

```html
<fa-icon [icon]="['fas', 'coffee']" flip="horizontal"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" flip="vertical"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" flip="both"></fa-icon>
```

### 尺寸:

```html
<fa-icon [icon]="['fas', 'coffee']" size="xs"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" size="lg"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" size="6x"></fa-icon>
```

### 按角度偏转:

```html
<fa-icon [icon]="['fas', 'coffee']" rotate="90"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" rotate="180"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" rotate="270"></fa-icon>
```

### 靠左或靠右排列:

```html
<fa-icon [icon]="['fas', 'coffee']" pull="left"></fa-icon>
<fa-icon [icon]="['fas', 'coffee']" pull="right"></fa-icon>
```