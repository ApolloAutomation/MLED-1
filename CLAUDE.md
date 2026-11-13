# Apollo MLED-1: rules for AI agents and contributors

This repository is Apollo Automation's MLED-1 firmware: upstream WLED (16.0.x release line)
plus the MLED-1 changes. WLED's coding conventions are in AGENTS.md (inherited from upstream).
The Apollo rules below take precedence over anything inherited from upstream, including
AGENTS.md and .github/copilot-instructions.md.

## Never touch upstream WLED

- Never push to, open pull requests or issues against, or comment on wled/WLED (formerly
  Aircoookie/WLED). Nothing goes upstream.
- This repository is NOT a GitHub fork (GitHub does not allow a private fork of a public
  repo). It is a standalone repo seeded with upstream history up to the v16.0.1 tag, so
  upstream cannot see it and it never appears in wled/WLED's fork network.
- Pass `-R ApolloAutomation/MLED-1` on every gh command that writes.
- AGENTS.md says pull requests target upstream's `main`. Here, pull requests target this
  repository's `main`.

## Local clone setup (do this in every clone)

    git remote add wled https://github.com/wled/WLED.git
    git config remote.wled.pushurl DISABLED_never_push_to_upstream
    git config remote.wled.tagopt --no-tags
    git config --replace-all remote.wled.fetch '+refs/tags/*:refs/remotes/wled-tags/*'
    git config --add remote.wled.fetch '+refs/heads/16_x:refs/remotes/wled/16_x'
    gh repo set-default ApolloAutomation/MLED-1

- The upstream remote is named `wled`, not `upstream` (gh prefers a remote named upstream).
- Upstream tags live only under `refs/remotes/wled-tags/`. `git tag -l` must stay free of
  upstream tags, so a stray `git push --tags` can never publish them. Never run
  `git push --tags` or `git push --mirror`.

## Taking upstream changes

- Merge an upstream release tag (for example `refs/remotes/wled-tags/v16.0.2`) on a sync
  branch, then open a PR into `main`. Never merge upstream `main` (17.0.0-dev on ESP-IDF 5)
  or the `16_x` branch tip; single fixes from 16_x come in as `git cherry-pick -x`.
- Merge commits only (squash and rebase are disabled): squashing a sync rewrites upstream
  history and breaks the next 3-way merge.
- On every sync, re-delete any upstream workflow that comes back under .github/workflows/
  (nightly, release, stale, CI). They burn private-repo Actions minutes, and release.yml
  fires on any tag push.
- Never rebase or force-push `main`.

## Building and shipping

- Build env: `apollo_mled1` in platformio.ini (pure delta on upstream `esp32dev_8M`).
  Needs `npm ci` once and `export npm_config_cache=<writable dir>`.
- Keep `WLED_RELEASE_NAME` = `Apollo_MLED-1` forever. WLED rejects OTA uploads whose
  release name differs, so changing it strands every fielded unit.
- Factory flashing must use a full erase (`write_flash --erase-all 0x0 <full_install.bin>`).
  Compiled defaults (data pin, hotspot) only apply when no saved config exists. Any
  settings save during QC writes cfg.json and locks that unit's config.
- Do not use `-D LEDPIN` (removed upstream) or `-D DEFAULT_LED_COUNT` / `DEFAULT_LED_TYPE` /
  `DEFAULT_COLOR` (unguarded in headers, silently ignored). Use DATA_PINS, PIXEL_COUNTS,
  LED_TYPES.
- Strings with spaces in build_flags use the `'"text"'` form; `\"text\"` splits on spaces.

## Licence (EUPL-1.2)

- Handing even one unit to anyone outside Apollo (content creators, beta testers, retail)
  is distribution: the exact source must go with it, or this repo must be public.
- Keep LICENSE, NOTICE and all upstream copyright notices. Credit WLED in the README.

Decisions and open questions live in DECISIONS.md.
