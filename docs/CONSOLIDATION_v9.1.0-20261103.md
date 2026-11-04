# Consolidation notes: v9.1.0 (2026-11-03)

Selective security-policy port onto upstream MimicGate `v9.1.0` / `origin/main` at `89b7908`, without carrying obsolete local attachment code or overwriting upstream CDP/SSRF work.

## Provenance

| Ref | Role | SHA |
|---|---|---|
| `origin/main` (base) | Upstream v9.1.0 line used for this worktree | `89b790820188d10c8a507c059ff93f6cd8a85623` |
| `local/relay-hardened-v9` | Original hardening branch (untouched) | `c0cf8cb` at `X:/Code/MimicGate` |
| `backup/relay-hardened-v9-preconsolidation-20261103` | Pre-consolidation backup of local hardening | `c0cf8cb` |
| Worktree branch | `consolidation/v9.1.0-20261103` | this branch only |

## Upstream merges preserved

These landed on `main` before consolidation and must remain the source of truth for their areas:

| PR | Title | Merge commit |
|---|---|---|
| [#95](https://github.com/TheBadFella/MimicGate/pull/95) | feat(browser): add reusable Chrome CDP profile mode | `d977426` |
| [#96](https://github.com/TheBadFella/MimicGate/pull/96) | security: harden API token comparison and CORS defaults | `33dd7d9` |
| [#97](https://github.com/TheBadFella/MimicGate/pull/97) | security: harden remote attachment downloads against SSRF | `bebc93f` |
| [#103](https://github.com/TheBadFella/MimicGate/pull/103) | Complete remote attachment hardening from #97 | `457687c` |

Local `src/api/openai_routes.py` attachment changes and the older local CDP delta in `src/browser/manager.py` were **not** copied. Upstream already owns those concerns via the PRs above.

## Ported from local hardening

- Secure logging defaults: `LOG_LEVEL=INFO`, `VERBOSE=false` (Config, Dockerfile, Compose).
- No insecure builtin VNC password; Compose requires `MIMICGATE_VNC_PASSWORD` or `CATGPT_VNC_PASSWORD`.
- Compose requires a strong API bearer via `MIMICGATE_API_KEY` / `CATGPT_API_KEY`; `API_TOKEN_OPTIONAL=false`.
- Loopback-only published ports (`127.0.0.1:8650` / VNC).
- `Config.API_ALLOW_UNAUTHENTICATED=false` with lifespan refusal for unauthenticated non-loopback binds, including `API_TOKEN_OPTIONAL=true` when a token is configured; explicit opt-in still available.
- Healthcheck limited to `/healthz` (no authenticated `/v1/models` fallback).
- Env catalog + generated `docs/ENVIRONMENT_VARIABLES.md` updated to match.
- Focused middleware/lifespan tests restore `API_TOKEN`, `API_TOKEN_OPTIONAL`, `API_HOST`, and `API_ALLOW_UNAUTHENTICATED`.
- `pytest.ini` restored for discovery; inactive-issues workflow uses `context.repo.owner`.

## Intentionally not ported

- Obsolete local remote-attachment implementation in `src/api/openai_routes.py` and related local-only helper tests that diverge from upstream SSRF/deadline hardening.
- Local-only CDP browser-manager delta superseded by upstream #95.
- Lockfile, current OpenAI-compatible API surface, and `AGENTS.md` remain upstream.

## Validation and publication

- Full Windows Python 3.13 test suite passed three times: 286 passed, 2 skipped; 2 upstream dependency deprecation warnings.
- Targeted authentication, CDP, MiniMax and attachment suite: 41 passed.
- `pylint --errors-only` passed on changed Python files, `compileall` passed, and generated env reference `--check` passed.
- `docker-compose config --quiet` passed with temporary placeholder credentials; no live container or browser acceptance was performed.
- Fork `main` was synchronized by fast-forward from the integrated branch; no history rewrite or force-push.
- Former PR worktrees retired after clean-state checks; their branches and original hardening backup refs were preserved.

## Unreconciled runtime caveats

1. **Compose now fails closed** without `MIMICGATE_API_KEY`/`CATGPT_API_KEY` and `MIMICGATE_VNC_PASSWORD`/`CATGPT_VNC_PASSWORD`. Existing deployments that relied on `dummy123` / `mimicgate` defaults must set real secrets before `docker compose up`.
2. **Published ports bind to `127.0.0.1`**. Remote Docker hosts, LAN clients, and reverse proxies that previously used `0.0.0.0` publication need an override or host-side proxy.
3. **`API_HOST=0.0.0.0` inside the container** remains valid only when a required token is configured (Compose sets `API_TOKEN_OPTIONAL=false`). Direct Python runs with empty token or `API_TOKEN_OPTIONAL=true` on a non-loopback host need `API_ALLOW_UNAUTHENTICATED=true`.
4. **Example docs** in API guides may still show `dummy123` as a sample token string; that is illustrative only and is no longer a Compose default.
5. **Preserve historical refs**: do not reset, delete, or force-update `local/relay-hardened-v9` or `backup/relay-hardened-v9-preconsolidation-20261103`; a verified checkout switch in `X:/Code/MimicGate` does not rewrite their commits.
