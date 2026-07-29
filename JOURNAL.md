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

**Reproduction commit link:** [TODO: fill in after committing this file]

**Reproduction summary:**
Reproduced by hitting the `/health` endpoint directly on the running local app (`curl http://localhost:8000/health`) instead of going through the frontend. The response came back as `503 Service Unavailable` with `"redis":"unhealthy"` in the body, and the `make run` server logs showed the actual error: `redis_health_check_failed error="'Settings' object has no attribute 'redis_host'"`. This confirms the bug: `core/config.py` only defines `redis_url` on `Settings`, but `api/routes/health.py` tries to read `settings.redis_host` / `settings.redis_port`, which don't exist, so the Redis check always throws an `AttributeError` (caught, so it doesn't crash the server, but it always reports Redis as unhealthy regardless of Redis's real state). Steps to reproduce:
1. Start the app locally (`docker compose up -d`, `make run`).
2. Run `curl http://localhost:8000/health`.
3. Observe `503` response with `"redis":"unhealthy"`.
4. Check the server logs for the `redis_health_check_failed` error line confirming the `AttributeError` on `settings.redis_host`.

**PLAN.md link:** [TODO: add once PLAN.md is created]

**Walkthrough video (recommended):** [not recorded]

**Blockers or open questions:**
None so far.
