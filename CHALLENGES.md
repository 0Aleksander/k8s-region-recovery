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
