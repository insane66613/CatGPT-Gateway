# MimicGate consolidation receipt (2026-11-03)

## Source and preservation
- Upstream authority: TheBadFella/MimicGate `origin/main` at `89b7908` (includes v9.1.0).
- User fork: insane66613/CatGPT-Gateway `fork/main` at `fb79ea1` before reconciliation.
- Original local hardening: `local/relay-hardened-v9` at `c0cf8cb`.
- Immutable recovery pointer: `backup/relay-hardened-v9-preconsolidation-20261103`.
- Upstream PRs #95 (CDP), #96 (API auth/CORS), #97 and #103 (attachment security) were merged upstream.
- Keep upstream's current attachment download implementation, provider APIs, tests, lockfile, and AGENTS.md.

## Selective local policies retained
- Compose publishes API/VNC only to loopback and requires explicitly supplied API and VNC credentials.
- Dockerfile defaults to INFO/less verbose logging and a health-only probe.
- Runtime refuses unauthenticated non-loopback API listeners unless explicitly opted in, including when `API_TOKEN_OPTIONAL=true`.
- No hard-coded fallback VNC password; application logging defaults to INFO/VERBOSE=false.
- Fork-safe stale-issue workflow, scoped pytest collection, corresponding tests and regenerated environment reference.

## Validation
- Windows Python 3.13: `python -m pytest -q` => 286 passed, 2 skipped (two dependency deprecation warnings).
- Focused middleware, CDP, MiniMax and attachment tests => 41 passed.
- `python -m pylint --errors-only` on modified Python sources/tests => 0 errors.
- `python -m compileall -q src scripts tests` passed.
- `python scripts/generate_env_reference.py --check` passed.
- `git diff --check` passed.
- `docker-compose config --quiet` passed with placeholder environment credentials.

## Limits
- The full suite does not establish live-provider login or deployed gateway health.
- No browser-session, container runtime, or external client acceptance test was run.
- Preserve old Git branches for audit even after clean PR worktree folders are retired.
