# Raw Requirement — 2026-06-30

> Transcribed from a conversational request (no raw.md was filed by the developer;
> recorded here for a complete artifact set).

When manually triggering the frontend CI (`docker-publish.yml`), I do not want it to
always use `secrets.VITE_APPS_SCRIPT_URL` and `secrets.VITE_ADMIN_PIN` (the prod
backend). I want to be able to choose which backend the image is built against.

Chosen approach: **Option B** — a `choice` input that switches between prod and demo
secrets (masked, fixed set), rather than free-text values typed on the CLI.
