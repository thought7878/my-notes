# redux 入门

## 为什么使用 redux

### redux 要解决什么问题？

1. 解决 react 带来的问题：
   React 只是 DOM 的一个抽象层，并不是 Web 应用的完整解决方案。有两个方面，它没涉及。一个是，代码结构；一个是，组件之间的通信。对于大型的复杂应用来说，这两方面恰恰是最关键的。**因此，只用 React 没法写大型应用。**

   为了解决这个问题，2014 年 Facebook 提出了 Flux 架构的概念，引发了很多的实现。2015 年，Redux 出现，将 Flux 与函数式编程结合一起，很短时间内就成为了最热门的前端架构。

2. 在遇到这些情况时，需要 redux：

   - Redux 的适用场景（多交互、多数据源）：

     - 用户的使用方式复杂
     - 不同身份的用户有不同的使用方式（比如普通用户和管理员）
     - 多个用户之间可以协作
     - 与服务器大量交互，或者使用了 WebSocket
     - View 要从多个来源获取数据

   - 从组件角度看，如果你的应用有以下场景：
     - 某个组件的状态，需要共享
     - 某个状态需要在任何地方都可以拿到
     - 一个组件需要改变全局状态
     - 一个组件需要改变另一个组件的状态

发生上面情况时，如果不使用 Redux 或者其他状态管理工具，不按照一定规律处理状态的读写，代码很快就会变成一团乱麻。因此，需要 redux，可以在同一个地方查询状态、改变状态、传播状态的变化。

### 还有什么别的解决方案？

Redux 只是 Web 架构的一种解决方案（Flux 架构），也可以选择其他架构方案。

其他架构方案：

- MVC
- MVVM
- Observer
- Reactive

### redux 跟他们的区别是什么，有什么优点或缺点？

## redux 是什么 & 怎么用

### 设计思想

Redux 的设计思想很简单：

（1）Web 应用是一个状态机，视图与状态是一一对应的。

（2）所有的状态，保存在一个对象里面。

状态就是数据

### store

负责保存数据。

##### 创建 store

```js
import { createStore } from 'redux'
const store = createStore(fn)
```

##### 获取数据

```js
import { createStore } from 'redux'
const store = createStore(fn)

const state = store.getState()
```

Store 对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。

当前时刻的 State，可以通过 store.getState()拿到。
Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。

### reducer

负责修改数据

### action

action 是新数据，用户行为（或程序行为）产生的新数据。

State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。

Action 是一个对象。其中的 type 属性是必须的，表示 Action 的名称。其他属性可以自由设置，社区有一个规范可以参考。

```js
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
}
```

上面代码中，Action 的名称是 ADD_TODO，它携带的信息是字符串 Learn Redux。

可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。

### store/reducer/action 之间的关系

## 参考

- [Redux 入门教程（一）：基本用法](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)

- [React 技术栈](https://github.com/ruanyf/jstraining/blob/master/docs/react.md)
