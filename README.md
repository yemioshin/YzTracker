# Yz Daily Macro Tracker — Context & Maintenance

**File:** `tracker.html` — a single self-contained HTML file. No build step, no
dependencies, no network calls. Open in any browser. Data persists in that
device's `localStorage` under the key `yz_tracker_v1`.

This README exists so **any future chat can read it, understand the tool, and
produce an updated `tracker.html` without re-deriving everything**. Keep both
files together under git.

---

## Why this exists

Yz is doing a body recomposition (target ~80 kg at 10–14% body fat) and the
core failure mode in every previous attempt was **untracked calories** — snacks,
juice, alcohol, and large Nigerian-food portions. The deficit was never real or
sustained. This tracker is the feedback loop: log everything, see it against the
targets, glance at the week. The recurring 2–3 week drop-off is treated as a
**systems problem, not a willpower problem**.

Authoritative source of truth for all numbers: `/mnt/project/fitness_plan.md`.
If that plan changes, update the constants in `tracker.html` to match.

---

## The targets (hard-coded in `tracker.html`)

```
Calories : 2,250 kcal
Protein  : 147 g      (THE non-negotiable anchor — 1g per lb lean mass)
Fat      : 59 g       (minimum, not a ceiling)
Carbs    : ~283 g
Flex slot: ~350 kcal  (lives INSIDE the 2,250 — juice/snacks/alcohol)
```

The flex slot is not extra. It is a carve-out *within* the daily total to
capture the things that were previously invisible. Logging against it ≠ adding
on top.

Find these in the JS as `const TARGET = {...}` and `const FLEX = 350;`.

---

## What the tool does

- **Macro cards** — calories/protein/fat/carbs vs target, progress bar, "to go / over".
- **Flex slot bar** — separate dashed bar; mark any item as "flex" when adding.
- **Add food** — two ways:
  - *From your list:* pick from the built-in food database + quantity (grams, or
    units for things like eggs/bagels). Macros auto-calculated.
  - *Custom:* type name + portion + the four macro numbers manually.
- **Today's log** — every item, with remove button.
- **Honesty anchors** — 5 daily tick-boxes tied to known blind spots (alcohol
  units, jollof weighed, juice/snacks logged, protein hit, morning weigh-in).
- **Morning weigh-in** — one number per day; last 7 days shown as a strip.
- **This week** — 7-day calorie strip for a quick adherence glance.
- **Export / Import JSON** — back up or move data between devices.
- **Date nav** — ‹ › to log past/future days.

---

## Data model (`localStorage` key `yz_tracker_v1`)

```json
{
  "version": 1,
  "days": {
    "2026-06-08": {
      "items": [
        {"name":"Sweet potato","portion":"248g","cal":213,"pro":4,"fat":0.2,"carb":50,"flex":false}
      ],
      "anchors": [false,false,false,false,false],
      "weight": 89.25
    }
  }
}
```

Each day keyed by ISO date `YYYY-MM-DD`. `anchors` is a 5-bool array matching
`ANCHOR_DEFS` order. `weight` is kg or `null`.

---

## Pre-loaded data

On first run (empty storage) the tool **seeds today** with the full day Yz
logged on 2026-06-08 (first real tracking day), so the file opens showing a
complete real day rather than a blank slate:

1. Bagel (plain) — 1 snack
2. Rotisserie chicken — 100g snack
3. Sweet potato — 248g
4. Turkey mince — 225g cooked
5. Thai spicy prawns — 10 prawns
6. Grapes — bowl ~150g
7. Bagel (plain) — 2nd snack
8. Fanta Fruit Twist — 330ml can *(flex)*
9. Oats — 80g (final meal)
10. Semi-skimmed milk — 125ml
11. Whey protein — 1 scoop (20g protein)
12. Strawberries — 170g
13. Grapes — 95g
14. Dates — 30g
15. Honey — 15g

Day totals: ~2,333 kcal · 149g protein · 46g fat · 330g carbs · 95 kcal flex.
Note this day landed protein on target, but carb-heavy / fat-light (~12g under
the 59g fat minimum) — a pattern worth watching given the visceral-fat flag.

Once real data exists in storage, the seed never runs again. To wipe and
re-seed: clear `localStorage` for the file (or Import a fresh JSON).

---

## Food database

In `tracker.html`, `const FOODS = {...}`. Values are per 100g unless the entry
has `unit:'each'` / `'tbsp'` etc. They're reasonable estimates — **replace with
packaging actuals when available** (rotisserie chicken fat varies with skin;
Thai prawn sauce varies). To add a food, copy the pattern:

```js
"Cod fillet (cooked)": {per:100, cal:105, pro:23, fat:0.9, carb:0},
"Rice cake":           {unit:"each", cal:35, pro:0.7, fat:0.3, carb:7},
```

---

## How a future chat should update this file

1. **Read `README.md` first** (this file) for context, then `tracker.html`.
2. Make edits in `tracker.html` directly. Common changes:
   - Targets changed in the plan → edit `TARGET` / `FLEX`.
   - New foods → add to `FOODS`.
   - New anchor → edit `ANCHOR_DEFS` (and the seed `anchors` array length must
     match — it's currently 5 booleans).
3. Do **not** break the `localStorage` schema (`version`, `days`) or Yz loses
   saved data on the next open. If a breaking change is unavoidable, bump
   `version` and write a small migration in `load()`.
4. Hand back the updated `tracker.html`. Yz commits it to git and re-opens it.

> Note on persistence: project files on the assistant side are **read-only**, so
> the assistant cannot save back into the project. The real save loop is: Yz
> keeps `tracker.html` + `README.md` locally under git → uploads to the project
> as knowledge so chats can *read* them → assistant returns an updated file → Yz
> commits. Data itself (logged foods, weights) lives in the browser's
> localStorage, separate from the code.

---

## Roadmap (queued, not built)

- Day 7 / Phase review tool (auto-average kcal/protein → hold or adjust target)
- Recomp trajectory visualiser (12–18 mo path; doubles as a coaching teaching aid)
- Coach Nourish v2 (client intake → assessment → phased plan)

---
*Living document — update alongside `fitness_plan.md` as stats, habits, and goals evolve.*
