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
| 2026-06-30 | Manual CI: select prod/demo backend (separate secrets, distinct demo tag) | frontend | [#92](https://github.com/depa-platform/tdv-room-booking-frontend/issues/92) | [#93](https://github.com/depa-platform/tdv-room-booking-frontend/pull/93) | Merged | `requirements/2026-06-30/`, `issues/2026-06-30-ci-manual-backend-select/` |
| 2026-06-30 | Consolidate applicationStatus transition logic into one function (dedupe + fix drift) | backend | [#24](https://github.com/depa-platform/tdv-room-booking-backend/issues/24) | [#25](https://github.com/depa-platform/tdv-room-booking-backend/pull/25) | Merged | `requirements/2026-06-30/` (`*-consolidate-status-transition.md`), `issues/2026-06-30-consolidate-status-transition/` |
| 2026-06-30 | Security hardening: escape HTML in emails + validate clientUpdateDocs link | backend | [#26](https://github.com/depa-platform/tdv-room-booking-backend/issues/26) | [#27](https://github.com/depa-platform/tdv-room-booking-backend/pull/27) | Merged | `requirements/2026-06-30/` (`*-security-hardening.md`), `issues/2026-06-30-security-hardening/` |
| 2026-06-30 | Workflow: require linking the PR to its issue with a closing keyword | frontend, backend | [fe#94](https://github.com/depa-platform/tdv-room-booking-frontend/issues/94) · [be#28](https://github.com/depa-platform/tdv-room-booking-backend/issues/28) | [fe#95](https://github.com/depa-platform/tdv-room-booking-frontend/pull/95) · [be#29](https://github.com/depa-platform/tdv-room-booking-backend/pull/29) | Merged | `requirements/2026-06-30/` (`*-workflow-link-pr-to-issue.md`), `issues/2026-06-30-workflow-link-pr-to-issue/` |
| 2026-06-30 | Split main.js into 9 focused modules (structural refactor, no behavior change) | backend | [#30](https://github.com/depa-platform/tdv-room-booking-backend/issues/30) | [#31](https://github.com/depa-platform/tdv-room-booking-backend/pull/31) | Merged | `requirements/2026-06-30/` (`*-split-main-into-modules.md`), `issues/2026-06-30-split-main-into-modules/` |
