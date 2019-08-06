# Start thinking declaratively

> 开始以声明的方式思考

If you’re coming to Flutter from an imperative framework (such as Android SDK or iOS UIKit), you need to start thinking about app development from a new perspective.

> 如果您从命令式框架（例如 Android SDK 或 iOS UIKit）来到 Flutter，您需要从新的角度开始考虑应用程序开发。

Many assumptions that you might have don’t apply to Flutter. For example, in Flutter it’s okay to rebuild parts of your UI from scratch instead of modifying it. Flutter is fast enough to do that, even on every frame if needed.

> 您可能有的很多假设不适用于 Flutter。 例如，在 Flutter 中，可以从头开始重建部分 UI，而不是修改它。 如果需要，即使在每一帧上，Flutter 都足够快。

Flutter is declarative. This means that Flutter builds its user interface to reflect the current state of your app:

> Flutter 是陈述性的/声明式的。 这意味着 Flutter 构建其用户界面以反映您应用的当前状态：
> ![](./1.png)

When the state of your app changes (for example, the user flips a switch in the settings screen), you change the state, and that triggers a redraw of the user interface. There is no imperative changing of the UI itself (like widget.setText) — you change the state, and the UI rebuilds from scratch.

> 当应用程序的状态发生变化时（例如，用户在设置屏幕中翻转一个开关），您将更改状态，并触发重绘用户界面。 没有必要更改 UI 本身（如 widget.setText） - 您更改状态，UI 从头重建。（你只需要修改数据，Flutter 自动从头 rebuild UI，而不必自己去修改 UI-就像 react）

Read more about the declarative approach to UI programming [in the get started guide](https://flutter.dev/docs/get-started/flutter-for/declarative).

> 在入门指南中阅读有关 UI 编程声明式方法的更多信息。

The declarative style of UI programming has many benefits. Remarkably, there is only one code path for any state of the UI. You describe what the UI should look like for any given state, once — and that is it.

> UI 编程的声明式风格有许多好处。 值得注意的是，UI 的任何状态只有一个代码路径。 您可以描述对于任何给定状态 UI 应该是什么样子 - 就是这样。

At first, this style of programming might not seem as intuitive as the imperative style. This is why this section is here. Read on.

> 起初，这种编程风格可能看起来不像命令式风格那么直观。 这就是本节的原因。 继续阅读。
