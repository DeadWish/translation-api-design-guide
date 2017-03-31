# Protocol Buffers v3

本章将讨论如何使用 Protocol Buffers 与 API 设计。为了简化开发人员的体验并提高运行时效率，gRPC API 应使用Protocol Buffers v3（proto3） API 定义。

[Protocol Buffers](https://github.com/google/protobuf) 是一种用于定义数据结构模式和编程接口的简单的语言中立和平台中立的接口定义语言（IDL）。它支持二进制和文本线格式，并且可以在不同平台上使用许多不同的线路协议。Proto3 是最新版本的 Protocol Buffers，并且与 proto2 相比包含以下更改：

* 字段存在，也称为 `hasField`，不适用于原始字段。未设置的原始字段具有语言定义的默认值。
	* 消息字段的存在仍然可用，可以使用编译器生成的 `hasField` 方法或与 `null` 进行比较，或者由实现定义的标记值进行测试。
* 用户定义的字段默认值不再可用。
* 枚举定义必须以枚举值零开始。
* 必须字段已不再可用。
* 扩展程序已不再可用。请改用 `google.protobuf.Any`。
	* 由于向后兼容性和运行时兼容性的原因，`google/protobuf/descriptor.proto` 特殊例外。
* 删除组语法。

删除这些功能的原因是使 API 设计更简单，更稳定，更高效。例如，在记录消息之前，通常需要过滤某些字段，例如删除敏感信息。如果需要这些字段则是不可能的。

详情请查看 [Protocol Buffers](https://developers.google.com/protocol-buffers/)