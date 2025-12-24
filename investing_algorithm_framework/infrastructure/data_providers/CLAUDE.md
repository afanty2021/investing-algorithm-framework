# Data Providers 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [infrastructure](../) > **data_providers**

## 模块职责

数据提供者模块，实现 `DataProvider` 接口，为框架提供多种数据源：

- **CCXT 数据**: 通过 CCXT 库从 100+ 交易所获取实时和历史数据
- **CSV 数据**: 从本地 CSV 文件加载 OHLCV 数据
- **Pandas 数据**: 从 Pandas DataFrame 提供数据
- **数据缓存**: 自动缓存和管理数据
- **回测支持**: 优化的回测数据准备和滑动窗口

## 入口与启动

### 模块导出

```python
from investing_algorithm_framework.infrastructure.data_providers import (
    CCXTOHLCVDataProvider,
    CSVOHLCVDataProvider,
    PandasOHLCVDataProvider,
    get_default_data_providers,
    get_default_ohlcv_data_providers
)
```

### 数据提供者基类

所有数据提供者实现 `DataProvider` 接口：

```python
class DataProvider:
    def has_data(
        self,
        data_source: DataSource,
        start_date: datetime = None,
        end_date: datetime = None
    ) -> bool

    def get_data(
        self,
        date: datetime = None,
        start_date: datetime = None,
        end_date: datetime = None,
        save: bool = False
    ) -> Union[pl.DataFrame, pd.DataFrame]

    def get_ticker(self, symbol: str, market: str) -> dict
    def prepare_backtest_data(...)
```

## CCXTOHLCVDataProvider

通过 CCXT 库从交易所获取 OHLCV 数据。

### 初始化

```python
CCXTOHLCVDataProvider(
    market: str = None,              # 交易所名称（默认: binance）
    symbol: str = None,              # 交易对（如 BTC/USDT）
    time_frame: str = None,          # 时间框架（如 1h, 1d）
    window_size: int = None,         # 滑动窗口大小
    data_provider_identifier: str = None,  # 自定义标识符
    storage_directory: str = None,   # 数据存储目录
    pandas: bool = False,            # 返回 Pandas DataFrame
    config: dict = None              # CCXT 配置
)
```

### 支持的交易所

通过 CCXT 支持 100+ 交易所，包括：

| 交易所 | market 值 |
|--------|----------|
| Binance | `binance` |
| OKX | `okx` |
| Coinbase | `coinbase` |
| Kraken | `kraken` |
| Bybit | `bybit` |
| Huobi | `huobi` |

### 支持的时间框架

| 值 | 描述 |
|----|------|
| `1m` | 1 分钟 |
| `5m` | 5 分钟 |
| `15m` | 15 分钟 |
| `1h` | 1 小时 |
| `4h` | 4 小时 |
| `1d` | 1 天 |
| `1w` | 1 周 |

### 数据获取

```python
from investing_algorithm_framework import CCXTOHLCVDataProvider
from datetime import datetime, timezone

# 创建提供者
provider = CCXTOHLCVDataProvider(
    market="binance",
    symbol="BTC/USDT",
    time_frame="1h"
)

# 获取实时数据（最近 window_size 条）
data = provider.get_data()

# 获取指定时间范围数据
data = provider.get_data(
    start_date=datetime(2023, 1, 1, tzinfo=timezone.utc),
    end_date=datetime(2023, 12, 31, tzinfo=timezone.utc),
    save=True  # 保存到本地缓存
)
```

### 回测数据准备

```python
# 准备回测数据（自动预加载和缓存）
provider.prepare_backtest_data(
    backtest_start_date=datetime(2023, 1, 1, tzinfo=timezone.utc),
    backtest_end_date=datetime(2023, 12, 31, tzinfo=timezone.utc)
)

# 获取窗口数据（高性能）
window_data = provider.get_data(
    date=datetime(2023, 6, 1, 12, 0, tzinfo=timezone.utc)
)
```

### 数据格式

返回 Polars DataFrame（或 Pandas）：

```python
# shape: (5, 6)
┌─────────────────────────────┬───────┬───────┬───────┬───────┬────────┐
│ Datetime                    ┆ Open  ┆ High  ┆ Low   ┆ Close ┆ Volume │
│ ---                         ┆ ---   ┆ ---   ┆ ---   ┆ ---   ┆ ---    │
│ datetime[ms, UTC]           ┆ f64   ┆ f64   ┆ f64   ┆ f64   ┆ f64    │
╞═════════════════════════════╪═══════╪═══════╪═══════╪═══════╪════════╡
│ 2023-01-01 00:00:00         ┆ 16500 ┆ 16600 ┆ 16400 ┆ 16550 ┆ 1000   │
│ 2023-01-01 01:00:00         ┆ 16550 ┆ 16700 ┆ 16500 ┆ 16650 ┆ 1200   │
│ 2023-01-01 02:00:00         ┆ 16650 ┆ 16800 ┆ 16600 ┆ 16700 ┆ 900    │
└─────────────────────────────┴───────┴───────┴───────┴───────┴────────┘
```

### 市场凭证配置

```python
from investing_algorithm_framework import MarketCredential

# 方式1：通过应用配置
app.add_market_credential(
    MarketCredential(
        market="binance",
        api_key="your_api_key",
        api_secret="your_api_secret"
    )
)

# 方式2：通过环境变量
# .env 文件
# BINANCE_API_KEY=your_key
# BINANCE_API_SECRET=your_secret
```

### 数据缓存机制

- **回测模式**: 自动缓存到 `storage_directory`
- **文件命名**: `{market}_{symbol}_{time_frame}.csv`
- **自动检测**: 优先从缓存加载，缺失时自动获取
- **缓存验证**: 检查数据时间范围是否满足需求

## CSVOHLCVDataProvider

从本地 CSV 文件加载 OHLCV 数据。

### 初始化

```python
CSVOHLCVDataProvider(
    storage_path: str,              # CSV 文件路径
    symbol: str,                    # 交易对标识
    time_frame: str,                # 时间框架
    market: str,                    # 市场标识
    window_size: int = None,        # 滑动窗口大小
    pandas: bool = False            # 返回 Pandas DataFrame
)
```

### CSV 文件格式

```csv
Datetime,Open,High,Low,Close,Volume
2023-01-01 00:00:00,16500,16600,16400,16550,1000
2023-01-01 01:00:00,16550,16700,16500,16650,1200
...
```

### 时间格式要求

- **时区**: 必须为 UTC
- **格式**: `YYYY-MM-DD HH:MM:SS` 或 ISO 8601
- **连续性**: 数据应连续，缺失点会被记录

### 使用示例

```python
from investing_algorithm_framework import CSVOHLCVDataProvider
from datetime import datetime, timezone

# 创建提供者
provider = CSVOHLCVDataProvider(
    storage_path="data/binance_btc_usdt_1h.csv",
    symbol="BTC/USDT",
    time_frame="1h",
    market="binance"
)

# 检查数据覆盖
has_data = provider.has_data(
    data_source=DataSource(...),
    start_date=datetime(2023, 1, 1, tzinfo=timezone.utc),
    end_date=datetime(2023, 12, 31, tzinfo=timezone.utc)
)

# 获取数据
data = provider.get_data(
    start_date=datetime(2023, 6, 1, tzinfo=timezone.utc),
    end_date=datetime(2023, 6, 30, tzinfo=timezone.utc)
)

# 检查缺失数据点
print(provider.missing_data_point_dates)
print(f"缺失 {provider.number_of_missing_data_points} 个数据点")
```

### 数据验证

```python
# 检查数据范围
print(f"开始: {provider._start_date_data_source}")
print(f"结束: {provider._end_date_data_source}")
print(f"总数: {provider.total_number_of_data_points}")
```

## PandasOHLCVDataProvider

从 Pandas DataFrame 提供数据。

### 初始化

```python
PandasOHLCVDataProvider(
    dataframe: pd.DataFrame,        # Pandas DataFrame
    symbol: str,                    # 交易对标识
    time_frame: str,                # 时间框架
    market: str,                    # 市场标识
    window_size: int = None,        # 滑动窗口大小
    pandas: bool = False            # 返回 Pandas DataFrame
)
```

### 使用示例

```python
import pandas as pd
from investing_algorithm_framework import PandasOHLCVDataProvider

# 准备 DataFrame
df = pd.DataFrame({
    'Datetime': pd.date_range('2023-01-01', periods=100, freq='1h'),
    'Open': [16500 + i*10 for i in range(100)],
    'High': [16600 + i*10 for i in range(100)],
    'Low': [16400 + i*10 for i in range(100)],
    'Close': [16550 + i*10 for i in range(100)],
    'Volume': [1000 + i*10 for i in range(100)]
})

# 创建提供者
provider = PandasOHLCVDataProvider(
    dataframe=df,
    symbol="BTC/USDT",
    time_frame="1h",
    market="binance"
)

# 获取数据
data = provider.get_data()
```

## 数据格式规范

### 标准列名

| 列名 | 类型 | 描述 |
|------|------|------|
| `Datetime` | datetime | 时间戳（UTC） |
| `Open` | float | 开盘价 |
| `High` | float | 最高价 |
| `Low` | float | 最低价 |
| `Close` | float | 收盘价 |
| `Volume` | float | 成交量 |

### Polars vs Pandas

```python
# 默认返回 Polars（高性能）
provider = CCXTOHLCVDataProvider(...)
data = provider.get_data()  # pl.DataFrame

# 设置返回 Pandas
provider = CCXTOHLCVDataProvider(..., pandas=True)
data = provider.get_data()  # pd.DataFrame
```

## 数据源配置

### DataSource 定义

```python
from investing_algorithm_framework import DataSource, DataType

data_source = DataSource(
    identifier="btc_ohlcv",          # 唯一标识
    data_type=DataType.OHLCV,        # 数据类型
    time_frame="1h",                 # 时间框架
    market="binance",                # 市场
    symbol="BTC/USDT",               # 交易对
    window_size=100                  # 窗口大小
)
```

### 添加数据源到策略

```python
from investing_algorithm_framework import TradingStrategy, DataSource

class MyStrategy(TradingStrategy):
    data_sources = [
        DataSource(
            identifier="btc_ohlcv",
            data_type=DataType.OHLCV,
            time_frame="1h",
            market="binance",
            symbol="BTC/USDT"
        )
    ]
```

## 工具函数

### get_default_data_providers()

获取默认数据提供者列表。

```python
providers = get_default_data_providers()
# [CCXTOHLCVDataProvider()]
```

### get_default_ohlcv_data_providers()

获取默认 OHLCV 数据提供者。

```python
providers = get_default_ohlcv_data_providers()
# [CCXTOHLCVDataProvider()]
```

## 高级功能

### 滑动窗口缓存

回测时自动预计算滑动窗口：

```python
provider = CCXTOHLCVDataProvider(window_size=100)
provider.prepare_backtest_data(start_date, end_date)

# 使用缓存快速获取窗口数据
window = provider.get_data(date=some_date)
```

### 缺失数据检测

自动检测和记录缺失数据点：

```python
provider.prepare_backtest_data(start_date, end_date)

if provider.missing_data_point_dates:
    print(f"发现 {len(provider.missing_data_point_dates)} 个缺失点")
    print(provider.missing_data_point_dates[:10])  # 前10个
```

### 多数据源组合

同时使用多个数据提供者：

```python
# CCXT 实时数据
ccxt_provider = CCXTOHLCVDataProvider(
    market="binance",
    symbol="BTC/USDT",
    time_frame="1h"
)

# CSV 历史数据
csv_provider = CSVOHLCVDataProvider(
    storage_path="historical_data.csv",
    symbol="BTC/USDT",
    time_frame="1h",
    market="binance"
)

# 添加到应用
app.add_data_source(ccxt_provider)
app.add_data_source(csv_provider)
```

## 性能优化

### 数据预加载

```python
# 回测前预加载所有数据
provider.prepare_backtest_data(
    backtest_start_date=start,
    backtest_end_date=end
)

# 使用内存缓存
data = provider.get_data(date=current_time)
```

### 批量获取

```python
# 一次性获取整个时间范围
data = provider.get_data(
    start_date=start_date,
    end_date=end_date,
    save=True  # 保存到缓存
)
```

### 使用 Polars

```python
# Polars 比 Pandas 快 10-100 倍
provider = CCXTOHLCVDataProvider(..., pandas=False)
data = provider.get_data()  # pl.DataFrame
```

## 常见问题 (FAQ)

### Q: 如何处理 API 速率限制？

A: CCXT 自动处理速率限制。如需自定义：

```python
config = {
    'enableRateLimit': True,
    'rateLimit': 1200  # 毫秒
}

provider = CCXTOHLCVDataProvider(config=config)
```

### Q: CSV 文件如何组织？

A: 建议按以下结构：

```
data/
├── binance/
│   ├── BTC_USDT_1h.csv
│   ├── ETH_USDT_1h.csv
│   └── ...
├── okx/
│   └── ...
```

### Q: 如何处理不同时区？

A: 所有时间必须转换为 UTC：

```python
from datetime import datetime, timezone

# 本地时间转 UTC
local_time = datetime(2023, 1, 1, 12, 0)
utc_time = local_time.replace(tzinfo=timezone.utc)
```

### Q: 数据缺失如何处理？

A:

1. 检查缺失数据：`provider.missing_data_point_dates`
2. 使用数据填充策略
3. 或从其他数据源补充

### Q: 如何同时获取多个交易对？

A: 创建多个提供者：

```python
providers = [
    CCXTOHLCVDataProvider(market="binance", symbol="BTC/USDT", ...),
    CCXTOHLCVDataProvider(market="binance", symbol="ETH/USDT", ...),
    CCXTOHLCVDataProvider(market="binance", symbol="BNB/USDT", ...)
]
```

## 相关文件清单

```
infrastructure/data_providers/
├── __init__.py                      # 模块导出
├── ccxt.py                          # CCXT 数据提供者（~1300 行）
├── csv.py                           # CSV 数据提供者（~600 行）
└── pandas.py                        # Pandas 数据提供者（~650 行）
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 data_providers 模块详细文档
- 整理三种数据提供者的使用方法
- 添加性能优化建议和常见问题
