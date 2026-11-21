# Communication protocol

The message queue is the backbone of the whole team. Get it wrong and every other protocol degrades.

## Why a file queue, not a chat channel

Agent-to-agent chat is unreliable. In practice, bot accounts often cannot receive messages from other bot accounts — a message that looks sent is never read. A shared file queue has three advantages:

- **Durable.** A file persists until it is consumed; a chat message does not.
- **Auditable.** Every turn is on disk and can be inspected.
- **Transport-agnostic.** It works whether the two agents share a filesystem, sync through object storage, or relay through a script.

Design assumption: one inbox file and one outbox file per executor. The coordinator writes to the inbox; the executor writes results to the outbox; the coordinator reads the outbox.

## One letter = one turn

The single most important rule: **one inbox file equals exactly one agent turn.**

- Pack every question for that turn into a single letter.
- Number each question if there is more than one.
- Keep the whole letter short — roughly 300 characters as a working ceiling.
- Drop greetings, thanks, and preamble. Open with the numbered request.
- Do not split one logical request across several files. Each file is a separate turn and costs latency plus a model invocation.

A good letter:

```
1. Check whether service X is running on the remote host; reply RUNNING/STOPPED.
2. Read the last 20 lines of /path/to/log and paste them verbatim.
3. Confirm free disk space with `df -h`; reply with the single line for /.
```

A bad letter:

```
Hi! Hope you're well. Could you maybe take a look at the service when you get a chance? Thanks!
```

The bad version is three times longer, unnumbered, and ambiguous about what output is expected.

## Numbering and scope

- Number only when there are multiple independent items. A single task needs no number.
- Each numbered item must specify its **expected output format** (RUNNING/STOPPED, verbatim lines, a single line). Ambiguous output expectations cause most rework.
- Keep all items in one letter topically related. Mixing unrelated tasks into one turn makes partial failures hard to reason about.

## Sentinels in the message body

A remote result can be empty for two completely different reasons: the connection failed, or the connection succeeded and produced no output. Locally these both look like an empty string. Never treat an empty result as success.

Require the remote side to append a distinct end marker, for example `__REPLY_END__`, as the last line of every result. The coordinator accepts a result as valid **only if the marker is present**. If the marker is missing, treat the turn as a transport failure and retry according to the back-off rule — do not interpret it as "no data."

The marker must be unlikely to occur naturally in command output. Prefix it with underscores and use uppercase.

## Turn lifecycle

1. Coordinator writes one numbered letter to the inbox.
2. Coordinator waits — does not write a second letter while the first is unanswered.
3. Executor reads the letter, performs the work, writes results plus the sentinel to the outbox.
4. Coordinator reads the outbox, validates the sentinel, and only then starts the next turn.
5. Both sides archive the consumed turn into a ledger (see `assets/ledger-template.md`).

## Common failure modes

- **Overlapping turns.** Writing a second letter before the first is answered produces interleaved results that cannot be matched to requests. Fix: one outstanding turn per executor at a time.
- **Missing expected output.** The executor does part of the work and omits the requested output format. Fix: put the output format inside each numbered item.
- **Empty result misread as success.** Fix: sentinel validation, as above.
- **Context drift.** Long threads cause the executor to lose track of which turn it is on. Fix: restate the turn scope in every letter; do not rely on history.
