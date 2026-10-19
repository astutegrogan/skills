---
name: bisect-debug
description: Stop guessing and bisect when black-box debugging has stalled. Use when a failure gives no usable trace, each test is expensive, and hypothesis-driven fixes keep coming back negative, especially when every theory concerns a component named in the error. It builds a minimal repro from real components and adds them back until the failure reappears. Not for failures with a readable stack trace or for verifying a fix.
---

# Bisect Debug

Use this to resist **anchoring**, not to replace ordinary trace-driven debugging. When several well-controlled hypotheses have been disproven but all target the components named in an error, change strategy: build a minimal repro from the real components and add them back one at a time.

## What this skill can and cannot decide

| Mechanical | Judgment |
|---|---|
| Log every hypothesis round in a ledger | Choose the useful component granularity: file, action, class, config key, or data record |
| Trip the switch after three disproven rounds | Decide whether a harness can load an arbitrary subset of the real components |
| Inventory every component, including silent ones | Find the smallest subset that still works |
| Copy components verbatim, never as toy substitutes | Decide whether a positive step recreates the real failure |
| Add one component or one half per step and log it | Interpret the localized minimal failing set |

## Step 0: hypothesis ledger

From the first hypothesis onward, keep one line per round in the conversation or a scratch file:

`#n | hypothesis | component(s) it concerns | named in the error? y/n | test run | result`

After **three disproven rounds with no resolution**, stop generating hypotheses and start Step 1. Switch after two if one more hypothesis test would cost about as much as the estimated bisection. If every ledger row concerns an error-named component, say that anchoring is the reason for switching.

## Step 1: preconditions

All three must hold:

1. **Reproducible:** run the full failing configuration twice. If it is intermittent, run each step `k` times and state `k` and why; otherwise stop because bisection is unreliable.
2. **Subsettable:** build a harness that loads a chosen subset of the *real* components. If that is impossible, use traces, telemetry, or normal diagnosis instead.
3. **Working base:** identify and run a small subset that passes. If the smallest base fails, shrink it further or bisect within it.

## Step 2: inventory

List every component of the failing configuration at the chosen granularity and tag each:

- `loud`: named in an error
- `tested`: exercised against this specific failure
- `silent`: never tested against this failure

`silent` does not mean cleared. It only means untested against this failure.

For non-trivial systems, ask a fresh-context subagent to create the inventory. Give it the system and failing configuration, but not the ledger or prior hypotheses. Merge both inventories. Estimate the cost: roughly `log2(N)` tests for halving or `N` for one-at-a-time, times the cost per test.

## Step 3: bisect upward from the working base

- Use **real components only**, copied unchanged. A toy reproduction proves only a general mechanism, not this failure.
- Add components in the failing system's own order, never by suspicion.
- For many components, add half of the remaining set. If it passes, retain it and add half of what remains; if it fails, bisect within that half. For a small set, add one at a time.
- Log every step as `base + {added} -> pass/fail (same failure signature? y/n)`. Record a different failure as a separate finding without chasing it now.

## Step 4: localize, then hypothesize

The first failing step identifies a **trigger** component, not necessarily the complete cause.

1. Test `base + trigger only`.
2. If it passes, it is an interaction: remove previously added components one at a time or by halves until the minimal failing set remains.
3. Form hypotheses only about that minimal set. Then implement and verify the fix using the appropriate test or review workflow.

## Beyond code and completion

The same method works for data-quality anomalies: components may be input records, time windows, or pipeline stages, as long as chosen slices can be replayed.

Report the ledger, the inventory and its tags, the bisection log, the minimal failing set, and whether the culprit began as `silent`.
