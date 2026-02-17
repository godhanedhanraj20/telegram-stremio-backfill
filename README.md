# Telegram Channel History Backfill for Google Colab

This repository provides a **beginner-friendly Google Colab workflow** to backfill media history from multiple Telegram channels into MongoDB Atlas, with production-oriented safety for large channels (30k–100k+ messages).

The notebook uses:
- **Pyrogram** with a user session (`api_id`, `api_hash`)
- **Google Drive** for persistent session + output files
- **MongoDB Atlas** for storage
- Advanced caption parsing into structured metadata
- Duplicate prevention with a unique index
- FloodWait retry handling
- Incremental JSONL export (memory-safe)
- Resume scanning support (`START_FROM_MESSAGE_ID`)
- `SAFE_MODE` dry-run option

---

## What this tool does

1. Mounts your Google Drive in Colab.
2. Installs Python dependencies.
3. Creates/reuses a Pyrogram user session file from Drive.
4. Scans one or more Telegram channels.
5. Collects media messages and parses captions.
6. Optionally writes to MongoDB Atlas (or dry-run in `SAFE_MODE`).
7. Avoids duplicates using unique key (`channel + message_id`).
8. Writes output incrementally as JSON Lines to avoid memory growth.
9. Prints periodic and final progress per channel.

---

## Prerequisites

Before running the notebook, make sure you have:

- A Google account (for Colab + Drive)
- A Telegram account
- Telegram API credentials (`api_id`, `api_hash`) from https://my.telegram.org
- A MongoDB Atlas cluster and connection URI (unless using `SAFE_MODE=True`)

---

## Files

- `backfill_colab.ipynb` — Main Colab notebook
- `requirements.txt` — Python dependencies
- `.env.example` — Optional template for environment variables

---

## Quick start (beginner friendly)

### 1) Open in Google Colab

1. Create a new Colab notebook, or upload `backfill_colab.ipynb`.
2. Run each cell in order.

### 2) Mount Google Drive

The first cell mounts your Drive at `/content/drive`.

This is important because:
- Your Telegram session file is stored there.
- Your JSONL output file is stored there.

### 3) Install dependencies

The notebook installs all required packages.

### 4) Set environment variables securely

Use Colab secrets or temporary runtime variables.

Required:
- `TG_API_ID`
- `TG_API_HASH`
- `MONGODB_URI` (not required when `SAFE_MODE=True`)

Optional:
- `MONGODB_DB` (default: `telegram_backfill`)
- `MONGODB_COLLECTION` (default: `messages`)
- `TG_SESSION_NAME` (default: `telegram_user_session`)

> ✅ Tip: Avoid hardcoding secrets directly in notebook cells. Use `os.getenv()` and `getpass` fallback.

### 5) Configure channels and runtime options

Set channels and optional controls in the config cell:

```python
CHANNELS = [
    "https://t.me/example_channel",
    "@another_channel",
]
SAFE_MODE = False
START_FROM_MESSAGE_ID = None
```

- `SAFE_MODE=True`: dry-run mode, no MongoDB writes.
- `START_FROM_MESSAGE_ID=12345`: resume mode. Messages with lower IDs are skipped.

### 6) Run scan cell

The notebook will:
- Iterate history for each channel
- Handle `FloodWait` by sleeping and continuing
- Keep only media messages
- Parse captions including:
  - key:value pairs
  - hashtags
  - year (`(19|20)\d{2}`)
  - quality (`480p`, `720p`, `1080p`, `2160p`)
  - season/episode (`S01E02`)
- Upsert into MongoDB Atlas (unless `SAFE_MODE=True`)
- Print progress every 1000 processed messages
- Print per-channel totals at the end

### 7) Review output

After completion:
- Check Colab logs
- Review documents in MongoDB Atlas (if not in safe mode)
- Review the JSONL export in your Drive output folder

---

## MongoDB duplicate prevention

The notebook creates a unique index on:

- `channel`
- `message_id`

Writes use `update_one(..., upsert=True)`, so reruns are idempotent.

---

## Resume mode details (`START_FROM_MESSAGE_ID`)

Telegram history is scanned from newest to oldest. When `START_FROM_MESSAGE_ID` is set, older messages are skipped. This lets you continue large backfills from a known message boundary.

Example:
- First run scans everything and reaches message ID 45000.
- Next run sets `START_FROM_MESSAGE_ID = 45000` to avoid reprocessing older history.

---

## Output format (JSONL)

Output is written as **JSON Lines** (`.jsonl`): one JSON object per line.

Benefits:
- Memory-safe for large channels
- Easy to stream/process later
- Partial progress preserved even if runtime stops

---

## Security notes

- Do **not** commit your real `.env` file.
- Do **not** hardcode API credentials.
- Use Colab secrets or prompt (`getpass`).
- Restrict MongoDB network access and use least-privilege users.

---

## Troubleshooting

- **Session asks for login every run**: Keep session path on mounted Drive.
- **FloodWait triggered**: Expected on large scans; notebook sleeps and resumes.
- **No messages found**: Verify channel access and media existence.
- **MongoDB auth errors**: Re-check URI and Atlas allowlist.

---

## License

MIT (or choose a license suitable for your usage).
