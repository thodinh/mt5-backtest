# MT5 Backtest

GitHub Action to run a MetaTrader 5 strategy-tester backtest on a headless
Wine container and return the report as a Markdown artifact.

Uses the [`mt5cli`](https://github.com/thodinh/mt5-cli) CLI (Bun) and the
public `ghcr.io/thodinh/mt5-exness` image. No npm dependencies beyond the CLI.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `ea` | ✅ | — | Path to the compiled `.ex5` Expert Advisor |
| `symbol` | | `BTCUSDm` | Trading symbol (broker-specific; Exness uses `EURUSDm`, `BTCUSDm`) |
| `period` | | `M1` | Timeframe |
| `from` | ✅ | — | Start date `YYYY.MM.DD` |
| `to` | ✅ | — | End date `YYYY.MM.DD` |
| `deposit` | | `100000` | Initial deposit |
| `currency` | | `USD` | Deposit currency |
| `model` | | `0` | Test model (0 = every tick, 1 = 1-min OHLC) |
| `set` | | | Path to a UTF-16LE `.set` preset |
| `mt5-login` | ✅ | — | MT5 account login |
| `mt5-password` | ✅ | — | MT5 account password |
| `mt5-server` | ✅ | — | MT5 broker server |
| `require-deals` | | `false` | Fail if zero deals were performed |

## Outputs

| Output | Description |
|---|---|
| `report` | Path to the Markdown report |
| `balance` | Final balance |
| `deals` | Total deals |
| `passed` | `yes` / `no` |

## Usage

```yaml
name: backtest
on: [push]

jobs:
  backtest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        # Provide a pre-compiled .ex5 — commit it, or build it in a prior step
        # (e.g. `bun install -g mt5cli && mt5 compile ./MyEA.mq5 --files ...`).

      - name: Run backtest
        id: bt
        uses: thodinh/mt5-backtest@v1
        with:
          ea: ./MyEA.ex5
          symbol: EURUSDm
          period: M15
          from: 2026.01.01
          to: 2026.04.01
          deposit: 100000
          mt5-login: ${{ secrets.MT5_LOGIN }}
          mt5-password: ${{ secrets.MT5_PASSWORD }}
          mt5-server: ${{ secrets.MT5_SERVER }}
          require-deals: 'true'

      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: backtest-report
          path: ${{ steps.bt.outputs.report }}

      - name: Summary
        run: |
          echo "Balance: ${{ steps.bt.outputs.balance }}"
          echo "Deals:   ${{ steps.bt.outputs.deals }}"
          echo "Passed:  ${{ steps.bt.outputs.passed }}"
```

## Notes

- The runner must be Linux (`ubuntu-latest`) with Docker available.
- Credentials are passed via `secrets`; create an MT5 demo account first.
- The first run pulls the ~6 GB MT5 image; subsequent runs are cached by
  GitHub Actions.

## License

MIT
