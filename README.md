# 🐝 HiveCore Bot — Step-by-Step Setup Guide

This guide walks you through setting up the HiveCore Discord bot from scratch on **Windows**, **Linux/Mac**, or directly on a **Pterodactyl Node.js egg**.

---

## ✅ What You Need (Prerequisites)

Before you start, gather these. **All of them are free**.

1. **Node.js 18+** — install from <https://nodejs.org/> (pick the LTS version)
2. **A Discord bot application** — you'll create it in step 1 below
3. **A Pterodactyl Panel** with:
   - **Application API key** (Admin → Application API → Create New)
   - **Client API key** (Account → API Credentials → Create New)
   - At least **one Node**, one **Location**, and one **Egg** (the default Minecraft egg works)
4. **Your Discord User ID** (you'll be the bot owner) — turn on Developer Mode in Discord, right-click your name, "Copy User ID"

---

## 🔧 Step 1 — Create the Discord Bot

1. Go to <https://discord.com/developers/applications> and click **New Application**. Name it `HiveCore` (or whatever you want).
2. In the left sidebar, click **Bot** → **Reset Token** → copy the token. **This is your `DISCORD_TOKEN`.** Save it somewhere safe.
3. On the same page, scroll down to **Privileged Gateway Intents**:
   - Toggle **MESSAGE CONTENT INTENT** ON (optional, good to have).
   - The bot uses **slash commands** so the other intents are not strictly required.
4. Go to the **OAuth2 → General** tab and copy the **Application ID**. **This is your `DISCORD_CLIENT_ID`.**
5. Go to **OAuth2 → URL Generator**:
   - **Scopes:** check `bot` and `applications.commands`
   - **Bot Permissions:** `View Channels`, `Send Messages`, `Embed Links`, `Attach Files`, `Read Message History`, `Use External Emojis`
   - Copy the generated URL at the bottom and open it in a new tab → invite the bot to your Discord server.

---

## 🔧 Step 2 — Get Your Pterodactyl Keys

1. Log in to your Pterodactyl panel.
2. **Application API key** (admin only):
   - Go to `https://YOUR-PANEL.com/admin/api`
   - Click **Create New** → tick **all read+write permissions** → save the key starting with `ptla_…`
3. **Client API key**:
   - Go to `https://YOUR-PANEL.com/account/api`
   - Click **Create** → save the key starting with `ptlc_…`
4. Note your **panel URL** (e.g. `https://panel.therynzo.in`) — no trailing slash.
5. Find the IDs you need:
   - Open `https://YOUR-PANEL.com/admin/locations` → note the **Location ID** (the number).
   - Open `https://YOUR-PANEL.com/admin/nests` → click your Minecraft nest → note the **Nest ID** and the **Egg ID** for "Vanilla Minecraft" (or whichever egg you want).

---

## 🔧 Step 3 — Download & Install the Bot

### On your computer (Windows/Mac/Linux):

1. **Unzip** `hivecore-bot.zip` somewhere convenient (e.g. `C:\bots\hivecore-bot` or `~/hivecore-bot`).
2. Open a terminal in that folder and run:
   ```bash
   npm install
   ```
   This installs all required dependencies (takes 1–2 minutes).

### Or directly on your Pterodactyl Node.js egg:

1. Create a new server using the **Node.js Generic** egg.
2. Set **Startup Command** to `npm start`.
3. SFTP / file-manager upload the contents of the zip into the server's root.
4. In the panel **Console**, run:
   ```bash
   npm install
   ```

---

## 🔧 Step 4 — Configure `.env`

1. In the project folder, copy `.env.example` to `.env`:
   - **Linux/Mac:** `cp .env.example .env`
   - **Windows (PowerShell):** `Copy-Item .env.example .env`
2. Open `.env` in any text editor and fill in **at minimum** these values:

```env
# --- Discord ---
DISCORD_TOKEN=paste_your_bot_token_here
DISCORD_CLIENT_ID=paste_your_application_id_here
# Optional but recommended for testing — paste your Discord server (guild) ID.
# Slash commands will appear instantly there. Leave empty for global (~1 hour).
DISCORD_DEV_GUILD_IDS=YOUR_TEST_SERVER_ID

# --- Pterodactyl ---
PTERO_PANEL_URL=https://panel.therynzo.in
PTERO_APP_API_KEY=ptla_your_application_key
PTERO_CLIENT_API_KEY=ptlc_your_client_key

# --- Auto-provisioning defaults (from step 2 above) ---
PTERO_DEFAULT_EGG_ID=1
PTERO_DEFAULT_NEST_ID=1
PTERO_DEFAULT_LOCATION_ID=1
PTERO_DEFAULT_DOCKER_IMAGE=ghcr.io/pterodactyl/yolks:java_17

# --- Bot ownership ---
OWNER_IDS=YOUR_DISCORD_USER_ID
```

Everything else has sensible defaults — you can tweak rewards, fees, etc. later.

---

## 🔧 Step 5 — Register Slash Commands

This tells Discord which `/commands` your bot supports. Run **once** (and again whenever you add/change commands):

```bash
npm run deploy
```

You should see something like `guild commands deployed` in the terminal.

> ⏰ If you used global registration (`DISCORD_DEV_GUILD_IDS` empty), commands take up to **1 hour** to appear in Discord. For instant testing, keep `DISCORD_DEV_GUILD_IDS` set to your test server.

---

## 🔧 Step 6 — Start the Bot

```bash
npm start
```

Expected output:
```
[10:23:01] INFO: database initialized
[10:23:01] INFO: commands loaded { count: 39 }
[10:23:01] INFO: events loaded
[10:23:02] INFO: bot online
```

In Discord, you should see your bot turn online. Try `/help` — you'll see the full command list.

---

## 🔧 Step 7 — Configure In Discord (Admin Setup)

Run these commands once in your server (you're an admin because of `OWNER_IDS`):

1. **Set the node-status auto-update channel:**
   ```
   /setchannel type:Node status auto-update channel:#status
   ```
   The bot will post and refresh a node status panel there every 3 minutes.

2. **(Optional) Set a log channel** for audits:
   ```
   /setchannel type:Log channel channel:#bot-logs
   ```

3. **(Optional) Tweak shop plans:**
   ```
   /plan list
   /plan add key:starter name:Starter Plan price:300 ram:1024 cpu:100 disk:10240 emoji:🌱
   /plan toggle key:starter enabled:true
   ```
   The default plans (Starter, Basic, Standard, Premium, Extreme) are already seeded, matching the spec.

---

## 🔧 Step 8 — Test the Flow

As a regular user (or with a second Discord account), try this end-to-end flow:

1. `/daily` — claim free starter coins
2. `/work` — earn more coins
3. `/balance` — confirm wallet balance
4. `/genaccount` — bot DMs you a Pterodactyl panel email + password
5. Log in to the panel with those credentials and verify the account exists
6. `/shop` — browse plans
7. `/buy plan:starter` — spend coins to buy
8. `/createserver plan:starter name:my-mc-server` — bot provisions a server on Pterodactyl
9. `/server-status` — see CPU/RAM/Disk progress bars
10. `/manage` — get the button-based control panel (Start / Stop / Restart / Status)

---

## 🛠️ Common Issues

| Problem | Fix |
| --- | --- |
| Bot is online but `/commands` don't appear | You skipped `npm run deploy`, or you set `DISCORD_DEV_GUILD_IDS` empty (then global commands take up to an hour). Re-run `npm run deploy`. |
| `genaccount` fails with `403` / `401` | Your `PTERO_APP_API_KEY` is missing the **users** read+write permission. Recreate it with all permissions ticked. |
| `createserver` fails with "No free allocation" | You haven't created any allocations on the node. In the panel: Admin → Nodes → your node → Allocations → Create. |
| `server-start` etc. fail with "401 Unauthorized" | Your `PTERO_CLIENT_API_KEY` is wrong, or the user has not run `/link` with their own key. The fallback global client key only works for servers owned by **the user who created that key**. For per-user control, ask each user to run `/link <their_key>` after `genaccount`. |
| Bot crashes on `npm install` (better-sqlite3 build error) | Install build tools: on Debian/Ubuntu `sudo apt install build-essential python3`. On Windows install **Visual Studio Build Tools** (C++ workload). On the Pterodactyl egg, switch to the `node:20` image which already has them. |
| `npm run deploy` says `Invalid Form Body` | Check your slash command names are all lowercase, no spaces. (The shipped commands already comply.) |

---

## 📂 Project Layout (For Reference)

```
hivecore-bot/
├── .env.example
├── package.json
├── README.md
├── data/                 # SQLite DB (auto-created)
└── src/
    ├── index.js          # Entry point
    ├── deploy-commands.js
    ├── api/              # Pterodactyl API wrapper
    ├── config/           # .env loader
    ├── database/         # All SQLite models
    ├── events/           # Discord event handlers
    ├── tasks/            # Scheduled tasks (node status updater)
    ├── utils/            # Logger, embeds, cooldowns, helpers
    └── commands/
        ├── account/      genaccount, link, unlink
        ├── admin/        setcoins, addcoins, reset, drop, plan, setchannel, admin-deleteserver
        ├── bank/         bank, deposit, withdraw
        ├── economy/      balance, daily, work, give, leaderboard, stats, profile
        ├── games/        coinflip, dice, slots, duel, rob, spin
        ├── info/         help, ping
        ├── nodes/        nodes
        ├── server/       server-status, server-start, server-stop, server-restart, server-kill, server-console, manage
        └── shop/         shop, buy, createserver
```

---

## 🚀 Running 24/7

If you're running on your computer, the bot stops when you close the terminal. To keep it online 24/7:

- **Recommended:** deploy on your **Pterodactyl Node.js egg** (you already have a panel).
- **Linux VPS:** use `pm2`:
  ```bash
  npm install -g pm2
  pm2 start src/index.js --name hivecore-bot
  pm2 save
  pm2 startup
  ```
- **Windows:** use `pm2` the same way, or set up a Task Scheduler entry that runs `npm start` on boot.

---

## 💬 Need help?

Re-read the matching section of `README.md` (in the zip) — it has more detail on every command and configuration option. If something specific is broken, share the **last 30 lines of console output** and I can debug it.

Have fun! 🐝


Mady By TheRynzo. For Any Copyright Contact Me.
