# Wind2THS

一条命令将 Wind（万得）Python 量化代码迁移为同花顺 iFinD API 代码。覆盖 10+ 函数、50+ 字段映射，自动处理字段分隔符、日期格式、认证方式、返回值格式等所有差异。

## 使用

```
/wind2ths <文件路径>
```

## 参考文档

迁移规则存储在 `references/` 目录下，slash command 执行时会自动读取：

- `api-mapping.md` — 核心函数映射（签名、参数、转换规则）
- `field-mapping.md` — 字段名映射（Wind 指标 → THS 指标）
- `return-format.md` — 返回值格式差异与转换模式
- `date-and-code-format.md` — 日期格式与证券代码格式转换
- `authentication.md` — 认证方式差异与最佳实践
- `examples.md` — 7 个完整的前后对比示例
