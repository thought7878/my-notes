# 初始化 server 开发环境

## 初始化 node 项目

```bash
npm init
```

## 初始化 index.js

### 安装 express 和 脚手架工具

```bash
npm i express
npm i -g nodemon
```

```js
// index.js
const express = require('express')
const app = express()

app.get('/', (req, res) => {
  res.send('Hello Client')
})

const { PORT } = require('./config')

app.listen(PORT, () => console.log(`running on port ${PORT}...`))
```

### 启动 sever

```json
"start": "nodemon ./src/index.js",
```

## 初始化 mongoDB

```bash
mkdir data
```

### 连接 DB

```js
// index.js
// connect DB START
const mongoose = require('mongoose')
mongoose.connect(`mongodb://localhost:27017/${DB_NAME}`)
const db = mongoose.connection
db.on('error', () => console.error('Mongo Failed to Connect!!!!'))
db.on('connected', () => console.log('Mongo Connected'))
// connect DB END
```

###启动 DB

```bash
"startDB": "mongod --dbpath=./data/"
```

##

## 初始化 git

```bash
git init
```

```bash
touch .gitignore
```

```
//.gitignore
/node_modules
/data
```
