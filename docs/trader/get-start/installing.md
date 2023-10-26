# 安装

NautilusTrader已在以下64位平台上针对Python 3.9-3.11进行了测试和支持：

- **操作系统**：
   - Linux (Ubuntu)：20.04或更高版本，CPU架构为x86_64
   - macOS：12或更高版本，CPU架构为x86_64, ARM64
   - Windows Server：2022或更高版本，CPU架构为x86_64

**提示**：建议使用最新的受支持的稳定版本的Python运行平台，并在虚拟环境中隔离依赖项。

## 从PyPI安装

要使用Python的pip包管理器从PyPI安装最新的二进制轮（或sdist包），运行以下命令：

```bash
pip install -U nautilus_trader
```

## 附加组件

安装特定集成的可选依赖项作为“额外”组件：

- betfair：Betfair适配器
- docker：使用IB网关时需要Docker
- ib：Interactive Brokers适配器
- redis：使用Redis作为缓存数据库

要使用pip安装特定的附加组件，运行以下命令：

```bash
pip install -U "nautilus_trader[docker,ib,redis]"
```

## 从源码安装

从源码安装需要Python.h头文件，该文件包含在例如python-dev这样的开发版本中。您还需要最新的稳定版本的rustc和cargo来编译Rust库。

对于MacBook Pro M1/M2，请确保使用pyenv安装的Python配置了--enable-shared选项：

```bash
PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install <python_version>
```

请参见 [https://pyo3.rs/latest/getting_started#virtualenvs](https://pyo3.rs/latest/getting_started#virtualenvs)。

如果您首先安装了pyproject.toml中指定的构建依赖项，那么可以使用pip从源码安装。然而，我们强烈推荐如下使用poetry安装。

- 安装rustup（Rust工具链安装程序）：
   - Linux和macOS：
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
   - Windows：
      - 下载并安装rustup-init.exe
      - 安装“Visual Studio 2019的C++桌面开发”及构建工具

- 在当前shell中启用cargo：
   - Linux和macOS：
   ```bash
   source $HOME/.cargo/env
   ```
   - Windows：
      - 启动新的PowerShell

- 安装poetry（或按照他们网站上的安装指南进行安装）：
```bash
curl -sSL https://install.python-poetry.org | python3 -
```

- 用git克隆源码，并从项目的根目录中安装：
```bash
git clone https://github.com/nautechsystems/nautilus_trader
cd nautilus_trader
poetry install --only main --all-extras
```

## 从GitHub发布版安装

要从GitHub安装二进制轮，请首先导航到最新的发布版。下载适用于您的操作系统和Python版本的适当的.whl文件，然后运行：

```bash
pip install <file-name>.whl
```

以上就是NautilusTrader的安装步骤，包括从PyPI安装，安装附加组件，从源码安装，以及从GitHub发布版安装。按照以上指南，您应该能够成功安装NautilusTrader，并准备开始您的算法交易之旅。
