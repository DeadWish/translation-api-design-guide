# 标准方法

> 目录
> * [列表](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/standard-methods.md#列表-list)
> * [获取](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/standard-methods.md#获取-get)
> * [创建](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/standard-methods.md#创建-create)
> * [更新](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/standard-methods.md#更新-update)
> * [删除](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/standard-methods.md#删除-delete)

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

* `Get` 方法必须使用 `HTTP GET` 动作。
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

## 创建 `Create`

`Create` 方法使用集合名称、资源、零个或多个参数。 它在指定的集合中创建一个新资源，并返回新创建的资源。

如果API支持创建资源，它应该为每种可以创建的资源类型准备一个 `Create` 方法。

`HTTP` 映射：

* `Create` 方法必须使用 `HTTP POST` 动作。
* 请求消息应该有一个名为 `parent` 的字段，以接收要创建资源的父资源名称。
* 所有剩余的请求消息字段应映射到 `URL` 查询参数
* 请求可能包含名为 `<resource>_id` 的字段，允许请求者选择客户端分配的ID。此字段必须映射到 `URL` 查询参数。
* 包含资源的请求字段应映射到请求主体（ `Request body` ）。 如果 `body` 的 `HTTP` 配置项使用 `Create` 方法，则必须使用 `body: "<resource_field>"` 形式。
* 返回的资源应映射到整个响应体（ `Reponse body` ）。

如果 `Create` 方法支持客户端分配的资源名称，并且该资源已经存在，它应该返回失败（建议使用错误 `google.rpc.Code.ALREADY_EXISTS` ）或使用服务器分配的不同的资源名称，文档应该描述清楚，资源名称可能与传入的不同。

```proto3
rpc CreateBook(CreateBookRequest) returns (Book) {
  // `Create` 映射到 `HTTP POST`。 `URL` 路径与资源名对应。
  // `HTTP` 请求体包含资源.
  option (google.api.http) = {
    // `parent` 捕获父资源名称, 例如 `"shelves/1"`。
    post: "/v1/{parent=shelves/*}/books"
    body: "book"
  };
}

message CreateBookRequest {
  // 要创建图书的父资源名称。
  string parent = 1;

  // 这本书要使用的 `id`
  string book_id = 3;

  // 要创建书的资源。
  // 字段名称应与方法名称中的名词匹配。
  Book book = 2;
}

rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = {
    post: "/v1/shelves"
    body: "shelf"
  };
}

message CreateShelfRequest {
  Shelf shelf = 1;
}
```

## 更新 `Update`

`Update` 方法接收包含资源和零个或多个参数的请求消息。它更新指定的资源及其属性，并返回更新后的资源。

资源的可变属性应该可以通过 `Update` 方法可变，但包含资源名称或父项的属性除外。任何重命名或移动资源的功能都不能包含在 `Update` 方法中，而应由自定义方法处理。

`HTTP` 映射：

* 标准的 `Update` 方法应支持更新部分资源，并使用名为 `update_mask` 的 `FieldMask` 字段的 `HTTP` 动作 `PATCH`。
* 需要更高级修补语义的更新方法（例如附加到重复字段），应该通过自定义方法提供。
* 如果 `Update` 方法仅支持完全资源更新，则它必须使用 `HTTP` 动作 `PUT`，但是极其不鼓励使用它，因为它在添加新的资源字段时有向后兼容性问题。
* 接收资源名称的消息字段必须映射到 `URL` 路径。该字段可以在资源消息本身中。
* 包含资源的请求消息字段必须映射到请求主体（Request body）。
* 所有剩余的请求消息字段必须映射到 `URL` 查询参数。
* 响应消息必须是更新的资源本身。

如果 `API` 接受客户端分配的资源名称，则服务器可以允许客户端指定不存在的资源名称并创建新资源。 否则，`Update` 方法应该返回不存在资源名称的失败信息。 如果它是唯一的错误，应该使用错误代码 `NOT_FOUND`。

具有支持资源创建的 `Update` 方法的 `API` ，也还应提供 `Create` 方法。原理是，如果 `Update` 方法是唯一的方法，它不清楚如何创建资源。

```proto3
rpc UpdateBook(UpdateBookRequest) returns (Book) {
  // `Update` 映射 `HTTP PATCH`。 `URL` 路径包括资源名。
  // 资源被包含在 `HTTP` 请求体中（Request body）。
  option (google.api.http) = {
    // 请注意 `URL` 模板变量，它捕获要更新的图书的资源名称。
    patch: "/v1/{book.name=shelves/*/books/*}"
    body: "book"
  };
}

message UpdateBookRequest {
  // 替换服务器上资源的图书资源。
  Book book = 1;

  // 适用于资源的更新掩码。对于 `FieldMask` 的定义,
  // 请查看 https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#fieldmask
  FieldMask update_mask = 2;
}
```

## 删除 `Delete`

`Delete` 方法使用资源名称和零个或多个参数，并删除或计划删除指定的资源。`Delete` 方法应该返回 `google.protobuf.Empty`。

请注意，`API` 不应依赖于由 `Delete` 方法返回的任何信息，因为它不能被重复调用。

`HTTP` 映射

* `Delete` 方法必须使用 `HTTP DELETE` 动作。
* 接收资源名称的请求字段应该映射到 `URL` 路径。
* 所有剩余的请求消息字段应映射到 `URL` 查询参数。
* 没有请求体；`API` 配置不能声明一个 `body` 项。
* 如果 `Delete` 方法立即删除资源，它应该返回一个空响应。
* 如果 `Delete` 方法启动长时间运行的操作，它应该返回长时间运行操作。（长时间运行操作指 `long-running operation`）
* 如果 `Delete` 方法只将资源标记为已删除，它应该返回更新的资源。

对 `Delete` 方法的调用应该是等幂的，但不需要产生相同的响应。任何数量的删除请求应该导致资源被（最终）删除，但只有第一个请求应该得到成功码，后续请求应导致 `google.rpc.Code.NOT_FOUND`。

```proto3
rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty) {
  // `Delete` 映射 `HTTP DELETE`。 资源名称映射到 `URL` 路径.
  // 没有请求体（Request body）。
  option (google.api.http) = {
    // 注意，`URL` 模板变量捕获要删除图书资源的多段名称，例如 `"shelves/shelf1/books/book2"`
    delete: "/v1/{name=shelves/*/books/*}"
  };
}

message DeleteBookRequest {
  // 被删除的资源名称, 例如 `"shelves/shelf1/books/book2"`
  string name = 1;
}
```