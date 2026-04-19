# Andesite Age — handover

This doc orients a fresh Claude Code session to pick up modpack authoring.
The research phase is done; the next session starts from `packwiz init`.

## What this project is

A Minecraft 1.21.1 NeoForge modpack named **Andesite Age**, built around the
Create mod as the *industrial backbone end-to-end* — no parallel tech mods,
no power-creep that outscales Create. Loose vanilla+ otherwise (Farmer's
Delight-style additions OK; aggressive worldgen overhauls out). Distribution
target: Modrinth `.mrpack` authored via packwiz.

Personal context: built for the user (`thomasar`) and friends, who came off
ATM10 / ATM10-Sky frustrated that Create got outscaled.

## Decisions locked

| Thing | Decision |
|---|---|
| MC version | **1.21.1** |
| Loader | **NeoForge** |
| Pack slug (Modrinth) | **`andesite-age`** |
| Title (visible in Modrinth search) | `Andesite Age — A Create-focused vanilla+ pack` *(or similar; ensures "Create" hits search)* |
| Author | `thomasar` |
| Authoring tool | `packwiz` (installed at `~/.local/bin/packwiz`) |
| Distribution | Modrinth (`.mrpack`) |

The full project memory lives in `~/.claude/projects/-Users-thomas-projects-minecraft-create-plus/memory/` and is loaded automatically. **Read it.** Especially `project_create_plus.md` for the pack thesis (the "does this trivialize Create?" / "does this break vanilla world feel?" filters that gate every mod-add decision).

## Repo layout

```
/pack/         ← packwiz pack root (currently empty — packwiz init goes here)
/scripts/      ← research/tooling helpers (see below)
/references/   ← research artifacts: per-pack mod JSON, comparison matrices,
                 version-availability report, name-candidate logs
/downloads/    ← raw modpack zips for analysis (gitignored)
.env.local     ← CF_API_KEY (gitignored). Quoted because CF keys are bcrypt-fmt
                 ($2a$10$…) and unquoted $ chars get expanded when sourced.
```

## Tooling already in place

- **`scripts/cf-api`** — wraps `curl` with the CF API key header. Sources `.env.local`. Usage: `scripts/cf-api "/v1/mods/search?gameId=432&classId=6&slug=copycats"` and similar.
- **`scripts/resolve-manifest`** — turn a CF modpack `manifest.json` into a clean per-mod metadata array.
- **`scripts/compare-packs`** — cross-pack mod comparison matrix (`--create` for Create-ecosystem only, `--min N` to filter by pack count).
- **`scripts/verify-versions`** — for each slug in `references/shortlist.txt`, query CF and report 1.21.x / NeoForge availability.
- **`scripts/check-modrinth-slug`** — bulk Modrinth slug-availability check (no auth needed).

CF API quick reference: `gameId=432` Minecraft, `classId=4471` Modpacks, `classId=6` Mods. `slug=foo` is exact, `searchFilter=foo` is fuzzy.

## Reference packs analyzed

Living in `references/`:

| Pack | MC | Loader | Mods |
|---|---|---|---|
| CABIN (Pansmith) | 1.20.1 | Forge | 176 |
| Create: Perfect World (mrbeardstone) | 1.20.1 | Forge | 120 |
| Create: Astral (Laskyyy_) | 1.18.2 | Fabric | 186 |
| Create: Arcane Engineering | 1.18.2 | Forge | 197 |

Detailed analysis: `references/matrix-create.md` (Create-ecosystem cross-pack matrix). `references/versions.md` (1.21.1 NeoForge availability for the shortlist).

## Verified-on-1.21.1-NeoForge shortlist (all ✅)

**Core Create ecosystem:** `create`, `copycats` (Copycats+), `create-new-age`, `create-deco`, `create-stuff-additions`, `createaddition` (Crafts & Additions), `create-central-kitchen`, `create-enchantment-industry`, `create-jetpack`, `create-confectionery`, `create-big-cannons`, `create-liquid-fuel`, `create-power-loader`, `create-encased`, `create-alloyed`, `interiors` (Create: Interiors), `rechiseled-create`

**Bridges/adjacent:** `ars-creo`, `ars-nouveau`, `cccbridge`, `custom-machinery`, `chisels-bits`

**Auxiliaries:** `jei`, `emi`, `jade`, `kubejs`, `sodium`

**Notable absence:** `create-steam-n-rails` is **1.20.1 only**. Acceptable because base Create has the full train system since 0.5; S'n'R is flavor/aesthetic. Re-check periodically with `scripts/verify-versions`.

**Skip (abandoned):** `create-chunkloading`, `create-extended-cogs`, `create-recycle-everything`, `daves-building-extended` — last update 2023–2024.

**Wait (active but 1.20.1-only):** `compressedcreativity` (blocked by PneumaticCraft), `create-additional-recipes`.

## Pack state on handover

`packwiz init` already ran. Current `pack/pack.toml`:

```toml
name = "Andesite Age"
author = "thomasar"
version = "0.1.0"
pack-format = "packwiz:1.1.0"
[versions]
minecraft = "1.21.1"
neoforge = "21.1.227"
```

`pack/index.toml` exists with `files = []` — no mods added yet. **Verify NeoForge `21.1.227` is still the right pin** (newer 21.1.x may exist; check Modrinth/NeoForge for the latest stable). Update via `packwiz settings` or by hand-editing if needed.

## Concrete next steps for this session

1. **Add mods in tiers** from `pack/`. Use `packwiz mr add <slug>` for Modrinth (preferred — cleaner licensing) and `packwiz cf add <slug>` for CurseForge fallbacks. Tiers from the shortlist above:
   - Tier 1 (Create core + must-have addons): `create`, `copycats`, `create-new-age`, `createaddition`, `create-deco`, `create-stuff-additions`, `create-enchantment-industry`, `create-central-kitchen`
   - Tier 2 (auxiliaries): `emi` (preferred over `jei` for Create packs), `jade`, `kubejs`, `sodium`
   - Tier 3 (vanilla+ gap-fillers — not yet researched): `farmers-delight` is the user's only confirmed pick. **Discuss with the user before adding more.**
   - Tier 4 (flavor — discuss with the user): `create-jetpack`, `create-big-cannons`, `create-liquid-fuel`, `create-confectionery`, `interiors`, `rechiseled-create`, `chisels-bits`, `ars-nouveau` + `ars-creo`

2. **Test in Prism Launcher.** Export `.mrpack` (`packwiz mr export`), import into a fresh Prism instance, launch, smoke-test that Create works.

3. **Reserve the Modrinth slug.** Requires Modrinth auth (PAT token). Not yet set up — ask the user. The slug `andesite-age` was confirmed available as of 2026-04-19; re-confirm before publishing.

## Open questions to ask the user

- **Performance stack beyond Sodium:** want NeoForge-era staples like `ferritecore`, `modernfix`, `embeddium-extras`? Need a verification pass.
- **Vanilla+ gap-fillers beyond Farmer's Delight:** any specific picks (Sophisticated Backpacks? Waystones? Graves? Supplementaries?). User said the *world feel* should stay vanilla — these are inventory/QoL, separate question.
- **Tier 4 flavor mods:** which of {Big Cannons, Jetpack, Ars Nouveau, etc.} actually fit the friend group's vibe?
- **Server-side considerations:** is this also being deployed to a dedicated server? (User mentioned a CF API key from their dedicated server setup. Worth knowing if server-only or client-only mods need flags.)

## Things NOT to redo

- The CF/Modrinth research is complete; don't re-fetch the four reference packs unless mods need re-verification.
- The name decision is locked. Don't re-litigate.
- 1.21.1 NeoForge is locked. Don't re-litigate.
