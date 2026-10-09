# Steam Achievement Manager

**This is [Endymi0n74's fork](https://github.com/Endymi0n74/SteamAchievementManager) of
[gibbed/SteamAchievementManager](https://github.com/gibbed/SteamAchievementManager).**

📦 **[Download the latest release](https://github.com/Endymi0n74/SteamAchievementManager/releases/latest)**
· 🇫🇷 **[README en français](README.fr.md)**

Steam Achievement Manager (SAM) is a lightweight, portable application used to manage
achievements and statistics in the popular PC gaming platform Steam. This application
requires the [Steam client](https://store.steampowered.com/about/), a Steam account and
network access. Steam must be running and the user must be logged in.

Current release: **7.1.0** (upstream open-source releases stop at 7.0.x).

## Why a fork?

Upstream is in maintenance freeze: pull requests and issue fixes pile up without being
merged. This fork collects the stability work needed to keep SAM usable today, ships it
as a regular GitHub release, and adds **automatic updates** so you do not have to watch
the repository.

SAM still only talks to your **local Steam client**. The only thing the fork downloads
from GitHub is its own update package.

## What's new in 7.1.0 (versus upstream 7.0.x)

### Automatic updates

`SAM.Picker` checks this repository's releases when it starts and offers to install
anything newer:

- version and archive come from this repository's GitHub releases;
- the archive is verified against the published **SHA-256 before anything is touched**;
- a helper process waits for `SAM.Picker` (and `SAM.Game`, if open) to exit, extracts
  over the application folder — refusing to write outside of it — and restarts the tool;
- a failure shows what went wrong instead of leaving a half-updated install.

A **Check for updates** button in the toolbar runs the same check on demand. The update
only replaces local files: nothing in Steam (achievements or statistics) is read or
written by it.

### Fixes

- **#593 / #594 / #625 — “half of the games show no achievement”**: the bare
  “Failed to load schema.” now explains itself (schema missing from Steam's appcache and
  how to make Steam fetch it, unreadable file, or no statistics for this app).
- **#466 — icons behind a proxy**: `SAM.Game` built its icon downloader without the
  TLS 1.2 default and the authenticated proxy credentials the picker uses, so achievement
  icons silently never downloaded. It now builds the downloader exactly like the picker.
- **#468 / #410 / #581 / #592 — “everything is owned”**: when a licence faking tool
  makes Steam claim more than 5000 owned games, the status bar now says the list is
  meaningless.
- **#405 / #458 / #429 / #599 — store results**: success or failure comes from Steam's
  own `UserStatsStored` callback (15 s timeout) instead of being assumed, and a failed
  store keeps your edits instead of discarding them.
- **#380 / #432 — stat constraints**: minimum/maximum, increment-only and maximum-delta
  rules from the Steam schema are enforced and explained, and average-rate statistics are
  flagged because Steam only accepts session updates for them.
- **#495 — sorting**: clicking a column header sorts achievements by name, unlock state
  or unlock time.
- **#531 — display language**: a Language selector in the toolbar reads achievements and
  statistics in any language the game actually ships (Steam's per-game language is the
  default, not the language of the Steam client), and the status bar states which
  language is in use.
- **#491 / #435 / #424 / #579 — silent startup failures**: missing Steam interfaces,
  callback failures and unhandled errors now show which step failed instead of closing
  without a word.
- **#601 / #466 / #618 — game list download**: TLS 1.2 forced, proxy credentials
  honoured, readable download errors, and the list/logo/search races fixed.
- **#421 — “Invalid value” on untouched rows**: statistic values handed back as numbers
  are accepted again, and unsupported statistics report which stat is at fault instead of
  throwing.
- Schema files with duplicate keys or unknown stat types no longer abort the load,
  pending edits survive a failed store, and the game list no longer stops half-way
  through its refresh (an error dialog on every start, “Refresh Games”/“Add Game”
  stuck disabled).
- Building the whole solution works with a plain `dotnet build SAM.sln -c Release`
  (no MSBuild workaround needed).

## Download and install

1. Download `SteamAchievementManager-7.1.0.zip` from the
   [releases page](https://github.com/Endymi0n74/SteamAchievementManager/releases/latest).
2. Extract it wherever you keep SAM.
3. Run `SAM.Picker.exe`.

The `.sha256` file published next to the archive is what the updater verifies against —
keep it next to the zip if you want the automatic path to work. There is nothing else to
install: SAM is portable (.NET Framework 4.8 is part of Windows 10/11).

## Building

```powershell
dotnet build SAM.sln -c Release
```

The solution only has `x86` configurations (the game communicates with the 32-bit Steam
client), so there is no `Any CPU` build. Binaries are written to `upload\`.

## Tests

Unit tests live in `SAM.Game.Tests` (xunit) and cover the pure logic: schema (KeyValue)
parsing, statistic constraints, the Steam error messages and the update logic.

```powershell
dotnet test SAM.sln -c Release
```

## Scope of this fork

- Fixes and features are developed and published **here**; only three pull requests have
  been proposed upstream: [#643](https://github.com/gibbed/SteamAchievementManager/pull/643)
  (build and fixes), [#644](https://github.com/gibbed/SteamAchievementManager/pull/644)
  (tests), [#645](https://github.com/gibbed/SteamAchievementManager/pull/645) (auto-update).
- Version numbering follows the fork: **7.1.0** means “7.0.x plus the changes below”.
- The window titles derive from the assembly version, so they can no longer go stale.

## Attribution and license

- Original project: [gibbed/SteamAchievementManager](https://github.com/gibbed/SteamAchievementManager)
  by Rick (gibbed), released under the [zlib license](LICENSE.txt). This fork keeps that
  license and marks itself as an altered version, as the license requires.
- Most (if not all) icons are from the [Fugue Icons](https://p.yusukekamiyamane.com/) set.
