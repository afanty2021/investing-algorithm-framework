# Services 模块

[根目录](../../CLAUDE.md) > [investing_algorithm_framework](../) > **services**

## 模块职责

业务服务模块，提供组合管理、订单处理、指标计算和回测服务：

- **BacktestService**: 回测执行和管理
- **OrderService**: 订单创建、更新和状态管理
- **PortfolioService**: 组合管理和同步
- **TradeService**: 交易生命周期管理
- **MetricsService**: 50+ 性能指标计算
- **PositionService**: 持仓管理
- **DataProviderService**: 数据提供者协调

## 入口与启动

### 主要入口点

| 文件 | 描述 |
|------|------|
| `__init__.py` | 模块公共导出 |
| `backtesting/` | 回测服务 |
| `order_service/` | 订单服务 |
| `portfolios/` | 组合服务 |
| `trade_service/` | 交易服务 |
| `metrics/` | 指标计算 |
| `data_providers/` | 数据提供者服务 |

### 核心导入

```python
from investing_algorithm_framework.services import (
    # 服务
    BacktestService,
    OrderService,
    PortfolioService,
    TradeService,
    TradeStopLossService,
    TradeTakeProfitService,

    # 指标函数
    get_sharpe_ratio,
    get_sortino_ratio,
    get_max_drawdown,
    get_cagr,
    get_win_rate,
    get_equity_curve,

    # 工具
    create_backtest_metrics,
    create_backtest_metrics_for_backtest
)
```

## 对外接口

### 回测服务

```python
class BacktestService:
    def run_backtest(
        self,
        strategy,
        backtest_date_range: BacktestDateRange,
        initial_amount: float
    ) -> Backtest

    def run_vectorized_backtest(
        self,
        strategy,
        backtest_date_range: BacktestDateRange,
        initial_amount: float
    ) -> Backtest
```

### 订单服务

```python
class OrderService:
    def create_order(self, order: Order) -> Order
    def get_order(self, order_id: str) -> Order
    def get_orders(self, **kwargs) -> List[Order]
    def update_order(self, order_id: str, **kwargs)
    def cancel_order(self, order_id: str)
```

### 组合服务

```python
class PortfolioService:
    def get_portfolio(self) -> Portfolio
    def sync_portfolio(self, market: str)
    def get_snapshot(self, portfolio_id: str) -> PortfolioSnapshot
    def create_snapshot(self)
```

### 交易服务

```python
class TradeService:
    def get_trade(self, trade_id: str) -> Trade
    def get_trades(self, **kwargs) -> List[Trade]
    def get_open_trades(self, **kwargs) -> List[Trade]
    def get_closed_trades(self) -> List[Trade]
    def close_trade(self, trade: Trade)

class TradeStopLossService:
    def add_stop_loss(self, trade, percentage, trailing, sell_percentage)
    def update_stop_loss(self, trade, percentage)
    def check_stop_loss(self, trade, current_price)

class TradeTakeProfitService:
    def add_take_profit(self, trade, percentage, trailing, sell_percentage)
    def update_take_profit(self, trade, percentage)
    def check_take_profit(self, trade, current_price)
```

### 指标服务

#### 收益指标

```python
get_total_return(equity_curve)
get_cagr(backtest)
get_cumulative_return(backtest)
get_yearly_returns(backtest)
get_monthly_returns(backtest)
```

#### 风险指标

```python
get_max_drawdown(equity_curve)
get_max_drawdown_duration(backtest)
get_annual_volatility(returns)
get_standard_deviation_returns(returns)
get_standard_deviation_downside_returns(returns)
```

#### 风险调整收益

```python
get_sharpe_ratio(returns, risk_free_rate)
get_sortino_ratio(returns, risk_free_rate)
get_calmar_ratio(backtest)
```

#### 交易统计

```python
get_win_rate(trades)
get_win_loss_ratio(trades)
get_average_trade_duration(trades)
get_profit_factor(trades)
get_number_of_trades(trades)
get_best_trade(trades)
get_worst_trade(trades)
```

#### 其他指标

```python
get_equity_curve(backtest)
get_exposure_ratio(backtest)
get_trade_frequency(backtest)
get_price_efficiency_ratio(trades)
```

## 关键依赖与配置

### 服务依赖

```
BacktestService
  ├── OrderService
  ├── PortfolioService
  ├── TradeService
  ├── DataProviderService
  └── MetricsService

OrderService
  ├── OrderExecutorLookup
  └── RepositoryService

PortfolioService
  ├── PortfolioProviderLookup
  └── RepositoryService
```

### 配置项

```python
# 无风险利率（用于 Sharpe/Sortino 计算）
RISK_FREE_RATE = 0.02  # 2% 年化

# 交易统计配置
TRADE_FREQUENCY_PERIOD = "daily"
```

## 数据模型

### BacktestMetrics

```python
@dataclass
class BacktestMetrics:
    # 收益
    total_return: float
    cagr: float
    cumulative_return: float

    # 风险
    max_drawdown: float
    max_drawdown_duration: int
    annual_volatility: float

    # 风险调整收益
    sharpe_ratio: float
    sortino_ratio: float
    calmar_ratio: float

    # 交易
    win_rate: float
    win_loss_ratio: float
    number_of_trades: int
    average_trade_duration: float

    # 其他
    equity_curve: pd.Series
    exposure_ratio: float
```

## 测试与质量

### 测试目录

```
tests/services/
├── metrics/                     # 指标测试
│   ├── test_cagr.py
│   ├── test_drawdowns.py
│   ├── test_sharpe_ratio.py
│   ├── test_sortino_ratio.py
│   ├── test_volatility.py
│   ├── test_win_rate.py
│   └── ...
└── ...
```

### 测试覆盖

- 50+ 性能指标计算
- 回测服务执行
- 订单生命周期
- 组合同步
- 交易管理

## 常见问题 (FAQ)

### Q: 如何计算自定义指标？

A: 使用 `equity_curve` 和 `trades` 数据：

```python
from investing_algorithm_framework.services import (
    get_equity_curve, get_trades
)

equity_curve = get_equity_curve(backtest)
trades = get_trades(backtest)

# 自定义计算
custom_metric = (equity_curve.iloc[-1] / equity_curve.iloc[0] - 1) * 100
```

### Q: Sharpe Ratio 和 Sortino Ratio 有什么区别？

A:
- **Sharpe Ratio**: 使用总波动率计算风险惩罚
- **Sortino Ratio**: 仅使用下行波动率计算风险惩罚，更关注亏损

### Q: 如何获取回测的详细交易列表？

A: 使用 Backtest 对象：

```python
backtest = app.run_backtest(...)

# 获取所有交易
trades = backtest.trades

# 获取已平仓交易
closed_trades = [t for t in trades if t.closed]

# 打印交易详情
for trade in closed_trades:
    print(f"Symbol: {trade.symbol}, P/L: {trade.profit_loss}")
```

### Q: 如何在回测后生成完整指标报告？

A: 使用 `create_backtest_metrics`：

```python
from investing_algorithm_framework.services import create_backtest_metrics

metrics = create_backtest_metrics(backtest)

print(f"Total Return: {metrics.total_return}%")
print(f"Sharpe Ratio: {metrics.sharpe_ratio}")
print(f"Max Drawdown: {metrics.max_drawdown}%")
print(f"Win Rate: {metrics.win_rate}%")
```

## 相关文件清单

### 回测服务目录

```
services/backtesting/
├── __init__.py
└── backtest_service.py           # 回测服务实现
```

### 订单服务目录

```
services/order_service/
├── __init__.py
├── order_service.py              # 订单服务
├── order_backtest_service.py     # 回测订单服务
└── order_executor_lookup.py      # 订单执行器查找
```

### 组合服务目录

```
services/portfolios/
├── __init__.py
├── portfolio_service.py          # 组合服务
├── backtest_portfolio_service.py # 回测组合服务
├── portfolio_sync_service.py     # 组合同步服务
├── portfolio_configuration_service.py  # 组合配置服务
├── portfolio_snapshot_service.py # 组合快照服务
└── portfolio_provider_lookup.py  # 组合提供者查找
```

### 交易服务目录

```
services/trade_service/
├── __init__.py
├── trade_service.py              # 交易服务
├── trade_stop_loss_service.py    # 止损服务
└── trade_take_profit_service.py  # 止盈服务
```

### 指标服务目录

```
services/metrics/
├── __init__.py
├── cagr.py                       # CAGR 计算
├── calmar_ratio.py               # Calmar 比率
├── drawdowns.py                  # 回撤计算
├── exposure.py                   # 敞口计算
├── price_efficiency.py           # 价格效率
├── profit_factor.py              # 盈利因子
├── returns.py                    # 收益计算
├── sharpe_ratio.py               # Sharpe 比率
├── sortino_ratio.py              # Sortino 比率
├── standard_deviations.py        # 标准差
├── trades.py                     # 交易统计
├── volatility.py                 # 波动率
└── ...
```

### 数据提供者服务目录

```
services/data_providers/
├── __init__.py
└── data_provider_service.py      # 数据提供者服务
```

### 其他服务目录

```
services/
├── configuration_service.py      # 配置服务
├── market_credential_service.py  # 市场凭证服务
├── positions/
│   ├── __init__.py
│   ├── position_service.py       # 持仓服务
│   └── position_snapshot_service.py  # 持仓快照服务
├── repository_service.py         # 仓储服务
└── trade_order_evaluator/
    ├── __init__.py
    ├── trade_order_evaluator.py      # 订单评估器
    ├── backtest_trade_order_evaluator.py  # 回测订单评估器
    └── default_trade_order_evaluator.py   # 默认订单评估器
```

## 变更记录 (Changelog)

### 2025-12-24 10:25:42 - 模块文档创建
- 创建 services 模块详细文档
- 整理服务接口和指标函数说明
- 添加文件清单和常见问题
