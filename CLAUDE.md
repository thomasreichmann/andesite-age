# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

An authored Minecraft 1.21.1 NeoForge modpack ("Andesite Age") managed with **packwiz** — not a conventional codebase. Source of truth is the packwiz metadata under `pack/`; the `.mrpack` is build output. See `HANDOVER.md` for current project state and open questions.

## Pack thesis — gate every mod-add decision

Create is the industrial backbone end-to-end; vanilla+ otherwise. Before adding any mod, check both filters:

1. **Does this trivialize Create's role?** (parallel tech mods, alt power, mod-quarries → no)
2. **Does this break the vanilla world feel?** (big worldgen overhauls → no)

Full thesis is in auto-loaded memory (`project_create_plus.md`).

## Common commands

Mod authoring (run from `pack/`):

- `packwiz mr add <slug>` — add from Modrinth (preferred; cleaner licensing)
- `packwiz cf add <slug>` — CurseForge fallback (requires `.env.local` with `CF_API_KEY`)
- `packwiz refresh` — regenerate hashes after hand-editing a `.pw.toml` (e.g., changing `side`)
- `packwiz mr export` — build the `.mrpack`

Validation (Docker Desktop must be running):

- `scripts/validate-server` — export + headless NeoForge boot + error scan + tear-down (~15–30s cached)
- `scripts/validate-server --clean` — wipe the validation volume for a pristine boot
- `scripts/validate-server --keep-up` — leave the container running for manual poking

Research / QA tooling:

- `scripts/cf-api <path> [curl args]` — thin curl wrapper with CF auth header
- `scripts/verify-versions [--mc 1.21.1]` — check `references/shortlist.txt` slugs against target MC version
- `scripts/compare-packs [--create] [--min N]` — cross-pack mod matrix from `references/*.json`
- `scripts/resolve-manifest <manifest.json>` — CF modpack manifest → per-mod metadata array
- `scripts/check-modrinth-slug < slugs.txt` — batch Modrinth slug availability (no auth)

## Architecture

- `pack/` — packwiz source. `pack.toml` pins MC + loader; `index.toml` is the manifest; `mods/*.pw.toml` are per-mod records. The `side` field (`client`/`server`/`both`) drives env flags in the exported `.mrpack`, which itzg honors to skip client-only mods on server installs.
- `server-validation/` — `docker-compose.yml` (itzg/minecraft-server, `TYPE=MODRINTH`, local mrpack mount, named volume for world data). `pack.mrpack` and `logs/` are gitignored; `scripts/validate-server` regenerates them each run.
- `scripts/` — research/QA helpers. `validate-server` is the live dev loop; the rest are research-phase tools that shouldn't be re-run casually.
- `references/` — **frozen research artifacts** (per-pack mod JSON, comparison matrix, version availability report, name candidates). Don't re-fetch unless re-verifying.
- `downloads/` — raw modpack zips (gitignored).
- `.env.local` — `CF_API_KEY` in single quotes. CF keys are bcrypt-formatted (`$2a$10$…`); unquoted `$` expands when sourced.

## Validation is the dev loop

`scripts/validate-server` catches server-side regressions (recipe JSON, KubeJS scripts, dep mismatches, registry collisions) without needing the user's Windows Prism box. Client-only issues (rendering, UI, tooltip overlays) still require a human-run client — say so explicitly rather than claiming a green validation run covers them.

Two currently-known upstream ERROR lines (`createdeco:placard` recipe parse failure, `RuntimeDistCleaner`/`AbstractClientPlayer`) are non-blocking and documented in `project_known_mod_issues.md` (auto-loaded memory). Treat a run with only those two as passing. Any *new* ERROR line is a real regression.

## Distribution plan

- **Dev / validation:** local `.mrpack` via `scripts/validate-server`.
- **Friends-server (future):** `itzg/minecraft-server` with `TYPE=MODRINTH`, pinned `MODRINTH_VERSION_ID`. Blocked on Modrinth publish.
- Repo is currently private; will be made public once a README + Modrinth listing exist.
