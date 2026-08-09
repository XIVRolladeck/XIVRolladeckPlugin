# XIV Rolladeck API Documentation
## Dalamud Plugin Integration Guide

---

### Base URL

```
https://us-central1-xiv-rolladeck.cloudfunctions.net/
```

All endpoints below are appended to this base. Example:
```
https://us-central1-xiv-rolladeck.cloudfunctions.net/apiV1Live
```

---

### Authentication

**All read endpoints are public — no authentication required.**

The only authenticated endpoints are `triggerDiscordSync` (admin-only) and `exchangeDiscordCode` (requires a signed-in Firebase user). Plugin integrations will never need these.

---

### Rate Limiting & Caching

| Endpoint | Server Cache TTL | Recommended Poll Interval |
|----------|-----------------|--------------------------|
| `apiV1Live` | 2 minutes | Every 5 minutes |
| `apiV1Djs` | 5 minutes | Every 30 minutes |
| `apiV1Venues` | 5 minutes | Every 30 minutes |
| `apiV1Events` | 10 minutes | Every 15 minutes |
| `apiV1Schedule` | 5 minutes | Every 15 minutes |
| `apiLiveDjs` *(legacy)* | 2 minutes | Every 5 minutes |

Polling more frequently than the cache TTL returns cached data without reducing latency.

---

## Endpoints

---

### `GET /apiV1Live` *(primary live data endpoint)*

Returns all currently live DJs, open venues, and active events in a single call. This is the primary endpoint for plugin integrations that display live activity.

**Auth**: None  
**Method**: GET  
**Cache**: `public, max-age=120`

#### Request

```bash
curl https://us-central1-xiv-rolladeck.cloudfunctions.net/apiV1Live
```

#### Response (200 OK)

```json
{
  "generatedAt": "2026-08-09T02:00:00.000Z",
  "liveDJs": [...],
  "openVenues": [...],
  "activeEvents": [...]
}
```

---

#### `liveDJs` Array

Each entry is a DJ currently streaming on Twitch who has a matched Rolladeck profile.

```json
{
  "djName": "DJ Neon",
  "djId": "dj-1769141951305",
  "djSlug": "dj-neon",
  "twitchUrl": "https://twitch.tv/djneon",
  "avatarUrl": "https://static-cdn.jtvnw.net/...",
  "genres": ["Deep House", "Ambient"],
  "viewerCount": 42,
  "streamTitle": "🎵 DJ Neon @ The Crystal Lounge | Crystal | LB W12 P8",
  "startedAt": "2026-08-09T01:30:00.000Z",
  "venueName": "The Crystal Lounge",
  "venueId": "venue-1769732219724",
  "venueSlug": "the-crystal-lounge-balmung",
  "server": "Balmung",
  "datacenter": "Crystal",
  "district": "Lavender Beds",
  "ward": 12,
  "plot": 8,
  "locationType": "housing",
  "lifestream": "/li Balmung Lavender Beds 12 8",
  "eventName": "Moonrise Festival",
  "eventId": "evt-abc123",
  "eventStart": "2026-08-09T01:00:00.000Z",
  "eventEnd": "2026-08-09T04:00:00.000Z",
  "bio": "Deep house and ambient from Crystal since 2022.",
  "discordName": "djneon",
  "twitter": "https://twitter.com/djneon",
  "bluesky": "https://bsky.app/profile/djneon.bsky.social",
  "instagram": "https://instagram.com/djneon",
  "youtube": "https://youtube.com/@djneon",
  "tiktok": "https://tiktok.com/@djneon",
  "website": "https://djneon.carrd.co",
  "musicPlatform": "https://soundcloud.com/djneon"
}
```

##### Per-DJ Field Reference

| Field | Type | Description | Nullable |
|-------|------|-------------|----------|
| `djName` | string | DJ's display name | No |
| `djId` | string | Firestore doc ID (`dj-{n}`). Use for account sync and deduplication. | Yes |
| `djSlug` | string | DJ's profile slug (`/dj/{slug}` on xivrolladeck.com) | Yes |
| `twitchUrl` | string | Full Twitch stream URL | Yes |
| `avatarUrl` | string | DJ's Twitch profile avatar URL | Yes |
| `genres` | string[] | Up to 3 genres from the DJ's profile | No (empty array) |
| `viewerCount` | number | Current Twitch viewer count | No |
| `streamTitle` | string | Current Twitch stream title | Yes |
| `startedAt` | ISO 8601 string | When the stream started | Yes |
| `venueName` | string | Matched Rolladeck venue name | Yes |
| `venueId` | string | Firestore doc ID for the matched venue | Yes |
| `venueSlug` | string | Venue's profile slug | Yes |
| `server` | string | FFXIV world/server (e.g. `"Balmung"`) | Yes |
| `datacenter` | string | FFXIV datacenter (e.g. `"Crystal"`) | Yes |
| `district` | string | Housing district (e.g. `"Lavender Beds"`) | Yes |
| `ward` | number | Ward number | Yes |
| `plot` | number | Plot number | Yes |
| `locationType` | string | `"housing"`, `"city-state"`, or `"open-world"` | Yes |
| `lifestream` | string | Ready-to-use `/li` command string | Yes |
| `eventName` | string | Active Rolladeck event at this DJ's address | Yes |
| `eventId` | string | Firestore event doc ID | Yes |
| `eventStart` | ISO 8601 string | Event start time | Yes |
| `eventEnd` | ISO 8601 string | Event end time | Yes |
| `bio` | string | DJ's profile bio | Yes |
| `discordName` | string | DJ's Discord username | Yes |
| `twitter` | string | Twitter/X profile URL | Yes |
| `bluesky` | string | Bluesky profile URL | Yes |
| `instagram` | string | Instagram profile URL | Yes |
| `youtube` | string | YouTube channel URL | Yes |
| `tiktok` | string | TikTok profile URL | Yes |
| `website` | string | Personal website or Carrd URL | Yes |
| `musicPlatform` | string | Music platform URL (SoundCloud, Mixcloud, etc.) | Yes |

> **Prefer `lifestream` over parsing `district`/`ward`/`plot`** — it gives you the exact `/li` command ready to execute, including the server name.

---

#### `openVenues` Array

Each entry is a venue currently open — either via a live DJ stream, an active Rolladeck event, or an active Discord scheduled event.

```json
{
  "id": "venue-1769732219724",
  "name": "The Crystal Lounge",
  "slug": "the-crystal-lounge-balmung",
  "server": "Balmung",
  "district": "Lavender Beds",
  "ward": 12,
  "plot": 8,
  "datacenter": "Crystal",
  "region": "NA",
  "logoUrl": "https://firebasestorage.googleapis.com/...",
  "bannerUrl": "https://firebasestorage.googleapis.com/...",
  "lifestream": "/li Balmung Lavender Beds 12 8",
  "openReason": "stream",
  "eventName": null,
  "eventStart": null,
  "eventEnd": null,
  "djName": "DJ Neon",
  "djTwitch": "https://twitch.tv/djneon",
  "djId": "dj-1769141951305",
  "activeDiscordEventName": null,
  "activeDiscordEventStart": null,
  "activeDiscordEventEnd": null,
  "activeDiscordEventImage": null,
  "description": "A deep house and ambient venue on Balmung since 2022.",
  "firstOpened": "2022-03-15T00:00:00.000Z",
  "websiteOrCarrd": "https://crystallounge.carrd.co",
  "discordServer": "https://discord.gg/example",
  "amenities": ["SFW", "Requests", "Tipping"]
}
```

##### Open Venue Field Reference

| Field | Type | Description | Nullable |
|-------|------|-------------|----------|
| `id` | string | Firestore doc ID (`venue-{n}`) | No |
| `name` | string | Venue display name | Yes |
| `slug` | string | Venue profile slug (`/venue/{slug}`) | Yes |
| `server` | string | FFXIV world/server | Yes |
| `district` | string | Housing district | Yes |
| `ward` | number | Ward number | Yes |
| `plot` | number | Plot number | Yes |
| `datacenter` | string | FFXIV datacenter | Yes |
| `region` | string | Region (`"NA"`, `"EU"`, `"JP"`, `"OCE"`) | Yes |
| `logoUrl` | string | Venue logo image URL | Yes |
| `bannerUrl` | string | Venue banner image URL | Yes |
| `lifestream` | string | Ready-to-use `/li` command string | Yes |
| `openReason` | string | Why this venue appears: `"stream"`, `"event"`, or `"discord"` | No |
| `eventName` | string | Name of the active Rolladeck event (when `openReason` is `"event"`) | Yes |
| `eventStart` | ISO 8601 string | Rolladeck event start time | Yes |
| `eventEnd` | ISO 8601 string | Rolladeck event end time | Yes |
| `djName` | string | Name of the live DJ playing here (when `openReason` is `"stream"`) | Yes |
| `djTwitch` | string | Twitch URL of the live DJ | Yes |
| `djId` | string | Firestore DJ doc ID | Yes |
| `activeDiscordEventName` | string | Active Discord event name (when `openReason` is `"discord"`) | Yes |
| `activeDiscordEventStart` | ISO 8601 string | Discord event start time | Yes |
| `activeDiscordEventEnd` | ISO 8601 string | Discord event end time | Yes |
| `activeDiscordEventImage` | string | Discord event cover image URL | Yes |
| `description` | string | Venue description / about text | Yes |
| `firstOpened` | ISO 8601 string | Date the venue first opened | Yes |
| `websiteOrCarrd` | string | Venue website or Carrd URL | Yes |
| `discordServer` | string | Discord server invite URL | Yes |
| `amenities` | string[] | List of venue amenities (e.g. `"SFW"`, `"Requests"`) | No (empty array) |

---

#### `activeEvents` Array

All Rolladeck events currently active or starting within the next 24 hours. Applies a 30-minute grace period after `endDate`.

```json
{
  "id": "evt-abc123",
  "name": "Moonrise Festival",
  "description": "A night of deep house and ambient sets.",
  "startDate": "2026-08-09T01:00:00.000Z",
  "endDate": "2026-08-09T04:00:00.000Z",
  "bannerUrl": "https://firebasestorage.googleapis.com/...",
  "website": null,
  "discordServer": "https://discord.gg/example",
  "lineup": [
    {
      "djName": "DJ Neon",
      "twitchUsername": "djneon",
      "djId": "1769141951305",
      "timeSlot": "9:00 PM – 11:00 PM"
    }
  ],
  "locationType": "housing",
  "server": "Balmung",
  "district": "Lavender Beds",
  "ward": 12,
  "plot": 8,
  "datacenter": "Crystal",
  "region": "NA",
  "venueId": "venue-1769732219724"
}
```

##### Active Event Field Reference

| Field | Type | Description | Nullable |
|-------|------|-------------|----------|
| `id` | string | Firestore document ID | No |
| `name` | string | Event name | No |
| `description` | string | Event description | Yes |
| `startDate` | ISO 8601 string | Event start time (UTC) | Yes |
| `endDate` | ISO 8601 string | Event end time (UTC) | Yes |
| `bannerUrl` | string | Event banner/poster image URL | Yes |
| `website` | string | External event website URL | Yes |
| `discordServer` | string | Discord server invite URL | Yes |
| `lineup` | array | Ordered performer list (see below) | No (empty array) |
| `locationType` | string | `"housing"`, `"city-state"`, or `"open-world"` | Yes |
| `server` | string | FFXIV world/server | Yes |
| `district` | string | Housing district | Yes |
| `ward` | number | Ward number | Yes |
| `plot` | number | Plot number | Yes |
| `datacenter` | string | FFXIV datacenter | Yes |
| `region` | string | Region code | Yes |
| `venueId` | string | Linked venue Firestore doc ID | Yes |

##### Lineup Entry Fields

| Field | Type | Description | Nullable |
|-------|------|-------------|----------|
| `djName` | string | DJ display name | No |
| `twitchUsername` | string | Twitch username (bare, no URL prefix) | Yes |
| `djId` | string | Firestore DJ doc ID — numeric string without `dj-` prefix | Yes |
| `timeSlot` | string | Human-readable time slot (e.g. `"9:00 PM – 11:00 PM"`) | Yes |

---

### `GET /apiV1Djs`

Full directory of active Rolladeck DJs. Use this to build a local DJ index for lookup or enrichment.

**Auth**: None  
**Method**: GET  
**Cache**: `public, max-age=300`

#### Response (200 OK)

```json
{
  "generatedAt": "2026-08-09T02:00:00.000Z",
  "count": 247,
  "djs": [
    {
      "id": "dj-1769141951305",
      "name": "DJ Neon",
      "slug": "dj-neon",
      "twitch": "djneon",
      "genres": ["Deep House", "Ambient"],
      "regions": ["NA"],
      "language": "English",
      "avatarSmall": "https://static-cdn.jtvnw.net/...",
      "avatarLarge": "https://static-cdn.jtvnw.net/...",
      "twitchAvatar": "https://static-cdn.jtvnw.net/...",
      "lastSeenLive": "2026-08-08T21:00:00.000Z",
      "discordServer": "https://discord.gg/example",
      "twitter": "https://twitter.com/djneon",
      "bluesky": "https://bsky.app/profile/djneon.bsky.social",
      "instagram": "https://instagram.com/djneon",
      "youtube": "https://youtube.com/@djneon",
      "website": "https://djneon.carrd.co",
      "mixcloud": null,
      "musicPlatform": "https://soundcloud.com/djneon",
      "tiktok": null
    }
  ]
}
```

**Notes**: Excludes DJs with `activityStatus` of `inactive` or `retired`.

---

### `GET /apiV1Venues`

Full directory of active Rolladeck venues. Use this to build a local venue index for location matching.

**Auth**: None  
**Method**: GET  
**Cache**: `public, max-age=300`

#### Response (200 OK)

```json
{
  "generatedAt": "2026-08-09T02:00:00.000Z",
  "count": 183,
  "venues": [
    {
      "id": "venue-1769732219724",
      "name": "The Crystal Lounge",
      "slug": "the-crystal-lounge-balmung",
      "server": "Balmung",
      "district": "Lavender Beds",
      "ward": 12,
      "plot": 8,
      "datacenter": "Crystal",
      "region": "NA",
      "logoUrl": "https://firebasestorage.googleapis.com/...",
      "bannerUrl": "https://firebasestorage.googleapis.com/...",
      "websiteOrCarrd": "https://crystallounge.carrd.co",
      "discordServer": "https://discord.gg/example",
      "amenities": ["SFW", "Requests"],
      "sfw": true,
      "lifestream": "/li Balmung Lavender Beds 12 8",
      "isOpenViaDiscord": false,
      "activeDiscordEventName": null,
      "activeDiscordEventStart": null,
      "activeDiscordEventEnd": null,
      "activeDiscordEventImage": null,
      "upcomingDiscordEvents": [
        {
          "id": "1234567890",
          "name": "Saturday Night House",
          "image": null,
          "startTime": "2026-08-10T01:00:00.000Z",
          "endTime": "2026-08-10T04:00:00.000Z"
        }
      ],
      "nextDiscordEventName": "Saturday Night House"
    }
  ]
}
```

**Notes**: Excludes venues marked `permanently_closed`, `status: 'closed'`, or `banStatus: 'banned'`, and those missing any address component. `sfw` is `true` when `"nsfw"` is not in the amenities array.

---

### `GET /apiV1Events`

Upcoming and active Rolladeck events within a configurable look-ahead window.

**Auth**: None  
**Method**: GET  
**Cache**: `public, max-age=600`  
**Query param**: `?days=N` — look-ahead window in days (default: 14, min: 1, max: 60)

#### Response (200 OK)

```json
{
  "generatedAt": "2026-08-09T02:00:00.000Z",
  "days": 14,
  "count": 12,
  "events": [
    {
      "id": "evt-abc123",
      "name": "Moonrise Festival",
      "description": "A night of deep house and ambient sets.",
      "startDate": "2026-08-09T01:00:00.000Z",
      "endDate": "2026-08-09T04:00:00.000Z",
      "bannerUrl": "https://firebasestorage.googleapis.com/...",
      "website": null,
      "discordServer": "https://discord.gg/example",
      "lineup": [
        {
          "djName": "DJ Neon",
          "twitchUsername": "djneon",
          "djId": "1769141951305",
          "timeSlot": "9:00 PM – 11:00 PM"
        }
      ],
      "locationType": "housing",
      "server": "Balmung",
      "district": "Lavender Beds",
      "ward": 12,
      "plot": 8,
      "datacenter": "Crystal",
      "region": "NA",
      "venueId": "venue-1769732219724"
    }
  ]
}
```

**Notes**: Applies a 30-minute post-end grace period. Results sorted ascending by `startDate`.

---

### `GET /apiV1Schedule`

Unified schedule aggregating manual DJ sets, confirmed bookings, Discord scheduled events, and Rolladeck events.

**Auth**: None  
**Method**: GET  
**Cache**: `public, max-age=300`  
**Query param**: `?days=N` — look-ahead window (default: 14, min: 1, max: 60)

#### Response (200 OK)

```json
{
  "generatedAt": "2026-08-09T02:00:00.000Z",
  "days": 14,
  "count": 34,
  "schedule": [
    {
      "type": "set",
      "id": "set-abc123",
      "name": "Saturday Night House",
      "djName": "DJ Neon",
      "djId": "dj-1769141951305",
      "djSlug": "dj-neon",
      "avatarUrl": "https://static-cdn.jtvnw.net/...",
      "venueName": "The Crystal Lounge",
      "venueId": "venue-1769732219724",
      "venueSlug": "the-crystal-lounge-balmung",
      "startDate": "2026-08-10T01:00:00.000Z",
      "endDate": "2026-08-10T03:00:00.000Z",
      "server": "Balmung",
      "datacenter": "Crystal",
      "district": "Lavender Beds",
      "ward": 12,
      "plot": 8
    }
  ]
}
```

##### Schedule Entry `type` Values

| `type` | Source |
|--------|--------|
| `"set"` | DJ manual sets (`dj-manual-sets` collection) |
| `"booking"` | Confirmed DJ bookings (`bookings` collection) |
| `"discord"` | Discord scheduled events from linked guild |
| `"event"` | Curated Rolladeck events (`events` collection) |

---

### `GET /apiLiveDjs` *(legacy — still active)*

The original plugin endpoint. Still fully functional. Prefer `apiV1Live` for new integrations.

**Auth**: None  
**Method**: GET  
**Cache**: `public, max-age=120`

#### Key differences from `apiV1Live`

- Response root uses `liveDjs` (not `liveDJs`) and `events` (not `activeEvents`)
- `liveDjs` entries use `world` and `dataCenter` instead of `server` and `datacenter`
- `liveDjs` entries include `address` (legacy formatted string), `djBannerUrl`, `venueBannerUrl`, `venueLogoUrl`
- `liveDjs` entries include `genres` array (added 2026-05-01)
- `events` array includes `isOfficial`, `isFeatured`, `isCharity`, `charityInfo` flags
- No `openVenues` section
- No DJ social/bio fields

See the [Field Reference](#field-reference-legacy) section below for the complete legacy schema.

---

## Location Types

The `locationType` field determines which location fields are populated:

| `locationType` | Populated fields | Example |
|----------------|-----------------|---------|
| `"housing"` | `district`, `ward`, `plot`, `lifestream` | `"Lavender Beds, Ward 12, Plot 8"` |
| `"city-state"` | `cityState` *(legacy `apiLiveDjs` only)* | `"Limsa Lominsa"` |
| `"open-world"` | `zone`, `coordinates` *(legacy `apiLiveDjs` only)* | `"Il Mheg (15, 30)"` |

Non-housing types are rare. Housing is the overwhelming majority of venues.

---

## C# Models (System.Text.Json)

```csharp
using System.Collections.Generic;
using System.Text.Json.Serialization;

// ── apiV1Live ────────────────────────────────────────────────────────────────

public class V1LiveResponse
{
    [JsonPropertyName("generatedAt")]  public string           GeneratedAt  { get; set; } = "";
    [JsonPropertyName("liveDJs")]      public List<V1LiveDj>   LiveDJs      { get; set; } = new();
    [JsonPropertyName("openVenues")]   public List<V1OpenVenue> OpenVenues  { get; set; } = new();
    [JsonPropertyName("activeEvents")] public List<V1Event>    ActiveEvents { get; set; } = new();
}

public class V1LiveDj
{
    [JsonPropertyName("djName")]        public string   DjName        { get; set; } = "";
    [JsonPropertyName("djId")]          public string?  DjId          { get; set; }
    [JsonPropertyName("djSlug")]        public string?  DjSlug        { get; set; }
    [JsonPropertyName("twitchUrl")]     public string?  TwitchUrl     { get; set; }
    [JsonPropertyName("avatarUrl")]     public string?  AvatarUrl     { get; set; }
    [JsonPropertyName("genres")]        public List<string> Genres    { get; set; } = new();
    [JsonPropertyName("viewerCount")]   public int      ViewerCount   { get; set; }
    [JsonPropertyName("streamTitle")]   public string?  StreamTitle   { get; set; }
    [JsonPropertyName("startedAt")]     public string?  StartedAt     { get; set; }
    [JsonPropertyName("venueName")]     public string?  VenueName     { get; set; }
    [JsonPropertyName("venueId")]       public string?  VenueId       { get; set; }
    [JsonPropertyName("venueSlug")]     public string?  VenueSlug     { get; set; }
    [JsonPropertyName("server")]        public string?  Server        { get; set; }
    [JsonPropertyName("datacenter")]    public string?  Datacenter    { get; set; }
    [JsonPropertyName("district")]      public string?  District      { get; set; }
    [JsonPropertyName("ward")]          public int?     Ward          { get; set; }
    [JsonPropertyName("plot")]          public int?     Plot          { get; set; }
    [JsonPropertyName("locationType")]  public string?  LocationType  { get; set; }
    [JsonPropertyName("lifestream")]    public string?  Lifestream    { get; set; }
    [JsonPropertyName("eventName")]     public string?  EventName     { get; set; }
    [JsonPropertyName("eventId")]       public string?  EventId       { get; set; }
    [JsonPropertyName("eventStart")]    public string?  EventStart    { get; set; }
    [JsonPropertyName("eventEnd")]      public string?  EventEnd      { get; set; }
    [JsonPropertyName("bio")]           public string?  Bio           { get; set; }
    [JsonPropertyName("discordName")]   public string?  DiscordName   { get; set; }
    [JsonPropertyName("twitter")]       public string?  Twitter       { get; set; }
    [JsonPropertyName("bluesky")]       public string?  Bluesky       { get; set; }
    [JsonPropertyName("instagram")]     public string?  Instagram     { get; set; }
    [JsonPropertyName("youtube")]       public string?  Youtube       { get; set; }
    [JsonPropertyName("tiktok")]        public string?  Tiktok        { get; set; }
    [JsonPropertyName("website")]       public string?  Website       { get; set; }
    [JsonPropertyName("musicPlatform")] public string?  MusicPlatform { get; set; }
}

public class V1OpenVenue
{
    [JsonPropertyName("id")]                    public string   Id                    { get; set; } = "";
    [JsonPropertyName("name")]                  public string?  Name                  { get; set; }
    [JsonPropertyName("slug")]                  public string?  Slug                  { get; set; }
    [JsonPropertyName("server")]                public string?  Server                { get; set; }
    [JsonPropertyName("district")]              public string?  District              { get; set; }
    [JsonPropertyName("ward")]                  public int?     Ward                  { get; set; }
    [JsonPropertyName("plot")]                  public int?     Plot                  { get; set; }
    [JsonPropertyName("datacenter")]            public string?  Datacenter            { get; set; }
    [JsonPropertyName("region")]                public string?  Region                { get; set; }
    [JsonPropertyName("logoUrl")]               public string?  LogoUrl               { get; set; }
    [JsonPropertyName("bannerUrl")]             public string?  BannerUrl             { get; set; }
    [JsonPropertyName("lifestream")]            public string?  Lifestream            { get; set; }
    [JsonPropertyName("openReason")]            public string   OpenReason            { get; set; } = "";
    [JsonPropertyName("eventName")]             public string?  EventName             { get; set; }
    [JsonPropertyName("eventStart")]            public string?  EventStart            { get; set; }
    [JsonPropertyName("eventEnd")]              public string?  EventEnd              { get; set; }
    [JsonPropertyName("djName")]                public string?  DjName                { get; set; }
    [JsonPropertyName("djTwitch")]              public string?  DjTwitch              { get; set; }
    [JsonPropertyName("djId")]                  public string?  DjId                  { get; set; }
    [JsonPropertyName("activeDiscordEventName")]  public string? ActiveDiscordEventName  { get; set; }
    [JsonPropertyName("activeDiscordEventStart")] public string? ActiveDiscordEventStart { get; set; }
    [JsonPropertyName("activeDiscordEventEnd")]   public string? ActiveDiscordEventEnd   { get; set; }
    [JsonPropertyName("activeDiscordEventImage")] public string? ActiveDiscordEventImage { get; set; }
    [JsonPropertyName("description")]           public string?  Description           { get; set; }
    [JsonPropertyName("firstOpened")]           public string?  FirstOpened           { get; set; }
    [JsonPropertyName("websiteOrCarrd")]        public string?  WebsiteOrCarrd        { get; set; }
    [JsonPropertyName("discordServer")]         public string?  DiscordServer         { get; set; }
    [JsonPropertyName("amenities")]             public List<string> Amenities         { get; set; } = new();
}

public class V1Event
{
    [JsonPropertyName("id")]            public string   Id           { get; set; } = "";
    [JsonPropertyName("name")]          public string?  Name         { get; set; }
    [JsonPropertyName("description")]   public string?  Description  { get; set; }
    [JsonPropertyName("startDate")]     public string?  StartDate    { get; set; }
    [JsonPropertyName("endDate")]       public string?  EndDate      { get; set; }
    [JsonPropertyName("bannerUrl")]     public string?  BannerUrl    { get; set; }
    [JsonPropertyName("website")]       public string?  Website      { get; set; }
    [JsonPropertyName("discordServer")] public string?  DiscordServer { get; set; }
    [JsonPropertyName("lineup")]        public List<V1LineupEntry> Lineup { get; set; } = new();
    [JsonPropertyName("locationType")]  public string?  LocationType { get; set; }
    [JsonPropertyName("server")]        public string?  Server       { get; set; }
    [JsonPropertyName("district")]      public string?  District     { get; set; }
    [JsonPropertyName("ward")]          public int?     Ward         { get; set; }
    [JsonPropertyName("plot")]          public int?     Plot         { get; set; }
    [JsonPropertyName("datacenter")]    public string?  Datacenter   { get; set; }
    [JsonPropertyName("region")]        public string?  Region       { get; set; }
    [JsonPropertyName("venueId")]       public string?  VenueId      { get; set; }
}

public class V1LineupEntry
{
    [JsonPropertyName("djName")]         public string  DjName         { get; set; } = "";
    [JsonPropertyName("twitchUsername")] public string? TwitchUsername { get; set; }
    [JsonPropertyName("djId")]           public string? DjId           { get; set; }
    [JsonPropertyName("timeSlot")]       public string? TimeSlot       { get; set; }
}
```

---

## Recommended Implementation

```csharp
using System;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

public class RolladeckClient : IDisposable
{
    private readonly HttpClient _http;
    private const string BaseUrl = "https://us-central1-xiv-rolladeck.cloudfunctions.net/";

    public RolladeckClient(string userAgent)
    {
        _http = new HttpClient();
        _http.DefaultRequestHeaders.Add("User-Agent", userAgent);
    }

    public async Task<V1LiveResponse?> GetLiveAsync()
    {
        var response = await _http.GetAsync(BaseUrl + "apiV1Live");
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<V1LiveResponse>(json);
    }

    public void Dispose() => _http.Dispose();
}

// Poll every 5 minutes
public async Task RefreshLiveData()
{
    try
    {
        var data = await _client.GetLiveAsync();
        if (data == null) return;

        UpdateDJList(data.LiveDJs);
        UpdateVenueList(data.OpenVenues);
        UpdateEventList(data.ActiveEvents);
    }
    catch (Exception ex)
    {
        Log.Error($"Rolladeck API error: {ex.Message}");
    }
}
```

---

## Lifestream Integration

```csharp
public void TeleportToVenue(V1OpenVenue venue)
{
    if (!string.IsNullOrEmpty(venue.Lifestream))
        Chat.SendMessage(venue.Lifestream); // e.g. "/li Balmung Lavender Beds 12 8"
}
```

---

## Error Handling

| Status Code | Meaning | Action |
|-------------|---------|--------|
| 200 | Success | Parse response |
| 405 | Method Not Allowed | Use GET |
| 500 | Server Error | Retry after cache TTL |

---

## Field Reference (Legacy `apiLiveDjs`)

The `apiLiveDjs` endpoint is still active. Its `liveDjs` entries differ from `apiV1Live` as follows:

| `apiLiveDjs` field | Notes |
|--------------------|-------|
| `world` | Server name — use `server` in `apiV1Live` |
| `dataCenter` | Datacenter name — use `datacenter` in `apiV1Live` |
| `address` | Legacy formatted string (e.g. `"Lavender Beds, Ward 12, Plot 8"`) — not present in `apiV1Live` |
| `avatarSmall` | DJ avatar — use `avatarUrl` in `apiV1Live` |
| `djBannerUrl` | DJ profile banner — not in `apiV1Live` |
| `venueBannerUrl` | Venue banner — not in `apiV1Live` |
| `venueLogoUrl` | Venue logo with site-logo fallback — not in `apiV1Live` (use `openVenues[].logoUrl`) |
| `slug` | Venue slug — use `venueSlug` in `apiV1Live` |
| `genres` | DJ genres array — present in both |

The `events` array in `apiLiveDjs` also includes `isOfficial`, `isFeatured`, `isCharity`, and `charityInfo` badge fields not present in `apiV1Live`'s `activeEvents`.

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2026-08-09 | Documented `apiV1Live` as primary endpoint with `openVenues` + `activeEvents`; added `apiV1Djs`, `apiV1Venues`, `apiV1Events`, `apiV1Schedule`; updated C# models to System.Text.Json; added DJ social/bio fields |
| 1.5 | 2026-04-14 | Added `events[]` (active + 24h lookahead) and `eventName/Id/Start/End` per-DJ annotation to `apiLiveDjs` |
| 1.4 | 2026-04-08 | Added `djId`, `venueId` for plugin account sync |
| 1.3 | 2026-03-28 | Added `djBannerUrl`, `venueBannerUrl`, `venueLogoUrl` |
| 1.2 | 2026-03-01 | Added `avatarSmall`, `viewerCount`, `streamTitle` |
| 1.1 | 2026-03-01 | Added `slug`, `district`, `ward`, `plot`, `lifestream`, `locationType`, `count` |
| 1.0 | 2026-03-01 | Initial release (`apiLiveDjs`) |

---

## Quick Reference

```
Base:   https://us-central1-xiv-rolladeck.cloudfunctions.net/
Auth:   None (all read endpoints are public)

Primary live endpoint:  GET /apiV1Live     (cache: 2 min,  poll: 5 min)
DJ directory:           GET /apiV1Djs      (cache: 5 min,  poll: 30 min)
Venue directory:        GET /apiV1Venues   (cache: 5 min,  poll: 30 min)
Upcoming events:        GET /apiV1Events   (cache: 10 min, poll: 15 min)
Unified schedule:       GET /apiV1Schedule (cache: 5 min,  poll: 15 min)
Legacy live:            GET /apiLiveDjs    (cache: 2 min,  poll: 5 min)

apiV1Live response shape:
  liveDJs[]       — live Twitch streams with Rolladeck profile match
  openVenues[]    — venues currently open (stream | event | discord)
  activeEvents[]  — Rolladeck events active or starting within 24h

Key DJ fields (liveDJs):
  djId / djSlug   — identity
  twitchUrl       — stream URL
  server          — world (e.g. Balmung)
  datacenter      — DC (e.g. Crystal)
  lifestream      — ready-to-use /li command (preferred over parsing address)
  genres[]        — up to 3 genres
  bio / twitter / bluesky / instagram / youtube / tiktok / website / musicPlatform

Key venue fields (openVenues):
  id / slug       — identity
  openReason      — "stream" | "event" | "discord"
  lifestream      — ready-to-use /li command
  djName/djTwitch — live DJ info (when openReason = "stream")
  eventName       — active Rolladeck event (when openReason = "event")
  activeDiscordEventName — active Discord event (when openReason = "discord")
  amenities[]     — venue amenity tags
  description / websiteOrCarrd / discordServer
```

---

## Support

- **Website**: https://xivrolladeck.com
- **API Issues**: Contact the XIV Rolladeck admin team via the Discord server
