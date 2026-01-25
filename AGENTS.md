# AGENTS.md - Meteor MCP MetaTrader5 Server

## 🎯 Context
This project is an **MCP Server** that provides a bridge between AI Agents (like Claude or Gemini) and a running MetaTrader 5 (MT5) terminal.

## 🤖 Capabilities
This server allows an agent to:
1.  **Read Market Data**: Get current prices, historical candles (rates), and ticks.
2.  **Execute Trades**: Send Buy/Sell orders to the broker.
3.  **Manage Account**: check balance, positions, and history.

## 🛠️ Usage
This is typically run as a subprocess by the MCP Client (the Agent).
- **Standard Mode**: Stdio communication.
- **Tools**: It exposes tools like `get_market_data`, `place_order`, etc.

## ⚠️ Safety
- This server has direct access to the trading account.
- **Risk Management** logic is generally expected to be in the *Agent* (the client), not strictly in this server (though the server may have basic validations).

# File: api_reference.md

# API Reference

Complete reference for all tools, resources, and models provided by the MetaTrader 5 MCP Server.

## Connection Management Tools

### `initialize(path: str) -> bool`

Initialize the MetaTrader 5 terminal.

**Parameters:**
- `path` (str): Full path to the MT5 terminal executable

**Returns:**
- `bool`: True if initialization was successful, False otherwise

**Example:**
```python
initialize(path="C:\\Program Files\\MetaTrader 5\\terminal64.exe")
```

---

### `login(login: int, password: str, server: str) -> bool`

Log in to a MetaTrader 5 trading account.

**Parameters:**
- `login` (int): Trading account number
- `password` (str): Trading account password
- `server` (str): Trading server name

**Returns:**
- `bool`: True if login was successful, False otherwise

**Example:**
```python
login(login=123456, password="your_password", server="YourBroker-Demo")
```

---

### `shutdown() -> bool`

Shut down the connection to the MetaTrader 5 terminal.

**Returns:**
- `bool`: True if shutdown was successful

---

### `get_account_info() -> AccountInfo`

Get information about the current trading account.

**Returns:**
- `AccountInfo`: Account information including balance, equity, margin, etc.

**AccountInfo Model:**
```python
{
    "login": int,
    "balance": float,
    "equity": float,
    "margin": float,
    "margin_free": float,
    "margin_level": float,
    "profit": float,
    "currency": str,
    "leverage": int,
    "name": str,
    "server": str,
    # ... and more fields
}
```

---

### `get_terminal_info() -> Dict[str, Any]`

Get information about the MetaTrader 5 terminal.

**Returns:**
- `Dict[str, Any]`: Terminal information

---

### `get_version() -> Dict[str, Any]`

Get the MetaTrader 5 version.

**Returns:**
```python
{
    "version": int,
    "build": int,
    "date": str
}
```

---

## Market Data Tools

### `get_symbols() -> List[str]`

Get all available symbols (financial instruments).

**Returns:**
- `List[str]`: List of symbol names

---

### `get_symbols_by_group(group: str) -> List[str]`

Get symbols that match a specific group or pattern.

**Parameters:**
- `group` (str): Filter pattern (e.g., "*", "EUR*", "*.US")

**Returns:**
- `List[str]`: List of matching symbol names

---

### `get_symbol_info(symbol: str) -> SymbolInfo`

Get detailed information about a specific symbol.

**Parameters:**
- `symbol` (str): Symbol name (e.g., "EURUSD")

**Returns:**
- `SymbolInfo`: Detailed symbol information

---

### `get_symbol_info_tick(symbol: str) -> Dict[str, Any]`

Get the latest tick data for a symbol.

**Parameters:**
- `symbol` (str): Symbol name

**Returns:**
```python
{
    "time": int,
    "bid": float,
    "ask": float,
    "last": float,
    "volume": float,
    # ... more fields
}
```

---

### `copy_rates_from_pos(symbol: str, timeframe: int, start_pos: int, count: int) -> List[Dict[str, Any]]`

Get bars from a specified position.

**Parameters:**
- `symbol` (str): Symbol name
- `timeframe` (int): Timeframe constant (1=M1, 5=M5, 15=M15, 60=H1, 240=H4, 1440=D1, etc.)
- `start_pos` (int): Initial position (0 = most recent bar)
- `count` (int): Number of bars to retrieve

**Returns:**
- `List[Dict]`: List of bars with OHLCV data

**Example:**
```python
# Get last 100 bars of EURUSD on 15-minute timeframe
rates = copy_rates_from_pos("EURUSD", 15, 0, 100)
```

---

### `copy_rates_from_date(symbol: str, timeframe: int, date_from: datetime, count: int) -> List[Dict[str, Any]]`

Get bars starting from a specific date.

**Parameters:**
- `symbol` (str): Symbol name
- `timeframe` (int): Timeframe constant
- `date_from` (datetime): Start date
- `count` (int): Number of bars to retrieve

**Returns:**
- `List[Dict]`: List of bars with OHLCV data

---

### `copy_rates_range(symbol: str, timeframe: int, date_from: datetime, date_to: datetime) -> List[Dict[str, Any]]`

Get bars within a date range.

**Parameters:**
- `symbol` (str): Symbol name
- `timeframe` (int): Timeframe constant
- `date_from` (datetime): Start date
- `date_to` (datetime): End date

**Returns:**
- `List[Dict]`: List of bars with OHLCV data

---

## Trading Tools

### `order_send(request: OrderRequest) -> OrderResult`

Send an order to the trade server.

**Parameters:**
- `request` (OrderRequest): Order parameters

**OrderRequest Model:**
```python
{
    "action": int,        # TRADE_ACTION_DEAL (0) for market orders
    "symbol": str,        # Symbol name
    "volume": float,      # Lot size
    "type": int,          # ORDER_TYPE_BUY (0) or ORDER_TYPE_SELL (1)
    "price": float,       # Order price
    "sl": float,          # Stop loss (optional)
    "tp": float,          # Take profit (optional)
    "deviation": int,     # Max price deviation in points (optional)
    "magic": int,         # Magic number (optional)
    "comment": str,       # Order comment (optional)
    "type_time": int,     # Order time type (optional)
    "type_filling": int   # Order filling type (optional)
}
```

**Returns:**
- `OrderResult`: Order execution result

**Example:**
```python
request = {
    "action": 0,  # TRADE_ACTION_DEAL
    "symbol": "EURUSD",
    "volume": 0.1,
    "type": 0,  # ORDER_TYPE_BUY
    "price": 1.1000,
    "sl": 1.0950,
    "tp": 1.1050,
    "deviation": 20,
    "type_filling": 2  # ORDER_FILLING_IOC
}
result = order_send(request)
```

---

### `order_check(request: OrderRequest) -> Dict[str, Any]`

Check if an order can be placed with the specified parameters.

**Parameters:**
- `request` (OrderRequest): Order parameters to check

**Returns:**
- `Dict[str, Any]`: Check result with validation information

---

### `positions_get(symbol: Optional[str] = None, group: Optional[str] = None) -> List[Position]`

Get open positions.

**Parameters:**
- `symbol` (str, optional): Filter by symbol
- `group` (str, optional): Filter by group pattern

**Returns:**
- `List[Position]`: List of open positions

---

### `positions_get_by_ticket(ticket: int) -> Optional[Position]`

Get an open position by its ticket number.

**Parameters:**
- `ticket` (int): Position ticket

**Returns:**
- `Position` or `None`: Position information

---

### `orders_get(symbol: Optional[str] = None, group: Optional[str] = None) -> List[Dict[str, Any]]`

Get active pending orders.

**Parameters:**
- `symbol` (str, optional): Filter by symbol
- `group` (str, optional): Filter by group pattern

**Returns:**
- `List[Dict]`: List of active orders

---

### `history_orders_get(...) -> List[HistoryOrder]`

Get orders from history within a specified date range.

**Parameters:**
- `symbol` (str, optional): Filter by symbol
- `group` (str, optional): Filter by group
- `ticket` (int, optional): Filter by ticket
- `position` (int, optional): Filter by position ticket
- `from_date` (datetime, optional): Start date
- `to_date` (datetime, optional): End date

**Returns:**
- `List[HistoryOrder]`: List of historical orders

---

### `history_deals_get(...) -> List[Deal]`

Get deals from history within a specified date range.

**Parameters:**
- Similar to `history_orders_get()`

**Returns:**
- `List[Deal]`: List of historical deals

---

## Resources

### `mt5://timeframes`

Available timeframe constants for use with market data functions.

### `mt5://tick_flags`

Tick flag constants for filtering tick data.

### `mt5://order_types`

Order type constants (BUY, SELL, BUY_LIMIT, etc.).

### `mt5://order_filling_types`

Order filling type constants (FOK, IOC, RETURN).

### `mt5://order_time_types`

Order time type constants (GTC, DAY, SPECIFIED, etc.).

### `mt5://trade_actions`

Trade action constants (DEAL, PENDING, SLTP, MODIFY, REMOVE, CLOSE_BY).

---

## Constants Reference

### Timeframes
- `1` - M1 (1 minute)
- `5` - M5 (5 minutes)
- `15` - M15 (15 minutes)
- `30` - M30 (30 minutes)
- `60` - H1 (1 hour)
- `240` - H4 (4 hours)
- `1440` - D1 (1 day)
- `10080` - W1 (1 week)
- `43200` - MN1 (1 month)

### Order Types
- `0` - ORDER_TYPE_BUY
- `1` - ORDER_TYPE_SELL
- `2` - ORDER_TYPE_BUY_LIMIT
- `3` - ORDER_TYPE_SELL_LIMIT
- `4` - ORDER_TYPE_BUY_STOP
- `5` - ORDER_TYPE_SELL_STOP

### Order Filling Types
- `0` - ORDER_FILLING_FOK (Fill or Kill)
- `1` - ORDER_FILLING_IOC (Immediate or Cancel)
- `2` - ORDER_FILLING_RETURN

### Trade Actions
- `0` - TRADE_ACTION_DEAL (Market order)
- `1` - TRADE_ACTION_PENDING (Pending order)
- `2` - TRADE_ACTION_SLTP (Modify SL/TP)
- `3` - TRADE_ACTION_MODIFY (Modify order)
- `4` - TRADE_ACTION_REMOVE (Remove order)
- `5` - TRADE_ACTION_CLOSE_BY (Close by opposite position)

# File: getting_started.md

# Getting Started with MetaTrader 5 MCP Server

This MCP server provides access to the MetaTrader 5 API for trading and market data analysis through the Model Context Protocol.

## Installation

### Quick Install

```bash
git clone https://github.com/Qoyyuum/mcp-metatrader5-server
cd mcp-metatrader5-server
uv sync
```

### Configure for Claude Desktop

```bash
uv run fastmcp install src/mcp_mt5/main.py
```

Or manually add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp-metatrader5-server": {
      "command": "uv",
      "args": [
        "--directory",
        "C:\\path\\to\\mcp-metatrader5-server",
        "run",
        "mt5mcp"
      ]
    }
  }
}
```

## Basic Workflow

1. **Initialize the MT5 terminal**:
   - Use the `initialize()` tool to connect to the MT5 terminal.

2. **Log in to your trading account**:
   - Use the `login(account, password, server)` tool to log in to your trading account.

3. **Access market data**:
   - Use tools like `get_symbols()`, `copy_rates_from_pos()`, etc. to access market data.

4. **Place trades**:
   - Use the `order_send()` tool to place trades.

5. **Manage positions**:
   - Use tools like `positions_get()` to manage your open positions.

6. **Analyze trading history**:
   - Use tools like `history_orders_get()` and `history_deals_get()` to analyze your trading history.

7. **Shut down the connection**:
   - Use the `shutdown()` tool to close the connection to the MT5 terminal.

## Available Resources

The server provides helpful resources:
- `mt5://timeframes` - Available timeframe constants
- `mt5://tick_flags` - Tick flag constants
- `mt5://order_types` - Order type constants
- `mt5://order_filling_types` - Order filling type constants
- `mt5://order_time_types` - Order time type constants
- `mt5://trade_actions` - Trade action constants

## Example: Connecting to MT5 and Getting Market Data

When using with Claude Desktop or other MCP clients, you can ask the AI assistant:

> "Initialize MT5 at path C:\Program Files\MetaTrader 5\terminal64.exe, then login with account 123456, password 'your_password', and server 'your_server'. After that, get the available symbols and show me recent price data for EURUSD on the 15-minute timeframe."

The AI will use these tools:
1. `initialize(path="C:\\Program Files\\MetaTrader 5\\terminal64.exe")`
2. `login(login=123456, password="your_password", server="your_server")`
3. `get_symbols()`
4. `copy_rates_from_pos(symbol="EURUSD", timeframe=15, start_pos=0, count=100)`

## Example: Placing a Trade

Ask the AI assistant:

> "Place a buy order for 0.1 lots of EURUSD at market price with 20 pips deviation, stop loss at 1.0950, and take profit at 1.1050."

The AI will:
1. Get the current ask price using `get_symbol_info_tick("EURUSD")`
2. Create an order request with:
   - `action`: TRADE_ACTION_DEAL (0)
   - `symbol`: "EURUSD"
   - `volume`: 0.1
   - `type`: ORDER_TYPE_BUY (0)
   - `price`: current ask price
   - `sl`: 1.0950
   - `tp`: 1.1050
   - `deviation`: 20
   - `type_filling`: ORDER_FILLING_IOC (2)
3. Send the order using `order_send(request)`

## Important Notes

- **Always initialize MT5** before using any other tools
- **Login is required** to access account-specific data and place trades
- **Use `shutdown()`** when done to properly close the connection
- **Check order results** for success/failure before proceeding
- **Demo accounts** are recommended for testing


## Disclaimer

This software/model context protocol/author is not liable for any financial losses resulting from the use of the tools provided. Use at your own risk.


# File: index.md

# MetaTrader 5 MCP Server

[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/qoyyuum-mcp-metatrader5-server-badge.png)](https://mseep.ai/app/qoyyuum-mcp-metatrader5-server)
[![PyPI version](https://badge.fury.io/py/mcp-metatrader5-server.svg)](https://badge.fury.io/py/mcp-metatrader5-server)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)

A Model Context Protocol (MCP) server for MetaTrader 5, allowing AI assistants to interact with the MetaTrader 5 platform for trading and market data analysis.

## Features

- 🔌 **Connect to MetaTrader 5 terminal** - Initialize and manage MT5 connections
- 📊 **Access market data** - Get symbols, rates, ticks, and historical data
- 💹 **Place and manage trades** - Send orders, manage positions, and track history
- 📈 **Analyze trading history** - Review past orders and deals
- 🤖 **AI Integration** - Seamlessly integrate with AI assistants through MCP

## Quick Start

### Installation

=== "From Source"

    ```bash
    git clone https://github.com/Qoyyuum/mcp-metatrader5-server.git
    cd mcp-metatrader5-server
    uv sync
    ```

=== "From PyPI (Coming Soon)"

    ```bash
    uv pip install mcp-metatrader5-server
    ```

### Running the Server

=== "Stdio Mode (Default)"

    For MCP clients like Claude Desktop:

    ```bash
    uv run mt5mcp
    ```

=== "HTTP Mode (Development)"

    Create a `.env` file:

    ```env
    MT5_MCP_TRANSPORT=http
    MT5_MCP_HOST=127.0.0.1
    MT5_MCP_PORT=8000
    ```

    Then run:

    ```bash
    uv run mt5mcp
    ```

### Claude Desktop Setup

Install for Claude Desktop:

```bash
uv run fastmcp install src/mcp_mt5/main.py
```

Or manually configure `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp-metatrader5-server": {
      "command": "uv",
      "args": [
        "--directory",
        "C:\\path\\to\\mcp-metatrader5-server",
        "run",
        "mt5mcp"
      ]
    }
  }
}
```

## Requirements

- **uv** (recommended) or pip
- **Python 3.11 or higher**
- **MetaTrader 5 terminal** installed on Windows
- **MetaTrader 5 account** (demo or real)

## Available Tools

### Connection Management
- `initialize(path)` - Initialize the MT5 terminal
- `login(login, password, server)` - Log in to a trading account
- `shutdown()` - Close the connection to the MT5 terminal
- `get_account_info()` - Get trading account information
- `get_terminal_info()` - Get terminal information
- `get_version()` - Get MT5 version

### Market Data
- `get_symbols()` - Get all available symbols
- `get_symbols_by_group(group)` - Get symbols by group
- `get_symbol_info(symbol)` - Get symbol information
- `get_symbol_info_tick(symbol)` - Get latest tick data
- `copy_rates_from_pos()` - Get bars from position
- `copy_rates_from_date()` - Get bars from date
- `copy_rates_range()` - Get bars in date range
- `copy_ticks_from_pos()` - Get ticks from position
- `copy_ticks_from_date()` - Get ticks from date
- `copy_ticks_range()` - Get ticks in date range

### Trading
- `order_send(request)` - Send an order
- `order_check(request)` - Check order validity
- `positions_get()` - Get open positions
- `positions_get_by_ticket(ticket)` - Get position by ticket
- `orders_get()` - Get active orders
- `orders_get_by_ticket(ticket)` - Get order by ticket
- `history_orders_get()` - Get historical orders
- `history_deals_get()` - Get historical deals

## Resources

The server provides helpful resources for AI assistants:

- `mt5://timeframes` - Available timeframe constants
- `mt5://tick_flags` - Tick flag constants
- `mt5://order_types` - Order type constants
- `mt5://order_filling_types` - Order filling type constants
- `mt5://order_time_types` - Order time type constants
- `mt5://trade_actions` - Trade action constants

## Example Usage

Ask your AI assistant:

> "Initialize MT5, login to my account, and show me the current price of EURUSD"

> "Get the last 100 bars of GBPUSD on the 1-hour timeframe"

> "Place a buy order for 0.1 lots of EURUSD at market price"

> "Show me all my open positions"

## Next Steps

- [Getting Started Guide](getting_started.md) - Detailed setup and usage
- [Market Data Guide](market_data_guide.md) - Working with market data
- [Trading Guide](trading_guide.md) - Placing and managing trades
- [Pydantic AI Integration](pydantic_ai_integration.md) - Using with Pydantic AI framework
- [Publishing Guide](publishing.md) - Publishing to PyPI

## License

MIT License - see [LICENSE](https://github.com/Qoyyuum/mcp-metatrader5-server/blob/main/LICENSE) for details.

## Acknowledgements

- [MetaQuotes](https://www.metaquotes.net/) for the MetaTrader 5 platform
- [FastMCP](https://github.com/jlowin/fastmcp) for the MCP server implementation

# File: market_data_guide.md

# Market Data Guide for MetaTrader 5 API

This guide provides information on how to access and analyze market data using the MetaTrader 5 API.

## Timeframes

- `TIMEFRAME_M1`: 1 minute
- `TIMEFRAME_M5`: 5 minutes
- `TIMEFRAME_M15`: 15 minutes
- `TIMEFRAME_M30`: 30 minutes
- `TIMEFRAME_H1`: 1 hour
- `TIMEFRAME_H4`: 4 hours
- `TIMEFRAME_D1`: 1 day
- `TIMEFRAME_W1`: 1 week
- `TIMEFRAME_MN1`: 1 month

## Accessing Price Data

### Getting Bars (Candlesticks)

```python
# Get the last 100 bars for EURUSD on the H1 timeframe
rates = copy_rates_from_pos(symbol="EURUSD", timeframe=60, start_pos=0, count=100)

# Get bars for EURUSD on the D1 timeframe from a specific date
from datetime import datetime
rates = copy_rates_from_date(
    symbol="EURUSD",
    timeframe=1440,
    date_from=datetime(2023, 1, 1),
    count=100
)

# Get bars for EURUSD on the M15 timeframe within a date range
rates = copy_rates_range(
    symbol="EURUSD",
    timeframe=15,
    date_from=datetime(2023, 1, 1),
    date_to=datetime(2023, 1, 31)
)
```

### Getting Ticks

```python
# Get the last 1000 ticks for EURUSD
ticks = copy_ticks_from_pos(symbol="EURUSD", start_pos=0, count=1000)

# Get ticks for EURUSD from a specific date
ticks = copy_ticks_from_date(
    symbol="EURUSD",
    date_from=datetime(2023, 1, 1),
    count=1000
)

# Get ticks for EURUSD within a date range
ticks = copy_ticks_range(
    symbol="EURUSD",
    date_from=datetime(2023, 1, 1),
    date_to=datetime(2023, 1, 2)
)
```

## Analyzing Price Data

Once you have the price data, you can analyze it using pandas and numpy:

```python
import pandas as pd
import numpy as np

# Convert rates to a pandas DataFrame
df = pd.DataFrame(rates)

# Convert time to datetime
df['time'] = pd.to_datetime(df['time'], unit='s')

# Calculate moving averages
df['sma_20'] = df['close'].rolling(window=20).mean()
df['sma_50'] = df['close'].rolling(window=50).mean()

# Calculate RSI
def calculate_rsi(series, period=14):
    delta = series.diff()
    gain = delta.where(delta > 0, 0).rolling(window=period).mean()
    loss = -delta.where(delta < 0, 0).rolling(window=period).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

df['rsi'] = calculate_rsi(df['close'])

# Calculate MACD
def calculate_macd(series, fast=12, slow=26, signal=9):
    ema_fast = series.ewm(span=fast, adjust=False).mean()
    ema_slow = series.ewm(span=slow, adjust=False).mean()
    macd = ema_fast - ema_slow
    signal_line = macd.ewm(span=signal, adjust=False).mean()
    histogram = macd - signal_line
    return macd, signal_line, histogram

df['macd'], df['signal'], df['histogram'] = calculate_macd(df['close'])
```

## Getting Symbol Information

```python
# Get all available symbols
symbols = get_symbols()

# Get symbols by group
forex_symbols = get_symbols_by_group("*USD*")

# Get information about a specific symbol
symbol_info = get_symbol_info("EURUSD")

# Get the latest tick for a symbol
tick = get_symbol_info_tick("EURUSD")
```

# File: publishing.md

# Publishing to PyPI

This project is configured to automatically publish to PyPI when a new GitHub release is created.

## Setting up PyPI Token

To enable automatic publishing, you need to set up a PyPI API token as a GitHub secret:

1. **Create a PyPI API token**:
   - Log in to your PyPI account at https://pypi.org/
   - Go to Account Settings → API tokens
   - Create a new API token with scope "Entire account (all projects)"
   - Copy the token value (you won't be able to see it again)

2. **Add the token to GitHub secrets**:
   - Go to your GitHub repository
   - Navigate to Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `PYPI_API_TOKEN`
   - Value: Paste your PyPI token
   - Click "Add secret"

## Creating a Release

To trigger the publishing workflow:

1. Go to your GitHub repository
2. Navigate to Releases
3. Click "Create a new release"
4. Choose a tag version (e.g., v0.1.3)
5. Add a title and description
6. Click "Publish release"

The GitHub Actions workflow will automatically build and publish your package to PyPI.

## Manual Publishing

If you need to publish manually, you can use `uv` for faster builds and publishing:

```bash
# Install uv if you don't have it already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Build the package
uv build

# Upload to PyPI
uv publish --username __token__ --password YOUR_PYPI_TOKEN
```

Alternatively, you can use the standard Python tools:

```bash
# Build the package
python -m build

# Upload to PyPI
python -m twine upload dist/*
```

# File: pydantic_ai_integration.md

# Pydantic AI Integration Guide

This guide demonstrates how to integrate the MetaTrader 5 MCP Server with [Pydantic AI](https://ai.pydantic.dev/), a Python framework for building production-grade AI agents with type safety and structured outputs.

## What is Pydantic AI?

Pydantic AI is a Python agent framework designed to make it easy to build production-grade applications with Generative AI. It provides:

- **Type-safe agent development** using Pydantic models
- **Model-agnostic design** supporting OpenAI, Anthropic, Gemini, and more
- **Structured outputs** with validation
- **Tool/function calling** with type hints
- **Dependency injection** for clean architecture

## Prerequisites

- Python 3.11 or higher
- MetaTrader 5 terminal installed on Windows
- MCP MetaTrader 5 Server installed
- Pydantic AI installed

## Installation

```bash
# Install Pydantic AI
pip install pydantic-ai

# Or with specific model support
pip install 'pydantic-ai[openai]'  # For OpenAI
pip install 'pydantic-ai[anthropic]'  # For Anthropic Claude
pip install 'pydantic-ai[gemini]'  # For Google Gemini

# Install the MCP MetaTrader 5 Server
pip install mcp-metatrader5-server

# Or install from source
git clone https://github.com/Qoyyuum/mcp-metatrader5-server
cd mcp-metatrader5-server
uv sync
```

## Architecture Overview

The integration combines three components:

1. **Pydantic AI Agent** - Orchestrates the AI logic and tool calls
2. **MCP Client** - Communicates with the MCP server using the Model Context Protocol
3. **MT5 MCP Server** - Provides tools to interact with MetaTrader 5

```text
┌─────────────────────┐
│   Pydantic AI       │
│     Agent           │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│   MCP Client        │
│   (via stdio/http)  │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│ MT5 MCP Server      │
│  (FastMCP)          │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│  MetaTrader 5       │
│    Terminal         │
└─────────────────────┘
```

## Basic Integration Example

Here's a minimal example of using the MT5 MCP Server with Pydantic AI:

```python
import asyncio
from datetime import datetime
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

# Initialize the Pydantic AI agent
model = OpenAIModel('gpt-4o', api_key='your-api-key-here')
agent = Agent(
    model,
    system_prompt="""You are a trading assistant with access to MetaTrader 5.
    You can help analyze markets, retrieve data, and execute trades safely."""
)

async def use_mt5_with_pydantic_ai():
    # Start the MCP server connection
    server_params = StdioServerParameters(
        command="uvx",
        args=["--from", "mcp-metatrader5-server", "mt5mcp"]
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # Initialize the session
            await session.initialize()
            
            # List available tools from the MCP server
            tools_result = await session.list_tools()
            print(f"Available tools: {[tool.name for tool in tools_result.tools]}")
            
            # Call a tool - Initialize MT5
            init_result = await session.call_tool(
                "initialize",
                arguments={"path": r"C:\Program Files\MetaTrader 5\terminal64.exe"}
            )
            print(f"MT5 Initialization: {init_result.content}")
            
            # Get account info using the agent
            result = await agent.run(
                "Get my account information and summarize the balance and equity.",
                message_history=[]
            )
            print(f"Agent response: {result.data}")

# Run the async function
asyncio.run(use_mt5_with_pydantic_ai())
```

## Advanced Example: Trading Assistant Agent

This example shows a more sophisticated agent that can analyze market data and execute trades:

```python
import asyncio
from pydantic import BaseModel, Field
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIChatModel
from pydantic_ai.providers.openrouter import OpenRouterProvider
from pydantic_ai.mcp import MCPServerStdio
import os
import dotenv

dotenv.load_dotenv()
# Define structured output for market analysis
class MarketAnalysis(BaseModel):
    """Market analysis result"""
    symbol: str
    timeframe: int
    trend: str = Field(description="Current market trend: bullish, bearish, or sideways")
    support_level: float | None = Field(default=None, description="Key support level")
    resistance_level: float | None = Field(default=None, description="Key resistance level")
    recommendation: str = Field(description="Trading recommendation")
    risk_level: str = Field(description="Risk level: low, medium, or high")

# Setup MCP server for MetaTrader 5
mt5_server = MCPServerStdio(
    'uvx',
    args=['--from', 'mcp-metatrader5-server', 'mt5mcp'],
    timeout=30
)

# Create the AI model
model = OpenAIChatModel(
    'mistralai/mistral-small-3.2-24b-instruct:free',
    provider=OpenRouterProvider(api_key=os.getenv("OPENROUTER_API_KEY"))
)

# Create the trading agent with MT5 MCP server as a toolset
trading_agent = Agent(
    model,
    output_type=MarketAnalysis,
    system_prompt="""You are an expert trading analyst with access to MetaTrader 5 market data.
    
    Your responsibilities:
    1. Initialize MetaTrader 5 first using the initialize tool
    2. Analyze market data using technical indicators
    3. Identify key support and resistance levels
    4. Determine market trends (bullish, bearish, sideways)
    5. Provide clear trading recommendations
    6. Assess risk levels for each recommendation
    
    Always consider:
    - Multiple timeframe analysis
    - Risk management principles
    - Market volatility
    - Recent price action
    
    Available tools from MT5:
    - initialize: Initialize MT5 connection
    - copy_rates_from_pos: Get historical price data
    - get_symbol_info_tick: Get current price tick
    - get_account_info: Get account balance and info
    - positions_get: Get open positions
    - shutdown: Shutdown MT5 connection
    
    Be conservative with recommendations and always prioritize capital preservation.""",
    toolsets=[mt5_server],  # Register MT5 MCP server as a toolset
    retries=2
)

# Run the trading agent
async def run_trading_analysis():
    """Run a complete market analysis using the trading agent with MCP"""
    
    # Use the agent context manager which handles MCP server lifecycle
    async with trading_agent:
        print("\n🔍 Analyzing EURUSD market...")
        
        # The agent will automatically use the MT5 MCP tools
        result = await trading_agent.run(
            """Analyze the EURUSD market on the 1-hour timeframe.
            
            Steps to follow:
            1. First initialize MetaTrader 5 with path: C:\\Program Files\\MetaTrader 5\\terminal64.exe
            2. Get the last 100 bars of price data for EURUSD on 60-minute timeframe
            3. Get the current price for EURUSD
            4. Analyze the data to identify the trend
            5. Find key support and resistance levels
            6. Provide a trading recommendation with risk assessment
            7. Finally, shutdown the MT5 connection
            
            Return your analysis in the structured format.
            """
        )
        
        # Display results
        print("\n📊 Market Analysis Results:")
        print(f"Symbol: {result.output.symbol}")
        print(f"Timeframe: {result.output.timeframe} minutes")
        print(f"Trend: {result.output.trend}")
        print(f"Support: {result.output.support_level}")
        print(f"Resistance: {result.output.resistance_level}")
        print(f"Recommendation: {result.output.recommendation}")
        print(f"Risk Level: {result.output.risk_level}")

# Run the analysis
if __name__ == "__main__":
    asyncio.run(run_trading_analysis())
```

## Example: Automated Trading Bot

Here's a complete example of an automated trading bot using Pydantic AI and the MT5 MCP Server:

```python
import asyncio
from datetime import datetime
from typing import Any, List
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext
from pydantic_ai.models.anthropic import AnthropicModel
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

class TradingDecision(BaseModel):
    """Structured trading decision"""
    should_trade: bool
    action: str | None = Field(default=None, description="BUY or SELL")
    symbol: str
    volume: float = 0.1
    stop_loss: float | None = None
    take_profit: float | None = None
    reason: str = Field(description="Reason for the decision")

class TradingBot:
    """Automated trading bot using Pydantic AI and MT5 MCP"""
    
    def __init__(self, api_key: str, mt5_path: str):
        self.mt5_path = mt5_path
        
        # Create the decision-making agent
        model = AnthropicModel('claude-3-5-sonnet-20241022', api_key=api_key)
        self.agent = Agent(
            model,
            result_type=TradingDecision,
            system_prompt="""You are a conservative trading bot focused on capital preservation.
            
            Rules:
            1. Only trade when you have high confidence (>80%)
            2. Always use stop loss and take profit
            3. Maximum risk per trade: 2% of account balance
            4. Prefer trending markets over ranging markets
            5. Never trade during high-impact news events
            6. Always check existing positions before opening new ones
            
            Decision criteria:
            - Strong trend confirmation
            - Clear support/resistance levels
            - Risk/reward ratio of at least 1:2
            - Favorable market conditions""",
        )
        
        # Add tools
        self._register_tools()
    
    def _register_tools(self):
        """Register MCP tools with the agent"""
        
        @self.agent.tool
        async def analyze_trend(
            ctx: RunContext[Any],
            symbol: str,
            timeframe: int = 240  # 4-hour
        ) -> str:
            """Analyze market trend for a symbol"""
            # Get 200 bars for trend analysis
            result = await ctx.deps.session.call_tool(
                "copy_rates_from_pos",
                arguments={
                    "symbol": symbol,
                    "timeframe": timeframe,
                    "start_pos": 0,
                    "count": 200
                }
            )
            # Return the data for AI to analyze
            return f"Price data retrieved: {len(result.content)} bars"
        
        @self.agent.tool
        async def check_positions(
            ctx: RunContext[Any],
            symbol: str | None = None
        ) -> int:
            """Check number of open positions"""
            args = {"symbol": symbol} if symbol else {}
            result = await ctx.deps.session.call_tool(
                "positions_get",
                arguments=args
            )
            positions = result.content[0].text if result.content else []
            return len(positions) if isinstance(positions, list) else 0
        
        @self.agent.tool
        async def get_balance(ctx: RunContext[Any]) -> float:
            """Get account balance"""
            result = await ctx.deps.session.call_tool(
                "get_account_info",
                arguments={}
            )
            # Parse and return balance
            return 10000.0  # Placeholder
    
    async def make_trading_decision(
        self,
        session: ClientSession,
        symbol: str,
        timeframe: int = 240
    ) -> TradingDecision:
        """Make a trading decision for a symbol"""
        
        class Deps(BaseModel):
            session: Any = Field(exclude=True)
        
        deps = Deps(session=session)
        
        result = await self.agent.run(
            f"""Analyze {symbol} on the {timeframe}-minute timeframe and decide if we should trade.
            
            Consider:
            1. Current trend direction
            2. Existing positions (don't over-trade)
            3. Account balance and risk
            4. Support/resistance levels
            5. Overall market conditions
            
            Provide your decision with clear reasoning.""",
            deps=deps
        )
        
        return result.data
    
    async def execute_trade(
        self,
        session: ClientSession,
        decision: TradingDecision
    ) -> dict:
        """Execute a trade based on the decision"""
        
        if not decision.should_trade:
            return {"status": "skipped", "reason": decision.reason}
        
        # Get current price
        tick_result = await session.call_tool(
            "get_symbol_info_tick",
            arguments={"symbol": decision.symbol}
        )
        
        # Prepare order (simplified - should include proper price, SL, TP calculation)
        order_params = {
            "action": 1,  # TRADE_ACTION_DEAL
            "symbol": decision.symbol,
            "volume": decision.volume,
            "type": 0 if decision.action == "BUY" else 1,
            "price": 1.1000,  # Should use actual price from tick_result
            "sl": decision.stop_loss or 0.0,
            "tp": decision.take_profit or 0.0,
            "deviation": 20,
            "magic": 123456,
            "comment": f"PydanticAI: {decision.reason[:50]}",
            "type_filling": 2  # IOC
        }
        
        # Send order
        result = await session.call_tool(
            "order_send",
            arguments={"request": order_params}
        )
        
        return {"status": "executed", "result": result.content}
    
    async def run(self, symbols: List[str], login: int, password: str, server: str):
        """Run the trading bot"""
        
        server_params = StdioServerParameters(
            command="uvx",
            args=["--from", "mcp-metatrader5-server", "mt5mcp"]
        )
        
        async with stdio_client(server_params) as (read, write):
            async with ClientSession(read, write) as session:
                # Initialize
                await session.initialize()
                await session.call_tool(
                    "initialize",
                    arguments={"path": self.mt5_path}
                )
                await session.call_tool(
                    "login",
                    arguments={
                        "login": login,
                        "password": password,
                        "server": server
                    }
                )
                
                print("🤖 Trading bot started")
                
                # Analyze each symbol
                for symbol in symbols:
                    print(f"\n📊 Analyzing {symbol}...")
                    decision = await self.make_trading_decision(session, symbol)
                    
                    print(f"Decision: {'TRADE' if decision.should_trade else 'SKIP'}")
                    print(f"Reason: {decision.reason}")
                    
                    if decision.should_trade:
                        result = await self.execute_trade(session, decision)
                        print(f"Trade result: {result['status']}")
                
                # Cleanup
                await session.call_tool("shutdown", arguments={})
                print("\n✅ Trading bot completed")

# Usage
async def main():
    bot = TradingBot(
        api_key="your-anthropic-api-key",
        mt5_path=r"C:\Program Files\MetaTrader 5\terminal64.exe"
    )
    
    await bot.run(
        symbols=["EURUSD", "GBPUSD", "USDJPY"],
        login=123456,
        password="your-password",
        server="your-server"
    )

if __name__ == "__main__":
    asyncio.run(main())
```

## Best Practices

### 1. Error Handling

Always handle MCP call failures gracefully:

```python
try:
    result = await session.call_tool("get_account_info", arguments={})
    if result.isError:
        print(f"Error: {result.content}")
    else:
        account_info = result.content[0].text
except Exception as e:
    print(f"MCP call failed: {e}")
```

### 2. Connection Management

Properly manage the MCP session lifecycle:

```python
async with stdio_client(server_params) as (read, write):
    async with ClientSession(read, write) as session:
        # Initialize at start
        await session.initialize()
        
        try:
            # Your trading logic here
            pass
        finally:
            # Always shutdown MT5
            await session.call_tool("shutdown", arguments={})
```

### 3. Type Safety

Use Pydantic models for structured outputs:

```python
class AccountSummary(BaseModel):
    balance: float
    equity: float
    margin_free: float
    margin_level: float
    
agent = Agent(model, result_type=AccountSummary)
result = await agent.run("Get my account summary", deps=deps)
# result.data is typed as AccountSummary
```

### 4. Dependency Injection

Inject the MCP session and configuration:

```python
class TradingDeps(BaseModel):
    session: Any = Field(exclude=True)
    max_risk_per_trade: float = 0.02
    max_open_positions: int = 3

agent = Agent(model, deps_type=TradingDeps)
```

### 5. Retry Logic

Use Pydantic AI's built-in retry mechanism:

```python
agent = Agent(
    model,
    retries=3,  # Retry failed calls up to 3 times
    result_retries=2  # Retry result validation failures
)
```

## Common Patterns

### Pattern 1: Multi-Timeframe Analysis

```python
async def analyze_multiple_timeframes(session, symbol: str):
    """Analyze a symbol across multiple timeframes"""
    timeframes = [15, 60, 240, 1440]  # 15min, 1h, 4h, 1day
    
    analyses = {}
    for tf in timeframes:
        result = await session.call_tool(
            "copy_rates_from_pos",
            arguments={"symbol": symbol, "timeframe": tf, "start_pos": 0, "count": 100}
        )
        analyses[f"{tf}min"] = result.content
    
    return analyses
```

### Pattern 2: Risk Management

```python
async def calculate_position_size(
    session,
    symbol: str,
    balance: float,
    risk_percent: float = 0.02,
    stop_loss_pips: int = 50
) -> float:
    """Calculate position size based on risk"""
    symbol_info = await session.call_tool(
        "get_symbol_info",
        arguments={"symbol": symbol}
    )
    
    # Calculate position size
    risk_amount = balance * risk_percent
    pip_value = 10  # Simplified, should calculate from symbol_info
    position_size = risk_amount / (stop_loss_pips * pip_value)
    
    return round(position_size, 2)
```

### Pattern 3: Monitoring Loop

```python
async def monitor_positions(session, interval_seconds: int = 60):
    """Continuously monitor open positions"""
    while True:
        positions = await session.call_tool(
            "positions_get",
            arguments={}
        )
        
        for position in positions.content[0].text:
            # Check if stop loss or take profit should be adjusted
            # Check for trailing stop logic
            # Log position status
            pass
        
        await asyncio.sleep(interval_seconds)
```

## Troubleshooting

### Issue: MCP Server Not Starting

**Solution**: Verify the server path and check it's installed:

```bash
uvx --from mcp-metatrader5-server mt5mcp --help
```

### Issue: MT5 Connection Failed

**Solution**: Ensure MT5 terminal is installed and path is correct:

```python
# Try different common paths
paths = [
    r"C:\Program Files\MetaTrader 5\terminal64.exe",
    r"C:\Program Files (x86)\MetaTrader 5\terminal64.exe",
]

for path in paths:
    result = await session.call_tool("initialize", arguments={"path": path})
    if not result.isError:
        break
```

### Issue: Tool Calls Failing

**Solution**: Check tool availability and parameters:

```python
# List all available tools
tools = await session.list_tools()
for tool in tools.tools:
    print(f"Tool: {tool.name}")
    print(f"Description: {tool.description}")
    print(f"Parameters: {tool.inputSchema}")
```

## Resources

- [Pydantic AI Documentation](https://ai.pydantic.dev/)
- [MCP Protocol Specification](https://modelcontextprotocol.io/)
- [MetaTrader 5 Python Documentation](https://www.mql5.com/en/docs/python_metatrader5)
- [MCP MetaTrader 5 Server Repository](https://github.com/Qoyyuum/mcp-metatrader5-server)

## Next Steps

1. Review the [API Reference](api_reference.md) for available tools
2. Check the [Trading Guide](trading_guide.md) for trade execution details
3. Explore the [Market Data Guide](market_data_guide.md) for data analysis
4. Build your custom trading agents with Pydantic AI's advanced features

## License

This integration guide is part of the MCP MetaTrader 5 Server project and is licensed under the MIT License.

# File: trading_guide.md

# Trading Guide for MetaTrader 5 API

This guide provides information on how to place and manage trades using the MetaTrader 5 API.

## Order Types

- **Market Orders**:
  - `ORDER_TYPE_BUY`: Buy at market price
  - `ORDER_TYPE_SELL`: Sell at market price

- **Pending Orders**:
  - `ORDER_TYPE_BUY_LIMIT`: Buy at specified price (lower than current price)
  - `ORDER_TYPE_SELL_LIMIT`: Sell at specified price (higher than current price)
  - `ORDER_TYPE_BUY_STOP`: Buy at specified price (higher than current price)
  - `ORDER_TYPE_SELL_STOP`: Sell at specified price (lower than current price)
  - `ORDER_TYPE_BUY_STOP_LIMIT`: Buy stop limit order
  - `ORDER_TYPE_SELL_STOP_LIMIT`: Sell stop limit order

## Trade Actions

- `TRADE_ACTION_DEAL`: Place a market order
- `TRADE_ACTION_PENDING`: Place a pending order
- `TRADE_ACTION_SLTP`: Modify stop loss and take profit levels
- `TRADE_ACTION_MODIFY`: Modify an existing order
- `TRADE_ACTION_REMOVE`: Remove a pending order
- `TRADE_ACTION_CLOSE_BY`: Close a position by an opposite one

## Example: Placing a Market Buy Order

```python
from mt5_server import OrderRequest

# Create an order request
request = OrderRequest(
    action=mt5.TRADE_ACTION_DEAL,
    symbol="EURUSD",
    volume=0.1,
    type=mt5.ORDER_TYPE_BUY,
    price=mt5.symbol_info_tick("EURUSD").ask,
    deviation=20,
    magic=123456,
    comment="Buy order",
    type_time=mt5.ORDER_TIME_GTC,
    type_filling=mt5.ORDER_FILLING_IOC
)

# Send the order
result = order_send(request)
```

## Example: Placing a Pending Order

```python
from mt5_server import OrderRequest

# Create a pending order request
request = OrderRequest(
    action=mt5.TRADE_ACTION_PENDING,
    symbol="EURUSD",
    volume=0.1,
    type=mt5.ORDER_TYPE_BUY_LIMIT,
    price=1.08,  # Price to buy at
    sl=1.07,     # Stop loss
    tp=1.09,     # Take profit
    deviation=20,
    magic=123456,
    comment="Buy limit order",
    type_time=mt5.ORDER_TIME_GTC,
    type_filling=mt5.ORDER_FILLING_IOC
)

# Send the order
result = order_send(request)
```

## Example: Modifying an Existing Position

```python
from mt5_server import OrderRequest

# Get the position
position = positions_get_by_ticket(ticket=123456)

# Create a request to modify stop loss and take profit
request = OrderRequest(
    action=mt5.TRADE_ACTION_SLTP,
    symbol=position.symbol,
    sl=1.07,     # New stop loss
    tp=1.09,     # New take profit
    position=position.ticket
)

# Send the order
result = order_send(request)
```

## Example: Closing a Position

```python
from mt5_server import OrderRequest

# Get the position
position = positions_get_by_ticket(ticket=123456)

# Create a request to close the position
request = OrderRequest(
    action=mt5.TRADE_ACTION_DEAL,
    symbol=position.symbol,
    volume=position.volume,
    type=mt5.ORDER_TYPE_SELL if position.type == mt5.ORDER_TYPE_BUY else mt5.ORDER_TYPE_BUY,
    price=mt5.symbol_info_tick(position.symbol).bid if position.type == mt5.ORDER_TYPE_BUY else mt5.symbol_info_tick(position.symbol).ask,
    position=position.ticket,
    deviation=20,
    magic=123456,
    comment="Close position",
    type_time=mt5.ORDER_TIME_GTC,
    type_filling=mt5.ORDER_FILLING_IOC
)

# Send the order
result = order_send(request)
```

# File: tests/README.md

# Tests

This directory contains unit and integration tests for the MCP MetaTrader 5 Server.

## Running Tests

### Install Test Dependencies

```bash
uv sync --extra dev
```

### Run All Tests

```bash
uv run pytest
```

### Run Specific Test Files

```bash
# Test timeframe validation
uv run pytest tests/test_timeframes.py

# Test connection management
uv run pytest tests/test_connection.py

# Test account info
uv run pytest tests/test_account_info.py

# Test Pydantic models
uv run pytest tests/test_models.py
```

### Run Tests by Marker

```bash
# Run only unit tests (don't require MT5)
uv run pytest -m unit

# Run only integration tests (require MT5 connection)
uv run pytest -m integration

# Skip slow tests
uv run pytest -m "not slow"
```

### Run with Coverage

```bash
# Generate coverage report
uv run pytest --cov=mcp_mt5 --cov-report=html

# View HTML report
open htmlcov/index.html  # macOS/Linux
start htmlcov/index.html  # Windows
```

### Run with Verbose Output

```bash
uv run pytest -v
```

### Run Specific Test

```bash
uv run pytest tests/test_timeframes.py::TestTimeframeValidation::test_valid_timeframe_conversion
```

## Test Structure

### Unit Tests (`@pytest.mark.unit`)
- Don't require MT5 connection
- Use mocks for MT5 functions
- Fast execution
- Test individual functions in isolation

### Integration Tests (`@pytest.mark.integration`)
- Require actual MT5 connection
- Test real MT5 interactions
- Slower execution
- Require MT5 terminal to be running

### Test Files

- `conftest.py` - Shared fixtures and configuration
- `test_timeframes.py` - Timeframe validation and conversion
- `test_connection.py` - Connection management (initialize, login, shutdown)
- `test_account_info.py` - Account and terminal information
- `test_models.py` - Pydantic model validation

## Writing New Tests

### Example Unit Test

```python
import pytest
from unittest.mock import patch

@pytest.mark.unit
def test_my_function():
    """Test description."""
    # Arrange
    expected = "result"
    
    # Act
    result = my_function()
    
    # Assert
    assert result == expected
```

### Example Integration Test

```python
import pytest

@pytest.mark.integration
def test_mt5_connection():
    """Test actual MT5 connection."""
    # This test requires MT5 to be running
    result = initialize(path="C:\\Program Files\\MetaTrader 5\\terminal64.exe")
    assert result is True
```

## Continuous Integration

Tests are automatically run on GitHub Actions for:
- Pull requests
- Pushes to main branch
- Tagged releases

## Coverage Goals

- **Target**: 80%+ code coverage
- **Current**: Run `pytest --cov` to see current coverage
- Focus on critical paths and error handling

## Disclaimer

This software/model context protocol/author is not liable for any financial losses resulting from the use of the tools provided. Use at your own risk.
