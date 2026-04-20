# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

An authored Minecraft 1.21.1 NeoForge modpack ("Andesite Age") managed with **packwiz** — not a conventional codebase. Source of truth is the packwiz metadata under `pack/`; the `.mrpack` is build output. Pack thesis and mod-vetting filters live in auto-loaded memory (`project_create_plus.md`). Current state and open questions: `HANDOVER.md`.

## Commands worth knowing

- `cd pack && packwiz refresh` — regenerate hashes after hand-editing a `.pw.toml` (e.g., changing `side`). This is the non-obvious step; stock `packwiz add / export / mr / cf` subcommands are documented by `packwiz --help`.
- `scripts/validate-server` — the live dev loop. Flags: `--clean` (wipe validation volume), `--keep-up` (leave container running). Requires Docker Desktop running.
- Other `scripts/*` have self-documenting usage in their header comments; read them when needed. Don't re-run the research-phase ones (`cf-api`, `verify-versions`, `compare-packs`, `resolve-manifest`, `check-modrinth-slug`) casually — they were used to freeze `references/`.

## Architecture

- `pack/` — packwiz source. `mods/*.pw.toml` are per-mod records; the `side` field (`client`/`server`/`both`) drives `env.client`/`env.server` flags in the exported `.mrpack`, which `itzg/minecraft-server` honors to auto-skip client-only mods on server installs. This is the cross-system glue that binds the packwiz side tag to itzg's install behavior.
- `server-validation/` — `docker-compose.yml` for the headless validation server. `pack.mrpack` and `logs/` are regenerated each run; don't hand-edit them.
- `references/` — **frozen research artifacts**. Don't re-fetch unless re-verifying against a new MC version.
- `downloads/` — raw modpack zips (gitignored).

## Validation is the dev loop

`scripts/validate-server` catches server-side regressions (recipe JSON, KubeJS scripts, dep mismatches, registry collisions) without needing the user's Windows Prism box. It does not cover client-only concerns — rendering, UI, tooltip overlays need a human client run.

Known upstream non-blocking errors are documented in `project_known_mod_issues.md` (auto-loaded). A run surfacing only those is passing; any new ERROR line is a real regression.

## Distribution plan

- **Dev / validation:** local `.mrpack` via `scripts/validate-server`.
- **Friends-server (future):** `itzg/minecraft-server` with `TYPE=MODRINTH`, pinned `MODRINTH_VERSION_ID`. Blocked on Modrinth publish.
- Repo is currently private; public once a README + Modrinth listing exist.
