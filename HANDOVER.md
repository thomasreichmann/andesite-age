# Andesite Age — handover

This supersedes the prior research-phase handover (see git log / commit
`5ec2d96` for that content). The pack now boots cleanly on a headless
NeoForge server; the next session is for product/scope decisions, not
plumbing.

## Read first

Project memory auto-loads from
`~/.claude/projects/-Users-thomas-projects-minecraft-create-plus/memory/`.
Indexed via `MEMORY.md`. Key ones for this session:

- `project_create_plus.md` — pack thesis and the two filter questions
  that gate every mod add (*does this trivialize Create?* / *does this
  break vanilla world feel?*)
- `user_background.md`, `feedback_open_to_forking.md`

Known upstream errors (`createdeco:placard` recipe, DistCleaner
AbstractClientPlayer) now live in
`server-validation/known-issues.txt` — the validation script
allowlists them automatically and reports stale entries if they stop
firing.

## Where things stand

- Repo: <https://github.com/thomasreichmann/andesite-age> (private; will
  be made public once a README is written and a Modrinth listing exists).
- Pack: MC 1.21.1, NeoForge `21.1.227`, packwiz-managed in `pack/`.
- Tier 1 (Create core + must-have addons) and Tier 2 (EMI, Jade,
  KubeJS, Sodium) are installed. Transitive deps: Create: Dragons Plus,
  Rhino.
- EMI is `side = "client"`. Sodium was already. Jade intentionally left
  `both` for multiplayer info parity.
- Client smoke-test passed in Prism on Windows (user's machine).
- Headless validation passes: server ready in ~15–30s, only the two
  known upstream errors.

## Validation pipeline

`scripts/validate-server` is the dev loop. It:

1. `scripts/check-mod-manifest` — diffs current `pack/mods/*.pw.toml`
   against `server-validation/expected-mods.tsv` and asserts MC +
   NeoForge version pins in `pack/pack.toml`.
2. Hashes pack source; runs `packwiz refresh` + `packwiz mr export`
   only if the hash changed (reuses `server-validation/pack.mrpack`
   otherwise — `.last-export-hash` is the cache).
3. `scripts/check-sidedness` — confirms every `.pw.toml` mod is tagged
   in the exported `modrinth.index.json` with `env.client`/`env.server`
   matching its `side` field.
4. `docker compose up -d` in `server-validation/` (itzg +
   `TYPE=MODRINTH`; RCON enabled with an ephemeral local password).
5. Polls `rcon-cli list` every 2s up to 600s — readiness signal is
   "commands accepted", not the log marker (which fires too early on
   modded boots).
6. Runs post-boot probes: `rcon-cli reload` (re-evaluates datapacks,
   surfaces recipe/tag parse failures) and `rcon-cli forceload add ~ ~`
   (exercises chunk-gen).
7. Dumps logs (only since this boot, via `--since`), single-pass awk
   split into ERROR/FATAL, WARN-tier recipe/tag/advancement failures,
   and `Caused by:` hints. Dedupes in the same pass.
8. Subtracts matches against `server-validation/known-issues.txt`
   (regex allowlist). Prints a "stale allowlist" warning for patterns
   that matched nothing.
9. Checks `/data/crash-reports` in the container — any file there is a
   hard failure.
10. Teardown is backgrounded (`docker compose stop &`) so the next run
    waits for stop implicitly. `--keep-up` leaves it running;
    `--clean` does `down -v` up front.

Exit 0 iff: ready, no unallowed ERROR/FATAL, no WARN-tier parse
failures, no crash reports. Any *new* error line is a real regression.

**Requires Docker daemon running.** On macOS: Docker Desktop must be
launched.

## Distribution plan (locked in this session)

- **Dev / validation:** local mrpack via `scripts/validate-server`.
- **Friends-server:** `itzg/minecraft-server` with `TYPE=MODRINTH`,
  `MODRINTH_PROJECT=andesite-age`, pinned `MODRINTH_VERSION_ID` (pin
  explicitly so Modrinth updates don't break a live session; bump
  deliberately). **Blocked on Modrinth publish** — Modrinth listing
  must be live before the friends-server stands up.
- **GitHub Releases** as an intermediate distribution step: skipped.
  Publishing to Modrinth first makes it redundant.

## Open questions for this session

These deferred from the previous handover, still unanswered:

1. **Tier 3 — vanilla+ gap-fillers.** Only `farmers-delight` is a
   confirmed user pick. Candidates to discuss: Sophisticated
   Backpacks, Waystones, Supplementaries, graves mod. User's stated
   constraint is that *world feel* stays vanilla — inventory/QoL is a
   separate allowance.
2. **Tier 4 — flavor mods.** From the verified shortlist:
   `create-jetpack`, `create-big-cannons`, `create-liquid-fuel`,
   `create-confectionery`, `interiors`, `rechiseled-create`,
   `chisels-bits`, `ars-nouveau` + `ars-creo`. Which actually fit the
   friend group's vibe?
3. **Perf stack beyond Sodium.** Want NeoForge staples like
   `ferritecore`, `modernfix`, `embeddium-extras`? These need a
   verification pass against 1.21.1 NeoForge.
4. **Server-side specifics.** Any server-only mods worth considering
   (performance, admin tools)? Anything in the current pack that
   should be flagged client-only but isn't?

## Pending operational tasks

- [ ] Reserve Modrinth slug `andesite-age`. Requires a Modrinth PAT —
  not set up. Ask the user. Slug was confirmed available
  (`scripts/check-modrinth-slug`) as of 2026-04-19; re-confirm before
  claiming.
- [ ] Write the README. Needed to make the repo public and to seed the
  Modrinth description.
- [ ] Publish first version on Modrinth (can be unlisted/draft).
- [ ] Make the repo public.
- [ ] Stand up the friends-server (itzg docker compose pointing at the
  Modrinth project).

## Things NOT to redo

- Research on the reference packs, the 1.21.1 NeoForge shortlist, the
  pack name, or the loader/MC version choice — all locked.
- The validation pipeline design. It works; extend it if needed, don't
  rewrite.
- The two known upstream errors. See
  `server-validation/known-issues.txt`.
