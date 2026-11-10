# Activity Coach

A small, **self-use** productivity tracker. It logs what you do on *your own*
computer, then at an interval you choose it sends a compact summary to an AI
provider (Claude, Gemini, or OpenAI — whichever you give an API key for) and
gets back an analysis of what you got done and where your pattern is heading.

Everything is stored locally. Nothing leaves your machine except the periodic
summary you send to the AI provider you configured.

> Built for tracking **your own** device. Putting this on someone else's
> machine (e.g. staff) turns it into employee monitoring, which carries
> disclosure/consent obligations under local law — not covered here.

## What it captures

- Active application + window title, sampled every few seconds
- Idle vs. active time (from OS idle timers — *not* keystroke logging)
- Optional: screenshots at analysis time (off by default)
- A rolling history, so the AI can see trends across periods

## How "real-time" works

LLMs are request/response, so there's no literal live stream. The tracker logs
continuously and **batches** a summary to the AI every N minutes (you set N). A
short interval (e.g. 15 min) feels near-live; end-of-day is cheapest.

## Install

Requires Python 3.9+.

```bash
cd activity-coach
python -m pip install -r requirements.txt
```

Platform notes:
- **Windows / macOS**: idle detection works out of the box.
- **Linux (X11)**: install `xprintidle` for idle detection
  (`sudo apt install xprintidle`). Without it, idle time is left blank.

## Configure

```bash
cp config.example.json config.json
```

Edit `config.json`:
- `goals` — what you're trying to achieve, in your words. This is what the AI
  measures your activity against.
- `provider` — `"claude"`, `"gemini"`, or `"openai"`.
- `analysis_interval_minutes` — how often the AI analyses you.
- `send_screenshots` — `true`/`false`. When false, only text (app names,
  window titles, time) is sent.
- Under `providers.<name>.model`, set a **current** model name for that vendor
  (model IDs change over time — check the provider's docs).

Put your API key in an environment variable (recommended) matching
`api_key_env`, e.g.:

```bash
export ANTHROPIC_API_KEY=sk-ant-...     # or GEMINI_API_KEY / OPENAI_API_KEY
```

(You need the vendor's **paid API key**, not a chat subscription.)

## Run

```bash
# Run continuously (samples + analyses on your schedule)
python -m activity_coach --config config.json

# Analyse just the last interval once, then exit (good for a first test)
python -m activity_coach --config config.json --once

# Print the last few AI analyses
python -m activity_coach --config config.json --report
```

Reports are also written to `<data_dir>/reports/`.

## Switching AI providers

Change the single `"provider"` field. Each provider's key/model lives under
`"providers"`, so you can keep all three configured and flip between them.

## Files

```
activity_coach/
  tracker.py    active-window + idle capture (per-OS, best-effort)
  storage.py    local SQLite (samples + analyses)
  analyzer.py   aggregates samples, builds the AI prompt
  providers.py  Claude / Gemini / OpenAI adapters (add more here)
  config.py     loads config.json
  __main__.py   sampler + scheduler + CLI
```
