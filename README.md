# interaction-ledger

## Sample output

Running the pipeline on the generated sample data (`sample_data/`):

```
$ python -m ledger ingest sample_data/*.csv sample_data/*.jsonl --db ledger.db
events_a.csv: rows=842 loaded=795 duplicate=39 conflict=0 rejected=8
events_b.jsonl: rows=865 loaded=822 duplicate=41 conflict=0 rejected=2
events_overlap.csv: rows=422 loaded=0 duplicate=422 conflict=0 rejected=0
zz_conflicts.jsonl: rows=2 loaded=0 duplicate=0 conflict=2 rejected=0

$ python -m ledger ingest sample_data/*.csv sample_data/*.jsonl --db ledger.db
events_a.csv: already ingested (skipped)
... (all files skipped, state hash unchanged)

$ python -m ledger report --db ledger.db --out out
TOTAL: 400 interactions, 206 resolved (51.50%), collected 4360955.53, avg handle 1134s
Checks: collected_reconciles_to_ledger=OK, interactions_reconcile_to_ledger=OK,
        resolved_not_above_interactions=OK
```

| Metric | Value |
|---|---|
| Input rows read | 2,131 |
| Loaded as new events | 1,617 |
| Duplicates ignored | 502 |
| Conflicting events (logged) | 2 |
| Rejected rows (with reasons) | 10 |
| Interactions | 400 |
| Resolution rate | 51.50% |
| Collected | ₹43,60,955.53 |
| Average handle time | 18m 54s |

Reports are written to `out/report.json` (full detail) and `out/daily_channel.csv` (one row per day and channel).

## Dashboard

The report is also shown as a dashboard (a single self-contained HTML page) with:
- KPI tiles: interactions, resolution rate, money collected, average handle time
- Stacked daily charts of interactions and money collected, by channel, with hover details
- A per-channel scorecard
- A ledger-health panel: reconciliation checks, where every input row went, rejected-row reasons
- A filterable daily detail table

![Dashboard](docs/dashboard.png)

## Assumptions

The assignment brief was only available as a title and one-line summary, so the input format,
event types, metric definitions and schema were designed from that description. These are
the choices I made, all in `ledger/normalize.py` and `ledger/reports.py`:

- Dates such as `12/11/2026` are day-first. Timestamps with no timezone are treated as IST.
- Only INR amounts are accepted. Other currencies are rejected.
- An interaction is "resolved" if it has a `claim_resolved` or `payment_collected` event.
- Each interaction is counted on the local date its start event occurred.
- Events with no start event are reported separately rather than assigned a guessed day.
- On conflicting copies of an event, the first one wins and the conflict is logged.

If the real specification differs, only the normalisation and report definitions need to change.
The schema, idempotency and testing approach stay the same.
