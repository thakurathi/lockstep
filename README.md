# lockstep

**A deterministic-simulation-tested distributed KV store. Inject chaos, replay any failure from a single seed.**

> Status: just getting started. This README describes what the project aims to be. Almost none of it is built yet.

`lockstep` is a learning project: a small distributed key-value store (a Raft-replicated log) wrapped in a **deterministic simulation testing (DST)** harness. The idea is to run the whole cluster — every node, every network link, every clock — inside a single-threaded, seeded runtime, so that any failure the simulator finds can be reproduced exactly by replaying its seed.

---

## The idea

Ordinary unit tests only check the things you already know can go wrong. They say little about the partition that arrives mid-election, the clock that jumps backward during a commit, or the packet that gets reordered behind a crash-restart — the kinds of bugs that actually take distributed systems down.

DST aims at that gap. By running the system deterministically and driving it with a seeded random schedule of faults, the simulator can explore execution paths you'd never write down by hand — and because everything is deterministic, a rare failure becomes a one-line reproduction: re-run the same seed.

This technique was popularized by [FoundationDB](https://apple.github.io/foundationdb/testing.html) and is used by [TigerBeetle](https://tigerbeetle.com) and others. The goal here is to build a small version of it from scratch and understand it well.

## Planned design

Three layers, kept separate so the test harness stays reusable:

1. **The system under test** — a Raft-replicated KV store (leader election, log replication). It talks only to interfaces the runtime controls — no real sockets, clock, or threads.
2. **The deterministic runtime** — a single-threaded scheduler with a seeded RNG and a virtual clock, so a given seed produces the same run every time.
3. **The fault injector + checker** — drives a seeded schedule of partitions, clock skew, crashes, and packet loss/reordering, while checking that safety invariants hold.

## Goals

- **Determinism first.** Same seed should mean the same run, every time. Everything else builds on that.
- **Replayable failures.** A failing run should be reproducible from its seed alone.
- **Checked invariants.** Validate the observed client history rather than assuming correctness.
- **Clarity.** Keep the harness and protocol simple enough to explain.

## Roadmap

Starting small, roughly in this order:

| Step | Description | Status |
|---|---|---|
| Deterministic runtime | Single-threaded scheduler, seeded RNG, virtual clock | Not started |
| Simulated network | Message passing with loss, reordering, partitions | Not started |
| Raft core | Leader election + log replication | Not started |
| KV state machine | `get`/`set`/`delete` applied from the replicated log | Not started |
| Fault injection | Crashes, restarts, clock skew on a seeded schedule | Not started |
| Seed replay | Reproduce any run from its seed alone | Not started |
| Invariant checker | Validate the observed client history | Not started |

## Non-goals (for now)

- Not a production datastore — this is a learning project focused on correctness, not features.
- No cloud, GPU, or special hardware. It's meant to run locally on a normal CPU.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
