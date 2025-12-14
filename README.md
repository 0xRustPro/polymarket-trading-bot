# Polymarket Copy Trading Bot

[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg)](LICENSE)
[![Node.js Version](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen.svg)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-blue.svg)](https://www.typescriptlang.org/)

> Enterprise-grade automated copy trading system for Polymarket. Automatically replicate trades from top-performing traders with intelligent position sizing, risk management, and real-time execution.

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Deployment](#deployment)
- [Documentation](#documentation)
- [Security](#security)
- [Contributing](#contributing)
- [License](#license)

## Overview

The Polymarket Copy Trading Bot is a production-ready TypeScript application that enables automated replication of trading strategies from successful Polymarket traders. The system monitors trader activity in real-time, calculates optimal position sizes based on your capital allocation, and executes matching orders with built-in risk management and slippage protection.

### How It Works

1. **Trader Selection** - Identify top performers using Polymarket leaderboard analytics
2. **Continuous Monitoring** - Real-time polling of trader positions via Polymarket Data API
3. **Intelligent Sizing** - Dynamic position sizing based on capital ratios and risk parameters
4. **Order Execution** - Automated order placement through Polymarket CLOB API
5. **Performance Tracking** - Comprehensive trade history and analytics stored in MongoDB

## Key Features

### Core Functionality

- **Multi-Trader Support** - Simultaneously track and replicate trades from multiple traders
- **Advanced Position Sizing** - Three strategies: Percentage-based, Fixed amount, and Adaptive scaling
- **Tiered Multipliers** - Apply different risk multipliers based on trade size ranges
- **Trade Aggregation** - Intelligent batching of small trades to optimize execution and reduce fees
- **Real-Time Execution** - Sub-second trade detection and execution latency

### Risk Management

- **Slippage Protection** - Automatic price validation before order execution
- **Position Limits** - Configurable maximum order size and position size constraints
- **Balance Protection** - Automatic order size reduction when insufficient funds
- **Minimum Order Validation** - Prevents execution of orders below exchange minimums

### Data & Analytics

- **MongoDB Integration** - Persistent storage of all trades, positions, and historical data
- **Position Tracking** - Accurate tracking of purchases and sales across balance changes
- **Performance Analytics** - Built-in scripts for profitability analysis and trader evaluation
- **Health Monitoring** - Comprehensive health check system for system validation

## Architecture

### System Components

```
┌─────────────────┐
│  Trade Monitor  │ → Polls Polymarket Data API
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Trade Executor │ → Calculates positions & executes orders
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  MongoDB Store   │ → Persistent data storage
└─────────────────┘
```

### Technology Stack

- **Runtime:** Node.js 18+ with TypeScript
- **Blockchain:** Polygon Network (Ethereum-compatible)
- **Database:** MongoDB with Mongoose ODM
- **APIs:** Polymarket CLOB Client, Polymarket Data API
- **Web3:** ethers.js v5 for blockchain interactions

## Prerequisites

Before installation, ensure you have:

- **Node.js** v18.0.0 or higher
- **MongoDB** database (local or [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register) free tier)
- **Polygon Wallet** with:
  - USDC balance for trading
  - POL/MATIC for gas fees
- **RPC Endpoint** for Polygon network:
  - [Infura](https://infura.io) (recommended)
  - [Alchemy](https://www.alchemy.com)
  - [Ankr](https://www.ankr.com)

## Installation

### Quick Setup

```bash
# Clone the repository
git clone https://github.com/vladmeer/polymarket-copy-trading-bot.git
cd polymarket-copy-trading-bot

# Install dependencies
npm install

# Run interactive setup wizard
npm run setup

# Build the application
npm run build

# Verify configuration
npm run health-check

# Start the bot
npm start
```

### Manual Configuration

If you prefer manual configuration, create a `.env` file in the root directory:

```env
# Trader Configuration
USER_ADDRESSES='0xABC...,0xDEF...'

# Wallet Configuration
PROXY_WALLET='0x123...'
PRIVATE_KEY='your_private_key_without_0x_prefix'

# Database
MONGO_URI='mongodb+srv://user:pass@cluster.mongodb.net/database'

# Network
RPC_URL='https://polygon-mainnet.infura.io/v3/YOUR_PROJECT_ID'
USDC_CONTRACT_ADDRESS='0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174'

# Polymarket API
CLOB_HTTP_URL='https://clob.polymarket.com/'
CLOB_WS_URL='wss://ws-subscriptions-clob.polymarket.com/ws'

# Trading Strategy
COPY_STRATEGY='PERCENTAGE'
COPY_SIZE='10.0'
MAX_ORDER_SIZE_USD='100.0'
MIN_ORDER_SIZE_USD='1.0'

# Monitoring
FETCH_INTERVAL='1'
RETRY_LIMIT='3'
```

For complete configuration options, see [Configuration Guide](./docs/QUICK_START.md).

## Configuration

### Copy Trading Strategies

The bot supports three copy trading strategies:

#### 1. Percentage Strategy
Replicates a fixed percentage of each trader's order size.

```env
COPY_STRATEGY='PERCENTAGE'
COPY_SIZE='10.0'  # Copy 10% of trader's order
```

#### 2. Fixed Strategy
Executes a fixed dollar amount per trade, regardless of trader's order size.

```env
COPY_STRATEGY='FIXED'
COPY_SIZE='50.0'  # Always trade $50
```

#### 3. Adaptive Strategy
Dynamically adjusts percentage based on trade size (higher % for small trades, lower % for large trades).

```env
COPY_STRATEGY='ADAPTIVE'
COPY_SIZE='10.0'
ADAPTIVE_MIN_PERCENT='5.0'
ADAPTIVE_MAX_PERCENT='15.0'
ADAPTIVE_THRESHOLD_USD='300.0'
```

### Tiered Multipliers

Apply different risk multipliers based on trade size:

```env
TIERED_MULTIPLIERS='1-10:2.0,10-100:1.0,100-500:0.2,500+:0.1'
```

This configuration:
- Trades $1-$10: 2.0x multiplier
- Trades $10-$100: 1.0x multiplier
- Trades $100-$500: 0.2x multiplier
- Trades $500+: 0.1x multiplier

### Finding Traders

1. Visit [Polymarket Leaderboard](https://polymarket.com/leaderboard)
2. Identify traders with:
   - Positive P&L over extended periods
   - Win rate >55%
   - Active trading history
   - Consistent performance metrics
3. Verify detailed analytics on [Predictfolio](https://predictfolio.com)
4. Add wallet addresses to `USER_ADDRESSES` (comma-separated)

## Usage

### Starting the Bot

```bash
# Development mode
npm run dev

# Production mode
npm run build
npm start
```

### Utility Scripts

The bot includes comprehensive utility scripts:

#### Trading Operations
```bash
npm run manual-sell          # Manually sell positions
npm run sell-large           # Sell large positions
npm run close-stale          # Close stale positions
npm run close-resolved       # Close resolved markets
npm run redeem-resolved      # Redeem resolved positions
```

#### Analysis & Simulation
```bash
npm run simulate             # Simulate profitability
npm run find-traders         # Find best traders
npm run find-low-risk         # Find low-risk traders
npm run check-stats          # Check your trading stats
npm run check-pnl            # Check P&L discrepancies
```

#### Configuration & Maintenance
```bash
npm run setup                # Interactive setup wizard
npm run health-check         # Verify system configuration
npm run check-allowance      # Check token allowances
npm run set-token-allowance  # Set token allowances
```

## Deployment

### Docker Deployment

Deploy using Docker Compose for production environments:

```bash
# Copy environment template
cp .env.example .env
# Edit .env with your configuration

# Start services
docker-compose up -d

# View logs
docker-compose logs -f polymarket

# Stop services
docker-compose down
```

For detailed Docker deployment instructions, see [Docker Guide](./docs/DOCKER.md).

### Production Considerations

- Use environment-specific configuration files
- Implement proper secret management (AWS Secrets Manager, HashiCorp Vault)
- Set up monitoring and alerting
- Configure log rotation
- Use process managers (PM2, systemd) for reliability
- Implement backup strategies for MongoDB

## Documentation

### Getting Started
- **[Getting Started Guide](./docs/GETTING_STARTED.md)** - Comprehensive setup instructions for beginners
- **[Quick Start Guide](./docs/QUICK_START.md)** - Fast setup for experienced developers

### Advanced Topics
- **[Multi-Trader Guide](./docs/MULTI_TRADER_GUIDE.md)** - Managing multiple traders
- **[Tiered Multipliers](./docs/TIERED_MULTIPLIERS.md)** - Advanced risk management
- **[Simulation Guide](./docs/SIMULATION_GUIDE.md)** - Backtesting and analysis
- **[Position Tracking](./docs/POSITION_TRACKING.md)** - Understanding position management

### Deployment & Operations
- **[Docker Guide](./docs/DOCKER.md)** - Container deployment
- **[Deployment Guide](./docs/DEPLOYMENT.md)** - Production deployment strategies

## Security

### Best Practices

- **Never commit `.env` files** - Already excluded in `.gitignore`
- **Use dedicated trading wallets** - Never use wallets with significant holdings
- **Rotate API keys regularly** - Change Polymarket API credentials periodically
- **Monitor wallet activity** - Set up alerts for unexpected transactions
- **Use read-only database users** - Limit MongoDB permissions when possible
- **Keep dependencies updated** - Regularly update npm packages

### Security Features

- Environment variable validation
- Input sanitization for addresses and configuration
- Error handling to prevent information leakage
- Secure private key handling (never logged or transmitted)

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Setup

```bash
# Install dependencies
npm install

# Run tests
npm test

# Run linter
npm run lint

# Format code
npm run format
```

## License

This project is licensed under the ISC License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built on [Polymarket CLOB Client](https://github.com/Polymarket/clob-client)
- Trader analytics powered by [Predictfolio](https://predictfolio.com)
- Deployed on Polygon network

## Disclaimer

**IMPORTANT:** This software is provided for educational and research purposes only. Trading cryptocurrencies and prediction markets involves substantial risk of loss. The developers and contributors are not responsible for any financial losses incurred while using this bot. Always:

- Trade with funds you can afford to lose
- Understand the risks involved
- Test thoroughly before deploying real capital
- Monitor your bot's activity regularly
- Use appropriate risk management strategies

---

**Support:** For questions, issues, or contributions, please open an issue on GitHub or contact the maintainers.
