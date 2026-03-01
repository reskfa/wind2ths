# 返回值格式差异

## Wind 返回值 — WindData 对象

Wind API 返回 `WindData` 对象，具有以下属性：

```python
result = w.wsd("600519.SH", "close,open", "20230101", "20230131")

result.ErrorCode   # 错误码，0 表示成功
result.Data        # 二维列表 [[close values], [open values]]
result.Times       # 日期列表 [datetime(2023,1,3), datetime(2023,1,4), ...]
result.Fields      # 字段列表 ['CLOSE', 'OPEN']
result.Codes       # 证券代码列表 ['600519.SH']
```

### 常见的 WindData 处理模式

```python
# 模式1：直接访问 Data
close_prices = result.Data[0]  # 第一个字段的所有值

# 模式2：转为 DataFrame
import pandas as pd
df = pd.DataFrame(result.Data, index=result.Fields, columns=result.Times).T

# 模式3：错误检查
if result.ErrorCode != 0:
    print(f"Error: {result.ErrorCode}")
```

## THS iFinD 返回值

THS iFinD API 返回值因函数不同而异，但都可以通过 `THS_Trans2DataFrame()` 转换为 pandas DataFrame。

### THS_HistoryQuotes 返回值

```python
result = THS_HistoryQuotes("600519.SH", "close;open", "CPS:6", "2023-01-01", "2023-01-31")

# 直接转为 DataFrame
df = THS_Trans2DataFrame(result)
# DataFrame 列: ['time', 'thscode', 'close', 'open']
```

### THS_BD 返回值

```python
result = THS_BD("600519.SH", "ths_pe_ttm;ths_pb_mrq", "2023-01-31")

df = THS_Trans2DataFrame(result)
# DataFrame 列: ['thscode', 'ths_pe_ttm', 'ths_pb_mrq']
```

### THS_RQ / THS_Snapshot 返回值

```python
result = THS_Snapshot("600519.SH", "latest;open;high;low", "", "2023-01-31")

df = THS_Trans2DataFrame(result)
```

### THS_DataPool 返回值

```python
result = THS_DataPool("block", "2023-01-31;001005010", "date:Y, security_name:Y, thscode:Y")

df = THS_Trans2DataFrame(result)
```

## 转换模式

### 模式 A：WindData.Data 直接访问 → DataFrame 列访问

```python
# === Wind ===
result = w.wsd("600519.SH", "close", "20230101", "20230131")
prices = result.Data[0]
dates = result.Times

# === THS ===
result = THS_HistoryQuotes("600519.SH", "close", "CPS:6", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(result)
prices = df['close'].tolist()
dates = pd.to_datetime(df['time']).tolist()
```

### 模式 B：WindData 转 DataFrame → THS 直接转 DataFrame

```python
# === Wind ===
result = w.wsd("600519.SH", "close,open", "20230101", "20230131")
df = pd.DataFrame(result.Data, index=result.Fields, columns=result.Times).T

# === THS ===
result = THS_HistoryQuotes("600519.SH", "close;open", "CPS:6", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(result)
df = df.set_index('time')  # 如果需要以日期为索引
df.index = pd.to_datetime(df.index)
```

### 模式 C：错误检查

```python
# === Wind ===
if result.ErrorCode != 0:
    print(f"Wind Error: {result.ErrorCode}")

# === THS ===
# THS 返回值中包含错误信息，可通过检查返回对象判断
# 或者检查 DataFrame 是否为空
df = THS_Trans2DataFrame(result)
if df is None or df.empty:
    print("THS 查询失败或无数据")
```

### 模式 D：多证券数据拆分

```python
# === Wind ===
result = w.wsd("600519.SH,000001.SZ", "close", "20230101", "20230131")
# result.Data[0] 包含第一只证券的数据（按 Codes 区分）

# === THS ===
result = THS_HistoryQuotes("600519.SH,000001.SZ", "close", "CPS:6", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(result)
# DataFrame 有 'thscode' 列可用于区分证券
df_519 = df[df['thscode'] == '600519.SH']
df_001 = df[df['thscode'] == '000001.SZ']
```

## THS_Trans2DataFrame 使用要点

1. **始终使用** `THS_Trans2DataFrame()` 将结果转为 DataFrame，这是最可靠的处理方式
2. 返回的 DataFrame 列名通常是小写的指标英文名
3. 包含 `thscode` 列用于标识证券代码
4. 时间序列数据包含 `time` 列
5. 如果查询失败，`THS_Trans2DataFrame()` 可能返回 `None` 或空 DataFrame
