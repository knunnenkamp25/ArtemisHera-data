# Maintenance

## Repository size

This repo stores generated JSON, so every rewrite of a file is a new blob kept
in history forever. Two things keep that in check:

**1. Packing is idempotent (already in place).** `pack_openstates_session.py`
compares content before writing, so a repack that changes nothing produces an
empty diff. A targeted fix — like the committee/chamber resolution — only
rewrites the sessions it actually affects, instead of all ~1,150 files.

**2. Squash when history outgrows its usefulness.** Not yet needed; see below.

Current footprint:

| | |
|---|---|
| Working tree | ~500 MB apparent (~44 MB on disk, APFS compresses it) |
| `.git` | ~120 MB |

## When to squash

Squash once `.git` approaches **1 GB**, or when a clone becomes painful.

**Consider what you lose first.** History is currently the only record of *when
a data correction landed* — that a report generated before the chamber fix used
different data than one generated after. If you ever need to reproduce or
defend a specific report, that provenance matters. Squashing discards it.

A middle path that keeps provenance: tag before squashing
(`git tag data-2026-08 && git push --tags`), so the old tree stays reachable.

## Squash procedure

```bash
cd ArtemisHera-data
git tag "pre-squash-$(date +%Y%m%d)" && git push --tags   # keep an escape hatch
git checkout --orphan squashed
git add -A
git commit -m "Squash data history — snapshot $(date +%Y-%m-%d)"
git branch -M squashed main
git push --force origin main
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

Force-pushing rewrites remote history: make sure nobody else has the repo
cloned, and confirm the working tree is correct **before** the force-push, since
that's the only copy afterward.

## Alternative: release artifacts

If the repo keeps growing, publish each refresh as a GitHub Release asset (a
tarball of `data/state/`) instead of committing files. Releases don't count
against repo history. The tradeoff is that the app would need to unpack an
archive rather than fetching a single session over `raw.githubusercontent`,
which is the property that makes the current design simple.

## Routine

- **Weekly:** `state-votes.yml` refreshes the current session per state via the
  Open States API.
- **After any bulk repack:** confirm the diff is proportionate. A one-field fix
  touching every file means `write_if_changed` regressed.
- **After a backfill:** re-run `scripts/enrich_people.py` to fill party and
  district on new sessions.
