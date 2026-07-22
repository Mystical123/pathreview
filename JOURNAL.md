## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/155

**Issue title:** Health check references `settings.redis_host`, which does not exist on Settings

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The `/health` endpoint is supposed to check whether Redis is reachable, but it currently crashes instead of reporting a clean status. The code tries to read a `redis_host` attribute off the `Settings` config object, but that field was never actually defined there — only a `redis_url` field exists — so Python throws an `AttributeError` the moment the health check tries to probe Redis. This means the health endpoint is currently unusable for its intended purpose, since instead of returning "Redis: down" it just crashes the whole request. A working fix would have the health check build its Redis connection from the existing `redis_url` field (or add proper host/port fields to `Settings`), so `/health` reports Redis status correctly instead of erroring out.

**Branch name:** fix/155-health-check-redis-host

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger
