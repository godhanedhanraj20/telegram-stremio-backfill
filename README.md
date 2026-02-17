# Telegram Channel History Backfill for Google Colab

This repository provides a **beginner-friendly Google Colab workflow** to backfill media history from multiple Telegram channels into MongoDB Atlas.

The notebook uses:
- **Pyrogram** with a user session (`api_id`, `api_hash`)
- **Google Drive** for persistent session + export files
- **MongoDB Atlas** for storage
- Caption parsing into structured metadata
- Duplicate prevention with a unique index

---

## What this tool does

1. Mounts your Google Drive in Colab.
2. Installs Python dependencies.
3. Creates/reuses a Pyrogram user session file from Drive.
4. Scans one or more Telegram channels.
5. Collects media messages and parses captions.
6. Saves records to MongoDB Atlas.
7. Avoids duplicate inserts using a unique key (`channel + message_id`).
8. Prints progress while running.

---

## Prerequisites

Before running the notebook, make sure you have:

- A Google account (for Colab + Drive)
- A Telegram account
- Telegram API credentials (`api_id`, `api_hash`) from https://my.telegram.org
- A MongoDB Atlas cluster and connection URI

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
- Output JSON files are stored there.

### 3) Install dependencies

The notebook installs all required packages from `requirements.txt`.

### 4) Set environment variables securely

You can set credentials via Colab secrets or temporary runtime variables.

Required variables:
- `TG_API_ID`
- `TG_API_HASH`
- `MONGODB_URI`
- `MONGODB_DB` (default: `telegram_backfill`)
- `MONGODB_COLLECTION` (default: `messages`)

Optional:
- `TG_SESSION_NAME` (default: `telegram_user_session`)

> ✅ Tip: Avoid hardcoding secrets directly in notebook cells. Use `os.getenv()` and prompt with `getpass` when missing.

### 5) Configure channels to scan

Edit the channel list in the notebook, for example:

```python
CHANNELS = [
    "https://t.me/example_channel",
    "@another_channel",
]
```

You can include public usernames, links, or numeric IDs.

### 6) Run scan cell

The notebook will:
- Iterate history for each channel
- Keep only media messages
- Parse caption metadata
- Upsert into MongoDB Atlas
- Print inserted/skipped counters

### 7) Review output

After completion:
- Check logs in Colab output
- Review documents in MongoDB Atlas
- Optional JSON export is saved to your Drive output folder

---

## MongoDB duplicate prevention

The notebook creates this unique index:

- `channel`
- `message_id`

If a message already exists, MongoDB rejects duplicate insertion. The notebook also uses idempotent updates via `update_one(..., upsert=True)` to make reruns safe.

---

## Data model overview

Each saved document includes fields like:

- `channel`
- `message_id`
- `date`
- `media_type`
- `file_id`
- `caption_raw`
- `caption_meta` (parsed key/value metadata)
- `views`
- `forwards`
- `collected_at`

---

## Security notes

- Do **not** commit your real `.env` file.
- Do **not** hardcode API credentials in notebooks.
- Use Colab secrets or runtime prompts (`getpass`).
- Restrict MongoDB network access and use least privilege users.

---

## Troubleshooting

- **Session asks for login every run**: Ensure session directory is on mounted Drive.
- **FloodWait / rate limits**: Reduce scan pace and retry later.
- **No messages found**: Verify channel access and that posts contain media.
- **MongoDB auth errors**: Re-check URI, username/password, and IP allowlist.

---

## License

MIT (or choose a license suitable for your usage).
