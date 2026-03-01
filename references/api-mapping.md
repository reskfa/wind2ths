# 核心 API 函数映射

## 1. 导入与认证

### 导入

```python
# Wind
from WindPy import w

# THS
from iFinDPy import *
```

### 登录

```python
# Wind
w.start()

# THS
import os
THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))
```

### 登出

```python
# Wind
w.stop()

# THS
THS_iFinDLogout()
```

---

## 2. w.wsd() → THS_HistoryQuotes() / THS_DS()

日频时间序列数据。

### 函数签名

```python
# Wind
w.wsd(codes, fields, beginTime, endTime, options)

# THS
THS_HistoryQuotes(codes, fields, params, beginDate, endDate)
```

### 参数映射

| Wind 参数 | THS 参数 | 转换说明 |
|-----------|---------|---------|
| `codes` — `str`，逗号分隔 | `codes` — `str`，逗号分隔 | 格式一致，无需转换 |
| `fields` — `str`，逗号分隔 | `fields` — `str`，**分号分隔** | 逗号→分号，字段名可能需映射 |
| `beginTime` — `str` 或 `datetime` | `beginDate` — `str` `"YYYY-MM-DD"` | 日期格式统一为 `YYYY-MM-DD` |
| `endTime` — `str` 或 `datetime` | `endDate` — `str` `"YYYY-MM-DD"` | 同上 |
| `options` — `str`，如 `"PriceAdj=F"` | `params` — `str`，如 `"CPS:1"` | 见 field-mapping.md |

### 转换示例

```python
# Wind
data = w.wsd("600519.SH", "close,open,high,low", "20230101", "20230131", "PriceAdj=F")

# THS
data = THS_HistoryQuotes("600519.SH", "close;open;high;low", "CPS:1", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(data)
```

### 使用 THS_DS（数据超市）的场景

当需要获取非标准行情字段（如财务指标的时间序列）时，使用 `THS_DS`：

```python
# Wind — 用 wsd 获取 PE 时间序列
data = w.wsd("600519.SH", "pe_ttm", "20230101", "20230131")

# THS — 用 THS_DS
data = THS_DS("600519.SH", "ths_pe_ttm", "", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(data)
```

---

## 3. w.wss() → THS_BD()

截面数据（某一时点的多指标数据）。

### 函数签名

```python
# Wind
w.wss(codes, fields, options)
# options 中含 "tradeDate=20230131" 等参数

# THS
THS_BD(codes, fields, params)
# params 中含日期信息
```

### 参数映射

| Wind 参数 | THS 参数 | 转换说明 |
|-----------|---------|---------|
| `codes` — 逗号分隔 | `codes` — 逗号分隔 | 格式一致 |
| `fields` — 逗号分隔 | `fields` — **分号分隔** | 字段名通常需加 `ths_` 前缀 |
| `options` — `"tradeDate=20230131;..."` | `params` — `"2023-01-31"` | 日期提取并格式化 |

### 转换示例

```python
# Wind
data = w.wss("600519.SH,000001.SZ", "pe_ttm,pb_mrq,sec_name", "tradeDate=20230131")
names = data.Data[2]

# THS
data = THS_BD("600519.SH,000001.SZ", "ths_pe_ttm;ths_pb_mrq;ths_stock_short_name", "2023-01-31")
df = THS_Trans2DataFrame(data)
names = df['ths_stock_short_name'].tolist()
```

---

## 4. w.wsq() → THS_RQ() / THS_Snapshot()

实时行情数据。

### 函数签名

```python
# Wind — 实时订阅
w.wsq(codes, fields, func=callback)

# Wind — 快照
w.wsq(codes, fields)

# THS — 实时订阅
THS_RQ(codes, fields, callback)

# THS — 快照
THS_Snapshot(codes, fields, params, endDate)
```

### 转换示例

```python
# Wind — 快照
data = w.wsq("600519.SH", "rt_last,rt_open,rt_high,rt_low")

# THS — 快照
data = THS_Snapshot("600519.SH", "latest;open;high;low", "", "")
df = THS_Trans2DataFrame(data)
```

```python
# Wind — 订阅
def wind_callback(indata):
    print(indata.Data)

w.wsq("600519.SH", "rt_last,rt_vol", func=wind_callback)

# THS — 订阅
def ths_callback(data):
    df = THS_Trans2DataFrame(data)
    print(df)

THS_RQ("600519.SH", "latest;volume", ths_callback)
```

---

## 5. w.wsi() → THS_HF()

分钟级高频数据。

### 函数签名

```python
# Wind
w.wsi(codes, fields, beginTime, endTime, options)
# options: "BarSize=5" 表示5分钟

# THS
THS_HF(codes, fields, params, beginDate, endDate)
# params: "CPS:6,Fill:Previous,Interval:5" 表示5分钟
```

### 参数映射

| Wind options | THS params | 说明 |
|-------------|-----------|------|
| `"BarSize=1"` | `"Interval:1"` | 1分钟 |
| `"BarSize=5"` | `"Interval:5"` | 5分钟 |
| `"BarSize=15"` | `"Interval:15"` | 15分钟 |
| `"BarSize=30"` | `"Interval:30"` | 30分钟 |
| `"BarSize=60"` | `"Interval:60"` | 60分钟 |

### 转换示例

```python
# Wind
data = w.wsi("600519.SH", "close,open,high,low,volume", "2023-01-31 09:30:00", "2023-01-31 15:00:00", "BarSize=5")

# THS
data = THS_HF("600519.SH", "close;open;high;low;volume", "CPS:6,Fill:Previous,Interval:5", "2023-01-31 09:30:00", "2023-01-31 15:00:00")
df = THS_Trans2DataFrame(data)
```

---

## 6. w.wst() → THS_Snapshot()

Tick 级逐笔数据。

```python
# Wind
data = w.wst("600519.SH", "last,volume,amt", "2023-01-31 09:30:00", "2023-01-31 15:00:00")

# THS — 使用 THS_Snapshot 或 THS_HF 以最小间隔获取
# 注意：THS 没有与 w.wst 完全等价的 tick 函数，需根据实际需求选择替代方案
data = THS_Snapshot("600519.SH", "latest;volume;amount", "", "2023-01-31")
df = THS_Trans2DataFrame(data)
```

> **注意**：THS 的 tick 级数据获取方式与 Wind 不完全对等，迁移时需标注为"需人工确认"。

---

## 7. w.wset() → THS_DataPool()

数据集查询（成分股、行业分类等）。

### 函数签名

```python
# Wind
w.wset("sectorconstituent", "date=20230131;sectorid=a]001010100000000")

# THS
THS_DataPool("block", "2023-01-31;001005010", "date:Y,security_name:Y,thscode:Y")
```

### 常用数据集映射

| 用途 | Wind wset | THS DataPool |
|------|----------|-------------|
| 指数成分股 | `"sectorconstituent"` + `sectorid` | `"block"` + 板块代码 |
| 行业成分股 | `"sectorconstituent"` + `sectorid` | `"block"` + 行业代码 |
| 全部A股 | `"sectorconstituent"` + `"a001010100000000"` | `"block"` + `"001005010"` |

### 转换示例

```python
# Wind — 获取沪深300成分股
data = w.wset("sectorconstituent", "date=20230131;windcode=000300.SH")

# THS — 获取沪深300成分股
data = THS_DataPool("block", "2023-01-31;001005030", "date:Y,security_name:Y,thscode:Y")
df = THS_Trans2DataFrame(data)
```

> **注意**：Wind 和 THS 的板块/行业编码体系不同，需查找对应的 THS 板块代码。

---

## 8. w.edb() → THS_EDBQuery()

宏观经济数据库。

### 函数签名

```python
# Wind
w.edb(codes, beginTime, endTime, options)

# THS
THS_EDBQuery(codes, beginDate, endDate)
```

### 转换示例

```python
# Wind — CPI 当月同比
data = w.edb("M0000612", "20200101", "20231231")

# THS — 需使用 THS 对应的 EDB 指标编码
data = THS_EDBQuery("M001620067", "2020-01-01", "2023-12-31")
df = THS_Trans2DataFrame(data)
```

> **注意**：Wind EDB 编码与 THS EDB 编码完全不同，需逐一查找对应关系，应标注为"需人工检查"。

---

## 9. w.tdays() → THS_DateQuery()

交易日历查询。

### 函数签名

```python
# Wind
w.tdays(beginTime, endTime, options)
# 返回 WindData，Data[0] 为日期列表

# THS
THS_DateQuery("SSE", "dateType:0,period:D", beginDate, endDate)
# 第一参数: 交易所代码 (SSE=上交所, SZSE=深交所)
# dateType:0 交易日, dateType:1 自然日
```

### 转换示例

```python
# Wind
data = w.tdays("20230101", "20231231")
trade_days = data.Data[0]

# THS
data = THS_DateQuery("SSE", "dateType:0,period:D", "2023-01-01", "2023-12-31")
df = THS_Trans2DataFrame(data)
trade_days = df['time'].tolist()
```

---

## 10. w.tdaysoffset() → THS_DateOffset()

交易日偏移计算。

### 函数签名

```python
# Wind
w.tdaysoffset(offset, baseDate, options)
# offset: 正数向后，负数向前

# THS
THS_DateOffset("SSE", baseDate, offset, "Period:D")
```

### 转换示例

```python
# Wind — 从 20230131 往前推 5 个交易日
data = w.tdaysoffset(-5, "20230131")
target_date = data.Data[0][0]

# THS
data = THS_DateOffset("SSE", "2023-01-31", -5, "Period:D")
df = THS_Trans2DataFrame(data)
target_date = df['time'].iloc[0]
```
