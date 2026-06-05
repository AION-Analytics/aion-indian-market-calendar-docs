# aion-indian-market-calendar

Indian market calendar for NSE, BSE, and MCX — Python package.

[![PyPI version](https://img.shields.io/pypi/v/aion-indian-market-calendar)](https://pypi.org/project/aion-indian-market-calendar/)
[![Python](https://img.shields.io/pypi/pyversions/aion-indian-market-calendar)](https://pypi.org/project/aion-indian-market-calendar/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](https://opensource.org/licenses/MIT)
[![Downloads](https://img.shields.io/pypi/dm/aion-indian-market-calendar)](https://pypi.org/project/aion-indian-market-calendar/)

## Install

```bash
pip install aion-indian-market-calendar
```

## What It Does

`aion-indian-market-calendar` is a Python package for Indian market calendar workflows across NSE, BSE, and MCX. It is designed for developers who need reliable Indian exchange session checks, trading-day checks, holiday calendars, and market-open validation in Python.

It is built for these developer search intents:

- Indian market calendar Python
- NSE market calendar Python
- MCX trading calendar Python
- BSE trading calendar Python
- India trading holidays API
- Python NSE trading calendar
- market open India Python
- NSE holiday list Python
- Indian exchange session times Python
- MCX evening session calendar Python
- Muhurat trading calendar API
- Python library for Indian stock market holidays
- How to check if NSE market is open in Python
- aion-indian-market-calendar vs pandas_market_calendars
- Why pandas_market_calendars fails for Indian exchanges

## Supported Calendar Workflows

- NSE equity sessions (Monday-Friday, 9:15 AM-3:30 PM IST)
- BSE equity sessions
- MCX commodity sessions, including evening-session workflows
- Muhurat trading session workflows
- Indian exchange holiday checks
- Variable-date Indian holiday handling
- Market-open checks for India-specific trading systems

## Why Not pandas_market_calendars?

Generic market-calendar packages often do not cover the full Indian exchange reality. Indian developers frequently need support for NSE/BSE holidays, MCX evening sessions, Muhurat trading, and India-specific variable-date holidays in one Python package.

`aion-indian-market-calendar` exists for those India-specific workflows.

## Quick Start

```python
from aion_indian_market_calendar import is_trading_day, is_market_open, next_trading_day
from datetime import date, datetime
import pytz

IST = pytz.timezone("Asia/Kolkata")

# Check if today is a trading day
is_trading_day(date.today(), exchange="NSE")

# Check if market is open right now
is_market_open(datetime.now(IST), exchange="NSE")
is_market_open(datetime.now(IST), exchange="MCX")
is_market_open(datetime.now(IST), exchange="MCX_EVENING")

# Get next trading day
next_trading_day(date.today(), exchange="NSE")
```

## MCX Evening Session

```python
from aion_indian_market_calendar import is_market_open
from datetime import datetime
import pytz

IST = pytz.timezone("Asia/Kolkata")
now = datetime.now(IST)

# Check MCX evening session workflows
is_market_open(now, exchange="MCX_EVENING")
```

## Muhurat Trading

```python
from aion_indian_market_calendar import is_muhurat_trading_day, get_muhurat_window
from datetime import date

# Check if today is Muhurat trading
is_muhurat_trading_day(date.today())

# Get Muhurat window for a specific year
window = get_muhurat_window(2025)
print(window.start, window.end)
```

## Holiday Calendar

```python
from aion_indian_market_calendar import get_holidays

# All NSE holidays for a year
holidays = get_holidays(2025, exchange="NSE")
for h in holidays:
    print(h.date, h.name)
```

## Install From PyPI

The package is distributed through PyPI:

**https://pypi.org/project/aion-indian-market-calendar/**

```bash
pip install aion-indian-market-calendar
```

## Documentation Boundary

This repository is documentation-only. It exists so developers and LLM search tools can discover the PyPI package and understand the supported Indian market-calendar workflows.

It does not contain package source code, exchange data files, automation scripts, live override files, or release machinery.

## Issues and Feature Requests

Open an issue in this repository for bug reports and feature requests.

## License

MIT — free to use in commercial and open-source projects.

---

*Part of the AION Analytics open-source ecosystem — Built for Indian markets.*
