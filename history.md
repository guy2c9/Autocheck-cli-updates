# CLI Update Checker — History & Recommendations

## Session History

### Session 1 — 2026-04-01 (Initial Build)

**Commits made:**

1. `5bdf2c7` — Initial commit: CLI update checker script and README
2. `9b5ac65` — Detect latest Zulu version dynamically instead of hardcoding zulu@17
3. `1da265a` — Update README with run instructions for Warp and Terminal
4. `80b6b43` — Fix Warp instructions — use ~/.zshrc instead of non-existent startup command setting
5. `0b88ae6` — Show existing JDK installations and uninstall steps before suggesting Zulu install
6. `69d1bd1` — Rewrite README with detailed auto-run setup instructions for Warp and Terminal
7. `7c4b315` — Improve setup instructions with Warp-specific details and simpler save flow

**What was built:**

- `check-cli-updates.sh` — checks and updates 12 CLI tools (Homebrew, pip3, Python, SF CLI, Claude Code, GitHub CLI, 1Password CLI, Warp, Java/Zulu, Slack CLI, Google Cloud CLI)
- Colour-coded output with summary table
- Auto-run wrapper in `~/.zshrc` that triggers once per day on first `claude`/`cca` invocation

### Session 2 — 2026-04-02 (Review & Fixes)

**Commits made:**

8. `59c93c6` — Fix curl-pipe-to-bash causing script to terminate early
9. `a30eb74` — Rename repo references from Terminal-check-cli-updates to Autocheck-cli-updates
10. `b6af6e6` — Remove pip auto-upgrade, fix forced cd, align README with script
11. `69ac4b7` — Add sf plugins update to Salesforce CLI section
12. `8c57d47` — Add reset-and-run command to README

**What was fixed:**

- Replaced `curl | bash` with download-to-file in `.zshrc` wrapper (fixed summary table not showing)
- Removed forced `cd` from `claude()` wrapper — now launches in current directory
- Replaced pip3 auto-upgrade with report-only (removed `--break-system-packages` risk)
- Fixed README table to match actual SF CLI behaviour
- Updated all repo URL references to Autocheck-cli-updates
- Added `sf plugins update` to Salesforce CLI section
- Added reset-and-run command to README for manual testing

### Session 3 — 2026-04-03 (Add Playwright CLI)

**Commits made:**

13. `0de3beb` — Add Playwright CLI update check

**What was added:**

- Playwright CLI section: checks current version via `npx playwright --version`, compares against npm registry (`npm view @playwright/test version`), updates via `npm install -g @playwright/test@latest` + `npx playwright install` for browsers
- README table updated with Playwright row

### Session 4 — 2026-04-05 (Add Codex & Gemini CLIs)

**Commits made:**

14. `f26f2cf` — Add Codex CLI and Gemini CLI update checks
15. `e8f1198` — Add codex and gemini as trigger commands for daily update check

**What was added:**

- Codex CLI section (OpenAI): checks current version via `codex --version`, compares against npm registry (`npm view @openai/codex version`), updates via `npm install -g @openai/codex@latest`
- Gemini CLI section (Google): checks current version via `gemini --version`, compares against npm registry (`npm view @google/gemini-cli version`), updates via `npm install -g @google/gemini-cli@latest`
- README table updated with both new rows
- `codex()` and `gemini()` wrapper functions added to `.zshrc` setup — daily update check now triggers on `claude`, `cca`, `codex`, or `gemini`
- Tool count now 15

---

## Current State of ~/.zshrc Wrapper

```bash
_cli_update_check() {
  local last_run_file="$HOME/.cli-update-last-run"
  local today=$(date +%Y-%m-%d)
  if [ "$(cat "$last_run_file" 2>/dev/null)" != "$today" ]; then
    local tmp="/tmp/check-cli-updates.sh"
    if curl -sL https://raw.githubusercontent.com/guy2c9/Autocheck-cli-updates/main/check-cli-updates.sh -o "$tmp" && [ -s "$tmp" ]; then
      bash "$tmp" && echo "$today" > "$last_run_file"
    else
      echo "⚠ CLI update check: could not fetch script"
    fi
  fi
}

claude() {
  _cli_update_check
  command claude "$@"
}

codex() {
  _cli_update_check
  command codex "$@"
}

gemini() {
  _cli_update_check
  command gemini "$@"
}

alias cca="claude"
```

### Session 5 — 2026-04-05 (Code Quality & Safety)

**Commits made:**

16. `3be075c` — Improve code quality: refactor helpers, fix safety issues, add CI
17. `bc1df69` — Fix false status reporting and add post-update verification

**What was changed (commit 16):**

- Extracted `check_npm_tool` helper — Codex, Gemini, and Playwright now share one function instead of 3 copy-pasted blocks
- Extracted `check_brew_cask_tool` helper — 1Password and Slack CLI share one function instead of 2 copy-pasted blocks
- Extracted `brew_json_field` helper — replaces `python3 -c` JSON parsing with `grep`/`sed` (no new dependency needed)
- Replaced automatic `sudo rm -rf` of legacy JDKs with report-only output (prints manual removal commands instead)
- Fixed unquoted variables in `for` loops by using properly quoted arrays via `read` loops (ShellCheck compliance)
- Removed unused status variables to reduce dead code
- Added `.gitignore` (`.DS_Store`, editor swap files)
- Added MIT `LICENSE`
- Added `.github/workflows/lint.yml` — ShellCheck CI on push/PR to main

**What was changed (commit 17):**

- Registry/API lookup failures (npm, brew cask, GitHub API) now report "Check failed" instead of falsely showing "up to date"
- All update commands (`npm install -g`, `brew upgrade --cask`, `brew upgrade gh`) now verify the version changed post-install — reports "Update failed" if version is unchanged
- Summary table renders "Check failed" and "Update failed" statuses in red
- `.zshrc` wrapper (README + history) now only stamps the day-file on successful script execution — a failed curl or crashed script will retry next invocation instead of silently skipping for the rest of the day

**No user-facing changes** — script path, invocation, output format, and `.zshrc` setup are all unchanged. Existing users who update their `.zshrc` wrapper get the retry-on-failure improvement, but the old wrapper continues to work.

---

## Recommendations

### Resolved (Session 2 — 2026-04-02)

1. ~~`.zshrc` wrapper pipes curl directly to bash~~ — **Fixed.** Changed to download-to-file with validation
2. ~~`claude()` function forces a `cd` on every invocation~~ — **Fixed.** Removed `cd`, launches in current directory
3. ~~README curl command differs from .zshrc~~ — **Fixed.** Both now use download-to-file approach
4. ~~pip3 `--break-system-packages` auto-upgrade~~ — **Fixed.** Changed to report-only, no longer auto-upgrades
5. ~~README SF CLI commands don't match script~~ — **Fixed.** Table now matches actual behaviour

### Resolved (Session 5 — 2026-04-05)

6. ~~Legacy JDK removal uses `sudo rm -rf` which silently fails without sudo~~ — **Fixed.** Changed to report-only with manual removal commands
7. ~~Duplicated tool-check blocks across Codex/Gemini/Playwright/1Password/Slack~~ — **Fixed.** Extracted `check_npm_tool` and `check_brew_cask_tool` helpers
8. ~~`python3 -c` JSON parsing fragile if Python unavailable~~ — **Fixed.** Replaced with `grep`/`sed` via `brew_json_field` helper
9. ~~Unquoted variables in word-splitting contexts~~ — **Fixed.** All path loops now use quoted arrays
10. ~~No CI pipeline~~ — **Fixed.** Added GitHub Actions ShellCheck workflow
11. ~~No license file~~ — **Fixed.** Added MIT LICENSE
12. ~~Registry/API lookup failures silently collapse to "up to date"~~ — **Fixed.** Now reports "Check failed" with reason
13. ~~Failed updates reported as successful (no post-update verification)~~ — **Fixed.** All helpers now re-check version after install
14. ~~`.zshrc` wrapper stamps date even on fetch/script failure~~ — **Fixed.** Day-stamp only written on successful run

### Remaining

- **`set -uo pipefail`** — `set -u` could abort on unset variables in edge cases. Variables are mostly initialised but worth verifying if new tool sections are added
