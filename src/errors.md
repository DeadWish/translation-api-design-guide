# 错误

本章提供了 Google API 错误模型的概述以及开发人员如何正确生成和处理错误的通用指导。

Google API 使用简单的协议无关的错误模型，这使我们能够在不同的 API 协议（如gRPC或HTTP）以及错误上下文（例如异步，批处理或工作流错误）中得到一致的体验。

## 错误模型

错误模型在逻辑上由 `google.rpc.Status` 定义，当发生 API 错误时，返回该模型的一个实例给客户端。 以下代码片段展示了错误模型的总体设计：
```proto3
package google.rpc;

message Status {
  // 一个简单的错误代码，可以很容易地由客户端处理。 实际的错误代码由“google.rpc.Code”定义。
  int32 code = 1;

  // 面向开发人员的可读取的英文错误消息。 它应该不解释错误，并提供可行的解决方案。
  string message = 2;

  // 客户端代码可用于处理错误的其他错误信息，例如延迟重试或提供帮助链接。
  repeated google.protobuf.Any details = 3;
}
```

由于大多数 Google API 都使用面向资源的 API 设计，所以错误处理遵循相同的设计原则，通过使用一小批标准错误来设置大量资源的错误。例如，服务器不是定义不同种类的“未找到”错误，而是使用一个标准的 `google.rpc.Code.NOT_FOUND` 错误代码，告诉客户端没有找到哪个特定的资源。较小的状态空间降低了文档的复杂性，在客户端库中提供了更好的惯用映射，并降低了客户端逻辑复杂性，同时不限制可操作信息的包含。

## 错误码

Google API 必须使用 `google.rpc.Code` 定义的规范错误代码。个人的 API 应避免定义其他错误代码，因为开发人员不太可能会写逻辑来处理大量的错误代码。作为参考，每个API调用平均处理3个错误代码意味着大多数应用程序逻辑只是用于错误处理，这不是一个好的开发者体验。

## 错误信息

错误消息可以帮助用户轻松快速地了解和解决API错误。一般来说，在撰写错误消息时，请考虑以下准则：

* 不要以为该用户是你的API的专家用户。用户可以是客户端开发人员，操作人员，IT人员或应用程序的最终用户。
* 不要以为用户了解您的服务实现或熟悉错误的上下文（如日志分析）。
* 如果可能，应构建技术用户（但不一定是API的开发人员）可以对错误进行响应并进行更正的错误消息。
* 保持错误消息简短。如果需要，请提供一个链接，让困惑的读者可以提出问题，提供反馈或获取不完全符合错误信息的更多信息。或者，使用详细信息字段展开。

## 错误详情

Google API定义了一组错误详细信息的标准错误有效内容，您可以在 [google/rpc/error_details.proto](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto) 中找到。这些涵盖了API错误的最常见需求，例如磁盘配额失败和无效参数。像错误代码一样，错误详情应尽可能使用这些标准的有效内容。

只有在辅助应用程序代码来处理错误时，才应引入其他错误详细信息类型。如果错误信息只能由人类处理，请依赖于错误消息内容，让开发人员手动处理，而不是引入新的错误详细信息类型。请注意，如果引入了其他错误详细信息类型，则必须明确注册它们。

以下是一些示例 `error_details` 的有效内容：
* RetryInfo 描述客户端可以重试失败的请求，可能会在 `Code.UNAVAILABLE` 或 `Code.ABORTED` 上返回。
* QuotaFailure 描述硬盘配额检查如何失败，可能会在 `Code.RESOURCE_EXHAUSTED` 上返回。
* BadRequest 描述客户端请求中的违规行为，可能会在 `Code.INVALID_ARGUMENT` 上返回。

## HTTP 映射

虽然 proto3 消息具有本机 JSON 编码，但 Google 的 API 平台使用以下 JSON 表示形式进行直接 HTTP-JSON 错误响应，以允许向后兼容性：

```proto3
{
  "error": {
    "code": 401,
    "message": "Request had invalid credentials.",
    "status": "UNAUTHENTICATED",
    "details": [{
      "@type": "type.googleapis.com/google.rpc.RetryInfo",
      ...
    }]
  }
}
```
字段 | 描述
------- | ------- | -------  
`error` | 为了向后兼容 Google API 客户端库的外加层，它还使得 JSON 对人类更易读。 
`code` | HTTP 状态码，对应 `Status.code`
`message` | 这对应于 `Status.message` 
`status` | 这对应于 `Status.code`  
`details` | 这对应于 `Status.details` 

## RPC 映射

不同的RPC协议以不同的方式映射错误模型。对于 gRPC，错误模型由生成代码和每种支持语言的运行时库本地支持。你可以在 gRPC API 文档中找到更多内容（例如，请参阅 gRPC Java [`io.grpc.Status`](https://github.com/grpc/grpc-java/blob/master/core/src/main/java/io/grpc/Status.java)）。

## 客户端库映射

Google 客户端库为了和已建立的习惯表达方式保持一致，可能会对不同的语言进行不同的处理。例如，`google-cloud-go` 库将返回一个实现了 `google.rpc.Status` 接口的错误，而 `google-cloud-java` 将引发一个 `Exception`。

## 错误本地化

`google.rpc.Status` 中的消息字段是面向开发人员的，必须是英文。

如果需要面向用户的错误消息，请使用 `google.rpc.LocalizedMessage` 作为您的详细信息字段。虽然 `google.rpc.LocalizedMessage` 中的消息字段可以进行本地化，但请确保 `google.rpc.Status` 中的消息字段为英文。默认情况下，API 服务应使用经过身份验证的用户的区域设置或 HTTP `Accept-Language` 头来确定本地化的语言。

## 处理错误

以下是包含 `google.rpc.Code` 中定义的所有 gRPC错误代码的表格，以及其原因的简短说明。要处理错误，您可以检查返回的状态代码的描述，并相应地修改您的调用。

HTTP | RPC | 描述  
------- | ------- | -------  
200 | `OK` | 无错误  
400 | `INVALID_ARGUMENT ` |  客户端指定了无效参数。 检查错误消息和错误详细信息以获取更多信息。
400 | `FAILED_PRECONDITION ` | 请求无法在当前系统状态下执行，如删除非空目录。  
400 | `OUT_OF_RANGE ` | 客户端指定了无效的范围。  
401 | `UNAUTHENTICATED ` | 由于缺少，无效或过期的OAuth令牌，请求未通过身份验证。  
403 | `PERMISSION_DENIED ` | 客户端没有足够的权限。这可能是因为 OAuth 令牌没有正确的范围，客户端没有权限，还没有为客户端项目启用 API。 
404 | `NOT_FOUND ` | 未找到指定的资源，或者由于未公开的原因（如白名单）拒绝该请求。  
409 | `ABORTED ` | 并发冲突，如读 - 修改 - 写冲突。
409 | `ALREADY_EXISTS ` | 客户端尝试创建的资源已经存在。  
429 | `RESOURCE_EXHAUSTED ` | 资源配额用完或者达到限额。客户端应该查找 `google.rpc.QuotaFailure` 错误详细信息以获取更多信息。
499 | `CANCELLED ` | 请求被客户取消。  
500 | `DATA_LOSS ` | 不可恢复的数据丢失或数据损坏。客户端应该向用户报告错误。 
500 | `UNKNOWN ` | 未知服务器错误。通常是服务器错误。  
500 | `INTERNAL ` | 内部服务器错误。通常是服务器错误。  
501 | `NOT_IMPLEMENTED	` | API 方法未由服务器实现。 
503 | `UNAVAILABLE ` | 暂停服务。通常服务器关闭。  
504 | `DEADLINE_EXCEEDED ` | 请求超过最后期限。如果重复发生，请考虑减少请求的复杂性。  

## 重试错误

客户端应该使用指数退避算法重试 500，503 和 504 的错误，最小延迟应为1秒，除非另有说明。对于429错误，客户端可以延迟30秒重试。对于所有其他错误，重试可能不适用——首先确保您的请求是幂等的，并查看错误消息以获得指导。

## 错误传播

如果你的 API 服务依赖于其他服务，则不应盲目地将这些服务的错误透传给客户。翻译错误时，我们建议如下：

* 隐藏实施细节和机密信息。
* 调整负责该错误的一方。例如，从其他服务接收 `INVALID_ARGUMENT` 错误的服务器应将 `INTERNAL` 传播到其自己的调用方。

## 产生错误

如果您是服务器开发人员，您应该产生足够的信息来帮助客户开发人员了解并解决问题。同时，您必须注意用户数据的安全性和隐私，并避免在错误消息和错误详细信息中暴露敏感信息，因为错误经常被记录并可能被其他人访问。例如，诸如“客户端IP地址不在白名单 128.0.0.0/8”之类的错误消息暴露了有关服务器端策略的信息，用户应该无法访问这些信息。

为了产生正确的错误，您首先需要熟悉 [`google.rpc.Code`](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)，为每个错误条件选择最合适的错误代码。服务器应用程序可以并行检查多个错误条件，并返回第一个。

下表列出了每个错误代码并有一个良好的错误消息示例。（错误消息应该是英文的，因此消息示例不翻译。）

HTTP | RPC | 错误消息示例  
------- | ------- | -------  
400 | `INVALID_ARGUMENT ` | Request field x.y.z is xxx, expected one of [yyy, zzz].  
400 | `FAILED_PRECONDITION ` | Resource xxx is a non-empty directory, so it cannot be deleted.  
400 | `OUT_OF_RANGE ` | Parameter 'age' is out of range [0, 125].  
401 | `UNAUTHENTICATED` | Invalid authentication credentials.  
403 | `PERMISSION_DENIED ` | Permission 'xxx' denied on file 'yyy'.  
404 | `NOT_FOUND ` | Resource 'xxx' not found.  
409 | `ABORTED ` | Couldn’t acquire lock on resource ‘xxx’.  
409 | `ALREADY_EXISTS ` | Resource 'xxx' already exists.  
429 | `RESOURCE_EXHAUSTED ` | Quota limit 'xxx' exceeded.  
499 | `CANCELLED ` | Request cancelled by the client.  
500 | `DATA_LOSS ` | See note.  
500 | `UNKNOWN ` | See note.  
500 | `INTERNAL ` | See note.  
501 | `NOT_IMPLEMENTED ` | Method 'xxx' not implemented.  
503 | `UNAVAILABLE ` | See note.  
504 | `DEADLINE_EXCEEDED ` | See note.  

注意：由于客户端无法修复服务器错误，因此生成其他错误详细信息没有用。为了避免在错误条件下泄露敏感信息，建议不要生成任何错误消息，只生成 `google.rpc.DebugInfo` 错误详细信息。`DebugInfo` 是专门为服务器端日志记录而设计的，必须不能发送给客户端。

`google.rpc` 软件包定义了一组标准错误有效数据，它们优先于自定义错误有效数据。下表列出了每个错误代码及其匹配的标准错误有效数据（如果适用）。

HTTP | RPC | 推荐错误详情  
------- | ------- | -------  
400 | `INVALID_ARGUMENT ` | `google.rpc.BadRequest` 
400 | `FAILED_PRECONDITION ` | `google.rpc.PreconditionFailure`  
400 | `OUT_OF_RANGE ` | `google.rpc.BadRequest`
401 | `UNAUTHENTICATED` |  
403 | `PERMISSION_DENIED ` |  
404 | `NOT_FOUND ` |  
409 | `ABORTED ` | 
409 | `ALREADY_EXISTS ` |  
429 | `RESOURCE_EXHAUSTED ` | 	`google.rpc.QuotaFailure`
499 | `CANCELLED ` |  
500 | `DATA_LOSS ` | 
500 | `UNKNOWN ` |  
500 | `INTERNAL ` | 
501 | `NOT_IMPLEMENTED ` |  
503 | `UNAVAILABLE ` | 
504 | `DEADLINE_EXCEEDED ` | 



