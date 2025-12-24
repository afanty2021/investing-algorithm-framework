# Infrastructure 模块

[根目录](../../CLAUDE.md) > [investing_algorithm_framework](../) > **infrastructure**

## 模块职责

基础设施模块，提供数据持久化、外部系统集成和数据获取能力：

- **数据库**: SQLAlchemy ORM 配置和会话管理
- **数据提供者**: CCXT、CSV、Pandas 数据源集成
- **订单执行器**: CCXT 交易所集成、回测订单执行
- **组合提供者**: CCXT 组合同步
- **仓储**: 数据访问层抽象
- **云存储**: Azure Blob Storage、AWS S3 状态管理

## 入口与启动

### 主要入口点

| 文件 | 描述 |
|------|------|
| `__init__.py` | 模块公共导出 |
| `database/` | 数据库配置和模型 |
| `data_providers/` | 数据提供者实现 |
| `order_executors/` | 订单执行器实现 |
| `portfolio_providers/` | 组合提供者实现 |
| `repositories/` | 仓储模式实现 |
| `services/` | 云存储服务 |

### 核心导入

```python
from investing_algorithm_framework.infrastructure import (
    # 数据库
    setup_sqlalchemy, Session, create_all_tables,
    # ORM 模型
    SQLPortfolio, SQLOrder, SQLPosition, SQLTrade,
    # 数据提供者
    CCXTOHLCVDataProvider, CSVOHLCVDataProvider, PandasOHLCVDataProvider,
    # 订单执行
    CCXTOrderExecutor, BacktestOrderExecutor,
    # 组合提供
    CCXTPortfolioProvider,
    # 云存储
    AzureBlobStorageStateHandler, AWSS3StorageStateHandler
)
```

## 对外接口

### 数据提供者

#### CCXTOHLCVDataProvider

使用 CCXT 获取交易所 OHLCV 数据：

```python
CCXTOHLCVDataProvider(
    market="binance",           # 交易所名称
    symbol="BTC/USDT",          # 交易对
    time_frame="1h",            # 时间框架
    market_credentials=None     # 市场凭证（可选）
)
```

#### CSVOHLCVDataProvider

从 CSV 文件读取 OHLCV 数据：

```python
CSVOHLCVDataProvider(
    csv_file_path="data.csv",   # CSV 文件路径
    date_column="Date",         # 日期列名
    symbol="BTC"                # 标的代码
)
```

#### PandasOHLCVDataProvider

从 Pandas DataFrame 提供数据：

```python
PandasOHLCVDataProvider(
    data_frame=df,              # Pandas DataFrame
    symbol="BTC"                # 标的代码
)
```

### 订单执行器

#### CCXTOrderExecutor

通过 CCXT 在交易所执行订单：

```python
CCXTOrderExecutor(
    market="binance",           # 交易所名称
    market_credentials=None     # 市场凭证
)
```

#### BacktestOrderExecutor

回测模式下的订单执行器：

```python
BacktestOrderExecutor()
```

### 组合提供者

#### CCXTPortfolioProvider

从 CCXT 交易所同步组合信息：

```python
CCXTPortfolioProvider(
    market="binance",           # 交易所名称
    trading_symbol="USDT",      # 交易货币
    market_credentials=None     # 市场凭证
)
```

### 云存储状态管理

#### AzureBlobStorageStateHandler

Azure Blob Storage 状态管理：

```python
AzureBlobStorageStateHandler(
    connection_string="...",    # 连接字符串
    container_name="state"      # 容器名称
)
```

#### AWSS3StorageStateHandler

AWS S3 状态管理：

```python
AWSS3StorageStateHandler(
    bucket_name="my-bucket",    # S3 存储桶名称
    region="us-east-1"          # AWS 区域
)
```

### 数据库配置

```python
from investing_algorithm_framework.infrastructure import setup_sqlalchemy

# 设置 SQLAlchemy
setup_sqlalchemy(
    db_uri="sqlite:///bot.db"   # 数据库连接 URI
)

# 创建所有表
create_all_tables()

# 清空数据库
clear_db()
```

## 关键依赖与配置

### 依赖项

- `SQLAlchemy`: ORM 框架
- `Flask-Migrate`: 数据库迁移
- `CCXT`: 交易所集成
- `azure-storage-blob`: Azure 存储
- `boto3`: AWS SDK

### 数据库模型

| ORM 模型 | 对应领域模型 |
|----------|-------------|
| `SQLPortfolio` | Portfolio |
| `SQLOrder` | Order |
| `SQLPosition` | Position |
| `SQLTrade` | Trade |
| `SQLPortfolioSnapshot` | PortfolioSnapshot |
| `SQLPositionSnapshot` | PositionSnapshot |
| `SQLTradeTakeProfit` | TradeTakeProfit |
| `SQLTradeStopLoss` | TradeStopLoss |

### 仓储模式

| 仓储 | 功能 |
|------|------|
| `SQLPortfolioRepository` | 组合数据访问 |
| `SQLOrderRepository` | 订单数据访问 |
| `SQLPositionRepository` | 持仓数据访问 |
| `SQLTradeRepository` | 交易数据访问 |
| `SQLTradeTakeProfitRepository` | 止盈数据访问 |
| `SQLTradeStopLossRepository` | 止损数据访问 |

## 数据模型

### 数据库表结构

```sql
-- portfolios 表
CREATE TABLE portfolios (
    id VARCHAR PRIMARY KEY,
    identifier VARCHAR,
    trading_symbol VARCHAR,
    total DECIMAL,
    unallocated DECIMAL,
    allocated DECIMAL
);

-- orders 表
CREATE TABLE orders (
    id VARCHAR PRIMARY KEY,
    portfolio_id VARCHAR,
    order_type VARCHAR,
    order_side VARCHAR,
    order_status VARCHAR,
    target_symbol VARCHAR,
    amount DECIMAL,
    price DECIMAL
);

-- positions 表
CREATE TABLE positions (
    id VARCHAR PRIMARY KEY,
    portfolio_id VARCHAR,
    symbol VARCHAR,
    amount DECIMAL,
    entry_price DECIMAL
);

-- trades 表
CREATE TABLE trades (
    id VARCHAR PRIMARY KEY,
    order_id VARCHAR,
    portfolio_id VARCHAR,
    symbol VARCHAR,
    status VARCHAR,
    opened_at TIMESTAMP,
    closed_at TIMESTAMP
);
```

## 测试与质量

### 测试目录

```
tests/infrastructure/
├── data_providers/         # 数据提供者测试
│   ├── test_ccxt_ohlcv_data_provider.py
│   └── test_csv_ohlcv_data_provider.py
├── models/                 # ORM 模型测试
│   ├── test_portfolio.py
│   ├── test_order.py
│   └── test_position.py
└── repositories/           # 仓储测试
    ├── orders/
    └── trades/
```

### 测试覆盖

- 数据提供者数据获取
- ORM 模型 CRUD 操作
- 仓储模式数据访问
- 订单执行器模拟

## 常见问题 (FAQ)

### Q: 如何添加自定义数据提供者？

A: 实现 `DataProvider` 接口：

```python
from investing_algorithm_framework.domain import DataProvider

class MyDataProvider(DataProvider):
    def get_data(self, symbol, time_frame, ...):
        # 返回 Polars 或 Pandas DataFrame
        return data_frame

    def get_ticker(self, symbol, ...):
        # 返回行情数据
        return ticker_data
```

### Q: 如何配置市场凭证？

A: 通过环境变量或代码配置：

```python
# 方式1：环境变量
# .env 文件
BINANCE_API_KEY=your_key
BINANCE_API_SECRET=your_secret

# 方式2：代码配置
from investing_algorithm_framework import MarketCredential

credential = MarketCredential(
    market="binance",
    api_key="your_key",
    api_secret="your_secret"
)

app.add_market_credential(credential)
```

### Q: 如何使用云存储？

A: 创建状态处理器并传递给应用：

```python
from investing_algorithm_framework import (
    create_app, AWSS3StorageStateHandler
)

state_handler = AWSS3StorageStateHandler(
    bucket_name="my-bot-state",
    region="us-east-1"
)

app = create_app(state_handler=state_handler)
```

## 相关文件清单

### 数据库目录

```
infrastructure/database/
├── __init__.py
├── sql_alchemy.py           # SQLAlchemy 配置
└── models/                  # ORM 模型定义
    ├── __init__.py
    ├── portfolio.py
    ├── order.py
    ├── position.py
    ├── trades.py
    └── ...
```

### 数据提供者目录

```
infrastructure/data_providers/
├── __init__.py
├── ccxt_ohlcv_data_provider.py      # CCXT OHLCV 数据提供者
├── csv_ohlcv_data_provider.py       # CSV OHLCV 数据提供者
├── pandas_ohlcv_data_provider.py    # Pandas 数据提供者
└── data_provider_service.py         # 数据提供者服务
```

### 订单执行器目录

```
infrastructure/order_executors/
├── __init__.py
├── ccxt_order_executor.py           # CCXT 订单执行器
└── backtest_order_executor.py       # 回测订单执行器
```

### 组合提供者目录

```
infrastructure/portfolio_providers/
├── __init__.py
├── ccxt_portfolio_provider.py       # CCXT 组合提供者
└── portfolio_provider_service.py    # 组合提供者服务
```

### 仓储目录

```
infrastructure/repositories/
├── __init__.py
├── order_repository.py              # 订单仓储
├── portfolio_repository.py          # 组合仓储
├── position_repository.py           # 持仓仓储
├── trade_repository.py              # 交易仓储
└── ...
```

### 云服务目录

```
infrastructure/services/
├── __init__.py
├── azure/
│   ├── __init__.py
│   └── azure_blob_storage_state_handler.py   # Azure Blob 存储
└── aws/
    ├── __init__.py
    └── aws_s3_state_handler.py               # AWS S3 存储
```

## 变更记录 (Changelog)

### 2025-12-24 10:25:42 - 模块文档创建
- 创建 infrastructure 模块详细文档
- 整理数据提供者和订单执行器说明
- 添加文件清单和常见问题
