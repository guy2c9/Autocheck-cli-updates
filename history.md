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

---

## Current State of ~/.zshrc Wrapper

```bash
_cli_update_check() {
  local last_run_file="$HOME/.cli-update-last-run"
  local today=$(date +%Y-%m-%d)
  if [ "$(cat "$last_run_file" 2>/dev/null)" != "$today" ]; then
    curl -sL https://raw.githubusercontent.com/guy2c9/Terminal-check-cli-updates/main/check-cli-updates.sh | bash
    echo "$today" > "$last_run_file"
  fi
}

claude() {
  _cli_update_check
  cd "$HOME/Documents/Claude CLI (Warp)" && command claude "$@"
}

alias cca="claude"
```

---

## Recommendations (Pending Fixes)

### 1. `.zshrc` wrapper pipes curl directly to bash

**Issue:** The current `.zshrc` uses `curl ... | bash` which means the script runs in a subshell. If curl fails silently or GitHub is unreachable, you get no feedback — it just skips the check.

**Fix:** Download to a temp file first, check it's non-empty, then run:

```bash
_cli_update_check() {
  local last_run_file="$HOME/.cli-update-last-run"
  local today=$(date +%Y-%m-%d)
  if [ "$(cat "$last_run_file" 2>/dev/null)" != "$today" ]; then
    local tmp="/tmp/check-cli-updates.sh"
    if curl -sL https://raw.githubusercontent.com/guy2c9/Terminal-check-cli-updates/main/check-cli-updates.sh -o "$tmp" && [ -s "$tmp" ]; then
      bash "$tmp"
    else
      echo "⚠ CLI update check: could not fetch script"
    fi
    echo "$today" > "$last_run_file"
  fi
}
```

### 2. `claude()` function forces a `cd` on every invocation

**Issue:** The wrapper does `cd "$HOME/Documents/Claude CLI (Warp)"` before running Claude. This means every time you run `claude` or `cca`, your working directory changes — even if you're deliberately in another project folder.

**Fix:** Remove the `cd` so Claude launches in whatever directory you're already in:

```bash
claude() {
  _cli_update_check
  command claude "$@"
}
```

### 3. The `set -uo pipefail` in the script may cause silent failures

**Issue:** `set -u` (nounset) will error on any unset variable. Combined with various `|| true` guards, some edge cases could fail unexpectedly if a tool produces unexpected output.

**Recommendation:** Keep `set -uo pipefail` but initialise all variables with defaults at the top (this is already mostly done — just verify no edge cases).

### 4. pip3 `--break-system-packages` flag

**Issue:** Line 110 uses `--break-system-packages` which bypasses Python's externally-managed check. This is intentional but risky — a pip upgrade could break Homebrew's Python installation.

**Recommendation:** Consider whether pip packages should be managed at all, or use `pipx` for CLI tools and leave system Python alone.

### 5. README curl command differs from .zshrc

**Issue:** The README shows downloading to a file then running it, but the `.zshrc` wrapper pipes directly to bash. These should be consistent.

**Fix:** Update the `.zshrc` instructions in the README to use the download-then-run approach (see fix #1 above).

---

## Priority Order

1. **Fix #2** (remove forced `cd`) — actively changes your working directory unexpectedly
2. **Fix #1** (download-then-run) — improves reliability of the auto-check
3. **Fix #5** (README consistency) — documentation alignment
4. **Fix #4** (pip strategy) — optional, depends on your pip usage
5. **Fix #3** (variable defaults) — minor robustness improvement
