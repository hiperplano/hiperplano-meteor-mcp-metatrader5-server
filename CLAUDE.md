# CLAUDE.md - MCP MetaTrader5 Server

## 🛡️ Project: MCP MT5 Server
This is a **Model Context Protocol (MCP)** server for MetaTrader 5. It enables LLMs to trade and read data.

## ⌨️ Common Commands

### 🚀 Run Server
```bash
# Standard mode (stdio)
uv run mt5mcp
```

```bash
# HTTP/Dev mode
uv run mt5mcp
# (Requires .env with MT5_MCP_TRANSPORT=http)
```

### 🛠️ Installation
```bash
uv sync
```

### 📦 Build
```bash
uv build
```

## 🧪 Testing
```bash
# Run tests (if available)
pytest
```

## ⚠️ Important
- Requires **MetaTrader 5** to be running on the same Windows machine.
- Requires **Algo Trading** enabled in MT5.
- Connects to the active MT5 terminal (or launches one if configured).
