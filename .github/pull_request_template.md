## What changes

<!-- One or two sentences. -->

## Why

<!-- The problem this solves or the roadmap item it belongs to (e.g. S4). -->

## How I tested it

<!-- Commands run, tests added, manual checks. -->

## Checklist

- [ ] `pytest` passes and `ruff check . && ruff format --check .` is clean
- [ ] New or changed behavior has tests (concurrency / payment failure when relevant)
- [ ] Schema change → new Alembic migration (no edits to already-merged ones)
- [ ] Design decision → ADR added in `docs/decisions.md`
- [ ] `docs/progress.md` and `docs/learning.md` updated
- [ ] No secrets in the diff