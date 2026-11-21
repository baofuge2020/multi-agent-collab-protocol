# Role protocol

Most multi-agent teams fail because roles are assigned by preference ("you handle X, I'll handle Y") instead of by physics. Assign each task to the side where it **must physically happen**.

## The two archetypes

- **Coordinator** — the local agent with full project context. It thinks, writes, checks, analyzes, and decides. It holds the whole picture.
- **Executor** — a remote agent running on another machine or runtime. It does only what cannot be done locally, plus the trade-offs that require its local knowledge.

You can have more than one executor. The rule stays the same: an executor exists because some work must happen on its machine, not to spread work around for its own sake.

## The decisive question

For every proposed task, ask: **must this happen on the remote machine?**

- Reading a log file on the remote host → yes, executor.
- Deciding what that log means → no, coordinator.
- Running a command that needs local credentials or local network → yes, executor.
- Formatting the result into a report → no, coordinator.
- Judging whether a threshold was breached → no, coordinator.
- Reporting a value only visible on that host → yes, executor.

If the answer is "it could be done either place," default to the coordinator. Every hop to a remote agent costs a turn, latency, and a failure surface.

## Decision authority

The executor reports facts; the coordinator makes decisions. Concretely:

- Executor answers: is it running, what does the file say, what is the value, did the command succeed.
- Coordinator decides: what to do about it, whether to retry, whether to escalate, what the numbers mean.

This split matters because the executor sees only its slice. Letting it make global decisions produces locally-reasonable but globally-wrong actions. Push judgment up to the side with context.

## The exception: local knowledge

The executor is allowed — expected, even — to make trade-offs that require knowledge only it has. Examples: choosing which local interface to query, noticing that a path is a symlink, recognizing that a service name changed. Encode these as "use your judgment on X, report what you chose and why."

## Anti-patterns

- **Mirror roles.** Both agents "do everything." This doubles cost and creates conflicts with no benefit.
- **Executor as analyst.** Asking the remote agent to interpret, summarize, or decide. This burns tokens on the expensive side and loses context.
- **Coordinator as courier.** The coordinator repeatedly asks the executor to read files it could have fetched itself in one operation. Fetch in bulk locally; ask remotely only when unavoidable.
- **Undefined authority.** No written rule on who decides. Disagreements then get resolved ad hoc, and the same argument recurs every turn.

## Writing role cards

Keep each role to one line of responsibility plus one line of authority. See `assets/role-card-template.md`. If a role cannot be stated in two lines, it is too broad — split it or remove it.
