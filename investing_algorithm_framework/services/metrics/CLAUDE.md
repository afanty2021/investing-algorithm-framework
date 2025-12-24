# Metrics Service 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [services](../) > **metrics**

## 模块职责

指标计算服务模块，提供 50+ 性能指标计算函数：

- **收益指标**: 总收益、CAGR、累计收益
- **风险指标**: 最大回撤、波动率、下行风险
- **风险调整收益**: Sharpe、Sortino、Calmar 比率
- **交易统计**: 胜率、盈亏比、平均交易时长
- **其他**: 敞口率、交易频率、价格效率

## 指标分类

### 收益指标

| 函数 | 描述 |
|------|------|
| `get_total_return` | 总收益率 |
| `get_cagr` | 复合年化增长率 |
| `get_cumulative_return` | 累计收益 |
| `get_yearly_returns` | 年度收益 |
| `get_monthly_returns` | 月度收益 |

### 风险指标

| 函数 | 描述 |
|------|------|
| `get_max_drawdown` | 最大回撤 |
| `get_max_drawdown_duration` | 最大回撤持续天数 |
| `get_annual_volatility` | 年化波动率 |
| `get_standard_deviation_returns` | 收益标准差 |
| `get_standard_deviation_downside_returns` | 下行收益标准差 |

### 风险调整收益

| 函数 | 描述 |
|------|------|
| `get_sharpe_ratio` | 夏普比率 |
| `get_sortino_ratio` | 索提诺比率 |
| `get_calmar_ratio` | 卡玛比率 |
| `get_treynor_ratio` | 特雷纳比率（待实现） |

### 交易统计

| 函数 | 描述 |
|------|------|
| `get_win_rate` | 胜率 |
| `get_win_loss_ratio` | 盈亏比 |
| `get_average_trade_duration` | 平均交易时长 |
| `get_profit_factor` | 盈利因子 |
| `get_best_trade` | 最佳交易 |
| `get_worst_trade` | 最差交易 |
| `get_number_of_trades` | 交易数量 |

### 其他指标

| 函数 | 描述 |
|------|------|
| `get_equity_curve` | 权益曲线 |
| `get_exposure_ratio` | 敞口率 |
| `get_trade_frequency` | 交易频率 |
| `get_price_efficiency_ratio` | 价格效率 |

## 使用示例

### 计算单个指标

```python
from investing_algorithm_framework.services.metrics import (
    get_sharpe_ratio,
    get_max_drawdown,
    get_win_rate
)

# 计算夏普比率
sharpe = get_sharpe_ratio(
    returns=returns_df,
    risk_free_rate=0.02
)

# 计算最大回撤
max_dd = get_max_drawdown(equity_curve)

# 计算胜率
win_rate = get_win_rate(trades)
```

### 生成完整指标

```python
from investing_algorithm_framework.services.metrics import create_backtest_metrics

# 生成完整回测指标
metrics = create_backtest_metrics(backtest)

print(f"总收益: {metrics.total_return}%")
print(f"夏普比率: {metrics.sharpe_ratio}")
print(f"最大回撤: {metrics.max_drawdown}%")
print(f"胜率: {metrics.win_rate}%")
```

## 指标公式

### Sharpe Ratio

```
Sharpe = (Rp - Rf) / σp

Rp = 投资组合收益率
Rf = 无风险利率
σp = 收益标准差
```

### Sortino Ratio

```
Sortino = (Rp - Rf) / σd

σd = 下行收益标准差
```

### Calmar Ratio

```
Calmar = CAGR / |Max Drawdown|
```

### Maximum Drawdown

```
Drawdown = (Peak - Trough) / Peak

从最高点跌到最低点的百分比
```

## 相关文件清单

```
services/metrics/
├── __init__.py                    # 模块导出
├── generate.py                    # 指标生成主函数
├── cagr.py                        # CAGR 计算
├── calmar_ratio.py                # Calmar 比率
├── drawdown.py                    # 回撤计算
├── equity_curve.py                # 权益曲线
├── exposure.py                    # 敞口计算
├── mean_daily_return.py           # 平均日收益
├── price_efficiency.py            # 价格效率
├── profit_factor.py               # 盈利因子
├── recovery.py                    # 恢复指标
├── returns.py                     # 收益计算
├── risk_free_rate.py              # 无风险利率
├── sharpe_ratio.py                # Sharpe 比率
├── sortino_ratio.py               # Sortino 比率
├── standard_deviation.py          # 标准差
├── trades.py                      # 交易统计
├── volatility.py                  # 波动率
└── win_rate.py                    # 胜率
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 metrics service 模块文档
