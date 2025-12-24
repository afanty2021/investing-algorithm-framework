# Domain Models 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [domain](../) > **models**

## 模块职责

核心领域模型模块，定义框架的基础业务实体：

- **Order**: 订单模型
- **Portfolio**: 组合模型
- **Position**: 持仓模型
- **Trade**: 交易模型
- **TradeStopLoss / TradeTakeProfit**: 风险管理
- **PortfolioSnapshot**: 组合快照
- **Strategy**: 策略配置
- **AppMode**: 应用模式

## 核心模型

### Order（订单）

订单实体，表示交易指令。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 订单 ID |
| `order_type` | OrderType | 订单类型（LIMIT, MARKET） |
| `order_side` | OrderSide | 订单方向（BUY, SELL） |
| `order_status` | OrderStatus | 订单状态 |
| `target_symbol` | str | 目标标的 |
| `trading_symbol` | str | 交易货币 |
| `amount` | Decimal | 数量 |
| `price` | Decimal | 价格（限价单） |
| `market` | str | 市场/交易所 |
| `created_at` | datetime | 创建时间 |

#### 状态流转

```
PENDING -> SUBMITTED -> (PARTIALLY_FILLED | FILLED | CANCELED | FAILED)
```

### Portfolio（组合）

投资组合实体，表示账户状态。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 组合 ID |
| `identifier` | str | 标识符 |
| `trading_symbol` | str | 交易货币 |
| `total` | Decimal | 总价值 |
| `unallocated` | Decimal | 未分配资金 |
| `allocated` | Decimal | 已分配资金 |
| `positions` | List[Position] | 持仓列表 |

#### 关系

```
Portfolio (1) ----< Position (N)
```

### Position（持仓）

持仓实体，表示持有的资产。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 持仓 ID |
| `symbol` | str | 标的代码 |
| `amount` | Decimal | 数量 |
| `entry_price` | Decimal | 入场价格 |
| `current_price` | Decimal | 当前价格 |
| `profit_loss` | Decimal | 浮动盈亏 |
| `profit_loss_percentage` | Decimal | 盈亏百分比 |

### Trade（交易）

交易实体，表示开仓/平仓交易。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 交易 ID |
| `order_id` | str | 关联订单 ID |
| `symbol` | str | 标的代码 |
| `opened_at` | datetime | 开仓时间 |
| `closed_at` | datetime | 平仓时间 |
| `status` | TradeStatus | 状态（OPEN, CLOSED） |
| `stop_losses` | List[TradeStopLoss] | 止损列表 |
| `take_profits` | List[TradeTakeProfit] | 止盈列表 |
| `orders` | List[Order] | 关联订单 |

### TradeStopLoss / TradeTakeProfit

风险规则实体。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 规则 ID |
| `trade_id` | str | 关联交易 ID |
| `percentage_threshold` | Decimal | 触发百分比 |
| `trailing` | bool | 是否追踪 |
| `sell_percentage` | Decimal | 卖出百分比 |
| `triggered` | bool | 是否已触发 |
| `trigger_price` | Decimal | 触发价格 |

### PortfolioSnapshot

组合快照，用于回测和分析。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 快照 ID |
| `portfolio_id` | str | 组合 ID |
| `total` | Decimal | 总价值 |
| `unallocated` | Decimal | 未分配 |
| `allocated` | Decimal | 已分配 |
| `created_at` | datetime | 创建时间 |

## 枚举类型

### OrderType

```python
class OrderType(Enum):
    LIMIT = "LIMIT"      # 限价单
    MARKET = "MARKET"    # 市价单
```

### OrderSide

```python
class OrderSide(Enum):
    BUY = "BUY"          # 买入
    SELL = "SELL"        # 卖出
```

### OrderStatus

```python
class OrderStatus(Enum):
    PENDING = "PENDING"                  # 待处理
    SUBMITTED = "SUBMITTED"              # 已提交
    PARTIALLY_FILLED = "PARTIALLY_FILLED"  # 部分成交
    FILLED = "FILLED"                    # 已成交
    CANCELED = "CANCELED"                # 已取消
    FAILED = "FAILED"                    # 失败
```

### TradeStatus

```python
class TradeStatus(Enum):
    OPEN = "OPEN"        # 开仓
    CLOSED = "CLOSED"    # 已平仓
```

### TimeUnit

```python
class TimeUnit(Enum):
    SECOND = "SECOND"
    MINUTE = "MINUTE"
    HOUR = "HOUR"
    DAY = "DAY"
    WEEK = "WEEK"
    MONTH = "MONTH"
```

### DataType

```python
class DataType(Enum):
    OHLCV = "OHLCV"      # OHLCV 数据
    TICKER = "TICKER"    # 行情数据
    ORDER_BOOK = "ORDER_BOOK"  # 订单簿
```

## 使用示例

### 创建订单

```python
from investing_algorithm_framework.domain import Order, OrderType, OrderSide
from decimal import Decimal

order = Order(
    order_type=OrderType.LIMIT,
    order_side=OrderSide.BUY,
    target_symbol="BTC",
    trading_symbol="USDT",
    amount=Decimal("0.1"),
    price=Decimal("50000")
)
```

### 创建组合

```python
from investing_algorithm_framework.domain import Portfolio, PortfolioConfiguration

# 从配置创建
portfolio = Portfolio.from_portfolio_configuration(
    PortfolioConfiguration(
        identifier="my_portfolio",
        trading_symbol="USDT",
        initial_balance=Decimal("10000")
    )
)
```

### 创建交易

```python
from investing_algorithm_framework.domain import Trade
from datetime import datetime, timezone

trade = Trade(
    symbol="BTC",
    opened_at=datetime.now(tz=timezone.utc),
    status=TradeStatus.OPEN
)
```

## 相关文件清单

```
domain/models/
├── __init__.py
├── base_model.py                 # 基础模型
├── event.py                      # 事件模型
├── strategy.py                   # 策略模型
├── app_mode.py                   # 应用模式
├── data/
│   ├── __init__.py
│   ├── data_source.py            # 数据源
│   └── data_type.py              # 数据类型
├── market/
│   ├── __init__.py
│   └── market_credential.py      # 市场凭证
├── order/
│   ├── __init__.py
│   ├── order.py                  # 订单模型
│   ├── order_type.py             # 订单类型
│   ├── order_side.py             # 订单方向
│   └── order_status.py           # 订单状态
├── portfolio/
│   ├── __init__.py
│   ├── portfolio.py              # 组合模型
│   ├── portfolio_configuration.py  # 组合配置
│   └── portfolio_snapshot.py     # 组合快照
├── position/
│   ├── __init__.py
│   ├── position.py               # 持仓模型
│   ├── position_size.py          # 仓位大小
│   └── position_snapshot.py      # 持仓快照
├── risk_rules/
│   ├── __init__.py
│   ├── stop_loss.py              # 止损规则
│   └── take_profit.py            # 止盈规则
└── trades/
    ├── __init__.py
    ├── trade.py                  # 交易模型
    ├── trade_status.py           # 交易状态
    ├── trade_stop_loss.py        # 交易止损
    └── trade_take_profit.py      # 交易止盈
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 domain models 模块文档
