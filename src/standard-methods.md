# 标准方法

> 目录
> * [列表](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#完整资源名称)
> * [获取](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#相对资源名称)
> * [创建](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#资源id)
> * [更新](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#集合id)
> * [删除](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#资源名称-vs-url)

本章定义了标准方法的概念，即`List`，`Get`，`Create`，`Update` 和 `Delete`。定义标准方法的原因是许多跨度广泛的 API 方法有非常相似的语义。通过将这些类似的 API 融合到标准方法中，我们可以显著地降低复杂性并提高一致性。对于 Google API 仓库中的 API 方法，其中70％以上是标准方法，这使得它们更容易被学习和使用。

下表描述了如何将它们映射到REST方法，也称为CRUD方法：

| 方法 | HTTP 映射 |  HTTP 请求 Body  | HTTP 响应 Body |
| ----------------------   | -------- | ---------  | -------- |
| `List`	| `GET <collection URL>`		| Empty		| Resource* list |
| `Get`		| `GET <resource URL>`			| Empty		| Resource* 	 |
| `Create`	| `POST <collection URL>`		| Resource	| Resource* 	 |
| `Update`	| `PUT or PATCH <resource URL>`	| Resource	| Resource* 	 |
| `Delete`	| `DELETE <resource URL>`		| Empty		| Empty** 		 

*如果方法支持指定要返回的字段子集的字段掩码，则从 `List`，`Get`，`Create` 和 `Update` 方法返回的资源可能包含部分数据。 在某些情况下，API平台本身支持所有方法的字段掩码。

**从一个 `Delete` 方法返回的响应应该包含长时间运行的操作或者被修改的资源，这个 `Delete` 方法不会立即删除资源（例如更新标志位或创建一个长时间运行的删除操作）。

标准方法还可以为在单个API调用的时间跨度内不能完成的请求返回长时间运行的操作。

以下部分详细介绍了每种标准方法。这些示例显示了用于 `HTTP` 映射的 `.proto` 文件中定义的方法，它们使用了特别的注释。您可以在 [Google API](https://github.com/googleapis/googleapis) 代码库中找到在产品中使用标准方法的许多示例。

## 列表 `List`

`List` 方法使用集合名称以及零个或多个参数作为输入，并返回与输入匹配的资源列表。它通常也用于搜索资源。

`List` 使用与单个集合的数据，该数据集的数量是有限的，并且不是被缓存的。对于更广泛的情况，应使用自定义方法 `Search`。

批获取方法（例如获取多个资源ids并为每个ids返回一个对象的方法）应该实现自定义 `BatchGet` 方法，而不是 `List`。但是，如果您已有一个提供相同功能的 `List` 方法，则可以重用此方法来实现此目的。如果你使用自定义 `BatchGet` 方法，则应将其映射到 `HTTP GET`。

适用的通用模式：[分页]()，[结果排序]()。

适用的命名约定：[筛选字段]()，[结果字段]()。

HTTP 映射：

* `List` 方法必须使用一个 `HTTP GET` 动作。
* 拥有被列出集合名称的请求字段，它必须映射到 `URL` 路径中。 如果集合名称映射到URL路径，则URL模板的最后一个部分（集合 `ID`）必须是文字。
* 所有剩余的请求消息字段应映射到 `URL` 参数中。
* 没有请求体；`API` 配置绝对不能声明一个 `body` 子句。
* 响应正文应包含资源列表以及可选元数据。

```proto3

// 列出给定货架上的所有图书。
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  // 列出到 HTTP GET 的方法映射。
  option (google.api.http) = {
    // `parent` 捕获父资源名称，例如 `"shelves/shelf1"`
    get: "/v1/{parent=shelves/*}/books"
  };
}

message ListBooksRequest {
  // 父资源名称, 例如, `"shelves/shelf1"`。
  string parent = 1;

  // 最大返回数量。
  int32 page_size = 2;

  // 从上一个 `List` 请求返回的 `next_page_token` 值（如果有）。
  string page_token = 3;
}

message ListBooksResponse {
  // 字段名称应与方法名称中的名词 “books” 匹配。
  // 根据请求中的page_size字段，将返回最大数量的项目。
  repeated Book books = 1;

  // 为了取回下一页结果的属性, 如果列表中没有更多的结果了将返回空。
  string next_page_token = 2;
}

```
## 获取 `Get`

`Get` 方法接受资源名称、零个或多个参数，并返回指定的资源。

`HTTP` 映射：

* `Get` 方法必须使用 `HTTP Get` 动作。
* 接收资源名称的请求字段应该映射到 `URL` 路径。
* 所有剩余的请求消息字段应映射到URL查询参数。
* 没有请求体; API配置不能声明一个body子句。
* 返回的资源应映射到整个响应体。

```proto3

// 获取特定的书籍.
rpc GetBook(GetBookRequest) returns (Book) {
  // Get 方法对应 HTTP GET。 资源名被映射到 URL 上。 没有请求体。
  option (google.api.http) = {
    // 请注意 `URL` 模板变量，它捕获所请求书籍的多段资源名称，例如 `"shelves/shelf1/books/book2"`
    get: "/v1/{name=shelves/*/books/*}"
  };
}

message GetBookRequest {
  // 该字段将包含所请求资源的名称，例如 `"shelves/shelf1/books/book2"`
  string name = 1;
}

```








