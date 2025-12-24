# Backtesting 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [domain](../) > **backtesting**

## 模块职责

回测领域模型模块，定义框架的回测核心实体和指标计算：

- **回测模型**: `Backtest` - 完整回测容器，包含多次运行
- **回测运行**: `BacktestRun` - 单次回测运行记录
- **回测指标**: `BacktestMetrics` - 详细回测指标
- **汇总指标**: `BacktestSummaryMetrics` - 多次回测汇总统计
- **日期范围**: `BacktestDateRange` - 回测时间窗口
- **排列测试**: `BacktestPermutationTest` - 参数排列测试
- **回测合并**: `combine_backtests()` - 合并多次回测结果

## 入口与启动

### 模块导出

```python
from investing_algorithm_framework.domain.backtesting import (
    Backtest,
    BacktestRun,
    BacktestMetrics,
    BacktestSummaryMetrics,
    BacktestDateRange,
    BacktestPermutationTest,
    BacktestEvaluationFocus,
    combine_backtests,
    generate_backtest_summary_metrics
)
```

### 核心类层次

```
Backtest (回测容器)
    ├── backtest_runs: List[BacktestRun]    # 多次运行
    ├── backtest_summary: BacktestSummaryMetrics  # 汇总指标
    ├── backtest_permutation_tests: List[BacktestPermutationTest]
    └── metadata: Dict

BacktestRun (单次运行)
    ├── backtest_metrics: BacktestMetrics
    ├── trades: List[Trade]
    ├── orders: List[Order]
    ├── positions: List[Position]
    └── portfolio_snapshots: List[PortfolioSnapshot]
```

## 核心模型

### Backtest（回测容器）

完整的回测结果容器，可包含多次回测运行。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `id` | str | 唯一标识符（UUID） |
| `backtest_runs` | List[BacktestRun] | 多次回测运行列表 |
| `backtest_summary` | BacktestSummaryMetrics | 汇总统计指标 |
| `backtest_permutation_tests` | List[BacktestPermutationTest] | 参数排列测试 |
| `metadata` | Dict | 元数据配置 |
| `risk_free_rate` | float | 无风险利率（用于夏普比率等） |
| `strategy_ids` | List[int] | 关联策略 ID 列表 |
| `algorithm_id` | int | 关联算法 ID |

#### 核心方法

```python
# 获取所有回测运行
get_all_backtest_runs() -> List[BacktestRun]

# 按日期范围获取运行
get_backtest_run(date_range: BacktestDateRange) -> BacktestRun | None

# 获取所有回测指标
get_all_backtest_metrics() -> List[BacktestMetrics]

# 按日期范围获取指标
get_backtest_metrics(date_range: BacktestDateRange) -> BacktestMetrics | None

# 合并另一个回测
merge(other: Backtest) -> Backtest

# 从目录加载
@staticmethod open(directory_path: Path) -> Backtest

# 保存到目录
save(directory_path: Path) -> None
```

#### 文件结构

```
backtest_directory/
├── id.json                    # 回测 ID
├── metadata.json              # 元数据
├── risk_free_rate.json        # 无风险利率
├── summary.json               # 汇总指标
├── runs/                      # 回测运行目录
│   ├── backtest_BTC_20230101_20231231/
│   │   ├── metrics.json       # 指标
│   │   └── run.json           # 运行数据
│   └── ...
└── permutation_tests/         # 排列测试目录
    └── ...
```

### BacktestRun（回测运行）

单次回测运行记录，包含完整的交易历史和组合快照。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `backtest_start_date` | datetime | 回测开始时间 |
| `backtest_end_date` | datetime | 回测结束时间 |
| `backtest_date_range_name` | str | 日期范围名称 |
| `trading_symbol` | str | 主要交易标的 |
| `initial_unallocated` | float | 初始未分配资金 |
| `number_of_runs` | int | 运行次数 |
| `backtest_metrics` | BacktestMetrics | 回测指标 |
| `symbols` | List[str] | 涉及的交易标的 |
| `trades` | List[Trade] | 所有交易 |
| `orders` | List[Order] | 所有订单 |
| `positions` | List[Position] | 所有持仓 |
| `portfolio_snapshots` | List[PortfolioSnapshot] | 组合快照 |

#### 查询方法

```python
# 获取特定交易
get_trade(trade_id: str) -> Trade | None

# 按条件筛选交易
get_trades(
    target_symbol: str = None,
    trade_status: TradeStatus = None,
    opened_at: datetime = None,
    opened_at_lt: datetime = None,
    opened_at_lte: datetime = None,
    opened_at_gt: datetime = None,
    opened_at_gte: datetime = None,
    order_id: str = None
) -> List[Trade]

# 获取止损/止盈
get_stop_losses(trade_id: str = None, triggered: bool = None) -> List[TradeStopLoss]
get_take_profits(trade_id: str = None, triggered: bool = None) -> List[TradeTakeProfit]

# 获取组合快照
get_portfolio_snapshots(
    created_at_lt: datetime = None,
    created_at_lte: datetime = None,
    created_at_gt: datetime = None,
    created_at_gte: datetime = None
) -> List[PortfolioSnapshot]

# 获取订单
get_orders(
    target_symbol: str = None,
    order_side: str = None,
    order_status: OrderStatus = None,
    created_at: datetime = None,
    ...
) -> List[Order]
```

### BacktestMetrics（回测指标）

详细的回测性能指标，包含 50+ 项指标。

#### 收益指标

| 指标 | 描述 |
|------|------|
| `total_growth` | 总增长额 |
| `total_growth_percentage` | 总增长率 |
| `total_net_gain` | 净收益 |
| `total_net_gain_percentage` | 净收益率 |
| `cumulative_return` | 累计收益 |
| `cagr` | 复合年化增长率 (CAGR) |

#### 风险指标

| 指标 | 描述 |
|------|------|
| `max_drawdown` | 最大回撤（百分比） |
| `max_drawdown_absolute` | 最大回撤（绝对值） |
| `max_drawdown_duration` | 最大回撤持续天数 |
| `annual_volatility` | 年化波动率 |
| `drawdown_series` | 回撤序列 |

#### 风险调整收益

| 指标 | 描述 |
|------|------|
| `sharpe_ratio` | 夏普比率 |
| `rolling_sharpe_ratio` | 滚动夏普比率序列 |
| `sortino_ratio` | 索提诺比率 |
| `calmar_ratio` | 卡玛比率 |
| `profit_factor` | 盈利因子 |

#### 交易统计

| 指标 | 描述 |
|------|------|
| `number_of_trades` | 总交易数 |
| `number_of_trades_closed` | 已平仓交易数 |
| `number_of_trades_opened` | 开仓交易数 |
| `number_of_trades_open_at_end` | 期末未平仓数 |
| `number_of_positive_trades` | 盈利交易数 |
| `number_of_negative_trades` | 亏损交易数 |
| `win_rate` | 胜率 |
| `current_win_rate` | 当前胜率 |
| `win_loss_ratio` | 盈亏比 |
| `current_win_loss_ratio` | 当前盈亏比 |

#### 交易收益分析

| 指标 | 描述 |
|------|------|
| `average_trade_return` | 平均交易收益 |
| `average_trade_return_percentage` | 平均交易收益率 |
| `average_trade_gain` | 平均盈利 |
| `average_trade_gain_percentage` | 平均盈利率 |
| `average_trade_loss` | 平均亏损 |
| `average_trade_loss_percentage` | 平均亏损率 |
| `median_trade_return` | 交易收益中位数 |
| `best_trade` | 最佳交易 |
| `worst_trade` | 最差交易 |
| `average_trade_duration` | 平均持仓时长（小时） |

#### 时间序列

| 指标 | 描述 |
|------|------|
| `equity_curve` | 权益曲线 [(datetime, value), ...] |
| `cumulative_return_series` | 累计收益序列 |
| `monthly_returns` | 月度收益 |
| `yearly_returns` | 年度收益 |

#### 月度/年度统计

| 指标 | 描述 |
|------|------|
| `average_monthly_return` | 平均月度收益 |
| `average_monthly_return_winning_months` | 盈利月份平均收益 |
| `average_monthly_return_losing_months` | 亏损月份平均收益 |
| `best_month` | 最佳月份 |
| `worst_month` | 最差月份 |
| `best_year` | 最佳年份 |
| `worst_year` | 最差年份 |
| `percentage_winning_months` | 盈利月份占比 |
| `percentage_winning_years` | 盈利年份占比 |

#### 其他指标

| 指标 | 描述 |
|------|------|
| `trades_per_year` | 年均交易次数 |
| `trade_per_day` | 日均交易次数 |
| `exposure_ratio` | 平均仓位暴露率 |
| `cumulative_exposure` | 累计暴露率 |
| `average_trade_size` | 平均交易规模 |

### BacktestSummaryMetrics（汇总指标）

多次回测运行的汇总统计。

#### 属性

包含 `BacktestMetrics` 的核心指标，以及：

| 属性 | 描述 |
|------|------|
| `average_net_gain` | 平均净收益（多次回测） |
| `average_net_gain_percentage` | 平均净收益率 |
| `average_growth` | 平均增长 |
| `average_growth_percentage` | 平均增长率 |
| `average_loss` | 平均亏损 |
| `average_loss_percentage` | 平均亏损率 |

### BacktestDateRange（日期范围）

回测时间窗口定义。

#### 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `start_date` | datetime | 开始时间（必须整点，UTC） |
| `end_date` | datetime | 结束时间（必须整点，UTC） |
| `name` | str | 可选名称 |

#### 验证规则

```python
# 时间必须是整点（分钟、秒、微秒都为0）
# 时间必须是 UTC 时区
# 结束时间不能早于开始时间

# 有效示例
range1 = BacktestDateRange(
    start_date=datetime(2023, 1, 1, 0, 0, 0, tzinfo=timezone.utc),
    end_date=datetime(2023, 12, 31, 23, 0, 0, tzinfo=timezone.utc),
    name="2023 Full Year"
)

# 字符串解析
range2 = BacktestDateRange(
    start_date="2023-01-01 00:00:00",
    end_date="2023-12-31 23:00:00"
)
```

### BacktestPermutationTest（排列测试）

参数排列测试结果，用于验证策略稳健性。

### BacktestEvaluationFocus（评估焦点）

回测评估焦点配置。

## 工具函数

### combine_backtests()

合并多个回测结果。

```python
def combine_backtests(
    backtests: List[Backtest],
    backtest_date_ranges: List[BacktestDateRange]
) -> Backtest:
    """
    合并多个 Backtest 实例，生成汇总指标
    """
```

### generate_backtest_summary_metrics()

从多个回测指标生成汇总统计。

```python
def generate_backtest_summary_metrics(
    metrics_list: List[BacktestMetrics]
) -> BacktestSummaryMetrics:
    """
    从多个 BacktestMetrics 生成汇总统计
    """
```

## 持久化

### 保存回测

```python
backtest = Backtest(...)
backtest.save("/path/to/backtest_dir")
```

### 加载回测

```python
# 加载所有运行
backtest = Backtest.open("/path/to/backtest_dir")

# 加载特定日期范围
backtest = Backtest.open(
    "/path/to/backtest_dir",
    backtest_date_ranges=[range1, range2]
)
```

## 使用示例

### 创建回测结果

```python
from datetime import datetime, timezone
from investing_algorithm_framework.domain.backtesting import (
    Backtest,
    BacktestRun,
    BacktestMetrics,
    BacktestDateRange
)

# 创建日期范围
date_range = BacktestDateRange(
    start_date=datetime(2023, 1, 1, tzinfo=timezone.utc),
    end_date=datetime(2023, 12, 31, tzinfo=timezone.utc)
)

# 创建回测运行
run = BacktestRun(
    backtest_start_date=date_range.start_date,
    backtest_end_date=date_range.end_date,
    trading_symbol="BTC",
    trades=[...],
    orders=[...],
    backtest_metrics=BacktestMetrics(...)
)

# 创建回测容器
backtest = Backtest(
    backtest_runs=[run],
    risk_free_rate=0.03
)

# 保存
backtest.save("./backtest_results")
```

### 合并多次回测

```python
from investing_algorithm_framework.domain.backtesting import combine_backtests

# 合并多个时间段的回测
combined_backtest = combine_backtests(
    backtests=[backtest_q1, backtest_q2, backtest_q3, backtest_q4],
    backtest_date_ranges=[range_q1, range_q2, range_q3, range_q4]
)
```

### 查询交易数据

```python
# 获取所有 BTC 相关的已平仓交易
trades = backtest_run.get_trades(
    target_symbol="BTC",
    trade_status=TradeStatus.CLOSED
)

# 获取特定时间段的交易
recent_trades = backtest_run.get_trades(
    opened_at_gte=datetime(2023, 6, 1, tzinfo=timezone.utc)
)

# 获取触发的止损
stop_losses = backtest_run.get_stop_losses(triggered=True)
```

## 指标计算说明

### 夏普比率 (Sharpe Ratio)

```
Sharpe = (Rp - Rf) / σp

Rp = 投资组合收益率
Rf = 无风险利率
σp = 投资组合标准差
```

### 索提诺比率 (Sortino Ratio)

```
Sortino = (Rp - Rf) / σd

σd = 下行偏差标准差
```

### 卡玛比率 (Calmar Ratio)

```
Calmar = CAGR / |Max Drawdown|

CAGR = 复合年化增长率
```

### 盈利因子 (Profit Factor)

```
Profit Factor = Gross Profit / Gross Loss
```

## 常见问题 (FAQ)

### Q: Backtest 和 BacktestRun 有什么区别？

A:
- **Backtest**: 回测容器，可包含多次回测运行（如不同时间段、不同参数）
- **BacktestRun**: 单次回测运行，包含完整的交易历史

### Q: 日期为什么要整点？

A: 框架使用小时级别的时间粒度，确保数据对齐和计算一致性。

### Q: 如何实现参数排列测试？

A: 使用 `BacktestPermutationTest` 记录不同参数组合的回测结果，并添加到 `Backtest.backtest_permutation_tests`。

### Q: equity_curve 的数据格式是什么？

A: `List[Tuple[datetime, float]]`，每个元组包含时间戳和对应的组合价值。

## 相关文件清单

```
domain/backtesting/
├── __init__.py                      # 模块导出
├── backtest.py                      # 回测容器模型
├── backtest_run.py                  # 回测运行模型
├── backtest_metrics.py              # 回测指标模型
├── backtest_summary_metrics.py      # 汇总指标模型
├── backtest_date_range.py           # 日期范围值对象
├── backtest_permutation_test.py     # 排列测试模型
├── backtest_evaluation_focuss.py    # 评估焦点配置
└── combine_backtests.py             # 回测合并工具
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 backtesting 模块详细文档
- 整理核心模型和指标说明
- 添加使用示例和常见问题
