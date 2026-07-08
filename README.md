# Swell Event 🌊🎉

The surf-trip automation. When the forecast fires, it books the whole thing.

A single [n8n](https://n8n.io) workflow that watches the surf forecast for your spot, and the moment it goes big and offshore for a few days straight, it lines up the flights, the villa, the taxi and the cold beers — then drops one proposal in the group chat. Whoever taps **yes** gets pushed into the booking flow.

Built in n8n + [Claude Code](https://claude.com/claude-code). This repo is the **template** — import it, drop in your own keys, and go.

> Heads up: this is a fun content build, shared honestly. The taxi + Gojek legs use AI voice calls (Bland) and the flight/villa lookups use paid scrapers (Apify) — set hard spend caps before you run it live. Nothing here places a real booking without your opt-in tap.

---

## What it does

```
every 6h ─► Surfline forecast (swell · period · wind)
              │
              ▼  is it 6ft+ and offshore for 3 days?
        ┌─────┴─────┐
       no          YES ─► scrape cheapest flights (Apify)
     (idle)              scrape best villa near the surf (Apify)
                         │
                         ▼  build ONE proposal → group chat (Telegram)
                         │   dates · swell · flight price · villa
                         ▼  poll: "lock in" or "miss out"
                    tapped YES
                         │
                         ├─► flight options emailed to you
                         ├─► out-of-office flips on when the plane lands
                         ├─► live flight tracking → on landing, order a taxi
                         │     (AI voice call, with the Dire Straits request)
                         └─► taxi ETA → fire a Gojek order so cold Bintangs
                               hit the villa as you walk in
```

## Import it

1. In n8n: **Workflows → Import from File** → `swell-event.workflow.json`.
2. All credentials come in blank (this template ships with none). Reconnect your own on each node.
3. Fill the placeholders below, then run the **manual trigger** once to test before you let the schedule take over.

## What you need to plug in

| Placeholder in the workflow | What it is | Free? |
|---|---|---|
| `YOUR_TELEGRAM_BOT_TOKEN` + `YOUR_TELEGRAM_CHAT_ID` | a Telegram bot (via @BotFather) + your group chat id | free |
| `YOUR_APIFY_TOKEN` | [Apify](https://apify.com) token for the flight + villa scrapers | free tier, **set a spend cap** |
| `YOUR_ORG_KEY` | [Bland AI](https://bland.ai) key for the taxi + Gojek voice calls | paid per minute |
| Gmail credential | on the "Check inbox" + "Email me" nodes (n8n's Gmail OAuth) | free |
| `YOUR-INSTANCE.app.n8n.cloud` | your n8n instance URL (for the Bland call-back webhook) | — |
| `+YOUR_PHONE` | the number Bland dials for the taxi | — |

The forecast trigger uses Surfline's public `kbyg` endpoint via the free [r.jina.ai](https://r.jina.ai) relay — no login, no key. Swap the spot in the two "Surfline" nodes for your break.

## Safety rails (please read)

- **Spend caps:** the Apify scrape nodes carry `maxTotalChargeUsd` limits in the URL — keep them low. Set a monthly cap in your Apify account too.
- **Voice calls cost money and call real numbers.** Test with your own number first. The taxi node is set to hang up on voicemail and never leave a message.
- Nothing books a flight or a trip without a human tapping **yes** in the chat.

## License

MIT — do what you like, no warranty. Have a mean one. 🤙
