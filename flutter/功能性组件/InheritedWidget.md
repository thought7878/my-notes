# InheritedWidget

- Why

它能实现组件跨级传递数据。和逐级传递数据相比，通过 widget 构造函数来一级一级传递数据，当层级少的时候还可以，而层级多的时候就非常繁琐麻烦。

- What 概述

InheritedWidget 是一种让数据在 widget 树中从上到下传递的组件间共享数据/组件间通信的工具。比如我们在应用的根 widget 中通过 InheritedWidget 共享了一个数据，那么我们便可以在任意子 widget 中来获取该共享的数据；通过提供的修改数据的 API 来修改数据，进而实现组件间通信！

这个特性在一些需要在 widget 树中共享数据的场景中非常方便！如 Flutter SDK 中正是通过 InheritedWidget 来共享应用主题（Theme）和 Locale (当前语言环境)信息的。

### InheritedWidget 与 依赖它的 StatefulWidget（didChangeDependencies/build）的关系

State 对象有一个 didChangeDependencies 回调，它会在“依赖”发生变化时被 Flutter Framework 调用。而这个“依赖”指的就是子 widget 是否使用了父 widget 中 InheritedWidget 的数据！如果使用了，则代表子 widget 依赖 InheritedWidget；如果没有使用则代表没有依赖。**这种机制可以使子组件在所依赖的 InheritedWidget 变化时来更新自身**！比如当主题、locale(语言)等发生变化时，依赖其的子 widget 的 didChangeDependencies 方法将会被调用。
