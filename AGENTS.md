# AGENTS.md

## What this repo is

No build, test, lint, typecheck, or git hooks. Files:

- `DaD.md` — full OCR'd text of *"Dragons at Dawn: Supplement I — Twilight"*
  (Daniel Hugh Boggs, 2011), ~7,555 lines. Source of truth **for map and
  settlement generation only** (see "Rule sources" below).
- `DaD_Hexcrawl.html` — a single-file, dependency-free generator (open directly
  in a browser). Four tabs: "Броски по таблице" (single-hex roll log),
  "Генератор карты" (map), "Генератор встреч" (wilderness encounters),
  "Путешествие" (direction / getting lost).
- `index.html` — redirect to `DaD_Hexcrawl.html`. The book references both the
  redirect and the HTML file name directly; keep both paths working.
- `progress.md` — status log; record durable state there.

## Rule sources — two, by purpose

- **Map and settlements** ("Броски по таблице", "Генератор карты"): ZED/DaD.
  `DaD.md` is authoritative; never invent biome or settlement rules beyond it.
- **Travel, encounters and treasure** ("Генератор встреч"): the book
  *«Поиски Славы и Удачи» 2.0* — Appendix II (inhabitants and encounter tables,
  pp. 38–63), Appendix IV (treasure, pp. 118–124), "Исследование Дикой
  Местности" (pp. 35–36), threat table (p. 27). The source ODT lives **outside**
  this repo, in the book working folder
  `Документы/DnD_Materials/Рулбуки/seekingfamenfortune/Поиски Славы и Удачи_2.0.odt`;
  do not commit the book or its exports here. The migration plan
  «Гекскроул — анализ перехода на правила 2.0.md» (same folder) holds the
  control examples and acceptance criteria.
- The "Путешествие" tab (random direction, getting lost) is still CoZ-based and
  was deliberately left unchanged by the 2.0 migration.
- Do not derive encounter rules from `DaD.md`, and do not derive map rules from
  the 2.0 book. The old "all rules from DaD.md" instruction is obsolete.

## Where the encounter rules live in the HTML

Hardcoded JS data tables (names Russian, matching the 2.0 book):

- `ENC2_COLS` / `ENC2_TABLES` — 11 terrain columns × sums 2–12 (121 cells,
  weights 1,2,3,4,5,6,5,4,3,2,1 of 36). Cell kinds: `one`, `pair` (d6 1–3/4–6),
  `d6` (stated ranges), `cond` (referee choice), `time` (day/night), `sub`
  (conditional replacement on d6 4–6).
- `BESTIARY2` — per-article cards: wounds, attacks, morale, habits, number
  rules (`num`), lair text, treasure basis (`tr`). `ENC2_ALIAS` maps cell names
  to articles/variants. `THREAT2` — general threat table (p. 27) fallback.
- `HUMAN2`, `RETINUE2_CALLING`, `RETINUE2_SCALE` — human parties (pp. 57–59).
- `DRAGON2_AGES` and friends — dragon generation (pp. 39–40); `genDragon2` is
  the single calculation shared by table hits and the manual "Создать дракона"
  entry.
- `TRAVEL2` / `PASS2` — daily movement and encounter thresholds (pp. 35–36).
- Treasure (Appendix IV): `CACHE2` (6×6 cache table), `GEM2_TYPES`,
  `THING2_TABLE` / `THING2_BASE` / `BARREL2`, `MAGIC2_CATS`, `INCOME2`,
  `BUYERS2`.

Flow: `enc2Run` → `resolveCell2` → `inhabitantInfo2` → `enc2Finalize`
(`numPrefill2`) → `enc2Continue` (`leaders2`, treasure basis, `ledger2`).
Pure calculation functions (`parseCache2`, `cacheCoins2`, `rollCache2`,
`rollGem2`, `rollThing2`, `rollDragonHoard2`, `skolaHoard2`, `humanMoney2`,
`humanMagic2`, `saleBronze2`, `ledger2`, `genDragon2`, `travelSpeed2`) are kept
separate from rendering so they can be tested with forced dice.

## Encounter gotchas (2.0)

- The merged cell "Побережье / Реки и озёра, 7" must stay identical in both
  columns; an empty river result is a bug.
- Single-face d6 ranges are written `[n, n]`, not `[n]`.
- Number ranges the book gives without dice are **referee inputs** — do not
  reintroduce `RANGE_DICE`-style automation. The threat table (p. 27) is the
  fallback only when an article is silent; the separate "2d6 sum → threat"
  table on p. 27 is a **dungeon** template, not wilderness.
- Dragons: one age roll (2d6) drives wounds, attacks, cone and hoard multiplier
  (×1/×3/×6). Sko'la has fixed stats and a fixed hoard; Gorynych's head count is
  a referee input; lun/lindwurm follow their article clauses. Never roll a
  random variety on top of a named creature. The lair visit state (returns /
  sleeps / awake) is rolled only on demand, not at preparation.
- Cache: the first d6 is the **column**, the second the **row**; a six counts
  as zero **for coins only**; silver and bronze multiply the product by 30;
  "2Б"/"2С" double their metal. "Ещё 2"/"Ещё 3" spawn independent child caches;
  a repeated "Ещё" in a child means a looted cache; at most three additional
  caches per original roll. Keep both letters of mixed cells and the child
  history visible. Zero coins do not cancel non-coin results.
- Ledger: whole bronze (1 gold = 100 silver = 10 000 bronze); appraisal of
  items is separate from coins; unpriced items show a count and a note. Sale
  (½ / ¾ / full) floors to whole bronze; changing the buyer or re-expanding
  results must not reroll the treasure.
- City and open ocean have no Appendix II column — the encounter is entered
  manually. For an ambiguous hex (canyon, open water, forest density) the
  referee picks the inhabitant column and the day's passability; the tab works
  without a map. A road removes movement slowdown only, not the encounter
  threshold.
- The old CoZ procedures (discovery/evasion percentages, surprise, presence
  40–60%, letter treasures A–I, personal pocket change, `LAIR_TYPES`,
  `RESTING`) were removed from the encounter tab by the migration plan, which
  defers them to separate consideration. Do not restore them without a new
  task.

## Map gotchas (ZED/DaD, unchanged)

- UI text is Russian; map creature/terrain labels stay in the book's English,
  encounter names are Russian (2.0 book).
- `DaD.md` lines ~310–1007 hold the map pipeline: STEP 1 physiographics (biome
  d6, terrain d10, elevation d12), STEP 2 population (settlement table d6).
  The STEP 3 encounter pipeline in `DaD.md` is no longer used by the tool.
- Non-rulebook presentation features (not in `DaD.md`, don't treat as rules):
  the syllable-based settlement-name generator (cities + castles, fantasy-
  English flavour), the manual river/road drawing tools, the hex numbering
  scheme (`%02d%02d` column+row), and the PNG/MD export buttons.
- Settlement ranges (e.g. "4–40 hamlets") are sub-rolled to concrete numbers
  via dice (4–40 → 4d10, 20–80 farmsteads → 2d4×10, 2–10 villages → 2d5,
  10–60 farmsteads → 1d6×10, city populations → 1dN×1000), mirroring the lair
  size convention. "Keep" is rendered as "Твердыня (Keep)" — a short
  tower/fortified structure housing town militia, armory and jail
  (`DaD.md:484`), distinct from "Замок (Castle)" — a castle with retainers.
- Terrain generation must NOT invent biome rules beyond the book. Arctic
  (biome 6) has no STEP 2 population rule in `DaD.md`, so it is left
  unpopulated. The Ocean variant of biome 1 has no separate sea table in the
  book — it reuses the Arid d10 terrain (Hills / Hills and Canyons / Open
  Country / Deep Canyon) with islands on a water roll of 1; non-island hexes
  are drawn as blue water.
- Map hexes show a defence icon with priority castle > keep > wall > settlement
  type (🏰/🏯/🧱 override 🏙️/🏘️/🏡/🛖/🌾). Elevated hexes are filled with the
  elevation-tier colour (hills/mountains/tall mountains/grand peaks) and their
  ▲ markers are drawn as outlines (stroke), not solid fills.
- Roads follow CoZ (p. 14): they run from settlements of 500+ people (and from
  trade/industry sites); rivers have a source and a mouth.

## No build / how to verify

No test suite; the JS is inline in `DaD_Hexcrawl.html`. Sanity-check by extracting the
`<script>` body and running `new Function(js)` under Node (optionally with a
stub DOM), e.g.:
`node -e 'const h=require("fs").readFileSync("DaD_Hexcrawl.html","utf8");new Function(h.match(/<script>([\s\S]*?)<\/script>/)[1]);'`

For logic checks, run the extracted script in a Node `vm` context with a stub
`document`/canvas, then override the global `roll(n)` with a queue of forced
faces; the pure treasure/dragon/travel functions can also be enumerated
exhaustively (the migration plan's averages: 104.514 bronze, 123.426 silver,
1.493 gold, 0.697 gems, 0.066 things, 0.033 magic per original cache roll with
one level of additional caches). Keep such test scripts out of the repo.

## OCR gotchas — grep/parse carefully

The `DaD.md` text was extracted from a PDF and breaks naive `grep`, regex, and
heading parsing:

- Curly/smart quotes and dashes are used throughout (‘ ’ ― ‖), not ASCII.
- Bare page-number lines (`1`…`98`) and `---` page separators are interleaved
  in the body — strip them before parsing.
- Logical headings are split across multiple lines/`###` markers, e.g.
  `### CHANCE ### OF ### ADVENTURE ### WHEN` followed by `### RESTING`.
- Heading levels are inconsistent (mixed `##` and `###`), not a clean tree.
- OCR misspellings: `Apendix` (Appendix IV), `PHYSIOGRAHICS`, `Effects 0f Lost
  Limbs` (zero instead of letter o).
- Tables are flattened to plain text (e.g. character-sheet fields, Appendix X
  combat-matrix steps).

## Licensing

Derivative works distributed for free must carry the "Unofficial unlicensed
product designed for use with Dragons at Dawn…" statement; commercial
derivatives require Southerwood Publishing approval (see `DaD.md` lines 23–40).
The HTML `<footer>` carries the required statement (also emitted in the MD
export) — preserve it on edits.
