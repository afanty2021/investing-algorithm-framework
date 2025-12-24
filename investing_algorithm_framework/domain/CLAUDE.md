# Domain 模块

[根目录](../../CLAUDE.md) > [investing_algorithm_framework](../) > **domain**

## 模块职责

领域模型模块，定义框架的核心业务实体、枚举类型和领域服务：

- **实体模型**: Order, Portfolio, Position, Trade, Backtest
- **枚举类型**: OrderType, OrderSide, OrderStatus, TimeUnit, DataType
- **值对象**: TimeInterval, TimeFrame, BacktestDateRange
- **异常**: OperationalException, ApiException, DataError
- **配置**: 应用配置和常量定义

## 入口与启动

### 主要入口点

| 文件 | 描述 |
|------|------|
| `__init__.py` | 模块公共导出 |
| `models/` | 领域模型定义 |
| `backtesting/` | 回测相关模型 |
| `config.py` | 配置定义 |
| `constants.py` | 常量定义 |

### 核心导入

```python
from investing_algorithm_framework.domain import (
    # 实体
    Order, Portfolio, Position, Trade,
    # 枚举
    OrderType, OrderSide, OrderStatus, TimeUnit, TimeFrame, DataType,
    # 值对象
    TimeInterval, BacktestDateRange, DataSource,
    # 回测
    Backtest, BacktestMetrics, BacktestRun,
    # 异常
    OperationalException, ApiException, DataError
)
```

## 对外接口

### 核心实体

#### Order（订单）

```python
class Order:
    id: str                      # 订单 ID
    order_type: OrderType        # 订单类型（LIMIT, MARKET）
    order_side: OrderSide        # 订单方向（BUY, SELL）
    order_status: OrderStatus    # 订单状态（PENDING, FILLED, CANCELED）
    target_symbol: str           # 目标标的
    trading_symbol: str          # 交易货币
    amount: Decimal              # 数量
    price: Decimal               # 价格
    market: str                  # 市场/交易所
```

#### Portfolio（组合）

```python
class Portfolio:
    id: str                      # 组合 ID
    identifier: str              # 标识符
    trading_symbol: str          # 交易货币
    total: Decimal               # 总价值
    unallocated: Decimal         # 未分配资金
    allocated: Decimal           # 已分配资金
```

#### Position（持仓）

```python
class Position:
    id: str                      # 持仓 ID
    symbol: str                  # 标的代码
    amount: Decimal              # 数量
    entry_price: Decimal         # 入场价格
    current_price: Decimal       # 当前价格
    profit_loss: Decimal         # 浮动盈亏
```

#### Trade（交易）

```python
class Trade:
    id: str                      # 交易 ID
    order_id: str                # 关联订单 ID
    symbol: str                  # 标的代码
    opened_at: datetime          # 开仓时间
    closed_at: datetime          # 平仓时间
    status: TradeStatus          # 状态（OPEN, CLOSED）
```

#### Backtest（回测）

```python
class Backtest:
    id: str                      # 回测 ID
    strategy_id: str             # 策略 ID
    start_date: datetime         # 开始日期
    end_date: datetime           # 结束日期
    initial_amount: Decimal      # 初始资金
    metrics: BacktestMetrics     # 回测指标
```

### 枚举类型

| 枚举 | 值 |
|------|-----|
| `OrderType` | LIMIT, MARKET |
| `OrderSide` | BUY, SELL |
| `OrderStatus` | PENDING, SUBMITTED, FILLED, PARTIALLY_FILLED, CANCELED, FAILED |
| `TradeStatus` | OPEN, CLOSED |
| `TimeUnit` | SECOND, MINUTE, HOUR, DAY, WEEK, MONTH |
| `DataType` | OHLCV, TICKER, ORDER_BOOK |

### 值对象

#### DataSource（数据源）

```python
class DataSource:
    identifier: str              # 唯一标识符
    data_type: DataType          # 数据类型
    time_frame: str              # 时间框架（如 "1h", "1d"）
    market: str                  # 市场/交易所
    symbol: str                  # 交易对
    window_size: int             # 窗口大小
    pandas: bool                 # 是否返回 Pandas DataFrame
```

#### BacktestDateRange（回测日期范围）

```python
class BacktestDateRange:
    start_date: datetime         # 开始日期
    end_date: datetime           # 结束日期
```

#### PositionSize（仓位大小）

```python
class PositionSize:
    symbol: str                  # 标的代码
    percentage_of_portfolio: Decimal  # 组合百分比
```

#### StopLossRule / TakeProfitRule

```python
class StopLossRule:
    symbol: str
    percentage_threshold: Decimal   # 触发百分比
    trailing: bool                   # 是否追踪
    sell_percentage: Decimal         # 卖出百分比

class TakeProfitRule:
    symbol: str
    percentage_threshold: Decimal   # 触发百分比
    trailing: bool                   # 是否追踪
    sell_percentage: Decimal         # 卖出百分比
```

### 回测模型

#### BacktestMetrics（回测指标）

```python
class BacktestMetrics:
    # 收益指标
    total_return: Decimal
    cagr: Decimal
    cumulative_return: Decimal

    # 风险指标
    max_drawdown: Decimal
    max_drawdown_duration: int
    annual_volatility: Decimal

    # 风险调整收益
    sharpe_ratio: Decimal
    sortino_ratio: Decimal
    calmar_ratio: Decimal

    # 交易统计
    win_rate: Decimal
    win_loss_ratio: Decimal
    number_of_trades: int
```

#### BacktestRun（回测运行）

一次回测的单次运行，包含多个时间段的回测。

#### BacktestSummaryMetrics（汇总指标）

多次回测运行的汇总统计。

## 关键依赖与配置

### 常量定义

```python
# 时间格式
DATETIME_FORMAT = "%Y-%m-%d %H:%M:%S"
CCXT_DATETIME_FORMAT = "%Y-%m-%dT%H:%M:%SZ"

# 目录
APPLICATION_DIRECTORY    # 应用目录
RESOURCE_DIRECTORY       # 资源目录
DATABASE_DIRECTORY_PATH  # 数据库目录
DATA_DIRECTORY           # 数据目录

# 配置键
APP_MODE                  # 应用模式
BACKTESTING_FLAG          # 回测标志
BACKTESTING_START_DATE    # 回测开始日期
BACKTESTING_END_DATE      # 回测结束日期
```

### 异常层级

```
Exception
├── OperationalException      # 操作异常
├── ApiException             # API 异常
│   └── PermissionDeniedApiException
├── DataError                # 数据异常
├── NetworkError             # 网络异常
└── ImproperlyConfigured     # 配置错误
```

## 数据模型

### 模型关系

```
Portfolio (1) ----< Position (N)
                 |
                 ----< Order (N) ----< Trade (N)
                                            |
                                    TradeStopLoss / TradeTakeProfit
```

### 时间处理

- 所有时间字段使用 `datetime` 类型
- 默认时区为 UTC
- 支持 Polars 和 Pandas 时间索引

## 测试与质量

### 测试目录

```
tests/domain/
├── models/
│   ├── backtesting/      # 回测模型测试
│   ├── data/             # 数据模型测试
│   ├── portfolio/        # 组合模型测试
│   └── trades/           # 交易模型测试
├── metrics/              # 指标测试
└── utils/                # 工具函数测试
```

### 测试覆盖

- 模型创建和验证
- 枚举值转换
- 日期范围计算
- 指标计算

## 常见问题 (FAQ)

### Q: OrderType.LIMIT 和 OrderType.MARKET 有什么区别？

A:
- **LIMIT**: 指定价格的限价单，可能不会立即成交
- **MARKET**: 市价单，立即以当前市场价格成交

### Q: TimeUnit 和 TimeFrame 有什么区别？

A:
- **TimeUnit**: 策略运行的时间单位（HOUR, DAY 等）
- **TimeFrame**: 数据的时间框架（如 "1h", "1d"）

### Q: 如何创建自定义数据源？

A: 创建 DataSource 实例并指定参数：

```python
from investing_algorithm_framework import DataSource, DataType

data_source = DataSource(
    identifier="my_data",
    data_type=DataType.OHLCV,
    time_frame="1h",
    market="binance",
    symbol="BTC/USDT",
    window_size=100,
    pandas=True
)
```

## 相关文件清单

### 核心文件

```
domain/
├── __init__.py
├── config.py                 # 配置定义
├── constants.py              # 常量定义
├── data_provider.py          # 数据提供者接口
├── order_executor.py         # 订单执行器接口
├── portfolio_provider.py     # 组合提供者接口
├── decimal_parsing.py        # 十进制解析
├── exceptions.py             # 异常定义
├── data_structures.py        # 数据结构
└── utils.py                  # 工具函数
```

### 模型目录

```
domain/models/
├── __init__.py
├── app_mode.py               # 应用模式
├── base_model.py             # 基础模型
├── event.py                  # 事件模型
├── strategy.py               # 策略模型
├── data/
│   ├── __init__.py
│   ├── data_source.py        # 数据源
│   └── data_type.py          # 数据类型
├── market/
│   ├── __init__.py
│   └── market_credential.py  # 市场凭证
├── order/
│   ├── __init__.py
│   ├── order.py              # 订单模型
│   ├── order_type.py         # 订单类型
│   ├── order_side.py         # 订单方向
│   └── order_status.py       # 订单状态
├── portfolio/
│   ├── __init__.py
│   ├── portfolio.py          # 组合模型
│   ├── portfolio_configuration.py  # 组合配置
│   └── portfolio_snapshot.py       # 组合快照
├── position/
│   ├── __init__.py
│   ├── position.py           # 持仓模型
│   ├── position_size.py      # 仓位大小
│   └── position_snapshot.py  # 持仓快照
├── risk_rules/
│   ├── __init__.py
│   ├── stop_loss.py          # 止损规则
│   └── take_profit.py        # 止盈规则
└── trades/
    ├── __init__.py
    ├── trade.py              # 交易模型
    ├── trade_status.py       # 交易状态
    ├── trade_stop_loss.py    # 交易止损
    └── trade_take_profit.py  # 交易止盈
```

### 回测目录

```
domain/backtesting/
├── __init__.py
├── backtest.py                       # 回测模型
├── backtest_date_range.py            # 回测日期范围
├── backtest_evaluation_focuss.py     # 回测评估焦点
├── backtest_metrics.py               # 回测指标
├── backtest_permutation_test.py      # 排列测试
├── backtest_run.py                   # 回测运行
├── backtest_summary_metrics.py       # 汇总指标
└── combine_backtests.py              # 回测合并
```

## 变更记录 (Changelog)

### 2025-12-24 10:25:42 - 模块文档创建
- 创建 domain 模块详细文档
- 整理核心实体和枚举说明
- 添加文件清单和常见问题
