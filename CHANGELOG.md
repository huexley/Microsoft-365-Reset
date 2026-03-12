# Microsoft 365 Reset

## Changelog

### Version 0.0.1a1 (12-Mar-2026)
- Initial unified script implementation
- Added CLI validation for missing `--mode` and `--operations` values
- Added preflight guard to require a non-root console user and block unsafe `/var/root` user-scope targeting
- Updated keychain credential removal helpers and operation flows to execute user-keychain deletes in logged-in user context
- Made package downloads deterministic by removing background transfer mode from repair downloads
- Updated README exit code documentation for `silent` mode with no selected operations
