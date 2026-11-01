# event-pred-live

The public record of the live paper-trading test in the final-year project *The Law of One Dollar: Machine Learning
for Market Making in Kalshi Prediction Markets* (NUS, CP4101).

From 15 October 2026 to 11 November 2026, every hour:

1. a **producer** reads the best bid and ask of every open Kalshi basket in the test, computes the features and records
   which maker quotes each strategy would post;
2. after the hour ends, a **paper trader** fetches that hour's real Kalshi trades, works out which of those quotes they
   would have filled, and appends one entry to a **hash-chained ledger**.

No order is ever sent to Kalshi. The fills are simulated on the real trade tape.

The live system copies its records into this repository every 10 minutes. Every commit here is written by that
automation, so the history shows when each record became public. Please do not edit the data files.

- **Dashboard:** https://curiousfun88.github.io/event-pred-live/
- **Code, test plan and analysis:** https://github.com/curiousfun88/event-pred (the test plan, with its dated
  amendments and host log, is `docs/planning/lumid-forward-test-preregistration.md`)

## Contents

| Path | What it holds |
|---|---|
| `runs/live/runs.jsonl` | One line per producer hour: snapshot and decision times, the hashes of that hour's files and of the models, and each strategy's posting share |
| `runs/live/scores/` | Every candidate quote of each hour, with every strategy's score (`one_*`: one price level, `two_*`: two levels) |
| `runs/live/stamps/` | Each producer hour's audit record and its OpenTimestamps proof (`.ots`) |
| `runs/live/paper/ledger/entries/` | The ledger: one entry per hour, with the new fills, the events settled and running totals |
| `runs/live/paper/ledger/stamps/` | OpenTimestamps proofs of the ledger heads |
| `runs/live/paper/ledger/corrections/` | Dated corrections, each recorded as its own stamped file, never by editing an entry |
| `runs/live/paper/tape/` | The real Kalshi trades each hour was settled against |
| `runs/live/paper/markets.jsonl` | Settlement results of the events traded |
| `heartbeat.json` | The latest producer hour and ledger entry, and the host |
| `site/`, `index.html` | The dashboard's page and data |
| `logs/sbx_keeper.log` | The keeper's operations log |

## Checking the record

All three checks need only public data.

**The hash chain.** Each ledger entry stores the SHA-256 hash of its own contents and the hash of the previous entry,
so changing, removing or reordering any entry breaks the chain. With the code repository cloned:

```bash
git clone --depth 1 --filter=blob:none --sparse https://github.com/curiousfun88/event-pred-live.git live
git -C live sparse-checkout set runs/live/paper/ledger/entries
python stg_infra/verify_ledger_chain.py live/runs/live/paper/ledger/entries
```

**The timestamps.** Every producer hour and ledger head was submitted to the public OpenTimestamps calendars when it
was made. `ots verify <file>.ots` shows that the record existed by that time.

**The results.** Every fill is in the ledger, so the reported figures can be recomputed with
`stg_infra/cut_report.py` in the code repository (see its `REPRODUCE.md`).

## Timeline

| Date (UTC) | Event |
|---|---|
| 15 Oct 19:00 | Live start: the ledger's first hour |
| 16 Oct 17:00 | Start of the main result (23 series) |
| 28 Oct 00:00 | Interim cut, reported in the project report |
| 11 Nov | Final cut |

Missing hours, host moves and every other incident are recorded in the test plan's host log.
