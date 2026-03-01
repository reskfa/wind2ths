# 字段名映射（Wind → THS iFinD）

## 重要说明

- Wind 字段用 **逗号** 分隔：`"close,open,high,low"`
- THS 字段用 **分号** 分隔：`"close;open;high;low"`
- 部分 THS 字段名与 Wind 不同，需要查表转换
- THS 许多指标需要加前缀 `ths_`

## 行情类字段

### 日线行情（w.wsd → THS_HistoryQuotes）

| Wind 字段 | THS 字段 | 说明 |
|-----------|---------|------|
| `close` | `close` | 收盘价 |
| `open` | `open` | 开盘价 |
| `high` | `high` | 最高价 |
| `low` | `low` | 最低价 |
| `volume` | `volume` | 成交量 |
| `amt` | `amount` | 成交额 |
| `pct_chg` | `changeRatio` | 涨跌幅(%) |
| `chg` | `change` | 涨跌额 |
| `turn` | `turnoverRatio` | 换手率(%) |
| `vwap` | `avgPrice` | 均价 |
| `pre_close` | `preClose` | 前收盘价 |
| `adjfactor` | `CPS` | 复权因子 |
| `mkt_cap_ard` | `ths_market_value` | 总市值 |
| `mkt_cap_float` | `ths_float_market_value` | 流通市值 |

### THS_HistoryQuotes 的 params 参数

THS_HistoryQuotes 第三个参数是控制参数字符串，常用值：

| params | 含义 |
|--------|------|
| `"CPS:1"` | 不复权 |
| `"CPS:2"` | 前复权 |
| `"CPS:3"` | 后复权 |
| `"CPS:6"` | 等比前复权（推荐，对应 Wind 默认复权方式） |
| `"Period:D"` | 日频（默认） |
| `"Period:W"` | 周频 |
| `"Period:M"` | 月频 |

多个参数用逗号连接：`"CPS:6,Period:D"`

### Wind wsd 的 options 参数对应

| Wind options | THS params | 说明 |
|-------------|-----------|------|
| `"PriceAdj=F"` | `"CPS:1"` | 不复权 |
| `"PriceAdj=B"` | `"CPS:3"` | 后复权 |
| `"PriceAdj=A"` 或不传 | `"CPS:6"` | 前复权 |
| `"Period=W"` | `"Period:W"` | 周频 |
| `"Period=M"` | `"Period:M"` | 月频 |

## 截面数据字段（w.wss → THS_BD）

### 估值指标

| Wind 字段 | THS 字段 | 说明 |
|-----------|---------|------|
| `pe_ttm` | `ths_pe_ttm` | 市盈率(TTM) |
| `pe_lyr` | `ths_pe_lyr` | 市盈率(静态) |
| `pb_lf` / `pb_mrq` | `ths_pb_mrq` | 市净率(MRQ) |
| `ps_ttm` | `ths_ps_ttm` | 市销率(TTM) |
| `pcf_ocf_ttm` | `ths_pcf_ttm` | 市现率(TTM) |
| `ev` | `ths_ev` | 企业价值 |
| `dividendyield2` | `ths_dividend_yield` | 股息率 |

### 财务指标

| Wind 字段 | THS 字段 | 说明 |
|-----------|---------|------|
| `roe_ttm2` | `ths_roe_ttm` | ROE(TTM) |
| `roa_ttm2` | `ths_roa_ttm` | ROA(TTM) |
| `grossprofitmargin_ttm2` | `ths_gross_profit_margin_ttm` | 毛利率(TTM) |
| `netprofitmargin_ttm2` | `ths_net_profit_margin_ttm` | 净利率(TTM) |
| `debttoassets` | `ths_asset_liability_ratio` | 资产负债率 |
| `tot_shrhldr_eqy_incl_min_int` | `ths_total_equity` | 股东权益 |
| `or_ttm2` / `revenue_ttm` | `ths_revenue_ttm` | 营业收入(TTM) |
| `net_profit_ttm2` | `ths_net_profit_ttm` | 净利润(TTM) |

### 公司信息

| Wind 字段 | THS 字段 | 说明 |
|-----------|---------|------|
| `sec_name` | `ths_stock_short_name` | 证券简称 |
| `industry_sw` | `ths_the_sw_industry` | 申万行业 |
| `industry_citic` | `ths_the_citic_industry` | 中信行业 |
| `ipo_date` | `ths_listing_date` | 上市日期 |
| `delist_date` | `ths_delisting_date` | 退市日期 |
| `totalshares` | `ths_total_shares` | 总股本 |
| `floatshares` | `ths_float_shares` | 流通股本 |

### THS_BD 的日期参数

Wind `w.wss` 的 `tradeDate` 参数在 THS_BD 中通过第三个参数传入：

```python
# Wind
w.wss("600519.SH", "pe_ttm,pb_mrq", "tradeDate=20230131")

# THS
THS_BD("600519.SH", "ths_pe_ttm;ths_pb_mrq", "2023-01-31")
```

## 实时行情字段（w.wsq → THS_RQ / THS_Snapshot）

| Wind 字段 | THS 字段 | 说明 |
|-----------|---------|------|
| `rt_last` | `latest` | 最新价 |
| `rt_open` | `open` | 今开 |
| `rt_high` | `high` | 最高 |
| `rt_low` | `low` | 最低 |
| `rt_vol` | `volume` | 成交量 |
| `rt_amt` | `amount` | 成交额 |
| `rt_pre_close` | `preClose` | 昨收 |
| `rt_pct_chg` | `changeRatio` | 涨跌幅 |
| `rt_bid1` ~ `rt_bid5` | `bid1` ~ `bid5` | 买价 |
| `rt_ask1` ~ `rt_ask5` | `ask1` ~ `ask5` | 卖价 |
| `rt_bsize1` ~ `rt_bsize5` | `bidVol1` ~ `bidVol5` | 买量 |
| `rt_asize1` ~ `rt_asize5` | `askVol1` ~ `askVol5` | 卖量 |

## 宏观经济指标（w.edb → THS_EDBQuery）

Wind EDB 和 THS EDB 使用各自的指标编码体系，**编码不通用**，需要根据指标含义查找对应的 THS 指标 ID。

常用宏观指标对照：

| 指标 | Wind 编码 | THS 编码 |
|------|----------|---------|
| GDP 当季值 | `M0001228` | 需查询 THS 指标库 |
| CPI 当月同比 | `M0000612` | 需查询 THS 指标库 |
| PPI 当月同比 | `M0001227` | 需查询 THS 指标库 |
| M2 同比 | `M0001385` | 需查询 THS 指标库 |

> **注意**：宏观经济指标的编码映射需要人工查找确认，无法自动转换。转换时应标注为"需人工检查"。

## 字段分隔符转换

迁移时务必将 Wind 的逗号分隔转换为 THS 的分号分隔：

```python
# Wind
fields_wind = "close,open,high,low,volume"

# THS — 替换分隔符
fields_ths = fields_wind.replace(",", ";")
# 注意：如果字段名也需要映射，应先映射字段名再替换分隔符
```
