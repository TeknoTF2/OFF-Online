# OFF TTRPG — Build Specification

The consolidation document. Everything designed across the system, bestiary, economy, and UI work, organized as the handoff for building the actual client. Read this first; everything else is a component it points into.

## 1. The document stack

| Artifact | Role |
|---|---|
| **System Document** (yours) | The engine: stats, damage, gauge, elements, statuses, classes, pricing, Part 5 schema. Canonical rules. |
| **off-restat-worksheet.md** | The enemy conversion procedure and calibration examples. Methodology reference; superseded numerically by the bestiary. |
| **off-canon-bestiary.json** | Raw merged canon (2008 + remaster + remake), per-field source tags. The audit trail. |
| **off-campaign-bestiary.json** | **The enemy database.** 59 entries, schema v2: element, HP (units[] for compound fights), DEF/RES/LCK, gauge (phases where needed), status tiers, structured moves[]. Software-ready; loads directly into the GM console's Enemies panel. |
| **off-economy.md** | Items, equipment, orbs, income, prices, Zacharie stock, Bandit EV, rest-zone placement. The attrition spine: 7–8 fights per leg, one leg per session. |
| **off-ui-combat-mockup.html** | Player client: shell (zone bar / inventory column / play viewport), battle screen, full interaction contract demonstrated. |
| **off-ui-gm-console.html** | GM seat: Location/Enemies/Encounter/Items/Players panels, CREATE/PARTY column, enemy party strip, staging, pause. |
| **off-ui-shop-mockup.html** | Zacharie's shop, greyscale with yellow, Buy/Sell/Leave, bubble-as-description. Renders in the player viewport. |

The mockups are living specs: their behaviors are requirements, their code is disposable.

## 2. Engine addenda

Rules created during encounter and UI design that are now canon and must be treated as part of the System Document:

**Enemy AGI is deleted.** Enemy blocks carry `gauge_s` directly (and `gauge_phases` for form fights). Enemy accuracy is hand-set per move. Both per the worksheet.

**The enemy shared object pool.** The GM's creatures share one item pool per encounter, mirroring the party inventory rule exactly: any creature may spend its turn using a pool item. The pool is set in the Encounter builder and is **distinct from drops**, which are curated per enemy instance. **Ursa Shot steals from the pool** — stocking the pool is setting the fight's steal table. Cutpurse draws from the zone drop table, not the pool.

**Waves.** Encounters queue in waves; a wave spawns at launch, on the previous wave's death, or on manual GM trigger. Queued instances carry: GM/AI control flag, drop assignment, and optional per-instance overrides (template untouched — the modified copy exists only in that encounter).

**GM/AI control.** Each enemy instance is either GM-piloted (GM receives its action stack on gauge fill, mirroring the player interface) or AI-run (engine picks from the moveset per simple weights). The flag is per instance and can be flipped mid-fight.

**Staged rooms.** Locations persist a saved layout of placed pieces. Placement is silent. Any piece can be flagged hidden (invisible on player clients, marked on the GM's); toggling visibility is the GM's reveal action. Teleporting the party to a location loads its saved room.

**Formations.** The battle screen has fixed sprite slots: **six party slots** on the right in the Batter-and-rings arrangement (one forward anchor, the rest orbiting), and **eight enemy slots** on the left in two loose rows. Party slot assignment is **randomized per encounter** — who stands where is fresh every fight. Enemy slot assignment is **GM-set in the Encounter builder** per queued instance; waves inherit their assigned slots on spawn, and slot markers render on the GM's field only. Slots are presentation, not mechanics — position carries no rules weight.

**Reveal gating.** Player clients render an enemy as sprite, name, and its visible effects only — active statuses and stat changes, which are public because the party watched them land. Element, stats, HP, and tiers are hidden until **Wide Angle or an Eye reveals that instance**; a reveal is party-wide and lasts the encounter. The GM sees everything always. This is why Wide Angle costs 2 CP and why the Eye exists in the shop — identification is a purchase, and the UI must never give it away.

**Pause.** The GM can freeze time globally: every gauge on every client stops, a PAUSED state renders everywhere. For narration, phase transitions, and stepping away. No game state advances while paused; talking was always free — pause makes deliberation free too.

**Statuses added by the endgame** (enemy-only, riding existing status machinery): Thorns (10% max HP on acting, 2 turns), Famine (double-tick Poison), Impure (weak to a random element, 2 turns; one-instance and native-cancellation rules apply), Vilified (= Muted; Cob's Corrupted variant also locks items), Defamed (the crit gift: 3 turns of guaranteed charged crits, then all output produced reflects onto the whole party; curable by Focus).

**Remake status vocabulary map** (for reading canon sources): Lethargic=Blinded, Frail=DEF Down, Sturdy=DEF Up, Confused=Madness (GM ruling), Impure=as above.

## 3. Architecture

Server-authoritative real-time state; seven seats (six players + GM). The server owns gauges, resolution, and legality; clients render and request.

Core state objects: `party[6]` (stats, statuses with per-status escalation counters, stat changes with timers, element changes, gauge, held-crit flag), `enemyTeam` (instances from bestiary rows + overrides, control flags, shared pool), `partyInventory` + credits (shared), `rooms{location: pieces[]}`, `encounterQueue` (waves), `clock` (paused flag).

Resolution rules the server enforces, from the System Document verbatim: gauge holds at full; crit rolled at gauge-fill start and flagged visibly; **validate-target-before-initiate** (a dead target refuses input — never resolve-then-undo); a miss spends CP and resets the gauge; AoE rolls accuracy per living target; Defend expires on the defender's next gauge fill; status durations count the afflicted's own turns; one-instance and Up/Down cancellation on contact; Madness auto-resolves at fill; Taunted dies with its applier.

Latency note: the refused-click rule is why the server validates and the client renders greyed-out dead targets from server state — a click racing a death must land as a refusal, not a rollback. This is the one place netcode design touches game feel directly.

## 4. Data layer

`off-campaign-bestiary.json` is the enemy table: load it, render the Enemies panel from it, instantiate encounters from it. Its meta carries the party DPS/heal tables (1–20) used to derive every number — keep them beside the GM as tuning context. One field is deliberately unfinished: **per-move ATK/ESP splits are hand-tuned in software** against each entry's `dmg_per_action` target; the moves[] carry MP multipliers and accuracy, which is everything except the split.

Player classes come from the System Document's tables as-is. Economy tables from off-economy.md: item catalog (with the Joker at 35% overriding canon), per-zone prices and income, Zacharie stock lists, equipment tiers (gear share is inside the printed checkpoints — a character without current-tier gear is below curve), orb caps.

## 5. UI

Built and specified: the player shell (zone bar, shared-inventory column wired to the Objects action, battle viewport), the battle screen (gauge hold + pulse, crit as fire on the armed menu and red gauge in the strip, MISS as event, dead-target refusal, icon rows for statuses/stat changes in two visual vocabularies, competence drawer with affordability), the GM console (five panels, CREATE/PARTY column, enemy party strip with red gauges, enemy action stacks, pool, staging, pause), and the shop (greyscale + yellow, ribbons, bubble-as-description, price slip, half-price selling).

Visual language: flat zone-color field per zone (Z1 blue / Z2 ochre / Z3 green / Room red), monochrome outlined sprites, damask motifs, slanted black boxes, Jersey-10-class pixel display type, amber selection, red reserved for crits/death/GM-side, shop outside the palette entirely. No rules prose in play surfaces — data only.

Remaining screens, in build order of need: exploration mode in the player viewport (the staged-room renderer — players see placed visible pieces, click pickups); rest zone (full restore + shop entry + the session-boundary beat); Wide Angle / Eye reveal card (per-enemy: stats, element, tiers — GM-gated); character sheet and level-up; death and ending screens. All follow the established language; none need new design decisions.

Art is the honest open item: every sprite is a glyph placeholder. Canon sprites are copyrighted — the build needs original art in the monochrome style, which is the single largest non-code cost in the project.

## 5b. The overworld

A room is three layers: **backdrop + collision + staged pieces** — and the primary backdrop source is **procedural**: rooms are JSON (floor rects with a pattern, wall rects, prop placements) rendered by a ~20-prop drawing kit in the client (proof: off-room-renderer.html). OFF's visual language — flat two-tone fills, black outlines, orthogonal geometry, repeating floor patterns, a small prop vocabulary — makes this near-lossless. Procedural rooms mean collision derives automatically from walls and solid props, zone-palette reskinning is free, room data is tiny and console-editable, and the room art is original (deleting the copyright flag for the overworld layer). Image backdrops (screenshots or Tiled compositions) remain a supported fallback for complex setpieces the prop kit can't express — the engine treats backdrop:'procedural'|'image' per room. Canon maps were sized for one sprite; rooms are composed wide enough for six independently-moving players. Rooms are authored one leg ahead of the party, never in bulk.

**Movement:** grid-snapped to the backdrop's tile size, four directions, all six players moving freely and independently within a room. **Transitions are GM-authority**: door pieces ping the GM on contact; only the Location panel moves the party. **Interaction:** the core verb is *examine* — clicking an adjacent piece announces it to all screens, and the GM answers by voice. Three prop behaviors cover the rest: sign/note (displays staged text), switch (visible state, consequences by voice), keypad (staged code; attempts ping the GM privately, examines announce publicly). Block puzzles and anything bespoke are the GM moving pieces by hand — the GM is the event system. **Encounter triggers** are manual LAUNCH by default, with optional painted trigger zones for unauthored-feeling ambushes. **Sprites:** character and enemy sprites with walk animations are GM-provided assets; the room kit covers everything else. Glyph placeholders serve until art exists.

Copyright posture: for the private table, screenshots and tileset compositions are practically equivalent; the public-release art flag in §8 applies to both identically.

## 6. Build order

Extends the System Document's Part 5, now with the UI layer sequenced: (1) server gauge loop + four buttons + party state, (2) damage formula with elements and gauge-fill crits, (3) statuses keyed to target gauge fills, escalation counters, stat-change timers, (4) competences and enemies as data rows from the bestiary JSON, (5) objects, shared pools, drops, credits, (6) the GM seat: encounter queue, waves, control flags, instance overrides, pause, (7) staging: rooms as backdrop+collision+pieces, Tiled import, hidden pieces, location switching, exploration movement and examine, (8) shop and rest zones off the economy tables, (9) polish: announcements, floats, reveal gating, sound.

A playable vertical slice exists at the end of step 5 with a hotseat GM; steps 6–7 make it a campaign tool.

## 7. Playtest plan

Session one is Zone 1, leg one, exactly as statted. Watch, in priority order: the slow-seat experience (Epsilon and Omega players' felt pacing at 6–8 s gauges — the one thing no formula answered); real fight lengths against the 30–40 s target (recalibrates all HP pools if off); whether the Purifier feels obsolete next to Fortune Tickets (pre-registered fix: Ticket to 400); Epsilon's damage share against groups of 3–4; and the attrition model's two soft assumptions (45% in-combat healing share, 30% HP buffer tolerance) — one session hardens both into facts and recalibrates the economy for Zones 2–4 automatically, since everything derives from the same throughput model.

Later, dedicated watch items already flagged in the bestiary: Vela Shot with crit-hits-everyone as the burst outlier, Blinded's uneven bite on the Bandit, the Dedan post-40% race length, and — much later — whether the Defamed crit gift's table psychology plays as designed.

## 8. Open items

Original art (sprites, Zacharie, bosses) — largest cost, no design blocker. Netcode for the real-time gauge (see §3 latency note). Justus's canon kit is wiki-flagged incomplete; his four known mirror-competences stand, backfill freely. Per-move stat splits (§4). The three prose-only boss kits that were hand-built (Sugar, Judge, Hugo-excluded) rest on your supplied data — final authority is yours. And the remaining UI screens (§5), none of which block the vertical slice.

Everything else is sessions.
