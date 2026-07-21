# Ritual Apotheosis — Server Administrator's Guide

> **Spoiler level: Architectural.** This guide does not describe specific
> rituals, clues, or their solutions, but it does reveal the *shape* of the
> mod - how discovery is tuned, what the progression timers look like, and
> what operator tools exist. If you plan to play the mod as a fresh
> participant on your own server, try to finish at least one ritual before
> reading on. And if something seems horribly broken despite my playtesting,
> consider reaching out first (Discord or the mod page) so I can resolve it
> without forcing you to spoil yourself!
>
> (This guide lives here because the ModDB description has a size limit.)

## Where settings live

Every tunable value lives in a single config file at:

```
%AppData%\VintagestoryData\ModConfig\ritualapotheosis.json
```

The file is written on first load with sensible defaults. Every field has a doc comment in source (`src/Config/RitualConfig.cs`) explaining what it does. Editing the file while the server is running will not take effect until the next world load; changing values via the live `/ra configset` command does. For a full breakdown of every config field and its expected values, see the config dump section of `docs/testing/TESTING_COMMANDS_REFERENCE.md`. Note that the config has grown considerably with the new systems - there are now knob groups for battle totems, the astral systems, and ships alongside the original ritual/idol values; `/ra configdump` is the fastest way to see the full current surface.

I've attempted to group all available commands by gameplay stage - and the output of each command should also dump its output to the server log, if I've configured it right. The most common knobs you might want to reach for:

* `BlueprintDropChance` (default `0.05`) - probability that an eligible creature kill yields a clue fragment. Raise this to speed up discovery on a fresh world; lower it on a long-running server where players have saturated the drop tables.
* `ClueAcquisitionCooldownHours` (default `48`) - per-player, per-ritual cooldown between clue drops. Keeps any single session or ambitious player from firehosing all of a ritual's fragments at once. You can set to `0` if you are testing, or if you want rapid progression.
* `PassiveDecodeHours` (default ~600 hours - about one in-game season) - represents the number of in-game hours a player must spend holding a fragment before a slow background decode step for one acquired ritual clue ticks forward. This is the safety net for players who do not chase clue-advancement probes actively - I understand some players like to just focus on one thing, and I don't want to exclude them! That being said, active probing is the fast track. Lower this value for more forgiving servers.

Probe thresholds (the cumulative-experience counters that drive active clue advancement) live in the same file and are documented per-probe. Each probe has a threshold and a "per-step" value; raising the per-step value makes clues slower to advance, lowering it makes them faster.

## Hosting multiplayer

* **Server-authoritative by design.** Ritual state, clue progress, idol tier, and zone activation all live on the server. A client cannot fake a ritual.
* **Save-safe.** Mid-ritual state, per-player clue progress, and active zone registrations all survive world save/load. Ship state, totem charge, and astral progression ride the same save envelope.
* **Compatibility.** Harmony patches target vanilla chiseling and temporal-stability plumbing (plus, for the newer systems, a handful of combat, fluid, and map surfaces). The mod should coexist with most vanilla-adjacent mods; anything that also patches those surfaces should be tested before deployment. `docs/VERSION_SURFACE.md` catalogs every internal VS dependency (Harmony targets, reflected fields, private-class access) so you can audit compatibility at a glance, and the [Mod Compatibility Ledger](https://github.com/athomas404/ra-media/blob/main/compat/CHECKED_MODS.md) summarizes the playtester-pack audits.

## The `/ra` command tree

The `/ra` command is gated on `DevMode` (default `false` in final releases - note that the current 0.1.x builds are playtest releases and ship with dev tooling enabled). To enable it on your server, set `DevMode: true` in the config and restart. The commands themselves still require gamemode privilege, so enabling dev mode does not open the tree to untrusted players on a shared server.

Here is a partial tour of commands that operators find useful for assisting players or running playtests. All output is routed through a formatting-stripping layer so what you see in chat is what the command actually produced (see above note about server-log dumps).

* `/ra configset <field> <value>` - hot-patch any config field without a restart. Good for dialing in drop rates mid-session.
* `/ra configdump` - dump the current effective config to chat. Useful for confirming that a config edit actually loaded.
* `/ra clue give <player> <ritual> <clue>` - manually hand a player a specific clue. Rescues a player who missed an expected drop.
* `/ra clue list <player>` - show which clues a player currently holds and at what decode stage.
* `/ra idol capture <tier>` - snapshot the chiseled block you are looking at as a template at a specific tier. This is how new idol templates get authored.
* `/ra idol zones` - list every currently active ward zone on the server, with its grace-window state.
* `/ra idol wardfx test` - fire the enter/exit ward-zone visual effects at your location. Handy for verifying that a new client has the assets wired correctly.
* `/ra ritual list` - show every registered ritual definition and its current state.
* `/ra maxout` - force one player's Grimoire to its most complete state in a single command - every clue decoded, every pattern granted, every ladder maxed. The fastest way to inspect end-game state or unstick a hopeless save.
* `/ra ship ...` - the ship operator's subtree: spawn/delete/inspect ships, and two healing commands worth knowing about - `/ra ship healmap` clears "ghost ship" images fossilized into players' cached world maps, and `/ra ship purge-orphan-carves` refills hull-shaped water dents left behind by ships that despawned badly in older versions. Both default to a dry run.
* `/ra totem ...` - battle-totem harness: altar validation, pentagram preview, forced consecrations/recharges at any tier, resonance readouts, mastery set/reset.
* `/ra sky ...` and `/ra astral reset ...` - astral harness: list/show/hide constellations, check tonight's visibility, preview the picker's choice, and reset a player's astral progression wholesale, per-pool, or per-shape.

There are many more commands under the tree, most of them related to individual development phases (idol capture, pattern rewards, mastery counters, probe triggers). They are self-documenting via `/help /ra <subcommand>`. The exhaustive command reference -- including example output for every command -- lives in `docs/testing/TESTING_COMMANDS_REFERENCE.md`. If you want to run a full end-to-end playtest from first clue drop through ward-zone activation, `docs/testing/END_TO_END_RA_01_MODTEST.md` walks through each step - and there are now sibling manuals for the totem arc (`END_TO_END_RA_02`), the astral systems (`END_TO_END_RA_03`), and ships-with-oars (`END_TO_END_RA_04`). It's been a nightmare to keep updated so if you fall in love with the mod and want to help playtest, please excuse any deprecated bits.

## Emergency unsticking

If a player gets wedged in an inconsistent state - a ritual that started and never completed, a zone that registered but should not have, a clue that advanced prematurely - the two commands most likely to help are:

* `/ra ritual reset <player>` to clear any in-flight ritual phase back to idle.
* `/ra idol ruin` / `/ra idol unruin` on a targeted block to force its ruined state or escape it, respectively.

For ship-related scars specifically - ghost hulls on the world map, or hull-shaped dents in open water - see `/ra ship healmap` and `/ra ship purge-orphan-carves` above; both were built after real playtest worlds accumulated exactly this damage on older versions.

Please report any situation where these were necessary - a player needing an operator to unstick them is a bug in the mod's handling of whatever edge case they hit, and I would like to fix it. For a catalog of every player-facing toast notification (what triggers it, what visual/audio effects accompany it, and where in code it lives), see `docs/testing/PLAYER_TOAST_TRACKER.md`.
