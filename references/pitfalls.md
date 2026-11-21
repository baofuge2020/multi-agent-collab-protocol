# Pitfalls — field mistakes and fixes

A distilled list of mistakes that cost real time in production multi-agent setups. Each has a concrete fix.

## 1. Assuming the chat channel works between bots

Bot-to-bot messages are frequently not delivered. Two agents in the same chat group may be unable to hear each other at all. Never build the collaboration backbone on a bot chat channel. Use a durable file queue. If a human is in the loop, that is fine — but agent-to-agent traffic needs its own path.

## 2. Paying a model to read files the coordinator could read itself

Non-interactive remote agents often return useless output for file reads (e.g. only confirming a write succeeded). Reading logs and inspecting files should be done locally by the coordinator whenever the file is reachable. Reserve the remote agent for judgment and for operations that truly require its machine.

## 3. One message per question

Splitting several related questions into several files turns one turn into several, multiplying latency and cost. Merge them into one numbered letter. Conversely, do not cram unrelated topics into one letter — partial failures become unmanageable.

## 4. Treating an empty output as a good result

Covered in `fault-tolerance.md` but worth repeating: empty output from a failed connection is indistinguishable from empty output from a successful one. Without a sentinel, the system will confidently report false recoveries. Always mark.

## 5. High-frequency retry storms

Twenty-five-second and ninety-five-second retry loops failed continuously; switching to a three-minute cadence succeeded on the first attempt. Fast retries create half-open connections that crowd out legitimate ones. Slow down.

## 6. Sending one SSH command per line

Each `ssh host "command"` opens a connection. If the link is flaky, each one is a new chance to fail. Write the probe or batch of commands into a local script and run it in one connection: `ssh host 'bash -s' < probe.sh`. One disconnect then costs one reconnect.

## 7. Short connect timeouts that ignore the config file

A command-line timeout option overrides the config file. If the config sets a long timeout but the command passes a short `-o ConnectTimeout`, the short value wins. Raise the timeout in the command itself, not only in config, when the link is slow.

## 8. Hostile network tuning copied without verification

Before tuning ciphers, MACs, QoS, or MTU, verify them one at a time with a same-second socket/handshake comparison. Changing several variables at once destroys the ability to attribute the fix. Record each change and its measured effect.

## 9. Hardcoding environment specifics into shared artifacts

IPs, instance names, paths, credentials, and vendor-specific command flags must never appear in a skill or protocol package meant to be shared. Extract them into runtime configuration. A shared artifact with someone's real infrastructure details is both a security leak and a usability dead end for buyers on different setups.

## 10. Binding the design to one transport

The protocols in this package are transport-agnostic on purpose. A buyer may use SSH, a cloud console HTTPS API, a message bus, or a sync folder. If the design assumes one vendor, the addressable market shrinks to people with that exact setup. Keep the link replaceable.

## 11. No ledger

Without a record of turns, failures, and decisions, the same bugs get rediscovered and cost cannot be measured. Keep a minimal ledger — see `assets/ledger-template.md`.
