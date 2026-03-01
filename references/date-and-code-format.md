# 日期格式与证券代码格式差异

## 日期格式

### Wind 日期格式

Wind API 接受多种日期格式：
- `"20230101"` — 紧凑格式（最常用）
- `"2023-01-01"` — 带分隔符
- `datetime` 对象
- `"2023/01/01"` — 斜杠分隔

### THS iFinD 日期格式

THS iFinD 统一使用 **带连字符的格式**：
- `"2023-01-01"` — 标准格式（**唯一接受的字符串格式**）
- `datetime` 对象也可接受

### 转换规则

```python
# Wind 格式 → THS 格式

# 1. 紧凑格式 "20230101" → "2023-01-01"
wind_date = "20230101"
ths_date = f"{wind_date[:4]}-{wind_date[4:6]}-{wind_date[6:8]}"

# 2. datetime 对象 → 字符串
from datetime import datetime
dt = datetime(2023, 1, 1)
ths_date = dt.strftime("%Y-%m-%d")

# 3. 带斜杠 "2023/01/01" → "2023-01-01"
wind_date = "2023/01/01"
ths_date = wind_date.replace("/", "-")
```

### 常见日期参数位置

| Wind 函数 | 日期参数 | THS 函数 | 日期参数 |
|-----------|---------|----------|---------|
| `w.wsd(code, field, beginDate, endDate)` | 第3、4参数 | `THS_HistoryQuotes(code, field, params, beginDate, endDate)` | 第4、5参数 |
| `w.wss(code, field, date=)` | `tradeDate` 参数 | `THS_BD(code, field, params)` | `params` 中指定日期 |
| `w.wsi(code, field, beginDate, endDate)` | 第3、4参数 | `THS_HF(code, field, params, beginDate, endDate)` | 第4、5参数 |

## 证券代码格式

### Wind 证券代码

Wind 使用 **后缀标识交易所**：

| 交易所 | Wind 后缀 | 示例 |
|--------|----------|------|
| 上交所 | `.SH` | `600519.SH` |
| 深交所 | `.SZ` | `000001.SZ` |
| 北交所 | `.BJ` | `430047.BJ` |
| 港交所 | `.HK` | `00700.HK` |
| 中金所 | `.CFE` | `IF2301.CFE` |
| 上期所 | `.SHF` | `AU2306.SHF` |
| 大商所 | `.DCE` | `M2305.DCE` |
| 郑商所 | `.CZC` | `SR305.CZC` |

### THS iFinD 证券代码

THS 也使用类似的后缀格式，**大部分与 Wind 一致**：

| 交易所 | THS 后缀 | 示例 |
|--------|---------|------|
| 上交所 | `.SH` | `600519.SH` |
| 深交所 | `.SZ` | `000001.SZ` |
| 北交所 | `.BJ` | `430047.BJ` |
| 港交所 | `.HK` | `00700.HK` |
| 中金所 | `.CFE` | `IF2301.CFE` |
| 上期所 | `.SHF` | `AU2306.SHF` |
| 大商所 | `.DCE` | `M2305.DCE` |
| 郑商所 | `.CZC` | `SR305.CZC` |

### 证券代码转换规则

**大部分情况下证券代码格式不需要转换**，Wind 和 THS 使用相同的后缀体系。

需要注意的特殊情况：
1. 基金代码格式基本一致：`510050.SH`
2. 指数代码基本一致：`000300.SH`（沪深300）
3. 多证券传入时两者都用逗号分隔：`"600519.SH,000001.SZ"`

## 多证券分隔符

两者一致，均使用逗号分隔：

```python
# Wind
w.wsd("600519.SH,000001.SZ", "close", "20230101", "20230131")

# THS（证券代码分隔方式不变，但注意字段分隔符的差异）
THS_HistoryQuotes("600519.SH,000001.SZ", "close", "CPS:6", "2023-01-01", "2023-01-31")
```
