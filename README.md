# arbitrazs-bot

A Python-based sports betting automation bot built as a freelance project and portfolio reference. Monitors a Telegram channel for arbitrage (surebet) signals and automatically places bets on two bookmakers simultaneously using browser automation.

This is not a proof-of-concept. The bot handles real money, real accounts, and real time pressure — arbitrage windows can close in under 60 seconds.

---

## What It Does

Arbitrage betting exploits odds discrepancies between bookmakers: by placing opposing bets on the same event at two different sites, a guaranteed profit is locked in regardless of the outcome.

The bot automates the entire pipeline:

1. Listens to a Telegram channel for surebet signals ("A" scanner format)
2. Filters signals below 4.5% profit threshold
3. Calculates optimal stake distribution for a 30,000 HUF total bet
4. Opens both bookmaker URLs in parallel headless browsers
5. Logs in, finds the market, enters the stake, and places the bet simultaneously
6. Handles A/B account switching on limit hits
7. Validates odds before placing (±0.02 tolerance)
8. Logs all activity with rotating file output

---

## Stack

| Layer | Technology |
|---|---|
| Language | Python 3 |
| Browser Automation | Selenium (headless Chromium) |
| Telegram Listener | Telethon (MTProto API) |
| Parallelism | asyncio + threading |
| Config | YAML (human-editable, no dev skills needed) |
| Launcher | Windows `.bat` script |
| OS Target | Windows 10/11 |

---

## Supported Bookmakers

| Bookmaker | Status |
|---|---|
| Vegas.hu | ✅ Full pipeline tested (dry-run) |
| Boabet | ✅ Full pipeline tested (dry-run) |
| Sportaza | ✅ Full pipeline tested (dry-run) |
| Betinia | ✅ Full pipeline tested (dry-run) |
| Piwi247 | ⏳ Live test pending (geo-blocked in WSL2, works on Windows) |
| Tippmixpro | ✅ Full pipeline tested (dry-run) |

---

## Repository Structure

```
arbitrazs-bot/
├── main.py                  # Entry point (--dry-run / --test flags)
├── config.yaml              # All user-editable settings
├── indit.bat                # Windows launcher
├── requirements.txt
├── src/
│   ├── parser.py            # Telegram message parser (regex, 3 format variants)
│   ├── calculator.py        # Stake distribution calculator
│   ├── telegram_listener.py # Telethon-based channel monitor
│   ├── bet_placer.py        # Parallel bet execution engine
│   ├── browser.py           # Selenium driver factory (WSL2-aware)
│   ├── event_search.py      # Event matching and outcome normalization
│   ├── outcome_mapper.py    # Outcome label normalization (HU/EN)
│   └── logger.py            # Rotating file + console logging
└── bookmakers/
    ├── base.py              # BaseBookmaker, BetResult, A/B switching
    ├── vegas.py
    ├── boabet.py
    ├── sportaza.py
    ├── betinia.py
    ├── piwi247.py
    └── tippmix.py
```

---

## Project Status

| Component | Status |
|---|---|
| Project structure | ✅ Complete |
| Telegram parser (A scanner) | ✅ Complete |
| Stake calculator | ✅ Complete |
| Selenium browser factory | ✅ Complete |
| Vegas.hu — full pipeline dry-run | ✅ Complete |
| Boabet — full pipeline dry-run | ✅ Complete |
| Sportaza — full pipeline dry-run | ✅ Complete |
| Betinia — full pipeline dry-run | ✅ Complete |
| Piwi247 — selectors mapped | ✅ Complete |
| Tippmixpro — selectors mapped | ✅ Complete |
| Parallel bet execution | ✅ Complete |
| A/B account switching logic | ✅ Complete |
| Odds validation pre-bet | ✅ Complete |
| Error handling (limit, missing market, timeout) | ✅ Complete |
| End-to-end dry-run test (5/6 irodán) | ✅ Complete |
| User documentation | ✅ Complete |
| Piwi247 live test | ⏳ Windows live test pending |
| Tippmix live test | ✅ Complete |
| Live test with client | ⏳ Pending |

---

## E2E Dry-Run Results

Latest end-to-end dry-run test (parallel mode, 2026-06-09):

| Bookmaker | Result | Time | Notes |
|---|---|---|---|
| Tippmix Pro | ✅ OK | 46.6s | Login → event search → odds button → stake entered |
| Vegas.hu | ✅ OK | 70.9s | Login → event search → odds button → stake entered |
| Boabet | ✅ OK | 37.3s | API login → outcome selected → stake entered |
| Sportaza | ✅ OK | 56.9s | API login → outcome selected → stake entered |
| Betinia | ✅ OK | 52.6s | API login → outcome selected → stake entered |
| Piwi247 | ❌ Login sikertelen | 3.0s | Geo-blocked in WSL2 — expected, works on Windows |

Parse + calculation pipeline: ✅ Perfect

---

## Configuration

All settings live in `config.yaml` — editable with any text editor, no development environment needed.

```yaml
telegram:
  api_id: YOUR_API_ID
  api_hash: YOUR_API_HASH
  phone: +36...
  source_channels:
    - "@scanner_channel"

betting:
  total_stake: 30000
  min_profit_percent: 4.5
  odds_tolerance: 0.02
  dry_run: true          # set to false for live betting

bookmakers:
  vegas:
    enabled: true
    accounts:
      - username: email@example.com
        password: yourpassword
      - username: email2@example.com
        password: yourpassword2
  # ... same structure for all bookmakers
```

---

## Running the Bot

```
indit.bat                # Live mode
indit.bat --dry-run      # Logs everything, places no real bets
indit.bat --test         # Parser and calculator test, no Telegram needed
```

---

## Why This Project

Built as a freelance delivery for a client running a manual arbitrage operation. The goal: eliminate human reaction time as the bottleneck in a time-critical, multi-site workflow.

Key engineering challenges:
- Parallel browser sessions with synchronized timing across multiple bookmakers
- Bookmaker-specific DOM navigation: iframe switching (Tippmix), Shadow DOM (Boabet), Angular/React input handling (Sportaza, Betinia)
- API-based login with session cookie injection where UI login was unreliable in headless mode
- Graceful degradation when one side of a bet fails (value bet logging, not panic)
- A/B account switching logic: switches on limit/auth errors, not on missing market or odds drift
- Odds validation with configurable tolerance per bookmaker

---

## Notes

- Client credentials are excluded via `.gitignore` — `config.yaml` is never committed
- B scanner (image-based signals) is out of scope for this version
- Balance management (deposits/withdrawals) is out of scope
- Tippmixpro requires verified Hungarian identity — client handles this account directly
- Piwi247 is geo-blocked from WSL2 but works on Windows natively
