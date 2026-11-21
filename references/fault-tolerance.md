# Fault-tolerance protocol

Three rules prevent the majority of multi-agent outages: back off, distinguish, and mark. Each one addresses a failure mode that is easy to misdiagnose.

## Rule 1 — back off, do not hammer

High-frequency retries are counterproductive. Rapid failed attempts accumulate half-open connections on the remote host and make recovery less likely, not more.

- Use a slow, fixed cadence — e.g. one attempt every few minutes — not a tight loop.
- Observe that a slow cadence often succeeds on the first try after a run of rapid failures. The retry storm itself is the problem.
- Cap consecutive failures and stop rather than retrying forever. Log the stop condition.

## Rule 2 — distinguish "host down" from "link broken"

These two failures look identical from the outside but need different responses.

Diagnosis procedure:

1. **Raw socket probe.** Open a TCP socket to the service port and read the banner. If a banner arrives and starts with the expected protocol prefix, the host and daemon are alive.
2. **Full handshake probe.** In the same second, attempt the real authenticated connection.
3. **Compare.** 
   - Socket fails → host or daemon is down. Escalate to host-level checks.
   - Socket succeeds but handshake stalls (e.g. stops after sending the initial key-exchange packet) → the link drops large packets while small packets pass. This is a link/path/firewall issue, not a host issue.

Doing the socket and handshake probes in the same second avoids confusing a transient state with a stable one. The diagnosis is about the path, not the machine.

Practical mitigations for a link that drops large packets: tune the connection config (disable QoS tagging, disable compression, narrow cipher/MAC sets), route the command channel through an HTTPS-based console API that does not use the affected port, or reduce payload size. Never conclude "host dead" from a handshake failure alone.

## Rule 3 — mark every remote value with a sentinel

This is the highest-value rule in the whole protocol.

**The problem:** when a remote command is run and the connection drops, the captured output is an empty string. When the connection succeeds and the command legitimately produces nothing, the output is *also* an empty string. Code that treats empty as success will report "connection recovered" on every failure — and the operator will believe it.

**The fix:**

- Remote side: after every command, append a distinct end marker as the final line, e.g. `__OUTPUT_END__`.
- Local side: accept the output as a successful result **only if the marker is present**. Parse everything before the marker as the payload.
- If the marker is absent, treat it as a transport failure regardless of what text came back. Retry per Rule 1.

The marker must be chosen so it cannot appear naturally in command output — use a leading/trailing double underscore and uppercase.

## Fetching files over an unreliable link

When a built-in file-pull command is unreliable, a robust fallback is to compress and encode the file inline and decode it locally:

1. On the remote: `gzip -c <path> | base64 -w0` and print the result plus the sentinel.
2. Locally: decode the base64, then gunzip.
3. Verify byte count against the remote `stat` output before accepting the file.

Always verify with a size or checksum. Silent truncation on a bad link is otherwise invisible.

## Correlating with the host side

Before blaming the remote host, rule out host-side causes that are actually under your control: service config, fail2ban, firewall rules, MTU/DF packet behavior. If the host shows a clean config and zero packet loss on large DF packets from its side, the fault is in the path or an external access layer the host cannot see. Record this conclusion so the same investigation is not repeated from scratch next time.
