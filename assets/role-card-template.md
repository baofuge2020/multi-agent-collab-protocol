# Role card template

One role per card. Two lines only. If a role cannot be stated this briefly, split it.

---

## Coordinator

- Responsibility: thinks, writes, checks, analyzes, decides; owns full context.
- Authority: makes all global decisions; the final word on retry, escalation, and interpretation.

## Executor <name>

- Responsibility: runs commands on <machine/runtime>; reports facts only visible there; exercises local knowledge on <specific judgment call>.
- Authority: chooses how to accomplish local actions; reports the choice and why. Does not make global decisions.

---

## Test for each line

- "Must this happen on that machine?" → if no, move it to the coordinator.
- "Does this side have the context to decide?" → if no, make it report facts instead.
- "Could the coordinator do this locally in one operation?" → if yes, do not spend a turn on it.
