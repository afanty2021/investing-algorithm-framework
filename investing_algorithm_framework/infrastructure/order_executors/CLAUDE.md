# Order Executors 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [infrastructure](../) > **order_executors**

## 模块职责

订单执行器模块，提供订单执行能力：

- **CCXTOrderExecutor**: 通过 CCXT 在真实交易所执行订单
- **BacktestOrderExecutor**: 回测模式下的订单执行模拟

## 核心类

### CCXTOrderExecutor

通过 CCXT 与交易所交互执行订单。

#### 初始化

```python
CCXTOrderExecutor(
    market: str,                          # 交易所名称
    market_credentials: MarketCredential = None  # 市场凭证
)
```

#### 支持的交易所

通过 CCXT 支持 100+ 交易所，包括：
- Binance
- OKX
- Coinbase
- Kraken
- Bybit
- Huobi

### BacktestOrderExecutor

回测模式下的订单执行器。

#### 特点

- 模拟订单执行
- 假设完美成交（无滑点）
- 不连接真实交易所

## 使用示例

### 配置订单执行器

```python
from investing_algorithm_framework import create_app, MarketCredential

# 创建带凭证的应用
app = create_app()

# 添加市场凭证
app.add_market_credential(
    MarketCredential(
        market="binance",
        api_key="your_api_key",
        api_secret="your_api_secret"
    )
)

# 运行（自动使用 CCXT 订单执行器）
app.run()
```

### 回测模式

```python
# 回测自动使用 BacktestOrderExecutor
backtest = app.run_backtest(
    backtest_date_range=date_range,
    initial_amount=10000
)
```

## 相关文件清单

```
infrastructure/order_executors/
├── __init__.py
├── ccxt_order_executor.py          # CCXT 订单执行器（~250 行）
└── backtest_oder_executor.py       # 回测订单执行器（~30 行）
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 order executors 模块文档
