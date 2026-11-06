---
title: "Orbi — Pi harness vs Pi Durable vs kill -9 (structured notes)"
author: "Larry (Orbi)"
source_url: "https://orbi.build/blog/pi-agent-harness/"
published_at: "2026-10-06"
archived_at: "2026-11-06"
category: "practice"
archive_note: "Structured research notes for an in-house coding-agent harness engineer. Not a full reprint or sentence-by-sentence translation of the source."
---

## Source

Larry (Orbi), 2026-10-06: empirical kill-tests of three unattended Pi setups—`pi --print`, AI SDK `HarnessAgent`, Cloudflare `PiHarness`—on the same “create step1–5 with 8s sleeps” task using DeepSeek flash. Part of Orbi’s “how Orbi is built on Pi” series.

## What “Pi harness” means here

Since 2026-10-01, “Pi harness” can mean the **coding harness** (Pi 1.0 CLI / coding agent: if the process dies, a human looks and continues) or **Pi Durable** (checkpointed library for unattended runs). Orbi itself mostly runs `pi --print` with a supervisor; this post measures what each path leaves after `SIGKILL`.

## Comparison (outcomes)

| | `pi --print` | AI SDK `HarnessAgent` | CF `PiHarness` |
|---|---|---|---|
| Stack | Pi 1.0.0 child process | pi-coding-agent ~0.85 via `@ai-sdk/harness-pi` | Pi Durable ~1.0.4 in DO via `agents` |
| Session | JSONL in `--session-dir` | Host temp during turn; copy to sandbox on stop/detach/suspend | `pi_*` SQLite tables |
| SIGKILL | `--continue` finished (2/2) | Conversation lost (2/2) | After restarting local wrangler/runtime, object self-resumed (3/3) |
| SIGTERM | — | With `session.stop()` handler, resumed (7/7) | — |
| Who restarts | Your supervisor (+ reap orphans) | Your code from saved state | Heartbeat alarm ~30s (docs); local resume 1–10s after restart |

Not like-for-like: Cloudflare used custom `sleep`/`write_file` (Worker has no shell). Interrupted AI SDK `just-bash` sleeps were recorded as exit 0—model skipped remaining sleeps. PiHarness default-unsafe `sleep` wrote interrupted errors; model often **called sleep again**—so unsafe ≠ “won’t run twice” without app guards.

## CLI operational lessons

Close stdin (`</dev/null` / `DEVNULL`) or print mode can hang forever on open pipes. `SIGKILL` leaves detached bash process groups running—supervisors must reap (systemd scope/container). Orphan tool calls get synthetic “No result provided” when building the next request. Sample `pi-task.sh`: timeout with SIGTERM then SIGKILL fallback, `--continue` retries.

## Fit guidance

Git checkout + real shell + a machine → `pi --print` + supervisor. Pi as a feature inside a Node product with per-user sandbox → AI SDK (accept crash gap; wire SIGTERM). Crash survival without your own supervisor + API-shaped tools → Pi Durable on Cloudflare (newest/experimental; coding tools need an execution env you supply). All three still leave scheduling, exclusive claim, review, and merge to an outer layer (Orbi’s job).

## Takeaways for an in-house harness

Run your own kill matrix before choosing a control plane. Distinguish “session file survived” from “tool side effects were honest.” Plan orphan-process cleanup for any CLI-spawned bash. Treat model re-calls after interrupted tools as expected adversarial behavior toward idempotency design.

## Quote

> “I compared one thing: what is left after a hard kill, and who has to act.”
> — https://orbi.build/blog/pi-agent-harness/
