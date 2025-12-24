# App 模块

[根目录](../../CLAUDE.md) > [investing_algorithm_framework](../) > **app**

## 模块职责

应用核心模块，提供框架的核心功能：

- **App 类**: 应用程序主入口，管理策略、市场和数据源
- **TradingStrategy 类**: 交易策略基类，所有用户策略的父类
- **Algorithm 类**: 算法执行引擎，处理交易逻辑
- **回测引擎**: 事件驱动和向量化回测
- **报告生成**: HTML 和 ASCII 格式的回测报告
- **Web API**: REST 接口用于监控和管理

## 入口与启动

### 主要入口点

| 文件 | 描述 |
|------|------|
| `app.py` | App 主类，应用程序核心 |
| `strategy.py` | TradingStrategy 基类 |
| `algorithm/algorithm.py` | Algorithm 执行引擎 |
| `web/create_app.py` | Flask 应用创建 |
| `context.py` | 策略执行上下文 |

### 创建应用

```python
from investing_algorithm_framework import create_app

# 创建默认应用
app = create_app()

# 创建 Web 应用
app = create_app(web=True, name="my-bot")

# 添加配置
app = create_app(config={"KEY": "value"})
```

## 对外接口

### 核心类

#### App 类

```python
class App:
    def add_strategy(self, strategy: TradingStrategy)
    def add_market(self, market: str, trading_symbol: str)
    def add_data_provider(self, data_provider: DataProvider)
    def run(self)
    def run_backtest(self, backtest_date_range, initial_amount)
    def run_vectorized_backtest(self, backtest_date_range, initial_amount)
```

#### TradingStrategy 类

```python
class TradingStrategy:
    time_unit: TimeUnit        # 时间单位（HOUR, DAY, WEEK, MONTH）
    interval: int              # 运行间隔
    symbols: List[str]         # 交易标的
    data_sources: List[DataSource]  # 数据源

    def generate_buy_signals(self, data: Dict[str, Any]) -> Dict[str, pd.Series]
    def generate_sell_signals(self, data: Dict[str, Any]) -> Dict[str, pd.Series]
```

#### Algorithm 类

算法执行器，提供策略执行期间可用的方法：

```python
class Algorithm:
    # 订单管理
    def create_limit_order(...)
    def get_order(self, order_id)
    def get_pending_orders(...)

    # 持仓管理
    def get_position(self, symbol)
    def get_positions(...)
    def has_position(symbol)

    # 交易管理
    def get_trades(...)
    def get_open_trades(...)
    def get_closed_trades(...)

    # 组合管理
    def get_portfolio(self)
    def get_allocated(self)
    def get_unallocated(self)
```

### Web API 控制器

| 控制器 | 路由 | 功能 |
|--------|------|------|
| `portfolio.py` | `/api/portfolio` | 获取组合信息 |
| `orders.py` | `/api/orders` | 订单管理 |
| `positions.py` | `/api/positions` | 持仓管理 |

## 关键依赖与配置

### 依赖项

- `Flask`: Web API 框架
- `Flask-Cors`: 跨域支持
- `dependency-injector`: 依赖注入容器
- `schedule`: 任务调度

### 配置项

```python
# 应用配置
APP_MODE                    # 运行模式 (TRADING/BACKTESTING)
APPLICATION_DIRECTORY       # 应用目录路径
RESOURCE_DIRECTORY          # 资源目录路径

# 数据库配置
DATABASE_DIRECTORY_PATH     # 数据库目录
DATABASE_NAME               # 数据库名称
SQLALCHEMY_DATABASE_URI     # 数据库连接 URI

# 回测配置
BACKTESTING_FLAG            # 回测标志
BACKTESTING_START_DATE      # 回测开始日期
BACKTESTING_END_DATE        # 回测结束日期
BACKTESTING_INITIAL_AMOUNT  # 初始资金
```

## 数据模型

### 核心模型

| 模型 | 描述 |
|------|------|
| `StrategyProfile` | 策略配置文件 |
| `Event` | 系统事件 |
| `AppMode` | 应用模式枚举 |

### 报告组件

| 组件 | 描述 |
|------|------|
| `BacktestReport` | 回测报告生成器 |
| `Chart` | 图表生成（权益曲线、回撤、热力图等） |
| `Table` | 表格生成（关键指标、交易记录等） |

## 测试与质量

### 测试目录结构

```
tests/app/
├── algorithm/           # 算法测试
├── analysis/            # 分析功能测试
├── backtesting/         # 回测测试
├── reporting/           # 报告测试
├── web/                 # Web API 测试
└── test_*.py            # 应用测试
```

### 测试覆盖

- 算法创建和执行
- 订单管理
- 持仓管理
- 回测运行
- 报告生成

## 常见问题 (FAQ)

### Q: 如何创建自定义策略？

A: 继承 `TradingStrategy` 并实现信号生成方法：

```python
class MyStrategy(TradingStrategy):
    time_unit = TimeUnit.HOUR
    interval = 1
    symbols = ["BTC"]

    def generate_buy_signals(self, data):
        return {"BTC": pd.Series([True, False, ...])}

    def generate_sell_signals(self, data):
        return {"BTC": pd.Series([False, True, ...])}
```

### Q: 事件驱动和向量化回测有什么区别？

A:
- **事件驱动**: 逐条数据处理，精确模拟真实交易，较慢
- **向量化**: 批量处理数据，适合快速原型开发，较快

### Q: 如何在策略中访问组合信息？

A: 通过 `context` 参数或 `self.context`：

```python
def run_strategy(self, context, data):
    portfolio = context.get_portfolio()
    balance = context.get_unallocated()
```

## 相关文件清单

### 核心文件

```
app/
├── __init__.py
├── app.py                    # App 主类
├── strategy.py               # TradingStrategy 基类
├── context.py                # 策略上下文
├── eventloop.py              # 事件循环
├── task.py                   # 任务定义
├── app_hook.py               # 应用钩子
├── create_app.py             # 应用工厂（Web 模式）
└── algorithm/
    ├── __init__.py
    ├── algorithm.py          # Algorithm 类
    └── algorithm_factory.py  # 算法工厂
```

### 分析模块

```
app/analysis/
├── __init__.py
├── backtest_data_ranges.py   # 回测日期范围选择
├── backtest_utils.py         # 回测工具
├── permutation.py            # 排列测试
└── ranking.py                # 结果排名
```

### 报告模块

```
app/reporting/
├── __init__.py
├── backtest_report.py        # 回测报告生成器
├── ascii.py                  # ASCII 格式输出
├── generate.py               # 报告生成工具
├── charts/                   # 图表组件
│   ├── equity_curve.py
│   ├── equity_curve_drawdown.py
│   ├── monthly_returns_heatmap.py
│   └── ...
└── tables/                   # 表格组件
    ├── key_metrics_table.py
    ├── trade_metrics_table.py
    └── ...
```

### Web 模块

```
app/web/
├── __init__.py
├── create_app.py             # Flask 应用创建
├── error_handler.py          # 错误处理
├── responses.py              # API 响应
├── setup_cors.py             # CORS 配置
├── run_strategies.py         # 策略运行端点
├── controllers/              # API 控制器
│   ├── portfolio.py
│   ├── orders.py
│   └── positions.py
└── schemas/                  # 请求/响应模式
    ├── portfolio.py
    ├── order.py
    └── position.py
```

### 无状态模块

```
app/stateless/
├── __init__.py
├── exception_handler.py      # 异常处理
└── action_handlers/          # 动作处理器
    ├── check_online_handler.py
    ├── run_strategy_handler.py
    └── action_handler_strategy.py
```

## 变更记录 (Changelog)

### 2025-12-24 10:25:42 - 模块文档创建
- 创建 app 模块详细文档
- 整理核心类和接口说明
- 添加文件清单和常见问题
