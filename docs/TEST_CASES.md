# Test Cases & Production-Readiness Assessment

Oracle Fusion HCM MCP Server. This documents the read/write tool matrix, the
automated test coverage, a post-install smoke checklist for your pod, and a
production-readiness assessment.

## 1. Tool matrix (16 tools)

| Tool | Kind | Pod? | Redaction | Gate |
|---|---|---|---|---|
| `server_info` | diagnostic | no | n/a | — |
| `list_resources` | read (catalog) | only if `full_index` | n/a | — |
| `describe_resource` | read (schema) | yes | not PII | — |
| `get_capabilities` | read (probe) | yes | n/a | — |
| `query_resource` | read | yes | ✅ default | — |
| `get_record` | read | yes | ✅ default | — |
| `find_worker` | read (workflow) | yes | ✅ floor | — |
| `get_worker_profile` | read (workflow) | yes | ✅ floor | — |
| `list_direct_reports` | read (workflow) | yes | ✅ floor | — |
| `get_reporting_chain` | read (workflow) | yes | ✅ floor | — |
| `lookup_org` | read (workflow) | yes | ✅ floor | — |
| `get_current_compensation` | read (workflow) | yes | ✅ rows redacted | — |
| `list_absences` | read (workflow) | yes | ✅ floor | — |
| `list_changes` | read (ATOM) | yes | ✅ | `features.atom_enabled` |
| `mutate_record` | **write** | yes | diff redacted | `writes_enabled` + dry-run + schema |
| `run_action` | **write** | yes | — | `writes_enabled` + dry-run + schema |

## 2. Automated tests (offline, no pod) — 60 passing

| Suite | Covers |
|---|---|
| `test_config` | env override / defaults / missing base_url |
| `test_auth` | keyring fallback precedence, missing password → ConfigError, never-raises |
| `test_client_safety` | query redacts+audits; `redact=False` bypasses but still audits; write flagged; action in audit; failed op audited `error:<code>` then re-raised; sensitive flag; SSRF guard |
| `test_catalog` | seed search/filter; `summarize_describe` incl **child_actions** (CRUD excluded); live-index merge/dedupe |
| `test_filters` | extract / validate / build_q + quote-escape + bad-operator |
| `test_redaction` | masks / nested / disabled / variants |
| `test_workflows` | find_worker compacts + no PII + escaping; chain up via child link + PersonId; chain down levels; empty-chain note; effective_date pass-through; direct reports; comp wrapper-not-redacted-but-rows-are; absences person-only q + client-side filter; lookup_org unknown type |
| `test_atom` | parse feed + context; feature-gate off → note & no pod call; unknown feed rejected; date-only `since` expanded |
| `test_writes` | one test **per gate layer**: flag-off blocks; dry-run diffs RAW & doesn't write; unknown attr blocked; read-only blocked; fail-closed on missing schema; explicit commit writes+audits; run_action name validated via child_actions |

Run: `pytest -q`

## 3. Post-install smoke checklist (run against YOUR pod)

Every Fusion pod differs — licensing, roles, flexfields, release. After
installing (see docs/INSTALL_CLAUDE_DESKTOP.md), run these once against your
pod to confirm your deployment end to end:

- [ ] `get_capabilities` shows real modules `provisioned` (concurrent probes complete < tool timeout)
- [ ] `describe_resource workers` returns real attributes + `child_actions` on `workRelationships`
- [ ] `query_resource emps fields=[...]` returns real, redacted rows
- [ ] `find_worker` / `get_worker_profile` against a real person number
- [ ] `get_reporting_chain up` resolves managers via the assignment `managers` HATEOAS link
- [ ] `get_current_compensation` returns salary rows (amounts redacted)
- [ ] `list_absences` person-only query + client-side filter
- [ ] `list_changes` parses a real ATOM feed (with `atom_enabled`)
- [ ] `mutate_record` update **dry-run** shows a correct RAW diff on a real record
- [ ] `run_action terminate` dry-run validates against real `child_actions`
- [ ] A wrong-scope write returns Oracle's **403 unchanged** (RBAC ceiling holds)

## 4. Production-readiness assessment

| Dimension | Status | Notes |
|---|---|---|
| Read functionality | ✅ Built, unit-tested | ADF behavior per pod-verified ground truth (DESIGN §13) |
| Write safety (4-layer gate) | ✅ Built, unit-tested | Off by default |
| Client redaction+audit floor | ✅ Built, unit-tested | Inescapable |
| Packaging (Docker + stdio) | ✅ Verified | Image boots, 16 tools |
| Per-caller authorization | ◻ Deferred | RBAC is at the integration user (DESIGN §14a) |
| Capability enforcement | ◻ Reported not enforced | DESIGN §14b |
| HTTP transport auth | ◻ None yet | Use stdio for production (DESIGN §14c) |
| Audit durability | ◻ Local JSONL | Ship to SIEM for compliance retention (DESIGN §14d) |
| OAuth2 (prod auth) | ◻ Working stub | App identity; confirm token flow on your IDCS/IAM |

**Recommended first deployment:** a non-production pod with a least-privilege,
read-only integration user and writes left off (the default). Run the §3
checklist, then widen scope deliberately — enable modules, then ATOM, then
writes — as each layer proves out on your pod.
