# AGENTS.md

Guidance for coding agents working in this repository.

## Project Context

- Project: Microsoft 365 Reset
- Primary artifact: `Microsoft-365-Reset.zsh`
- Package-era behavior reference: `Resources/Microsoft_Office_Reset_2.0.0b1_expanded/`
- Primary docs: `README.md` and `CHANGELOG.md`

## Mission

Microsoft 365 Reset should provide a safe, clear, swiftDialog-driven workflow to repair, reset, or remove Microsoft 365 components on macOS while preserving parity with the original package workflows where intended.

## Product Boundaries

### In Scope

- Microsoft 365 reset/repair/removal workflows on macOS
- swiftDialog-driven user flow (`self-service`, `test`, `debug`)
- silent automation flow (`silent`)
- deterministic operation ordering and dependency resolution
- structured logging and predictable exit codes

### Out of Scope

- non-macOS support
- broad architectural rewrites unless explicitly requested
- adding production dependencies without explicit user approval
- changing operation semantics without documenting and confirming intent

## Implementation Priorities

1. Treat current MOFA behavior as the primary parity baseline for reset and removal workflows.
2. Use the package-era reference to preserve retained chooser logic, dependency relationships, and legacy coverage where MOFA does not provide a current equivalent.
3. Preserve operator and end-user clarity in dialog text and warnings.
4. Keep changes minimal, targeted, and safe.
5. Maintain deterministic execution and reliable failure handling.
6. Keep docs synchronized with script behavior.

## Behavior Precedence

- Prefer MOFA behavior over package-era behavior by default.
- Use the package-era reference as secondary context unless MOFA does not cover the behavior in question.
- Keep any divergence from MOFA only when there is a defensible product, safety, platform, or workflow reason.
- When diverging from MOFA, document the reason in `README.md` and call out the parity impact in change notes or review summaries.

## Key Files

- `Microsoft-365-Reset.zsh`: main script
- `scripts/mofa-consult.zsh`: maintainer helper for MOFA sync and inclusion reporting
- `README.md`: usage and behavior documentation
- `CHANGELOG.md`: release/change history
- `Resources/Microsoft_Office_Reset_2.0.0b1_expanded/`: original package scripts and Distribution reference
- `.gitignore`: ignore rules for expanded package artifacts

## Scripting Style (Required)

Maintain the established style of `Microsoft-365-Reset.zsh` unless the user explicitly asks for a different style.

- Treat scripting style consistency as a primary requirement.
- Preserve section headers and separator style (`####################################################################################################`).
- Keep function declaration style and naming conventions consistent (`function xyz() { ... }`).
- Use explicit quoting (`"${var}"`) and existing variable naming patterns.
- Keep comments concise, practical, and in the existing voice.
- Prefer ASCII punctuation in script text and logs unless there is a clear reason not to.
- Route operational logs through the helper wrappers:
  - `preFlight`, `notice`, `info`, `warning`, `errorOut`, `fatal`
- Keep log format consistent:
  - `<script name> (<version>): <timestamp>  [LEVEL] <message>`
- Keep elapsed-time format consistent:
  - `Elapsed Time: %dh:%dm:%ds`
- Keep dialog conventions consistent:
  - Use global `fontSize` via `--messagefont "size=${fontSize}"`
  - Keep warning emphasis readable and intentional (Markdown where applicable)
  - Keep selection UI behavior consistent with current picker flow
- Do not add/remove CLI parameters unless explicitly requested.
- Keep the client-side log path hard-coded unless explicitly requested:
  - `scriptLog="/var/log/org.churchofjesuschrist.log"`

## Required Validation

1. Run `zsh -n` against every modified Zsh file (required).
2. At minimum, run `zsh -n Microsoft-365-Reset.zsh` after modifying the main script and `zsh -n scripts/mofa-consult.zsh` after modifying the MOFA helper.
3. Verify behavior-sensitive changes against operation flow (`self-service` and `silent` assumptions).
4. Update `README.md` when behavior, parameters, or examples change.
5. Update `CHANGELOG.md` for meaningful user-visible behavior changes.
6. Do not add new production dependencies without explicit user confirmation.

## Change Discipline

- Prefer minimal, targeted edits over broad rewrites.
- Avoid hidden behavior changes during refactors.
- If changing operation behavior, call out parity impact explicitly.
- Keep naming, formatting, and copy consistent with existing script patterns.
