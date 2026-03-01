# Wind2THS

**一条命令，告别天价 Wind 终端。**

Wind2THS 是一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 自定义 Slash Command，能够自动将你的 Wind（万得）Python 量化代码迁移为同花顺 iFinD API 代码。

> Wind 终端一年好几万，iFinD 只要几千块，API 功能几乎等价。
> 唯一的迁移成本？现在也没有了。

## 它能做什么

```
/wind2ths your_strategy.py
```

就这么简单。Claude 会：

1. **扫描**你的 Python 文件，识别所有 Wind API 调用
2. **转换**函数名、字段名、日期格式、返回值处理
3. **输出**可直接运行的 iFinD 代码
4. **标注**无法自动转换的地方（如 EDB 宏观指标编码），让你精准排查

### 支持的 API 覆盖

| Wind API | THS iFinD API | 场景 |
|----------|--------------|------|
| `w.wsd()` | `THS_HistoryQuotes()` | 日线行情 |
| `w.wss()` | `THS_BD()` | 截面数据（PE/PB 等） |
| `w.wsq()` | `THS_RQ()` / `THS_Snapshot()` | 实时行情 |
| `w.wsi()` | `THS_HF()` | 分钟 K 线 |
| `w.wset()` | `THS_DataPool()` | 成分股/板块 |
| `w.edb()` | `THS_EDBQuery()` | 宏观经济数据 |
| `w.tdays()` | `THS_DateQuery()` | 交易日历 |
| `w.tdaysoffset()` | `THS_DateOffset()` | 交易日偏移 |

完整映射包含 **10+ 函数、50+ 字段、4 种返回值处理模式**。

### 自动处理的差异

你不需要记住这些，Wind2THS 全部自动搞定：

- 字段分隔符：逗号 → 分号
- 日期格式：`"20230101"` → `"2023-01-01"`
- 认证方式：无感连接 → 显式登录（自动添加环境变量读取）
- 返回值：`WindData.Data[0]` → `THS_Trans2DataFrame()` + DataFrame 操作
- 复权参数：`PriceAdj=F` → `CPS:1`

## 快速开始

### 1. 安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. 克隆本项目

```bash
git clone https://github.com/yourname/wind2ths.git
cd wind2ths
```

### 3. 运行迁移

```bash
claude
# 进入 Claude Code 后：
/wind2ths path/to/your_wind_script.py
```

### 4. 配置 THS 凭据

转换后的代码默认从环境变量读取凭据：

```bash
export THS_ACCOUNT="your_account"
export THS_PASSWORD="your_password"
```

## 转换效果预览

**迁移前** — Wind API：

```python
from WindPy import w

w.start()
data = w.wsd("600519.SH", "close,open,high", "20230101", "20230131", "PriceAdj=F")
df = pd.DataFrame(data.Data, index=data.Fields, columns=data.Times).T
w.stop()
```

**迁移后** — THS iFinD API：

```python
import os
from iFinDPy import *

THS_iFinDLogin(os.environ.get('THS_ACCOUNT'), os.environ.get('THS_PASSWORD'))
data = THS_HistoryQuotes("600519.SH", "close;open;high", "CPS:1", "2023-01-01", "2023-01-31")
df = THS_Trans2DataFrame(data)
THS_iFinDLogout()
```

更多示例见 [`references/examples.md`](references/examples.md)。

## 项目结构

```
wind2ths/
├── CLAUDE.md                          # Claude Code 项目说明
├── commands/
│   └── wind2ths.md                    # /wind2ths 命令定义
└── references/
    ├── api-mapping.md                 # 函数级映射（签名、参数、代码片段）
    ├── field-mapping.md               # 50+ 字段名映射表
    ├── return-format.md               # 返回值处理的 4 种转换模式
    ├── date-and-code-format.md        # 日期 & 证券代码格式规则
    ├── authentication.md              # 认证差异 & 安全最佳实践
    └── examples.md                    # 7 个完整的前后对比示例
```

## 局限性

以下场景需要人工介入，Wind2THS 会自动标注 `TODO`：

- **EDB 宏观指标编码** — Wind 和 THS 的编码体系完全不同，无法自动映射
- **板块/行业编码** — Wind sectorid 与 THS 板块代码不通用
- **Tick 级数据** — THS 没有与 `w.wst()` 完全等价的函数
- **Wind 私有字段** — 少数 Wind 独有指标在 THS 中没有直接对应

## 贡献

欢迎提交 PR 补充：
- 更多字段映射（`references/field-mapping.md`）
- 更多前后对比示例（`references/examples.md`）
- EDB 宏观指标编码对照表

## License

MIT
