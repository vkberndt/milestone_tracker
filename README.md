# 🏆 Milestone Bot

A purpose‑built Discord bot for logging, displaying, and managing player **milestones** with Google Sheets as a structured, auditable backend.  
Designed for **self‑service**, **resilience**, and a **smooth user experience** — with leaderboards, personal stats, autocomplete, and safe entry removal.

---

## ✨ Features

- **Log Milestones** (`/milestone`)  
  - Species (with autocomplete to bypass Discord’s 25‑choice limit)  
  - Tier (Bronze, Silver, Gold, Diamond)  
  - Character Sheet URL  
  Data is stored in Google Sheets and announced with rich embeds.

- **View Leaderboards** (`/leaderboard`)  
  - Top 5 players with tier breakdown.  
  - On‑demand private view or automated daily announcement.

- **Personal Stats** (`/my_stats`)  
  - Private breakdown of your milestones by species and tier.

- **Self‑Service Removal** (`/remove_milestone`)  
  - Delete your most recent entry for a given species and tier (only your own).

- **Ephemeral UX**  
  - Most responses are private to the user for clarity and to keep channels clean.

---

## 🛠️ Tech Stack

- **Language:** Python 3.12  
- **Framework:** [discord.py 2.x](https://discordpy.readthedocs.io/en/stable/) (`app_commands` for slash commands)  
- **Data Store:** [Google Sheets](https://www.google.com/sheets/about/) via [gspread](https://github.com/burnash/gspread) and a Google Service Account   

---

## 🚀 Setup

### 1. Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/milestone-bot.git
cd milestone-bot
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```
### 3. Configure Environmental Variables
```bash
DISCORD_TOKEN=your_discord_bot_token
LOG_CHANNEL_ID=123456789012345678
SHEET_ID=your_google_sheet_id
GOOGLE_SHEETS_CREDENTIALS_PATH=credentials.json
```

### 4. Intents

In the Discord Developer Portal:
1. Enable **Server Members Intent**
2. Enable **Server Message Content Intent**

### 5. Create a Google Service Account

Follow the gspread authentication guide and share your target spreadsheet with the account's email. Save your credentials JSON as credentials.json in the project root (or update the path in .env)

## 📜 Command Reference

| Command             | Description                                                                                   |
|---------------------|-----------------------------------------------------------------------------------------------|
| `/milestone`        | Log a species, tier, and character-sheet URL; stores the entry in Google Sheets and announces it. |
| `/leaderboard`      | Show the top 5 players with their milestone counts broken down by tier (ephemeral/private).    |
| `/my_stats`         | Display your own milestone statistics by species and tier in a private embed.                  |
| `/remove_milestone` | Remove your most recent matching milestone entry for a given species and tier (self-service).   |

## 🏗️ Design Decisions & Challenges

Decisions based on challenges encountered during development.

- **Autocomplete for Species**  
  **Challenge:** Discord slash commands limit static choice lists to 25 entries, but our species list exceeds that.  
  **Decision & Impact:** Replaced static `@choices` with an `@autocomplete` callback. This bypasses the cap, scales indefinitely, and improves UX with live filtering.

- **On-Ready Task Startup**  
  **Challenge:** Scheduled tasks (`@tasks.loop`) were starting before the event loop was ready, causing race-condition errors.  
  **Decision & Impact:** Moved task startup into `on_ready()` and added `.is_running()` guards. Tasks now launch only after the bot is fully synced.

- **String-Normalized IDs**  
  **Challenge:** Google Sheets sometimes returned numeric Discord IDs, while Discord.py uses strings, leading to mismatches.  
  **Decision & Impact:** Cast all IDs to strings before comparison. Ensures correct matching in stats retrieval and row deletion.

- **Precise Row Deletion**  
  **Challenge:** `delete_row`/`delete_rows` methods were inconsistently available on `Worksheet` objects.  
  **Decision & Impact:** Switched to spreadsheet-level `batch_update` with a `deleteDimension` request to reliably remove the exact row across all environments.

- **Rate-Limit Friendly Interactions**  
  **Challenge:** Using `defer()` plus `followup.send()` twice sometimes triggered Discord 429 errors.  
  **Decision & Impact:** Consolidated to a single `response.send_message()` call where possible, reducing HTTP calls and avoiding rate limits.

- **Self-Service & Transparency**  
  **Challenge:** Players needed a way to view and correct their own data without moderator help.  
  **Decision & Impact:** Added `/my_stats` for private stats breakdown and `/remove_milestone` with strict DiscordID + Species + Tier validation to safely remove entries.
