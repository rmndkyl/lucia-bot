# lucia-bot

Automation for the **Lucia Protocol** airdrop campaign (`airdrop.luciaprotocol.com`,
Season 1) — wallet sign-in, social tasks and the daily share streak, across a fleet
of wallets.

Part of the [LYID](https://github.com/layerairdrop) script collection.

![run](docs/screenshots/run.png)

## How it works

| Step | What happens |
|---|---|
| Sign-in | SIWE message signed locally (`personal_sign`), verified with one Cloudflare Turnstile token per wallet |
| Turnstile | Real Chrome (headed) renders the site's own widget and mints a token in ~3 s; one browser session mints tokens for the whole fleet |
| Session | Bearer token valid 24 h — the daily run refreshes only the wallets that need it |
| Telegram tasks | Worker emits a one-time `t.me/<bot>?start=<code>` link; the bot drives a Telethon session, joins the required chats, reads the confirmation code and claims both tasks |
| X follow | One-time OAuth2 link per account (`--link-x`): Chrome opens, the human logs in and presses Authorize, the bot waits until the identity is attached and claims the task |
| Daily share | Streak claim every day, multiplier 2.5x → 5x → 10x |

## Features

- Fleet config: one entry per wallet (label, wallet key, Telegram session), `--account` selector.
- Session cache with expiry, so logins only happen when needed.
- Idempotent claims: already-verified tasks are skipped, `409` codes are reported, not crashed on.
- Telegram link codes expire in ~10 min, so a fresh link is minted on every run.
- Machine-readable report: `data/last-run.json` plus the console log in `data/last-run.txt`.
- Windows launchers with an Indonesian menu (`JALANKAN.bat`, `CEK-POIN.bat`, `SET-HARIAN.bat`, `LINK-X.bat`) — no CLI needed.
- Built-in DNS fallback (`*.workers.dev` is dropped by some resolvers): resolves through 8.8.8.8 and then Cloudflare anycast.

## Requirements

Windows, Chrome, Node 18+, Python 3.11+, Telethon session files, one EVM wallet per account.

```bat
python -m venv .venv && .venv\Scripts\python.exe -m pip install -r requirements.txt
set PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 && npm install
copy config.json.example config.json
.venv\Scripts\python.exe main.py --init-wallets
.venv\Scripts\python.exe main.py --run
```

## Commands

```
--status                 points, task states, streak per account
--login                  refresh sessions (opens Chrome for Turnstile)
--run                    join + wallet + telegram + claims
--link-x --account X     one-time X OAuth for that account
--daily                  daily-share claim (cron friendly)
--init-wallets           generate wallets for accounts missing one
```

## Notes

- No deposit, no approvals, no swaps: the bot only signs in, claims points and links socials.
- The X follow task is human-gated by design — one login per X account, then it is automatic.
- Campaign terms and point values belong to Lucia Protocol and can change between seasons.

Private build used by LYID for the Season 1 farm. Full source lives in a private repo.
