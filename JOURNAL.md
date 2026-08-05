## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/155

**Issue title:** Health check references `settings.redis_host`, which does not exist on Settings

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The `/health` endpoint is supposed to check whether Redis is reachable, but it currently crashes instead of reporting a clean status. The code tries to read a `redis_host` attribute off the `Settings` config object, but that field was never actually defined there — only a `redis_url` field exists — so Python throws an `AttributeError` the moment the health check tries to probe Redis. This means the health endpoint is currently unusable for its intended purpose, since instead of returning "Redis: down" it just crashes the whole request. A working fix would have the health check build its Redis connection from the existing `redis_url` field (or add proper host/port fields to `Settings`), so `/health` reports Redis status correctly instead of erroring out.

**Branch name:** fix/155-health-check-redis-host

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Mystical123/pathreview/commit/1eb462b

**Reproduction summary:**
Reproduced by hitting the `/health` endpoint directly on the running local app (`curl http://localhost:8000/health`) instead of going through the frontend. The response came back as `503 Service Unavailable` with `"redis":"unhealthy"` in the body, and the `make run` server logs showed the actual error: `redis_health_check_failed error="'Settings' object has no attribute 'redis_host'"`. This confirms the bug: `core/config.py` only defines `redis_url` on `Settings`, but `api/routes/health.py` tries to read `settings.redis_host` / `settings.redis_port`, which don't exist, so the Redis check always throws an `AttributeError` (caught, so it doesn't crash the server, but it always reports Redis as unhealthy regardless of Redis's real state). Steps to reproduce:
1. Start the app locally (`docker compose up -d`, `make run`).
2. Run `curl http://localhost:8000/health`.
3. Observe `503` response with `"redis":"unhealthy"`.
4. Check the server logs for the `redis_health_check_failed` error line confirming the `AttributeError` on `settings.redis_host`.

**PLAN.md link:** https://github.com/Mystical123/pathreview/blob/fix/155-health-check-redis-host/PLAN.md

**Walkthrough video (recommended):** [not recorded]

**Blockers or open questions:**
None so far.



## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I have just updated JOURNAL.md and made a PLAN.md understanding what the issue is and where to go about it. I have also gone through which files need to be changed that I previously mentioned for last weeks PR and I have gone through each file seeing where the main issue is.

**Next steps:**
This week im going to begin on implementing a real fix towards the codebase and modifiying the code to try to fix the issue. Then I will attempt to make my test cases and confirm my code is working and not just on my local machine. The Final step would be to document everything and submit a final PR for review.

**Blockers:**
Nothing slowing me down right now but I am just going to implement each step of the plan and go from there.
---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/934

**Branch:** fix/155-health-check-redis-host

**What you built:**
Fixed `api/routes/health.py` so the Redis check builds its client from `settings.redis_url` (via `redis.Redis.from_url(...)`) instead of the nonexistent `settings.redis_host` / `settings.redis_port` fields. `/health` now correctly reports Redis as `"healthy"` or `"unhealthy"` based on its real connection state, instead of always failing with an `AttributeError`.

**Tests added or updated:**
Added `tests/unit/test_health.py` with two tests: one confirms `/health` reports `"redis":"healthy"` when Redis is reachable (and that `redis.Redis.from_url` is called with the real `redis_url` config), and one confirms it reports `"redis":"unhealthy"` with a clean `503` (not an `AttributeError`) when Redis is unreachable.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

Verified scoped to my changed files: `ruff`, `black`, and `mypy` are all clean on `api/routes/health.py` and `tests/unit/test_health.py`, and both new tests in `test_health.py` pass. I also confirmed via a `mypy` before/after diff that this fix resolves the two `attr-defined` errors on `redis_host`/`redis_port` with no new errors introduced. The full repo (`make check`/`make test-unit` run with no path scoping) surfaces pre-existing lint and test failures from other unclaimed issues elsewhere in this 66+ issue seeded codebase — confirmed unrelated by checking that none of the failing test files import `health.py` or `core/config.py`.

**Draft PR feedback received from:** none
