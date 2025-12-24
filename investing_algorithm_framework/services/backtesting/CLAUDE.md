# Backtesting Service 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [services](../) > **backtesting**

## 模块职责

回测服务模块，提供完整的策略回测执行能力：

- **向量化回测**: 批量处理数据，快速原型开发
- **事件驱动回测**: 逐条处理，精确模拟真实交易
- **多策略支持**: 同时回测多个策略
- **指标计算**: 自动计算 50+ 性能指标

## 核心服务

### BacktestService

回测执行的核心服务。

#### 初始化

```python
BacktestService(
    data_provider_service: DataProviderService,
    order_service: OrderService,
    portfolio_service: PortfolioService,
    portfolio_snapshot_service: PortfolioSnapshotService,
    position_repository,
    trade_service: TradeService,
    configuration_service,
    portfolio_configuration_service: PortfolioConfigurationService
)
```

#### 主要方法

```python
# 向量化回测（快速）
def create_vector_backtest(
    strategy: TradingStrategy,
    backtest_date_range: BacktestDateRange,
    risk_free_rate: float = 0.027,
    initial_amount: float = None,
    trading_symbol: str = None,
    market: str = None
) -> BacktestRun

# 事件驱动回测（精确）
def run(
    strategy: TradingStrategy,
    backtest_date_range: BacktestDateRange,
    initial_amount: float = None,
    trading_symbol: str = None
) -> BacktestRun

# 验证策略
def validate_strategy_for_vector_backtest(strategy)
```

## 回测类型

### 向量化回测

基于策略的批量信号生成，快速执行回测。

#### 要求

策略必须实现：
- `generate_buy_signals(data)` -> 返回买入信号 Series
- `generate_sell_signals(data)` -> 返回卖出信号 Series

#### 示例

```python
from investing_algorithm_framework import TradingStrategy
import pandas as pd

class MyStrategy(TradingStrategy):
    time_unit = TimeUnit.HOUR
    interval = 1
    symbols = ["BTC"]

    def generate_buy_signals(self, data):
        # data 是 Dict[str, pd.DataFrame]
        df = data["BTC"]
        # 简单移动平均线策略
        sma_short = df['Close'].rolling(10).mean()
        sma_long = df['Close'].rolling(30).mean()
        return {"BTC": sma_short > sma_long}

    def generate_sell_signals(self, data):
        df = data["BTC"]
        sma_short = df['Close'].rolling(10).mean()
        sma_long = df['Close'].rolling(30).mean()
        return {"BTC": sma_short < sma_long}
```

### 事件驱动回测

逐条处理数据，精确模拟真实交易场景。

#### 特点

- 精确的订单执行模拟
- 考虑滑点和延迟
- 支持复杂订单类型

## 使用示例

### 基本回测

```python
from investing_algorithm_framework import create_app, BacktestDateRange
from datetime import datetime, timezone

app = create_app()
app.add_strategy(MyStrategy())

# 运行回测
backtest_date_range = BacktestDateRange(
    start_date=datetime(2023, 1, 1, tzinfo=timezone.utc),
    end_date=datetime(2023, 12, 31, tzinfo=timezone.utc)
)

backtest = app.run_backtest(
    backtest_date_range=backtest_date_range,
    initial_amount=10000
)

# 查看结果
print(f"总收益: {backtest.total_return}%")
print(f"夏普比率: {backtest.sharpe_ratio}")
print(f"最大回撤: {backtest.max_drawdown}%")
```

### 向量化回测

```python
# 使用向量化回测（更快）
backtest_run = backtest_service.create_vector_backtest(
    strategy=MyStrategy(),
    backtest_date_range=backtest_date_range,
    initial_amount=10000,
    risk_free_rate=0.03
)
```

## 数据源配置

### 添加数据源

```python
from investing_algorithm_framework import DataSource, DataType

class MyStrategy(TradingStrategy):
    data_sources = [
        DataSource(
            identifier="btc_ohlcv",
            data_type=DataType.OHLCV,
            time_frame="1h",
            market="binance",
            symbol="BTC/USDT",
            window_size=100
        )
    ]
```

## 常见问题 (FAQ)

### Q: 两种回测方式如何选择？

A:
- **向量化**: 快速原型开发、参数优化
- **事件驱动**: 最终验证、生产前测试

### Q: 如何处理多交易对？

A: 策略中为每个交易对返回信号：

```python
def generate_buy_signals(self, data):
    signals = {}
    for symbol, df in data.items():
        signals[symbol] = df['Close'].rolling(10).mean() > df['Close']
    return signals
```

## 相关文件清单

```
services/backtesting/
├── __init__.py
└── backtest_service.py        # 回测服务实现（~700 行）
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 backtesting service 模块文档
