# AGENTS.md

## What this repo is

No build, test, lint, typecheck, or git. Two files:

- `DaD.md` — the sole source of truth: full OCR'd text of
  *"Dragons at Dawn: Supplement I — Twilight"* (Daniel Hugh Boggs, 2011), an
  OSR/Arnesonian hexcrawl RPG ruleset (~7,555 lines).
- `DaD_Hexcrawl.html` — a single-file, dependency-free generator (open directly in a
  browser). Three tabs: "Броски по таблице" (single-hex roll log), "Генератор
  карты" (map) and "Генератор встреч" (wilderness encounters).

All generator rules must derive from `DaD.md`, never be invented.

## Where the hexcrawl rules live

`DaD.md` lines ~310–1007 hold the whole pipeline: STEP 1 physiographics
(biome d6, terrain d10, elevation d12), STEP 2 population (settlement table
d6), STEP 3 encounters — creature-type table (d8 × 8 terrains), the six
creature sub-tables, lair table (d8), present-in-lair 40–60% (d6), group
splitting, discovery/evasion table, resting and surprise rules. Appendix IV
sample monsters (~line 6533) has `% in Lair` only for those samples, not for
the STEP 3 creatures.

## DaD_Hexcrawl.html gotchas

- UI text is Russian; creature/terrain names and table labels stay in the
  book's English.
- Rules are hardcoded as JS data tables (`CREATURE_TYPE`, `CREATURE_TABLES`,
  `LAIR_TYPES`, `RESTING`, `DISCOVERY`). `DaD.md` is authoritative — if a table
  drifts from the book, fix the code.
- Deliberate book gaps handled in code: the d8 table yields type `Dragon` (no
  dragon sub-table exists → output "referee decides") and `Human` (maps to the
  `Humans` table). `% in lair`, number appearing, and treasure are NOT in the
  book for STEP 3 creatures, so the tool leaves them as referee inputs rather
  than inventing values. The "random direction chart" is also absent from the
  book.
- Retainers (`RETAINER_LEADER`/`RETAINER_CLASS`/`RETAINER_COUNTS`): CoZ pp.158–159
  gives no fixed Number Appearing — the count comes from the leader (Lord 100–500 /
  Superhero 1–100 / Hero 1–10 Men at Arms). `startEncounter` rolls leader + class and
  pre-fills the number; fantastic creatures and companions are shown as a
  referee note (not rolled). The CoZ source text is external to this repo
  (see the `ZED/` working folder); do not invent values.
- Humans (Bandits/Rebels/Angry Mob/Nomads/Retainers) have no `% in lair` in CoZ.
  When the book value is `null` and the field is left blank, `continueEncounter`
  treats the lair as undetermined (`found = false`) and skips the lair/treasure
  steps instead of blocking on manual input; a manually entered value is still
  honoured. Creatures with no lair at all (`Elemental*`, `Tarn*`, `Unicorn`) get
  the same treatment.
- The HTML carries the required "Unofficial unlicensed product…" statement in
  its `<footer>` — preserve it on edits (it is also emitted in the MD export).
- Non-rulebook presentation features (not in `DaD.md`, don't treat as rules):
  the syllable-based settlement-name generator (cities + castles, fantasy-
  English flavour), the manual river/road drawing tools, the hex numbering
  scheme (`%02d%02d` column+row), and the PNG/MD export buttons.
- Settlement ranges (e.g. "4–40 hamlets") are sub-rolled to concrete numbers
  via dice (4–40 → 4d10, 20–80 farmsteads → 2d4×10, 2–10 villages → 2d5,
  10–60 farmsteads → 1d6×10, city populations → 1dN×1000), mirroring the lair
  size convention. "Keep" is rendered as "Твердыня (Keep)" — a short
  tower/fortified structure housing town militia, armory and jail
  (`DaD.md:484`), distinct from "Замок (Castle)" (castle with retainers).
- Terrain generation must NOT invent biome rules beyond the book. Arctic
  (biome 6) has no STEP 2 population rule in `DaD.md`, so it is left
  unpopulated. The Ocean variant of biome 1 has no separate sea table in the
  book — it reuses the Arid d10 terrain (Hills / Hills and Canyons / Open
  Country / Deep Canyon) with islands on a water roll of 1; non-island hexes
  are drawn as blue water.
- The forest discovery/evasion modifier in the book is a range ("10% to 25%",
  `DaD.md:806`) — the code rolls a random value in 10–25 rather than a fixed
  midpoint.
- Map hexes show a defence icon with priority castle > keep > wall > settlement
  type (🏰/🏯/🧱 override 🏙️/🏘️/🏡/🛖/🌾). Elevated hexes are filled with the
  elevation-tier colour (hills/mountains/tall mountains/grand peaks) and their
  ▲ markers are drawn as outlines (stroke), not solid fills.

## No build / how to verify

No test suite; the JS is inline in `DaD_Hexcrawl.html`. Sanity-check by extracting the
`<script>` body and running `new Function(js)` under Node (optionally with a
stub DOM), e.g.:
`node -e 'const h=require("fs").readFileSync("DaD_Hexcrawl.html","utf8");new Function(h.match(/<script>([\s\S]*?)<\/script>/)[1]);'`

## OCR gotchas — grep/parse carefully

The `DaD.md` text was extracted from a PDF and breaks naive `grep`, regex, and
heading parsing:

- Curly/smart quotes and dashes are used throughout (`‘ ’ ― ‖`), not ASCII.
- Bare page-number lines (`1`…`98`) and `---` page separators are interleaved
  in the body — strip them before parsing.
- Logical headings are split across multiple lines/`###` markers, e.g.
  `### CHANCE ### OF ### ADVENTURE ### WHEN` followed by `### RESTING`.
- Heading levels are inconsistent (mixed `##` and `###`), not a clean tree.
- OCR misspellings: `Apendix` (Appendix IV), `PHYSIOGRAHICS`, `Effects 0f Lost
  Limbs` (zero for the letter "o").
- Tables are flattened to plain text (e.g. character-sheet fields, Appendix X
  combat-matrix steps).

## Licensing

Derivative works distributed for free must carry the "Unofficial unlicensed
product designed for use with Dragons at Dawn…" statement; commercial
derivatives require Southerwood Publishing approval (see `DaD.md` lines 23–40).
