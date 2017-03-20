# 标准字段

本节描述了在需要类似概念时应使用的一组标准消息字段定义。这将确保相同的概念在不同的 `API` 上具有相同的名称和语义。

| 名称 | 类型 | 描述 |
| ----- | ----- | ----- |
| `name`            | string                | `name` 字段应包含[相对资源名称]()。 |
| `parent`          | string                | 对于资源定义和列表/创建请求，`parent` 字段应包含父[相对资源名称]()。 |
| `create_time`     | Timestamp             | 实体的创建时间戳。 |
| `update_time`     | Timestamp             | 实体的最后更新时间戳。注意：执行 `create/patch/delete` 操作时，更新 `update_time`。 |
| `delete_time`     | Timestamp             | 实体的删除时间戳，仅当它支持保留时。 |
| `time_zone`       | string                | 时区名称。它应该是 IANA TZ 名称，例如 “America / Los_Angeles”。 有关详细信息，请参阅[Wiki](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。 |
| `region_code`     | string                | 地理位置的 Unicode 国家/地区码（CLDR），例如 “US” 和 “419”。有关详细信息，请参阅[Unicode](http://www.unicode.org/reports/tr35/#unicode_region_subtag)。 |
| `language_code`   | string                | BCP-47 语言代码，如 “en-US” 或 “sr-Latn”。有关详细信息，请参阅[Unicode](http://www.unicode.org/reports/tr35/#Unicode_locale_identifier)。 |
| `display_name`    | string                | 实体的显示名称。 |
| `title`           | string                | 实体的正式名称，例如公司名称。它应该被视为正式版本的 `display_name`。 |
| `description`     | string                | 描述实体的一个或多个文本段落。 |
| `filter`          | string                | `List` 方法的标准过滤器参数。 |
| `query`           | string                | 如果应用于搜索方法与过滤器相同（即[`:search`]()）。 |
| `page_token`      | string                | `List` 请求中的分页记号。 |
| `page_size`       | int32                 | `List` 请求中的分页大小。 |
| `total_size`      | int32                 | 不考虑分页时，列表中的项目总数。 |
| `next_page_token` | string                | `List` 响应中下一页的分页记号。它应该当做 `page_token` 用于下次请求。空值意味着没有更多的结果。 |
| `resume_token`    | string                | 用于恢复流式传输请求的不透明令牌。 |
| `labels`          | `map<string, string>` | 表示云资源标签。 |
| `deleted`         | bool                  | 如果资源允许取消删除行为，则它必须具有指示资源被删除的 `deleted` 字段。 |
| `show_deleted`    | bool                  | 如果资源允许取消删除行为，相应的 `List` 方法必须有一个 `show_deleted` 字段，以便客户端可以发现已删除的资源。 |
| `update_mask`     | FieldMask             | 它用于对资源执行部分更新的 `Update` 请求消息。 |
| `validate_only`   | bool                  | 如果为 `true`，则表示给定的请求应该仅被验证，而不是被执行。 |