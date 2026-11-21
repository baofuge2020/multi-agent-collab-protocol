---
name: multi-agent-collab-protocol
description: A set of protocols for running a multi-agent team where a local coordinator agent and one or more remote executor agents collaborate over an asynchronous file-based message queue. Use this skill when designing, setting up, or debugging collaboration between multiple AI agents — especially when defining communication rules, role boundaries, cost controls, link-failure handling, or remote value retrieval. This skill should be used when a user asks how to make several agents work together reliably and cheaply, or when an existing multi-agent setup drops messages, loops on retries, or misreads empty results.
agent_created: true
---

# Multi-Agent Collaboration Protocol

## Overview

This skill encodes a field-tested set of protocols for running a small team of AI agents — one **coordinator** (local, has full context) plus one or more **executors** (remote, run on another machine or runtime). The agents talk through an asynchronous, file-based message queue rather than a chat channel, because agent-to-agent chat is unreliable: bots often cannot receive each other's messages.

The value is not the tools. It is the rules that decide **who does what, how a message is framed, how failures are judged, and how cost stays near zero**. Anyone can install the tools; buyers pay for the rules that have been debugged in production.

## When to use this skill

- Designing a new multi-agent collaboration from scratch.
- Defining who holds decision authority vs. who only executes.
- Setting up the message queue format (one inbox file = one agent turn).
- Diagnosing dropped messages, repeated retries, or false "connection recovered" signals.
- Retrieving values or files from a remote agent over an unreliable link.
- Reducing token/API cost in a multi-agent setup.

## The four protocols

These are summaries. Load the referenced file for the full rules before applying a protocol.

### 1. Communication protocol — one letter, one turn

Treat **one inbox file as exactly one agent turn**. Pack every question for that turn into a single numbered letter, keep it short (roughly 300 characters), and drop the pleasantries. Never split one logical request into several small messages — each message costs a turn and multiplies latency.

See `references/communication.md`.

### 2. Role protocol — judge remotely, fetch locally

Assign by **where the work must physically happen**, not by preference:

- The remote executor only does what must happen on its machine, plus judgment and trade-offs that require its local knowledge.
- The coordinator does all thinking, writing, checking, and analysis locally — reading logs and files itself rather than asking the remote agent to paste them back.

See `references/roles.md`.

### 3. Cost protocol — think cheap, execute cheaper

Cheap or free models are fine for the executor; expensive reasoning belongs on the coordinator. Batch operations into a single connection. One letter = one turn. Never let a cheap executor do expensive analysis just because it can.

See `references/cost.md`.

### 4. Fault-tolerance protocol — back off, distinguish, mark

Three rules that prevent most outages:

- **Do not retry at high frequency.** Failed rapid retries pile up half-open connections and make things worse. Back off to a slow, fixed cadence (e.g. one attempt every few minutes).
- **Distinguish "host down" from "link broken."** Compare a raw socket/banner probe with a full handshake. If the socket opens but the handshake stalls, the link is dropping large packets — not the host.
- **Mark every remote value with a sentinel.** An empty output from a failed connection looks identical to a successful connection with no content. Append an explicit end marker on the remote side and require it locally before treating a result as valid.

See `references/fault-tolerance.md`.

## Quick start

1. Define the roles. Write down, in one line each, what the coordinator does and what each executor does. Use `assets/role-card-template.md`.
2. Define the queue. Set up one inbox file and one outbox file per executor. Use the letter format in `assets/letter-template.md`.
3. Establish the sentinel. Pick an end marker and require it on every remote result. See `references/fault-tolerance.md`.
4. Run one end-to-end round trip and verify the sentinel arrives before scaling up.
5. Keep a ledger of turns and failures. Use `assets/ledger-template.md`.

## Security boundary

- Never hardcode credentials, IPs, API keys, or machine-specific paths into shared protocol files. Inject them at runtime via environment variables or a private config that is not distributed.
- Keep the transport pluggable. The protocols here are transport-agnostic; a buyer may use SSH, a cloud console channel, or any other link. Do not bind the design to one vendor.

See `references/pitfalls.md` for the full list of field mistakes.

## Resources

### references/
Detailed protocol documents — load the relevant one when applying that protocol:
- `communication.md` — message queue, letter format, numbering, sentinels
- `roles.md` — decision authority, local vs. remote work split
- `cost.md` — model selection, batching, turn economy
- `fault-tolerance.md` — retry cadence, link diagnosis, sentinel retrieval
- `pitfalls.md` — field mistakes and how to avoid them

### assets/
Ready-to-copy templates used in the output:
- `letter-template.md` — one-turn numbered letter
- `role-card-template.md` — one-line role definitions
- `ledger-template.md` — turn/failure ledger
