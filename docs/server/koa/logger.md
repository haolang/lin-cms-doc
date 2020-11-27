---
title: 行为日志
---

# <H2Icon /> 行为日志

## 行为日志(logger)

在诸多 CMS 系统中，权限被分配给用户后，那么就代表这一用户拥有某个权限的绝对控制
权。但是有些权限又比较危险，比如删除所有图书这个权限，它很可能直接清除了系统里面
的所有图书数据，这显然是一个危险的操作。那么这么危险的操作，如果发生了，作为管理
员你该怎么办了。很简单，找到该用户，并禁用该用户，随后联系开发者，恢复数据。

logger 主要用来解决这一类的问题，当然你可以将记录任何你觉得敏感的操作，比如某用
户访问了某私密数据。接下来，我们来实操一下 logger 的使用。

我们修改一个视图函数来看一看 logger 的使用：

```js
test.linGet(
  "getTestMsg",
  "/json",
  test.permission('测试日志记录'),
  loginRequired,
  logger("就是皮了一波"), // logger，参数为日志内容
  async ctx => {
    ctx.json({
      msg: "物质决定意识，经济基础决定上层建筑"
    });
  }
);
```

接下来，当有任何用户请求这个 API 时，均会在数据库中写入一条日志信息。该日志信息
的数据模型的定义在`lin-cms`中，对应的数据表名为`lin_log`。

但有时，*就是皮了一波*这样的信息未免显得太过于单薄，它无法很好的向前端说明更多的
信息。因此 Lin 提供了一个简单的模板语法，你可以在`template`这个参数中，写入一些
变量，如`{user.username}就是皮了一波`，请记住每一个`{}`中就可以写入一个变量
，`user.username`就表示当前用户的昵称。如下：

```js
test.linGet(
  "getTestMsg",
  "/json",
  test.permission('测试日志记录'),
  loginRequired,
  logger("{user.username}就是皮了一波"), // logger，参数为日志内容
  async ctx => {
    ctx.json({
      msg: "物质决定意识，经济基础决定上层建筑"
    });
  }
);
```

此时，你每请求一次这个 API，它就会在数据中写下下面类似的信息（请注意，这个 API
现在请求必须登陆）。

```bash
+----+---------------------+---------------------+---------+-----------+-------------+--------+------------+-----------+
| id | message             | time                | user_id | user_name | status_code | method | path       | permission |
+----+---------------------+---------------------+---------+-----------+-------------+--------+------------+-----------+
|  1 | pedro就是皮了一波 | 2018-10-24 18:11:40 |       4 | pedro     |         200 | GET    | /v1/book/1 |           |
+----+---------------------+---------------------+---------+-----------+-------------+--------+------------+-----------+
1 row in set (0.13 sec)
```

我们的模板语法，不仅仅可以在其中嵌入`user`这个实例，你还可以嵌入 koa reponse 的
任何属性，如：`response.status`，还有 koa request 的诸多属性，如`request.url`。

| name     |       说明        |  类型  |
| -------- | :---------------: | :----: |
| user     |  userModel 实例   | object |
| response | koa Response 实例 | object |
| request  | koa Request 实例  | object |

这一切取决于你的需求，关于 user 的所有属性，请阅读 userModel 中的所有属性
，response 和 request 的所有属性，请阅读[koa 文档](https://koajs.com/)。

<RightMenu />