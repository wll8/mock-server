# 尝试
本文仅简单演示 mockm 的部分功能, 你可以选择感兴趣的部分尝试.

## 以简单对象的方式写接口
让我们来写一个自己的 api, 下面演示直接写 `api 路径`和`返回值`的方式来生成接口.

在刚刚运行 mm 的目录创建一个 `mm.config.js` 文件, 输入以下内容:

``` js
module.exports = {
  api: {
    // 表示访问 /ip 就会返回后面指定的数据
    '/ip': {
      res: `0.0.0.0`
    },
  }
}
```

然后重新启动 mm. 再使用浏览器打开 http://localhost:9000/ip , 你将会看到:
``` json
{
  "res": "0.0.0.0"
}
```

恭喜你, 实现了你的第一个 API .

再试试添加另一个 api 吧:
``` js {7-9}
module.exports = {
  api: {
    // 表示访问 /ip 就会返回后面指定的数据
    '/ip': {
      res: `0.0.0.0`
    },
    '/start': {
      msg: `hello world`
    },
  }
}
```

提示: 现在你不需要重新运行 mm 就能看到更改效果了.

使用浏览器打开 http://localhost:9000/start , 你将会看到:
``` json
{
  "msg": "hello world"
}
```


## 从接口获取请求信息
当我们需要根据接口传入的值来返回不同的内容时, 也是很容易:

``` js {4-6}
module.exports = {
  api: {
    // 表示访问 /ip 就会返回后面指定的数据
    '/ip' (req, res) {
      res.json({desc: `你传入的值`, data: req.query})
    },
    '/start': {
      msg: `hello world`
    },
  },
}
```

接下访问接口传入一些 url 参数测试一下 http://localhost:9000/ip?city=上海 结果为:

``` json
{
  "desc": "你传入的值",
  "query": {
    "city": "上海"
  }
}
```

从 req 中, 可以获取请求信息, 例如 req.query 是 url 上的参数, req.body 是 body 中的内容.
也可以从请求中获取文件、路径参数等.

## 快速生成增删改查功能的 Restful API 风格接口
假设我要写一个博客文章的列表, 并且要实现添加文章, 查询文章, 分页, 模糊搜索, 删除, 修改等各种功能的接口. 那么只需添加以下内容:

``` js {11-19}
module.exports = {
  api: {
    // 表示访问 /ip 就会返回后面指定的数据
    '/ip' (req, res) {
      res.json({desc: `你传入的值`, data: req.query})
    },
    '/start': {
      msg: `hello world`
    },
  },
  db: {
    'blogs': [
      {
        id: 1,
        content: `mockm 是一款便于使用, 功能灵活的接口工具. 看起来不错~`,
        title: `认识 mockm 的第一天`,
      },
    ],
  },
}
```

这时候上面要实现的所有接口已经实现了. 
这里我用 http 作为请求工具简单表示几个功能, 你可以使用你喜欢的工具发送请求.

``` sh
# 查看 id 为 1 的博文详情
http :9000/blogs/1

# 创建一篇关于同事的文章
http post :9000/blogs title=同事的一天 content=今天他的生活还是同样的苦涩

# 获取所有文章
http :9000/blogs

# 查询所有含有 `苦涩` 的文章
http :9000/blogs?q=苦涩
```
::: details 接口请求结果
``` sh
# 查看 id 为 1 的博文详情
http :9000/blogs/1

{
  "code": 200,
  "data": {
    "content": "mockm 是一款便于使用, 功能灵活的接口工具. 看起来不错~",
    "id": 1,
    "title": "认识 mockm 的第一天"
  },
  "success": true
}

# 创建一篇关于同事的文章
http post :9000/blogs title=同事的一天 content=今天他的生活还是同样的苦涩

{
  "code": 200,
  "data": {
    "content": "今天他的生活还是同样的苦涩",
    "id": 2,
    "title": "同事的一天"
  },
  "success": true
}

# 获取所有文章
http post :9000/blogs

{
  "code": 200,
  "data": [
    {
      "content": "mockm 是一款便于使用, 功能灵活的接口工具. 看起来不错~",
      "id": 1,
      "title": "认识 mockm 的第一天"
    },
    {
      "content": "今天他的生活还是同样的苦涩",
      "id": 2,
      "title": "同事的一天"
    }
  ],
  "success": true
}

# 查询所有含有 `苦涩` 的文章
http post :9000/blogs?q=苦涩

{
  "code": 200,
  "data": [
    {
      "content": "今天他的生活还是同样的苦涩",
      "id": 2,
      "title": "同事的一天"
    }
  ],
  "success": true
}

```
:::

::: details 接口示例
所有的创建或修改都会像真实的后台接口把操作结果存储在数据库一样.

- 基本操作
GET    /books -- 获取所有
POST   /books -- 增加一条
GET    /books/1 -- 获取某条
PUT    /books/1 -- 修改某条
PATCH  /books/1 -- 部分修改某条
DELETE /books/1 -- 删除某条

- 过滤
GET /books?discount=1&type=js -- 不同字段查询
GET /books?id=1&id=2 -- 相同字段不同的值
GET /books?author.name=张三 -- 使用点查询深层数据

- 分页
GET /books?_page=2 -- 分页
GET /books?_page=2&_limit=5 -- 分页并指定每页数量

- 排序
GET /books?_sort=view&_order=asc -- 排序
GET /books?_sort=user,view&_order=desc,asc -- 多字段排序

- 截取
GET /books?_start=2&_end=5 -- 截取 _start 到 _end 之间的内容
GET /books?_start=20&_limit=10 -- 截取 _start 后面的 _limit 条内容

- 运算
GET /books?view_gte=3000&view_lte=7000 -- 范围  `_gte` `_lte`
GET /books?id_ne=1 -- 排除 `_ne`
GET /books?type_like=css|js -- 过滤器 `_like`, 支持正则

- 全文检索
GET /books?q=张三 -- 精确全文匹配

参考 [json-server](https://github.com/typicode/json-server).

:::

## 使用数据模拟工具生成内容
mockm 自带 [mockjs](http://mockjs.com/examples.html), 下面用它生成一批用户信息:
注意 `module.exports` 的值已经变成函数, [util 中提供了一系列的工具](../config/config_fn.md).

``` js {20-28}
module.exports = util => {
  return {
    api: {
      // 表示访问 /ip 就会返回后面指定的数据
      '/ip' (req, res) {
        res.json({desc: `你传入的值`, query: req.query})
      },
      '/start': {
        msg: `hello world`
      },
    },
    db: {
      'blogs': [
        {
          id: 1,
          content: `mockm 是一款便于使用, 功能灵活的接口工具. 看起来不错~`,
          title: `认识 mockm 的第一天`,
        },
      ],
      'users': util.libObj.mockjs.mock({
        'data|15-23': [ // 随机生成 15 至 23 条数据
          {
            'id|+1': 1, // id 从 1 开始自增
            name: `@cname`, // 随机生成中文名字
            'sex|1': [`男`, `女`, `保密`], // 性别从这三个选项中随机选择一个
          },
        ]
      }).data,
    },
  }
}
```

现在访问 http://localhost:9000/users 已经可以看到很多看起来很真实的用户数据了.

::: details FQA
**无法访问接口**
- json 语法错误
- 访问的路径不是自己添加的路径 `/start` 或 `/text`

::: 