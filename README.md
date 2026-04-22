# Andesite Age

A Create-focused vanilla+ modpack for Minecraft 1.21.1 on NeoForge.

> **Status: beta.** Playable end-to-end and server-validated, but the
> Tier 3/4 mod list is still being finalized and balance has not been
> road-tested by a full group.

## The pitch

Most Create packs bolt Create onto a kitchen-sink tech stack and let
Mekanism, Thermal, or AE2 outscale it by mid-game. Andesite Age goes
the other way: **Create is the industrial backbone, and nothing in the
pack is allowed to trivialize it.**

Every mod passes two filters before it gets in:

1. **Does this trivialize Create's role?** No parallel tech trees, no
   alt power systems, no mod-added quarries or magic-box ore doublers.
2. **Does this break the vanilla world feel?** No aggressive worldgen
   overhauls. Modern vanilla worldgen is good enough; we're adding to
   it, not replacing it.

What you get is Create + a curated set of Create addons, plus a thin
layer of QoL, performance, and vanilla-gap-fillers. Nothing else.

## What's in

**Core**
- Create — the whole reason we're here
- Create Crafts & Additions — electrical bridge between Create and
  vanilla redstone
- Create: Copycats+ — cosmetic block shaping
- Create Deco — decorative blocks styled to match Create
- Create: Central Kitchen — Create ↔ Farmer's Delight style cooking
- Create: Enchantment Industry — automated enchanting and XP fluid
- Create: New Age — late-game electrical tier on top of CC&A
- Create Stuff 'N Additions — tools, weapons, gadgets that feel Create-native
- Create: Aeronautics — airships, propulsion, flight mechanics powered by Create
- Create: Dragons Plus — transitive dependency

**Adventure & exploration**
- YUNG's Better Dungeons / Mineshafts / Strongholds / Witch Huts / Nether
  Fortresses / Ocean Monuments — vanilla structure overhauls with no new
  loot tiers
- When Dungeons Arise + Seven Seas companion — large hand-built dungeons
  and naval structures
- Dungeons and Taverns — small dungeons, taverns, and quest traders
  scattered in villages and the wilderness
- Towns and Towers — more pillager outposts, portal ruins, desert
  outposts
- Repurposed Structures — vanilla-flavored variants of existing
  structures

**Early-game QoL**
- AppleSkin — hunger/saturation tooltips
- Bucketable — pick up more mobs in water buckets
- Easy Anvils — removes the "Too Expensive!" cap
- Easy Magic — better enchanting table UI
- Easy Villagers — block-based breeding, trading, and trade management
- Gravestone Mod — drops a grave on death so you don't lose your inventory
- Polymorph — recipe-conflict picker
- Mouse Tweaks — inventory drag-to-transfer behavior
- Controlling — searchable keybinds menu

**Vanilla+ flavor**
- Farmer's Delight — cooking, crops, and kitchen tools that fill vanilla's
  food gap (and pairs with Create: Central Kitchen for automation)
- Supplementaries — decorative and utility blocks (sconces, signs, globes,
  pedestals) in pure vanilla visual style

**UI / info**
- EMI — recipe viewer (client-only)
- Jade — in-world tooltips

**Performance**
- Sodium + Sodium Extra — rendering (client-only)
- FerriteCore — memory deduplication of block/item state tables
- ModernFix — bundled upstream patches and load-time speedups
- spark — in-game profiler (`/spark profiler`) for diagnosing TPS issues

**Scripting**
- KubeJS + Rhino — pack-side recipe and tag tweaks

A full, machine-readable list lives in
[`server-validation/expected-mods.tsv`](server-validation/expected-mods.tsv).

## What's deliberately out

- Mekanism, Thermal Series, Applied Energistics 2, Industrial
  Foregoing, and other parallel-tech stacks
- Mod-added quarries, digital miners, and ore-doubling shortcuts
- Terralith and other aggressive worldgen replacements
- Magic tech (Ars Nouveau etc.) is on the Tier 4 shortlist but not yet
  in; will only land if it fits the vibe without stealing Create's role

## Requirements

- Minecraft **1.21.1**
- NeoForge **21.1.227**
- Java 21
- ~4–6 GB RAM allocated to the client; similar for the server

## Installing

### Client (Prism / ATLauncher / MultiMC-likes)

Download the `.mrpack` from the Modrinth listing (or from Releases
once published) and import it as a new instance. Prism handles mrpack
natively.

### Server (Docker, via itzg)

The pack is authored against the
[`itzg/minecraft-server`](https://github.com/itzg/docker-minecraft-server)
Modrinth flow. Minimal `docker-compose.yml`:

```yaml
services:
  mc:
    image: itzg/minecraft-server
    ports: ["25565:25565"]
    environment:
      EULA: "TRUE"
      TYPE: MODRINTH
      MODRINTH_PROJECT: andesite-age
      MODRINTH_VERSION_ID: <pin-a-specific-version-id>
      MEMORY: 6G
    volumes: ["./data:/data"]
    stdin_open: true
    tty: true
```

Pin `MODRINTH_VERSION_ID` explicitly so a Modrinth update doesn't
silently change the pack under a live world. Bump it deliberately.

Client-only mods (EMI, Sodium) are tagged as such in the mrpack and
itzg skips them on server installs automatically.

## Known issues

A handful of upstream warnings fire on every boot and are harmless:

- `DistCleaner` errors about `AbstractClientPlayer` and `ClientLevel`
  during server init — unguarded @OnlyIn references in some mods'
  mixins; the class load fails safely and the server continues.
- Four advancements fail to load: When Dungeons Arise's
  `find_fishing_hut` and `find_thornborn_towers` (orphaned parent in
  WDA 2.1.68), and Dungeons and Taverns' `wander_add_map` and
  `give_quest_trader_trade` (upstream schema/function errors in D&T
  4.4.4). Players just won't earn those four specific achievements.

All of these are allowlisted in
[`server-validation/known-issues.txt`](server-validation/known-issues.txt).
Anything outside that file is a real regression — please
[open an issue](https://github.com/thomasreichmann/andesite-age/issues).

## Development

Pack source lives under `pack/` and is managed with
[packwiz](https://packwiz.infra.link/). The `.mrpack` is build output.

```sh
cd pack && packwiz refresh        # regenerate hashes after editing a .pw.toml
scripts/check                     # fast gate: manifest + sidedness + cached re-export
scripts/validate-server           # heavy gate: boots a headless NeoForge server
```

See [`CLAUDE.md`](CLAUDE.md) for the full dev-loop documentation,
including how validation works and which research-phase scripts not to
re-run casually.

## Credits

Built on the shoulders of the Create team and the wider Create addon
community. This pack is a curation, not a creation — every mod listed
above is the work of its respective author(s), and their licenses
apply to their jars.
