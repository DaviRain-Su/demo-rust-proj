# 快速入门

该指南解释了如何使用NautilusTrader和一些外汇（FX）数据进行回测。Nautilus的维护者已经使用标准的Nautilus持久性格式（Parquet）预加载了一些测试数据以供本指南使用。

关于如何将数据加载到Nautilus中的更多详细信息，请参见[回测教程](https://docs.nautilustrader.io/tutorials/backtest_high_level.html)。

## 在Docker中运行
提供了一个自包含的dockerized jupyter notebook服务器，无需任何设置或安装。这是尝试Nautilus的最快方法。请注意，当容器被删除时，任何数据都将被删除。

开始之前，安装docker：
- 访问docker.com并按照说明进行操作。
- 从终端下载最新的镜像：
```bash
docker pull ghcr.io/nautechsystems/jupyterlab:develop --platform linux/amd64
```
- 运行docker容器，暴露jupyter端口：
```bash
docker run -p 8888:8888 ghcr.io/nautechsystems/jupyterlab:develop
```
- 在您的网络浏览器中打开 localhost:{port}：
```url
https://localhost:8888
```

**警告**：NautilusTrader目前超过了Jupyter notebook日志记录（stdout输出）的速率限制，这就是为什么在示例中将log_level设置为“ERROR”。如果您降低此级别以查看更多日志记录，那么笔记本将在单元格执行期间挂起。

## 获取样本数据
为了节省时间，我们准备了一个脚本，将样本数据加载到Nautilus格式中，以便与此示例一起使用。首先，通过运行下一个单元格下载并加载数据（大约需要1-2分钟）：
```bash
!apt-get update && apt-get install curl -y
!curl https://raw.githubusercontent.com/nautechsystems/nautilus_data/main/scripts/hist_data_to_catalog.py | python -
```
## 连接到ParquetDataCatalog
如果一切正常，您应该能够在目录中看到一个EUR/USD工具：
```python
from nautilus_trader.persistence.catalog import ParquetDataCatalog

# 你也可以使用`ParquetDataCatalog.from_env()`，它将使用`NAUTILUS_PATH`环境变量
# catalog = ParquetDataCatalog.from_env()
catalog = ParquetDataCatalog("./catalog")
catalog.instruments()
```
## 编写交易策略
NautilusTrader包括一些内置的指标，在这个示例中，我们将使用MACD指标构建一个简单的交易策略。

您可以在这里阅读更多关于MACD的信息，所以这个指标仅仅作为一个没有预期alpha的例子。也有一种方法可以注册指标以接收某些数据类型，但是在这个例子中，我们在on_quote_tick方法中手动将接收到的QuoteTick传递给指标。
```python
# 代码省略，为了简洁，只显示了主要的类和方法
class MACDStrategy(Strategy):
    # ... 省略了构造函数和其他方法 ...

    def on_quote_tick(self, tick: QuoteTick):
        # 代码省略，显示了如何处理报价滴答和检查条目和退出条件 ...

    def check_for_entry(self):
        # ... 省略了检查条目条件的代码 ...

    def check_for_exit(self):
        # ... 省略了检查退出条件的代码 ...
```
## 配置回测
现在我们有了交易策略和数据，可以开始配置回测运行了！Nautilus使用BacktestNode来编排回测运行，这需要一些设置。这可能起初看起来有点复杂，但是这对于Nautilus所努力实现的功能来说是必要的。

要配置`BacktestNode`，我们首先需要创建一个`BacktestRunConfig`实例，配置回测的以下（最小）方面：
- engine - 代表我们核心系统的回测引擎，其中也包含我们的策略。
- venues - 回测中可用的模拟场所（交易所或经纪人）。
- data - 我们想要在回测中执行的输入数据。

## 场所（Venue）
首先，我们创建一个场所配置。对于这个示例，我们将创建一个模拟的FX ECN。场所需要一个名称，作为ID（在这种情况下是SIM），以及一些基本配置，例如账户类型（CASH与MARGIN）、可选的基础货币和起始余额。
```python
from nautilus_trader.config import BacktestVenueConfig

venue = BacktestVenueConfig(
    name="SIM",
    oms_type="NETTING",
    account_type="MARGIN",
    base_currency="USD",
    starting_balances=["1_000_000 USD"],
)
```
## 乐器（Instruments）
其次，我们需要知道我们想要加载数据的乐器，我们可以使用ParquetDataCatalog来实现这一点：
```python
instruments = catalog.instruments(as_nautilus=True)
instruments
```
## 数据
接下来，我们需要为回测配置数据。Nautilus在加载回测数据方面具有很高的灵活性，但这也意味着需要一些配置。

对于每种滴答类型（和乐器），我们添加一个`BacktestDataConfig`。在这种情况下，我们只是为我们的EUR/USD乐器添加了`QuoteTick`：
```python
from nautilus_trader.config import BacktestDataConfig
from nautilus_trader.model.data import QuoteTick

data = BacktestDataConfig(
    catalog_path=str(catalog.path),
    data_cls=QuoteTick,
    instrument_id=str(instruments[0].id),
    end_time="2020-01-10",
)
```
## 引擎
然后，我们需要一个`BacktestEngineConfig`，它代表我们核心交易系统的配置。在这里我们需要传递我们的交易策略，我们还可以调整日

志级别并配置许多其他组件（不过，默认设置也可以）：

通过`ImportableStrategyConfig`添加策略，该配置允许从任意文件或用户包中导入策略。在这种情况下，我们的`MACDStrategy`定义在当前模块中，python将其引用为`__main__`。
```python
from nautilus_trader.config import BacktestEngineConfig
from nautilus_trader.config import ImportableStrategyConfig
from nautilus_trader.config import LoggingConfig

engine = BacktestEngineConfig(
    strategies=[
        ImportableStrategyConfig(
            strategy_path="__main__:MACDStrategy",
            config_path="__main__:MACDConfig",
            config=dict(
              instrument_id=instruments[0].id.value,
              fast_period=12,
              slow_period=26,
            ),
        )
    ],
    logging=LoggingConfig(log_level="ERROR"),  # Lower to `INFO` to see more logging about orders, events, etc.
)
```
## 运行回测
我们现在可以将我们的各种配置片段传递给`BacktestRunConfig`。这个对象现在包含了我们回测的完整配置。
```python
from nautilus_trader.config import BacktestRunConfig

config = BacktestRunConfig(
    engine=engine,
    venues=[venue],
    data=[data],
)
```
`BacktestNode`类将编排回测运行。将配置和执行分离的原因是`BacktestNode`允许运行多个配置（不同的参数或数据批次）。

我们现在准备运行一些回测了！
```python
from nautilus_trader.backtest.node import BacktestNode
from nautilus_trader.backtest.results import BacktestResult

node = BacktestNode(configs=[config])

# 同步运行一个或多个配置
results: list[BacktestResult] = node.run()
```
现在运行完成后，我们还可以直接查询`BacktestNode`内部使用的`BacktestEngine`，方法是使用运行配置的ID。

引擎可以提供额外的报告和信息。
```python
from nautilus_trader.backtest.engine import BacktestEngine
from nautilus_trader.model.identifiers import Venue

engine: BacktestEngine = node.get_engine(config.id)

engine.trader.generate_order_fills_report()
engine.trader.generate_positions_report()
engine.trader.generate_account_report(Venue("SIM"))
```

以上内容涵盖了使用NautilusTrader进行回测的基本步骤，从获取样本数据，编写交易策略，配置回测，到运行回测和查看结果。在每个步骤中，都提供了详细的代码和说明，以帮助您理解如何使用NautilusTrader进行算法交易的回测。
