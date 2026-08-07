# ArtemisHera — Data

Static legislative data for [ArtemisHera](https://github.com/knunnenkamp25/ArtemisHera).
Split out from the app repo because GitHub Pages caps a published site at 1 GB;
a full 50-state backfill runs well past that. Repos allow far more, and
`raw.githubusercontent.com` serves these files with `access-control-allow-origin: *`,
so the app fetches them cross-origin without any proxy.

## Layout

```
data/state/sessions.json                    index: state → available sessions
data/state/{ST}/{session}/people.json       [{people_id, name, party, role, district}]
data/state/{ST}/{session}/rollcalls.json    [[bill_number, title, date, question, result, chamber], …]
data/state/{ST}/{session}/votes.json        {people_id: [[rollcall_index, code], …]}
```

**Vote codes:** `1` Yes · `2` No · `3` Present/Other · `4` Not Voting/Absent

## Why packed this way

The source format repeats every rollcall's full text once per legislator — Virginia's
2025 session alone is 124 MB that way. Storing each rollcall once and reducing each
member to `[rollcall_index, code]` pairs brings the same session to 4.9 MB. Keywords
are not stored; the app derives them from bill titles at load time.

## Provenance

- **2017–present:** [Open States](https://open.pluralpolicy.com/data/) bulk session CSV
  archives, converted by `scripts/pack_openstates_session.py` in the app repo.
  Open States data is public domain; attribution appreciated.
- **Current session:** refreshed weekly through the Open States v3 API by
  `.github/workflows/state-votes.yml` in the app repo.
- **Pre-2017:** would come from LegiScan, whose bulk archives reach further back.

Generated data — don't hand-edit. Re-run the packing scripts instead.
