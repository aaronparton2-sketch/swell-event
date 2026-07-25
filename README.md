# Powder Event 🎿❄️

The ski-trip automation. When the snow forecast fires, it books the whole thing.

A single [n8n](https://n8n.io) workflow that watches the snow forecast across the Australian alps — **Thredbo, Perisher, Mt Buller and Mt Hotham** — and the moment one of them is set to get buried for a few days straight, it picks the winning mountain, lines up the flights out of Sydney, the lodge, the snow transfer and the après drinks — then drops one proposal in the group chat. Whoever taps **yes** gets pushed into the booking flow.

Built in n8n + [Claude Code](https://claude.com/claude-code). This repo is the **template** — import it, drop in your own keys, and go.

> Heads up: this is a fun content build, shared honestly. The transfer + bottle-shop legs use AI voice calls (Bland) and the flight/lodge lookups use paid scrapers (Apify) — set hard spend caps before you run it live. Nothing here places a real booking without your opt-in tap.

---

## What it does

```
every 6h ─► Open-Meteo forecast for all 4 mountains (snowfall · temp · wind · base)
              │
              ▼  is any mountain 10cm+ fresh, cold and under 40kts for 3 days?
        ┌─────┴─────┐
       no          YES ─► pick the winning mountain
     (idle)              scrape cheapest flights SYD → its airport (Apify)
                         scrape best lodge on the hill (Apify)
                         │
                         ▼  build ONE proposal → group chat (Telegram)
                         │   mountain · dates · snowfall · flight price · lodge
                         ▼  poll: "lock in" or "miss out"
                    tapped YES
                         │
                         ├─► flight options emailed to you
                         ├─► live flight tracking → on landing, book a snow transfer
                         │     (AI voice call — chains on board, Dire Straits request)
                         └─► transfer ETA → order the après drinks so they
                               hit the lodge as you walk in
```

## The mountains

| Mountain | Coordinates | Region | Flies into |
|---|---|---|---|
| Thredbo | -36.5058, 148.3044 | Snowy Mountains, NSW | Canberra (CBR) |
| Perisher | -36.4052, 148.4120 | Snowy Mountains, NSW | Canberra (CBR) |
| Mt Buller | -37.1472, 146.4439 | Victorian Alps, VIC | Melbourne (MEL) |
| Mt Hotham | -36.9764, 147.1356 | Victorian Alps, VIC | Mt Hotham (MHU) |

Origin is **Sydney (SYD)**, -33.8688, 151.2093. Swap any of these in the `MOUNTAINS` table at the top of the **Forecast** node — and mirror the coordinates into the `latitude`/`longitude` query params on the **Open-Meteo Snow** node, which are read in the same order.

## Import it

1. In n8n: **Workflows → Import from File** → `powder-event.workflow.json`.
2. All credentials come in blank (this template ships with none). Reconnect your own on each node.
3. Fill the placeholders below, then run the **manual trigger** once to test before you let the schedule take over.

## What you need to plug in

| Placeholder in the workflow | What it is | Free? |
|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` + `YOUR_TELEGRAM_CHAT_ID` | a Telegram bot (via @BotFather) + your group chat id | free |
| `YOUR_APIFY_TOKEN` | [Apify](https://apify.com) token for the flight + lodge scrapers | free tier, **set a spend cap** |
| `YOUR_ORG_KEY` | [Bland AI](https://bland.ai) key for the transfer + bottle-shop voice calls | paid per minute |
| Gmail credential | on the "Check inbox" + "Email me" nodes (n8n's Gmail OAuth) | free |
| `YOUR-INSTANCE.app.n8n.cloud` | your n8n instance URL (for the Bland call-back webhook) | — |
| `+YOUR_PHONE` | the number Bland dials for the transfer | — |

The snow trigger uses [Open-Meteo](https://open-meteo.com)'s free forecast API — no login, no key. One call covers all four mountains (it accepts comma-separated coordinates and answers with one block per location, in request order).

## Tuning the trigger

Thresholds live at the top of the **Evaluate snow** node:

| Const | Default | Meaning |
|---|---|---|
| `MIN_SNOWFALL_CM` | `10` | fresh snow needed to count a day as "on" |
| `MAX_TEMP_C` | `2` | daily max above this and it's turning to slush |
| `MAX_WIND_KTS` | `40` | above this the lifts stop spinning |
| `MIN_ON_DAYS` | `3` | how many "on" days before the trip fires |
| `FORCE_FIRE` | `false` | flip to `true` to fire the whole chain on demand |

Mountains are ranked by number of "on" days, then by total snowfall — the winner drives the flight destination, the lodge search and every message downstream.

## Safety rails (please read)

- **Spend caps:** the Apify scrape nodes carry `maxTotalChargeUsd` limits in the URL — keep them low. Set a monthly cap in your Apify account too.
- **Voice calls cost money and call real numbers.** Test with your own number first. The transfer node is set to hang up on voicemail and never leave a message.
- Nothing books a flight or a trip without a human tapping **yes** in the chat.

## License

MIT — do what you like, no warranty. Send it. 🤙
