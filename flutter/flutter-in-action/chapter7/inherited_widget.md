# 7.2 数据共享（InheritedWidget）

## 概述

- Why

它能实现组件跨级传递数据。和逐级传递数据相比，通过 widget 构造函数来一级一级传递数据，当层级少的时候还可以，而层级多的时候就非常繁琐麻烦。

- What

InheritedWidget 是一种让数据在 widget 树中从上到下传递的组件间共享数据/组件间通信的工具。比如我们在应用的根 widget 中通过 InheritedWidget 共享了一个数据，那么我们便可以在任意子 widget 中来获取该共享的数据；通过提供的修改数据的 API 来修改数据，进而实现组件间通信！

这个特性在一些需要在 widget 树中共享数据的场景中非常方便！如 Flutter SDK 中正是通过 InheritedWidget 来共享应用主题（Theme）和 Locale (当前语言环境)信息的。

## InheritedWidget 与 依赖它的 StatefulWidget（didChangeDependencies/build）的关系

State 对象有一个 didChangeDependencies 回调，它会在“依赖”发生变化时被 Flutter Framework 调用。而这个 **“依赖”** 指的就是子 widget 是否使用了父 widget 中 InheritedWidget 的数据！如果使用了，则代表子 widget 依赖 InheritedWidget；如果没有使用则代表没有依赖。**这种机制可以使子组件在所依赖的 InheritedWidget 变化时来更新自身---调用 didChangeDependencies/build**！比如当主题、locale(语言)等发生变化时，依赖其的子 widget 的 didChangeDependencies 方法将会被调用。

### 例子解释这种机制

本示例主要是为了演示`InheritedWidget`的功能特性。

首先，我们通过继承`InheritedWidget`，将当前计数器点击次数保存在`ShareDataWidget`的`data`属性中：

```dart
class ShareDataWidget extends InheritedWidget {
  ShareDataWidget({
    @required this.data,
    Widget child
  }) :super(child: child);

  final int data; //需要在子树中共享的数据，保存点击次数

  //定义一个便捷方法，方便子树中的widget获取共享数据---获取的是此类的实例
  static ShareDataWidget of(BuildContext context) {
    //维护当前widget和子widget的关系，context是子widget的
    return context.inheritFromWidgetOfExactType(ShareDataWidget);
  }

  //该回调决定当data发生变化时，是否通知子树中依赖data的Widget
  @override
  bool updateShouldNotify(ShareDataWidget old) {
    //如果返回true，则子树中依赖(build函数中有调用)本widget
    //的子widget的`state.didChangeDependencies`会被调用
    return old.data != data;
  }
}
```

然后我们实现一个子组件`_TestWidget`，在其`build`方法中引用`ShareDataWidget`中的数据。同时，在其`didChangeDependencies()` 回调中打印日志：

```dart
class _TestWidget extends StatefulWidget {
  @override
  __TestWidgetState createState() => new __TestWidgetState();
}

class __TestWidgetState extends State<_TestWidget> {
  @override
  Widget build(BuildContext context) {
    //使用InheritedWidget中的共享数据
    return Text(ShareDataWidget
        .of(context)
        .data
        .toString());
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    //父或祖先widget中的InheritedWidget改变(updateShouldNotify返回true)时会被调用。
    //如果build中没有依赖InheritedWidget，则此回调不会被调用。
    print("Dependencies change");
  }
}
```

最后，我们创建一个按钮，每点击一次，就将`ShareDataWidget`的值自增：

```dart
class InheritedWidgetTestRoute extends StatefulWidget {
  @override
  _InheritedWidgetTestRouteState createState() => new _InheritedWidgetTestRouteState();
}

class _InheritedWidgetTestRouteState extends State<InheritedWidgetTestRoute> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return  Center(
      child: ShareDataWidget( //使用ShareDataWidget
        data: count,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.only(bottom: 20.0),
              child: _TestWidget(),//子widget中依赖ShareDataWidget
            ),
            RaisedButton(
              child: Text("Increment"),
              //每点击一次，将count自增，然后重新build,ShareDataWidget的data将被更新
              onPressed: () => setState(() => ++count),
            )
          ],
        ),
      ),
    );
  }
}
```

运行后界面如图 7-1 所示：

![图7-1](../imgs/7-1.png)

每点击一次按钮，计数器就会自增，控制台就会打印一句日志：

```
I/flutter ( 8513): Dependencies change
```

可见依赖发生变化后，其`didChangeDependencies()`会被调用。但是读者要注意，**如果\_TestWidget 的 build 方法中没有使用 ShareDataWidget 的数据，那么它的`didChangeDependencies()`将不会被调用，因为它并没有依赖 ShareDataWidget**。例如，我们将`__TestWidgetState`代码该为下面这样，`didChangeDependencies()`将不会被调用:

```dart
class __TestWidgetState extends State<_TestWidget> {
  @override
  Widget build(BuildContext context) {
    // 使用InheritedWidget中的共享数据
    //    return Text(ShareDataWidget
    //        .of(context)
    //        .data
    //        .toString());
     return Text("text");
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // build方法中没有依赖InheritedWidget，此回调不会被调用。
    print("Dependencies change");
  }
}
```

上面的代码中，我们将`build()`方法中依赖`ShareDataWidget`的代码注释掉了，然后返回一个固定`Text`，这样一来，当点击 Increment 按钮后，`ShareDataWidget`的`data`虽然发生变化，但由于`__TestWidgetState`并未依赖`ShareDataWidget`，所以`__TestWidgetState`的`didChangeDependencies`方法不会被调用。**其实，这个机制很好理解，因为在数据发生变化时只对使用该数据的 Widget 更新是合理并且性能友好的。**

> 思考题：Flutter framework 是怎么知道子 widget 有没有依赖 InheritedWidget 的？

### 应该在 didChangeDependencies()中做什么？

一般来说，子 widget 很少会重写此方法，因为在依赖改变后 framework 也都会调用`build()`方法。但是，如果你需要在依赖改变后执行一些昂贵的操作，**比如网络请求**，这时最好的方式就是在此方法中执行，这样可以避免每次`build()`都执行这些昂贵操作。

## 深入了解 InheritedWidget

**如果我们只想在`__TestWidgetState`中引用`ShareDataWidget`数据，但却不希望在`ShareDataWidget`发生变化时调用`__TestWidgetState`的`didChangeDependencies()`方法应该怎么办？** 其实答案很简单，我们只需要将`ShareDataWidget.of()`的实现改一下即可：

```dart
//定义一个便捷方法，方便子树中的widget获取共享数据
static ShareDataWidget of(BuildContext context) {
  //return context.inheritFromWidgetOfExactType(ShareDataWidget);
  return context.ancestorInheritedElementForWidgetOfExactType(ShareDataWidget).widget;
}
```

唯一的改动就是获取`ShareDataWidget`对象的方式，把`inheritFromWidgetOfExactType()`方法换成了`context.ancestorInheritedElementForWidgetOfExactType(ShareDataWidget).widget`，那么他们到底有什么区别呢，我们看一下这两个方法的源码（实现代码在`Element`类中，`Context`和`Element`的关系我们将在后面专门介绍）：

```dart 
@override
InheritedElement ancestorInheritedElementForWidgetOfExactType(Type targetType) {
  final InheritedElement ancestor = _inheritedWidgets == null ? null :  _inheritedWidgets[targetType];
  return ancestor;
}

@override
InheritedWidget inheritFromWidgetOfExactType(Type targetType, { Object aspect }) {
  final InheritedElement ancestor = _inheritedWidgets == null ? null :   _inheritedWidgets[targetType];
  //多出的部分
  if (ancestor != null) {
    assert(ancestor is InheritedElement);
    return inheritFromElement(ancestor, aspect: aspect);
  }
  _hadUnsatisfiedDependencies = true;
  return null;
}
```

我们可以看到，`inheritFromWidgetOfExactType()` 比 `ancestorInheritedElementForWidgetOfExactType()`多调了`inheritFromElement`方法，`inheritFromElement`源码如下：

```dart
@override
InheritedWidget inheritFromElement(InheritedElement ancestor, { Object aspect }) {
  //注册依赖关系
  _dependencies ??= HashSet<InheritedElement>();
  _dependencies.add(ancestor);
  ancestor.updateDependencies(this, aspect);
  return ancestor.widget;
}
```

可以看到`inheritFromElement`方法中主要是注册了依赖关系！看到这里也就清晰了，**调用`inheritFromWidgetOfExactType()` 和 `ancestorInheritedElementForWidgetOfExactType()`的区别就是前者会注册依赖关系，而后者不会**，**所以在调用`inheritFromWidgetOfExactType()`时，`InheritedWidget`和依赖它的子孙组件关系便完成了注册，之后当`InheritedWidget`发生变化时，就会更新依赖它的子孙组件，也就是会调这些子孙组件的`didChangeDependencies()`方法和`build()`方法**。而当调用的是 `ancestorInheritedElementForWidgetOfExactType()`时，由于没有注册依赖关系，所以之后当`InheritedWidget`发生变化时，就不会更新相应的子孙 Widget。

注意，如果将上面示例中`ShareDataWidget.of()`方法实现改成调用`ancestorInheritedElementForWidgetOfExactType()`，运行示例后，点击"Increment"按钮，会发现`__TestWidgetState`的`didChangeDependencies()`方法确实不会再被调用，但是其`build()`仍然会被调用！造成这个的原因其实是，点击"Increment"按钮后，会调用`_InheritedWidgetTestRouteState`的`setState()`方法，此时会重新构建整个页面，由于示例中，`__TestWidget` 并没有任何缓存，所以它也都会被重新构建，所以也会调用`build()`方法。

那么，**现在就带来了一个问题：** 实际上，我们只想更新子树中依赖了`ShareDataWidget`的组件，**而现在只要调用`_InheritedWidgetTestRouteState`的`setState()`方法，所有子节点都会被重新 build**，这很没必要，那么有什么办法可以避免呢？答案是缓存！一个简单的做法就是通过封装一个`StatefulWidget`，将子 Widget 树缓存起来，具体做法下一节我们将通过实现一个`Provider` Widget 来演示如何缓存，以及如何利用`InheritedWidget` 来实现 Flutter 全局状态共享。