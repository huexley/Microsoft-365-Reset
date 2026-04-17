# Microsoft 365 Reset — HEP Vaud fork

A French-localized fork of [Dan Snelson's Microsoft-365-Reset](https://github.com/dan-snelson/Microsoft-365-Reset), adapted for the HEP Vaud (Haute École Pédagogique Vaud) Mac fleet and deployed via Jamf Pro.

All the heavy lifting — the swiftDialog UI, the MOFA-based operation set, the dependency resolver, the execution pipeline — comes from Dan. This fork only carries local adaptations that are not in scope for the upstream project: French translation of every user-facing string, removal of operations that don't apply to our environment, and a new `creation_poste_examen` operation that hardens a Mac before high-stakes written exams.

The canonical upstream remains [`dan-snelson/Microsoft-365-Reset`](https://github.com/dan-snelson/Microsoft-365-Reset). If you are not French-speaking and do not have the HEP Vaud-specific needs described below, you almost certainly want upstream instead of this fork.

---

## Relationship to upstream

This fork tracks `dan-snelson/Microsoft-365-Reset` and applies six numbered patches on top of each upstream release. The patches are documented in the script's `HISTORY` header and summarized below.

| # | Patch | Summary |
| --- | --- | --- |
| 1 | **French localization** | Every user-facing string is centralized in `STR_*` variables near the top of the script and translated to French. |
| 2 | **Auto-repair disabled by default** | `autoRepairEnabled="false"` at script start. An optional dialog lets a technician opt-in at runtime. Reinstallation is otherwise handled by a separate Jamf policy using the HEP-packaged installer. |
| 3 | **Log out instead of Restart** | Shared lab Macs use `restartMode="Logout"` — clearing user-scoped Office state is enough; a full restart is disruptive for the next session. |
| 4 | **Intro splash dialog removed** | The Self Service context already conveys intent. |
| 5 | **Operations removed** | `remove_skypeforbusiness`, `remove_defender`, `reset_autoupdate` are stripped — Skype for Business is decommissioned, Defender is not deployed at HEP Vaud, and MAU is managed centrally. |
| 6 | **New operation: `creation_poste_examen`** | Prepares a Mac for a controlled-environment written exam by removing Word's proofing tools and freezing Microsoft AutoUpdate. See [The exam station operation](#the-exam-station-operation) below. |

---

## Requirements

- macOS 13 Ventura or later (macOS 14+ recommended)
- Root execution (`sudo` or a Jamf Pro policy running as root)
- [swiftDialog](https://github.com/swiftDialog/swiftDialog) 3.0.0.4952 or later — the script will auto-install or auto-upgrade it in interactive modes
- A logged-in console user (the script refuses to run user-scoped operations without one)
- Network access to Microsoft package hosts *only* if auto-repair is enabled at runtime (disabled by default in this fork)

Matches the upstream requirement set; nothing added here.

---

## Installation

### Standalone (CLI)

```bash
sudo curl -o /usr/local/bin/Microsoft-365-Reset.zsh \
  https://raw.githubusercontent.com/huexley/Microsoft-365-Reset/hep-vaud/fr-localization/Microsoft-365-Reset.zsh
sudo chmod 755 /usr/local/bin/Microsoft-365-Reset.zsh
```

### Jamf Pro

1. Upload `Microsoft-365-Reset.zsh` to **Settings → Computer Management → Scripts**.
2. Create a policy scoped to your target group, with the script executed **After** the install phase and running **as root**.
3. Populate Jamf parameters 4 and 5 for the execution mode:

| Jamf parameter | Meaning | Accepted values |
| --- | --- | --- |
| `$4` | Operation mode | `self-service` (default), `silent`, `test`, `debug` |
| `$5` | Pre-selected operations (CSV, optional) | e.g. `reset_outlook,reset_teams` |

4. (Recommended) Add a Self Service category and a user-facing description in French.

---

## Usage

```
sudo Microsoft-365-Reset.zsh [--mode MODE] [--operations CSV]
```

| Mode | Behavior |
| --- | --- |
| `self-service` | Full interactive flow: swiftDialog operation picker, confirmation dialog for destructive choices, progress HUD, completion summary. This is the default. |
| `silent` | No UI. `--operations` is required. Intended for automated Jamf triggers. |
| `test` | Same UI as `self-service`, but all destructive actions are logged and skipped. Safe to run on a production Mac. |
| `debug` | Verbose tracing (`set -x`). For script development only. |

CLI flags override Jamf parameters when both are set. Positional arguments before the first `--` flag are tolerated so the script can be called directly from a Jamf policy without extra wrapping.

### Silent example (used by our Jamf exam-station workflow)

```bash
sudo Microsoft-365-Reset.zsh --mode silent --operations creation_poste_examen
```

### Self Service example (what the user sees)

A technician runs the policy from Self Service; they are presented with a French operation picker with switch-style checkboxes, confirm their selection, optionally opt-in to auto-repair, and watch a progress HUD until completion. At the end, the recommended action is **Log out** (not Restart).

---

## Operation catalog

All French titles are surfaced verbatim in the Self Service UI. Descriptions are also displayed inline below each switch.

| Operation ID | French title (what the user sees) |
| --- | --- |
| `reset_factory` | Réinitialisation complète des apps Office |
| `reset_word` | Réinitialiser Word |
| `reset_excel` | Réinitialiser Excel |
| `reset_powerpoint` | Réinitialiser PowerPoint |
| `reset_outlook` | Réinitialiser Outlook |
| `remove_outlook_data` | Supprimer les données locales Outlook |
| `reset_onenote` | Réinitialiser OneNote |
| `remove_onenote_data` | Supprimer les données locales OneNote |
| `reset_onedrive` | Réinitialiser OneDrive |
| `reset_teams` | Réinitialiser Teams |
| `reset_teams_force` | Réinitialiser Teams (réinstallation forcée) |
| `reset_license` | Réinitialiser la licence uniquement |
| `reset_credentials` | Réinitialiser la licence et les identifiants |
| `creation_poste_examen` | **Création poste examen** (HEP Vaud only) |
| `remove_office` | Supprimer complètement Microsoft 365 |
| `remove_acrobat_addin` | Supprimer le module Acrobat pour Office |
| `remove_zoomplugin` | Supprimer le plugin Zoom pour Outlook |
| `remove_webexpt` | Supprimer WebEx Productivity Tools |

Dependency resolution, suppression rules, and execution order follow the upstream behavior documented in [Dan Snelson's README](https://github.com/dan-snelson/Microsoft-365-Reset). The only addition is that `creation_poste_examen` is ordered in the "ancillary removals" phase, before any full Office removal.

---

## The exam station operation

`creation_poste_examen` ("Exam station creation") is the one operation that is fully HEP-specific. It prepares a Mac for a controlled-environment written exam that runs inside Microsoft Word.

### What it does

1. **Removes Word's Proofing Tools bundle.**
   `/Applications/Microsoft Word.app/Contents/SharedSupport/Proofing Tools` contains the spelling, grammar, hyphenation, and thesaurus dictionaries for every language Word supports on macOS. Removing this folder disables the red and blue underlines and the entire Spelling & Grammar pane — the student gets a plain word processor.

2. **Freezes Microsoft AutoUpdate (MAU).**
   A `chmod 000` is applied to `/Library/Application Support/Microsoft/MAU2.0`. This is a blunt but reliable block: MAU cannot read its own bootstrapper, so no Office component can silently update during the exam window. The lock survives reboot, user switching, and Office relaunch, and cannot be undone from a standard user session.

### Why this combination

Proofing Tools are trivially restored by any MAU run. Freezing MAU is what makes step 1 persist for the duration of the exam. Both steps are needed; neither is sufficient alone.

### When to use it

Run it before a written or practical exam on a pooled HEP Vaud Mac, typically as part of an imaging or relocation workflow. It is **not** meant for day-to-day Self Service — deploy it via a tech-only Jamf policy scoped to exam-room smart groups.

### Reverting an exam station

The operation is fully reversible and touches no user data. Use a companion `release_poste_examen` Jamf policy that does:

```bash
/bin/chmod 755 "/Library/Application Support/Microsoft/MAU2.0"
```

...and then triggers your standard Microsoft 365 reinstall policy to restore Proofing Tools. Writing that companion policy is left as an exercise for the reader; it's two lines.

### Safety notes

- Idempotent: re-running on an already-prepared Mac is a no-op.
- Does not kill running apps. If Word is open, run `reset_word` first (the upstream workflow already handles this if both are selected).
- No user data is touched at any point.

---

## Logging

All output goes to `/var/log/org.churchofjesuschrist.log` (the path is inherited from upstream and not changed by this fork to keep log collection workflows compatible). Entries are tagged with a severity prefix (`PRE-FLIGHT`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `FATAL ERROR`).

For a quick tail during testing:

```bash
tail -f /var/log/org.churchofjesuschrist.log
```

---

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | All selected operations completed successfully |
| `2` | Silent mode was requested with no operations selected |
| `10` | Invalid argument, invalid mode, or fatal preflight failure |
| `20` | One or more operations failed (or `remove_office` preinstall failed) |

Non-zero exits are surfaced to Jamf Pro as policy failures, which is the intended behavior for reporting in the dashboard.

---

## Development and contribution

This is a downstream fork maintained for HEP Vaud's internal needs. External contributions are welcome for bug fixes in HEP-specific code (patches 1–6), but feature requests and general improvements should go upstream to `dan-snelson/Microsoft-365-Reset` — they will reach this fork automatically on the next sync.

### Branch layout

| Branch | Purpose |
| --- | --- |
| `main` | Mirror of `upstream/main`. Never committed to directly. |
| `hep-vaud/fr-localization` | Active HEP Vaud branch. All local patches live here. Rebased on `main` after each upstream sync. |

### Syncing with upstream

```bash
git checkout main
git fetch upstream
git merge --ff-only upstream/main
git push origin main

git checkout hep-vaud/fr-localization
git rebase main
# resolve conflicts, re-test, then:
git push --force-with-lease origin hep-vaud/fr-localization
```

---

## Credits

- Original author: **Dan K. Snelson** ([@dan-snelson](https://github.com/dan-snelson)) — [snelson.us](https://snelson.us/)
- Upstream project: [`dan-snelson/Microsoft-365-Reset`](https://github.com/dan-snelson/Microsoft-365-Reset)
- Built on the [MOFA community](https://github.com/MOFA-Tools) reset scripts and [Paul Bowden's Office-Reset.com](https://office-reset.com/macadmins/) toolkit
- swiftDialog UI: [bartreardon/swiftDialog](https://github.com/swiftDialog/swiftDialog)
- HEP Vaud fork: **Yannick** ([@huexley](https://github.com/huexley)) — Unité Informatique, HEP Vaud

## License

Inherits the upstream license. See [`LICENSE`](LICENSE) in the upstream repository.

## Support

This fork is provided as-is, without warranty. For upstream behavior, use the [Mac Admins Slack](https://www.macadmins.org/) or open an issue on the upstream repository. For HEP Vaud-specific patches (1–6), open an issue on this fork.
