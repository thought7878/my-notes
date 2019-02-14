# Async Actions

[已经翻译的](https://cn.redux.js.org/docs/advanced/AsyncActions.html)

In the basics guide, we built a simple todo application. It was fully synchronous. Every time an action was dispatched, the state was updated immediately.

```
在基础指南中，我们构建了一个简单的todo应用程序。它是完全同步的。每次action被发送时，state立即被更新。
```

In this guide, we will build a different, asynchronous application. It will use the Reddit API to show the current headlines for a selected subreddit. How does asynchronicity fit into Redux flow?

```
在本指南中，我们将构建一个不同的异步的应用程序。它将使用Reddit API显示所选subreddit的当前标题。异步性如何适应Redux流程？
```

## Actions

```

```

When you call an asynchronous API, there are two crucial moments in time: the moment you start the call, and the moment when you receive an answer (or a timeout).

```
当您调用异步API时，有两个关键时刻：您开始调用的那一刻，以及您收到响应（或超时）的那一刻。
```

Each of these two moments usually require a change in the application state; to do that, you need to dispatch normal actions that will be processed by reducers synchronously. Usually, for any API request you'll want to dispatch at least three different kinds of actions:

```
这两个时刻中的每一个通常都需要改变应用状态state; 要做到这一点，您需要发出将由reducers同步处理的正常actions。 通常，对于任何API请求，您都需要至少发出三种不同的action：
```

- An action informing the reducers that the request began.

```

```

The reducers may handle this action by toggling an isFetching flag in the state. This way the UI knows it's time to show a spinner.

```

```

- An action informing the reducers that the request finished successfully.

```

```

The reducers may handle this action by merging the new data into the state they manage and resetting isFetching. The UI would hide the spinner, and display the fetched data.

```

```

- An action informing the reducers that the request failed.

```

```

The reducers may handle this action by resetting isFetching. Additionally, some reducers may want to store the error message so the UI can display it.

```

```

You may use a dedicated status field in your actions:

```

```
