# Cost protocol

A multi-agent setup can quietly become expensive. Cost is controlled by three levers: which model runs where, how work is batched, and how many turns are consumed.

## Lever 1 — model placement

- **Executor: cheap or free tier.** Its job is deterministic and narrow — run a command, read a file, report a value. This does not need an expensive reasoning model.
- **Coordinator: the strong model.** All analysis, synthesis, judgment, and writing happen here. This is where reasoning budget is worth paying for.

The mistake is flipping this: using a powerful model on the executor "just in case" while starving the coordinator. That pays for reasoning where it is unused and skimps where it matters.

## Lever 2 — batching

Every connection and every turn has overhead. Batch aggressively:

- Put multiple related questions in one letter (see `communication.md`).
- Write a probe as a local script and run the whole script in **one connection**, rather than sending commands one at a time. One dropped connection then costs one reconnect, not one per command.
- Fetch multiple files in a single operation instead of one round trip per file.

Rule of thumb: if two operations will be requested back-to-back, they belong in the same turn.

## Lever 3 — turn economy

- One letter = one turn. Never send a follow-up letter while one is outstanding.
- Do not use a turn for something the coordinator can do locally.
- Do not re-ask for information already in the ledger.

## A note on flat pricing vs. usage

When you later price a skill like this for others, remember that an AI skill's running cost scales with usage. Flat pricing either overcharges light users or undercharges heavy ones. Two workable models:

- Price on **outcome value**, not on tokens. A skill that saves a user 10 hours a month is worth far more than its compute cost.
- For hosted execution, consider **usage-based billing** (per run) so the buyer's cost tracks their consumption.

For a one-time downloadable protocol package, flat pricing is fine — the buyer supplies their own models and bears their own usage cost.

## Measuring before optimizing

Track two numbers per period: total turns, and total re-turns caused by failures. Re-turns are pure waste. Reducing re-turns (via sentinel validation and correct link diagnosis) usually saves more than switching models.
