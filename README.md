# ORB Breakout Indicator for TradingView

A free indicator for [TradingView](https://www.tradingview.com) that watches the
stock market for a specific "opening range breakout" pattern each morning and
puts a green arrow on your chart (and can send you an alert) when it happens.

You do **not** need to know any coding to use this. This guide walks you through
every single step. If you can copy and paste, you can install this.

---

## What it actually does

Every trading day, the stock market opens at 9:30 AM Eastern Time. This indicator
watches the first 30 minutes (9:30–10:00 AM ET) and remembers the highest price
during that window. That window is called the **"opening range."**

After that, it draws an orange line at that high price and waits. It puts a green
**"BO"** (breakout) arrow on your chart the moment **all four** of these line up
at the same time:

1. **Price breaks out** — the price closes above the orange opening-range line.
2. **Volume confirms it** — a lot of shares are trading (at least 1.5× the
   average of that opening 30-minute window), which suggests the move is real.
3. **Above VWAP** — the price is above the blue VWAP line (a common "fair value"
   line traders watch).
4. **It's still early** — this all happens before 11:30 AM ET.

By default it only marks the **first** breakout of the day, so your chart stays
clean.

> **Important, please read:** This is a charting tool. It highlights when a
> pattern appears — it does **not** tell you whether a trade is a good idea, and
> it is not financial advice. Treat the arrow as "hey, look at this," nothing
> more.

---

## What you need before you start

- A Mac (this guide is written for macOS).
- A free TradingView account. If you don't have one, go to
  [tradingview.com](https://www.tradingview.com) and click **Sign up** in the top
  right. A free account is enough to use this.

That's it. There is nothing to download or install on your Mac. Everything
happens inside your web browser (Safari, Chrome, etc.) on the TradingView
website.

---

## Part 1 — Get the indicator's text

The indicator is just a page of text (called "Pine Script"). You need to copy
that text so you can paste it into TradingView.

1. In this repository, click on the file named **`orb_breakout_indicator.pine`**
   (it's in the file list near the top of this page).
2. You'll see the text of the script fill the screen. Look for a button on the
   right side above the text labeled **"Raw"** and click it. The page will turn
   into plain text.
3. Now select **all** of the text:
   - Click anywhere in the text, then press **Command (⌘) + A** on your keyboard.
     This highlights everything.
   - Press **Command (⌘) + C** to copy it.

   (Command is the key next to the spacebar with the ⌘ symbol on it.)

Leave this browser tab open for now — you've got the text copied, and we'll come
back if needed.

---

## Part 2 — Open the Pine Editor in TradingView

1. Open a **new browser tab** and go to
   [tradingview.com](https://www.tradingview.com).
2. Make sure you're logged in (your profile icon shows in the top-right corner).
3. Click **Products** in the top menu, then click **Supercharts** (on some
   accounts this is just called **Chart**). You should now be looking at a price
   chart.
4. Look at the very **bottom** of the screen. There's a thin toolbar there. Click
   the button that says **Pine Editor**.
   - If you don't see it, look for a small **^** (up-arrow) or a menu at the
     bottom edge and click it — the Pine Editor button is in that bottom strip.
5. A text panel opens at the bottom of the screen. This is the **Pine Editor**.
   It usually starts with a couple of lines of example text already in it.

---

## Part 3 — Paste in the indicator

1. Click inside the Pine Editor panel (the text area you just opened).
2. Select everything that's already in there and delete it:
   - Press **Command (⌘) + A** to select all.
   - Press **Delete** (the big key above Return) to clear it out.
3. Now paste in the indicator you copied earlier:
   - Press **Command (⌘) + V**.
   - The full script should appear, starting with a line near the top that reads
     `//@version=5`.
4. Click the **"Add to chart"** button. It's in the top-right corner of the Pine
   Editor panel.

If it worked, you'll see new things appear on your chart: a **blue VWAP line**,
and after 10:00 AM an **orange line** marking the opening-range high. 🎉

> **If you see a red error message instead:** the most common cause is that the
> old example text didn't get fully deleted before pasting, so the text got
> mixed together. Click in the editor, press **⌘ + A**, then **Delete**, then
> **⌘ + V** to paste again cleanly, and click **Add to chart** once more.

---

## Part 4 — Set your chart up so the indicator works

This indicator only works on **intraday** charts (charts that show individual
minutes of the day), not daily or weekly charts. It needs to see the morning
tick by tick.

1. At the **top-left** of the chart there's a number that sets the timeframe
   (it might say something like `1D` or `5m`). Click it.
2. Choose a **minute-based** timeframe. Good choices: **1 minute**, **5 minutes**,
   or **15 minutes**. (5-minute is a nice, readable default.)
   - Avoid oddball numbers like 7 minutes — they don't line up cleanly with the
     30-minute opening range.
3. Type a US stock symbol into the search box at the top-left (for example
   **AAPL**, **SPY**, or **TSLA**) and pick it.

The indicator is designed for **US stocks and futures**, because it relies on
volume and the 9:30 AM ET open. It won't work well on things like foreign
exchange (forex) that don't have proper volume data.

Once you're on, say, a 5-minute chart of SPY during or after market hours, you'll
see the orange opening-range line and, when the pattern hits, the green **BO**
arrow.

---

## Part 5 (optional but recommended) — Turn on alerts

An alert means TradingView notifies you the moment the breakout happens, so you
don't have to stare at the screen. Here's how to set one up.

1. Make sure the indicator is on your chart (Parts 3 and 4 done).
2. At the top of the chart, click the **alarm-clock / bell icon** (the **Alerts**
   button). A settings window pops up.
3. In that window, find the **"Condition"** dropdown (the first setting).
   - Set the first dropdown to **ORB Breakout | 30m OR + Volume + VWAP** (that's
     this indicator).
   - Set the second dropdown to **"Any alert() function call."** This is the
     important one — it's what lets the alert include the live price and details.
4. Find the setting called **"Trigger"** (sometimes labeled "Options") and set it
   to **"Once Per Bar Close."**
   - This matters. It stops the alert from firing early and then changing its
     mind. Waiting for the bar to close gives you a confirmed signal.
5. Under **"Notifications,"** pick how you want to be told — for example, check
   **"Notify on app"** (if you have the TradingView phone app), **"Show pop-up,"**
   or **"Send email."**
6. Click **Create** at the bottom.

That's it. TradingView will now watch for you and alert you when the breakout
fires, even if you close the tab.

> **Why do the triangles sometimes flash and disappear during a live bar?**
> That's normal. While a 5-minute bar is still forming, the price bounces around,
> so the arrow can appear and then vanish if the price drops back. The
> "Once Per Bar Close" alert setting above exists precisely to only tell you
> about the final, confirmed result.

---

## Adjusting the settings (optional)

You can tweak how picky the indicator is without touching any code:

1. On the chart, find the indicator's name (**ORB Breakout | 30m...**) in the
   top-left list of indicators.
2. Hover over it and click the little **gear / cog icon** that appears.
3. A settings window opens where you can change:

| Setting | What it does |
|---|---|
| **Opening Range** | The morning window it measures. Default `0930-1000` = the first 30 minutes. Change to `0930-1030` for a 60-minute range. |
| **Valid Signal Window** | The window during which a breakout is allowed to count. Default `0930-1130`. |
| **Volume Multiple** | How much extra volume is required. Default `1.5`. Raise it (e.g. `2.0`) to only catch bigger, louder moves; lower it to catch more. |
| **Only signal the first breakout each day** | On by default. Turn off if you want an arrow on *every* qualifying bar, not just the first. |
| **Also draw opening range low** | Off by default. Turn on to also see the orange line for the morning's *low*. |

Change a setting, and the chart updates instantly. Nothing can break — if you
mess it up, just close and reopen the settings, or remove and re-add the
indicator.

---

## Frequently confused things

- **"I put it on a daily chart and nothing shows up."** Correct — it needs a
  minute-based chart (see Part 4). Switch to 1m, 5m, or 15m.
- **"The lines look different from my other chart."** If one chart has
  "extended hours" (pre-market) turned on and the other doesn't, the VWAP line
  starts from a different point and the numbers won't match. That's expected, not
  a bug.
- **"It's the afternoon and I don't see an arrow today."** The pattern simply may
  not have happened today, or it happened and the one arrow is earlier in the
  morning. No arrow means no signal — that's the tool working correctly.
- **"It doesn't work on EUR/USD / Bitcoin / this foreign stock."** It's built for
  US stocks and futures with real volume data. Other markets may not have the
  volume information it needs.

---

## A note on saving

Once you've added it to a chart, TradingView remembers it for that chart layout.
To keep it handy, you can also save it into your personal indicator list: in the
Pine Editor, there's a **"Save"** button (top-right of the editor) — give it a
name like "ORB Breakout" and save. After that it'll show up when you click the
**Indicators** button at the top of any chart and search for that name.

---

## For tinkerers: changing or adding features with an AI

You don't have to learn Pine Script to customize this. This repo is set up so you
can hand it to an **AI coding assistant** — like **Claude Code** or **ChatGPT
Codex** — and just *describe* what you want in plain English. For example:

> "Add a red arrow for breakdowns below the opening-range low."
> "Only signal if volume is 2× instead of 1.5×."
> "Also draw a target line one range-height above the breakout."

The AI reads the [`AGENTS.md`](AGENTS.md) file included here, which is a detailed
handoff written **for the AI, not for you**. It spells out the TradingView Pine
Script rules the AI must follow so the changes don't break the indicator, the
layout of the code, and how to hand you back a working file. (There's also a
`CLAUDE.md` that simply points Claude Code to the same handoff.)

**How to use it:**

1. Point your AI assistant at this repository (open the folder in Claude Code, or
   give ChatGPT Codex the repo). Both automatically read the handoff file.
2. Describe the change you want in normal words.
3. The AI gives you back the **complete updated script** and a plain-English
   summary of what changed.
4. Paste that new script into TradingView's Pine Editor exactly like in Part 3
   above (clear the old text first, then paste, then "Add to chart").

Because Pine Script can only run inside TradingView, the AI can't test it for you
— so after pasting, glance at your chart to confirm the change looks right. The
handoff tells the AI to always point out what to check.

---

## License

This indicator is released under the [MIT License](LICENSE) — free to use, copy,
and modify.
