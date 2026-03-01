# 认证方式差异

## Wind 认证

Wind 终端在本地运行时自动完成认证，代码中无需传入凭据：

```python
from WindPy import w
w.start()  # 自动连接本地 Wind 终端，无需用户名密码
# ... 使用 API ...
w.stop()
```

## THS iFinD 认证

THS iFinD 需要在代码中显式传入账号和密码：

```python
from iFinDPy import *

account = 'your_account'
password = 'your_password'
result = THS_iFinDLogin(account, password)

if result == 0:
    print('登录成功')
else:
    print(f'登录失败，错误码: {result}')

# ... 使用 API ...

THS_iFinDLogout()
```

### 登录返回值

| 返回值 | 含义 |
|--------|------|
| `0` | 登录成功 |
| `-1` | 登录失败 |
| `-201` | 账号或密码错误 |
| `-202` | 无权限 |

## 安全最佳实践

**不要在代码中硬编码账号密码**，推荐使用环境变量：

```python
import os
from iFinDPy import *

account = os.environ.get('THS_ACCOUNT')
password = os.environ.get('THS_PASSWORD')

if not account or not password:
    raise ValueError('请设置环境变量 THS_ACCOUNT 和 THS_PASSWORD')

result = THS_iFinDLogin(account, password)
if result != 0:
    raise ConnectionError(f'THS 登录失败，错误码: {result}')
```

设置环境变量（在 `.bashrc` / `.zshrc` 中）：

```bash
export THS_ACCOUNT="your_account"
export THS_PASSWORD="your_password"
```

## 转换规则

1. 将 `from WindPy import w` 替换为 `from iFinDPy import *`
2. 将 `w.start()` 替换为 `THS_iFinDLogin(account, password)`，并在文件顶部添加环境变量读取逻辑
3. 将 `w.stop()` 替换为 `THS_iFinDLogout()`
4. 如果原代码中有 `w.isconnected()` 检查，替换为检查 `THS_iFinDLogin` 返回值是否为 `0`
