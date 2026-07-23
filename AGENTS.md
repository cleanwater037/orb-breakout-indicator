# AGENTS.md — Handoff for coding agents

> **You are an AI coding agent (Claude Code, ChatGPT Codex, or similar) picking
> up this repo to add or adjust a TradingView indicator.** Read this whole file
> before touching anything. It encodes the rules that keep the script compiling
> inside TradingView. Break them and the "plugin" (the indicator) fails to load
> and the user sees a red error. You cannot run this code — TradingView is the
> only runtime — so correctness is on you, up front.

---

## 1. What this project is

- A **single-file TradingView indicator** written in **Pine Script v5**:
  `orb_breakout_indicator.pine`.
- **No build system, no dependencies, no tests, no package manager.** The entire
  product is that one `.pine` file. The user copies its text and pastes it into
  TradingView's Pine Editor.
- **You cannot compile or run it locally.** Pine only runs on TradingView's
  servers. There is no CLI, no linter you can invoke, no way to see a chart from
  here. Reason about correctness before you hand anything back, and say honestly
  what needs to be eyeballed on a live chart.
- The current script detects a **30-minute opening-range breakout** on US
  equities/futures. Read the header comment and `README.md` for the trading
  logic; read Section 4 below for the code anatomy.

---

## 2. Working style (the user is not a programmer)

- **Deliver the complete file every time**, never a diff, patch, or "insert this
  snippet." The user pastes the whole thing into a browser text box. A partial
  answer is unusable to them.
- **Explain changes in plain English.** Say what's different and what they'll see
  on the chart. Skip Pine internals unless they ask.
- **State your confidence honestly.** Flag anything you couldn't verify and tell
  them exactly what to look at on the chart to confirm it (e.g. "after 10:00 AM
  ET you should see a second orange line").
- This is a **charting tool**, not trading advice. Don't describe a signal as a
  good or bad trade — only as "the pattern appeared."

---

## 3. Pine Script v5 rules that keep the plugin from breaking

These are the rules most likely to produce a compile error, a runtime error, or
silently wrong output. Treat this as a hard checklist. **Violating any of these
is the usual reason an edit breaks the indicator.**

### 3.1 File-level structure
- The **first non-comment line must be `//@version=5`.** Nothing before it but
  comments.
- There must be **exactly one declaration statement** — `indicator(...)`,
  `strategy(...)`, or `library(...)` — and it must be at the **top level** (not
  inside an `if`, loop, or function). This file uses `indicator(...)`. Do **not**
  add a second one, and do **not** switch to `strategy()` unless the user
  explicitly asks (that's a meaningful rewrite, not a one-word swap).
- Comments are `//` only. There are **no block comments** (`/* */`).

### 3.2 Global-scope-only calls (the #1 thing agents get wrong)
These functions **must be called in the global scope** — never inside an `if`,
`for`, or a user-defined function:

`plot()`, `plotshape()`, `plotchar()`, `plotcandle()`, `plotbar()`, `hline()`,
`fill()`, `bgcolor()`, `barcolor()`, `alertcondition()`, `input.*()`.

To make them **conditional**, don't wrap them in an `if` — instead feed the
condition into their arguments:
```pine
// WRONG — will not compile:
if signal
    plotshape(true, ...)

// RIGHT — pass na / the condition as the series:
plotshape(signal, ...)                 // plots only when `signal` is true
plot(cond ? value : na, ...)           // gap when cond is false
bgcolor(signal ? color.new(color.green, 85) : na)
```

### 3.3 Stateful `ta.*` functions must run every bar
Functions that carry history — `ta.vwap`, `ta.ema`, `ta.sma`, `ta.rsi`,
`ta.atr`, `ta.highest`, `ta.change`, etc. — maintain internal state that only
stays correct if they're **evaluated on every bar**. Call them unconditionally in
the global scope and store the result; **do not call them inside an `if` block**,
or their state desyncs and values go wrong:
```pine
// RIGHT:
vwapValue = ta.vwap(hlc3)              // every bar, global scope
atrValue  = ta.atr(14)
// then use vwapValue / atrValue inside conditions freely
```
Their `length` argument is usually **`simple int`** — pass an `input.int` or a
constant, not a series that changes per bar.

### 3.4 Persistent state: `var`, `=`, `:=`
- `var x = ...` initializes **once** and persists across bars. Use it for
  accumulators and latches (this script uses it for `orHigh`, `firedToday`, etc.).
- Plain `x = ...` **re-initializes every bar** — use for per-bar values.
- `:=` **reassigns** an existing variable. Using `=` when you meant `:=` (or vice
  versa) is a common bug.
- **Ordering of stateful blocks matters.** In this file the `if newDay` reset
  block must run **before** the `if inOR` accumulation block — on the 9:30 bar
  both are true, and reversing them wipes the first bar's data. Preserve intended
  ordering when you edit.

### 3.5 Alerts
- **`alertcondition()` messages must be a compile-time constant string** — no
  dynamic price/symbol/time. This is a hard Pine limit, not an oversight.
- For **dynamic** alert text (live price, ticker, etc.), use **`alert()`**, which
  is an action called in **local scope** (inside an `if`). The user wires it up in
  TradingView via the "Any alert() function call" condition.
- This file intentionally has **both** — `alert()` for rich text and
  `alertcondition()` as a static fallback in the classic dropdown. Don't "clean
  up" by deleting one; they serve different TradingView UI paths.

### 3.6 Types and `na`
- Pine is typed: `int`, `float`, `bool`, `color`, `string`, plus the
  series/simple/const/input qualifiers. **Both branches of a ternary must be the
  same type**; `close > x ? close : na` is fine because `na` adopts the float
  type.
- Guard against `na` before comparing (`not na(orHigh) and close > orHigh`).
  Comparisons against `na` yield `na` (falsey but a frequent logic bug).

### 3.7 Drawing-object limits
- If you add `line.new`, `label.new`, or `box.new`, you can exhaust the default
  cap (50 each). Raise it in the declaration when needed:
  `indicator("...", overlay = true, max_lines_count = 500, max_labels_count = 500)`.
  Also **delete or manage old objects** or you'll hit the ceiling and drawing
  stops.
- Total plots are capped (~64). Don't add plots frivolously.

### 3.8 Sessions, time, repainting
- Session membership: `time(timeframe.period, sessionStr, tz)` returns `na`
  **outside** the session and a timestamp inside it. That's how `inOR` /
  `inWindow` work here.
- `request.security()` for higher-timeframe data can **repaint**. Use
  `barmerge.lookahead_off` and reference confirmed values; warn the user about
  repaint if you introduce it.
- **Intrabar repainting:** any `signal` evaluated on the live (unconfirmed) bar
  can appear and vanish as price moves. To fully confirm, gate on
  `barstate.isconfirmed` — at the cost of a one-bar delay. Alerts should be set to
  "Once Per Bar Close."

### 3.9 Syntax mechanics that cause hard errors
- **Indentation is significant.** Local blocks (inside `if`/`for`/functions) are
  indented consistently (this file uses 4 spaces). Mixed or wrong indentation is
  a syntax error.
- Long function calls wrap by **indenting the continued arguments** deeper than
  the call — see the `str.format(...)` alert message in the file for the pattern.
- v5 namespaces: `math.*`, `str.*`, `ta.*`, `array.*`, `color.*`, `input.*`,
  `request.*`. (In v4 these were bare — don't copy v4 snippets verbatim.)

---

## 4. Code anatomy of the current file

Top to bottom, `orb_breakout_indicator.pine` is:

| Section | What it does |
|---|---|
| `//@version=5` + `indicator(...)` | Version pragma and the one declaration. `overlay = true` draws on the price chart. |
| Inputs | `tz`, two `input.session` strings, `volMult`, two `input.bool` toggles. |
| Day rollover | `dayofmonth(time, tz)` compared to a `var` — timezone-aware daily reset. |
| Session flags | `inOR`, `inWindow` from `time(...)` returning `na` outside session. |
| OR state | Five `var` accumulators reset on `newDay`, accumulated while `inOR`. |
| `orDone` latch | Flips true on the first bar **after** the OR session ends, so breakouts can't fire mid-range. |
| Conditions | Four booleans (`cBreakout`, `cVolume`, `cVwap`, `cWindow`) ANDed into `signal`, gated by the `firedToday` latch. |
| Visuals | OR-high line, optional OR-low, VWAP line, triangle marker, two `bgcolor` calls — all global scope. |
| Alerts | Dynamic `alert()` inside `if signal` + static `alertcondition()` fallback. |

---

## 5. How to add or adjust a feature (the workflow)

1. **Read** `orb_breakout_indicator.pine` in full, plus this file.
2. **Locate the section** from the table in Section 4 that owns the behavior.
3. **Make the change** honoring every rule in Section 3. In particular re-check:
   plots/inputs stay in global scope (3.2); any new `ta.*` runs every bar (3.3);
   new persistent state uses `var`/`:=` (3.4); ternary type consistency (3.6).
4. **Self-review** against the checklist in Section 6 before responding.
5. **Output the entire updated file** in one code block.
6. **Explain in plain language** what changed and **what the user should look for
   on the chart** to confirm it works, plus anything you couldn't verify.

### Common requests, with the approach pre-sketched
- **Short/breakdown signals** — mirror the logic against `orLow`: `close < orLow`,
  `close < vwapValue`, red `shape.triangledown` below bar. Needs its **own**
  `firedToday`-style latch, separate from the long side.
- **ATR or % filter on breakout distance** — compute `ta.atr(len)` in global scope
  (3.3), require `close - orHigh >= atrValue * k` to cut marginal breaks.
- **Retest mode** — signal on a pullback tagging `orHigh` after the initial break
  instead of the first cross. Needs a "broke out already" state flag.
- **Target/stop plotting** — measured move (OR height projected up from `orHigh`)
  or R-multiple lines via `line.new` (mind the limits in 3.7).
- **Strategy conversion** — swap `indicator()` for `strategy()` and add
  `strategy.entry/exit` so TradingView's backtester can score it. **Meaningful
  rewrite; confirm with the user first.**
- **Stats table** — signal count / per-day tally via `table.new` (global scope).

---

## 6. Pre-response self-review checklist

Before returning code, confirm **all** of these:

- [ ] First non-comment line is `//@version=5`; exactly one `indicator()`.
- [ ] Every `plot*`, `bgcolor`, `hline`, `fill`, `alertcondition`, `input.*` is in
      **global scope** — conditionality is via arguments, not wrapping `if`.
- [ ] Every stateful `ta.*` is evaluated **every bar** in global scope, not inside
      an `if`; `length` args are `simple`/const.
- [ ] Persistent state uses `var`; reassignments use `:=`; stateful block ordering
      preserved (reset before accumulate).
- [ ] `alertcondition()` message is a **constant string**; dynamic text uses
      `alert()` in local scope; both kept if both were present.
- [ ] Ternary branches share a type; `na` guards precede comparisons.
- [ ] New drawing objects respect / raise the count limits and clean up.
- [ ] Indentation is consistent (4 spaces for local blocks).
- [ ] Output is the **whole file**, plus a plain-English summary and a "check this
      on your chart" note.

---

## 7. Known constraints (don't file these as bugs)

- **Intraday timeframes only.** On a daily/weekly chart the OR session never
  resolves to multiple bars and nothing plots.
- **Timeframe should divide 30 evenly** for a clean opening range (1m/5m/15m fine;
  7m misaligns).
- **`ta.vwap` needs volume data** — forex and some indices break the VWAP
  condition. It's built for US stocks/futures.
- **Half days / holidays / pre-market** aren't specially handled; session strings
  apply regardless, and extended-hours charts shift the VWAP anchor.
