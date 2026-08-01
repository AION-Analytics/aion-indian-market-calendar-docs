# Changelog

Release history for [`aion-indian-market-calendar`](https://pypi.org/project/aion-indian-market-calendar/).

This file is generated from the changelog section of the package README published to PyPI, so the
two cannot drift. PyPI is the single source of installation; this repository is documentation-only.

---

### v1.3.3 — Data coverage range (2026-08-01)

**Fixed**

- **The bundled data declared coverage to 2027-03-24 but contains no 2027 events.** Querying a 2027
  date returned a confident wrong answer — `is_trading_day` reported Republic Day 2027 as a trading
  day, with no error and no warning, because outside the data range the logic degrades to
  weekday-only. The declared range now matches the data actually present (2026-01-01 to 2026-12-25).

**Added**

- `calendar.coverage_start`, `calendar.coverage_end` and `calendar.covers(day)` so callers can check
  the range before trusting an answer.
- A one-time warning when a queried date falls outside the bundled range, so the degraded answer is
  visible rather than silent. Answers are unchanged; only the logging is new.

```python
cal = IndiaMarketCalendar.bundled(2026)
cal.coverage_start, cal.coverage_end   # date(2026, 1, 1), date(2026, 12, 25)
cal.covers(date(2027, 1, 26))          # False — do not trust an answer for this date
```

Note that 2027 data is not yet bundled: exchanges publish the next year's holiday list towards the
end of the preceding year.

### v1.3.2 — 2026 holiday audit (2026-08-01)

All 2026 dates were re-verified against the published NSE, BSE and MCX calendars. **The 16 equity
holidays matched exactly, including every remaining 2026 date**, as did the four extra bank-linked
holidays on CDS / interest-rate derivatives / corporate bonds.

**Fixed**

- **1 January 2026 was inverted.** `MCX` and `NSE_COMMODITY_DERIVATIVES` recorded it as a full-day
  closure. The morning session trades and only the **evening** is shut, tracking global commodity
  markets. MCX full closures are now four dates, not five.
- **Muhurat Trading was missing.** Diwali Laxmi Pujan falls on **Sunday 8 November 2026** and the
  Muhurat session runs that day, but there was no NSE or BSE record at all and the MCX record carried
  no session — so `is_trading_day` returned `False` for the whole day and
  `events_on("2026-11-08", exchange="NSE")` returned an empty list, the exact failure this package
  documents as a reason not to use a generic calendar. NSE, BSE and MCX Muhurat sessions are now
  present and the Sunday resolves as a trading day.
- Corrected the MCX source URL, which had `survelliance` misspelled.

**Provisional data**

- The Muhurat window is recorded as **18:15 – 19:15**, the long-standing convention. The exchange
  circular fixing the 2026 timing is published a few weeks before Diwali, so these records carry
  `metadata.timings_pending_exchange_circular` and `metadata.timings_provisional`. The **date** is
  confirmed; the hour is not. Refresh via the live-events channel to pick up the final timing.

### v1.3.1 — Currency derivatives segment fix (2026-08-01)

**Fixed**

- **`NSE_CURRENCY_DERIVATIVES` never resolved.** The segment was declared in the alias table and had
  its own bundled sessions and 20 holidays, but was never registered as a canonical market, so every
  lookup raised `ValueError: Unknown market input`. `holidays("CDS")`, `is_market_open("USDINR")` and
  `get_session("NSE_CURRENCY_DERIVATIVES")` all failed — including the calls in this README's own
  quick start. Canonical segments are now the union of those with default sessions and those declared
  in the alias table, so a segment declared either way resolves.
- Registered the CDS default session (09:00 – 17:00) alongside its bundled one.

**Added**

- Regression coverage asserting every declared segment resolves, that NSE and BSE equity holidays are
  identical, that the bank-linked segments are a strict superset of the equity list, and that MCX
  evening sessions run on equity-holiday mornings.

### v1.3.0 — SEBI circular alignment, BSE segments (2026-08-01)

Verified against **SEBI circular HO/47/11/11(3)2025-MRD-POD2/I/2765/2026 dated 16 January 2026**.

> **Corrects v1.2.0.** v1.2.0 ended the Closing Auction Session at **15:30**. Para 4.2.1 defines it
> as *"a separate session of 20 minutes from 3:15 pm to 3:35 pm"*, so v1.2.0 reported F&O underlying
> stocks as closed between 15:30 and 15:35 when the auction was still running. **Upgrade from
> v1.2.0.**

**Fixed**

- Closing Auction Session now ends **15:35**, not 15:30 (para 4.2.1).
- Post-close session moved to **15:50 – 16:00**, from 15:40 – 16:00 (para 4.2.4).
- `session_rules` keys on the live-refresh wire are now alias-resolved, so a payload keyed `"BSE"` or
  `"NFO"` reaches the canonical segment instead of being stored under an unread key.

**Added**

- Split BSE into `BSE_EQUITY`, `BSE_EQUITY_FNO_UNDERLYING`, and `BSE_EQUITY_DERIVATIVES`, since the
  circular is addressed to all recognised stock exchanges. `BSE` remains an alias for `BSE_EQUITY`,
  so existing callers are unaffected. New aliases: `BFO`, `BSE_FNO`, `BSE_CASH_FNO`.
- `SENSEX` and `BANKEX` now resolve to `BSE_EQUITY_DERIVATIVES` (previously the undivided `BSE`),
  matching how `NIFTY` and `BANKNIFTY` resolve. Index derivatives have no auction and trade to 15:40
  (para 4.2.3).
- BSE equity holidays, mirrored from the NSE equity trading-holiday list with per-record provenance
  in `metadata.bse_mirrored_from_nse`. Previously **no** BSE holiday data was bundled, so every BSE
  weekday — including Christmas — was reported as a trading day.
- Revised Pre-Open Auction Session phases effective **2026-09-07** (para 6.2) in `market_timings`:
  order entry to 09:10 with a random close 09:08 – 09:10, matching 09:10 – 09:12, transition
  09:12 – 09:15. The 09:00 – 09:15 window itself is unchanged.
- Full CAS phase detail in `market_timings`: the four sessions, the 15:00 – 15:15 reference-price
  window, and the ±3% band (paras 4.2.1, 4.3.1, 4.4.1).

### v1.2.0 — Closing Auction Session (2026-08-01)

Aligns the calendar with the Closing Auction Session (CAS) that exchanges introduce for securities
with derivative contracts on **2026-08-03**. No breaking API changes.

**Added**

- `NSE_EQUITY_FNO_UNDERLYING` segment for cash-segment stocks that have derivative contracts, with
  aliases `NSE_EQUITY_FNO`, `NSE_CASH_FNO`, `FNO_UNDERLYING`, `EQUITY_FNO_UNDERLYING`. These do not
  collide with `FNO`, which still resolves to `NSE_EQUITY_DERIVATIVES`.
- `SessionSegment.kind` (`"continuous"` | `"closing_auction"`) and the `is_continuous` property.
  Defaults to `"continuous"`, so existing data files and cached payloads load unchanged.
- `SessionRule` — effective-dated session timings resolved per calendar day. Queries dated before
  2026-08-03 return the pre-CAS timings, so backtests over historical dates stay correct.
- `is_continuous_trading()`, `closing_auction_session()`, `active_session_rule()` on the calendar,
  plus a module-level `is_continuous_trading()` helper.
- `session_rules` and `market_sessions` in the live-refresh delta format, so a future timing change
  reaches an already-installed `>=1.2.0` package without a release. Cached to
  `~/.aion_indian_market/live_cache.json` and restored offline.

**Changed**

- Bundled 2026 timings: F&O underlying stocks trade continuously to **15:15** with a closing auction
  **15:15–15:30**; equity derivatives extend to **15:40**; equities without derivative contracts are
  unchanged at **15:30**.
- `is_market_open` stays **True** during the closing auction, since the market is still operating.
  Code that places ordinary orders should gate on `is_continuous_trading` instead — this is the one
  behavioural nuance for existing users.
- `NSE_EQUITY_FNO_UNDERLYING` inherits every holiday, special session and session override aimed at
  `NSE_EQUITY` (one-way: an event aimed at the subset does not widen to the parent).
- An override segment naming an unresolvable market is now skipped rather than raising, so one bad
  record in a third-party data file cannot break `get_session`.

**Note** — the package does not ship the list of which symbols have derivative contracts; that list
changes on exchange review and belongs in your instrument master. Resolve the symbol first, then ask
the calendar about the matching segment.

### v1.1.4

Migrated to src-layout. Canonical module renamed from `_calendar` to `calendar`. Added a
privacy-safe anonymous install ID for live-refresh telemetry.

### v1.1.3

Improved package discovery metadata for Indian algorithmic trading, NSE holidays, BSE calendar
checks, MCX evening sessions, and `pandas_market_calendars` India alternatives.

### v1.1.2

Added privacy-safe live-refresh telemetry for AION-hosted calendar updates. Bundled/offline use
remains silent.

### v1.1.1

Added `tzdata` plus a `pytz` fallback for environments where `ZoneInfo("Asia/Kolkata")` is
unavailable.

### v1.1.0

Fixed incorrect market resolution for `NFO` and common index inputs.
