# QuestsMarket

The community quest catalog for the **Golden Grace** habit-RPG app. The app reads this repo over
HTTPS (no server, no accounts, no tracking) and lets people search it, sort by rating, preview a
quest, and add it to their own list. New quests are added by **pull request**, so review is the
moderation.

> A "quest" is just a habit: something you do on a schedule that earns XP and keeps your character
> (and pet) thriving.

## Layout

A light index for browsing, and one folder per quest for the full detail (so we can add more to a
quest later — extra media, variants — without bloating the index):

```
catalog.json                 # the browse INDEX: array of summaries
bundles.json                 # themed packs: array of bundles (each a list of quest ids)
quests/
  morning-run/
    quest.json               # the quest's full parameters (no description text)
    description.md           # the description (or description.html)
    …                        # future: image.png, video.mp4, etc.
  drink-water/
    quest.json
    description.md
  …
```

## How the app uses it

1. Fetches **`catalog.json`** (the index) to build the browse list. Cached on-device, with a bundled
   copy as offline fallback.
2. When someone opens a quest, fetches **`quests/<id>/quest.json`** + **`quests/<id>/description.<md|html>`**.
3. **Add** turns it into a real, editable quest in their app — nothing here touches their data.

---

## `catalog.json` — the index

A single JSON **array** of summaries. Keep it light; it's just for the list, search, and sort.

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `id` | string | ✅ | Unique, kebab-case, stable. **Must match the folder name** `quests/<id>/`. |
| `name` | string | ✅ | Display name. |
| `category` | string | | Filter chip, e.g. Fitness, Health, Mind, Home, Finance, Social, Productivity. |
| `tags` | string[] | | Searchable keywords. |
| `rating` | number | | `0`–`5`. Editorial; drives sort. |
| `icon` | string | | An [SF Symbol](https://developer.apple.com/sf-symbols/) name. |
| `colorHex` | string | | `"#RRGGBB"`. |

```json
{ "id": "morning-run", "name": "Morning run", "category": "Fitness",
  "tags": ["cardio", "outdoors"], "rating": 4.7, "icon": "figure.run", "colorHex": "#E8743B" }
```

## `quests/<id>/quest.json` — the full quest

Everything needed to create the habit. The description text is **not** here — it lives in the
description file next to it. Only `id` and `name` are required.

| Field | Type | Notes |
|---|---|---|
| `id` | string | Same as the folder name and the index entry. |
| `name` | string | Display name. |
| `category`, `tags`, `rating`, `icon`, `colorHex` | — | Same meaning as the index (repeated here so the detail is self-contained). |
| `recurrence` | string | `"daily"` (default) · `"weekly"` · `"monthly"` · `"yearly"`. |
| `daysOfWeek` | int[] | **Weekly only.** `0`=Mon … `6`=Sun. |
| `dayOfMonth` | int | **Monthly only.** `1`–`31` (clamps to month length). |
| `month`, `day` | int | **Yearly only** (`month` 1–12, `day` 1–31). |
| `times` | string[] | 24-hour `"HH:mm"`. Omit for an "anytime" quest (no reminder). |
| `owner` | string | `"character"` (default) · `"pet"` — whose stats the quest boosts. |
| `stats` | string[] | Stats boosted. For `owner: "character"`: `vitality`, `fitness`, `wealth`, `social`, `grooming`. For `owner: "pet"`: `cleanliness`, `health`, `wellness`. |
| `xpValue` | int | XP per completion (default `10`). XP always accrues to your character. |
| `notesFormat` | string | `"markdown"` (default) · `"html"` — tells the app whether to load `description.md` or `description.html`. |
| `author` | string | Your handle / credit. |

```json
{
  "id": "morning-run",
  "name": "Morning run",
  "category": "Fitness",
  "tags": ["cardio", "outdoors"],
  "rating": 4.7,
  "recurrence": "daily",
  "times": ["07:00"],
  "stats": ["fitness", "vitality"],
  "colorHex": "#E8743B",
  "icon": "figure.run",
  "xpValue": 15,
  "notesFormat": "markdown",
  "author": "your-handle"
}
```

### Recurrence cheat-sheet

| `recurrence` | Extra fields | Meaning |
|---|---|---|
| `"daily"` | — | Every day. |
| `"weekly"` | `daysOfWeek` | Those weekdays each week. |
| `"monthly"` | `dayOfMonth` | That day each month. |
| `"yearly"` | `month`, `day` | Once a year on that date. |

## `quests/<id>/description.md` (or `.html`)

The text shown on the quest's preview. Markdown by default; use `description.html` (and set
`"notesFormat": "html"` in `quest.json`) if you need embedded images. 1–3 sentences is plenty.

## `bundles.json` — themed packs (optional)

A single JSON **array** of bundles. A bundle is a curated set of *existing* quests (e.g. "Sportsman",
"Balanced Person") that the app shows as a pack with one **"Add all quests"** button. Bundles only
**reference** quests by `id` — they never duplicate quest data.

| Field | Type | Required | Notes |
|---|---|:---:|---|
| `id` | string | ✅ | Unique, kebab-case, stable. |
| `name` | string | ✅ | Display name, e.g. "Sportsman". |
| `description` | string | | One-line pitch shown on the card. |
| `icon` | string | | An SF Symbol name. |
| `colorHex` | string | | `"#RRGGBB"`. |
| `questIds` | string[] | ✅ | The quests in the pack. **Each must be an existing `id` in `catalog.json`.** |
| `owner` | string | | `"character"` (default) · `"pet"` — a `"pet"` bundle only appears when the pet is enabled. |

```json
{ "id": "sportsman", "name": "Sportsman",
  "description": "Build strength and endurance with a daily athletic routine.",
  "icon": "figure.run", "colorHex": "#E8743B",
  "questIds": ["morning-run", "strength-workout", "stretch", "walk-8k", "cold-shower"] }
```

---

## Add a quest (pull request)

1. **Fork** the repo and create a branch.
2. Create the folder **`quests/<your-id>/`** with:
   - **`quest.json`** — the full parameters (table above).
   - **`description.md`** (or `description.html`) — the description text.
3. Add a matching summary object to **`catalog.json`** (the index). The `id` must match the folder.
4. Open a **pull request**.

See [CONTRIBUTING.md](CONTRIBUTING.md) for validation commands.

## Guidelines

- **`id` is unique, kebab-case, and stable** — it's the folder name, the index key, and how the app
  de-dupes. Renaming it later orphans anyone who added it.
- **`icon`** must be a real SF Symbol, or omit it.
- **`owner`** is `"character"` (default) or `"pet"`. Set `"pet"` for cat-care quests.
- **`stats`** must match the owner: character quests use `vitality`/`fitness`/`wealth`/`social`/`grooming`; pet quests use `cleanliness`/`health`/`wellness`. Stats outside the owner's set are ignored.
- **`times`** are 24-hour `"HH:mm"`; omit for an anytime quest.
- **`rating`** is honest and editorial — don't ship everything at 5.0.
- Keep colors readable (`#RRGGBB`).
- **Bundles** (`bundles.json`) only reference quests — every `questIds` entry must be an existing `id`
  in `catalog.json`, or it's silently skipped when the pack is added.
