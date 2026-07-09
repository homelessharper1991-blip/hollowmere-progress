# Hollowmere — Master Inventory & Roadmap

## Changelog (append-only; newest first)

- **2026-07-09** — **OWNER DECISION: spatial index = uniform grid cells as indexed DB columns** (kd-tree benched with re-entry condition). §1.11(b) recommendation confirmed; DESIGN.md #8 updated.
- **2026-07-09** — Server core landed content embedding (`Content.g.cs` exists; module builds/publishes locally per workstream A).
- **2026-07-09** — Full regeneration under all six owner directives (simulation doctrine, no timelines, procedural zones, world-never-sleeps, player economy, rules bijection). Maintenance process: `tools/roadmap/check_status.py` flags stale tags from file-system evidence; updates are tag flips + changelog lines at milestones; full regeneration only on directive-level changes.

**Status:** living document, regenerated 2026-07-09 under the **Simulation Doctrine** (DESIGN.md non-negotiables #6, #7, #8). Companion to `DESIGN.md` (the binding contract) and `LEGAL.md`. Replaces the previous balance-driven roadmap in full.
**Scope:** everything a browser-based, exact-D&D-5e-SRD **simulation** MMO needs, from the current vertical slice ("The Drowned Vigil", 3 zones, 4 classes, level cap 3) to a persistent world of living settlements.

**Doctrine (governs every line below):** this is an **exact D&D simulation first, a game second**. Rules fidelity wins over balance even when the result is degenerate in play. Every mechanic mimics its SRD rule. Where the tabletop calls for DM judgment, a **rulings engine** — authored data plus deterministic resolvers — supplies what a DM would rule; nothing is left undefined. Settlements are the heart of the product: **every NPC is a full statted character** obeying all the rules — stat blocks, action economy, D&D time units, schedules, skill-checked daily activities, real death and consequences. **The world never sleeps:** every NPC always has a current activity and a scheduled next event, players present or not (SpacetimeDB scheduled reducers run with zero clients connected — the enabling property). The v1 slice ships as-is; the doctrine governs everything after it.

**The rules bijection (non-negotiable #7, highest importance):** every single SRD rule is implemented as a game mechanic, and every single game mechanic derives from an SRD rule. Unimplemented rule = defect; mechanic without rule provenance = defect. Enforcement artifact: **`docs/RULES_LEDGER.md` + `data/rules_ledger.json`** — the traceability matrix SRD section ⇆ implementing mechanic/code (being seeded now by the SRD-ingestion workstream, G below). Every feature change extends the ledger; validators flag unmapped mechanics. Items in this roadmap marked **[NO-PROVENANCE]** require owner sign-off or grounding before build.

**Locked decisions (do not relitigate):** browser client (Godot 4.6 web export, Compatibility renderer, no-threads build); 2D isometric pre-rendered look; SpacetimeDB C# module is the sole authority; real-time 6-second rounds — now also a **fidelity** argument: the round is the SRD's own unit of combat time; 5e SRD CC-BY only (no WotC Product Identity — no beholders/mind flayers/named settings; SRD subclasses only); no designed-in on-screen player cap.

**Tags:** `[DONE]` exists and works in repo · `[WIP]` in the current slice per DESIGN.md / actively being built · `[V2]` next dependency ring after the slice · `[V3]` depends on V2 layers. **V2/V3 are dependency order only — never calendar time.**
**Effort tags are parallelization granularity, never duration:** `S` = one agent, one file/sitting · `M` = one agent, a few files · `L` = decomposes into 2–4 parallel agent tasks · `XL` = its own workstream with an internal file-ownership partition. There are no wall-clock estimates anywhere in this document, deliberately: the plan is a dependency graph to blitz with parallel agents, not a schedule.

**Doctrine reversal ledger** (prior MMO-balance house rules, now flagged deviations with their replacements — each re-appears in its section):

| Prior house rule | Doctrine ruling | Where |
|---|---|---|
| CC diminishing returns | **Removed.** Chain CC is legal. The SRD's own answer is Legendary Resistance on legendary stat blocks, save-ends conditions, and Counterspell/Dispel in the world | §5 |
| No friendly fire on AoE | **Removed.** Fireball hits allies and bystanders per SRD templates; consequences flow through the crime system, not code prevention | §5, §1.7 |
| Threat/aggro tables | **Replaced** by rules-faithful creature decision-making (INT/WIS-tiered targeting, stat-block traits, codified morale) | §9 |
| Simplified death (downed 15 s → respawn) | **Replaced** by RAW death saves + stabilization; persistent-world answer = SRD resurrection magic with costly consumed components; **permadeath-if-unraised flagged for owner decision** | §4, §1.6, (b) |
| Reactions as stances only | **Replaced** by a real reaction economy: opportunity attacks, readied actions, Shield/Counterspell as player-configured standing policies (1 reaction/round enforced) | §5 |
| Milestone-only leveling | **Replaced** by RAW XP from CR as primary progression; authored XP awards for non-combat challenges via the rulings engine | §10 |
| Continuous per-entity 6 s cooldowns | **Initiative order within the round** adopted after evaluation (§5) — turns get anchors inside the 6 s round | §5 |
| Bind-on-pickup/equip, durability repair fees | **Removed** — items are freely tradable property per SRD; the economy's sinks are lifestyle expenses and resurrection components; RMT handled by moderation | §8 |
| Rest only at inns/campfires | **Removed** — RAW resting anywhere, with RAW interruption, the 1-long-rest-per-24 h limit, and wilderness encounter checks doing the pacing | §4 |
| PvP flags / consensual-only duels | **Doctrine-derived open simulation:** a player character is a creature; attacking one is an attack, adjudicated by the same rules and the same law/crime system. **Flagged for owner sign-off** (griefing-by-rules risk, (b)) | §6, §1.7 |

---

## 0. Browser Runtime Constraints (first-class, read before building anything)

The browser is not a port target; it is *the* platform. Every domain below carries inline *Browser:* notes where the platform bends a design. The simulation doctrine changes nothing here — all simulation is server-side; the browser only ever renders the observed slice of it. The new pressure point is **settlement rendering**: a living town puts hundreds of citizens on screen, so crowd LOD stops being polish and becomes doctrine-critical.

- [WIP][M] **Initial payload budget < ~50 MB** — engine wasm (~40 MB raw, ~15–17 MB brotli), main `.pck` with UI + village assets, first-zone atlases. CI check that fails the build if compressed initial payload exceeds budget; report per-asset sizes in `tools/deploy/package_client.ps1`.
- [V2][L] **Asset streaming for growth** — zone atlases/music fetched on demand (HTTP `ResourceLoader`/`HTTPRequest` into `user://` cache) rather than shipped in the main pck; prefetch adjacent zones while idling. Procgen (§7) makes zone count parametric — streaming is what keeps first paint constant while the world grows.
- [DONE][—] **Compatibility renderer / WebGL2 ceiling (decision)** — no compute shaders, no SSBOs; 4096² safe max texture size (atlas paging accordingly); canvas-item shaders only; MultiMesh2D works and is the crowd-LOD workhorse. All VFX must be sprite/CanvasItem-shader based.
- [DONE][—] **No-threads build (decision)** — plain static hosting, no COOP/COEP headers; cost: everything (render, JSON drain parse, audio trigger) shares one ~16 ms frame budget. Corollaries: chunk all JSON parsing, never parse a full zone in one frame, cap per-frame `drain()` event processing with carry-over queue. *Doctrine note:* citizen-heavy zones raise row churn — the drain cap plus event-granularity position updates for far citizens (§1.11) is the answer.
- [WIP][M] **Tab-throttling tolerance** — browsers clamp rAF/timers to ≥1 Hz when unfocused; the server keeps simulating (you can be attacked — and under RAW death rules, *die* — in a background tab). Client must: (a) keep the WebSocket alive via the JS bridge, (b) on refocus, drain the backlog in bounded chunks or force a zone resubscribe past a threshold, (c) show a "while you were away" summary. Gameplay policy for background death saves is part of the death-RAW owner decision (risk (b)#3).
- [V2][M] **wasm memory ceiling & eviction** — practical ceiling ~2 GB desktop, ~1 GB iOS Safari; on zone transition, free previous zone atlases/audio buffers; memory HUD in debug builds; bestiary/portrait atlases load per-zone, never globally.
- [DONE][—] **WebSocket-only networking (decision, and a strength)** — no UDP/WebRTC; TCP head-of-line spikes are invisible when actions resolve on 250 ms ticks and rounds last 6 s. The initiative-anchored round (§5) is *more* latency-tolerant, not less: intents queue, resolution happens on the server's clock. Say no to any future mechanic requiring <150 ms reactions — standing reaction policies (§5) exist precisely so no human has to click inside a reaction window.
- [WIP][S] **Persistence: localStorage/IndexedDB** — auth token in `localStorage` (bridge), settings/intro-seen flag in Godot `user://` (IndexedDB-backed; always `flush()` before navigation-away paths). Chat history session-only; never persist large blobs client-side.
- [V2][M] **iOS Safari quirks** — audio requires a user gesture (route through the "ENTER THE MERE" click); memory-pressure tab reloads must resume cleanly (token + charId restore → straight back into world); test WebGL context-loss recovery.
- [DONE][—] **Zero-install patching (advantage)** — static files behind a CDN; hash-suffixed pck/wasm, cache-busted `index.html`; client/module version pairing per §11 binding compatibility.
- [V2][M] **Settlement crowd budget (new, doctrine-driven)** — a market square at noon may hold 100–300 citizens. Client rule: full `EntityView` only inside the observation radius (~20 tiles), MultiMesh static-frame beyond it, nameplate/HP bars culled past ~12 tiles; subscription already zone-scoped, and far-citizen positions update at event granularity (a handful of row updates per citizen per game-day, §1.11), so network and render load are both bounded by the observed radius, not by population.

---

## 1. Settlement & World Simulation (first-class, the heart of the product)

Everything in this section is new architecture. File home: `server/sim/` (new directory, `partial class Module` spread across files — zero contention with the existing slice files); data home: `data/rulings/`, `data/srd/`, and procgen-emitted `data/zones/<zone>.population.json`.

### 1.1 NPC-as-character architecture

- [V2][XL] **`citizen` table — every NPC is a full statted character.** Replaces the slice's decorative `npc` table (which stays for v1 and migrates). Columns: identity fields (citizen_id pk, name, species, age, portrait), **full SRD stat block reference** (`stat_block_id` into compiled SRD data — commoner/guard/noble/priest/acolyte/cultist/thug/scout/spy/veteran/knight/mage etc. from SRD Appendix "Nonplayer Characters" — plus per-citizen variation deltas: ability-score adjustments, skill proficiencies, trained tools), current HP/max HP/hit dice, conditions via the existing `condition_row` (holder_kind "citizen"), inventory via `inventory_item` (holder generalized), equipped weapon/armor, gold, spell slots for caster citizens, zone/x/y/dir, `current_activity`, `activity_started_at`, `next_event_at`, home/household id, occupation id, lifestyle tier, faction, attitude defaults. Every rule that applies to a player applies to a citizen: action economy in encounters, skill checks for daily work, exhaustion, death saves, encumbrance.
- [V2][M] **Stat-block compiler** — `tools/codegen` ingests the SRD NPC and monster stat blocks (extending `make_srd_json.py` → `data/srd/stat_blocks.json` → `server/GameData.cs` growth); variation is data (procgen emits deltas, §1.10), base blocks are compiled-in constants.
- [V2][M] **Unified creature interface** — `server/sim/Creature.cs`: one server-side accessor layer over player/citizen/mob rows (GetAC, GetSave, GetSkill, RollAttack, TakeDamage, GetSpeed, conditions) so combat (§5), AI (§9), crime response (§1.7), and the rulings engine resolve against *any* creature identically. This is the single most load-bearing refactor of the doctrine: build it before the settlement layers, and port the slice's player/mob combat onto it.
- [V2][S] **Action economy off the battlefield** — outside encounters, citizen activity is minute/hour-granular (§1.2); the moment a citizen enters an encounter (attacked, witnesses violence, guard response) it drops into the 6-second round loop with a real initiative roll like everyone else, then returns to schedule.

### 1.2 World clock & time-granularity tiers

- [V2][M] **World clock singleton** — `world_clock` table: epoch game-time (in game-seconds), configurable ratio (default **30:1 — 48 real minutes = 1 game day**; ratio is data, not code), calendar date, season, sunrise/sunset per season. All simulation timestamps are game-time; conversion helpers in `server/sim/WorldClock.cs`. *Fidelity dividend of 30:1:* a short rest (1 game hour) = 2 real minutes; a long rest (8 game hours) = 16 real minutes; a torch (1 game hour) = 2 real minutes; ritual casting (+10 game minutes) = +20 real seconds. RAW durations become playable MMO durations without house-ruling a single number. **Owner flag:** ratio value itself (30:1 default) is the one knob; everything downstream is RAW.
- [V2][M] **Granularity tiers** — round (6 s, encounters only — the SRD's combat time unit, already the locked combat model) · minute (fine civilian actions: haggling, conversations, lockpicking) · hour (work shifts, rests, travel legs) · day (lifestyle upkeep, downtime activities, restocking). Every system in §1 declares its tier; the event queue (§1.11) makes tiers a scheduling choice, not separate engines.
- [V2][S] **Calendar & season table** — 12-month year as data (`data/rulings/calendar.json`); season drives daylight hours, weather tables (§1.8), crop/fishing yields (§1.4), festival events (§7).

### 1.3 Schedules, needs, lifestyle economics

- [V2][L] **Daily schedule system** — `schedule` data per citizen (emitted by procgen §1.10): ordered entries `{start_time, activity, location, params}` in D&D time (e.g., wake 06:00, breakfast, open shop 08:00, market run 12:00, close 18:00, tavern 19:00, sleep 22:00). `server/sim/Schedule.cs` advances citizens entry-to-entry via the event queue; each entry schedules its own next event. Deviations (crime witnessed, festival, sick, jailed) push interrupt activities that resume the schedule after.
- [V2][M] **Needs, RAW-anchored** — food/water (1 lb food, 1 gallon water per day; going without → CON-modifier days of grace then **exhaustion per RAW**), sleep (no long rest in 24 h → exhaustion level per the rulings engine's codification of the standard ruling), shelter. Needs are satisfied *through the economy*: citizens buy meals at SRD prices, sleep in their households. A siege that stops food carts starves a town by the book.
- [V2][M] **Lifestyle expenses (SRD table, verbatim)** — every citizen and every player pays lifestyle upkeep per game-day: wretched free, squalid 1 sp, poor 2 sp, modest 1 gp, comfortable 2 gp, wealthy 4 gp, aristocratic 10 gp+. Deducted daily by the event queue; failure drops the citizen a tier with knock-on effects (attitude, health via needs, schedule change). For players this is the doctrine's answer to gold faucets — the SRD's own sink, applied as written. Lifestyle tier is also procgen's wealth knob (§1.10).
- [V2][M] **Exhaustion (RAW, 6 levels)** — disadvantage on ability checks → speed halved → disadvantage on attacks/saves → HP max halved → speed 0 → death; applied by needs, forced march (§4 travel), and effects; recovery 1 level per long rest with food/drink. Same `condition_row` machinery for players and citizens.

### 1.4 Skill-checked professions affecting stock & prices

- [V2][L] **Profession work as skill checks** — each work shift resolves a real server-side check at its scheduled event: tool or skill proficiency vs a DC ladder from the SRD difficulty ladder (Easy 10 / Medium 15 / Hard 20 — codified in `data/rulings/dcs.json`). Result quantizes output: smith's day produces goods worth X gp of progress (SRD crafting rate: **5 gp of market value per day, materials at half cost** — applied verbatim), fisher's catch, farmer's yield (seasonal modifier), performer's earnings (SRD "Practicing a Profession" downtime rule governs income tier). Failed checks produce less; crits (rulings-engine entry) produce masterwork-tagged stock.
- [V2][M] **Shop stock & prices from output** — `business` table per shop: stock rows fed by owner/employee production and by trade-route deliveries (§8); prices anchor to SRD equipment price lists, modulated by scarcity (stock below par → price multiplier band, a rulings-engine table, not a floating market). Players buying out all the arrows genuinely empties the fletcher until she makes more.
- [V2][M] **Employment & wages** — SRD service prices as written: unskilled labor 2 sp/day, skilled 2 gp/day; citizens hold `job` rows binding them to businesses; wages flow shop→citizen→lifestyle expense→other shops. The town's money actually circulates; telemetry (§11) watches the loop.
- [V2][M] **Player-owned shops & P2P trading (owner directive; staged NOW in `features/trading/`)** — players rent real stalls/buildings from the settlement registry (SRD property/downtime "running a business" rules); stock and price their own goods; while the owner is offline a **statted NPC clerk hireling** (SRD skilled wages, 2 gp/day, drawn from the population registry) runs the shop on the world clock — rent + wages are recurring RAW sinks; unpaid clerk quits, arrears evict to escrow. Direct trading = escrowed two-party sessions with atomic commit (the #1 dupe surface — adversarially reviewed). *Browser:* shop browsing is paginated char-scoped queries, never whole-table subscriptions.
- [V3][M] **Trade routes between settlements** — caravan citizens with travel schedules (RAW travel pace, §4) moving goods between procgen settlements; interdiction (bandits, weather) is real and resolved by the same rules.

### 1.5 Social simulation — codified reaction & attitude rules

- [V2][M] **Attitude model** — `attitude` rows (citizen → character, citizen → faction): the classic three-state ladder **hostile / indifferent / friendly** codified with a numeric score behind it; starting attitude from faction + first impression (rulings-engine table); shifted by deeds (gifts, quests, crimes witnessed, public heroics) with authored magnitudes in `data/rulings/attitude_shifts.json`.
- [V2][M] **Social checks by the book** — Persuasion/Deception/Intimidation rolled server-side vs DCs set by attitude tier and request size (the DC matrix is `data/rulings/social_dcs.json` — the codified DM ruling); results shift attitude and gate dialogue branches; the existing dialogue `check` schema already carries this — extend conditions language with `attitude:<tier>`.
- [V2][S] **Memory & gossip** — witnessed events (crimes, gifts, rescues) write `memory` rows on witnessing citizens; a daily gossip event diffuses notable memories through a household/tavern adjacency graph (deterministic, seeded), so reputation spreads at the speed of talk, not omnisciently.
- [V2][S] **Barks & visible life** — observed citizens surface their current activity and attitude as floating text/dialogue flavor (data-driven bark tables per occupation/attitude); pure client presentation over sim truth.

### 1.6 Mortality, replacement, population registry

- [V2][M] **Citizens die for real** — 0 HP → unconscious → **RAW death saves** (same implementation as players, §4); stabilized citizens convalesce (SRD recuperating downtime); dead citizens are dead. Corpse rows, funeral events (temple schedule), estate transfer to household heirs (rulings-engine succession table).
- [V2][M] **Resurrection as the persistent-world answer** — temples with caster citizens of sufficient level cast Revivify (300 gp diamond, ≤1 game-minute), Raise Dead (500 gp diamond, ≤10 game-days) per the caster registry (§1.9 #9) — for citizens whose household/faction can pay, and for players who pay. Components are consumed; diamonds are a real economy sink (§8). Important-NPC insurance (a reeve's temple retainer) is authored data, not plot armor.
- [V2][M] **Population registry & replacement** — `population_registry` per settlement: births are out of scope for now (owner flag); replacement is migration — vacancies (dead smith) generate migration events drawing statted replacements from procgen's population pool with arrival travel time; succession rules fill offices (reeve dies → assembly elects per settlement law data). No respawning innkeepers, ever: the *role* refills, the person does not.
- [V3][M] **Aging & natural death** — age advances on the calendar; venerable citizens retire/die by actuarial table (rulings-engine data); keeps multi-season worlds from freezing demographically.

### 1.7 Crime & guard response (real action economy, real morale)

- [V2][L] **Crime detection by the rules** — theft = Sleight of Hand vs passive Perception of witnesses in the spatial index's radius (§1.11); assault/murder in view = automatic witness memory + alarm; darkness and obscurement genuinely matter (§1.8) — night burglary is mechanically easier, as it should be. Witness → report event → `crime` row (type, perpetrator if identified, witnesses, settlement).
- [V2][L] **Guard response with real action economy** — guards are citizens with guard stat blocks on patrol schedules; response is an interrupt activity: nearest guards (spatial index query) move at real speeds, shout (alerting others via audible radius), and on contact initiate an encounter under full combat rules — initiative, opportunity attacks, their own reactions. Guards use nonlethal intent (RAW melee knockout option) for arrestable offenses; **morale** is the codified DM ruling (§1.9 #2): a guard reduced below half HP or seeing an ally die makes a morale check and may retreat and escalate rather than fight to death. Escalation ladder: watchman → squad → captain (veteran block) → posted bounty.
- [V2][M] **Law, trial, punishment** — per-settlement `law` data (`data/rulings/law.<settlement>.json`): offense → fine/jail-days/exile/execution; jail = schedule override in the jail building (a real place; breakouts are possible by the rules); bounties create hunt events for guard/bounty-hunter citizens. Applies identically to player characters — this is the doctrine's griefing answer: **consequences, not prevention** (friendly-fire fireball in the market is mass assault; see risk (b)#5).
- [V2][S] **Alarm & lockdown states** — settlement alert level (calm/wary/alarm/lockdown) driven by recent crime rows; modifies schedules (shops shutter, patrols double) and starting attitudes.

### 1.8 Weather, season, light — darkness that matters

- [V2][M] **Weather system** — per-region `weather` singleton advanced by scheduled events off seasonal tables (`data/rulings/weather.json`): clear/rain/fog/storm/snow; mechanical effects as RAW where the SRD speaks (heavily obscured in fog banks, difficult terrain in deep snow, disadvantage on ranged in strong wind — the rulings engine codifies the standard adjudications) and schedule effects (farmers stay in, market thins).
- [V2][M] **Light & vision, RAW** — bright/dim/darkness per tile computed from sun state + light sources (torch: bright 20 ft/dim 20 ft, 1 game-hour duration and *consumed*; lamps, candles per SRD); dim light = lightly obscured (disadvantage on sight Perception), darkness = heavily obscured (effectively blinded); **darkvision by species/stat block honored**. Feeds stealth (§5), crime (§1.7), and monster behavior (§9). Lamplighter is a real job with a real schedule.
- [V2][S] **Client rendering of all of it** — CanvasModulate day/night tint from the world clock, per-source glow sprites, weather particle overlays with reduced-motion switch. *Browser:* tint is free; cap particles; light *math* is server-side only — the client never computes visibility.

### 1.9 The rulings engine — enumerated

The codified DM. Every entry = a data file in `data/rulings/` + a deterministic resolver in `server/sim/Rulings/`. Initial catalog (grows under change control — every addition is a PR touching this list; see risk (b)#4):

1. **Reaction decisions** — when an NPC uses Shield/Counterspell/opportunity attack/readied action: policy tables per stat block; players get standing policies (§5). [V2][M]
2. **Morale, flee, surrender** — the SRD has no morale rule; codify the classic optional rule: DC 10 Wisdom save on first drop below half HP, leader slain, or outnumbered 3:1 — failure = flee/surrender by creature disposition; undead/constructs/oath-bound exempt. [V2][M]
3. **Combat target selection** — INT-tiered decision tables (§9). [V2][M]
4. **Ad-hoc check DCs** — the SRD difficulty ladder (5/10/15/20/25/30) mapped to every codified activity: professions, climbing walls, haggling. [V2][S]
5. **Attitude shifts & social DCs** — §1.5 tables. [V2][M]
6. **Cover determination** — auto-derived half/three-quarters/full cover from tile geometry between attacker and target. [V2][M]
7. **Object AC/HP** — breaking doors/locks/chests: material AC (cloth 11/wood 15/stone 17/iron 19) + size HP table; locks pickable with thieves' tools vs authored DC. [V2][S]
8. **Travel encounters** — per-region encounter tables with frequency dice per travel leg and watch (§4 travel). [V2][M]
9. **Spellcasting services** — caster registry per settlement (which citizens can cast what, from their real slots); price = consumed component cost + tiered fee table; availability is real (the priest who died can't cast). [V2][M]
10. **Resurrection willingness** — who casts for whom: temple faction attitude + payment + deceased's standing (memory/gossip data). [V2][S]
11. **Non-combat XP awards** — traps evaded, quests completed, social "victories": authored CR-equivalent XP values in content data (§10). [V2][S]
12. **World pacing** — what happens when players ignore a quest: authored world-advances-anyway event scripts (the cult ritual completes on its calendar date). [V3][M]
13. **Stealth state machine** — when hiding is possible, when it breaks, when re-rolls happen (§5); codifies the game's most DM-adjudicated rule. [V2][M]
14. **Treasure placement** — hoard/individual treasure tables by CR band as data, used by procgen and mob loot defaults. [V2][M]
15. **Succession & vacancy** — §1.6 registry rules. [V2][S]
16. **Law & punishment** — §1.7 per-settlement tables. [V2][M]
17. **Merchant restocking & scarcity pricing** — §1.4 band tables. [V2][S]
18. **Readied-action trigger grammar** — small condition language ("enemy enters reach", "caster begins casting") shared by player readies and NPC policies. [V2][M]
19. **Improvised actions** — curated verb set (shove off ledge, throw sand, overturn table) with codified adjudications; deliberately last — the open-ended DM gap. [V3][L]
20. **Falling, suffocation, fire, starvation** — already RAW; implement as written, listed here for completeness of the hazard resolver. [V2][S]

Size estimate for the full judgment-gap catalog: **~40–60 entries** at maturity (each S/M as data + resolver); the 20 above are the load-bearing core. Tracked as `docs/RULINGS_CATALOG.md` with owner-visible additions, cross-referenced into the rules ledger.

### 1.10 Procedural settlement & population generation

- [WIP-external][XL] **`tools/procgen`** — Python, seeded, deterministic; **being built now by a parallel workflow** (wilderness, dungeon, settlement generator families) emitting the *existing* zone-JSON contract (DESIGN.md Zones section). This roadmap treats its contracts as fixed interfaces: zone JSON in, `<zone>.population.json` beside it.
- [V2][M] **`population.json` contract (consumer side)** — per citizen: SRD base stat block + variation deltas, household id, occupation + workplace building, daily schedule in D&D time, lifestyle tier, faction, starting attitude class, caster flag/spell list where applicable. `server/sim/PopulationLoader.cs` seeds `citizen`/`household`/`business`/`job`/`schedule` rows at settlement init; `tools/codegen/embed_content.py` grows to validate and embed it.
- [V2][M] **Constraint overlays** — authored campaign content (named NPCs, quest buildings, plot flags) rides as overlay JSON the generator honors: Reeve Maera is a *constraint* pinning one generated citizen's identity/stat block/schedule, not a separate code path.
- [V2][S] **Parametric content scaling** — settlement size/wealth/culture knobs + seed = a new living town; content volume scales with generator quality and seed count, not authoring man-hours. The v1 slice's 3 hand-authored zones ship as-is and are grandfathered (village gains a population.json retroactively as the first generator test case).

### 1.11 Cost architecture — THE WORLD NEVER SLEEPS made affordable (Directive spec)

**(a) Event-driven simulation — never per-tick polling.** Every citizen row carries `current_activity` + `next_event_at` (game-time). One scheduled **dispatcher reducer** (`SimDispatch`, ~1 s cadence like `ai_tick`) pulls due events via a B-tree index on `next_event_at` and resolves them in batches; each resolution writes the new activity and schedules the next event. (Per-NPC SpacetimeDB scheduled rows are the alternative; the dispatcher wins because it batches transactions and gives back-pressure control — evaluate both at the 10 k bot test, §11.) **The arithmetic:** a citizen's day is ~20 events (wake, meals, work start, 2–3 work checks, market, social, return, sleep, plus need ticks). At the reference scale of **10,000 NPCs: 10,000 × 20 = 200,000 events per game-day; at the 30:1 clock a game-day is 2,880 real seconds → ~70 events/second** — trivial for a database that runs full transactions per reducer call. (Even at a 1:1 clock it's ~2.3/s; at 100,000 NPCs and 30:1 it's ~700/s — still plausible, measure it.) Contrast per-tick polling: 10,000 NPCs × 4 checks/s = 40,000 checks/s — ~600× worse for zero fidelity gain. Combat drops entities into the 250 ms/6 s round machinery, whose cost scales with *concurrent encounters*, not population. SpacetimeDB scheduled reducers run with zero clients connected — the world genuinely never sleeps.

**(b) Spatial index over players and creatures** — serves aggro, observation-radius queries, AoE target gathering, guard response, witness detection. Honest evaluation of the two candidates:
- **kd-tree (owner suggestion):** excels at k-nearest-neighbor and skewed density over large sparse extents; but under continuous movement it needs periodic rebuild or tolerates imbalance, and as a module-memory structure it must be rebuilt on every module restart/hot-publish and kept coherent with table state across reducer calls by hand — a real correctness surface in SpacetimeDB, where the durable truth is tables.
- **Uniform grid buckets:** O(1) incremental update (an entity only moves buckets when crossing a cell boundary); radius query = scan of the overlapped cells; ideal at the roughly uniform densities of bounded, tile-based zones (1 tile = 5 ft; zones are already finite grids); and — decisive — it expresses *as the database itself*: quantized `cell_x`, `cell_y` columns (cell = 16×16 tiles) with a composite B-tree index `(zone, cell_x, cell_y)` on player/citizen/mob tables. The index then survives hot-publish and restart for free, every reducer can use it with no warm-up, and there is no shadow-state coherence problem.
- **Recommendation: uniform grid via indexed cell columns.** Adopt [V2][M] as `server/sim/Spatial.cs` (cell math + query helpers). Revisit a module-memory kd-tree (rebuilt in `Init`, updated per move) only if profiling shows nearest-neighbor queries ("closest guard to the scream") dominating — and even then, grid + outward ring scan usually suffices at settlement densities. Decision recorded here; measure both under the bot harness before any switch.

**(c) Observation selects FIDELITY, never ACTIVITY.** No NPC is ever frozen; all tiers advance on the same clock through the same schedules and rules with real server-side dice:
- **Tier 0 — observed** (player within observation radius, ~20 tiles): full fidelity — `move_path` walking, minute-granular micro-actions, barks, and encounters at the full 6 s round loop.
- **Tier 1 — same zone, unobserved:** event-granularity — position updates only at activity boundaries (teleport-between-waypoints in the data; players never see it), checks still rolled at their scheduled times; NPC-vs-NPC encounters run round math without presentation rows.
- **Tier 2 — unobserved zone:** **outcome-equivalent batch resolution** — due events resolve in batches with the same dice and same rules; NPC-vs-NPC combat fast-forwards the round loop inside one transaction. Outcomes are distribution-identical to observed play; only the *path texture* (exact tile trajectories) is coarser.
- **Transitions:** a player arriving mid-activity finds citizens at schedule-implied positions (deterministic placement from activity + elapsed time); promotion Tier 2→0 is instant because the sim state was always current.
- **[OWNER SIGN-OFF REQUIRED]:** the policy is outcome-equivalence, not path-equivalence — e.g., an unobserved guard-vs-thief chase resolves as opposed Athletics checks per round of the same chase rules rather than tile-by-tile pursuit. This is the doctrine-compliant reading of "no NPC ever frozen"; the alternative (full path simulation for all 10,000, always) multiplies cost for no rules-visible difference. Sign-off recorded here when given.

---

## 2. Client / Rendering (re-audited under the doctrine)

- [WIP][L] **Isometric TileMap zone rendering** — ground/props/overlay TileMapLayers from `client/data/zones/*.json`, 128×64 diamonds, Y-sorted props. No World scene exists in `client/src` yet — this is open work, not polish. *Browser:* build layers incrementally across frames; procgen zones may exceed 48×48 — chunking is mandatory, not optional.
- [WIP][L] **EntityView sprites** — AnimatedSprite2D from generated 8-direction sheets, idle/walk/attack/cast/death @10 fps, HP bar, nameplate, 4 Hz→60 fps interpolation. Extends to citizens (occupation-varied bodies from the sprite pipeline).
- [WIP][M] **Tile/prop art set** — Blender-rendered per DESIGN legend (`tools/blender` is empty — generation scripts are the open gap; `assets/icons` [DONE], `assets/keyart` [DONE], portraits/models/sfx dirs empty).
- [V2][M] **Crowd LOD (CrowdRenderer)** — MultiMesh static frames beyond the observation radius; doctrine-critical for settlements (§0 crowd budget). Budget test at 300 citizens on a mid laptop.
- [WIP][S] **Camera** — smooth-follow, edge clamp, 2–3 fixed zoom steps.
- [V2][M] **Day/night/weather rendering** — consumes §1.8: CanvasModulate tint from world clock, glow sprites for light sources (which are real, consumed items), weather overlays. *Browser:* no per-tile light shaders; the server owns visibility math.
- [V2][M] **Activity presentation** — observed citizens visibly do their schedule: work-loop animations at workplaces, carried-goods props, market stalls populating; reads sim rows, invents nothing.
- [WIP][M] **Spell/combat VFX** — projectiles, impacts, floating text from `combat_event`. AoE ground templates (sphere/cone/line) now render *true SRD shapes* including over allies — the reticle warns, it does not prevent. *Browser:* pool everything; cap simultaneous systems (~16).
- [V2][M] **Fog of war / exploration reveal** — per-character explored bitmask (server, bitpacked); exploration-only fog (live LoS fog stays rejected — §0 economics unchanged by doctrine).
- [WIP][S] **Minimap** — tile data + entity dots; [V2][S] explored-mask integration; guards/known-citizens colored by attitude.
- [V2][M] **World map screen** — regional map (keyart pipeline) with zone nodes and *travel-time* labels (RAW pace, §4) instead of fast-travel buttons.
- [WIP][S] **Zone loading/transition** — portal → fade → resubscribe → rebuild; frees previous atlases (§0).
- [WIP][S] **Performance/debug HUD** — FPS, entity count, drain-queue depth, wasm heap; add sim-event lag (server `next_event_at` overdue depth) once §1 lands.
- [V3][L] **Painted zone backdrops** — keyart-pipeline large backgrounds; only after asset streaming.

## 3. GUI

- [WIP][M] **Main menu / intro cinematic** — `MainMenu.gd` + `IntroCinematic.gd` per DESIGN.md (neither file exists yet; Login.gd/Credits.gd/Theme.gd do); keyart [DONE] feeds both; doubles as iOS audio unlock.
- [WIP][M] **Character creation** — species/class/standard array/name (`CharCreate.gd` exists). [V2][M] backgrounds (SRD), starting equipment per class table (RAW packs), point-buy toggle (SRD variant — both are rules-legal). [V3][M] portrait picker.
- [WIP][S] **Character select** — exists; [V2][S] class/level/zone display + delete-with-confirm.
- [WIP][M] **HUD** — HP, target frame, cast bar, condition icons ([DONE] in `assets/icons/`), zone toast. [V2][S] add: death-save pips (3/3, doctrine-critical), exhaustion level, light-level indicator, world-clock/date widget.
- [WIP][M] **Action bar** — exists (`ActionBar.gd`). [V2][M] quickbar customization with server-persisted layout (`quickbar_slot` table). [V2][M] **stance/intent controls**: Dodge/Disengage/Dash/Ready as real actions, nonlethal-intent toggle (feeds arrest rules §1.7).
- [V2][M] **Reaction policy panel (new, doctrine-critical)** — the player-configured standing policies that *are* the reaction economy (§5): "Opportunity attacks: always/never/not-vs-allies", "Shield: when hit and slot ≥1 remains", "Counterspell: vs spells ≥ level N within 60 ft", "Ready: <trigger grammar> → <action>". Server-validated data rows (`reaction_policy` table).
- [V2][M] **Spellbook + preparation UI** — known vs prepared, prepare-on-long-rest, ritual tags, **component requirements displayed** (V/S/M with gp costs; §4).
- [V2][M] **Character sheet / paper doll** — full abilities/skills/saves/AC/attack breakdowns, equip slots (`equipment_slot` table), **carry weight vs STR×15**, hit dice, death-save history, XP to next level (RAW table §10), lifestyle tier and daily upkeep.
- [WIP][S] **Inventory** — flat list (slice). [V2][M] grid + tooltips with full SRD stat blocks + **weight column and encumbrance readout** (RAW capacity; variant encumbrance as an SRD-legal server option flag). [V2][S] component pouch/focus slot handling.
- [V2][M] **Loot & trade windows** — loot per SRD treasure results (§1.9 #14); trade = escrowed two-pane (`trade_session` table); items freely tradable (binding removed per ledger). Player-shop manage/browse panels per §1.4 (staged in `features/trading/`).
- [WIP][S] **Journal / quest tracker** — exists per slice; [V2][M] chapter grouping, archive, pinned tracker; quest lines display *world-clock deadlines* where the pacing engine (§1.9 #12) sets them.
- [WIP][M] **Dialogue UI** — `dialogue_view` renderer + portraits; [V2][S] history scrollback; attitude tier shown on the NPC nameplate/portrait frame.
- [V2][S] **Rest UI** — short (1 game hour) / long (8 game hours) rest initiation with real-time equivalents shown, hit-dice spend picker, interruption per RAW; campfire vignette.
- [V2][M] **Level-up wizard** — HP roll-or-average, features, subclass at 3, spells, ASI; triggered by RAW XP thresholds (§10); server validates every choice.
- [V2][M] **Party frames + target-of-target** — up to 6, HP/conditions/death-save pips; ToT stays useful for reading *any* creature's target (§9 decision-making is legible).
- [WIP][M] **Combat log with full roll math** — `CombatLog.gd` exists; every `combat_event.text` carries complete breakdowns ("d20 17 +5 = 22 vs AC 15, hit, 7 slashing"); [V2][S] initiative-order round header lines ("Round 3 — Wren 21, wight 14, you 9"), death-save lines, filter tabs.
- [WIP][S] **Chat panel** — zone + `/g` (exists). [V2][M] tabs/whisper/party channels, unread badges. *Browser:* ~500-line scrollback cap.
- [V2][S] **Emotes & bios** — chat verbs + 500-char server-stored bios.
- [WIP][S] **Settings** — volume (exists); [V2][M] video/keybinds/accessibility/UI scale.
- [V2][M] **Tooltip service** — unified: items, spells (full SRD text), conditions, rules terms ("what is heavily obscured?" — the teaching surface for §13).
- [V2][S] **Drag-drop framework** — one shared implementation (quickbar/inventory/trade/equip).
- [V2][M] **Death & dying UI (replaces respawn overlay)** — unconscious screen with live death-save rolls, stabilization status, nearby-help indicator; on death: corpse/afterlife screen with resurrection options, costs, and time windows (Revivify 1 game-minute countdown is a real, visible countdown). The slice's downed/respawn overlay [WIP] ships v1 and is replaced.
- [V3][M] **Mail UI** · [V3][L] **Auction house UI** — unchanged infra items; auction remains paginated-reducer-only (*Browser:* never subscribe the listings table).
- [WIP][S] **Credits & legal screen** — `Credits.gd` exists; must render CC-BY attribution (§14).

## 4. Character Systems (RAW)

- [WIP][M] **4 slice classes, levels 1–3** — Fighter/Rogue/Wizard/Cleric, SRD math in `server/Rules.cs`/`GameData.cs` (both live).
- [V2][L] **Levels 1–5, subclasses at 3** — Champion, Thief, Evoker, Life Domain; Extra Attack at 5; spell tiers 1–3; per-level feature tables data-driven.
- [V2][L] **+4 classes (Ranger, Paladin, Bard, Druid)** — Wild Shape = stat-block swap onto the unified creature interface (§1.1 makes this natural).
- [V3][XL] **All 12 SRD classes** — Barbarian, Monk, Sorcerer, Warlock (pact slots, ki as distinct resources).
- [V3][XL] **Levels 6–20** — spell tiers 4–9. Doctrine change: high-tier spells implement **as written** wherever the server can express them (Teleport, Plane Shift = travel between generated zones; Simulacrum = a real second statted creature); only spells that are *technically impossible* (Wish's open clause) get rulings-engine option menus — `mmo_variant` flags are replaced by `rulings_menu` data.
- [V2][M] **Backgrounds** — SRD backgrounds: 2 skills, tool/language, equipment, feature codified as a mechanical hook where possible (Acolyte's temple access = attitude bonus at temples via §1.5).
- [V2][M] **Skills & tools, full list** — all 18 skills + tool proficiencies usable in dialogue, world interactions, professions (§1.4), and downtime; passive Perception live everywhere (§1.7 detection).
- [V2][S] **Ability Score Improvements** — at RAW levels in the level-up wizard. [V3][M] **Feats** from SRD 5.2.1 (CC-BY) as ASI alternative.
- [V3][L] **Multiclassing** — RAW prerequisites, merged slot table.
- [V2][M] **Concentration (RAW)** — one effect; CON save DC 10-or-half-damage on damage; drop on incapacitated/dead/second concentration.
- [V2][M] **Full condition set** — all 15 SRD conditions + exhaustion 1–6 as `condition_row.kind` with exact rules effects (icons [DONE]).
- [V2][M] **Resting, RAW (reversal)** — short rest = 1 game hour (2 real min at 30:1), spend Hit Dice; long rest = 8 game hours (16 real min), ≥6 sleeping, all HP + half hit dice, one per 24 game-hours, broken by 1 game-hour of strenuous activity; **anywhere** — wilderness rests roll watch-by-watch encounter checks from regional tables (§1.9 #8). Inns sell *comfort* (lifestyle, roleplay, safety), not exclusive rest rights.
- [V2][M] **Spell components (new, RAW)** — V blocked by silence, S requires a free hand, M requires component pouch/focus; **costed materials must be owned and are consumed when the spell says so** (Revivify's 300 gp diamond, Identify's pearl). Inventory checks in `CastSpell`; component items in the SRD price data (§8).
- [V2][L] **Death rules, RAW (reversal, doctrine-critical)** — 0 HP → unconscious (not "downed"): death saving throws each round on the character's initiative anchor (flat d20 DC 10; 3 fails = dead; nat 1 = 2 fails; nat 20 = up with 1 HP; damage at 0 = 1 fail, 2 on crit; instant death on damage ≥ max HP overflow). Stabilization: DC 10 Wisdom (Medicine), healer's kit auto-stabilize, Spare the Dying. Stable = unconscious, 1 HP after 1d4 game-hours. **Dead = dead until resurrection magic** (§1.6 services; components consumed; time windows enforced on the world clock). **[OWNER DECISION FLAGGED]: permadeath-if-unraised** — full RAW means a character not raised within the Raise Dead window (10 game-days) is gone barring Resurrection (1,000 gp) or True Resurrection (25,000 gp); the soft-floor alternative keeps True Resurrection eventually reachable for any character at brutal cost. Decision recorded here when made; risk analysis in (b)#3. Slice keeps its 15 s downed rule until this lands.
- [V2][M] **Encumbrance (RAW)** — carrying capacity STR×15 lb, push/drag/lift ×30, enforced on pickup and movement; SRD *variant* encumbrance (STR×5/×10 speed penalties) as a server option flag — both are rules-legal, default = standard.
- [V2][M] **Travel (RAW, new)** — overland pace fast/normal/slow (30/24/18 mi per game-day) between zones on the world map; forced march CON saves (DC 10 + hours past 8) → exhaustion; Survival navigation checks in trackless regions; foraging; watches with encounter rolls; mounts per SRD speeds/prices. This *is* the fast-travel system: travel is simulated, arrival is scheduled on the world clock, and the player can play the encounters or auto-resolve them (auto-resolve uses Tier-2 batch resolution — same dice).
- [V2][M] **Downtime activities (RAW, new)** — between adventures at day granularity: Crafting (5 gp progress/day, half-cost materials), Practicing a Profession (lifestyle maintenance per SRD), Recuperating, Researching, Training (250 days, 1 gp/day for languages/tools). Same machinery citizens use (§1.4) — players are citizens with agency.
- [V3][S] **Alignment** — bio field only; the SRD gives it almost no mechanics; where a rule references it (protection spells), implement that rule only.

## 5. Combat Depth (real-time rounds, initiative inside them)

- [WIP][M] **Core auto-attack round loop** — d20+bonus vs AC per 6 s, crits, advantage, damage types (`server/Combat.cs`, `Tick.cs` — both live).
- [WIP][M] **Cast wind-up + resolution** — wind-up, slot validation at finish, attack/save/auto/heal/buff (slice).
- [V2][L] **Initiative order within the 6-second round (evaluated → ADOPT)** — the SRD's round is 6 seconds and *turns are ordered by initiative inside it*; the current model (personal 6 s cooldowns) loses "start/end of turn" anchors, which RAW needs for death saves, save-ends conditions, readied actions, and legendary actions. Design: on encounter start, all participants roll initiative (d20+DEX, RAW tiebreaks); each combatant's **turn anchor** = round_start + rank × (6 s ÷ N participants); at the anchor, the entity's queued intents resolve as its turn (action + bonus action + reaction reset + movement budget = speed per round — the existing `speed_tiles_per_s` already equals the RAW budget: 30 ft/6 s), start/end-of-turn effects fire, death saves roll. Movement input stays continuous (locked real-time decision) but consumes the turn's budget. Cost: an action clicked mid-round waits ≤6 s for its anchor — the same latency the cooldown model already imposed, now rules-shaped. *Browser:* strictly friendlier to WebSocket jitter — intents queue, the server's clock resolves. **Full-RAW variant (movement only on your turn) rejected as conflicting with the locked real-time decision; recorded as a deviation with rationale.**
- [V2][M] **Reaction economy, real (reversal)** — one reaction per round, reset at your anchor. Opportunity attacks per RAW (leaving reach without Disengage/teleport). **Readied actions**: trigger grammar (§1.9 #18) + held action, released when the trigger fires between anchors. **Shield/Counterspell as standing policies** (§3 panel): the policy *is* the player's reaction decision made in advance — the codified answer to "the player would decide in the moment", latency-proof by design. NPCs use per-stat-block policy tables.
- [V2][M] **Concentration breaks** — per §4; visible tether VFX.
- [V2][M] **AoE templates + friendly fire (reversal)** — true SRD shapes (sphere/cone/line/cylinder) in tile units, server-validated; **all creatures in the template are affected — allies, bystanders, shopkeepers**. Evoker's Sculpt Spells implements as written and matters again. Bystander harm engages crime/attitude systems (§1.7, §1.5). No open-world/instance policy split — one rule everywhere.
- [V2][M] **CC per RAW (reversal)** — diminishing returns deleted; conditions run their written durations/saves; chain-lock is legal. The world's own counters: Legendary Resistance on legendary stat blocks (SRD), repeat-save-ends conditions, Counterspell/Dispel Magic from enemy casters (§9 uses them), and morale (§1.9 #2) — degenerate lock-downs are answered by the simulation, not a modifier.
- [V2][M] **Saving-throw auras & recurring effects** — `aura_row` (holder, radius, save, effect, period) evaluated on turn anchors (Spirit-Guardians-shaped; foundation for bosses).
- [V2][M] **Line of sight & cover** — Bresenham over the wall grid; cover auto-derivation (§1.9 #6): +2 AC/DEX-save half, +5 three-quarters, full = untargetable; obscurement from light/weather (§1.8) applies as written.
- [V2][L] **Stealth/detection (RAW via §1.9 #13)** — Stealth vs passive Perception, light and obscurement honored, hidden = unseen attacker rules (advantage; attack reveals). Deviation removed: other players do *not* get a fairness silhouette — hidden is hidden from creatures who fail to notice, players included.
- [V2][M] **Mounted & underwater rules** — SRD chapters exist; data-cheap once the creature interface is in: mount speeds, underwater weapon restrictions, swim speeds. Fen and mere content wants them.
- [V2][M] **Boss encounters = SRD stat blocks run honestly** — legendary actions (between turn anchors), legendary resistance, lair actions on initiative 20 (anchor 0): the vocabulary is the SRD's own, data-driven from stat blocks — replaces the invented "boss script atoms". Drowned Warden upgrades to a wight run *by the book* plus authored lair data.
- [V3][M] **Grapple/shove/improvised** — contested checks per RAW; improvised verbs via §1.9 #19.

## 6. Party & Social

- [V2][L] **Grouping** — party up to 6: invite/accept/leave/kick, leader, `party`/`party_member` tables, party chat, minimap positions; shared XP per RAW (§10) and quest credit within zone + radius.
- [V2][M] **Loot: property, not loot rules** — a slain creature's possessions and rolled treasure (§1.9 #14) drop as claimable world property; party default = free-for-all with an optional leader-set rotation (a table convention, not a server rule). Personal-loot instancing removed as a house rule.
- [V2][S] **XP sharing (RAW)** — encounter XP divided equally among participants; participation = contributed action in the encounter.
- [V3][L] **Guilds** — create (gold sink), roster, ranks, MOTD, chat; [V3+] bank with audited withdrawals.
- [V2][M] **Friends/ignore** — online/zone status; ignore enforced server-side.
- [V3][M] **LFG listing board** — role-tagged listing + whisper; no teleport matchmaking (travel is real, §4).
- [V2][M] **Trading** — escrowed window (staged in `features/trading/` now); every trade audited (GM/telemetry).
- [V3][M] **Mail** — async transfer with delivery *time computed from actual courier travel* (§1.4 trade routes) — the doctrine version of the RMT-friction delay.
- [V2][S] **Emotes/RP tools** — verbs, `/e`, bios.
- [DONE][—] **Voice: none (decision)** — text-only, permanently.
- [V2][M] **Moderation (player-facing)** — /report + context capture, mutes, profanity filter, rate limits. Doctrine-neutral: keep in full.
- [V2][—] **PvP = the simulation (flagged)** — attacking a player is attacking a creature: same initiative, same death saves, same crime consequences in lawful areas (§1.7); lawless wilderness is lawless. **[OWNER SIGN-OFF REQUIRED]** before enabling outside duels — risk (b)#5; the doctrine-consistent mitigations are law severity data, resurrection economics, and bounty hunters, never a PvP flag.

## 7. World & Content (procgen-first)

- [WIP][M] **Slice zones ×3** — village/fen/crypt JSONs [DONE]; server embedding [DONE] (`Content.g.cs`); client rendering [WIP]. Ship as-is, grandfathered.
- [WIP-external][XL] **`tools/procgen` generator families** — wilderness, dungeon, settlement (with `population.json`), seeded/deterministic, emitting the existing zone-JSON contract; built by the parallel workflow. Consumer contracts: §1.10.
- [V2][M] **Constraint-overlay format** — authored quest/campaign content as overlays the generators honor; `tools/codegen` validates overlay-vs-output.
- [V2][L] **World assembly** — region graph (adjoining zones, travel distances in miles for §4), seeded generation of the launch region: the village (retrofit population) + N wilderness + M dungeon + a second full settlement as the procgen proof. Content scale is parametric — zone count is a knob, not a milestone.
- [V2][M] **Zone lifecycle at scale** — zones instantiate from seed on first relevance and persist deltas only (kills, looted props, construction); Tier-2 simulation (§1.11c) keeps inhabited zones alive unobserved; uninhabited wilderness needs no events until entered (nothing scheduled = zero cost — the honest "sleep" that isn't an NPC freeze).
- [V3][L] **Dungeon instancing decision (unchanged infra)** — instances are **logical**: `instance_id` columns, party-keyed spawn sets, subscription filters — one module simulates all. Module-per-shard with character transfer stays the [V3][XL] escape hatch. Doctrine note: dungeons are *places*, so default remains open-world shared; logical instancing reserved for story-critical set pieces.
- [V3][M] **Dynamic events → world pacing engine** — §1.9 #12 scripts: the world advances on its calendar whether or not players engage (the cult ritual fires; the village burns or doesn't) — consequences as physics.
- [V2][M] **Day/night & seasons** — from §1.2/§1.8; night-gated content becomes real: shadows spawn per spawn tables' time windows.
- [V3][L] **Player housing** — buy real buildings from the settlement registry (§1.6 estates); ownership rows, furniture from prop catalog; a gold sink that is also a simulation feature.
- [V2][L] **Campaign/module pipeline** — the data trio + overlays + validators = the module format; schema docs, `validate_all` CLI, hot-reload into a dev module; second campaign as proof. [V3][XL] **Community modules** — sandboxed namespaces, data-only scripting, quotas, review pipeline, separate databases per community shard; the NWN-shaped differentiator.

## 8. Economy & Items (SRD-anchored)

- [WIP][S] **Slice item catalog** — `data/items.json` [DONE]; server grant/use [WIP].
- [V2][L] **SRD equipment catalog at SRD prices, verbatim** — all weapons (~37), armor (~13), gear, tools, **spell components with costs**, light sources, food/drink/lodging, mounts/vehicles, trade goods. Prices are not tuning inputs — they are the rules. `data/srd/equipment.json`.
- [V2][M] **Magic items, Common/Uncommon** — SRD items as written; [V3][L] Rare+ with **attunement (max 3)** per RAW. Items whose text demands DM adjudication get rulings-menu data, not exclusion.
- [V2][M] **Vendors = businesses (§1.4)** — stock from production + trade, prices = SRD × scarcity band; buyback = the shop reselling what it bought; haggling = a real Persuasion interaction (§1.5) with codified DC/price steps.
- [V2][M] **Player-owned shops (owner directive)** — see §1.4; staged in `features/trading/` with the offline NPC-clerk model, rent + wages as RAW sinks, sales ledger, escrow eviction.
- [V2][M] **Money & sinks per RAW** — faucets: treasure per CR tables, wages for downtime work; sinks: **lifestyle expenses (daily, everyone)**, **resurrection components (the big one)**, spellcasting services, component consumption, training, mounts, property, shop rent + clerk wages. Removed house-rule sinks: durability fees, binding, listing deposits. Telemetry instruments faucets/sinks from day one (§11).
- [V2][S] **Currency per SRD** — cp/sp/ep/gp/pp with real weights (50 coins/lb counts against encumbrance).
- [V3][L] **Crafting per RAW downtime** — 5 gp/day progress, half-cost materials, tool proficiency required; the same system citizens use (§1.4).
- [V3][XL] **Auction house** — reframed as a **brokerage business** in a hub settlement, regional not global. *Browser:* paginated reducer queries only.
- [V2][M] **Bank/stash** — strongbox rental at the village hall (a business with a ledger); [V3][S] letter-of-credit shared access.
- [WIP][S] **Loot tables** — shapes [DONE]; [V2][M] regenerate defaults from CR treasure tables so every stat block gets rules-derived drops.

## 9. Monster & NPC Combat AI (rules-faithful decision-making — threat tables replaced)

- [WIP][S] **Slice mob AI** — idle→aggro→chase→attack→leash (`MobAi.cs` — skeletal). Ships for v1; superseded.
- [V2][L] **Creature decision engine (the threat-table replacement)** — per-turn-anchor decisions from the stat block: **INT ≤ 3**: attack nearest/last-attacker, flee at morale failure, Pack Tactics positioning; **INT 4–7**: strongest attack, focus wounded, retreat on morale break; **INT 8–11**: action-economy literate — Multiattack, Dodge when cornered, Disengage to reposition, focus casters, use cover; **INT 12+**: policy-table tactics — Counterspell priorities, focus-fire, terrain, parley when losing. Tables in `data/rulings/combat_ai.json`; resolver on the unified creature interface — guards, citizens, monsters, and summons all think with the same engine.
- [V2][M] **Stat-block fidelity** — Multiattack as written, recharge on real 5–6 rolls, spellcasting traits cast real spells from real slots, legendary/lair actions (§5), resistances/immunities exact, condition immunities exact.
- [V2][M] **Morale (§1.9 #2)** — flee/surrender/rout; undead and constructs fight to destruction; surrendered creatures are prisoners (crime system covers player brutality).
- [V2][L] **SRD bestiary, CR 0–4 (~120 blocks)** — PI-exclusion list guards the pipeline. [V3][L] CR 5–10; [V3][L] CR 11+ — gated on level range and sprite generation, not invented balance bands.
- [V2][M] **Ecology & spawning** — wilderness creatures get territory + schedule rows like citizens (a wolf pack dens, hunts at dusk, on the same event queue); "respawn" replaced by **population + migration** for intelligent creatures and breeding-rate abstractions for beasts; dungeon undead don't repopulate without an authored necromantic cause. Slice respawn timers grandfathered until this lands.
- [WIP][S] **Leashing** — slice mechanic; superseded by real pursuit decisions (morale/territory rules, not an invisible tether).

## 10. Progression (RAW XP)

- [WIP][S] **Milestone leveling (slice)** — ships v1, then replaced.
- [V2][M] **XP from CR, RAW (reversal — primary progression)** — every defeated creature awards its CR's XP, divided equally among participants; thresholds per the SRD table (300 → L2 … 355,000 → L20). "Defeated" includes routed/surrendered. `xp` column + award path; level-up wizard at thresholds.
- [V2][S] **Non-combat XP** — authored awards via §1.9 #11: quests, traps/hazards, social victories — CR-equivalent guidance documented.
- [V2][S] **No XP loss, no cap beyond content** — death costs resurrection components and time; level cap = highest implemented range, never a tuning cap.
- [V3][M] **Renown/faction standing** — attitude machinery at organizational scale: titles, access, services.
- [V3][M] **Achievements & titles** [NO-PROVENANCE — doctrine-neutral infra, owner-approved by prior use] · [V3][M] **Cosmetics rail** (monetization lane, zero rules impact) · [V3][S] **Leaderboards**.
- **Endgame under the doctrine** — not a treadmill: high-CR regions, faction politics in living settlements, property/downtime depth, world-pacing consequences. Dungeon lockouts/affix tiers deleted (pure balance constructs, no provenance).

## 11. Server / Infra & Live-Ops (doctrine-neutral — kept, with sim additions)

- [WIP][S] **Accounts/auth v1** — identity + localStorage token. [V2][M] **OIDC upgrade** — not optional for real launch (token loss = character loss).
- [WIP][S] **Characters per account** — multiple; [V2][S] cap 6 + delete grace.
- [V2][M] **Persistence/backups** — Maincloud backups + scheduled logical exports to owned storage.
- [V2][L] **Schema-upgrade strategy (critical)** — additive-only evolution, version column, lazy migration, staging rehearsal, written export→transform→import runbook. The §1 kernel adds many tables — additive discipline makes that cheap; *editing* `player` stays the hazard.
- [V3][XL] **Sharding & the no-cap truth** — measure single-module ceiling with bots; logical instances; module-per-region with character transfer third (a ferry ride's UX). §1.11's arithmetic raises the ceiling dramatically — measure, don't assume.
- [V2][S] **Anti-cheat audit cadence** — reducer checklist each milestone; new sim reducers join it.
- [V2][M] **Rate limiting** — per-identity token buckets in-module.
- [V2][M] **GM tools** — teleport, spawn, quest-step, mute/kick/ban, broadcast; **sim additions:** possess-citizen, attitude/law inspectors, world-clock step/pause (staging only), **rulings-trace viewer** (why did the guard do that? — dump decision inputs). [V3][M] web GM dashboard.
- [V2][M] **Telemetry** — counters (reducer rates, dispatch durations, event-queue overdue lag, encounters/hour, gold faucet/sink flows, deaths by cause, resurrection spend) + client beacons.
- [V2][M] **Crash/error reporting** — browser beacons; server log alert poller.
- [WIP][S] **Deploy pipeline** — `tools/deploy/*` [DONE]; [V2][M] CI: validate → build → staging publish → bot smoke → promote.
- [V2][M] **CDN/versioning + binding compatibility** — hashed assets; `protocol_version` config table; refuse-mismatch prompt.
- [V2][S] **Cost model** — Maincloud launch; self-host checkpoint at sustained-load data.
- [V2][L] **Load testing (elevated)** — Node bot harness + **sim-scale harness**: 10,000 citizens in staging, compressed game-days, dispatcher throughput/lag charts — the (b)#1 measurement. Blocks nothing; informs everything.
- [V2][M] **Chat moderation/filters** · [V2][S] **GDPR/privacy** (export/delete reducers) · [V2][S] **ToS/EULA** (versioned accept).

## 12. Audio

- [WIP][M] **Music per zone/state** — specced; `Audio.gd` exists; composer scripts are the open gap.
- [V2][M] **Ambience beds** — per-zone + **time-of-day variants from the world clock** — the sim is audible.
- [V2][M] **Positional SFX** — pan/attenuation; ~16-voice cap. Settlement soundscapes key off *actual citizen activities* in earshot.
- [WIP][S] **SFX set (~27)** · [WIP][S] **UI sounds** · [WIP][S] **Volume/ducking** — all init behind first gesture (§0 iOS).
- [V3][M] **Adaptive layers** — crossfade covers 80 % first.
- [V3][S] **Barks: text-only** — §1.5 tables; no voice ever.

## 13. Onboarding & Accessibility (teaching REAL 5e)

- [V2][M] **Teach the actual rules** — the tutorial teaches 5e as written because the game *is* 5e as written: popovers on first miss (AC), first save, first slot, first condition, **first death save**, first exhaustion, first lifestyle bill; every popover cites the real rule term. A player who learns Hollowmere has learned tabletop 5e — a feature, market it.
- [V2][M] **"Why?" expander everywhere** — combat rolls, guard responses, price changes, attitude shifts expose their rule/ruling inputs on demand. The simulation is legible or it is not trusted.
- [V2][S] **Contextual help** — `?` mode → rule card (tooltip service §3).
- [WIP][S] **Players' Guide** — [DONE]; regenerate under the doctrine: deviations chapter shrinks to the recorded, justified list.
- [V2][S] **Colorblind palettes** · [V2][S] **Font scaling** · [V2][S] **Remappable keys** (*Browser:* never bind Ctrl+W) · [V2][S] **Reduced motion**.
- [V2][S] **Screen-reader honesty** — ARIA live-region mirror for chat/log/dialogue; full canvas AT support out of scope; say so publicly.

## 14. Legal & Business

- [DONE][S] **SRD CC-BY attribution** — LEGAL.md exists; [WIP][S] verify in-game Credits + hosting footer.
- [V2][S] **Trademark hygiene** — PI blocklist in `tools/codegen` validators *and the procgen pipeline*; "Hollowmere" mark search.
- [V2][S] **SRD version posture** — 5.1 base; adopt 5.2.1 content deliberately per system; `docs/SRD_VERSIONS.md` ledger, cross-referenced to the rules ledger.
- [V3][M] **Monetization (goodwill-compatible)** — cosmetics + supporter tier + module box-price; hard nos: pay-for-power, loot boxes, energy timers. **New hard no under the doctrine: selling anything the rules price in gold** (resurrections, components).
- [V2][S] **Age rating posture** — PEGI 12/ESRB T; filters default on.
- [WIP][S] **License inventory** — [partially DONE; keep current].

---

## (a) V2 Cut — dependency-ordered, led by the settlement-simulation kernel

The kernel is items 1–8; nothing doctrine-flavored ships before it because everything consumes it.

1. **World clock + calendar** (§1.2) — every other system timestamps against it.
2. **Unified creature interface + citizen stat blocks** (§1.1) — the load-bearing refactor.
3. **Event queue + SimDispatch dispatcher** (§1.11a) — the world-never-sleeps engine.
4. **Schedules + needs + lifestyle upkeep** (§1.3) — first visible living settlement.
5. **Rulings engine v1** (§1.9 entries 1–8, 13, 16) — the codified DM's core.
6. **Spatial index (grid cells + composite B-tree)** (§1.11b).
7. **Death saves + resurrection services** (§4, §1.6) — owner decision filed alongside.
8. **Settlement/population generator integration** (§1.10) — retrofit the village; stand up settlement #2 from seed.
9. **Initiative anchors + real reaction economy + standing policies** (§5, §3).
10. **Friendly-fire AoE templates + LoS/cover** (§5).
11. **Full condition set + concentration + exhaustion** (§4).
12. **Creature decision engine + stat-block fidelity + morale** (§9).
13. **XP from CR + RAW thresholds + level-up wizard** (§10, §3).
14. **SRD equipment/price/component data + professions→stock→prices loop** (§8, §1.4).
15. **Player shops + P2P trading merge** (§1.4, §8 — staged in `features/trading/` now).
16. **Crime & guard response + law tables** (§1.7).
17. **RAW resting + travel + downtime** (§4).
18. **Rules ledger live** (docs/RULES_LEDGER.md + validators — seeded by workstream G; every subsequent item extends it).
19. **Doctrine-neutral floor, in parallel throughout:** party system + frames, chat channels/whispers/friends, OIDC, GM tools + rulings-trace, telemetry, rate limiting, CI + bot/sim load harness.

## (b) Hardest architectural risks

1. **Single-module city-sim cost.** Steady state is trivial (~70 events/s at 10 k citizens, 30:1 clock) but three multipliers are unproven: (i) *concurrent encounters* (a settlement-wide riot is thousands of round-granular turns), (ii) *observed-tier pathfinding* (A* per moving observed citizen), (iii) *subscription fan-out* (hundreds of citizen rows × dozens of observers). Mitigations in design — fidelity tiers, batch resolution, event-granularity positions, encounter caps as guard-escalation *data* — but the number comes only from the sim-scale harness. Build the 10 k-citizen staging measurement before settlement #3.
2. **Observed/unobserved tension.** Outcome-equivalence is clean philosophically, fragile at the seams: arrival mid-batch must see consistent state; path-dependent outcomes differ across tiers by construction; determinism must be per-event-seeded so tier promotion never re-rolls history. Owner sign-off required (§1.11c); guard = tier-transition test suite asserting identical rule outcomes across promotion/demotion.
3. **Death-RAW retention risk [OWNER DECISION].** RAW death + costly resurrection is the doctrine and the single most churn-dangerous rule in the game. File: permadeath-if-unraised vs True-Resurrection soft floor. Sub-risks: background-tab death saves (settle the throttling policy *with* the death rule); low-level players face the harshest window (Revivify needs a 5th-level cleric + 300 gp) — temple pricing data effectively sets the real death penalty curve. Instrument deaths-by-cause and resurrection uptake from day one.
4. **The judgment-gap catalog is the true scope of "exact simulation."** ~40–60 codified rulings; each small, collectively they *are* the DM; every one is a place where two agents can codify contradictory common sense. Control: single tracked catalog + rules-ledger cross-reference, every resolver cites its data file, rulings-trace makes each decision auditable in-game. The failure mode isn't missing rulings — it's silent, inconsistent ones.
5. **Browser carryovers + the lawful-griefing bet.** Schema evolution without wipes; single-threaded frame budget now rendering living crowds; tab-throttle fairness (entangled with #3); the SRD combinatorial validation surface (the headless scripted-combat harness must grow *with* V2). New: friendly fire + open-PvP-as-simulation means anti-griefing IS the crime/law/bounty simulation — elegant, unproven at MMO scale, gated on owner sign-off. If law-not-code fails in playtests, the doctrine-compliant lever is harsher law *data* (execution, exile, resurrection denial), not code prevention — plan the data escalation path now.

## (c) Execution Order — the blitz plan (dependency order only; no calendar exists)

**Workstreams launchable RIGHT NOW, in parallel, strict file-ownership partitions:**

| WS | Scope | Owns (exclusively) |
|---|---|---|
| **A — Slice server completion** | v1 combat/movement/mob-AI/dialogue to DESIGN parity | `server/*.cs` (existing files only) |
| **B — Bridge** | TS SDK wrapper → `hollowmere_net.js` | `bridge/**`, `client/web/**` |
| **C — Client world** | World scene, TileMap builder, EntityView, camera | `client/src/world/**`, `client/src/game/Game.gd` |
| **D — Client GUI** | MainMenu, IntroCinematic, HUD, dialogue, journal, settings | `client/src/ui/**` |
| **E — Asset pipelines** | Blender sprite/tile scripts, music/sfx synth | `tools/blender/**`, `tools/music/**`, `assets/**` |
| **F — Simulation kernel** | §1: clock, dispatcher, citizen tables, creature interface, schedules, spatial index | `server/sim/**` (staged now in `features/simulation/`) |
| **G — SRD data + rules ledger** | Stat blocks, equipment+prices, spells, XP/CR tables, conditions; ledger seed | `data/srd/**`, `docs/RULES_LEDGER.md`, `tools/srd/**` |
| **H — Rulings catalog** | §1.9 data authoring + resolver specs | `data/rulings/**`, `docs/RULINGS_CATALOG.md` |
| **I — Procgen** | (running) generators + `population.json` | `tools/procgen/**` |
| **J — Live-ops** | CI, bot harness, telemetry spec, runbooks, legal upkeep | `tools/deploy/**`, `.github/**`, `docs/runbooks/**` |
| **K — Player economy** | (running) shops + trading staged | `features/trading/**` |

Cross-workstream writes go through DESIGN.md contract PRs, never direct edits to another stream's files.

**The true critical path (hard dependencies only):**

```
A (slice combat parity) ──────────────┐
B (bridge) → C (world scene) → playable slice → publish v1
                                      │
F1 world clock → F2 dispatcher → F3 citizen tables + creature interface
      → F4 schedules/needs → F5 observed/unobserved tiers
      → [F6 spatial index ∥ F4]
G (SRD data) ────────→ feeds F3, §5 reversals, §10 XP, the rules ledger
H (rulings v1) ──────→ feeds F4/F5, §5, §9, §1.7
I (procgen population.json) → F7 population loader → settlement #2 from seed
      → 10k-citizen sim harness measurement (J) → scale decisions (b)#1
A + F3 → §5 initiative anchors → reactions/policies → death saves (§4) → crime/guards (§1.7)
K (trading/shops staged) → merges after A publishes v1
```

**Waves (batched strictly by what-blocks-what):**

- **Wave 1 — no prerequisites, in flight now:** all eleven workstreams. Inside F: clock → dispatcher → citizen table proceed against DESIGN shapes; G and H are pure data work; E is pure pipeline work; B/C/D race to the playable slice; K stages against conventions.
- **Wave 2 — blocked only by Wave-1 contracts:** initiative-in-round + reaction economy (A + F1); death saves/resurrection (A + F1 + G components); friendly-fire AoE + LoS/cover (A + rulings #6); full conditions/concentration/exhaustion (G + creature interface); XP-from-CR (G); population loader + village retrofit (F3 + I); citizen rendering + crowd budget (C + F3); reaction-policy + death UI (D + server rows); trading/shops merge (K + A published).
- **Wave 3 — integration layers:** crime & guards (spatial index + creature decisions + combat reversals); professions→stock→prices (schedules + SRD prices); social/attitude + gossip; weather/light with darkness rules; RAW rest/travel/downtime; creature decision engine everywhere; GM rulings-trace + sim telemetry.
- **Wave 4 — scale & proof:** 10k-citizen staging measurement; tier-transition test suite; settlement #2 fully alive from seed; second campaign as overlay proof; owner decisions executed live (death-RAW mode, PvP-as-simulation, observation policy, world-clock ratio).
- **V3 ring:** remaining classes/levels 6–20, CR 5+ bestiary, rare+ items/attunement, guilds/housing/brokerage, trade routes + courier mail, community modules, module-per-region sharding if — and only if — the Wave-4 measurement demands it.

Maximum speed rule: any item whose dependencies are met is launchable immediately by any free agent team; nothing in this document waits for a date, only for its inputs.

---

### Critical Files for Implementation

- `DESIGN.md` — the binding contract; non-negotiables #6/#7/#8 govern this roadmap; every new table/reducer shape lands here first.
- `server/Lib.cs` — existing schema + lifecycle; additive-evolution discipline and the player/citizen boundary live here.
- `server/sim/` (new; staged in `features/simulation/`) — the kernel: `WorldClock.cs`, `SimDispatch.cs`, `Citizen.cs`, `Creature.cs`, `Schedule.cs`, `Spatial.cs`, `Rulings/*.cs`.
- `docs/RULES_LEDGER.md` + `data/rules_ledger.json` — the bijection's enforcement artifact (workstream G seeds it; every feature extends it).
- `tools/codegen/embed_content.py` — the data→server/client pipeline; grows to validate/embed `data/srd/**`, `data/rulings/**`, and `population.json`.
- `data/rulings/` — the codified DM: DC ladders, morale, combat AI, law, attitude shifts — the judgment-gap catalog as data.
