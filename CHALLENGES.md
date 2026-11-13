# Challenges log

Every time something breaks, surprises me, or costs time, it goes here. One entry each: what happened, why, what it means for the thesis, the fix, and the time lost. This is the raw material for the "challenges" part of the practical chapters.

---

## 1. The Redis password changes on every install

**Date:** 2026-11-12
**Found:** reading the Helm chart before touching it (`chart/k8coins/templates/redis-secret.yaml`).

**What:** the chart generates a random 24-character Redis password on install. It only keeps that password across *upgrades* of the same release (it looks up the existing Secret). A fresh install on a new cluster gets a new password.

**Why it matters:** after a region loss I rebuild in a new region, so the restore is always a fresh install. The new password doesn't match the backup's world. The app has to come up with the right password, or the restored data is useless or unreachable. Also, the chart has no persistence at all yet (no volume, no RDB/AOF), so there is nothing to back up in the first place.

**Fix (planned):** let the chart take an existing password (e.g. `redis.existingSecret` or a value) instead of always generating one, and keep that secret outside the region with the backup. Then add a persistent volume.

**Time lost:** 0 (caught before building).

**Update 2026-11-13: mostly a non-issue.** The password is a *setting* Redis reads from an environment variable when it starts, not *data*. It is not stored in the RDB/AOF files. On a fresh install, Redis, worker and webui all read the same new Secret, so they always agree, and restored data works with any password. It only matters for things outside the cluster that need to log in to Redis (e.g. a backup tool). Kept here because finding out that a planned fix wasn't needed is also a result.

---

## 2. Deleting the Redis container deleted all the data

**Date:** 2026-11-12 (found), 2026-11-13 (fixed in compose, PR #3)
**Found:** the first local wipe test.

**What:** with `docker compose`, deleting the Redis container reset the app: 223 hashes before, 84 after (counting started again from zero).

**Why:** Redis keeps its data in RAM. The official image does write snapshots to `/data`, but `/data` was inside the container's own throwaway layer, so deleting the container deleted the snapshot too.

**Fix:** AOF on (`--appendonly yes`, default `appendfsync everysec`, so at most ~1 s lost) and a named volume `redis-data` mounted at `/data`. Test again on 13 Nov: 204 hashes before, 328 after, and the Redis log says `DB loaded from append only file`.

**Why it matters for the thesis:** a volume only survives the *container* dying. It lives in the same region, so it dies with the region. Surviving a region loss needs a backup copied somewhere else (V3). The data lost in a region loss is everything since the last backup (RPO).

**Time lost:** about 1 h over two days.

---

## 3. A comma inside the shell command became part of the password

**Date:** 2026-11-13
**Found:** review of my own `git diff`, before running it.

**What (my words):** when Docker runs the Redis command it opens a shell, and the shell reads the flags. I had `exec redis-server --requirepass`, then the password, then a comma, then `--appendonly yes`. Because of the comma, the comma gets added to the password.

**Why:** in `["sh", "-c", "..."]` the commas separate the list items, but everything after `-c` is one string. A comma inside that string is just a character. Redis would have required `k8coins-dev-only,` while the worker sent `k8coins-dev-only`, and nothing would mine.

**Two more in the same line:** the volume path copied from a Postgres example (`/var/lib/postgresql/data`, Redis uses `/data`, found on Docker Hub), and the flags in the wrong order (`--requirepass --appendonly yes "<password>"`; each flag's value has to come right after it). `docker compose config` said VALID for all of them: it checks YAML, not Redis.

**Time lost:** about 15 min, caught before running.

