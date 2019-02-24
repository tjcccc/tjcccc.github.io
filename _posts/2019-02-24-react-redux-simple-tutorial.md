---
layout: post
title: How to Use React-Redux Basically
key: 20190224
tags: react-redux react
---
# 初步搞懂了 react-redux 的用法

前天周五看了半天 YouTube 视频，感觉终于搞懂了 react-redux 最简单的用法，也就是说，入门了、会用了。这让我非常有成就感，年纪大了，每学会一样新东西，都是一次青春的夺回。

## 共享状态

React 组件有个很重要的元素就是状态（state），状态决定 UI 的呈现结果。那么，怎样让组件 A 的状态影响组件 B 的状态呢，就需要一个全局状态，也就是状态需要被共享。Redux 的主要作用就是解决这个共享问题。

<!--more-->

跟着大部分教程做，`index.js` 一般会被改成这样：

```jsx
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);
```

其中 `store` 就是一个存储共享状态的地方。组件如果要使用共享状态，就必须 `connect` 它：

```jsx
export default connect()(YourComponent)
```

然后关键就是 `connect()` 里面的参数了，它决定了两件基本的事：

1. 获取那些共享状态？
2. 怎样改变共享状态？

## Redux 三件套

Redux 比较麻烦的是，每个组件都需要三个文件去定义：

- `actionType`: 给每一个 `action.type` 设定一个标识，用来比对。
- `actions`: 定义方法名和对应的 `action.type`
- `reducer`: 编写代码，设计每一个 `action` 如何改变 `state`。

以 `Login` 为例：

`Login/actionTypes.js`:

```js
export const SIGN_IN = 'LOGIN/SIGN_IN';
export const SIGN_OUT = 'LOGIN/SIGN_OUT';
```

`Login/actions.js`:

```js
import { SIGN_IN, SIGN_OUT } from './actionTypes';

export const signIn = () => ({
  type: SIGN_IN
});

export const signOut = () => ({
  type: SIGN_OUT
});
```

`Login/reducer.js`:

```js
import { SIGN_IN, SIGN_OUT } from './actionTypes';

const initialState = {
  isSignedIn: false
}

export default (state = initialState, action) => {
  switch (action.type) {
    case SIGN_IN: {
      return {
        isSignedIn: true
      };
    }
    case SIGN_OUT: {
      return {
        isSignedIn: false
      }
    }
    default: {
      return state;
    }
  }
}
```

例子非常简单。`signIn()` 方法的 `action.type` 是 `SIGN_IN`，这个 `action` 的作用就是把 `isSignedIn` 这个状态的值变成 `true`，而`signOut` 是相反作用。

## 将 reducer 加入 store

Redux 组件的 `reducer` 做好后，必须添加到 store 里，才能被其他 react 组件用到。

各种教程都会用 `combineReducers` 来组合各种 `reducer`：

```js
const appReducer = combineReducers({
  login: LoginReducer
});
```

现在，当 `store` 被创建好后，`login.isSignedIn` 就是个全局共享的 `state` 了。


## connect

前面做的工作，都是为了 `connect()` 做准备。

假设有个控制登录和登出的菜单组件，名字叫 `LoginMenuComponent`。组件有两个按钮，按 `Sign In` 登录，显示 `Logged in`，按 `Sign Out` 登出，显示 `Unauthenticated`。

为了 `connect`，定义了两种关键方法 `mapStateToProps` 和 `mapDispatchToProps`。

代码如下：

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import { signIn, signOut } from '../redux/Login/actions';

class LoginMenuComponent extends Component {

  render = () => {
    const { isSignedIn } = this.props;

    return (
      <button onClick={this.props.onSignIn}>Sign In</button>
      <button onClick={this.props.onSignOut}>Sign Out</button>
      {isSigned ? <p>Logged in.</p> : <p>Unauthenticated.</p>}
    )
  }

  const mapStateToProps = state => ({
    isSignedIn: state.login.isSignedIn
  });

  const mapDispatchToProps = (dispatch) => ({
    onSignIn: () => {
      dispatch(signIn());
    },
    onSignOut: () => {
      dispatch(signOut());
    }
  });
}
```

其中，`mapStateToProps` 决定了组件的 props 用到哪个共享状态——`login.isSignedIn`，而 `mapDispatchToProps` 决定了会用到哪些方法去改变这个共享状态，其中定义了`onSignIn` 和 `onSignOut` 两个方法，你在组件里可以通过 `this.props.onSignIn()` 和 `this.props.onSignOut()` 去执行它。

简单而言，这两个方法搞定了共享状态的”读“和”写“，基本上，就完事了，最后连接一下：

```jsx
export default connect(mapStateToProps, mapDispatchToProps)(LoginMenuComponent);
```

一切就都 OK 了。如果有另一个组件通过它的 `mapStateToProps` 也读到了 `login.isSignedIn`，那么这个状态就被这两个组件共享了，这样就可以做很多事了。
