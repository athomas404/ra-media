# Ritual Apotheosis — Mod Compatibility Ledger

*Last updated: 2026-07-20, against Ritual Apotheosis 0.1.12.*

Ritual Apotheosis patches some sensitive vanilla surfaces (chiseling, temporal
stability, entity physics, fluids, the world map), so compatibility is treated
as real engineering, not hope. The work so far: two complete playtester
mod-packs — roughly **500 mods, decompiled and machine-scanned** against every
Harmony patch and engine seam RA touches, then deep-read where the scan flagged
overlap — plus a triage survey of the ModDB's **882 most-downloaded mods** to
prioritize future hardening. Findings ship as defensive measures inside RA
itself; you generally don't need a compat addon.

Absence from this page does **not** mean incompatibility — most mods simply
never touch the same surfaces. If you hit something strange on a modded
server, please report it (Discord or the mod page): the audit machinery exists
and gets re-run.

## Named findings and dedicated handling

| Mod(s) | What we found | What RA does about it |
|---|---|---|
| Frontiers Map (and any map-icon pack) | Waypoint-editor crash after texture reloads — an underlying vanilla issue RA's reload was exposing | RA no longer performs the destructive reload, and self-heals map icons after *any* reload (yours or another mod's); bug reported upstream |
| Hydrate or Diedrate | Hand-drinking bypassed a ritual drinking probe | Conditional bridge feeds its drink events into the same probe |
| A Culinary Artillery | Bottle-drinking bypassed the same probe | Same bridge treatment |
| eschisel32, chiseltools | Fine-chisel auto-conversion destroyed/desynced idol state one-way | Chisel-suite guards protect idol blocks from foreign conversion paths |
| Realistic Water | Replaces water's spreading behavior, silently detaching RA's ship liquid-flow gate | Gate re-targeted so ship fluid handling survives the replacement |
| Specialized Classes | Gravity-stat players skipped the entire deck-riding physics chain | Fixed — modified-gravity players ride decks correctly |
| CM Mobility | Shares the deck-rider physics seam | Verified composing — both mods' effects stack safely |
| Steady Greenhouses | Feared transpiler clash on berry-bush growth | Verified **no conflict** empirically; on-deck bushes get correct ship-frame climate |
| bedrespawner, blush | Ship-bed respawn anchors weren't cleared through their modified paths | Anchor-clearing generalized to cover both |
| overhaulliblegacycompat, animationslib, stackscoolslower | Call Harmony `UnpatchAll(null)` at world unload, stripping **every** mod's patches process-wide | RA self-heals its own patches and ships a startup detector that warns you by name |
| XSkills family (xskillsfork, xskillsxaldisclasses) | Shares the mining-speed surface | Verified commutative — order-independent, no conflict |
| Any mod binding the L key | First-registered handler can silently shadow RA's idol legend | Live key-collision detection: one-time chat/log notice naming the contender |
| Translocator Engineering Redux | Shares translocator surfaces | Verified no conflict |
| Translocator Relocator | Relocated translocators were invisible to clue discovery | Discovery hardened to find them |
| Eco Machina | Shares the mini-dimension system | Verified non-overlapping |
| Drifter reskin / loot / auto-pickup mods | Potential clash with ritual drop tables | Verified no conflict |
| Modded mechanical renderers (linearpower pattern) | Could crash RA's ship device wrapper | Wrapper inverted to a whitelist — unknown renderers degrade gracefully instead of crashing |
| Wilderlands | World-gen interplay | Dedicated compatibility shim |
| Seafarer | Alternate boats (decompiled with the creator's permission) | Modded vehicles are hull-solid; carrying them *as deck cargo* is a known limitation |
| Combat Overhaul | Projectiles fly through chunky-ship hulls | **Known limitation** — documented, fix anchored for a future engine surface |

Also verified clean (highlights): placeonslabs, electricalprogressiveequipment,
dryingspeedoverhaul, useful_stuff / still_useful_stuff, alchemy, wearandtear,
necessaries, insanitylib, slowtox, pinrecipes, farmlanddropswithnutrients,
catchlivestock, playermodellib, carryon, angelbelt — each shares a base type or
surface with RA but patches different methods or composes safely.

## Coverage and method

- **Playtester pack 1** (2026-06-08, plus a second structural pass 2026-06-10):
  a full server modlist decompiled into a 310-mod corpus (8,854 source files),
  scanned for patch-pair intersections, subclass bypasses, and
  reverse-direction breakage, then deep-read by four parallel review passes.
- **Playtester pack 2** (2026-06-10): a 261-zip modlist, 257 resolved (plus
  exact-version variants for high-drift mods), growing the decompiled corpus
  to ~508 mods.
- **ModDB survey** (2026-05-20): the full mod catalog (2,174 mods at fetch),
  filtered to the 882 with 10k+ downloads and tiered by probable overlap with
  RA's ship engine — the standing triage list for future hardening.

The audit doubles as RA bug-hunting: several RA-side fixes (a melee damage
filter, a fall-damage gate) were found by pointing this machinery at our own
code.
