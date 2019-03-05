# (编译)关于 package & package-lock.json 的一切

文章编译自: [Everything You Wanted To Know About package-lock.json But Were Too Afraid To Ask](https://medium.com/@Quigley_Ja/everything-you-wanted-to-know-about-package-lock-json-b81911aa8ab8)

意译，尽量翻译地看起来没有翻译腔的痕迹。。

## 介绍

你把 npm 更新到`v5.x.x`以后，会出现一种新的自动生成文件 - `Package-lock.json`。你如果打开这个文件，会发现它看着像`package.json`里面的依赖，不过看着更啰嗦。如果你不去管他，迟早你都会出现依赖的问题，比如依赖的版本没有安装正确。大多数人是直接删除掉`package-lock.json`，然后重新运行`npm install`来解决问题，这样也行，但是我们还是要搞清楚看这个文件是看什么的。

## 总结

- 如果你用的是`npm^5.x.x`的版本，它默认就会生成`package-lock.json`这个文件
- 你应该使用`package-lock.json`来保证每次安装的依赖都是一样的
- 你应该把`package-lock.json`提交到版本控制库里面
- `npm^5.1.x`以后，`package.json`在某些情况下（文章后面会说）可以优先于`package-lock.json`，所以你应该不会那么头疼了。
- 不用先删除`package-lock.json`，只需要运行`npm install`就可以重新生成`package-lock.json`
- 如果你的库提供了对外 API，应该遵循`semver`规范来变更版本号

## 背景知识

在你了解`package.json`和`package-lock.json`前，你需要先了解[semver 语义化版本号变更](http://semver.org/)。它是 npm 依赖管理背后的精髓。你可以点[这里](http://blog.npmjs.org/post/162134793605/why-use-semver)了解 npm 是如何使用它的。总的来说，所谓的语义化版本号变更，就是当你修改了你所维护的第三方库时，告诉那些使用了你的库的项目，你做的是哪种修改。一个版本号分为三个部分: X,Y,Z。X 表示主版本号，Y 表示次版本号，Z 表示补丁更新。当你只是简单的修复了 BUG，没有做任何新功能的添加，或者旧功能的修改，就需要更新补丁号（数值加一等等）。当你添加了新的功能，但没有破坏原有的功能，就需要更新次版本号。当你做了重大修改导致新版本不兼容旧的代码时，就需要更新主版本号。

## 包管理

npm 是为了更方便管理依赖。你的项目可能有上百个依赖，每个依赖本身又有上百个依赖。如果让你来记住这些依赖肯定会让你觉得想屎。所以 npm 诞生了，让你仅仅用几个命令就可以管理大量的依赖。

当你添加了一个依赖时，`package.json`里面会加一项条目，包含包的名称，并且使用 semver 来处理版本号。npm 支持版本号中包含通配符。一般来说，npm 会安装最新的版本的依赖，并且在版本号前面加上`^`符号，比如`^1.2.12`。这表示，至少要安装`1.2.12`的版本，但更高的版本也可以，只要主版本号也为 1 就行。因为补丁号和次版本号的变动不会变更已有的 API。你应该可以安全地使用任何主版本一致的更高版本依赖。你可以点击[这里](http://blog.npmjs.org/post/115305091285/introducing-the-npm-semantic-version-calculator)的版本计算器，了解通配符的含义。

## 项目共享

把依赖记录在`package.json`里面的优点是，只要别人可以访问`package.json`，他就可以安装里面所记录的依赖，然后就可以直接运行你的项目了。不过有些时候，这样子会出现一个问题。

比如我们的项目用了 express 这个框架。我们使用`npm init`初始化项目后，使用命令`npm install express --save`安装了依赖。在写这篇文章的时候，express 的版本号是`4.15.4`，所以`package.json`里面 express 的版本号会是`^4.15.4`，然后我的电脑上面就安装了版本为`4.15.4`的 express。然后可能第二天，express 的维护者修复了一个 BUG，现在最新版本变成了`4.15.5`。接着，有个人觉得我们的项目很不错，他想贡献一些代码，于是它 clone 下来，运行了`npm install`。由于最新 express 版本是`4.15.5`，并且主版本号也是一致的，因此他的电脑上安装的版本就是`4.15.5`。我们都安装了 express，但是版本却不一样。

理论上来说，这两个版本应该是兼容的，但是可能那个修复的 BUG 影响到了我们所使用的功能，使得在这两个不同的版本上出现不同运行结果。

## Package-lock.json

### 目的

`package-lock.json`诞生的目的是为了防止出现我们上述的情况。同一个`package.json`却产生了不同的运行结果。`package-lock.json`在 npm 5 时添加进来，所以如果你使用 5 以上的版本，你就会看到这个文件，除非你手动禁用掉它。

### 格式

`package.json`是一个包含你所有依赖的巨大列表，它包含明确的版本号（没有通配符），依赖的获取地址，一个用于验证完整性和正确性的哈希值，以及这个依赖本身所需要的依赖。我们来看一看 express 的

```javascript
"express": {
  "version": "4.15.4",
  "resolved": "https://registry.npmjs.org/express/-/express-4.15.4.tgz",
  "integrity": "sha1-Ay4iU0ic+PzgJma+yj0R7XotrtE=",
  "requires": {
    "accepts": "1.3.3",
    "array-flatten": "1.1.1",
    "content-disposition": "0.5.2",
    "content-type": "1.0.2",
    "cookie": "0.3.1",
    "cookie-signature": "1.0.6",
    "debug": "2.6.8",
    "depd": "1.1.1",
    "encodeurl": "1.0.1",
    "escape-html": "1.0.3",
    "etag": "1.8.0",
    "finalhandler": "1.0.4",
    "fresh": "0.5.0",
    "merge-descriptors": "1.0.1",
    "methods": "1.1.2",
    "on-finished": "2.3.0",
    "parseurl": "1.3.1",
    "path-to-regexp": "0.1.7",
    "proxy-addr": "1.1.5",
    "qs": "6.5.0",
    "range-parser": "1.2.0",
    "send": "0.15.4",
    "serve-static": "1.12.4",
    "setprototypeof": "1.0.3",
    "statuses": "1.3.1",
    "type-is": "1.6.15",
    "utils-merge": "1.0.0",
    "vary": "1.1.1"
  }
},
```

所以从此以后 npm 会根据`package-lock.json`里的内容来处理和安装依赖而不是根据`package.json`。因为`pacakge-lock.json`给每个依赖标明了版本，获取地址和哈希值，使得每次安装都会出现相同的结果。不管你在什么机器上面或什么时候安装。

### 争议

现在`package-lock.json`解决了一个经常遇到的难题，但为什么大家还是想把它禁用掉呢？

在 npm 5 以前，`package.json`是唯一的依赖数据源。`package.json`就是权威。npm 用户喜欢这个样子并且已经习惯维护`package.json`文件。但是，当`package-lock.json`文件引入时，它和用户所期待的相反，对`package.json`所做的改动并不会反应到`package-lock.json`文件上。

例如：依赖 A，在`package.json`和`package-lock.json`的版本号都为`1.0.0`。手动把`package.json`里 A 的版本号改为`1.1.0`，如果一个用户认为`npm install`会依据`package.json`来安装依赖，他们期望安装的版本是`1.1.0`，结果却安装了`1.0.0`。尽管`package.json`里面写的是`1.1.0`

只因为用户不知道为什么他们的依赖没有按照预想当中样子安装，他们就宁愿禁用`package-lock.json`文件。

这个预想和现实中的冲突引发了一个特别有趣的[issue](https://github.com/npm/npm/issues/16866)。一些人支持`package.json`是依赖的权威来源，一些人支持`package-lock.json`。这个争议的解决方案在[PR #17508](https://github.com/npm/npm/pull/17508)。npm 的维护者做了一个改动：当`package.json`文件有发生变更时，使用它作为依赖的来源。现在，不管在任何情况下，依赖安装都可以满足用户的预期了。这个改动发布于 npm`v5.1.0`。

[文章来源](https://codertx.github.io/2018/01/09/about-package-json/)
