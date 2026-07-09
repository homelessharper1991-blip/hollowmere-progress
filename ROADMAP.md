# Hollowmere â€” Master Inventory & Roadmap

## Changelog (append-only; newest first)

- **2026-07-09** - **FIRST COMBAT:** full loop live (create - travel - mob aggro/chase - 6s rounds with real d20 math - paralysis riders - crits - death - loot). Bridge bundle smoke-tested; world renderer parse-clean. A graveyard ghoul paralyzed and dropped the first fighter who challenged it - as the book intends.

- **2026-07-09** — **FIRST BOOT:** server module published locally; world seeded (8 NPCs, 33 mobs, 3 zones); first character created via reducer with correct RAW math (fighter, 12/12 HP). Movement/combat systems still landing.

- **2026-07-09 (later)** â€” **FOUR OWNER RULINGS:** â‘  PERMADEATH YES, with resurrection contracts â€” telepathic cleric-call (SRD *Sending*), player clerics get first claim, NPC casters are fallback, RAW time windows + gold set the real bounds. â‘¡ OPEN PVP YES, governed by full crime/bounty simulation. â‘¢ Observed/unobserved = outcome-equivalent batch resolution (delegated; DM-practice-accurate). â‘£ Prior kd-tree notes SCRAPPED; owner intent = kd-tree for dynamic server-cell assignment, settled in a dedicated review. **NEW DIRECTIVE: SERVER CELLS** (multi-database world, atlas module, portal-aligned transfers) â€” promoted to V2, staged in `features/cells/` (DESIGN.md #9).

- **2026-07-09** â€” **OWNER DECISION: spatial index = uniform grid cells as indexed DB columns** (kd-tree benched with re-entry condition). Â§1.11(b) recommendation confirmed; DESIGN.md #8 updated.
- **2026-07-09** â€” Server core landed content embedding (`Content.g.cs` exists; module builds/publishes locally per workstream A).
- **2026-07-09** â€” Full regeneration under all six owner directives (simulation doctrine, no timelines, procedural zones, world-never-sleeps, player economy, rules bijection). Maintenance process: `tools/roadmap/check_status.py` flags stale tags from file-system evidence; updates are tag flips + changelog lines at milestones; full regeneration only on directive-level changes.

**Status:** living document, regenerated 2026-07-09 under the **Simulation Doctrine** (DESIGN.md non-negotiables #6, #7, #8). Companion to `DESIGN.md` (the binding contract) and `LEGAL.md`. Replaces the previous balance-driven roadmap in full.
**Scope:** everything a browser-based, exact-D&D-5e-SRD **simulation** MMO needs, from the current vertical slice ("The Drowned Vigil", 3 zones, 4 classes, level cap 3) to a persistent world of living settlements.

**Doctrine (governs every line below):** this is an **exact D&D simulation first, a game second**. Rules fidelity wins over balance even when the result is degenerate in play. Every mechanic mimics its SRD rule. Where the tabletop calls for DM judgment, a **rulings engine** â€” authored data plus deterministic resolvers â€” supplies what a DM would rule; nothing is left undefined. Settlements are the heart of the product: **every NPC is a full statted character** obeying all the rules â€” stat blocks, action economy, D&D time units, schedules, skill-checked daily activities, real death and consequences. **The world never sleeps:** every NPC always has a current activity and a scheduled next event, players present or not (SpacetimeDB scheduled reducers run with zero clients connected â€” the enabling property). The v1 slice ships as-is; the doctrine governs everything after it.

**The rules bijection (non-negotiable #7, highest importance):** every single SRD rule is implemented as a game mechanic, and every single game mechanic derives from an SRD rule. Unimplemented rule = defect; mechanic without rule provenance = defect. Enforcement artifact: **`docs/RULES_LEDGER.md` + `data/rules_ledger.json`** â€” the traceability matrix SRD section â‡† implementing mechanic/code (being seeded now by the SRD-ingestion workstream, G below). Every feature change extends the ledger; validators flag unmapped mechanics. Items in this roadmap marked **[NO-PROVENANCE]** require owner sign-off or grounding before build.

**Locked decisions (do not relitigate):** browser client (Godot 4.6 web export, Compatibility renderer, no-threads build); 2D isometric pre-rendered look; SpacetimeDB C# module is the sole authority; real-time 6-second rounds â€” now also a **fidelity** argument: the round is the SRD's own unit of combat time; 5e SRD CC-BY only (no WotC Product Identity â€” no beholders/mind flayers/named settings; SRD subclasses only); no designed-in on-screen player cap.

**Tags:** `[DONE]` exists and works in repo Â· `[WIP]` in the current slice per DESIGN.md / actively being built Â· `[V2]` next dependency ring after the slice Â· `[V3]` depends on V2 layers. **V2/V3 are dependency order only â€” never calendar time.**
**Effort tags are parallelization granularity, never duration:** `S` = one agent, one file/sitting Â· `M` = one agent, a few files Â· `L` = decomposes into 2â€“4 parallel agent tasks Â· `XL` = its own workstream with an internal file-ownership partition. There are no wall-clock estimates anywhere in this document, deliberately: the plan is a dependency graph to blitz with parallel agents, not a schedule.

**Doctrine reversal ledger** (prior MMO-balance house rules, now flagged deviations with their replacements â€” each re-appears in its section):

| Prior house rule | Doctrine ruling | Where |
|---|---|---|
| CC diminishing returns | **Removed.** Chain CC is legal. The SRD's own answer is Legendary Resistance on legendary stat blocks, save-ends conditions, and Counterspell/Dispel in the world | Â§5 |
| No friendly fire on AoE | **Removed.** Fireball hits allies and bystanders per SRD templates; consequences flow through the crime system, not code prevention | Â§5, Â§1.7 |
| Threat/aggro tables | **Replaced** by rules-faithful creature decision-making (INT/WIS-tiered targeting, stat-block traits, codified morale) | Â§9 |
| Simplified death (downed 15 s â†’ respawn) | **Replaced** by RAW death saves + stabilization; persistent-world answer = SRD resurrection magic with costly consumed components; **permadeath-if-unraised flagged for owner decision** | Â§4, Â§1.6, (b) |
| Reactions as stances only | **Replaced** by a real reaction economy: opportunity attacks, readied actions, Shield/Counterspell as player-configured standing policies (1 reaction/round enforced) | Â§5 |
| Milestone-only leveling | **Replaced** by RAW XP from CR as primary progression; authored XP awards for non-combat challenges via the rulings engine | Â§10 |
| Continuous per-entity 6 s cooldowns | **Initiative order within the round** adopted after evaluation (Â§5) â€” turns get anchors inside the 6 s round | Â§5 |
| Bind-on-pickup/equip, durability repair fees | **Removed** â€” items are freely tradable property per SRD; the economy's sinks are lifestyle expenses and resurrection components; RMT handled by moderation | Â§8 |
| Rest only at inns/campfires | **Removed** â€” RAW resting anywhere, with RAW interruption, the 1-long-rest-per-24 h limit, and wilderness encounter checks doing the pacing | Â§4 |
| PvP flags / consensual-only duels | **Doctrine-derived open simulation:** a player character is a creature; attacking one is an attack, adjudicated by the same rules and the same law/crime system. **Flagged for owner sign-off** (griefing-by-rules risk, (b)) | Â§6, Â§1.7 |

---

## 0. Browser Runtime Constraints (first-class, read before building anything)

The browser is not a port target; it is *the* platform. Every domain below carries inline *Browser:* notes where the platform bends a design. The simulation doctrine changes nothing here â€” all simulation is server-side; the browser only ever renders the observed slice of it. The new pressure point is **settlement rendering**: a living town puts hundreds of citizens on screen, so crowd LOD stops being polish and becomes doctrine-critical.

- [WIP][M] **Initial payload budget < ~50 MB** â€” engine wasm (~40 MB raw, ~15â€“17 MB brotli), main `.pck` with UI + village assets, first-zone atlases. CI check that fails the build if compressed initial payload exceeds budget; report per-asset sizes in `tools/deploy/package_client.ps1`.
- [V2][L] **Asset streaming for growth** â€” zone atlases/music fetched on demand (HTTP `ResourceLoader`/`HTTPRequest` into `user://` cache) rather than shipped in the main pck; prefetch adjacent zones while idling. Procgen (Â§7) makes zone count parametric â€” streaming is what keeps first paint constant while the world grows.
- [DONE][â€”] **Compatibility renderer / WebGL2 ceiling (decision)** â€” no compute shaders, no SSBOs; 4096Â² safe max texture size (atlas paging accordingly); canvas-item shaders only; MultiMesh2D works and is the crowd-LOD workhorse. All VFX must be sprite/CanvasItem-shader based.
- [DONE][â€”] **No-threads build (decision)** â€” plain static hosting, no COOP/COEP headers; cost: everything (render, JSON drain parse, audio trigger) shares one ~16 ms frame budget. Corollaries: chunk all JSON parsing, never parse a full zone in one frame, cap per-frame `drain()` event processing with carry-over queue. *Doctrine note:* citizen-heavy zones raise row churn â€” the drain cap plus event-granularity position updates for far citizens (Â§1.11) is the answer.
- [WIP][M] **Tab-throttling tolerance** â€” browsers clamp rAF/timers to â‰¥1 Hz when unfocused; the server keeps simulating (you can be attacked â€” and under RAW death rules, *die* â€” in a background tab). Client must: (a) keep the WebSocket alive via the JS bridge, (b) on refocus, drain the backlog in bounded chunks or force a zone resubscribe past a threshold, (c) show a "while you were away" summary. Gameplay policy for background death saves is part of the death-RAW owner decision (risk (b)#3).
- [V2][M] **wasm memory ceiling & eviction** â€” practical ceiling ~2 GB desktop, ~1 GB iOS Safari; on zone transition, free previous zone atlases/audio buffers; memory HUD in debug builds; bestiary/portrait atlases load per-zone, never globally.
- [DONE][â€”] **WebSocket-only networking (decision, and a strength)** â€” no UDP/WebRTC; TCP head-of-line spikes are invisible when actions resolve on 250 ms ticks and rounds last 6 s. The initiative-anchored round (Â§5) is *more* latency-tolerant, not less: intents queue, resolution happens on the server's clock. Say no to any future mechanic requiring <150 ms reactions â€” standing reaction policies (Â§5) exist precisely so no human has to click inside a reaction window.
- [WIP][S] **Persistence: localStorage/IndexedDB** â€” auth token in `localStorage` (bridge), settings/intro-seen flag in Godot `user://` (IndexedDB-backed; always `flush()` before navigation-away paths). Chat history session-only; never persist large blobs client-side.
- [V2][M] **iOS Safari quirks** â€” audio requires a user gesture (route through the "ENTER THE MERE" click); memory-pressure tab reloads must resume cleanly (token + charId restore â†’ straight back into world); test WebGL context-loss recovery.
- [DONE][â€”] **Zero-install patching (advantage)** â€” static files behind a CDN; hash-suffixed pck/wasm, cache-busted `index.html`; client/module version pairing per Â§11 binding compatibility.
- [V2][M] **Settlement crowd budget (new, doctrine-driven)** â€” a market square at noon may hold 100â€“300 citizens. Client rule: full `EntityView` only inside the observation radius (~20 tiles), MultiMesh static-frame beyond it, nameplate/HP bars culled past ~12 tiles; subscription already zone-scoped, and far-citizen positions update at event granularity (a handful of row updates per citizen per game-day, Â§1.11), so network and render load are both bounded by the observed radius, not by population.

---

## 1. Settlement & World Simulation (first-class, the heart of the product)

Everything in this section is new architecture. File home: `server/sim/` (new directory, `partial class Module` spread across files â€” zero contention with the existing slice files); data home: `data/rulings/`, `data/srd/`, and procgen-emitted `data/zones/<zone>.population.json`.

### 1.1 NPC-as-character architecture

- [V2][XL] **`citizen` table â€” every NPC is a full statted character.** Replaces the slice's decorative `npc` table (which stays for v1 and migrates). Columns: identity fields (citizen_id pk, name, species, age, portrait), **full SRD stat block reference** (`stat_block_id` into compiled SRD data â€” commoner/guard/noble/priest/acolyte/cultist/thug/scout/spy/veteran/knight/mage etc. from SRD Appendix "Nonplayer Characters" â€” plus per-citizen variation deltas: ability-score adjustments, skill proficiencies, trained tools), current HP/max HP/hit dice, conditions via the existing `condition_row` (holder_kind "citizen"), inventory via `inventory_item` (holder generalized), equipped weapon/armor, gold, spell slots for caster citizens, zone/x/y/dir, `current_activity`, `activity_started_at`, `next_event_at`, home/household id, occupation id, lifestyle tier, faction, attitude defaults. Every rule that applies to a player applies to a citizen: action economy in encounters, skill checks for daily work, exhaustion, death saves, encumbrance.
- [V2][M] **Stat-block compiler** â€” `tools/codegen` ingests the SRD NPC and monster stat blocks (extending `make_srd_json.py` â†’ `data/srd/stat_blocks.json` â†’ `server/GameData.cs` growth); variation is data (procgen emits deltas, Â§1.10), base blocks are compiled-in constants.
- [V2][M] **Unified creature interface** â€” `server/sim/Creature.cs`: one server-side accessor layer over player/citizen/mob rows (GetAC, GetSave, GetSkill, RollAttack, TakeDamage, GetSpeed, conditions) so combat (Â§5), AI (Â§9), crime response (Â§1.7), and the rulings engine resolve against *any* creature identically. This is the single most load-bearing refactor of the doctrine: build it before the settlement layers, and port the slice's player/mob combat onto it.
- [V2][S] **Action economy off the battlefield** â€” outside encounters, citizen activity is minute/hour-granular (Â§1.2); the moment a citizen enters an encounter (attacked, witnesses violence, guard response) it drops into the 6-second round loop with a real initiative roll like everyone else, then returns to schedule.

### 1.2 World clock & time-granularity tiers

- [V2][M] **World clock singleton** â€” `world_clock` table: epoch game-time (in game-seconds), configurable ratio (default **30:1 â€” 48 real minutes = 1 game day**; ratio is data, not code), calendar date, season, sunrise/sunset per season. All simulation timestamps are game-time; conversion helpers in `server/sim/WorldClock.cs`. *Fidelity dividend of 30:1:* a short rest (1 game hour) = 2 real minutes; a long rest (8 game hours) = 16 real minutes; a torch (1 game hour) = 2 real minutes; ritual casting (+10 game minutes) = +20 real seconds. RAW durations become playable MMO durations without house-ruling a single number. **Owner flag:** ratio value itself (30:1 default) is the one knob; everything downstream is RAW.
- [V2][M] **Granularity tiers** â€” round (6 s, encounters only â€” the SRD's combat time unit, already the locked combat model) Â· minute (fine civilian actions: haggling, conversations, lockpicking) Â· hour (work shifts, rests, travel legs) Â· day (lifestyle upkeep, downtime activities, restocking). Every system in Â§1 declares its tier; the event queue (Â§1.11) makes tiers a scheduling choice, not separate engines.
- [V2][S] **Calendar & season table** â€” 12-month year as data (`data/rulings/calendar.json`); season drives daylight hours, weather tables (Â§1.8), crop/fishing yields (Â§1.4), festival events (Â§7).

### 1.3 Schedules, needs, lifestyle economics

- [V2][L] **Daily schedule system** â€” `schedule` data per citizen (emitted by procgen Â§1.10): ordered entries `{start_time, activity, location, params}` in D&D time (e.g., wake 06:00, breakfast, open shop 08:00, market run 12:00, close 18:00, tavern 19:00, sleep 22:00). `server/sim/Schedule.cs` advances citizens entry-to-entry via the event queue; each entry schedules its own next event. Deviations (crime witnessed, festival, sick, jailed) push interrupt activities that resume the schedule after.
- [V2][M] **Needs, RAW-anchored** â€” food/water (1 lb food, 1 gallon water per day; going without â†’ CON-modifier days of grace then **exhaustion per RAW**), sleep (no long rest in 24 h â†’ exhaustion level per the rulings engine's codification of the standard ruling), shelter. Needs are satisfied *through the economy*: citizens buy meals at SRD prices, sleep in their households. A siege that stops food carts starves a town by the book.
- [V2][M] **Lifestyle expenses (SRD table, verbatim)** â€” every citizen and every player pays lifestyle upkeep per game-day: wretched free, squalid 1 sp, poor 2 sp, modest 1 gp, comfortable 2 gp, wealthy 4 gp, aristocratic 10 gp+. Deducted daily by the event queue; failure drops the citizen a tier with knock-on effects (attitude, health via needs, schedule change). For players this is the doctrine's answer to gold faucets â€” the SRD's own sink, applied as written. Lifestyle tier is also procgen's wealth knob (Â§1.10).
- [V2][M] **Exhaustion (RAW, 6 levels)** â€” disadvantage on ability checks â†’ speed halved â†’ disadvantage on attacks/saves â†’ HP max halved â†’ speed 0 â†’ death; applied by needs, forced march (Â§4 travel), and effects; recovery 1 level per long rest with food/drink. Same `condition_row` machinery for players and citizens.

### 1.4 Skill-checked professions affecting stock & prices

- [V2][L] **Profession work as skill checks** â€” each work shift resolves a real server-side check at its scheduled event: tool or skill proficiency vs a DC ladder from the SRD difficulty ladder (Easy 10 / Medium 15 / Hard 20 â€” codified in `data/rulings/dcs.json`). Result quantizes output: smith's day produces goods worth X gp of progress (SRD crafting rate: **5 gp of market value per day, materials at half cost** â€” applied verbatim), fisher's catch, farmer's yield (seasonal modifier), performer's earnings (SRD "Practicing a Profession" downtime rule governs income tier). Failed checks produce less; crits (rulings-engine entry) produce masterwork-tagged stock.
- [V2][M] **Shop stock & prices from output** â€” `business` table per shop: stock rows fed by owner/employee production and by trade-route deliveries (Â§8); prices anchor to SRD equipment price lists, modulated by scarcity (stock below par â†’ price multiplier band, a rulings-engine table, not a floating market). Players buying out all the arrows genuinely empties the fletcher until she makes more.
- [V2][M] **Employment & wages** â€” SRD service prices as written: unskilled labor 2 sp/day, skilled 2 gp/day; citizens hold `job` rows binding them to businesses; wages flow shopâ†’citizenâ†’lifestyle expenseâ†’other shops. The town's money actually circulates; telemetry (Â§11) watches the loop.
- [V2][M] **Player-owned shops & P2P trading (owner directive; staged NOW in `features/trading/`)** â€” players rent real stalls/buildings from the settlement registry (SRD property/downtime "running a business" rules); stock and price their own goods; while the owner is offline a **statted NPC clerk hireling** (SRD skilled wages, 2 gp/day, drawn from the population registry) runs the shop on the world clock â€” rent + wages are recurring RAW sinks; unpaid clerk quits, arrears evict to escrow. Direct trading = escrowed two-party sessions with atomic commit (the #1 dupe surface â€” adversarially reviewed). *Browser:* shop browsing is paginated char-scoped queries, never whole-table subscriptions.
- [V3][M] **Trade routes between settlements** â€” caravan citizens with travel schedules (RAW travel pace, Â§4) moving goods between procgen settlements; interdiction (bandits, weather) is real and resolved by the same rules.

### 1.5 Social simulation â€” codified reaction & attitude rules

- [V2][M] **Attitude model** â€” `attitude` rows (citizen â†’ character, citizen â†’ faction): the classic three-state ladder **hostile / indifferent / friendly** codified with a numeric score behind it; starting attitude from faction + first impression (rulings-engine table); shifted by deeds (gifts, quests, crimes witnessed, public heroics) with authored magnitudes in `data/rulings/attitude_shifts.json`.
- [V2][M] **Social checks by the book** â€” Persuasion/Deception/Intimidation rolled server-side vs DCs set by attitude tier and request size (the DC matrix is `data/rulings/social_dcs.json` â€” the codified DM ruling); results shift attitude and gate dialogue branches; the existing dialogue `check` schema already carries this â€” extend conditions language with `attitude:<tier>`.
- [V2][S] **Memory & gossip** â€” witnessed events (crimes, gifts, rescues) write `memory` rows on witnessing citizens; a daily gossip event diffuses notable memories through a household/tavern adjacency graph (deterministic, seeded), so reputation spreads at the speed of talk, not omnisciently.
- [V2][S] **Barks & visible life** â€” observed citizens surface their current activity and attitude as floating text/dialogue flavor (data-driven bark tables per occupation/attitude); pure client presentation over sim truth.

### 1.6 Mortality, replacement, population registry

- [V2][M] **Citizens die for real** â€” 0 HP â†’ unconscious â†’ **RAW death saves** (same implementation as players, Â§4); stabilized citizens convalesce (SRD recuperating downtime); dead citizens are dead. Corpse rows, funeral events (temple schedule), estate transfer to household heirs (rulings-engine succession table).
- [V2][M] **Resurrection as the persistent-world answer** â€” temples with caster citizens of sufficient level cast Revivify (300 gp diamond, â‰¤1 game-minute), Raise Dead (500 gp diamond, â‰¤10 game-days) per the caster registry (Â§1.9 #9) â€” for citizens whose household/faction can pay, and for players who pay. Components are consumed; diamonds are a real economy sink (Â§8). Important-NPC insurance (a reeve's temple retainer) is authored data, not plot armor.
- [V2][M] **Population registry & replacement** â€” `population_registry` per settlement: births are out of scope for now (owner flag); replacement is migration â€” vacancies (dead smith) generate migration events drawing statted replacements from procgen's population pool with arrival travel time; succession rules fill offices (reeve dies â†’ assembly elects per settlement law data). No respawning innkeepers, ever: the *role* refills, the person does not.
- [V3][M] **Aging & natural death** â€” age advances on the calendar; venerable citizens retire/die by actuarial table (rulings-engine data); keeps multi-season worlds from freezing demographically.

### 1.7 Crime & guard response (real action economy, real morale)

- [V2][L] **Crime detection by the rules** â€” theft = Sleight of Hand vs passive Perception of witnesses in the spatial index's radius (Â§1.11); assault/murder in view = automatic witness memory + alarm; darkness and obscurement genuinely matter (Â§1.8) â€” night burglary is mechanically easier, as it should be. Witness â†’ report event â†’ `crime` row (type, perpetrator if identified, witnesses, settlement).
- [V2][L] **Guard response with real action economy** â€” guards are citizens with guard stat blocks on patrol schedules; response is an interrupt activity: nearest guards (spatial index query) move at real speeds, shout (alerting others via audible radius), and on contact initiate an encounter under full combat rules â€” initiative, opportunity attacks, their own reactions. Guards use nonlethal intent (RAW melee knockout option) for arrestable offenses; **morale** is the codified DM ruling (Â§1.9 #2): a guard reduced below half HP or seeing an ally die makes a morale check and may retreat and escalate rather than fight to death. Escalation ladder: watchman â†’ squad â†’ captain (veteran block) â†’ posted bounty.
- [V2][M] **Law, trial, punishment** â€” per-settlement `law` data (`data/rulings/law.<settlement>.json`): offense â†’ fine/jail-days/exile/execution; jail = schedule override in the jail building (a real place; breakouts are possible by the rules); bounties create hunt events for guard/bounty-hunter citizens. Applies identically to player characters â€” this is the doctrine's griefing answer: **consequences, not prevention** (friendly-fire fireball in the market is mass assault; see risk (b)#5).
- [V2][S] **Alarm & lockdown states** â€” settlement alert level (calm/wary/alarm/lockdown) driven by recent crime rows; modifies schedules (shops shutter, patrols double) and starting attitudes.

### 1.8 Weather, season, light â€” darkness that matters

- [V2][M] **Weather system** â€” per-region `weather` singleton advanced by scheduled events off seasonal tables (`data/rulings/weather.json`): clear/rain/fog/storm/snow; mechanical effects as RAW where the SRD speaks (heavily obscured in fog banks, difficult terrain in deep snow, disadvantage on ranged in strong wind â€” the rulings engine codifies the standard adjudications) and schedule effects (farmers stay in, market thins).
- [V2][M] **Light & vision, RAW** â€” bright/dim/darkness per tile computed from sun state + light sources (torch: bright 20 ft/dim 20 ft, 1 game-hour duration and *consumed*; lamps, candles per SRD); dim light = lightly obscured (disadvantage on sight Perception), darkness = heavily obscured (effectively blinded); **darkvision by species/stat block honored**. Feeds stealth (Â§5), crime (Â§1.7), and monster behavior (Â§9). Lamplighter is a real job with a real schedule.
- [V2][S] **Client rendering of all of it** â€” CanvasModulate day/night tint from the world clock, per-source glow sprites, weather particle overlays with reduced-motion switch. *Browser:* tint is free; cap particles; light *math* is server-side only â€” the client never computes visibility.

### 1.9 The rulings engine â€” enumerated

The codified DM. Every entry = a data file in `data/rulings/` + a deterministic resolver in `server/sim/Rulings/`. Initial catalog (grows under change control â€” every addition is a PR touching this list; see risk (b)#4):

1. **Reaction decisions** â€” when an NPC uses Shield/Counterspell/opportunity attack/readied action: policy tables per stat block; players get standing policies (Â§5). [V2][M]
2. **Morale, flee, surrender** â€” the SRD has no morale rule; codify the classic optional rule: DC 10 Wisdom save on first drop below half HP, leader slain, or outnumbered 3:1 â€” failure = flee/surrender by creature disposition; undead/constructs/oath-bound exempt. [V2][M]
3. **Combat target selection** â€” INT-tiered decision tables (Â§9). [V2][M]
4. **Ad-hoc check DCs** â€” the SRD difficulty ladder (5/10/15/20/25/30) mapped to every codified activity: professions, climbing walls, haggling. [V2][S]
5. **Attitude shifts & social DCs** â€” Â§1.5 tables. [V2][M]
6. **Cover determination** â€” auto-derived half/three-quarters/full cover from tile geometry between attacker and target. [V2][M]
7. **Object AC/HP** â€” breaking doors/locks/chests: material AC (cloth 11/wood 15/stone 17/iron 19) + size HP table; locks pickable with thieves' tools vs authored DC. [V2][S]
8. **Travel encounters** â€” per-region encounter tables with frequency dice per travel leg and watch (Â§4 travel). [V2][M]
9. **Spellcasting services** â€” caster registry per settlement (which citizens can cast what, from their real slots); price = consumed component cost + tiered fee table; availability is real (the priest who died can't cast). [V2][M]
10. **Resurrection willingness** â€” who casts for whom: temple faction attitude + payment + deceased's standing (memory/gossip data). [V2][S]
11. **Non-combat XP awards** â€” traps evaded, quests completed, social "victories": authored CR-equivalent XP values in content data (Â§10). [V2][S]
12. **World pacing** â€” what happens when players ignore a quest: authored world-advances-anyway event scripts (the cult ritual completes on its calendar date). [V3][M]
13. **Stealth state machine** â€” when hiding is possible, when it breaks, when re-rolls happen (Â§5); codifies the game's most DM-adjudicated rule. [V2][M]
14. **Treasure placement** â€” hoard/individual treasure tables by CR band as data, used by procgen and mob loot defaults. [V2][M]
15. **Succession & vacancy** â€” Â§1.6 registry rules. [V2][S]
16. **Law & punishment** â€” Â§1.7 per-settlement tables. [V2][M]
17. **Merchant restocking & scarcity pricing** â€” Â§1.4 band tables. [V2][S]
18. **Readied-action trigger grammar** â€” small condition language ("enemy enters reach", "caster begins casting") shared by player readies and NPC policies. [V2][M]
19. **Improvised actions** â€” curated verb set (shove off ledge, throw sand, overturn table) with codified adjudications; deliberately last â€” the open-ended DM gap. [V3][L]
20. **Falling, suffocation, fire, starvation** â€” already RAW; implement as written, listed here for completeness of the hazard resolver. [V2][S]

Size estimate for the full judgment-gap catalog: **~40â€“60 entries** at maturity (each S/M as data + resolver); the 20 above are the load-bearing core. Tracked as `docs/RULINGS_CATALOG.md` with owner-visible additions, cross-referenced into the rules ledger.

### 1.10 Procedural settlement & population generation

- [WIP-external][XL] **`tools/procgen`** â€” Python, seeded, deterministic; **being built now by a parallel workflow** (wilderness, dungeon, settlement generator families) emitting the *existing* zone-JSON contract (DESIGN.md Zones section). This roadmap treats its contracts as fixed interfaces: zone JSON in, `<zone>.population.json` beside it.
- [V2][M] **`population.json` contract (consumer side)** â€” per citizen: SRD base stat block + variation deltas, household id, occupation + workplace building, daily schedule in D&D time, lifestyle tier, faction, starting attitude class, caster flag/spell list where applicable. `server/sim/PopulationLoader.cs` seeds `citizen`/`household`/`business`/`job`/`schedule` rows at settlement init; `tools/codegen/embed_content.py` grows to validate and embed it.
- [V2][M] **Constraint overlays** â€” authored campaign content (named NPCs, quest buildings, plot flags) rides as overlay JSON the generator honors: Reeve Maera is a *constraint* pinning one generated citizen's identity/stat block/schedule, not a separate code path.
- [V2][S] **Parametric content scaling** â€” settlement size/wealth/culture knobs + seed = a new living town; content volume scales with generator quality and seed count, not authoring man-hours. The v1 slice's 3 hand-authored zones ship as-is and are grandfathered (village gains a population.json retroactively as the first generator test case).

### 1.11 Cost architecture â€” THE WORLD NEVER SLEEPS made affordable (Directive spec)

**(a) Event-driven simulation â€” never per-tick polling.** Every citizen row carries `current_activity` + `next_event_at` (game-time). One scheduled **dispatcher reducer** (`SimDispatch`, ~1 s cadence like `ai_tick`) pulls due events via a B-tree index on `next_event_at` and resolves them in batches; each resolution writes the new activity and schedules the next event. (Per-NPC SpacetimeDB scheduled rows are the alternative; the dispatcher wins because it batches transactions and gives back-pressure control â€” evaluate both at the 10 k bot test, Â§11.) **The arithmetic:** a citizen's day is ~20 events (wake, meals, work start, 2â€“3 work checks, market, social, return, sleep, plus need ticks). At the reference scale of **10,000 NPCs: 10,000 Ã— 20 = 200,000 events per game-day; at the 30:1 clock a game-day is 2,880 real seconds â†’ ~70 events/second** â€” trivial for a database that runs full transactions per reducer call. (Even at a 1:1 clock it's ~2.3/s; at 100,000 NPCs and 30:1 it's ~700/s â€” still plausible, measure it.) Contrast per-tick polling: 10,000 NPCs Ã— 4 checks/s = 40,000 checks/s â€” ~600Ã— worse for zero fidelity gain. Combat drops entities into the 250 ms/6 s round machinery, whose cost scales with *concurrent encounters*, not population. SpacetimeDB scheduled reducers run with zero clients connected â€” the world genuinely never sleeps.

**(b) Spatial index over players and creatures** â€” serves aggro, observation-radius queries, AoE target gathering, guard response, witness detection. Honest evaluation of the two candidates:
- **kd-tree (owner suggestion):** excels at k-nearest-neighbor and skewed density over large sparse extents; but under continuous movement it needs periodic rebuild or tolerates imbalance, and as a module-memory structure it must be rebuilt on every module restart/hot-publish and kept coherent with table state across reducer calls by hand â€” a real correctness surface in SpacetimeDB, where the durable truth is tables.
- **Uniform grid buckets:** O(1) incremental update (an entity only moves buckets when crossing a cell boundary); radius query = scan of the overlapped cells; ideal at the roughly uniform densities of bounded, tile-based zones (1 tile = 5 ft; zones are already finite grids); and â€” decisive â€” it expresses *as the database itself*: quantized `cell_x`, `cell_y` columns (cell = 16Ã—16 tiles) with a composite B-tree index `(zone, cell_x, cell_y)` on player/citizen/mob tables. The index then survives hot-publish and restart for free, every reducer can use it with no warm-up, and there is no shadow-state coherence problem.
- **Recommendation: uniform grid via indexed cell columns.** Adopt [V2][M] as `server/sim/Spatial.cs` (cell math + query helpers). Revisit a module-memory kd-tree (rebuilt in `Init`, updated per move) only if profiling shows nearest-neighbor queries ("closest guard to the scream") dominating â€” and even then, grid + outward ring scan usually suffices at settlement densities. Decision recorded here; measure both under the bot harness before any switch.

**(c) Observation selects FIDELITY, never ACTIVITY.** No NPC is ever frozen; all tiers advance on the same clock through the same schedules and rules with real server-side dice:
- **Tier 0 â€” observed** (player within observation radius, ~20 tiles): full fidelity â€” `move_path` walking, minute-granular micro-actions, barks, and encounters at the full 6 s round loop.
- **Tier 1 â€” same zone, unobserved:** event-granularity â€” position updates only at activity boundaries (teleport-between-waypoints in the data; players never see it), checks still rolled at their scheduled times; NPC-vs-NPC encounters run round math without presentation rows.
- **Tier 2 â€” unobserved zone:** **outcome-equivalent batch resolution** â€” due events resolve in batches with the same dice and same rules; NPC-vs-NPC combat fast-forwards the round loop inside one transaction. Outcomes are distribution-identical to observed play; only the *path texture* (exact tile trajectories) is coarser.
- **Transitions:** a player arriving mid-activity finds citizens at schedule-implied positions (deterministic placement from activity + elapsed time); promotion Tier 2â†’0 is instant because the sim state was always current.
- **[OWNER SIGN-OFF REQUIRED]:** the policy is outcome-equivalence, not path-equivalence â€” e.g., an unobserved guard-vs-thief chase resolves as opposed Athletics checks per round of the same chase rules rather than tile-by-tile pursuit. This is the doctrine-compliant reading of "no NPC ever frozen"; the alternative (full path simulation for all 10,000, always) multiplies cost for no rules-visible difference. Sign-off recorded here when given.

---

## 2. Client / Rendering (re-audited under the doctrine)

- [WIP][L] **Isometric TileMap zone rendering** â€” ground/props/overlay TileMapLayers from `client/data/zones/*.json`, 128Ã—64 diamonds, Y-sorted props. No World scene exists in `client/src` yet â€” this is open work, not polish. *Browser:* build layers incrementally across frames; procgen zones may exceed 48Ã—48 â€” chunking is mandatory, not optional.
- [WIP][L] **EntityView sprites** â€” AnimatedSprite2D from generated 8-direction sheets, idle/walk/attack/cast/death @10 fps, HP bar, nameplate, 4 Hzâ†’60 fps interpolation. Extends to citizens (occupation-varied bodies from the sprite pipeline).
- [WIP][M] **Tile/prop art set** â€” Blender-rendered per DESIGN legend (`tools/blender` is empty â€” generation scripts are the open gap; `assets/icons` [DONE], `assets/keyart` [DONE], portraits/models/sfx dirs empty).
- [V2][M] **Crowd LOD (CrowdRenderer)** â€” MultiMesh static frames beyond the observation radius; doctrine-critical for settlements (Â§0 crowd budget). Budget test at 300 citizens on a mid laptop.
- [WIP][S] **Camera** â€” smooth-follow, edge clamp, 2â€“3 fixed zoom steps.
- [V2][M] **Day/night/weather rendering** â€” consumes Â§1.8: CanvasModulate tint from world clock, glow sprites for light sources (which are real, consumed items), weather overlays. *Browser:* no per-tile light shaders; the server owns visibility math.
- [V2][M] **Activity presentation** â€” observed citizens visibly do their schedule: work-loop animations at workplaces, carried-goods props, market stalls populating; reads sim rows, invents nothing.
- [WIP][M] **Spell/combat VFX** â€” projectiles, impacts, floating text from `combat_event`. AoE ground templates (sphere/cone/line) now render *true SRD shapes* including over allies â€” the reticle warns, it does not prevent. *Browser:* pool everything; cap simultaneous systems (~16).
- [V2][M] **Fog of war / exploration reveal** â€” per-character explored bitmask (server, bitpacked); exploration-only fog (live LoS fog stays rejected â€” Â§0 economics unchanged by doctrine).
- [WIP][S] **Minimap** â€” tile data + entity dots; [V2][S] explored-mask integration; guards/known-citizens colored by attitude.
- [V2][M] **World map screen** â€” regional map (keyart pipeline) with zone nodes and *travel-time* labels (RAW pace, Â§4) instead of fast-travel buttons.
- [WIP][S] **Zone loading/transition** â€” portal â†’ fade â†’ resubscribe â†’ rebuild; frees previous atlases (Â§0).
- [WIP][S] **Performance/debug HUD** â€” FPS, entity count, drain-queue depth, wasm heap; add sim-event lag (server `next_event_at` overdue depth) once Â§1 lands.
- [V3][L] **Painted zone backdrops** â€” keyart-pipeline large backgrounds; only after asset streaming.

## 3. GUI

- [WIP][M] **Main menu / intro cinematic** â€” `MainMenu.gd` + `IntroCinematic.gd` per DESIGN.md (neither file exists yet; Login.gd/Credits.gd/Theme.gd do); keyart [DONE] feeds both; doubles as iOS audio unlock.
- [WIP][M] **Character creation** â€” species/class/standard array/name (`CharCreate.gd` exists). [V2][M] backgrounds (SRD), starting equipment per class table (RAW packs), point-buy toggle (SRD variant â€” both are rules-legal). [V3][M] portrait picker.
- [WIP][S] **Character select** â€” exists; [V2][S] class/level/zone display + delete-with-confirm.
- [WIP][M] **HUD** â€” HP, target frame, cast bar, condition icons ([DONE] in `assets/icons/`), zone toast. [V2][S] add: death-save pips (3/3, doctrine-critical), exhaustion level, light-level indicator, world-clock/date widget.
- [WIP][M] **Action bar** â€” exists (`ActionBar.gd`). [V2][M] quickbar customization with server-persisted layout (`quickbar_slot` table). [V2][M] **stance/intent controls**: Dodge/Disengage/Dash/Ready as real actions, nonlethal-intent toggle (feeds arrest rules Â§1.7).
- [V2][M] **Reaction policy panel (new, doctrine-critical)** â€” the player-configured standing policies that *are* the reaction economy (Â§5): "Opportunity attacks: always/never/not-vs-allies", "Shield: when hit and slot â‰¥1 remains", "Counterspell: vs spells â‰¥ level N within 60 ft", "Ready: <trigger grammar> â†’ <action>". Server-validated data rows (`reaction_policy` table).
- [V2][M] **Spellbook + preparation UI** â€” known vs prepared, prepare-on-long-rest, ritual tags, **component requirements displayed** (V/S/M with gp costs; Â§4).
- [V2][M] **Character sheet / paper doll** â€” full abilities/skills/saves/AC/attack breakdowns, equip slots (`equipment_slot` table), **carry weight vs STRÃ—15**, hit dice, death-save history, XP to next level (RAW table Â§10), lifestyle tier and daily upkeep.
- [WIP][S] **Inventory** â€” flat list (slice). [V2][M] grid + tooltips with full SRD stat blocks + **weight column and encumbrance readout** (RAW capacity; variant encumbrance as an SRD-legal server option flag). [V2][S] component pouch/focus slot handling.
- [V2][M] **Loot & trade windows** â€” loot per SRD treasure results (Â§1.9 #14); trade = escrowed two-pane (`trade_session` table); items freely tradable (binding removed per ledger). Player-shop manage/browse panels per Â§1.4 (staged in `features/trading/`).
- [WIP][S] **Journal / quest tracker** â€” exists per slice; [V2][M] chapter grouping, archive, pinned tracker; quest lines display *world-clock deadlines* where the pacing engine (Â§1.9 #12) sets them.
- [WIP][M] **Dialogue UI** â€” `dialogue_view` renderer + portraits; [V2][S] history scrollback; attitude tier shown on the NPC nameplate/portrait frame.
- [V2][S] **Rest UI** â€” short (1 game hour) / long (8 game hours) rest initiation with real-time equivalents shown, hit-dice spend picker, interruption per RAW; campfire vignette.
- [V2][M] **Level-up wizard** â€” HP roll-or-average, features, subclass at 3, spells, ASI; triggered by RAW XP thresholds (Â§10); server validates every choice.
- [V2][M] **Party frames + target-of-target** â€” up to 6, HP/conditions/death-save pips; ToT stays useful for reading *any* creature's target (Â§9 decision-making is legible).
- [WIP][M] **Combat log with full roll math** â€” `CombatLog.gd` exists; every `combat_event.text` carries complete breakdowns ("d20 17 +5 = 22 vs AC 15, hit, 7 slashing"); [V2][S] initiative-order round header lines ("Round 3 â€” Wren 21, wight 14, you 9"), death-save lines, filter tabs.
- [WIP][S] **Chat panel** â€” zone + `/g` (exists). [V2][M] tabs/whisper/party channels, unread badges. *Browser:* ~500-line scrollback cap.
- [V2][S] **Emotes & bios** â€” chat verbs + 500-char server-stored bios.
- [WIP][S] **Settings** â€” volume (exists); [V2][M] video/keybinds/accessibility/UI scale.
- [V2][M] **Tooltip service** â€” unified: items, spells (full SRD text), conditions, rules terms ("what is heavily obscured?" â€” the teaching surface for Â§13).
- [V2][S] **Drag-drop framework** â€” one shared implementation (quickbar/inventory/trade/equip).
- [V2][M] **Death & dying UI (replaces respawn overlay)** â€” unconscious screen with live death-save rolls, stabilization status, nearby-help indicator; on death: corpse/afterlife screen with resurrection options, costs, and time windows (Revivify 1 game-minute countdown is a real, visible countdown). The slice's downed/respawn overlay [WIP] ships v1 and is replaced.
- [V3][M] **Mail UI** Â· [V3][L] **Auction house UI** â€” unchanged infra items; auction remains paginated-reducer-only (*Browser:* never subscribe the listings table).
- [WIP][S] **Credits & legal screen** â€” `Credits.gd` exists; must render CC-BY attribution (Â§14).

## 4. Character Systems (RAW)

- [WIP][M] **4 slice classes, levels 1â€“3** â€” Fighter/Rogue/Wizard/Cleric, SRD math in `server/Rules.cs`/`GameData.cs` (both live).
- [V2][L] **Levels 1â€“5, subclasses at 3** â€” Champion, Thief, Evoker, Life Domain; Extra Attack at 5; spell tiers 1â€“3; per-level feature tables data-driven.
- [V2][L] **+4 classes (Ranger, Paladin, Bard, Druid)** â€” Wild Shape = stat-block swap onto the unified creature interface (Â§1.1 makes this natural).
- [V3][XL] **All 12 SRD classes** â€” Barbarian, Monk, Sorcerer, Warlock (pact slots, ki as distinct resources).
- [V3][XL] **Levels 6â€“20** â€” spell tiers 4â€“9. Doctrine change: high-tier spells implement **as written** wherever the server can express them (Teleport, Plane Shift = travel between generated zones; Simulacrum = a real second statted creature); only spells that are *technically impossible* (Wish's open clause) get rulings-engine option menus â€” `mmo_variant` flags are replaced by `rulings_menu` data.
- [V2][M] **Backgrounds** â€” SRD backgrounds: 2 skills, tool/language, equipment, feature codified as a mechanical hook where possible (Acolyte's temple access = attitude bonus at temples via Â§1.5).
- [V2][M] **Skills & tools, full list** â€” all 18 skills + tool proficiencies usable in dialogue, world interactions, professions (Â§1.4), and downtime; passive Perception live everywhere (Â§1.7 detection).
- [V2][S] **Ability Score Improvements** â€” at RAW levels in the level-up wizard. [V3][M] **Feats** from SRD 5.2.1 (CC-BY) as ASI alternative.
- [V3][L] **Multiclassing** â€” RAW prerequisites, merged slot table.
- [V2][M] **Concentration (RAW)** â€” one effect; CON save DC 10-or-half-damage on damage; drop on incapacitated/dead/second concentration.
- [V2][M] **Full condition set** â€” all 15 SRD conditions + exhaustion 1â€“6 as `condition_row.kind` with exact rules effects (icons [DONE]).
- [V2][M] **Resting, RAW (reversal)** â€” short rest = 1 game hour (2 real min at 30:1), spend Hit Dice; long rest = 8 game hours (16 real min), â‰¥6 sleeping, all HP + half hit dice, one per 24 game-hours, broken by 1 game-hour of strenuous activity; **anywhere** â€” wilderness rests roll watch-by-watch encounter checks from regional tables (Â§1.9 #8). Inns sell *comfort* (lifestyle, roleplay, safety), not exclusive rest rights.
- [V2][M] **Spell components (new, RAW)** â€” V blocked by silence, S requires a free hand, M requires component pouch/focus; **costed materials must be owned and are consumed when the spell says so** (Revivify's 300 gp diamond, Identify's pearl). Inventory checks in `CastSpell`; component items in the SRD price data (Â§8).
- [V2][L] **Death rules, RAW (reversal, doctrine-critical)** â€” 0 HP â†’ unconscious (not "downed"): death saving throws each round on the character's initiative anchor (flat d20 DC 10; 3 fails = dead; nat 1 = 2 fails; nat 20 = up with 1 HP; damage at 0 = 1 fail, 2 on crit; instant death on damage â‰¥ max HP overflow). Stabilization: DC 10 Wisdom (Medicine), healer's kit auto-stabilize, Spare the Dying. Stable = unconscious, 1 HP after 1d4 game-hours. **Dead = dead until resurrection magic** (Â§1.6 services; components consumed; time windows enforced on the world clock). **[OWNER DECISION FLAGGED]: permadeath-if-unraised** â€” full RAW means a character not raised within the Raise Dead window (10 game-days) is gone barring Resurrection (1,000 gp) or True Resurrection (25,000 gp); the soft-floor alternative keeps True Resurrection eventually reachable for any character at brutal cost. Decision recorded here when made; risk analysis in (b)#3. Slice keeps its 15 s downed rule until this lands.
- [V2][M] **Encumbrance (RAW)** â€” carrying capacity STRÃ—15 lb, push/drag/lift Ã—30, enforced on pickup and movement; SRD *variant* encumbrance (STRÃ—5/Ã—10 speed penalties) as a server option flag â€” both are rules-legal, default = standard.
- [V2][M] **Travel (RAW, new)** â€” overland pace fast/normal/slow (30/24/18 mi per game-day) between zones on the world map; forced march CON saves (DC 10 + hours past 8) â†’ exhaustion; Survival navigation checks in trackless regions; foraging; watches with encounter rolls; mounts per SRD speeds/prices. This *is* the fast-travel system: travel is simulated, arrival is scheduled on the world clock, and the player can play the encounters or auto-resolve them (auto-resolve uses Tier-2 batch resolution â€” same dice).
- [V2][M] **Downtime activities (RAW, new)** â€” between adventures at day granularity: Crafting (5 gp progress/day, half-cost materials), Practicing a Profession (lifestyle maintenance per SRD), Recuperating, Researching, Training (250 days, 1 gp/day for languages/tools). Same machinery citizens use (Â§1.4) â€” players are citizens with agency.
- [V3][S] **Alignment** â€” bio field only; the SRD gives it almost no mechanics; where a rule references it (protection spells), implement that rule only.

## 5. Combat Depth (real-time rounds, initiative inside them)

- [WIP][M] **Core auto-attack round loop** â€” d20+bonus vs AC per 6 s, crits, advantage, damage types (`server/Combat.cs`, `Tick.cs` â€” both live).
- [WIP][M] **Cast wind-up + resolution** â€” wind-up, slot validation at finish, attack/save/auto/heal/buff (slice).
- [V2][L] **Initiative order within the 6-second round (evaluated â†’ ADOPT)** â€” the SRD's round is 6 seconds and *turns are ordered by initiative inside it*; the current model (personal 6 s cooldowns) loses "start/end of turn" anchors, which RAW needs for death saves, save-ends conditions, readied actions, and legendary actions. Design: on encounter start, all participants roll initiative (d20+DEX, RAW tiebreaks); each combatant's **turn anchor** = round_start + rank Ã— (6 s Ã· N participants); at the anchor, the entity's queued intents resolve as its turn (action + bonus action + reaction reset + movement budget = speed per round â€” the existing `speed_tiles_per_s` already equals the RAW budget: 30 ft/6 s), start/end-of-turn effects fire, death saves roll. Movement input stays continuous (locked real-time decision) but consumes the turn's budget. Cost: an action clicked mid-round waits â‰¤6 s for its anchor â€” the same latency the cooldown model already imposed, now rules-shaped. *Browser:* strictly friendlier to WebSocket jitter â€” intents queue, the server's clock resolves. **Full-RAW variant (movement only on your turn) rejected as conflicting with the locked real-time decision; recorded as a deviation with rationale.**
- [V2][M] **Reaction economy, real (reversal)** â€” one reaction per round, reset at your anchor. Opportunity attacks per RAW (leaving reach without Disengage/teleport). **Readied actions**: trigger grammar (Â§1.9 #18) + held action, released when the trigger fires between anchors. **Shield/Counterspell as standing policies** (Â§3 panel): the policy *is* the player's reaction decision made in advance â€” the codified answer to "the player would decide in the moment", latency-proof by design. NPCs use per-stat-block policy tables.
- [V2][M] **Concentration breaks** â€” per Â§4; visible tether VFX.
- [V2][M] **AoE templates + friendly fire (reversal)** â€” true SRD shapes (sphere/cone/line/cylinder) in tile units, server-validated; **all creatures in the template are affected â€” allies, bystanders, shopkeepers**. Evoker's Sculpt Spells implements as written and matters again. Bystander harm engages crime/attitude systems (Â§1.7, Â§1.5). No open-world/instance policy split â€” one rule everywhere.
- [V2][M] **CC per RAW (reversal)** â€” diminishing returns deleted; conditions run their written durations/saves; chain-lock is legal. The world's own counters: Legendary Resistance on legendary stat blocks (SRD), repeat-save-ends conditions, Counterspell/Dispel Magic from enemy casters (Â§9 uses them), and morale (Â§1.9 #2) â€” degenerate lock-downs are answered by the simulation, not a modifier.
- [V2][M] **Saving-throw auras & recurring effects** â€” `aura_row` (holder, radius, save, effect, period) evaluated on turn anchors (Spirit-Guardians-shaped; foundation for bosses).
- [V2][M] **Line of sight & cover** â€” Bresenham over the wall grid; cover auto-derivation (Â§1.9 #6): +2 AC/DEX-save half, +5 three-quarters, full = untargetable; obscurement from light/weather (Â§1.8) applies as written.
- [V2][L] **Stealth/detection (RAW via Â§1.9 #13)** â€” Stealth vs passive Perception, light and obscurement honored, hidden = unseen attacker rules (advantage; attack reveals). Deviation removed: other players do *not* get a fairness silhouette â€” hidden is hidden from creatures who fail to notice, players included.
- [V2][M] **Mounted & underwater rules** â€” SRD chapters exist; data-cheap once the creature interface is in: mount speeds, underwater weapon restrictions, swim speeds. Fen and mere content wants them.
- [V2][M] **Boss encounters = SRD stat blocks run honestly** â€” legendary actions (between turn anchors), legendary resistance, lair actions on initiative 20 (anchor 0): the vocabulary is the SRD's own, data-driven from stat blocks â€” replaces the invented "boss script atoms". Drowned Warden upgrades to a wight run *by the book* plus authored lair data.
- [V3][M] **Grapple/shove/improvised** â€” contested checks per RAW; improvised verbs via Â§1.9 #19.

## 6. Party & Social

- [V2][L] **Grouping** â€” party up to 6: invite/accept/leave/kick, leader, `party`/`party_member` tables, party chat, minimap positions; shared XP per RAW (Â§10) and quest credit within zone + radius.
- [V2][M] **Loot: property, not loot rules** â€” a slain creature's possessions and rolled treasure (Â§1.9 #14) drop as claimable world property; party default = free-for-all with an optional leader-set rotation (a table convention, not a server rule). Personal-loot instancing removed as a house rule.
- [V2][S] **XP sharing (RAW)** â€” encounter XP divided equally among participants; participation = contributed action in the encounter.
- [V3][L] **Guilds** â€” create (gold sink), roster, ranks, MOTD, chat; [V3+] bank with audited withdrawals.
- [V2][M] **Friends/ignore** â€” online/zone status; ignore enforced server-side.
- [V3][M] **LFG listing board** â€” role-tagged listing + whisper; no teleport matchmaking (travel is real, Â§4).
- [V2][M] **Trading** â€” escrowed window (staged in `features/trading/` now); every trade audited (GM/telemetry).
- [V3][M] **Mail** â€” async transfer with delivery *time computed from actual courier travel* (Â§1.4 trade routes) â€” the doctrine version of the RMT-friction delay.
- [V2][S] **Emotes/RP tools** â€” verbs, `/e`, bios.
- [DONE][â€”] **Voice: none (decision)** â€” text-only, permanently.
- [V2][M] **Moderation (player-facing)** â€” /report + context capture, mutes, profanity filter, rate limits. Doctrine-neutral: keep in full.
- [V2][â€”] **PvP = the simulation (flagged)** â€” attacking a player is attacking a creature: same initiative, same death saves, same crime consequences in lawful areas (Â§1.7); lawless wilderness is lawless. **[OWNER SIGN-OFF REQUIRED]** before enabling outside duels â€” risk (b)#5; the doctrine-consistent mitigations are law severity data, resurrection economics, and bounty hunters, never a PvP flag.

## 7. World & Content (procgen-first)

- [WIP][M] **Slice zones Ã—3** â€” village/fen/crypt JSONs [DONE]; server embedding [DONE] (`Content.g.cs`); client rendering [WIP]. Ship as-is, grandfathered.
- [WIP-external][XL] **`tools/procgen` generator families** â€” wilderness, dungeon, settlement (with `population.json`), seeded/deterministic, emitting the existing zone-JSON contract; built by the parallel workflow. Consumer contracts: Â§1.10.
- [V2][M] **Constraint-overlay format** â€” authored quest/campaign content as overlays the generators honor; `tools/codegen` validates overlay-vs-output.
- [V2][L] **World assembly** â€” region graph (adjoining zones, travel distances in miles for Â§4), seeded generation of the launch region: the village (retrofit population) + N wilderness + M dungeon + a second full settlement as the procgen proof. Content scale is parametric â€” zone count is a knob, not a milestone.
- [V2][M] **Zone lifecycle at scale** â€” zones instantiate from seed on first relevance and persist deltas only (kills, looted props, construction); Tier-2 simulation (Â§1.11c) keeps inhabited zones alive unobserved; uninhabited wilderness needs no events until entered (nothing scheduled = zero cost â€” the honest "sleep" that isn't an NPC freeze).
- [V3][L] **Dungeon instancing decision (unchanged infra)** â€” instances are **logical**: `instance_id` columns, party-keyed spawn sets, subscription filters â€” one module simulates all. Module-per-shard with character transfer stays the [V3][XL] escape hatch. Doctrine note: dungeons are *places*, so default remains open-world shared; logical instancing reserved for story-critical set pieces.
- [V3][M] **Dynamic events â†’ world pacing engine** â€” Â§1.9 #12 scripts: the world advances on its calendar whether or not players engage (the cult ritual fires; the village burns or doesn't) â€” consequences as physics.
- [V2][M] **Day/night & seasons** â€” from Â§1.2/Â§1.8; night-gated content becomes real: shadows spawn per spawn tables' time windows.
- [V3][L] **Player housing** â€” buy real buildings from the settlement registry (Â§1.6 estates); ownership rows, furniture from prop catalog; a gold sink that is also a simulation feature.
- [V2][L] **Campaign/module pipeline** â€” the data trio + overlays + validators = the module format; schema docs, `validate_all` CLI, hot-reload into a dev module; second campaign as proof. [V3][XL] **Community modules** â€” sandboxed namespaces, data-only scripting, quotas, review pipeline, separate databases per community shard; the NWN-shaped differentiator.

## 8. Economy & Items (SRD-anchored)

- [WIP][S] **Slice item catalog** â€” `data/items.json` [DONE]; server grant/use [WIP].
- [V2][L] **SRD equipment catalog at SRD prices, verbatim** â€” all weapons (~37), armor (~13), gear, tools, **spell components with costs**, light sources, food/drink/lodging, mounts/vehicles, trade goods. Prices are not tuning inputs â€” they are the rules. `data/srd/equipment.json`.
- [V2][M] **Magic items, Common/Uncommon** â€” SRD items as written; [V3][L] Rare+ with **attunement (max 3)** per RAW. Items whose text demands DM adjudication get rulings-menu data, not exclusion.
- [V2][M] **Vendors = businesses (Â§1.4)** â€” stock from production + trade, prices = SRD Ã— scarcity band; buyback = the shop reselling what it bought; haggling = a real Persuasion interaction (Â§1.5) with codified DC/price steps.
- [V2][M] **Player-owned shops (owner directive)** â€” see Â§1.4; staged in `features/trading/` with the offline NPC-clerk model, rent + wages as RAW sinks, sales ledger, escrow eviction.
- [V2][M] **Money & sinks per RAW** â€” faucets: treasure per CR tables, wages for downtime work; sinks: **lifestyle expenses (daily, everyone)**, **resurrection components (the big one)**, spellcasting services, component consumption, training, mounts, property, shop rent + clerk wages. Removed house-rule sinks: durability fees, binding, listing deposits. Telemetry instruments faucets/sinks from day one (Â§11).
- [V2][S] **Currency per SRD** â€” cp/sp/ep/gp/pp with real weights (50 coins/lb counts against encumbrance).
- [V3][L] **Crafting per RAW downtime** â€” 5 gp/day progress, half-cost materials, tool proficiency required; the same system citizens use (Â§1.4).
- [V3][XL] **Auction house** â€” reframed as a **brokerage business** in a hub settlement, regional not global. *Browser:* paginated reducer queries only.
- [V2][M] **Bank/stash** â€” strongbox rental at the village hall (a business with a ledger); [V3][S] letter-of-credit shared access.
- [WIP][S] **Loot tables** â€” shapes [DONE]; [V2][M] regenerate defaults from CR treasure tables so every stat block gets rules-derived drops.

## 9. Monster & NPC Combat AI (rules-faithful decision-making â€” threat tables replaced)

- [WIP][S] **Slice mob AI** â€” idleâ†’aggroâ†’chaseâ†’attackâ†’leash (`MobAi.cs` â€” skeletal). Ships for v1; superseded.
- [V2][L] **Creature decision engine (the threat-table replacement)** â€” per-turn-anchor decisions from the stat block: **INT â‰¤ 3**: attack nearest/last-attacker, flee at morale failure, Pack Tactics positioning; **INT 4â€“7**: strongest attack, focus wounded, retreat on morale break; **INT 8â€“11**: action-economy literate â€” Multiattack, Dodge when cornered, Disengage to reposition, focus casters, use cover; **INT 12+**: policy-table tactics â€” Counterspell priorities, focus-fire, terrain, parley when losing. Tables in `data/rulings/combat_ai.json`; resolver on the unified creature interface â€” guards, citizens, monsters, and summons all think with the same engine.
- [V2][M] **Stat-block fidelity** â€” Multiattack as written, recharge on real 5â€“6 rolls, spellcasting traits cast real spells from real slots, legendary/lair actions (Â§5), resistances/immunities exact, condition immunities exact.
- [V2][M] **Morale (Â§1.9 #2)** â€” flee/surrender/rout; undead and constructs fight to destruction; surrendered creatures are prisoners (crime system covers player brutality).
- [V2][L] **SRD bestiary, CR 0â€“4 (~120 blocks)** â€” PI-exclusion list guards the pipeline. [V3][L] CR 5â€“10; [V3][L] CR 11+ â€” gated on level range and sprite generation, not invented balance bands.
- [V2][M] **Ecology & spawning** â€” wilderness creatures get territory + schedule rows like citizens (a wolf pack dens, hunts at dusk, on the same event queue); "respawn" replaced by **population + migration** for intelligent creatures and breeding-rate abstractions for beasts; dungeon undead don't repopulate without an authored necromantic cause. Slice respawn timers grandfathered until this lands.
- [WIP][S] **Leashing** â€” slice mechanic; superseded by real pursuit decisions (morale/territory rules, not an invisible tether).

## 10. Progression (RAW XP)

- [WIP][S] **Milestone leveling (slice)** â€” ships v1, then replaced.
- [V2][M] **XP from CR, RAW (reversal â€” primary progression)** â€” every defeated creature awards its CR's XP, divided equally among participants; thresholds per the SRD table (300 â†’ L2 â€¦ 355,000 â†’ L20). "Defeated" includes routed/surrendered. `xp` column + award path; level-up wizard at thresholds.
- [V2][S] **Non-combat XP** â€” authored awards via Â§1.9 #11: quests, traps/hazards, social victories â€” CR-equivalent guidance documented.
- [V2][S] **No XP loss, no cap beyond content** â€” death costs resurrection components and time; level cap = highest implemented range, never a tuning cap.
- [V3][M] **Renown/faction standing** â€” attitude machinery at organizational scale: titles, access, services.
- [V3][M] **Achievements & titles** [NO-PROVENANCE â€” doctrine-neutral infra, owner-approved by prior use] Â· [V3][M] **Cosmetics rail** (monetization lane, zero rules impact) Â· [V3][S] **Leaderboards**.
- **Endgame under the doctrine** â€” not a treadmill: high-CR regions, faction politics in living settlements, property/downtime depth, world-pacing consequences. Dungeon lockouts/affix tiers deleted (pure balance constructs, no provenance).

## 11. Server / Infra & Live-Ops (doctrine-neutral â€” kept, with sim additions)

- [WIP][S] **Accounts/auth v1** â€” identity + localStorage token. [V2][M] **OIDC upgrade** â€” not optional for real launch (token loss = character loss).
- [WIP][S] **Characters per account** â€” multiple; [V2][S] cap 6 + delete grace.
- [V2][M] **Persistence/backups** â€” Maincloud backups + scheduled logical exports to owned storage.
- [V2][L] **Schema-upgrade strategy (critical)** â€” additive-only evolution, version column, lazy migration, staging rehearsal, written exportâ†’transformâ†’import runbook. The Â§1 kernel adds many tables â€” additive discipline makes that cheap; *editing* `player` stays the hazard.
- [V2][XL] **SERVER CELLS (owner directive â€” promoted from V3)** â€” the world runs on multiple server cells, never one assumed process: cell = zone set on its own SpacetimeDB database; atlas/directory module (zoneâ†’cellâ†’URI, character locator); idempotent character-transfer protocol at zone portals; bridge-side transparent handoff; boundaries portal-aligned only (no cross-cell combat â€” no cross-database transactions exist). **Dynamic zoneâ†’cell reassignment by load; the partitioning algorithm (owner intends kd-tree) is settled in a dedicated owner design review â€” prior kd notes scrapped per owner.** Dev story: all cells on one local standalone instance; production spreads across nodes. Staged now in `features/cells/`. Single-module ceiling measurement (bot harness) still runs first â€” it tells us when cells must activate, not whether to build them.
- [V2][S] **Anti-cheat audit cadence** â€” reducer checklist each milestone; new sim reducers join it.
- [V2][M] **Rate limiting** â€” per-identity token buckets in-module.
- [V2][M] **GM tools** â€” teleport, spawn, quest-step, mute/kick/ban, broadcast; **sim additions:** possess-citizen, attitude/law inspectors, world-clock step/pause (staging only), **rulings-trace viewer** (why did the guard do that? â€” dump decision inputs). [V3][M] web GM dashboard.
- [V2][M] **Telemetry** â€” counters (reducer rates, dispatch durations, event-queue overdue lag, encounters/hour, gold faucet/sink flows, deaths by cause, resurrection spend) + client beacons.
- [V2][M] **Crash/error reporting** â€” browser beacons; server log alert poller.
- [WIP][S] **Deploy pipeline** â€” `tools/deploy/*` [DONE]; [V2][M] CI: validate â†’ build â†’ staging publish â†’ bot smoke â†’ promote.
- [V2][M] **CDN/versioning + binding compatibility** â€” hashed assets; `protocol_version` config table; refuse-mismatch prompt.
- [V2][S] **Cost model** â€” Maincloud launch; self-host checkpoint at sustained-load data.
- [V2][L] **Load testing (elevated)** â€” Node bot harness + **sim-scale harness**: 10,000 citizens in staging, compressed game-days, dispatcher throughput/lag charts â€” the (b)#1 measurement. Blocks nothing; informs everything.
- [V2][M] **Chat moderation/filters** Â· [V2][S] **GDPR/privacy** (export/delete reducers) Â· [V2][S] **ToS/EULA** (versioned accept).

## 12. Audio

- [WIP][M] **Music per zone/state** â€” specced; `Audio.gd` exists; composer scripts are the open gap.
- [V2][M] **Ambience beds** â€” per-zone + **time-of-day variants from the world clock** â€” the sim is audible.
- [V2][M] **Positional SFX** â€” pan/attenuation; ~16-voice cap. Settlement soundscapes key off *actual citizen activities* in earshot.
- [WIP][S] **SFX set (~27)** Â· [WIP][S] **UI sounds** Â· [WIP][S] **Volume/ducking** â€” all init behind first gesture (Â§0 iOS).
- [V3][M] **Adaptive layers** â€” crossfade covers 80 % first.
- [V3][S] **Barks: text-only** â€” Â§1.5 tables; no voice ever.

## 13. Onboarding & Accessibility (teaching REAL 5e)

- [V2][M] **Teach the actual rules** â€” the tutorial teaches 5e as written because the game *is* 5e as written: popovers on first miss (AC), first save, first slot, first condition, **first death save**, first exhaustion, first lifestyle bill; every popover cites the real rule term. A player who learns Hollowmere has learned tabletop 5e â€” a feature, market it.
- [V2][M] **"Why?" expander everywhere** â€” combat rolls, guard responses, price changes, attitude shifts expose their rule/ruling inputs on demand. The simulation is legible or it is not trusted.
- [V2][S] **Contextual help** â€” `?` mode â†’ rule card (tooltip service Â§3).
- [WIP][S] **Players' Guide** â€” [DONE]; regenerate under the doctrine: deviations chapter shrinks to the recorded, justified list.
- [V2][S] **Colorblind palettes** Â· [V2][S] **Font scaling** Â· [V2][S] **Remappable keys** (*Browser:* never bind Ctrl+W) Â· [V2][S] **Reduced motion**.
- [V2][S] **Screen-reader honesty** â€” ARIA live-region mirror for chat/log/dialogue; full canvas AT support out of scope; say so publicly.

## 14. Legal & Business

- [DONE][S] **SRD CC-BY attribution** â€” LEGAL.md exists; [WIP][S] verify in-game Credits + hosting footer.
- [V2][S] **Trademark hygiene** â€” PI blocklist in `tools/codegen` validators *and the procgen pipeline*; "Hollowmere" mark search.
- [V2][S] **SRD version posture** â€” 5.1 base; adopt 5.2.1 content deliberately per system; `docs/SRD_VERSIONS.md` ledger, cross-referenced to the rules ledger.
- [V3][M] **Monetization (goodwill-compatible)** â€” cosmetics + supporter tier + module box-price; hard nos: pay-for-power, loot boxes, energy timers. **New hard no under the doctrine: selling anything the rules price in gold** (resurrections, components).
- [V2][S] **Age rating posture** â€” PEGI 12/ESRB T; filters default on.
- [WIP][S] **License inventory** â€” [partially DONE; keep current].

---

## (a) V2 Cut â€” dependency-ordered, led by the settlement-simulation kernel

The kernel is items 1â€“8; nothing doctrine-flavored ships before it because everything consumes it.

1. **World clock + calendar** (Â§1.2) â€” every other system timestamps against it.
2. **Unified creature interface + citizen stat blocks** (Â§1.1) â€” the load-bearing refactor.
3. **Event queue + SimDispatch dispatcher** (Â§1.11a) â€” the world-never-sleeps engine.
4. **Schedules + needs + lifestyle upkeep** (Â§1.3) â€” first visible living settlement.
5. **Rulings engine v1** (Â§1.9 entries 1â€“8, 13, 16) â€” the codified DM's core.
6. **Spatial index (grid cells + composite B-tree)** (Â§1.11b).
7. **Death saves + resurrection services** (Â§4, Â§1.6) â€” owner decision filed alongside.
8. **Settlement/population generator integration** (Â§1.10) â€” retrofit the village; stand up settlement #2 from seed.
9. **Initiative anchors + real reaction economy + standing policies** (Â§5, Â§3).
10. **Friendly-fire AoE templates + LoS/cover** (Â§5).
11. **Full condition set + concentration + exhaustion** (Â§4).
12. **Creature decision engine + stat-block fidelity + morale** (Â§9).
13. **XP from CR + RAW thresholds + level-up wizard** (Â§10, Â§3).
14. **SRD equipment/price/component data + professionsâ†’stockâ†’prices loop** (Â§8, Â§1.4).
15. **Player shops + P2P trading merge** (Â§1.4, Â§8 â€” staged in `features/trading/` now).
16. **Crime & guard response + law tables** (Â§1.7).
17. **RAW resting + travel + downtime** (Â§4).
18. **Rules ledger live** (docs/RULES_LEDGER.md + validators â€” seeded by workstream G; every subsequent item extends it).
19. **Doctrine-neutral floor, in parallel throughout:** party system + frames, chat channels/whispers/friends, OIDC, GM tools + rulings-trace, telemetry, rate limiting, CI + bot/sim load harness.

## (b) Hardest architectural risks

1. **Single-module city-sim cost.** Steady state is trivial (~70 events/s at 10 k citizens, 30:1 clock) but three multipliers are unproven: (i) *concurrent encounters* (a settlement-wide riot is thousands of round-granular turns), (ii) *observed-tier pathfinding* (A* per moving observed citizen), (iii) *subscription fan-out* (hundreds of citizen rows Ã— dozens of observers). Mitigations in design â€” fidelity tiers, batch resolution, event-granularity positions, encounter caps as guard-escalation *data* â€” but the number comes only from the sim-scale harness. Build the 10 k-citizen staging measurement before settlement #3.
2. **Observed/unobserved tension.** Outcome-equivalence is clean philosophically, fragile at the seams: arrival mid-batch must see consistent state; path-dependent outcomes differ across tiers by construction; determinism must be per-event-seeded so tier promotion never re-rolls history. Owner sign-off required (Â§1.11c); guard = tier-transition test suite asserting identical rule outcomes across promotion/demotion.
3. **Death-RAW retention risk [OWNER DECISION].** RAW death + costly resurrection is the doctrine and the single most churn-dangerous rule in the game. File: permadeath-if-unraised vs True-Resurrection soft floor. Sub-risks: background-tab death saves (settle the throttling policy *with* the death rule); low-level players face the harshest window (Revivify needs a 5th-level cleric + 300 gp) â€” temple pricing data effectively sets the real death penalty curve. Instrument deaths-by-cause and resurrection uptake from day one.
4. **The judgment-gap catalog is the true scope of "exact simulation."** ~40â€“60 codified rulings; each small, collectively they *are* the DM; every one is a place where two agents can codify contradictory common sense. Control: single tracked catalog + rules-ledger cross-reference, every resolver cites its data file, rulings-trace makes each decision auditable in-game. The failure mode isn't missing rulings â€” it's silent, inconsistent ones.
5. **Browser carryovers + the lawful-griefing bet.** Schema evolution without wipes; single-threaded frame budget now rendering living crowds; tab-throttle fairness (entangled with #3); the SRD combinatorial validation surface (the headless scripted-combat harness must grow *with* V2). New: friendly fire + open-PvP-as-simulation means anti-griefing IS the crime/law/bounty simulation â€” elegant, unproven at MMO scale, gated on owner sign-off. If law-not-code fails in playtests, the doctrine-compliant lever is harsher law *data* (execution, exile, resurrection denial), not code prevention â€” plan the data escalation path now.

## (c) Execution Order â€” the blitz plan (dependency order only; no calendar exists)

**Workstreams launchable RIGHT NOW, in parallel, strict file-ownership partitions:**

| WS | Scope | Owns (exclusively) |
|---|---|---|
| **A â€” Slice server completion** | v1 combat/movement/mob-AI/dialogue to DESIGN parity | `server/*.cs` (existing files only) |
| **B â€” Bridge** | TS SDK wrapper â†’ `hollowmere_net.js` | `bridge/**`, `client/web/**` |
| **C â€” Client world** | World scene, TileMap builder, EntityView, camera | `client/src/world/**`, `client/src/game/Game.gd` |
| **D â€” Client GUI** | MainMenu, IntroCinematic, HUD, dialogue, journal, settings | `client/src/ui/**` |
| **E â€” Asset pipelines** | Blender sprite/tile scripts, music/sfx synth | `tools/blender/**`, `tools/music/**`, `assets/**` |
| **F â€” Simulation kernel** | Â§1: clock, dispatcher, citizen tables, creature interface, schedules, spatial index | `server/sim/**` (staged now in `features/simulation/`) |
| **G â€” SRD data + rules ledger** | Stat blocks, equipment+prices, spells, XP/CR tables, conditions; ledger seed | `data/srd/**`, `docs/RULES_LEDGER.md`, `tools/srd/**` |
| **H â€” Rulings catalog** | Â§1.9 data authoring + resolver specs | `data/rulings/**`, `docs/RULINGS_CATALOG.md` |
| **I â€” Procgen** | (running) generators + `population.json` | `tools/procgen/**` |
| **J â€” Live-ops** | CI, bot harness, telemetry spec, runbooks, legal upkeep | `tools/deploy/**`, `.github/**`, `docs/runbooks/**` |
| **K â€” Player economy** | (running) shops + trading staged | `features/trading/**` |

Cross-workstream writes go through DESIGN.md contract PRs, never direct edits to another stream's files.

**The true critical path (hard dependencies only):**

```
A (slice combat parity) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
B (bridge) â†’ C (world scene) â†’ playable slice â†’ publish v1
                                      â”‚
F1 world clock â†’ F2 dispatcher â†’ F3 citizen tables + creature interface
      â†’ F4 schedules/needs â†’ F5 observed/unobserved tiers
      â†’ [F6 spatial index âˆ¥ F4]
G (SRD data) â”€â”€â”€â”€â”€â”€â”€â”€â†’ feeds F3, Â§5 reversals, Â§10 XP, the rules ledger
H (rulings v1) â”€â”€â”€â”€â”€â”€â†’ feeds F4/F5, Â§5, Â§9, Â§1.7
I (procgen population.json) â†’ F7 population loader â†’ settlement #2 from seed
      â†’ 10k-citizen sim harness measurement (J) â†’ scale decisions (b)#1
A + F3 â†’ Â§5 initiative anchors â†’ reactions/policies â†’ death saves (Â§4) â†’ crime/guards (Â§1.7)
K (trading/shops staged) â†’ merges after A publishes v1
```

**Waves (batched strictly by what-blocks-what):**

- **Wave 1 â€” no prerequisites, in flight now:** all eleven workstreams. Inside F: clock â†’ dispatcher â†’ citizen table proceed against DESIGN shapes; G and H are pure data work; E is pure pipeline work; B/C/D race to the playable slice; K stages against conventions.
- **Wave 2 â€” blocked only by Wave-1 contracts:** initiative-in-round + reaction economy (A + F1); death saves/resurrection (A + F1 + G components); friendly-fire AoE + LoS/cover (A + rulings #6); full conditions/concentration/exhaustion (G + creature interface); XP-from-CR (G); population loader + village retrofit (F3 + I); citizen rendering + crowd budget (C + F3); reaction-policy + death UI (D + server rows); trading/shops merge (K + A published).
- **Wave 3 â€” integration layers:** crime & guards (spatial index + creature decisions + combat reversals); professionsâ†’stockâ†’prices (schedules + SRD prices); social/attitude + gossip; weather/light with darkness rules; RAW rest/travel/downtime; creature decision engine everywhere; GM rulings-trace + sim telemetry.
- **Wave 4 â€” scale & proof:** 10k-citizen staging measurement; tier-transition test suite; settlement #2 fully alive from seed; second campaign as overlay proof; owner decisions executed live (death-RAW mode, PvP-as-simulation, observation policy, world-clock ratio).
- **V3 ring:** remaining classes/levels 6â€“20, CR 5+ bestiary, rare+ items/attunement, guilds/housing/brokerage, trade routes + courier mail, community modules, module-per-region sharding if â€” and only if â€” the Wave-4 measurement demands it.

Maximum speed rule: any item whose dependencies are met is launchable immediately by any free agent team; nothing in this document waits for a date, only for its inputs.

---

### Critical Files for Implementation

- `DESIGN.md` â€” the binding contract; non-negotiables #6/#7/#8 govern this roadmap; every new table/reducer shape lands here first.
- `server/Lib.cs` â€” existing schema + lifecycle; additive-evolution discipline and the player/citizen boundary live here.
- `server/sim/` (new; staged in `features/simulation/`) â€” the kernel: `WorldClock.cs`, `SimDispatch.cs`, `Citizen.cs`, `Creature.cs`, `Schedule.cs`, `Spatial.cs`, `Rulings/*.cs`.
- `docs/RULES_LEDGER.md` + `data/rules_ledger.json` â€” the bijection's enforcement artifact (workstream G seeds it; every feature extends it).
- `tools/codegen/embed_content.py` â€” the dataâ†’server/client pipeline; grows to validate/embed `data/srd/**`, `data/rulings/**`, and `population.json`.
- `data/rulings/` â€” the codified DM: DC ladders, morale, combat AI, law, attitude shifts â€” the judgment-gap catalog as data.
