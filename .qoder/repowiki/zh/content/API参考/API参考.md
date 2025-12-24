# API参考

<cite>
**本文档中引用的文件**   
- [app.py](file://investing_algorithm_framework/app/app.py)
- [strategy.py](file://investing_algorithm_framework/app/strategy.py)
- [create_app.py](file://investing_algorithm_framework/create_app.py)
- [config.py](file://investing_algorithm_framework/domain/config.py)
- [data_provider.py](file://investing_algorithm_framework/domain/data_provider.py)
- [constants.py](file://investing_algorithm_framework/domain/constants.py)
- [simple_trading_bot_example.py](file://examples/simple_trading_bot_example.py)
- [algorithm.py](file://investing_algorithm_framework/app/algorithm/algorithm.py)
</cite>

## 目录
1. [App类核心方法](#app类核心方法)
2. [策略基类接口](#策略基类接口)
3. [配置系统](#配置系统)
4. [数据提供者](#数据提供者)
5. [使用示例](#使用示例)

## App类核心方法

App类是投资算法框架的核心，用于初始化和运行交易机器人。它提供了多种方法来配置应用程序、添加策略、数据提供者和任务。

### add_strategy 和 add_strategies
`add_strategy` 方法用于向应用程序添加一个交易策略。策略必须是 `TradingStrategy` 类或其子类的实例。

**参数**:
- `strategy`: `TradingStrategy` 实例
- `throw_exception`: 布尔值，指定当提供的策略不符合应用程序期望时是否抛出异常

**返回值**: 无

`add_strategies` 方法用于向应用程序添加多个交易策略。

**参数**:
- `strategies`: `TradingStrategy` 实例列表
- `throw_exception`: 布尔值，指定当提供的策略不符合应用程序期望时是否抛出异常

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1767-L1818)

### add_data_provider
`add_data_provider` 方法用于向应用程序添加数据提供者。数据提供者用于获取和准备交易算法所需的数据。

**参数**:
- `data_provider`: `DataProvider` 实例
- `priority`: 整数，指定数据提供者的优先级。数字越小，优先级越高

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1617-L1629)

### run_backtest
`run_backtest` 方法用于运行算法的回测。它接受一个回测日期范围、初始金额、算法、策略、快照间隔、风险自由率和元数据作为参数。

**参数**:
- `backtest_date_range`: `BacktestDateRange` 实例，指定回测的日期范围
- `name`: 字符串，回测的名称
- `initial_amount`: 浮点数，回测开始时的初始金额
- `algorithm`: `Algorithm` 实例，要回测的算法
- `strategy`: `TradingStrategy` 实例，要回测的策略
- `strategies`: `TradingStrategy` 实例列表，要回测的策略列表
- `snapshot_interval`: `SnapshotInterval` 实例，指定回测期间投资组合快照的频率
- `risk_free_rate`: 浮点数，用于计算夏普比率和其他性能指标的风险自由率
- `metadata`: 字典，附加到回测报告的元数据

**返回值**: `Backtest` 实例

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1294-L1431)

### run_backtests
`run_backtests` 方法用于运行多个算法在多个日期范围上的回测。它接受一个算法列表、日期范围列表、初始金额、快照间隔和风险自由率作为参数。

**参数**:
- `algorithms`: `Algorithm` 实例列表，要回测的算法
- `backtest_date_ranges`: `BacktestDateRange` 实例列表，回测的日期范围
- `initial_amount`: 浮点数，回测开始时的初始金额
- `snapshot_interval`: `SnapshotInterval` 实例，指定回测期间投资组合快照的频率
- `risk_free_rate`: 浮点数，用于计算夏普比率和其他性能指标的风险自由率

**返回值**: `Backtest` 实例列表

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1219-L1293)

### run_vector_backtest
`run_vector_backtest` 方法用于运行策略的向量化回测。它接受一个回测日期范围、策略、快照间隔、元数据、风险自由率、跳过数据源初始化标志、显示数据初始化进度标志、初始金额、市场、交易符号和继续出错标志作为参数。

**参数**:
- `backtest_date_range`: `BacktestDateRange` 实例，指定回测的日期范围
- `strategy`: `TradingStrategy` 实例，要回测的策略
- `snapshot_interval`: `SnapshotInterval` 实例，指定回测期间投资组合快照的频率
- `metadata`: 字典，附加到回测报告的元数据
- `risk_free_rate`: 浮点数，用于计算夏普比率和其他性能指标的风险自由率
- `skip_data_sources_initialization`: 布尔值，指定是否跳过数据源的初始化
- `show_data_initialization_progress`: 布尔值，指定是否在初始化数据源时显示进度条
- `initial_amount`: 浮点数，回测开始时的初始金额
- `market`: 字符串，用于回测的市场
- `trading_symbol`: 字符串，用于回测的交易符号
- `continue_on_error`: 布尔值，指定如果一个回测中发生错误是否继续运行其他回测

**返回值**: `Backtest` 实例

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1080-L1218)

### run_vector_backtests
`run_vector_backtests` 方法用于运行一组策略的向量化回测。它接受初始金额、策略列表、回测日期范围、回测日期范围列表、快照间隔、风险自由率、跳过数据源初始化标志、显示进度标志、市场、交易符号和继续出错标志作为参数。

**参数**:
- `initial_amount`: 浮点数，回测开始时的初始金额
- `strategies`: `TradingStrategy` 实例列表，要回测的策略
- `backtest_date_range`: `BacktestDateRange` 实例，指定回测的日期范围
- `backtest_date_ranges`: `BacktestDateRange` 实例列表，指定回测的日期范围
- `snapshot_interval`: `SnapshotInterval` 实例，指定回测期间投资组合快照的频率
- `risk_free_rate`: 浮点数，用于计算夏普比率和其他性能指标的风险自由率
- `skip_data_sources_initialization`: 布尔值，指定是否跳过数据源的初始化
- `show_progress`: 布尔值，指定是否显示进度条
- `market`: 字符串，用于回测的市场
- `trading_symbol`: 字符串，用于回测的交易符号
- `continue_on_error`: 布尔值，指定如果一个回测中发生错误是否继续运行其他回测

**返回值**: `Backtest` 实例列表

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L916-L1078)

### run_permutation_test
`run_permutation_test` 方法用于运行给定策略在指定日期范围内的排列测试。此测试用于通过将其性能与市场数据的随机排列集进行比较来确定策略性能的统计显著性。

**参数**:
- `strategy`: `TradingStrategy` 实例，要测试的策略
- `backtest_date_range`: `BacktestDateRange` 实例，回测的日期范围
- `number_of_permutations`: 整数，要运行的排列数。默认为100
- `initial_amount`: 浮点数，回测的初始金额。默认为1000.0
- `risk_free_rate`: 浮点数，用于回测指标的风险自由率。如果未提供，将尝试从美国财政部网站获取
- `market`: 字符串，用于回测的市场。如果未提供，将使用找到的第一个投资组合配置
- `trading_symbol`: 字符串，用于回测的交易符号。如果未提供，将使用投资组合配置中找到的第一个交易符号

**返回值**: `BacktestPermutationTest` 实例

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1433-L1599)

### add_portfolio_configuration
`add_portfolio_configuration` 方法用于向应用程序添加投资组合配置。投资组合配置应为 `PortfolioConfiguration` 实例。

**参数**:
- `portfolio_configuration`: `PortfolioConfiguration` 实例

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L687-L701)

### task
`task` 方法用于向应用程序添加任务。任务可以是函数或类。

**参数**:
- `function`: 装饰的函数
- `time_unit`: `TimeUnit` 实例，指定任务的时间单位
- `interval`: 整数，指定任务的间隔

**返回值**: `Task` 或 `Function`

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L702-L739)

### add_task 和 add_tasks
`add_task` 方法用于向应用程序添加一个任务。任务应为 `Task` 实例。

**参数**:
- `task`: `Task` 实例

**返回值**: 无

`add_tasks` 方法用于向应用程序添加多个任务。

**参数**:
- `tasks`: `Task` 实例列表

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L741-L765)

### on_initialize
`on_initialize` 方法用于添加在应用程序初始化时运行的钩子。钩子应为 `AppHook` 实例。

**参数**:
- `app_hook`: `AppHook` 实例

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1648-L1669)

### on_strategy_run
`on_strategy_run` 方法用于添加在策略运行时运行的钩子。钩子应为 `AppHook` 实例。

**参数**:
- `app_hook`: `AppHook` 实例

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1671-L1687)

### after_initialize
`after_initialize` 方法用于添加在应用程序初始化后运行的钩子。钩子应为 `AppHook` 实例。

**参数**:
- `app_hook`: `AppHook` 实例

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1688-L1698)

### strategy
`strategy` 方法用于注册一个交易策略。此装饰器可用于定义交易策略函数并将其注册到应用程序中。

**参数**:
- `function`: 装饰的函数
- `time_unit`: `TimeUnit` 实例，指定策略的时间单位
- `interval`: 整数，指定策略的间隔
- `data_sources`: `DataSource` 实例列表，策略使用的数据源

**返回值**: `TradingStrategy` 实例

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1699-L1747)

### add_market_credential
`add_market_credential` 方法用于向应用程序添加市场凭证。市场凭证应为 `MarketCredential` 实例。

**参数**:
- `market_credential`: `MarketCredential` 实例

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1630-L1647)

### add_portfolio_provider
`add_portfolio_provider` 方法用于向应用程序添加投资组合提供者。投资组合提供者应为 `PortfolioProvider` 实例。

**参数**:
- `portfolio_provider`: `PortfolioProvider` 实例

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1915-L1940)

### get_portfolio_providers
`get_portfolio_providers` 方法用于获取应用程序中的所有投资组合提供者。

**返回值**: `PortfolioProvider` 实例列表

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1941-L1950)

### initialize_order_executors
`initialize_order_executors` 方法用于初始化订单执行器。如果应用程序在回测模式下运行，所有订单执行器将被移除，并添加一个 `BacktestOrderExecutor`。否则，将添加默认的 `CCXTOrderExecutor`。

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1952-L1981)

### initialize_portfolios
`initialize_portfolios` 方法用于初始化投资组合。如果应用程序在回测模式下运行，将使用配置中指定的初始金额创建投资组合。否则，将检查是否已存在投资组合。

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L1982-L2119)

### initialize_backtest_portfolios
`initialize_backtest_portfolios` 方法用于初始化回测投资组合。它将为应用程序中配置的每个市场创建一个默认的投资组合提供者。

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L2119-L2153)

### initialize_portfolio_providers
`initialize_portfolio_providers` 方法用于初始化默认的投资组合提供者。它将为应用程序中配置的每个市场创建一个默认的投资组合提供者。

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L2154-L2181)

### get_run_history
`get_run_history` 方法用于获取应用程序的运行历史。它将返回所有已注册策略和任务的运行时间表历史。

**返回值**: 字典，应用程序的运行历史

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L2182-L2192)

### has_run
`has_run` 方法用于检查工作程序是否在应用程序中运行过。它将检查 `worker_id` 是否存在于应用程序的运行历史中。

**参数**:
- `worker_id`: 字符串，工作程序的ID

**返回值**: 布尔值，如果工作程序已运行则为 `True`，否则为 `False`

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L2193-L2208)

### get_algorithm
`get_algorithm` 方法用于获取当前在应用程序中运行的算法。

**返回值**: `Algorithm` 实例

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L2209-L2225)

### cleanup_backtest_resources
`cleanup_backtest_resources` 方法用于清理回测数据库并删除SQLAlchemy模型/表。

**返回值**: 无

**Section sources**
- [app.py](file://investing_algorithm_framework/app/app.py#L2226-L2237)

## 策略基类接口

`TradingStrategy` 是所有交易策略的基类。交易策略是一组定义何时买卖资产的规则。

### 属性
- `time_unit`: `TimeUnit` - 定义策略运行时间单位的策略时间单位，例如小时、天、周、月
- `interval`: 整数 - 定义策略在时间单位内运行频率的策略间隔，例如每5小时、每2天、每3周、每4个月
- `worker_id` (可选): 字符串 - 工作程序的ID
- `strategy_id` (可选): 字符串 - 策略的ID
- `decorated` (可选): 函数 - 装饰的函数
- `data_sources` (`List[DataSource]` 可选): 用于策略的数据源列表。数据源将用于识别将被调用以获取数据并传递给策略的提供者
- `metadata` (可选): 字典 - 包含有关策略的元数据的字典。这可用于存储有关策略的附加信息，例如其作者、版本、描述、参数等

### 生命周期方法
`TradingStrategy` 类定义了多个生命周期方法，可以在策略执行的不同阶段重写。

#### on_trade_created
`on_trade_created` 方法在创建交易时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L435-L442)

#### on_trade_opened
`on_trade_opened` 方法在交易打开时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L444-L445)

#### on_trade_updated
`on_trade_updated` 方法在交易更新时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L438-L439)

#### on_trade_closed
`on_trade_closed` 方法在交易关闭时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L435-L436)

#### on_trade_stop_loss_created
`on_trade_stop_loss_created` 方法在交易止损创建时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L471-L472)

#### on_trade_trailing_stop_loss_created
`on_trade_trailing_stop_loss_created` 方法在交易追踪止损创建时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L474-L477)

#### on_trade_take_profit_created
`on_trade_take_profit_created` 方法在交易止盈创建时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L478-L480)

#### on_trade_stop_loss_updated
`on_trade_stop_loss_updated` 方法在交易止损更新时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L460-L461)

#### on_trade_trailing_stop_loss_updated
`on_trade_trailing_stop_loss_updated` 方法在交易追踪止损更新时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L463-L466)

#### on_trade_take_profit_updated
`on_trade_take_profit_updated` 方法在交易止盈更新时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L468-L469)

#### on_trade_stop_loss_triggered
`on_trade_stop_loss_triggered` 方法在交易止损触发时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L447-L448)

#### on_trade_trailing_stop_loss_triggered
`on_trade_trailing_stop_loss_triggered` 方法在交易追踪止损触发时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L450-L453)

#### on_trade_take_profit_triggered
`on_trade_take_profit_triggered` 方法在交易止盈触发时调用。

**参数**:
- `context`: `Context` 实例
- `trade`: `Trade` 实例

**返回值**: 无

**Section sources**
- [strategy.py](file://investing_algorithm_framework/app/strategy.py#L455-L458)

## 配置系统

配置系统用于管理应用程序的配置。配置可以是环境变量、文件或字典。

### Environment
`Environment` 类定义了应用程序可以运行的不同环境。

**值**:
- `DEV`: 开发环境
- `PROD`: 生产环境
- `TEST`: 测试环境
- `BACKTEST`: 回测环境

**Section sources**
- [config.py](file://investing_algorithm_framework/domain/config.py#L9-L16)

### TimeUnit
`TimeUnit` 类定义了时间单位，例如秒、分钟、小时或天。

**值**:
- `SECOND`: 秒
- `MINUTE`: 分钟
- `HOUR`: 小时
- `DAY`: 天

**Section sources**
- [time_unit.py](file://investing_algorithm_framework/domain/models/time_unit.py#L14-L17)

### AppMode
`AppMode` 类定义了应用程序可以运行的不同模式。

**值**:
- `DEFAULT`: 默认模式
- `WEB`: Web模式

**Section sources**
- [config.py](file://investing_algorithm_framework/domain/config.py#L9-L16)

### ConfigurationService
`ConfigurationService` 类用于管理应用程序的配置。

**方法**:
- `initialize`: 初始化配置
- `config`: 获取配置
- `get_config`: 获取配置
- `get_flask_config`: 获取Flask配置
- `add_value`: 添加配置值
- `add_dict`: 添加配置字典
- `remove_value`: 删除配置值

**Section sources**
- [configuration_service.py](file://investing_algorithm_framework/services/configuration_service.py#L50-L82)

## 数据提供者

`DataProvider` 是数据提供者的抽象基类。`DataProvider` 负责获取和准备交易算法所需的数据。

### 属性
- `data_type`: `DataType` - 要获取的数据类型（例如，OHLCV、TICKER、CUSTOM_DATA）
- `symbol` (可选): 字符串列表 - 数据提供者可以提供数据的支持符号列表
- `priority`: 整数 - 数据提供者的优先级。数字越小，优先级越高
- `time_frame` (可选): 字符串 - 数据的时间框架。例如："1m"、"5m"、"1h"、"1d"
- `window_size` (可选): 整数 - 数据的窗口大小。例如：100、200、500
- `storage_path` (可选): 字符串 - 数据存储位置的路径

### 方法
- `has_data`: 检查数据提供者是否有给定参数的数据
- `get_data`: 获取给定符号和日期范围的数据
- `prepare_backtest_data`: 准备给定符号和日期范围的回测数据
- `get_backtest_data`: 获取给定数据源的回测数据
- `copy`: 返回基于给定数据源的数据提供者实例的副本
- `get_number_of_data_points`: 返回给定开始和结束日期之间可用的数据点数量
- `get_missing_data_dates`: 返回给定开始和结束日期之间缺少数据的日期列表
- `get_data_source_file_path`: 返回给定数据源的文件路径（如果适用）

**Section sources**
- [data_provider.py](file://investing_algorithm_framework/domain/data_provider.py#L12-L335)

## 使用示例

以下是一个使用 `investing_algorithm_framework` 创建简单交易机器人的示例。

```python
from typing import Dict, Any
from datetime import datetime, timezone

import pandas as pd
from pyindicators import ema, rsi, crossover, crossunder

from investing_algorithm_framework import TradingStrategy, DataSource, \
    TimeUnit, DataType, PositionSize, create_app, RESOURCE_DIRECTORY, \
    BacktestDateRange, BacktestReport


class RSIEMACrossoverStrategy(TradingStrategy):
    time_unit = TimeUnit.HOUR
    interval = 2
    symbols = ["BTC"]
    position_sizes = [
        PositionSize(
            symbol="BTC", percentage_of_portfolio=20.0
        ),
        PositionSize(
            symbol="ETH", percentage_of_portfolio=20.0
        )
    ]

    def __init__(
        self,
        time_unit: TimeUnit,
        interval: int,
        market: str,
        rsi_time_frame: str,
        rsi_period: int,
        rsi_overbought_threshold,
        rsi_oversold_threshold,
        ema_time_frame,
        ema_short_period,
        ema_long_period,
        ema_cross_lookback_window: int = 10
    ):
        self.rsi_time_frame = rsi_time_frame
        self.rsi_period = rsi_period
        self.rsi_result_column = f"rsi_{self.rsi_period}"
        self.rsi_overbought_threshold = rsi_overbought_threshold
        self.rsi_oversold_threshold = rsi_oversold_threshold
        self.ema_time_frame = ema_time_frame
        self.ema_short_result_column = f"ema_{ema_short_period}"
        self.ema_long_result_column = f"ema_{ema_long_period}"
        self.ema_crossunder_result_column = "ema_crossunder"
        self.ema_crossover_result_column = "ema_crossover"
        self.ema_short_period = ema_short_period
        self.ema_long_period = ema_long_period
        self.ema_cross_lookback_window = ema_cross_lookback_window
        data_sources = []

        for symbol in self.symbols:
            full_symbol = f"{symbol}/EUR"
            data_sources.append(
                DataSource(
                    identifier=f"{symbol}_rsi_data",
                    data_type=DataType.OHLCV,
                    time_frame=self.rsi_time_frame,
                    market=market,
                    symbol=full_symbol,
                    pandas=True,
                    window_size=800
                )
            )
            data_sources.append(
                DataSource(
                    identifier=f"{symbol}_ema_data",
                    data_type=DataType.OHLCV,
                    time_frame=self.ema_time_frame,
                    market=market,
                    symbol=full_symbol,
                    pandas=True,
                    window_size=800
                )
            )

        super().__init__(
            data_sources=data_sources, time_unit=time_unit, interval=interval
        )

        self.buy_signal_dates = {}
        self.sell_signal_dates = {}

        for symbol in self.symbols:
            self.buy_signal_dates[symbol] = []
            self.sell_signal_dates[symbol] = []

    def _prepare_indicators(
        self,
        rsi_data,
        ema_data
    ):
        ema_data = ema(
            ema_data,
            period=self.ema_short_period,
            source_column="Close",
            result_column=self.ema_short_result_column
        )
        ema_data = ema(
            ema_data,
            period=self.ema_long_period,
            source_column="Close",
            result_column=self.ema_long_result_column
        )
        # Detect crossover (short EMA crosses above long EMA)
        ema_data = crossover(
            ema_data,
            first_column=self.ema_short_result_column,
            second_column=self.ema_long_result_column,
            result_column=self.ema_crossover_result_column
        )
        # Detect crossunder (short EMA crosses below long EMA)
        ema_data = crossunder(
            ema_data,
            first_column=self.ema_short_result_column,
            second_column=self.ema_long_result_column,
            result_column=self.ema_crossunder_result_column
        )
        rsi_data = rsi(
            rsi_data,
            period=self.rsi_period,
            source_column="Close",
            result_column=self.rsi_result_column
        )

        return ema_data, rsi_data

    def generate_buy_signals(self, data: Dict[str, Any]) -> Dict[str, pd.Series]:
        """
        Generate buy signals based on the moving average crossover.

        data (Dict[str, Any]): Dictionary containing all the data for
            the strategy data sources.

        Returns:
            Dict[str, pd.Series]: A dictionary where keys are symbols and values
                are pandas Series indicating buy signals (True/False).
        """

        signals = {}

        for symbol in self.symbols:
            ema_data_identifier = f"{symbol}_ema_data"
            rsi_data_identifier = f"{symbol}_rsi_data"
            ema_data, rsi_data = self._prepare_indicators(
                data[ema_data_identifier].copy(),
                data[rsi_data_identifier].copy()
            )

            # crossover confirmed
            ema_crossover_lookback = ema_data[
                self.ema_crossover_result_column].rolling(
                window=self.ema_cross_lookback_window
            ).max().astype(bool)

            # use only RSI column
            rsi_oversold = rsi_data[self.rsi_result_column] \
                < self.rsi_oversold_threshold

            buy_signal = rsi_oversold & ema_crossover_lookback
            buy_signals = buy_signal.fillna(False).astype(bool)
            signals[symbol] = buy_signals

            # Get all dates where there is a sell signal
            buy_signal_dates = buy_signals[buy_signals].index.tolist()

            if buy_signal_dates:
                self.buy_signal_dates[symbol] += buy_signal_dates

        return signals

    def generate_sell_signals(self, data: Dict[str, Any]) -> Dict[str, pd.Series]:
        """
        Generate sell signals based on the moving average crossover.

        Args:
            data (Dict[str, Any]): Dictionary containing all the data for
                the strategy data sources.

        Returns:
            Dict[str, pd.Series]: A dictionary where keys are symbols and values
                are pandas Series indicating sell signals (True/False).
        """

        signals = {}
        for symbol in self.symbols:
            ema_data_identifier = f"{symbol}_ema_data"
            rsi_data_identifier = f"{symbol}_rsi_data"
            ema_data, rsi_data = self._prepare_indicators(
                data[ema_data_identifier].copy(),
                data[rsi_data_identifier].copy()
            )

            # Confirmed by crossover between short-term EMA and long-term EMA
            # within a given lookback window
            ema_crossunder_lookback = ema_data[
                self.ema_crossunder_result_column].rolling(
                window=self.ema_cross_lookback_window
            ).max().astype(bool)

            # use only RSI column
            rsi_overbought = rsi_data[self.rsi_result_column] \
               >= self.rsi_overbought_threshold

            # Combine both conditions
            sell_signal = rsi_overbought & ema_crossunder_lookback
            sell_signal = sell_signal.fillna(False).astype(bool)
            signals[symbol] = sell_signal

            # Get all dates where there is a sell signal
            sell_signal_dates = sell_signal[sell_signal].index.tolist()

            if sell_signal_dates:
                self.sell_signal_dates[symbol] += sell_signal_dates

        return signals


if __name__ == "__main__":
    app = create_app()
    app.add_strategy(
        RSIEMACrossoverStrategy(
            time_unit=TimeUnit.HOUR,
            interval=2,
            market="bitvavo",
            rsi_time_frame="2h",
            rsi_period=14,
            rsi_overbought_threshold=70,
            rsi_oversold_threshold=30,
            ema_time_frame="2h",
            ema_short_period=12,
            ema_long_period=26,
            ema_cross_lookback_window=10
        )
    )

    # Market credentials for coinbase for both the portfolio connection and data sources will
    # be read from .env file, when not registering a market credential object in the app.
    app.add_market(
        market="bitvavo",
        trading_symbol="EUR",
    )
    backtest_range = BacktestDateRange(
        start_date=datetime(2023, 1, 1, tzinfo=timezone.utc),
        end_date=datetime(2024, 6, 1, tzinfo=timezone.utc)
    )
    backtest = app.run_backtest(
        backtest_date_range=backtest_range, initial_amount=1000
    )
    report = BacktestReport(backtest)
    report.show(backtest_date_range=backtest_range, browser=True)
```

**Section sources**
- [simple_trading_bot_example.py](file://examples/simple_trading_bot_example.py#L1-L254)