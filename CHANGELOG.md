# Microsoft 365 Reset

## Changelog

### Version 1.0.0b3 (06-Apr-2026)
- Fresh run of `scripts/mofa-consult.zsh` to sync with the latest MOFA stable feed and generate an updated local inclusion report for this repo
- Modified interactive user cancellations to exit cleanly so Jamf Pro policy logs do not report them as failures.

### Version 1.0.0b2 (31-Mar-2026)
- Added `scripts/mofa-consult.zsh` to sync a sibling `../MOFA` checkout from upstream MOFA and generate a local inclusion report for this repo
- Documented the maintainer MOFA sync/report workflow in `README.md`
- Updated the Outlook primary repair URL to match the current MOFA stable feed
- Updated MOFA consultation reports to use resolved `file://` links for sibling MOFA scripts and mark unpublished stable-feed items as `Skipped`

### Version 1.0.0b1 (29-Mar-2026)
- Initial public beta release
- Updated `promptForRestart()` to align the restart recommendation dialog with the current SYM-Lite presentation and button layout

### Version 0.0.1a9 (26-Mar-2026)
- Added per-operation icons to the interactive selection dialog
- Promoted the generic Microsoft 365 icon to the shared `applicationIcon` variable for the intro dialog and fallback operation use
- Updated README screenshots and selection UI notes for the new picker presentation

### Version 0.0.1a8 (25-Mar-2026)
- Expanded `remove_acrobat_addin` cleanup targets to cover both `Startup` and `Startup.localized` variants for Word, Excel, and PowerPoint, including both `Powerpoint` and `PowerPoint` folder names
- Wait for Word, Excel, PowerPoint, and Acrobat to quit before interactive Acrobat add-in removal; `silent` mode now force-stops those apps before cleanup

### Version 0.0.1a7 (25-Mar-2026)
- Added `remove_acrobat_addin` as a standalone ancillary removal option for Adobe Acrobat add-in payloads in Word, Excel, and PowerPoint
- Updated README operation and execution-order documentation for the new Acrobat add-in removal workflow

### Version 0.0.1a6 (25-Mar-2026)
- Updated `startProgressDialog()` to show the resolved operation titles that will actually run in interactive modes
- Wait for the background progress dialog to close before continuing
- Suppressed `swiftDialog` stderr for captured JSON dialogs

### Version 0.0.1a5 (18-Mar-2026)
- Enabled moveable and minimizable window for `startProgressDialog()`

### Version 0.0.1a4 (18-Mar-2026)
- Improved Jamf parameter handling to skip all leading positionals regardless of count

### Version 0.0.1a3 (14-Mar-2026)
- Fixed argument parsing so Jamf-style leading positional parameters no longer trigger `Unknown argument` before CLI flags are processed (Addresses [Issue #3](https://github.com/dan-snelson/Microsoft-365-Reset/issues/3); thanks for the heads-up, @eirikt!)

### Version 0.0.1a2 (13-Mar-2026)
- Aligned cleanup targets with MOFA community-maintained reset scripts
- Added OneDrive integration cleanup targets to the full Microsoft 365 removal workflow
- Added MAU `HTTPStorages` binarycookie cleanup targets during AutoUpdate reset
- Added MOFA-style factory reset cleanup behavior to `reset_factory`
- Updated app reset flows to stop after repair/reinstall instead of continuing with configuration cleanup
- Added `reset_license` to preserve MOFA's license-only reset workflow alongside `reset_credentials`
- Added `reset_teams_force` to provide a Teams force-reinstall path within the unified operation model
- Updated Teams reset to preserve custom backgrounds, reset Teams TCC state in the logged-in user context, preserve installed app bundles outside force-reinstall mode, and reopen Screen Recording settings in interactive modes
- Expanded AutoUpdate registration to include current Teams, Edge channels, and Defender ATP when present
- Fixed Microsoft package header parsing so `reset_teams_force` can validate CDN downloads correctly
- Updated progress bar behavior so a running operation no longer appears complete before it finishes
- Updated new Teams AutoUpdate registration to use the current `TEAMS21` product ID and expanded Teams process shutdown coverage

### Version 0.0.1a1 (12-Mar-2026)
- Initial unified script implementation
- Added CLI validation for missing `--mode` and `--operations` values
- Added preflight guard to require a non-root console user and block unsafe `/var/root` user-scope targeting
- Updated keychain credential removal helpers and operation flows to execute user-keychain deletes in logged-in user context
- Made package downloads deterministic by removing background transfer mode from repair downloads
- Updated README exit code documentation for `silent` mode with no selected operations
