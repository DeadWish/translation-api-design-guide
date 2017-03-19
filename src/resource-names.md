# 资源名称

> 目录
* [完整资源名称](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#完整资源名称)
* [相对资源名称](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#相对资源名称)
* [资源 ID](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#资源id)
* [集合 ID](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#集合id)
* [资源名称 vs URL](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#资源名称-vs-url)
* [字符串的资源名称](https://github.com/DeadWish/translation-api-design-guide/blob/master/src/resource-names.md#字符串的资源名称)

在面向资源的API中，资源是命名实体，资源名称是其标识符。每个资源必须有自己的唯一资源名称。资源名称由资源本身的ID，任何父资源的ID及其API服务名称组成。资源名称由资源本身的ID，任何父资源的ID及其API服务名称组成。下面我们将着眼于资源ID和如何构建资源名称。

gRPC API应该为资源名称使用无scheme的URI。它们通常遵循REST URL约定，其表现与网络文件路径非常相似。它们可以轻松映射到REST URL：有关详细信息，请参阅[标准方法](http://something)部分。例如，文件资源的集合称为目录。集合的资源ID称为集合ID。

资源名称使用集合ID和资源ID分层组织，以斜杠分隔。如果资源包含子资源，则通过指定父资源名称后面跟子资源ID，再通过斜杠分隔，形成子资源的名称。

示例1：一个存储服务具有一组`buckets`，其中每个存储桶都有一组`objects`：

| API 服务名 | 集合ID |  资源ID  | 集合ID | 资源ID |
| ----------------------   | -------- | ---------  | -------- | ---------- |
| //storage.googleapis.com | /buckets | /bucket-id | /objects | /object-id |

示例2：一个邮件服务又一组`users`。每个用户有一个子资源`settings`，一个子资源`settings`有若干其他的子资源，包扩 `customForm`：

| API 服务名 | 集合ID |  资源ID  | 集合ID | 资源ID |
| --------------------- | -------| ----------------- | --------- | ----------- |
| //mail.googleapis.com | /users | /name@example.com | /settings | /customFrom |

API生产者可以为资源和集合ID选择任何可接受的值，只要它们在资源层次结构中是唯一的即可。你可以在下面找到更多关于选择适当的资源和集合ID的指南。

通过分割资源名称便可以获取单独的集合ID和资源ID，例如`name.split（“/”）[n]`，只要每个部分不包含任何斜杠。

## 完整资源名称

无scheme的URI是由DNS兼容的API服务名称和资源路径组成的。资源路径也称为相对资源名。例如：

	"//library.googleapis.com/shelves/shelf1/books/book2"

API服务名称用于客户端定位API服务端点; 它可能是仅内部服务的假DNS名称。如果API服务名称从上下文中显而易见，则通常使用相对的资源名称。

## 相对资源名称

一个 URI 路径（没有scheme的路径）没有前导的"/"。它在 API 服务中标识一个资源。例如：

	"shelves/shelf1/books/book2"

## 资源 ID

一个非空 URI 段使用父资源来标识一个资源的例子如上所示。

资源名称中的尾随资源ID可以具有多于一个的URI段。 例如：

| 集合ID |  资源ID |
| ------ | ------ |
| files | /source/py/parser.py |

API服务应该在可行时使用对URL友好的资源ID。资源ID必须清楚地记录，不管它们是由客户端、服务器还是其他地方分配的。例如，文件名通常由客户端分配，而电子邮件消息ID通常由服务器分配。

## 集合 ID

一个非空 URI 段使用父资源来标识一个集合的例子如上所示。

由于集合ID通常出现在生成的客户端库中，因此它们必须符合以下要求：

* 必须是有效的C / C ++标识符。
* 必须以 lowerCamelCase (小写开头的驼峰命名法)的复数形式。
* 必须使用清晰简明的英语词汇。
* 应避免或限定使用过于笼统的术语。例如，`RowValue` 就优于 `Value`。在没有限定条件的情况下应避免以下术语：
	* `Element`
	* `Entry`
	* `Instance`
	* `Item`
	* `Object`
	* `Resource`
	* `Type`
	* `Value`

## 资源名称 vs URL

虽然完整的资源名称与正常的URL类似，但它们不是同一个东西。单个资源可以由不同版本的 API 和不同的 API 协议公开。完整资源名称不指定此类信息，因此实际使用必须映射到特定的协议和 API 版本。

要通过 REST API 使用完整的资源名称，必须通过在服务名称前添加 HTTPS Scheme，在资源路径之前添加 API 主要版本，以及对资源路径进行 URL 转义，将其转换为 REST URL。 例如：

	// 这是一个日历事件资源名称。
	"//calendar.googleapis.com/users/john smith/events/123"

	// 这是一个相应的 HTTP URL.
	"https://calendar.googleapis.com/v3/users/john%20smith/events/123"

## 字符串的资源名称

Google API必须使用字符串表示资源名称，除非向后兼容性是一个问题。 资源名称应该像正常文件路径一样处理，并且它们不支持 ％-encoding。

对于资源定义，第一个字段应该是资源名称的字符串，它应该称为 `name`。其他与名称相关的字段应该可以避免混淆，例如`display_name`，`first_name`，`last_name`，`full_name`。

示例：
```Javascript
service LibraryService {
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/{name=shelves/*/books/*}"
    };
  };
  rpc CreateBook(CreateBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{parent=shelves/*}/books"
      body: "book"
    };
  };
}

message Book {
  // 书的资源名称。它必须具有格式 “shelves/*/books/*”。
  // 例如：“shelves/shelf1/books/book2”。
  string name = 1;

  // ... 其他属性
}

message GetBookRequest {
  // 书的资源名称。 例如：“shelves/shelf1/books/book2”。
  string name = 1;
}

message CreateBookRequest {
  // 用来创建一本书的父资源的资源名称。
  // 例如："shelves/shelf1"。
  string parent = 1;
  // 要创建的 Book 资源。 客户端不能设置 `Book.name` 字段。
  Book book = 2;
}
```

注意：为了资源名称的一致性，最前面的斜杠不能被任何 URL 模板变量捕获。 例如，必须使用 URL 模板`"/v1/{name=shelves/*/books/*}"`，而不是`"/v1{name=/shelves/*/books/*}"`。





