# Ritual Apotheosis — Design Philosophy, Architecture & Collaborator Notes

> **SPOILER WARNING.** This document discloses design reasoning and the layout
> of the codebase. It deliberately does *not* repeat the worked-example ritual
> spoilers (those stay on the mod page, behind its spoiler dropdown), but if
> you want to experience the mod completely fresh, close this tab.
>
> This page exists because the ModDB description has a size limit and this
> section outgrew it. File paths below refer to the main development repo,
> which is private (it vendors decompiled game source for diffing across game
> updates); if you want to contribute, reach out on the Discord and we'll get
> you access.

## Design pillars

Five principles shape every design decision in the mod:

1. **Rituals are actions, not menus.** The canonical shape of a rite is a sequence of physical conditions and actions in the world. There is no "begin ritual" button and, for the bulk of my already-planned rites, no required altar block. When a GUI dialog is unavoidable (this may take the form of shrines revisited for recurring blessings or oracle-style consultation rites I implement in the future), it should be the rare case, not the default. `docs/RITUAL_DESIGN_VISION.md` is the source of truth for this principle; `docs/design/ACTION_DRIVEN_RITUAL_HOST.md` documents the code pattern that implements it.
2. **The mystery is load-bearing.** Clues are organically discovered. The first fragment for any rite is deliberately too cryptic to act on. Instead, the arc of understanding is the content. Any design decision that reveals information faster than discovery warrants is re-examined.
3. **Collaboration must never be foreclosed.** Two players with different fragments of the same rite should be able to perform it together. Clues are not account-bound knowledge; they are intentionally cryptic texts that multiple players are free to interpret and discuss among themselves - and on larger servers, perhaps that knowledge could be bartered!
4. **Server-authoritative everything.** If a rite affects the world, the server decides. Client code renders and predicts, nothing more.
5. **Behaviors over subclassing.** The mod composes `CollectibleBehavior` / `BlockEntityBehavior` / `EntityBehavior` rather than building deep inheritance trees. This matches how Vintage Story's survival mod is built and plays well with other mods.

## The idol system

Rituals primarily reward idol patterns - knowledge of how to sculpt an idol, which can progress from its initial rough-hewn shape into more refined higher tiers. The idol system is a full subsystem of its own, governed by twelve design principles (see `docs/design/IDOL_DESIGN_PHILOSOPHY.md` for the principles, and `docs/design/IDOL_SYSTEM_DESIGN.md` for the technical implementation -- template registry, validator, pop-off handler, tier-state behavior, and mastery tracker). In brief:

* Pop-off (reaching an idol-tier threshold, causing the idol to be added to your inventory) fires on exact wireframe match against the next tier's canonical voxel pattern. There is no "close enough" threshold.
* Tiers run Novice -> Practiced -> Skilled -> Master. Each tier gates the next tier's wireframe; the chain must be walked from the bottom.
* Tier directly drives ritual mechanics (ward radius, effect magnitude). A Master formation is base-scale infrastructure; a Novice formation is a keyhole spot effect.
* Free aesthetic customization is allowed at every tier, bounded by a 50% similarity floor to the canonical. Drop below and the idol becomes a non-functional "Ruined Idol" - ruin is terminal.
* Mastery tiers (a separate 6-rung player-progression ladder, distinct from the 4 idol tiers) unlock perks like auto-chisel shortcuts at Skilled+.
* Every idol can be customized to some extent within its own tier, after it clears the tier pop-off threshold - note that customized idols are non-stackable. Every idol tier carries its own voxel snapshot; placement always restored voxels verbatim, not canonical defaults.
* Excessive customization (50% differential from the idol tier frame) caused an idol to be ruined - it can still be placed, but becomes purely decorative.

The category has since split three ways: **ward idols** (corner formations projecting zones - the Madness Ward, and more to come), **battle totems** (single-placement aura idols with charge/depletion, capped at Master and driven by their own credit-based mastery ladder), and **component idols** (non-tiered shapes that exist to be *parts* of larger works - ritual gems, shipyard corner totems, and the ship fixtures). The ship fixtures added one more wrinkle: **multiblock idols**, sculpted across several blocks with a segment-by-segment guided wireframe and a helm-view segment picker in the Idol Guide dialog.

Ward zones -- the persistent area effects that idol formations project -- have their own design spec covering geometry, tier-scaled radii, stability-suppression hooks, and grace-window rules: `docs/design/WARD_ZONE_DESIGN.md`.

For a deep-dive reference on every player-facing item and block the mod adds (idol, ritual scroll, ritual altar, and placeholders), including their JSON definitions, shape/texture sources, VFX tables, and attribute schemas, see `docs/testing/RITUAL_ITEM_REFERENCE.md`.

## Codebase tour

The repo is generally organized as follows - some of these files/folders may reference a number of partial-class files:

* `src/Config/RitualConfig.cs` - all tunable values, heavily commented.
* `src/RitualApotheosis.cs` - root ModSystem entry point and static singleton registries.
* `src/Blocks/` and `src/BlockEntities/` - block subclasses and their stateful counterparts.
* `src/Items/` - scroll, grimoire, idol, and reagent items.
* `src/Behaviors/` - `CollectibleBehavior` and `EntityBehavior` classes composed onto collectibles and entities.
* `src/Systems/` - heavyweight subsystems: ritual engine, clue probes, formation detection, ward-zone registry, network channels, save envelope.
* `src/Idol/` - the full idol-system subsystem (template registry, validator, pop-off handler, tier-state behavior, mastery tracker).
* `src/Totem/` - the battle-totem subsystem: aura registry, effect handlers, resonance tracker, and the altar (consecration + recharge hosts, vertex activation, pentagram geometry, barrier).
* `src/Shipwright/` - the chiseled-ship engine: mini-dimension plumbing, buoyancy/collision, fluid displacement, fixtures (wheel/sails/oars), shipyard, sanctification launch, and the `/ra ship` command surface. Deliberately decoupled from the ritual engine; the Sanctification ritual is the one bridge.
* `src/Celestial/` - Astral Communion: constellation registry + overlay renderer, witness ledger, trigger host, answer ledger, picker.
* `src/Compat/` - Harmony patches against vanilla and defensive compat measures (detectors, heals, contention notices).
* `src/Commands/` - the `/ra` dev-command tree, one file per command group.
* `src/Gui/` - the rare GUI dialogs (grimoire, shrine-style dialog, idol legend/guide).
* `src/Util/` - shared helpers (logging wrapper, math).
* `assets/ritualapotheosis/` - all JSON content (blocks, items, rituals, clues, idol templates, constellations), textures, sounds, shaders, and the `lang/` folder.
* `tools/star_catalog/` - offline Python tooling that mines constellation shapes out of the game's real star catalog.
* `tests/` - the xUnit suite (600+ tests at time of writing) covering the pure-logic cores.

For a Big-O complexity analysis of every system (tick budgets, per-player costs, scaling behavior under load), see `docs/testing/COMPLEXITY_REPORT.md`.

## Documentation index

The `docs/` folder is the long-term memory of the project. If you want to contribute, start here and follow the cross-links. I have used Claude extensively in preparing the codebase, and these documents have assisted myself, and Claude's agents along the way.

* `docs/CLAUDE_PROJECT_INSTRUCTIONS.md` - project orientation and session-start checklist.
* `docs/RITUAL_DESIGN_VISION.md` - source of truth on "what is a ritual" and "what is an altar." If a lower-level doc seems to contradict this, the vision wins.
* `docs/LEVERS.md` - the living index of every game system, API surface, JSON asset, or event hook the mod depends on, organized by ritual beat with a cross-cutting table.
* `docs/levers/beat{1..5}.md` - per-beat lever tables (Discovery, Lore & Purpose, Components, Location & Environment, Ritual Performance).
* `docs/levers/SESSION_LOG_INDEX.md` - the session journals are split per work area (idol-ward, ritual, ui-vfx, discovery, build, docs, shipwright); this index routes you to the right one. Newest-on-top; the "where did we leave off" anchor.
* `docs/ROADMAP.md` - the short, living "what's next" list. Priority-ordered but not contractual.
* `docs/design/CLUE_PROBE_SYSTEM.md` - how active clue-advancement probes work (tick vs event, per-ritual cooldowns, how to add one).
* `docs/design/RITUAL_STATE_MACHINE.md` - shared ritual backbone: validator, state machine, definition schema, outcome dispatcher, save/load, failure cooldown.
* `docs/design/ACTION_DRIVEN_RITUAL_HOST.md` - canonical non-BlockEntity ritual host pattern (see `MadnessConsecrationHost` as the live example).
* `docs/design/IDOL_DESIGN_PHILOSOPHY.md` - twelve principles for the idol system. Revisions here precede code changes, not follow them.
* `docs/design/WARD_ZONE_DESIGN.md` - ward-zone geometry, tier-scaled radii, stability-suppression hook, grace-window rules.
* `docs/design/RITUAL_SHIPWRIGHT.md` + `docs/design/SHIPWRIGHT_ROADMAP.md` - the ship engine's design spine and milestone map (M1 through M9: buoyancy, multiplayer, persistence, terrain collision, fluid displacement, seating, docking, pitch/roll, net-sync).
* `docs/design/BATTLE_TOTEM_DESIGN.md` + `docs/totem_consecration/` - the battle-totem system: design, altar construction, tiered rites, and the five Spirit Journeys.
* `docs/design/CELESTIAL_GUIDANCE.md` + `docs/design/CONSTELLATION_CATALOGUE.md` + `docs/design/ASTRAL_TRIGGERS.md` + `docs/design/ASTRAL_WITNESS_PLAN.md` - the astral stack: how the VS night sky actually works, the full constellation database, the trigger/answer framework, and the witness/mastery loop.
* `docs/compat/` - the mod-compatibility audit reports and conflict-resolution plan (public summary: [Mod Compatibility Ledger](https://github.com/athomas404/ra-media/blob/main/compat/CHECKED_MODS.md)).
* `docs/VERSION_SURFACE.md` - catalog of every place the mod depends on internal-to-Vintage-Story surfaces (private classes, Harmony patches, reflection). Audit this before migrating to a new game version.
* `docs/guides/AGENT_SPLIT_PROTOCOL.md` - the canonical playbook for file-split tasks (mojibake sweep, ownership boundaries, build verification).
* `docs/guides/CONTRIBUTOR_CONVENTIONS.md` - coding patterns, logging conventions, and the two-layer testing strategy.
* `docs/guides/VS_UPDATE_GUIDE.md` - six-phase checklist for migrating to a new Vintage Story version (most recently exercised for 1.22.4; see `docs/guides/UPDATE_1.22.4.md`).
* `docs/testing/PLAYER_TOAST_TRACKER.md` - every player-facing toast, its trigger conditions, and associated VFX/SFX.
* `docs/testing/DEV_TOAST_TRACKER.md` - every `/ra` command's console output, for verifying dev-harness behavior.
* `docs/testing/RITUAL_ITEM_REFERENCE.md` - deep-dive reference for all player-facing items and blocks the mod adds.
* `docs/testing/COMPLEXITY_REPORT.md` - Big-O analysis of every system, grouped by performance impact.
* `docs/testing/END_TO_END_RA_01_MODTEST.md` through `END_TO_END_RA_04...` - step-by-step end-to-end test manuals: Madness Consecration, the totem arc, the astral systems, and ships-with-oars.
* `docs/madness_consecration/` - per-stage ritual specs (clue arc, Stage 1 consecration, Stage 2 formation).

## Contributing

Three things are worth knowing before opening a PR:

1. Read `RITUAL_DESIGN_VISION.md` before proposing a new ritual shape. Most "wouldn't it be cool if" proposals might turn out to conflict with the "action-driven-rituals" pillar, and we might find that this conflict is subtle the first time we encounter it.
2. My logs were sometimes recorded in the middle of a coding session and sometimes at the end. The session-log files have a documented cadence rule: add your entry after the first substantive change and update it as work continues, rather than waiting until you are done. The reason is practical - if work is interrupted, the partial entry is still more useful than none.
3. `_decompiled/` is THE Vintage Story ground truth. When in doubt about how a vanilla Vintage Story system works, grep the decompiled source before guessing. Memories, guides, and even this document might drift; the decompiled source cannot.

For coding conventions, logging patterns, and the two-layer testing strategy (xUnit + in-game `/ra` commands), see `docs/guides/CONTRIBUTOR_CONVENTIONS.md`. If you are migrating the mod to a new Vintage Story version, `docs/guides/VS_UPDATE_GUIDE.md` walks through the six-phase upgrade process (re-decompile, audit the diff against `VERSION_SURFACE.md`, fix, build, test, bump).
