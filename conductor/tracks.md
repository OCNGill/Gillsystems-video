# Tracks

Kanban-style work tracking for the Gillsystems-video pipeline. Each track is a distinct project or video in the pipeline.

## Track States

| State | Meaning |
|-------|---------|
| `active` | Currently being worked on — has resources allocated |
| `next` | Queued up — ready to start when an active slot opens |
| `future` | Idea or planned — needs scripting, assets, or scheduling |
| `published` | Live on YouTube — in the 7-day cleanup window |
| `archived` | Published >7 days ago — renders deleted, only Git artifacts remain |
| `rework` | Published but needs correction — cleanup window reset |
| `blocked` | Waiting on external dependency (assets, review, etc.) |

---

## Active

<!-- Currently in-progress videos. Max 2 at a time. -->

| # | Title | Status | Started | Est. Publish | Notes |
|---|-------|--------|---------|-------------|-------|
| — | — | — | — | — | — |

---

## Next

<!-- Queued and ready. Scripted, assets prepared. -->

| # | Title | Status | Priority | Notes |
|---|-------|--------|----------|-------|
| — | — | — | — | — |

---

## Future

<!-- Ideas and planned projects. Not yet scripted or resourced. -->

| # | Title | Status | Notes |
|---|-------|--------|-------|
| — | — | — | — |

---

## Published (Cleanup Window)

<!-- Videos live on YouTube. Renders still on ZFS. Delete at Day 7. -->

| # | Title | Published | Cleanup By | Deleted? |
|---|-------|-----------|------------|----------|
| — | — | — | — | — |

---

## Archived

<!-- Published >7 days ago. Renders deleted. Only Git artifacts remain. -->

| # | Title | Published | Archived |
|---|-------|-----------|----------|
| — | — | — | — |

---

## Rework

<!-- Published but pulled for corrections. Cleanup window reset. -->

| # | Title | Reason | Reset Cleanup By |
|---|-------|--------|-----------------|
| — | — | — | — |

---

*Last updated: 2026-07-11*