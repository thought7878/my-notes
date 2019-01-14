# 使用 mongoose 的工作流程

## 装包

```bash
npm i mongoose
```

## 连接 MongoDB 数据库

- 代码：

```js
// mongoose START
const mongoose = require('mongoose')
mongoose.connect(
  'mongodb://localhost:27017/express-love-api',
  { useNewUrlParser: true }
)
const db = mongoose.connection
db.on('error', () => console.error('Mongo Failed to Connect!!!!'))
db.on('connected', () => console.log('Mongo Connected'))
// mongoose END
```

导入 mongoose 功能模块，然后通过 Mongoose 提供的 connect() 方法连接到名为 express-love-api 数据库。下面设置一下链接失败和成功的提示信息。

modemon 启动后，所在的终端中，可以看到 Mongo Connected 信息。

## 构建 model

api/models/post.js

```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema
const PostSchema = new Schema(
  {
    category: { type: String },
    title: { type: String, required: true },
    body: { type: String }
  },
  { timestamps: true }
)
module.exports = mongoose.model('Post', PostSchema)
```

- 导入 mongoose 功能模块以及调用它提供的 Schema() 接口创建一个新的 schema。
- 每个 schema 会映射为 MongoDB 数据库中的一个 collection（集合），同时还能定义所映射集合包含的字段，以及字段的类型等规范。
- 上面代码就创建了一个名为 PostSchema 的 schema, 并规定所映射的集合将包含三个字段：category、title 和 body，并且每个字段只能存储字符串类型的数据，其中 title 字段中存储的数据不能为空。

- 选项 timestamps 的值设置为 true，则自动给所映射集合添加 createdAt 和 updatedAt 两个字段。

- 通过 Mongoose 的 model() 方法把一个 schema 编译成一个 model，一个 model 实例会对应映射集合中的一条记录，这个 model() 方法的第一个参数 Post 则是映射集合名字的单数形式，所以 PostSchema 映射集合的名字是 posts。
- 上述代码还把构建成的 Post Model 导出供外部其它文件使用。

## 使用 model-增删改查

## 增

### body-parser

服务器端要接收客户端从 HTTP 的请求 body 中携带的 JSON 数据，需要 body-parser 来解析。

- 安装

```bash
npm i body-parser
```

- 使用

```js
const bodyParser = require('body-parser')

app.use(bodyParser.json())
```

导入 bodyParser ，以中间间的形式加载 bodyParser 解析 json 数据的功能。这样 API 中，就能通过 req.body 获取到客户端传递过来的数据了。

### 增

- 代码：

```js
const Post = require('./models/post')
app.post('/post', async (req, res) => {
  const p = new Post(req.body)
  try {
    const post = await p.save()
    res.json(post)
  } catch (err) {
    res.status(500).json({ msg: '保存失败', err })
  }
})
```

- 导入 Post model
- 在创建新文章的 API 中，通过 body-parser 解析过的 req.body 中的数据，创建一个 model 实例 p
- 执行 p.save() 就可以把数据保存到数据库
- 用 try/catch 分别进行数据保存和返回错误信息的操作。

Postman 中，发出 POST /post 请求。

### 测试 API

这个涉及到接受客户端传递的复杂 JSON 数据。

#### Postman

- 发出 POST /post 请求，负载数据 body 选 raw ，格式选择 application/json ，然后填写 json 数据。
- Postman 中，点 send 发出请求。title 数据不为空的时候，会成功返回新创建的 post 对象，删除 title 数据，返回报错信息。

#### curl

```bash
curl -H "Content-Type: application/json" -X POST -d '{"title":"happypeter"}' http://localhost:3000/post
```

- 这个 curl 命令模拟出来的请求，跟项目中用 axios 发送 POST 方法，数据格式为 JSON 的请求是完全等效的。

- -H 选项后面跟的是 HTTP 报头，设置数据格式为 json
- -d 后面跟的是负载数据( payload )

## 查

### 查所有数据

- 代码：

```js
app.get('/posts', async (req, res) => {
  try {
    const posts = await Post.find()
    res.json(posts)
  } catch (err) {
    res.status(500).json({ msg: '读取失败', err })
  }
})
```

- 读操作，一般用 GET 方法，所有文章，英文叫 posts ，所以这个 API 的路由就要写成 app.get 然后第一个参数是 posts 。这样客户端如果发出 GET /posts 请求，回调函数中的语句就会被执行。

- 通过 res.json 语句，把数据以 JSON 的格式传递给客户端。

- catch 语句可以把数据库查询中的错误捕捉到，然后通过 res.json 接口反馈给客户端。

Postman 中，发起 GET /posts 请求，可以看到返回的 posts 数组。

### 查一条数据

再来添加查询一篇文章的接口。

- 代码：

```js
app.get('/post/:id', async (req, res) => {
  try {
    const post = await Post.findById(req.params.id)
    res.json(post)
  } catch (err) {
    res.status(500).json({ msg: '读取失败', err })
  }
})
```

Postman 中，发出 GET /post/5ae0747cde0f07377ddff18e ，可以看到返回了一篇文章的信息。

## 改

- 代码：

```js
app.put('/post/:id', async (req, res) => {
  try {
    const p = await Post.findById(req.params.id)
    for (prop in req.body) {
      p[prop] = req.body[prop]
    }
    const post = await p.save()
    res.json(post)
  } catch (err) {
    res.status(500).json({ error: err.message })
  }
})
```

- 根据 Rest 的规范，更新用 put 方法。
- 如果执行 p.save() 的时候，发现 title 为空，这样 model 文件中，咱们刚刚设置的规则就会被触犯，这样，catch 语句就会被执行。

测试：

```bash
curl -X PUT -H 'Content-Type: application/json' -d '{"title": "newTitle"}' http://localhost:5000/post/593607542cf2f60539a17692
```

测试可以用 curl 命令。也可以用等价的 Postman 操作。

可以看到 Postman 返回了更新后的 post 信息。

## 删

- 代码：

```js
app.delete('/post/:id', async (req, res) => {
  try {
    const p = await Post.findById(req.params.id)
    const post = await p.remove()
    res.json(post)
  } catch (err) {
    res.status(500).json({ error: err.message })
  }
})
```

Postman 中测试一下，首次删除返回被删除的对象，再次删除，因为已经不存在被删对象了，所以就会报错。

等价的 curl 命令如下：

```bash
curl -X DELETE http://localhost:3000/post/134214321432
```
