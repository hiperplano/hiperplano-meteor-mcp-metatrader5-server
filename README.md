[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/qoyyuum-mcp-metatrader5-server-badge.png)](https://mseep.ai/app/qoyyuum-mcp-metatrader5-server)

[![codecov](https://codecov.io/github/Qoyyuum/mcp-metatrader5-server/graph/badge.svg?token=SRECTEZUAR)](https://codecov.io/github/Qoyyuum/mcp-metatrader5-server)

[![PyPI version](https://badge.fury.io/py/mcp-metatrader5-server.svg)](https://pypi.org/project/mcp-metatrader5-server/)

# MetaTrader 5 MCP Server

A Model Context Protocol (MCP) server for MetaTrader 5, allowing AI assistants to interact with the MetaTrader 5 platform for trading and market data analysis. [Documentation](https://mcp-metatrader5-server.readthedocs.io)

## Features

- Connect to MetaTrader 5 terminal
- Access market data (symbols, rates, ticks)
- Place and manage trades
- Analyze trading history
- Integrate with AI assistants through the Model Context Protocol

## Installation

### From PyPI 

```bash
uvx --from mcp-metatrader5-server mt5mcp
```

### From Source

```bash
git clone https://github.com/Qoyyuum/mcp-metatrader5-server.git
cd mcp-metatrader5-server
uv sync
uv run mt5mcp
```

## Requirements

- **uv** (recommended) or pip
- **Python 3.11 or higher**
- **MetaTrader 5 terminal** installed on Windows
- **MetaTrader 5 account** (demo or real)

## Usage

### Quick Start

The server runs in **stdio mode** by default for MCP clients like Claude Desktop:

```bash
uv run mt5mcp
```

### Development Mode (HTTP)

For testing with HTTP transport, create a `.env` file:

```env
MT5_MCP_TRANSPORT=http
MT5_MCP_HOST=127.0.0.1
MT5_MCP_PORT=8000
```

Then run:

```bash
uv run mt5mcp
```

The server will start at http://127.0.0.1:8000

### Installing for MCP Clients

#### Method 1: Using `uvx` (Simplest - No Installation Required) ⭐

Add this configuration to your MCP client's config file:

**For Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "mcp-metatrader5-server": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/Qoyyuum/mcp-metatrader5-server",
        "mt5mcp"
      ]
    }
  }
}
```

#### Method 2: Using FastMCP Install (Recommended)

```bash
git clone https://github.com/Qoyyuum/mcp-metatrader5-server
cd mcp-metatrader5-server
```

After git cloning the repo, run the following commands:

For MCP JSON format

```bash
uv run fastmcp install mcp-json src/mcp_mt5/main.py
```

For Claude Desktop

```bash
uv run fastmcp install claude-desktop src/mcp_mt5/main.py
```

For Claude Code

```bash
uv run fastmcp install claude-code src/mcp_mt5/main.py
```

For Cursor

```bash
uv run fastmcp install cursor src/mcp_mt5/main.py
```

For Gemini CLI

```bash
uv run fastmcp install gemini-cli src/mcp_mt5/main.py
```


#### Method 3: Manual Configuration

Add this to your `claude_desktop_config.json` or whatever LLM config file:

```json
{
  "mcpServers": {
    "mcp-metatrader5-server": {
      "command": "uvx",
      "args": [
        "--from",
        "mcp-metatrader5-server",
        "mt5mcp"
      ]
    }
  }
}
```

## API Reference

### Connection Management

- `initialize()`: Initialize the MT5 terminal
- `login(account, password, server)`: Log in to a trading account
- `shutdown()`: Close the connection to the MT5 terminal

### Market Data Functions

- `get_symbols()`: Get all available symbols
- `get_symbols_by_group(group)`: Get symbols by group
- `get_symbol_info(symbol)`: Get information about a specific symbol
- `get_symbol_info_tick(symbol)`: Get the latest tick for a symbol
- `copy_rates_from_pos(symbol, timeframe, start_pos, count)`: Get bars from a specific position
- `copy_rates_from_date(symbol, timeframe, date_from, count)`: Get bars from a specific date
- `copy_rates_range(symbol, timeframe, date_from, date_to)`: Get bars within a date range
- `copy_ticks_from_pos(symbol, start_pos, count)`: Get ticks from a specific position
- `copy_ticks_from_date(symbol, date_from, count)`: Get ticks from a specific date
- `copy_ticks_range(symbol, date_from, date_to)`: Get ticks within a date range

### Trading Functions

- `order_send(request)`: Send an order to the trade server
- `order_check(request)`: Check if an order can be placed with the specified parameters
- `positions_get(symbol, group)`: Get open positions
- `positions_get_by_ticket(ticket)`: Get an open position by its ticket
- `orders_get(symbol, group)`: Get active orders
- `orders_get_by_ticket(ticket)`: Get an active order by its ticket
- `history_orders_get(symbol, group, ticket, position, from_date, to_date)`: Get orders from history
- `history_deals_get(symbol, group, ticket, position, from_date, to_date)`: Get deals from history

## Example Workflows

### Connecting and Getting Market Data

```python
# Initialize MT5
initialize()

# Log in to your trading account
login(account=123456, password="your_password", server="your_server")

# Get available symbols
symbols = get_symbols()

# Get recent price data for EURUSD
rates = copy_rates_from_pos(symbol="EURUSD", timeframe=15, start_pos=0, count=100)

# Shut down the connection
shutdown()
```

### Placing a Trade

```python
# Initialize and log in
initialize()
login(account=123456, password="your_password", server="your_server")

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

# Shut down the connection
shutdown()
```

## Resources

The server provides the following resources to help AI assistants understand how to use the MetaTrader 5 API:

- `mt5://getting_started`: Basic workflow for using the MetaTrader 5 API
- `mt5://trading_guide`: Guide for placing and managing trades
- `mt5://market_data_guide`: Guide for accessing and analyzing market data
- `mt5://order_types`: Information about order types
- `mt5://order_filling_types`: Information about order filling types
- `mt5://order_time_types`: Information about order time types
- `mt5://trade_actions`: Information about trade request actions

## Prompts

The server provides the following prompts to help AI assistants interact with users:

- `connect_to_mt5(account, password, server)`: Connect to MetaTrader 5 and log in
- `analyze_market_data(symbol, timeframe)`: Analyze market data for a specific symbol
- `place_trade(symbol, order_type, volume)`: Place a trade for a specific symbol
- `manage_positions()`: Manage open positions
- `analyze_trading_history(days)`: Analyze trading history

## Development

### Project Structure

```
mcp-metatrader5-server/
├── src/
│   └── mcp_mt5/
│       ├── __init__.py      # Entry point with main()
│       ├── main.py          # FastMCP server with all tools
│       └── test_client.py   # Test client for development
├── docs/
│   ├── getting_started.md
│   ├── market_data_guide.md
│   ├── trading_guide.md
│   └── publishing.md
├── .env                     # Environment configuration (create from .env.example)
├── README.md
├── pyproject.toml           # Project metadata (using hatchling)
└── uv.lock                  # Dependency lock file
```

### Building the Package

Using uv (recommended):

```bash
uv build
```

This will create wheel and source distributions in the `dist/` directory.

### Publishing to PyPI

Using uv:

```bash
# Build first
uv build

# Publish to PyPI
uv publish

# Or publish to TestPyPI first
uv publish --publish-url https://test.pypi.org/legacy/
```

## License

MIT

## Acknowledgements

- [MetaQuotes](https://www.metaquotes.net/) for the MetaTrader 5 platform
- [FastMCP](https://github.com/jlowin/fastmcp) for the MCP server implementation
# Contributing to MCP MetaTrader 5 Server

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to the project.

## Getting Started

### Prerequisites

- Python 3.11 or higher
- [uv](https://docs.astral.sh/uv/) package manager
- MetaTrader 5 terminal (for integration testing)
- Git

### Development Setup

1. **Fork and clone the repository**

```bash
git clone https://github.com/YOUR_USERNAME/mcp-metatrader5-server.git
cd mcp-metatrader5-server
```

2. **Install dependencies**

```bash
# Install all dependencies including dev and docs
uv sync --all-extras
```

3. **Set up environment**

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your MT5 configuration
```

4. **Run tests to verify setup**

```bash
uv run pytest -m unit
```

## Development Workflow

### Creating a Branch

```bash
# Create a new branch for your feature or fix
git checkout -b feature/your-feature-name
# or
git checkout -b fix/issue-description
```

### Making Changes

1. Make your changes in the appropriate files
2. Add tests for new functionality
3. Update documentation if needed
4. Run tests to ensure everything works

### Running Tests

```bash
# Run all unit tests
uv run pytest -m unit

# Run with coverage
uv run pytest --cov=mcp_mt5 --cov-report=html

# Run specific test file
uv run pytest tests/test_timeframes.py -v

# Run linting
uvx ruff check src/ tests/
uvx ruff format src/ tests/
```

### Code Style

We use [ruff](https://docs.astral.sh/ruff/) for linting and formatting:

```bash
# Check code style
uvx ruff check src/ tests/

# Format code
uvx ruff format src/ tests/

# Fix auto-fixable issues
uvx ruff check --fix src/ tests/
```

### Writing Tests

- Place tests in the `tests/` directory
- Use the `@pytest.mark.unit` marker for unit tests
- Use the `@pytest.mark.integration` marker for tests requiring MT5
- Follow the FastMCP testing patterns (see `tests/README.md`)

Example test:

```python
import pytest
from fastmcp import Client
from mcp_mt5.main import mcp

@pytest.mark.unit
async def test_my_feature():
    """Test description."""
    async with Client(mcp) as client:
        result = await client.call_tool("my_tool", {"param": "value"})
        assert result.data == expected_value
```

### Documentation

- Update docstrings for any new or modified functions
- Update `docs/` if adding new features
- Update `README.md` if changing installation or usage

Build docs locally:

```bash
uv run mkdocs serve
# Visit http://127.0.0.1:8000
```

## Submitting Changes

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `test:` Adding or updating tests
- `refactor:` Code refactoring
- `ci:` CI/CD changes
- `chore:` Maintenance tasks

Examples:

```bash
git commit -m "feat: add support for custom timeframes"
git commit -m "fix: handle MT5 connection timeout"
git commit -m "docs: update installation instructions"
git commit -m "test: add tests for order validation"
```

### Creating a Pull Request

1. **Push your branch**

```bash
git push origin feature/your-feature-name
```

2. **Create a Pull Request on GitHub**

- Go to the repository on GitHub
- Click "New Pull Request"
- Select your branch
- Fill out the PR template
- Link any related issues

3. **Wait for Review**

- CI tests will run automatically
- Address any feedback from reviewers
- Make additional commits if needed

### Pull Request Checklist

- [ ] Tests pass locally
- [ ] New tests added for new functionality
- [ ] Documentation updated
- [ ] Code follows project style
- [ ] Commit messages follow conventions
- [ ] PR description is clear and complete

## Types of Contributions

### Bug Reports

- Use the bug report template
- Include steps to reproduce
- Provide error messages and logs
- Specify your environment details

### Feature Requests

- Use the feature request template
- Explain the use case
- Describe the proposed solution
- Consider implementation details

### Code Contributions

- Fix bugs
- Add new features
- Improve performance
- Enhance documentation
- Add tests

### Documentation

- Fix typos or unclear explanations
- Add examples
- Improve API documentation
- Write tutorials or guides

## Code Review Process

1. **Automated Checks**
   - CI tests must pass
   - Code coverage should not decrease significantly
   - Linting checks must pass

2. **Manual Review**
   - Code quality and style
   - Test coverage
   - Documentation completeness
   - Breaking changes consideration

3. **Approval and Merge**
   - At least one approval required
   - All conversations resolved
   - CI passing
   - No merge conflicts

## Release Process

Releases are managed by maintainers:

1. Update version in `pyproject.toml`
2. Update `CHANGELOG.md`
3. Create a git tag
4. Push tag to trigger release workflow
5. GitHub Actions publishes to PyPI

## Getting Help

- 📚 [Documentation](https://mcp-metatrader5-server.readthedocs.io)
- 💬 [GitHub Discussions](https://github.com/Qoyyuum/mcp-metatrader5-server/discussions)
- 🐛 [Issue Tracker](https://github.com/Qoyyuum/mcp-metatrader5-server/issues)

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn and grow
- Follow GitHub's [Community Guidelines](https://docs.github.com/en/site-policy/github-terms/github-community-guidelines)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Questions?

Feel free to open an issue or start a discussion if you have any questions!
