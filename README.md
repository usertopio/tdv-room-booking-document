# TDV Room Booking [v1.0.0] — Document

Central repository for the **requirement → issue → tracking** lifecycle across the
TDV Room Booking project. This is the single source of truth for requirements and
implementation artifacts that previously lived in each code repo's `material/` folder.

## Related Repositories

| Repo                          | Role                                           |
| ----------------------------- | ---------------------------------------------- |
| `tdv-room-booking-frontend`   | React 19.2 application                         |
| `tdv-room-booking-backend`    | Backend services                               |
| `tdv-room-booking-deployment` | Deployment configuration                       |
| `tdv-room-booking-document`   | **This repo** — requirements, issues, tracking |

GitHub issues are created in the **code repo** where the work happens. This repo
holds the requirement documents, planning artifacts, and a cross-repo tracking board
that links out to those issues.

## Folder Structure

```
requirements/<YYYY-MM-DD>/      # one folder per requirement intake date
  raw.md                        # developer pastes raw requirements
  rewrite.md                    # formal rewrite (EN + TH); declares target repo(s)

issues/<YYYY-MM-DD>-<slug>/     # one folder per requirement-turned-issue
  codebase_check.md             # implemented / partial / not-found (with file:line)
  plan.md                       # implementation plan + Draft Issue block
  summary.md                    # post-task summary, links to merged PR(s)

tracking/
  backlog.md                    # cross-repo index: requirement -> issue link -> status

templates/                      # bilingual starting points for the files above
```

## Workflow

1. Create `requirements/<YYYY-MM-DD>/raw.md` and paste the raw requirement.
2. Write `requirements/<YYYY-MM-DD>/rewrite.md` — a formal, structured rewrite in
   **English + Thai**. Do not add or remove scope. Declare the target repo(s).
3. For each requirement, create `issues/<YYYY-MM-DD>-<slug>/` and write:
   - `codebase_check.md` — for each requirement, mark **Implemented**,
     **Partially Implemented**, or **Not Found**, with file paths and line numbers.
   - `plan.md` — implementation plan plus a **Draft Issue** block.
4. Create the GitHub issue in the target code repo using the Draft Issue:
   ```bash
   gh issue create --repo depa-platform/tdv-room-booking-<repo> \
     --title "<title>" --body "<description>"
   ```
5. Record the requirement, issue link, and status in `tracking/backlog.md`.
6. After the work merges, write `issues/<...>/summary.md` and update the backlog status.

## Conventions

- **Bilingual:** all documents are written in **English first, then Thai**.
- **Dates:** `<YYYY-MM-DD>` is the date the requirement was received.
- **Slug:** short kebab-case description of the issue (e.g. `booking-cancellation`).
- Treat `origin/main` as the source of truth.
