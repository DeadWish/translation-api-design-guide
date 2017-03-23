# 自定义方法

本章将讨论设计 `API` 如何使用自定义方法。

自定义方法是指除了5种标准方法之外的 `API` 方法。它们只能用于不能简单地通过标准方法表达的功能。一般情况下，`API` 设计人员应该在可行的情况下选择标准方法，而不是自定义方法。标准方法有简单和明确的语义，大多数开发人员都熟悉，所以他们更容易使用且不易出错。标准方法的另一个优点是 `API` 平台对标准方法有更好的理解和支持, 例如计费，错误处理，日志记录和监视。

自定义方法可以与资源，集合或服务相关联。它可能需要任意的请求并返回任意的响应，并且还支持流请求和响应。

## `HTTP` 映射

对于自定义方法，他们应该使用以下通用HTTP映射：

	https://service.name/v1/some/resource/name:customVerb

使用 `:` 而不是 `/` 来分隔资源名的自定义动词是为了支持任意路径。例如，取消删除文件可以映射到 `POST /files/a/long/file/name:undelete` 。

选择HTTP映射时应遵循以下准则：

* 自定义方法应该使用 `HTTP POST` 动词，因为它具有最灵活的语义。
* 自定义方法不应该使用 `HTTP PATCH`，但可能使用其他 `HTTP` 动词。在这种情况下，方法必须遵循该动词标准的 `HTTP` 语义。
* 值得注意的是，使用 `HTTP GET` 的自定义方法必须是幂等的，并且没有副作用。 例如，在资源上实现特殊视图的自定义方法应该使用 `HTTP GET`。
* 接收自定义方法与之关联的资源或集合的资源名称的请求消息字段应映射到 `URL` 路径。
* `URL` 路径的结尾后缀必须为冒号与自定义动词。
* 如果用于自定义方法的 `HTTP` 动作允许使用 `HTTP` 请求体（`POST`，`PUT`，`PATCH` 或自定义 `HTTP` 动词），则此类自定义方法的 `HTTP` 配置必须使用 `body:"*"` 选项，所有剩余的请求消息字段应映射到 `HTTP` 请求体。
* 如果用于自定义方法的 `HTTP` 动作不接受 `HTTP` 请求体（`GET`，`DELETE`），则该方法的 `HTTP` 配置必须不能使用 `body` 项，所有剩余的请求消息字段都应该映射到 `URL` 查询参数 。

警告：如果服务实现多个 `API`，`API` 生产者必须仔细地创建服务配置，以避免 `API` 之间的自定义动词冲突。

```proto3
// 这是一个服务级别的自定义方法。
rpc Watch(WatchRequest) returns (WatchResponse) {
  // 自定义方法映射到 HTTP POST. 所有请求参数都在 `body` 中.
  option (google.api.http) = {
    post: "/v1:watch"
    body: "*"
  };
}

// 这是一个集合级别的自定义方法。
rpc ClearEvents(ClearEventsRequest) returns (ClearEventsResponse) {
  option (google.api.http) = {
    post: "/v3/events:clear"
    body: "*"
  };
}

// 这是一个资源级别的自定义方法。
rpc CancelEvent(CancelEventRequest) returns (CancelEventResponse) {
  option (google.api.http) = {
    post: "/v3/{name=events/*}:cancel"
    body: "*"
  };
}

// 这是一个批量获取自定义方法。
rpc BatchGetEvents(BatchGetEventsRequest) returns (BatchGetEventsResponse) {
  // 批量获取方法映射 HTTP GET 动词.
  option (google.api.http) = {
    get: "/v3/events:batchGet"
  };
}
```

## 用例

一些其他的方案自定义方法可能是正确的选择：

* 重启虚拟机。设计可选的方案可能是“创建一个重启集合里的重启资源”，这个情况有点复杂。或者是“虚拟机具有可变状态，客户端可以从 `RUNNING` 更新为 `RESTARTING`”，这个方法将会提出哪些状态转换是可行的问题。此外，重新启动是一个众所周知的概念，它可以很好地转换为直观地满足开发者期望的自定义方法。

* 发送邮件。创建电子邮件不一定会发送（草稿）。与设计的替代方案（将消息移动到“发件箱”集合）相比，自定义方法具有 `API` 用户更易于发现的优点，并且更直接地模拟概念。

* 员工晋升。如果用标准方法 `update` 实现，客户将不得不复制管理晋升过程的公司政策，以确保晋升是在相同职业阶梯等级的正确水平。

标准方法比自定义方法更合适的几个例子：

* 使用不同的查询参数查询资源（使用标准列表过滤的标准列表方法）。
* 简单资源属性更改（使用带有字段掩码的标准更新方法）。
* 取消通知（使用标准删除方法）。

## 常用自定义方法

常用或有用的自定义方法名称策略列表如下。API设计人员在提出自己的API之前应该考虑这些名称，以便于 `API` 之间的一致性。

方法名 | 自定义动词 | HTTP 动词 | 注意  
------- | ------- | ------- | ------- |
`Cancel` 				| `:cancel`    | POST | 取消未完成的操作（构建，计算等） |
`BatchGet<plural noun>` | `:batchGet`  | GET  | 批量获取多个资源（详情请查看[列表的描述]（https://github.com/DeadWish/translation-api-design-guide/blob/master/src/standard-methods.md#列表-list））|
`Move` 					| `:move`      | POST | 将资源从一个父项移动到另一个父项。 |
`Search` 				| `:search`    | GET  | 用于获取不符合 `List` 语义的数据的 `List` 的替代方法。 |
`Undelete`		       	| `:undelete`  | POST | 恢复以前删除的资源。推荐保留期为30天。 |





