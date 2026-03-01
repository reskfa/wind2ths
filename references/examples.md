# 完整前后对比示例

## 示例 1：获取日线收盘价

### Wind 版本

```python
from WindPy import w
import pandas as pd

w.start()

data = w.wsd("600519.SH", "close", "20230101", "20230131", "PriceAdj=F")
if data.ErrorCode != 0:
    print(f"Error: {data.ErrorCode}")
else:
    df = pd.DataFrame({
        'date': data.Times,
        'close': data.Data[0]
    })
    print(df)

w.stop()
```

### THS 版本

```python
import os
import pandas as pd
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

data = THS_HistoryQuotes("600519.SH", "close", "CPS:1", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(data)
if df is not None and not df.empty:
    print(df)
else:
    print("查询失败或无数据")

THS_iFinDLogout()
```

---

## 示例 2：获取截面数据（PE/PB）

### Wind 版本

```python
from WindPy import w

w.start()

codes = "600519.SH,000001.SZ,601318.SH"
data = w.wss(codes, "sec_name,pe_ttm,pb_mrq,dividendyield2", "tradeDate=20230131")

if data.ErrorCode == 0:
    for i, code in enumerate(data.Codes):
        print(f"{data.Data[0][i]}: PE={data.Data[1][i]:.2f}, PB={data.Data[2][i]:.2f}, 股息率={data.Data[3][i]:.2f}%")

w.stop()
```

### THS 版本

```python
import os
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

codes = "600519.SH,000001.SZ,601318.SH"
data = THS_BD(codes, "ths_stock_short_name;ths_pe_ttm;ths_pb_mrq;ths_dividend_yield", "2023-01-31")
df = THS_Trans2DataFrame(data)

if df is not None and not df.empty:
    for _, row in df.iterrows():
        print(f"{row['ths_stock_short_name']}: PE={row['ths_pe_ttm']:.2f}, PB={row['ths_pb_mrq']:.2f}, 股息率={row['ths_dividend_yield']:.2f}%")

THS_iFinDLogout()
```

---

## 示例 3：获取分钟数据

### Wind 版本

```python
from WindPy import w
import pandas as pd

w.start()

data = w.wsi("600519.SH", "close,open,high,low,volume",
             "2023-01-31 09:30:00", "2023-01-31 15:00:00", "BarSize=5")

df = pd.DataFrame({
    'time': data.Times,
    'close': data.Data[0],
    'open': data.Data[1],
    'high': data.Data[2],
    'low': data.Data[3],
    'volume': data.Data[4]
})
print(df)

w.stop()
```

### THS 版本

```python
import os
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

data = THS_HF("600519.SH", "close;open;high;low;volume",
              "CPS:6,Fill:Previous,Interval:5",
              "2023-01-31 09:30:00", "2023-01-31 15:00:00")

df = THS_Trans2DataFrame(data)
if df is not None and not df.empty:
    print(df)

THS_iFinDLogout()
```

---

## 示例 4：查询宏观经济数据

### Wind 版本

```python
from WindPy import w
import pandas as pd

w.start()

# 获取 CPI 当月同比
data = w.edb("M0000612", "20220101", "20231231")

df = pd.DataFrame({
    'date': data.Times,
    'CPI_YoY': data.Data[0]
})
print(df)

w.stop()
```

### THS 版本

```python
import os
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

# 注意：THS EDB 指标编码与 Wind 不同，此处编码需人工查找确认
# Wind M0000612 (CPI当月同比) → THS 对应编码需查询 THS 指标库
data = THS_EDBQuery("M001620067", "2022-01-01", "2023-12-31")  # 编码仅为示例，需确认

df = THS_Trans2DataFrame(data)
if df is not None and not df.empty:
    print(df)

THS_iFinDLogout()
```

> **人工检查点**：THS EDB 指标编码需要人工查找确认，Wind 编码无法自动映射。

---

## 示例 5：获取交易日历

### Wind 版本

```python
from WindPy import w

w.start()

# 获取2023年所有交易日
data = w.tdays("20230101", "20231231")
trade_days = data.Data[0]
print(f"2023年共有 {len(trade_days)} 个交易日")

# 获取某日期前第5个交易日
data2 = w.tdaysoffset(-5, "20230131")
print(f"2023-01-31 前第5个交易日: {data2.Data[0][0]}")

w.stop()
```

### THS 版本

```python
import os
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

# 获取2023年所有交易日
data = THS_DateQuery("SSE", "dateType:0,period:D", "2023-01-01", "2023-12-31")
df = THS_Trans2DataFrame(data)
if df is not None and not df.empty:
    print(f"2023年共有 {len(df)} 个交易日")

# 获取某日期前第5个交易日
data2 = THS_DateOffset("SSE", "2023-01-31", -5, "Period:D")
df2 = THS_Trans2DataFrame(data2)
if df2 is not None and not df2.empty:
    print(f"2023-01-31 前第5个交易日: {df2['time'].iloc[0]}")

THS_iFinDLogout()
```

---

## 示例 6：获取指数成分股

### Wind 版本

```python
from WindPy import w
import pandas as pd

w.start()

# 获取沪深300成分股
data = w.wset("sectorconstituent", "date=20230131;windcode=000300.SH")

df = pd.DataFrame({
    'code': data.Data[1],
    'name': data.Data[2]
})
print(f"沪深300成分股数量: {len(df)}")
print(df.head(10))

w.stop()
```

### THS 版本

```python
import os
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

# 获取沪深300成分股
# 注意：THS 板块代码与 Wind sectorid 不同，需查找对应代码
data = THS_DataPool("block", "2023-01-31;001005030", "date:Y,security_name:Y,thscode:Y")

df = THS_Trans2DataFrame(data)
if df is not None and not df.empty:
    print(f"沪深300成分股数量: {len(df)}")
    print(df.head(10))

THS_iFinDLogout()
```

> **人工检查点**：THS 板块代码（如 `001005030`）需根据实际的 THS 板块编码体系确认。

---

## 示例 7：综合多函数脚本

### Wind 版本

```python
from WindPy import w
import pandas as pd

w.start()

# 1. 获取交易日
trade_days = w.tdays("20230101", "20230131").Data[0]
last_day = trade_days[-1].strftime("%Y%m%d")

# 2. 获取沪深300成分股
constituents = w.wset("sectorconstituent", f"date={last_day};windcode=000300.SH")
codes = constituents.Data[1][:10]  # 取前10只
codes_str = ",".join(codes)

# 3. 获取截面数据
snapshot = w.wss(codes_str, "sec_name,close,pe_ttm,pb_mrq,mkt_cap_ard", f"tradeDate={last_day}")

df = pd.DataFrame({
    'code': snapshot.Codes,
    'name': snapshot.Data[0],
    'close': snapshot.Data[1],
    'pe': snapshot.Data[2],
    'pb': snapshot.Data[3],
    'mkt_cap': snapshot.Data[4]
})

# 4. 获取前10只股票的日线数据
for code in codes[:3]:
    hist = w.wsd(code, "close,volume", "20230101", last_day)
    print(f"{code}: {len(hist.Data[0])} 条日线数据")

print(df)

w.stop()
```

### THS 版本

```python
import os
import pandas as pd
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))

# 1. 获取交易日
trade_days_data = THS_DateQuery("SSE", "dateType:0,period:D", "2023-01-01", "2023-01-31")
trade_days_df = THS_Trans2DataFrame(trade_days_data)
last_day = trade_days_df['time'].iloc[-1]  # "2023-01-31" 格式

# 2. 获取沪深300成分股
# 注意：板块代码需确认
constituents_data = THS_DataPool("block", f"{last_day};001005030", "date:Y,security_name:Y,thscode:Y")
constituents_df = THS_Trans2DataFrame(constituents_data)
codes = constituents_df['thscode'].tolist()[:10]  # 取前10只
codes_str = ",".join(codes)

# 3. 获取截面数据
snapshot_data = THS_BD(codes_str, "ths_stock_short_name;ths_pe_ttm;ths_pb_mrq;ths_market_value", last_day)
df = THS_Trans2DataFrame(snapshot_data)

# 同时获取收盘价（THS_BD 不直接返回行情价格，需用 THS_HistoryQuotes）
close_data = THS_HistoryQuotes(codes_str, "close", "CPS:6", last_day, last_day)
close_df = THS_Trans2DataFrame(close_data)

# 4. 获取前3只股票的日线数据
for code in codes[:3]:
    hist_data = THS_HistoryQuotes(code, "close;volume", "CPS:6", "2023-01-01", last_day)
    hist_df = THS_Trans2DataFrame(hist_data)
    if hist_df is not None and not hist_df.empty:
        print(f"{code}: {len(hist_df)} 条日线数据")

print(df)

THS_iFinDLogout()
```

> **人工检查点**：
> 1. THS 板块代码 `001005030` 需确认
> 2. `ths_market_value` 字段名需确认
> 3. THS_BD 返回的 DataFrame 列名需根据实际返回调整
