# Letter template — one turn, one file

Copy this into the inbox file. Keep the whole letter short (roughly 300 characters). No greetings, no preamble.

---

[Turn #N]

1. <one action> — reply <expected output format>
2. <one action> — reply <expected output format>
3. <one action> — reply <expected output format>

End every numbered item with the exact output you expect. Example formats:
- `reply RUNNING or STOPPED`
- `paste the last 20 lines verbatim`
- `reply with one line: <label> <value>`

---

## Rules

- Every numbered item states its expected output format.
- All items in one letter are topically related.
- Do not send a new letter while the previous turn is unanswered.
- The remote side appends the sentinel (`__REPLY_END__`) as the final line of the result.
- A result without the sentinel is a transport failure, not "no data."
