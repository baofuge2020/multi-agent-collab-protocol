# Ledger template

Keep one row per turn. The ledger answers three questions at a glance: what was asked, what came back, and did it need a retry.

---

| Turn | Time | Request (short) | Result | Sentinel OK? | Retries | Notes |
|------|------|-----------------|--------|--------------|---------|-------|
| 1 | | | | yes / no | 0 | |
| 2 | | | | yes / no | | |

---

## Why track this

- **Re-turns are pure waste.** Count turns that had to be repeated because of missing sentinel, dropped connection, or ambiguous request. Reducing these saves more than switching models.
- **Failure patterns show up.** If failures cluster at certain times, suspect a time-of-day link problem rather than a host problem.
- **Cost becomes measurable.** Total turns × cost per turn = real spend. Guesses are not enough.

## Rules

- Mark `Sentinel OK? = no` for any result missing the end marker, even if text came back — that is a transport failure.
- Record the retry cadence used, not just the count, so back-off tuning can be compared.
- Keep the ledger append-only. Do not overwrite history.
