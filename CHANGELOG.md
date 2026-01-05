# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
- Upcoming changes.

---

## [1.4.82] - 2026-1-5
### Added
- Added new logging flow
- Added new clear codes flow
### Fixed
- 

---

## [1.4.63] - 2025-11-21
### Added
- Improvements to firmware update error handling
- Improvements to firmware update flow
- Improvements to latest firmware version checking
- Improved logging
### Fixed
- Fixes a bug in clear codes
- Fixes a iniital connection issues

---

## [1.4.50] - 2025-11-03
### Added
- Adds Firmware Update improvements
### Fixed
- Fixes a crash in ValueMonitor

---

## [1.4.40] - 2025-09-23
### Added
- Added support for all DTC statuses
### Fixed

---

## [1.4.33] - 2025-09-17
### Added
- Improvements to bus sync
### Fixed

---

## [1.4.20] - 2025-08-05
### Added
- Adds YMM and VIN to narrative
- Adds second generic codes req if needed
- Adds support for firmware 2.022.08
### Fixed

---

## [1.4.10] - 2025-07-11
### Added
- Adds logging of firmware update progress
### Fixed
- Fixes the clear codes progress bug

---

## [1.4.00] - 2025-07-11
### Added
- Added BluetoothState property and callback to SDK
- Updates Bus Sync to overwrite unused device bus slots
- Updates to Mode 06 decoding and logging
- Updates to vehicle info logging
- Adds overwriting of unused device bus slots
### Fixed
- 

## [1.3.60] - 2025-06-11
### Added
- Added additional vehicle entry descriptions that better handle empty fields
### Fixed
- Fixed a bug with Mode $06
- Fixed a big with regional decimals
- Fixed duplicate generic request on some older vehicles
- Cleaned up session logs


## [1.3.52] - 2025-06-06
### Added
- Additional vehicle info error dialogs notifications
### Fixed
- Smooths out the connection flow for VIN errors

## [1.3.41] - 2025-04-29
### Added
- Adds additional logging
### Fixed
- Fixed various bugs to improve stability and reliability.
- ISO bus sync fixes
- Fixes a few generic codes bugs
- Fixes advanced data logging
- Fixes vehicle detail requests

## [1.3.31] - 2025-04-02
- Updates to logging summary
- Improved NRC handling
- Improved decode error handling
- Fixes data point bug in new firmware

## [1.3.22] - 2025-03-12
- Fixes a firmware update bug
- Fixes a connection bug
- Updated to support API Level 3 encryption
- Fixes reconnect after firmware update bug

## [1.3.21] - 2025-06-04
- Fixes a bug with GM scans
- Fixes a BLE bug
- Fixes a firmware update bug
- Fixes a connection bug
- Fixes a readiness monitor bug

## [1.3.2]
- What's New:
  - Fixes a bug with GM scans

## [1.3.1]
- Encryption is enabled
- Fixes for Mode $06 bugs


---

## [1.0.0]
### Added
- Initial stable release.
- Basic BLE device discovery and connection via `RepairClubManager`.
- Trouble code scanning and DTC lookup APIs.
- Live data streaming support.
- Firmware update workflow.

### Changed
- N/A

### Deprecated
- N/A

### Removed
- N/A

### Fixed
- N/A

### Security
- N/A
