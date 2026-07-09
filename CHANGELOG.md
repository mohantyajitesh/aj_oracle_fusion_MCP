# Changelog

All notable changes to this project are documented here. Format loosely
follows [Keep a Changelog](https://keepachangelog.com/); versions follow
[SemVer](https://semver.org/).

## [0.1.0] — 2026-07-09

First public release.

### Added
- **16 MCP tools** over Oracle Fusion Cloud HCM REST (ADF): discovery
  (`list_resources` with optional ~650-resource live index,
  `describe_resource` incl. child business actions, `get_capabilities`),
  generic read (`query_resource`, `get_record`), 7 curated HR workflows
  (`find_worker`, `get_worker_profile`, `list_direct_reports`,
  `get_reporting_chain`, `lookup_org`, `get_current_compensation`,
  `list_absences`), ATOM change feeds (`list_changes`), gated writes
  (`mutate_record`, `run_action`), and `server_info`.
- **Safety floor:** client-layer PII redaction + append-only JSONL audit that
  apply to every pod operation, below the tool layer.
- **Write gates (4 layers):** off-by-default feature flag → dry-run default
  with diff preview → schema validation that fails closed (unknown/read-only
  attributes and unknown action names blocked) → distinct audit of
  dry-run/blocked/committed.
- **Capability discovery:** concurrent per-module probes; `on/off/auto` module
  flags mirror Oracle's licensing pillars.
- Auth: Basic (password via OS credential store through `keyring`, no
  plaintext) and OAuth2 client-credentials; corporate TLS-inspection support
  via guarded `truststore`.
- Packaging: hatchling wheel, `fusion-hcm-mcp-server` console script, multi-stage
  non-root Docker image; stdio and streamable-HTTP transports.
- 60 offline unit tests; install guide for Claude Desktop (venv and Docker
  modes); design doc with pod-verified ADF ground truth.

### Known limitations (documented in DESIGN.md §14)
- No per-caller authorization (RBAC ceiling is the integration user).
- Writes do not support effective-date (`Effective-Of`) yet; no ETag/If-Match
  optimistic locking.
- HTTP transport is unauthenticated — use stdio.
- Audit log is local JSONL; ship to a SIEM for compliance retention.
