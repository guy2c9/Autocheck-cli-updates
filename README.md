# CLI Update Checker

A shell script that checks for and installs updates across all CLI applications on macOS.

## What It Checks & Updates

| Tool | Check Method | Update Command |
|---|---|---|
| **Homebrew Formulae** | `brew outdated --formula` | `brew upgrade --formula` |
| **Homebrew Casks** | `brew outdated --cask` | `brew upgrade --cask` |
| **pip3 Packages** | `pip3 list --outdated` | `pip3 install --upgrade` |
| **Python** | Compares brew formula versions | `brew upgrade python@X.XX` |
| **Salesforce CLI** | `sf update --available` | `sf update --stable` |
| **Claude Code** | `claude update` | `claude update` |
| **GitHub CLI** | Compares against latest GitHub release | `brew upgrade gh` |
| **1Password CLI** | Compares against brew cask info | `brew upgrade --cask 1password-cli` |
| **Warp Terminal** | Compares installed vs latest brew cask | `brew upgrade --cask warp` |
| **Java (Azul Zulu)** | Compares brew cask installed vs latest | `brew upgrade --cask zulu@XX` |
| **Slack CLI** | Compares against brew cask info | `brew upgrade --cask slack-cli` |
| **Google Cloud CLI** | `gcloud version` before/after | `gcloud components update --quiet` |

## Run Manually (One-Off)

```bash
curl -sL https://raw.githubusercontent.com/guy2c9/Terminal-check-cli-updates/main/check-cli-updates.sh | bash
```

## Run Automatically on Warp Launch

1. Open **Warp**
2. Go to **Settings** (Cmd + ,)
3. Navigate to **Features > Session**
4. Set the startup command to:

```
curl -sL https://raw.githubusercontent.com/guy2c9/Terminal-check-cli-updates/main/check-cli-updates.sh | bash
```

## Run Automatically on Terminal.app Launch

1. Open **Terminal**
2. Go to **Terminal > Settings** (Cmd + ,)
3. Select your profile under **Profiles**
4. Go to the **Shell** tab
5. Under **Startup > Run command**, enter:

```
curl -sL https://raw.githubusercontent.com/guy2c9/Terminal-check-cli-updates/main/check-cli-updates.sh | bash
```

## Behaviour

- Skips any tool that isn't installed — no errors, just moves to the next
- Colour-coded output: green for up to date, yellow for outdated/upgrading
- Prints a summary at the end showing the status of every detected tool
- Safe to run repeatedly — idempotent, won't break anything if already up to date

## Requirements

- macOS
- [Homebrew](https://brew.sh) (primary package manager)
