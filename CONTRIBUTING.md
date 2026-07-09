# Contributing

Issues and pull requests are welcome — including bug reports from people just
trying the server against their own Fusion pod. "It failed on my pod like
this" reports are especially valuable, since every pod's licensing, roles, and
flexfields differ.

## Filing issues

- Include: what you ran, what you expected, what happened, and your setup
  (OS, Python version, venv or Docker, `rest_version`).
- **Never include credentials, pod URLs, or real employee data** in an issue.
  Redact person numbers and names from output.
- For suspected security problems, see [SECURITY.md](SECURITY.md) — do not
  open a public issue.

## Pull requests

1. Fork, branch from `main`.
2. `pip install -e ".[dev]"` and make your change.
3. Add or update tests — the suite is offline (FakeClient, no pod needed) and
   must stay that way: `pytest`.
4. Keep the safety invariants intact. PRs that weaken these will not merge:
   - Reads redact PII by default; the client-layer redaction+audit floor stays
     inescapable.
   - Writes stay off by default, dry-run first, schema-gated (fail closed).
   - No secrets in code, logs, error messages, or committed config.
5. `ruff check` clean.

## Design changes

[DESIGN.md](DESIGN.md) is the source of truth. If your change alters a design
decision (tool surface, safety model, ADF ground truth), update DESIGN.md in
the same PR.
