# Archive — legacy sections from `models/kimi.md`

Moved here on 2026-10-31 (v2.0.0) to keep the live reference lean. These models are retired, deprecated, or two or more generations behind the current lineup and are no longer the baseline that current-model deltas are written against. Content is kept verbatim as it stood when archived — it is **not maintained**: facts here may be stale. Load this file only when reviewing a prompt that still targets one of these models or when migrating off one.

## Kimi K2.5 (Moonshot, March 2026 — predecessor)

### Headline facts

- **Pure text** (no vision)
- Smaller Agent Swarms scope: 100 sub-agents × 1,500 steps
- Same temperature split (1.0 thinking / 0.6 instant) as K2.6
- 256 K context

### When K2.5 specific tuning is still useful

- Existing production stacks where you've measured prompt behavior and don't want to migrate yet
- Cost-sensitive deployments where K2.5 is "good enough"

Prompts written for K2.5 generally port forward to K2.6 with no rewrites. The reverse (K2.6 → K2.5) requires trimming swarm scale and removing image handling.

## Kimi K2 (legacy, retires May 25, 2026)

Earliest production-frontier Kimi model. Most current tooling has migrated past it. The official `platform.kimi.ai` quickstart still references K2 endpoints (`kimi-k2-0905-preview`, `kimi-k2-turbo-preview`, `kimi-k2-thinking`, `kimi-k2-thinking-turbo`), which **all retire May 25, 2026**.

If you encounter a prompt pinned to K2 endpoints in a current review, treat the retirement date as a forcing function and flag the prompt for migration to K2.6.
