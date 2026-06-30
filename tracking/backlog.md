# Backlog — Cross-Repo Tracking

Single index of all requirements and the issues they produced across the project's
code repositories. Update this whenever a requirement is filed, an issue is created,
or a status changes.

**Status values:** `Requirement` · `Planned` · `Issue Created` · `In Progress` · `In Review` · `Merged` · `Closed`

> Rows below dated 2026-06-25/26 are the **migration baseline** — work completed before
> this repo existed. Their artifacts were migrated here from the frontend's legacy
> `material/` folder (now deleted). New work uses `requirements/` + `issues/<date>-<slug>/`.

| Date | Requirement | Target repo | Issue | PR | Status | Artifacts |
|------|-------------|-------------|-------|----|--------|-----------|
| 2026-06-25 | Force refresh — bypass backend cache on manual refresh | frontend, backend | — | [frontend#69](https://github.com/depa-platform/tdv-room-booking-frontend/pull/69) | Merged | `requirements/2026-06-25/`, `issues/2026-06-25-force-refresh-nocache/` |
| 2026-06-26 | Format admin date display as DD-MM-YYYY | frontend | [#72](https://github.com/depa-platform/tdv-room-booking-frontend/issues/72) | [#73](https://github.com/depa-platform/tdv-room-booking-frontend/pull/73) | Merged | `requirements/2026-06-26/` (`*.md` base) |
| 2026-06-26 | Show promotion duration (cell-meta) under package in admin table | frontend | [#74](https://github.com/depa-platform/tdv-room-booking-frontend/issues/74) | [#75](https://github.com/depa-platform/tdv-room-booking-frontend/pull/75) | Merged | `requirements/2026-06-26/` (`*-issue74.md`) |
| 2026-06-26 | Colorize admin status dropdown by application status | frontend | [#76](https://github.com/depa-platform/tdv-room-booking-frontend/issues/76) | [#77](https://github.com/depa-platform/tdv-room-booking-frontend/pull/77) | Merged | `requirements/2026-06-26/` (`*-issue76.md`) |
| 2026-06-26 | Format date filter button as DD-MM-YYYY | frontend | [#78](https://github.com/depa-platform/tdv-room-booking-frontend/issues/78) | [#79](https://github.com/depa-platform/tdv-room-booking-frontend/pull/79) | Merged | `requirements/2026-06-26/` (`*-issue78.md`) |
| 2026-06-30 | Manual CI: select prod/demo backend (separate secrets, distinct demo tag) | frontend | [#92](https://github.com/depa-platform/tdv-room-booking-frontend/issues/92) | [#93](https://github.com/depa-platform/tdv-room-booking-frontend/pull/93) | In Review | `requirements/2026-06-30/`, `issues/2026-06-30-ci-manual-backend-select/` |
