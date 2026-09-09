# Changelog

All notable changes to kvm-echo are documented here.

This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## 0.8.1 — 2026-09-08

Automation Task and Workflow captures now honour the options you configure.

### Fixed

- **Task and Workflow captures were ignoring every configured option.** Interface,
  duration, packet cap, byte cap and capture preset were all discarded on the
  Automation Task path, and every Task capture ran on built-in defaults — 30 seconds,
  10,000 packets, 10 MB, unfiltered — no matter what the Task specified. The plugin was
  reading these from the wrong place; it now reads them from the Task's saved options,
  keyed by option code. A Task configured for a 90-second capture now runs for 90 seconds.

  This affects every Task-based capture taken before this release: their durations and
  limits were not what the Task said, and their metadata records no capture preset.
  Existing capture files remain readable and are otherwise unaffected — only the settings
  they were taken with were wrong.

- **The capture preset is now recorded in Task capture metadata**, so the Captures table
  shows which preset a scheduled capture used. Previously only tab-initiated captures
  carried this, and captures taken before this release keep a blank preset permanently —
  the metadata file is written once, at capture time, and is not backfilled.

### Verified

A 90-second Automation Task capture against a KVM host, confirmed end to end: requested
duration honoured, ciphertext produced, exported and opened in Wireshark.

## 0.8.0 — 2026-09-07

Capture presets became real filters.

### Added

- **48 capture presets now carry actual BPF filter expressions.** Previously the preset
  dropdown was a labelling device — choosing "Web / HTTP + HTTPS" recorded the name but did
  not filter anything. Each preset now selects a real filter, covering web, remote access,
  infrastructure, storage, database, virtualisation, container, messaging, email and
  diagnostic traffic.
- **A capture preset dropdown on the host and VM tab capture form**, alongside the existing
  free-form BPF filter field.

### Notes

- Four presets are deliberately conservative. `Storage / vSAN`, `Virt / vMotion` and
  `Virt / VXLAN overlay` cover the common port sets but may need adjusting for a given
  environment, and `Container / Container registry` ships with no filter at all — there is
  no correct default when registries commonly sit on 443, 5000, or a site-specific port.
  Each explains itself in the UI when selected.
- Preset filtering is verified on the host and VM tab. Preset filtering on the Automation
  Task path did not work in this release — see 0.8.1.

> **No release artifact is published for 0.8.0.** It was superseded within a day by 0.8.1,
> and its headline Task-path fix did not work. The tag exists to preserve the history.
> Install 0.8.1.

## 0.7.10 — 2026-08-13

First public release.

- Plugin icon replaced with the project's own mark in the plugins list. No third-party
  brand elements.
- Publisher attribution corrected to `builtbyfood`.
- No behaviour change.

## 0.7.9 — 2026-08-13 (pre-release)

Public-facing name aligned with the repository.

- The plugin now identifies itself as `kvm-echo` throughout the Morpheus UI — the plugins
  list, the per-host tab, and the panel itself. Previously these read `morpheus-echo`.
- No behaviour change. The plugin's internal identity is unchanged, so captures, exports,
  scheduled tasks and existing host bootstrap are unaffected.

## 0.7.8 — 2026-08-13 (pre-release)

The plugin was in private use and testing for some time before the first public release.
This entry records the hardening that shaped it.

### Added

- **Integrity gate on every export.** The ciphertext's SHA-256 is computed on the
  hypervisor and compared against the bytes the appliance actually received, before any
  decrypt is attempted. Transit corruption now surfaces as a clear integrity failure naming
  the expected and received hashes, instead of as a confusing crypto or format error several
  layers later.
- **Background lease refresh.** Per-capture encryption keys are held in Cypher with a lease
  that could expire well before the retention window the UI advertised, quietly making
  older captures unexportable while the table still showed days remaining. A background
  sweep now refreshes the lease for every live capture, so the retention you are shown is
  the retention you get. Runs for scheduled Automation Task captures too, which need it most
  since nobody is watching them.
- **BPF filter support** at capture time, with a set of common filters available from a
  dropdown alongside free-form entry.

### Changed

- **No NOPASSWD sudoers entry required.** Earlier versions needed a host-side sudoers file
  to clean up after the capture wrapper. The plugin now uses Morpheus's built-in sudo
  escalation to run the wrapper directly, so that configuration is gone. Host bootstrap is
  down to two steps, and there is no sudoers file to write, audit, or leave behind.
- **Preflight reduced from six checks to four**, following the sudoers removal. The panel
  only appears when something actually needs fixing, and each failure carries a copy-paste
  remedy.
- Export is now offered per row only when that capture can actually be exported, rather
  than presenting a button that was always going to fail.

### Fixed

- **Capture and export status panels render correctly.** A plugin CSS class collided with a
  framework class of the same name, which clipped every progress and result panel to an
  invisible strip — capture progress, the failure notice, and the download-ready banner
  were all affected. All plugin styles are now namespaced.
- Documentation and in-product description text that still described the manual workflow
  from much earlier versions, including one block that told operators to copy ciphertext
  off the host by hand and incorrectly claimed the encryption key was never persisted.

### Known limitations

Carried forward and documented in the
[User Guide](docs/USER-GUIDE.md#known-issues-check-here-first): multi-chunk exports can
intermittently fail their integrity check and need a retry, and captures whose key lease
lapsed during a plugin restart are not recoverable. Both have workarounds; neither risks
the confidentiality of a capture.
