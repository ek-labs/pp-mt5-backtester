# pp-mt5 · MT5 Backtester CLI

> Printed by Printing Press — printingpress.dev

Run MetaTrader 5 Expert Advisor backtests from the command line.
Compile MQL5 source files. Parse and compare reports. Batch test across symbols and timeframes.

---

## Install

### Windows (PowerShell) — recommended

```powershell
# 1. Install Go if not present
winget install GoLang.Go

# 2. Clone and build
git clone https://github.com/mvanhorn/printing-press-library
cd printing-press-library\library\finance\mt5-backtester
go install ./cmd/pp-mt5-backtester

# 3. Set your MT5 path once
$env:MT5_PATH = "C:\Program Files\MetaTrader 5"
[System.Environment]::SetEnvironmentVariable("MT5_PATH","C:\Program Files\MetaTrader 5","User")
```

### Build from source

```powershell
git clone https://github.com/mvanhorn/printing-press-library
cd printing-press-library\library\finance\mt5-backtester
go build -o pp-mt5.exe ./cmd/pp-mt5-backtester
```

---

## Requirements

- **Windows** (MetaTrader 5 only runs on Windows)
- **MetaTrader 5** installed with at least one broker account logged in
- **Go 1.22+** (only needed to build)
- MT5 terminal must have history downloaded for the symbols you test

---

## Quick Start

```powershell
# Check that pp-mt5 finds your terminal
pp-mt5 config

# Run a backtest on the built-in MACD Sample EA
pp-mt5 run --ea "MACD Sample" --symbol EURUSD --period H1 --from 2023.01.01 --to 2024.01.01

# Compile your EA first, then run it
pp-mt5 compile C:\MT5\MQL5\Experts\MyEA.mq5
pp-mt5 run --ea MyEA --symbol EURUSD --period M15 --model 0

# Parse a report file you already have
pp-mt5 report C:\Users\You\AppData\Roaming\MetaQuotes\Terminal\<hash>\tester\MyReport.htm

# Batch test across multiple symbols
pp-mt5 template                  # creates batch.json
pp-mt5 batch --file batch.json
```

---

## Commands

### `pp-mt5 run` — Run a backtest

```
pp-mt5 run [flags]

Flags:
  --ea string          EA name or path (required)
                       e.g. "MACD Sample", "MyEA", "Experts\MyEA.ex5"
  --symbol string      Trading symbol (default: EURUSD)
  --period string      Timeframe: M1 M5 M15 M30 H1 H4 D1 W1 (default: H1)
  --from string        Start date: YYYY.MM.DD (default: 1 year ago)
  --to string          End date:   YYYY.MM.DD (default: today)
  --model int          Tick model:
                         0 = Every Tick      (most accurate, slowest)
                         1 = 1 Minute OHLC   (good balance, default)
                         2 = Open Prices Only (fastest)
                         4 = Real Ticks       (most realistic)
  --deposit float      Initial deposit (default: 10000)
  --currency string    Deposit currency (default: USD)
  --leverage int       Account leverage (default: 100)
  --input key=value    EA input parameters (repeatable, override set file)
  --set string         .set file path or bare filename (see "Set files" below)
  --timeout duration   Max wait time (default: 4h)
  --no-shutdown        Keep MT5 open after test finishes
  -o, --output string  Output format: text, json, csv (default: text)
```

**Examples:**
```powershell
# Basic
pp-mt5 run --ea "MACD Sample" --symbol EURUSD --period H1 --from 2022.01.01 --to 2024.01.01

# Every tick model, custom deposit
pp-mt5 run --ea MyScalper --symbol GBPUSD --period M5 --model 0 --deposit 50000

# With EA inputs
pp-mt5 run --ea MyEA --symbol EURUSD --period H4 --input StopLoss=50 --input TakeProfit=150

# Drive inputs from a .set file (saved out of the MT5 strategy tester)
pp-mt5 run --ea MyEA --symbol EURUSD --set MyEA.set

# Set file + per-run override (override wins; rest of set file still applied)
pp-mt5 run --ea MyEA --set MyEA.set --input MagicNumber=99002

# JSON output for scripting
pp-mt5 run --ea "MACD Sample" --symbol EURUSD --period H1 --from 2023.01.01 --to 2024.01.01 -o json
```

---

### `pp-mt5 compile` — Compile MQL5 source

```
pp-mt5 compile <file.mq5> [flags]

Flags:
  --inc string   Additional include directory (repeatable)
```

**Examples:**
```powershell
pp-mt5 compile MyEA.mq5
pp-mt5 compile "C:\MT5\MQL5\Experts\MyEA.mq5" --inc "C:\MT5\MQL5\Include\MyLib"
```

---

### `pp-mt5 report` — Parse a report

```
pp-mt5 report [file.htm] [flags]

Flags:
  --dir string   Directory to search for most recent report
  -o string      Output format: text, json, csv
```

**Examples:**
```powershell
pp-mt5 report C:\path\to\report.htm
pp-mt5 report --dir "C:\Users\You\AppData\Roaming\MetaQuotes\Terminal\<hash>\tester"
pp-mt5 report report.htm -o json
pp-mt5 report report.htm -o csv >> results.csv
```

---

### `pp-mt5 batch` — Run multiple backtests

```
pp-mt5 batch --file batch.json [flags]

Flags:
  --file string      Path to batch JSON file (required)
  --timeout duration Timeout per job (default: 4h)
  -o string          Output format: text, json, csv
```

**batch.json format:**
```json
{
  "jobs": [
    {
      "ea": "MACD Sample",
      "symbol": "EURUSD",
      "period": "H1",
      "from": "2023.01.01",
      "to": "2024.01.01",
      "model": 1,
      "deposit": 10000
    },
    {
      "ea": "MACD Sample",
      "symbol": "GBPUSD",
      "period": "H1",
      "from": "2023.01.01",
      "to": "2024.01.01"
    },
    {
      "ea": "MyEA",
      "symbol": "EURUSD",
      "period": "M15",
      "model": 0,
      "deposit": 50000,
      "inputs": {
        "StopLoss": "50",
        "TakeProfit": "100",
        "MagicNumber": "12345"
      }
    },
    {
      "ea": "MyEA",
      "symbol": "EURUSD",
      "set_file": "MyEA.set"
    },
    {
      "ea": "MyEA",
      "symbol": "GBPUSD",
      "set_file": "MyEA.set",
      "inputs": { "MagicNumber": "99002" }
    }
  ]
}
```

Per-job fields:

| Field      | Purpose                                                                 |
|------------|-------------------------------------------------------------------------|
| `set_file` | Resolve a `.set` file (see "Set files" below) and emit `[TesterInputs]` |
| `inputs`   | Override individual params; preserves optimization ranges from the file |

Either field can also go in `defaults` to apply across all jobs.

**Examples:**
```powershell
# Generate a template
pp-mt5 template

# Run the batch
pp-mt5 batch --file batch.json

# Save results as CSV
pp-mt5 batch --file batch.json -o csv > results.csv

# JSON for further processing
pp-mt5 batch --file batch.json -o json | python -m json.tool
```

---

### `pp-mt5 config` — Show detected paths

```powershell
pp-mt5 config
```

---

### `pp-mt5 setfile` — Inspect, generate, and list `.set` files

```
pp-mt5 setfile show <file.set>      # pretty-print params and optimization ranges
pp-mt5 setfile export --output FILE --input KEY=VAL [--input ...]
pp-mt5 setfile list                 # all .set files under MQL5\Profiles\Tester\
```

**Examples:**
```powershell
# Inspect a set file from MQL5\Profiles\Tester\
pp-mt5 setfile show MyEA.set

# Or by absolute path
pp-mt5 setfile show "C:\Users\You\AppData\Roaming\MetaQuotes\Terminal\<hash>\MQL5\Profiles\Tester\MyEA.set"

# Generate a starter .set from inputs (no optimization ranges)
pp-mt5 setfile export --output MyEA.set --input StopLoss=50 --input TakeProfit=100 --input MagicNumber=99001

# List every .set file MT5 can see
pp-mt5 setfile list
```

---

## Set files

A MetaTrader 5 `.set` file is the persisted form of the strategy tester's "Inputs" tab — one parameter per line, in MT5's own pipe-delimited format:

```
; lines starting with ';' are comments
StopLoss=50||50||1||200||N
TakeProfit=100||100||1||500||Y
MagicNumber=99001
UseFilter=true
```

| Field         | Meaning                                       |
|---------------|-----------------------------------------------|
| `value`       | The value used during a normal backtest run.  |
| `opt_start`   | Optimization sweep start.                     |
| `opt_step`    | Optimization sweep step.                      |
| `opt_stop`    | Optimization sweep stop.                      |
| `Y` / `N`     | Whether this param is included in the sweep.  |

Lines without `||` are simple values (no optimization range). Boolean literals (`true` / `false`) are passed through verbatim. Blank lines and `;` comments are skipped.

### How `pp-mt5` uses set files

- `--set FILE` on `run` or `set_file` on a batch job parses the file and emits a proper `[TesterInputs]` block in the generated ini — pipes and Y/N flags preserved.
- `--input KEY=VALUE` (or per-job `inputs`) **overrides** values from the set file by name. Optimization ranges on the original line are preserved; only the value changes.
- If the path is not absolute, `pp-mt5` resolves bare filenames against `MQL5\Profiles\Tester\` under every MT5 data folder on the host. With `--portable`, the terminal's own directory is used instead.

### Workflow

```powershell
# 1. In MT5 Strategy Tester: configure inputs, click "Save" → MyEA.set
# 2. Inspect
pp-mt5 setfile show MyEA.set

# 3. Run with the saved inputs
pp-mt5 run --ea MyEA --symbol EURUSD --set MyEA.set

# 4. Batch the same file across symbols, with one override per job
pp-mt5 batch --file batch.json
```

---

## How it works

MT5 supports a headless CLI mode via `.ini` configuration files:

```powershell
terminal64.exe /config:backtest.ini
```

`pp-mt5` generates these ini files, launches the terminal, waits for completion (MT5 exits when `ShutdownTerminal=1`), then finds and parses the HTML report MT5 writes.

The generated ini looks like this:
```ini
[Tester]
Expert=Experts\MyEA.ex5
Symbol=EURUSD
Period=H1
Model=1
FromDate=2023.01.01
ToDate=2024.01.01
Deposit=10000.00
Currency=USD
Leverage=100
ShutdownTerminal=1
Report=MyEA_EURUSD_H1_20241201
ReplaceReport=1
```

---

## Environment variables

| Variable    | Description                                     |
|-------------|--------------------------------------------------|
| `MT5_PATH`  | MT5 install directory (e.g. `C:\Program Files\MetaTrader 5`) |

---

## Tips

- MT5 must be logged into a broker account for backtests to run (demo accounts work)
- Download history first: open charts for all symbols you want to test
- Every Tick model (`--model 0`) requires more history; start with 1 Minute OHLC (`--model 1`)
- The terminal's log files are in `%APPDATA%\MetaQuotes\Terminal\<hash>\logs\`
- Reports land in `%APPDATA%\MetaQuotes\Terminal\<hash>\tester\`

---

## Roadmap

- [ ] Auto-discover terminal data path (parse `origin.txt`)
- [ ] Optimization mode support with parameter ranges
- [ ] Watch mode (stream log lines during test)
- [ ] SQLite results store (compare runs over time)
- [ ] Walk-forward analysis helper
- [ ] MCP server surface

---

Printed by [Printing Press](https://printingpress.dev) — the agent-designed CLI factory.
