# Contributing a quest

Thanks for adding to the Golden Grace quest market! Quests are added by pull request. Each quest is
a folder; the root `catalog.json` is a light index. See [README.md](README.md) for the full format.

## Steps

1. **Fork** the repo and create a branch.
2. Create **`quests/<your-id>/`** (kebab-case `id` that isn't already taken) containing:
   - **`quest.json`** — the quest's parameters (recurrence, times, stats, color, icon, `notesFormat`, …).
   - **`description.md`** — the description (or `description.html` if you set `"notesFormat": "html"`).
3. Add a summary object to **`catalog.json`** (the index): `id`, `name`, `category`, `tags`, `rating`,
   `icon`, `colorHex`. The `id` **must match the folder name**.
4. Open a **pull request**. The template will ask you to confirm a few checks.

## Guidelines

- **One quest per folder**, focused and reusable ("Morning run", not "Get fit").
- **`id` must be unique and stable** — it's the folder name, the index key, and how the app de-dupes.
- **`icon`** must be a real [SF Symbol](https://developer.apple.com/sf-symbols/), or omit it.
- **`owner`** is `"character"` (default) or `"pet"`. Use `"pet"` for cat-care quests (brushing, vet, …).
- **`stats`** must match the owner: character → `vitality`, `fitness`, `wealth`, `social`, `grooming`; pet → `cleanliness`, `health`, `wellness`.
- **`times`** are 24-hour `"HH:mm"`. Omit them for an "anytime" quest (won't send reminders).
- **`rating`** is editorial (0–5). Be honest; new entries can start around 4.0.
- **description** is 1–3 sentences. Markdown preferred; HTML only when you need images.
- Keep colors readable (`#RRGGBB`).

## Validate before you PR

```bash
# every catalog + quest.json is valid JSON?
python3 -m json.tool catalog.json > /dev/null && echo "index OK"
for f in quests/*/quest.json; do python3 -m json.tool "$f" > /dev/null || echo "BAD: $f"; done; echo "quests checked"

# index ids unique, and each has a matching folder with quest.json + a description?
python3 - <<'PY'
import json, os, glob
idx = json.load(open("catalog.json"))
ids = [q["id"] for q in idx]
print("duplicate index ids:", [i for i in set(ids) if ids.count(i) > 1] or "none")
for q in idx:
    i = q["id"]
    if not os.path.isfile(f"quests/{i}/quest.json"):
        print(f"MISSING quest.json for: {i}")
    if not (glob.glob(f"quests/{i}/description.md") or glob.glob(f"quests/{i}/description.html")):
        print(f"MISSING description for: {i}")
print("checked", len(idx), "entries")
PY
```
