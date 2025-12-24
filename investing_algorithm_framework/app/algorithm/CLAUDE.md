# Algorithm 模块

[根目录](../../../../CLAUDE.md) > [investing_algorithm_framework](../../) > [app](../) > **algorithm**

## 模块职责

算法执行引擎模块，处理策略执行期间的交易逻辑：

- **Algorithm**: 策略执行期间的 API 接口
- **AlgorithmFactory**: 算法工厂，创建算法实例
- **订单管理**: 创建和查询订单
- **持仓管理**: 查询持仓信息
- **交易管理**: 管理开仓/平仓交易
- **组合管理**: 访问组合信息

## 核心类

### Algorithm

策略执行期间可用的 API 接口。

#### 属性

| 属性 | 描述 |
|------|------|
| `name` | 算法名称 |
| `description` | 算法描述 |
| `strategies` | 关联的策略列表 |
| `data_sources` | 数据源列表 |
| `context` | 执行上下文 |
| `metadata` | 元数据 |

#### 方法

##### 组合管理

```python
# 获取组合
portfolio = algorithm.get_portfolio()

# 获取资金
unallocated = algorithm.get_unallocated()
allocated = algorithm.get_allocated()
total = algorithm.get_total()
```

##### 持仓管理

```python
# 检查是否有持仓
has_position = algorithm.has_position("BTC")

# 获取持仓
position = algorithm.get_position("BTC")

# 获取所有持仓
positions = algorithm.get_positions()

# 获取特定交易对的持仓
positions = algorithm.get_positions_for_symbol("BTC")
```

##### 订单管理

```python
# 创建限价单
order = algorithm.create_limit_order(
    trading_symbol="USDT",
    target_symbol="BTC",
    order_side="buy",
    amount=0.1,
    price=50000
)

# 获取订单
order = algorithm.get_order(order_id)

# 获取待处理订单
pending_orders = algorithm.get_pending_orders()

# 取消订单
algorithm.cancel_order(order_id)
```

##### 交易管理

```python
# 获取交易
trade = algorithm.get_trade(trade_id)

# 获取持仓相关的交易
trades = algorithm.get_trades_for_position("BTC")

# 获取开仓交易
open_trades = algorithm.get_open_trades()

# 获取已平仓交易
closed_trades = algorithm.get_closed_trades()

# 获取所有交易
all_trades = algorithm.get_trades()
```

### AlgorithmFactory

算法工厂，创建和管理算法实例。

```python
class AlgorithmFactory:
    @staticmethod
    def create_algorithm(
        name: str,
        strategy=None,
        strategies=None,
        data_sources=None
    ) -> Algorithm

    @staticmethod
    def create_algorithm_from_profile(
        profile_path: str
    ) -> Algorithm
```

## 使用示例

### 创建算法

```python
from investing_algorithm_framework import AlgorithmFactory

# 从策略创建
algorithm = AlgorithmFactory.create_algorithm(
    name="my_algorithm",
    strategy=MyStrategy(),
    data_sources=[...]
)
```

### 策略中使用 Algorithm

```python
class MyStrategy(TradingStrategy):
    def run_strategy(self, context, data):
        # context 是 Algorithm 实例

        # 检查持仓
        if not context.has_position("BTC"):
            # 获取可用资金
            usdt = context.get_unallocated()

            # 计算买入数量
            amount = usdt * 0.1 / data["BTC"]["Close"].iloc[-1]

            # 创建买入订单
            context.create_limit_order(
                trading_symbol="USDT",
                target_symbol="BTC",
                order_side="buy",
                amount=amount,
                price=data["BTC"]["Close"].iloc[-1]
            )
```

## 常见问题 (FAQ)

### Q: Algorithm 和 App 有什么区别？

A:
- **App**: 应用程序主入口，管理整个生命周期
- **Algorithm**: 策略执行期间的 API，提供交易方法

### Q: 如何获取当前价格？

A: 通过 data 参数：

```python
def run_strategy(self, context, data):
    current_price = data["BTC"]["Close"].iloc[-1]
```

## 相关文件清单

```
app/algorithm/
├── __init__.py
├── algorithm.py             # Algorithm 类（~200 行）
└── algorithm_factory.py      # 算法工厂（~100 行）
```

## 变更记录 (Changelog)

### 2025-12-24 14:30:00 - 模块文档创建
- 创建 algorithm 模块文档
