# Portfolios Service 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [services](../) > **portfolios**

## 模块职责

组合管理服务模块，提供组合生命周期管理能力：

- **PortfolioService**: 组合查询和管理
- **PortfolioSyncService**: 与交易所组合同步
- **PortfolioSnapshotService**: 组合快照管理
- **PortfolioConfigurationService**: 组合配置管理
- **BacktestPortfolioService**: 回测组合服务
- **PortfolioProviderLookup**: 组合提供者查找

## 核心服务

### PortfolioService

核心组合管理服务。

```python
class PortfolioService:
    def get_portfolio(self, portfolio_id: str) -> Portfolio
    def get_portfolios(self) -> List[Portfolio]
    def add_portfolio(self, portfolio: Portfolio)
    def update_portfolio(self, portfolio_id: str, **kwargs)
    def delete_portfolio(self, portfolio_id: str)
```

### PortfolioSyncService

与交易所同步组合信息。

```python
class PortfolioSyncService:
    def sync(self, market: str, trading_symbol: str) -> Portfolio
    def sync_all(self) -> List[Portfolio]
```

### PortfolioSnapshotService

组合快照管理。

```python
class PortfolioSnapshotService:
    def create_snapshot(self, portfolio: Portfolio) -> PortfolioSnapshot
    def get_snapshots(
        self,
        portfolio_id: str = None,
        start_date: datetime = None,
        end_date: datetime = None
    ) -> List[PortfolioSnapshot]
```

### PortfolioConfigurationService

组合配置管理。

```python
class PortfolioConfigurationService:
    def add(self, configuration: PortfolioConfiguration)
    def get(self, identifier: str) -> PortfolioConfiguration
    def get_all(self) -> List[PortfolioConfiguration]
    def delete(self, identifier: str)
```

## 使用示例

### 获取组合信息

```python
from investing_algorithm_framework import create_app

app = create_app()

# 获取主组合
portfolio = app.portfolio
print(f"总价值: {portfolio.total}")
print(f"已分配: {portfolio.allocated}")
print(f"未分配: {portfolio.unallocated}")

# 获取所有持仓
positions = portfolio.positions
for position in positions:
    print(f"{position.symbol}: {position.amount}")
```

### 同步组合

```python
# 从交易所同步
portfolio = portfolio_service.sync(
    market="binance",
    trading_symbol="USDT"
)
```

### 创建快照

```python
snapshot = portfolio_snapshot_service.create_snapshot(
    portfolio=portfolio
)
```

## 相关文件清单

```
services/portfolios/
├── __init__.py
├── portfolio_service.py                # 组合服务
├── portfolio_sync_service.py           # 同步服务
├── portfolio_snapshot_service.py       # 快照服务
├── portfolio_configuration_service.py  # 配置服务
├── backtest_portfolio_service.py       # 回测组合服务
└── portfolio_provider_lookup.py        # 提供者查找
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 portfolios service 模块文档
