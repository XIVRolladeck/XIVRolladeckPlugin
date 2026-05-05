# XIV Rolladeck

A [Dalamud](https://goatcorp.github.io/) plugin for Final Fantasy XIV that brings the [XIV Rolladeck](https://xivrolladeck.com) live DJ and venue tracker directly into your game.

See who's spinning in Eorzea right now — and teleport there without ever leaving the game.

---

## Features

- **Live DJs** — See every DJ currently streaming from an in-game venue, including their name, stream viewer count, data center, world, venue name, and housing address.
- **Live Venues** — Browse the same data organized by venue instead of by DJ.
- **Track DJs & Venues** — Click the ★ star next to any DJ or venue name to track them. You'll see at a glance whether they're live or offline, and your saved list persists between sessions.
- **In-game notifications** — Get a Dalamud toast notification the moment a DJ or venue you've tracked goes live.
- **One-click teleport** — If you have the [Lifestream](https://github.com/NightmareXIV/Lifestream) plugin installed, you can teleport directly to any venue's housing address with a single button.
- **Lifestream address book** — Save a venue's address to Lifestream's address book under a "Rolladeck addresses" folder so you can revisit it later.
- **Venue detection** — When you enter a player housing plot where a DJ is streaming live, you automatically receive a clickable Dalamud notification with the DJ's name and banner. An optional Echo chat line with a watch link is also printed. All five housing districts are supported.
- **Account sync** — Link your xivrolladeck.com account using a short link code from your web dashboard. Once linked, your saved DJs and venues sync across all game installs and devices. Favorites you save in-game are reflected on the website instantly, and vice versa.
- **Party Finder companion** — When you open the Party Finder window, XIV Rolladeck automatically opens beside it and matches its height so you can browse live DJs and venues side by side while you recruit. Optionally closes when Party Finder closes.
- **Search & filter** — Filter the live list by DJ name or venue name, and set a default Region, Data Center, or World in settings so your home region is always front and center.
- **Auto-refresh** — The live list updates automatically in the background. The interval is configurable from 30 seconds to 10 minutes.

---

## Installation

XIV Rolladeck is distributed as a custom Dalamud repository. Updates are delivered automatically through the plugin installer once you've added the repo.

1. Open the Dalamud settings (`/xlsettings`) and go to **Experimental**.
2. Add the following custom repository URL:
   ```
   https://plugin.xivrolladeck.com/repo.json
   ```
3. Click **Save & Close**, then open the plugin installer (`/xlplugins`), search for **XIVRolladeck**, and install it.

---

## Usage

Type `/rolladeck` in chat to open the main window. You can also open it from the Dalamud plugin list.

### Tabs

| Tab | What it shows |
|-----|---------------|
| **Live DJs** | All DJs currently live, with their viewer count, world, venue, and address |
| **Live Venues** | All open venues, organized by location |
| **Saved DJs** | Your tracked DJs — split into live now and offline. Shows as **Fav DJs** when linked to a web account. |
| **Saved Venues** | Your tracked venues — split into live now and offline. Shows as **Fav Venues** when linked to a web account. |

### DJ & Venue Cards

Each card has a **★ star toggle** next to the name to track or untrack, plus a row of action buttons:

- **Save Address** — Saves the housing address to Lifestream's address book under a "Rolladeck addresses" folder *(requires Lifestream plugin)*.
- **Go to Venue** — Teleports you to the venue using Lifestream *(requires Lifestream plugin)*.
- **Open Stream** — Opens the DJ's stream. By default this uses the Rolladeck watch page (`xivrolladeck.com/?watch=username`); you can switch to direct Twitch links in settings.

---

## Configuration

Open the config window by clicking the **⚙** icon in the main window header, or via the Dalamud plugin settings.

### General

| Setting | Default | Description |
|---------|---------|-------------|
| Show viewer count | On | Displays the Twitch viewer count on each DJ card |
| Open streams using Rolladeck watch page | On | Opens `xivrolladeck.com/?watch=...` instead of going directly to Twitch |
| Enable auto-refresh | On | Automatically refreshes the live list in the background |
| Auto-refresh interval | 360s | How often to refresh, from 30 to 600 seconds |

### Venue Detection

| Setting | Default | Description |
|---------|---------|-------------|
| Enable venue detection | On | Notifies you when you enter a housing plot where a DJ is streaming live |
| Show chat message | On | Prints an Echo channel line with a watch link on detection |
| Show notification expanded | On | Toast starts expanded with the DJ's banner visible; uncheck to start collapsed |
| Re-notify cooldown | 30 min | How long before the same address can trigger a notification again |

### Party Finder

| Setting | Default | Description |
|---------|---------|-------------|
| Attach to Party Finder window | On | Automatically opens XIV Rolladeck beside Party Finder and matches its height |
| Close when Party Finder closes | Off | Closes the plugin window when you close Party Finder |

### Filters

Set a default Region, Data Center, or World so your home server is always front and center. Also configures default sort orders for each tab.

### Account

Link your xivrolladeck.com account to sync saved DJs and venues across all devices:

1. On the website, go to **Dashboard → Settings** and click **Link Plugin** to generate a 6-character link code.
2. In the plugin, open **Settings → Account**, enter the code, and click **Link Account**.

Once linked, favorites sync automatically on every refresh. Use **Sync Now** to pull the latest from the web immediately, or **Unlink This Install** to remove this game install from your account.

---

## Requirements

- **Final Fantasy XIV** (obviously)
- **Dalamud** API level 15 or higher
- **[Lifestream](https://github.com/NightmareXIV/Lifestream)** *(optional)* — required for the "Go to Venue" teleport and address book features

---

## About XIV Rolladeck

[XIV Rolladeck](https://xivrolladeck.com) is a community platform for tracking live DJ sets happening in FFXIV player housing. DJs register their streams on the website, and the data is published to a live API that this plugin reads from.

This plugin is the official companion plugin for the XIV Rolladeck platform.
