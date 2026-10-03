# DECISIONS — technical decision log

Written in English on purpose: it is portfolio-facing and good writing practice.

Format:

```text
## ADR-NNN: title
- Date:
- Status: proposed | accepted | superseded
- Context: what problem do we have?
- Options considered:
- Decision and reasoning:
- Consequences:
```

## ADR-001: Base stack

- Date: 2026-10-03
- Status: proposed
- Context: choose the stack for the booking backend.
- Options considered: FastAPI vs Django vs Flask; SQLAlchemy vs raw SQL.
- Decision and reasoning: FastAPI + PostgreSQL + SQLAlchemy 2.0 (to be confirmed and completed with my own reasons; PostgreSQL is required for the exclusion constraint).
- Consequences:

## ADR-002: Sync vs async database access

- Date: 2026-10-03
- Status: proposed
- Context: FastAPI supports both sync and async endpoints; SQLAlchemy 2.0 supports both drivers.
- Options considered: sync (simpler to reason about and to test concurrency with threads) vs async (more scalable, more moving parts).
- Decision and reasoning: TBD in S1.
- Consequences:

## ADR-003: Bookable resource and slot model

- Date: 2026-10-03
- Status: proposed
- Context: the availability model depends on what is being booked.
- Options considered: fixed-length slots vs free time ranges; opening hours and time zone per resource.
- Decision and reasoning: TBD in S1.
- Consequences: