---
name: v-model
description: NASA V-Model systems engineering for building and verifying code. Use when implementing a non-trivial feature, building a system, or when asked to apply the "V-model", "systems engineering", "traceability", or rigorous "verify and validate" approach. Drives a task down the left side (define each level plus its paired verification plan), implements at the bottom, then climbs the right side verifying bottom-up and validating against the original intent.
license: MIT
---

# V-Model: Systems Engineering for LLM Coding

The NASA V-Model adds three things generic planning modes lack: **leveled
altitude**, **paired verification defined on the way down**, and a
**traceability matrix** that is the spine of the work. Use those three, or this
is just planning.

This is a rigid workflow. Follow the phases in order. The tailoring gate (Phase 0)
is what keeps it from becoming ceremony on small tasks — do not skip it.

## Phase 0 — Tailor (do this first, every time)

Classify the task, **state the classification to the user**, and let them
override before proceeding:

| Tier | Trigger | What you run |
|---|---|---|
| **trivial** | one-liner, isolated bugfix, config change | Skip the middle levels. Write a single `traceability.md`: one-line intent, one row, one verification (a test). Implement. Verify. Done. |
| **feature** | a single coherent capability / one subsystem | Full single V (below). |
| **system** | multiple interacting subsystems | Decompose into sub-Vs — run a feature-tier V per subsystem — under one system-level V (one ConOps, one system-level validation). |

Announce: `V-Model tailoring: <tier> — <one-line reason>. Override if wrong.`

## Left side — DESCEND

At each level you define **what**, and *in the same step* write that level's
**verification plan**. Never descend before the paired verification exists.

1. **L1 ConOps** → write `conops.md`: why this exists, for whom, and what success
   looks like in the user's world. **Paired:** a *System Validation Plan* — concrete
   acceptance scenarios phrased in user terms ("when the user does X, they see Y").
2. **L2 System Requirements** → write `requirements.md`: observable behaviors and
   constraints, each tagged with a stable `REQ-id` (`REQ-1`, `REQ-2`, …). **Paired:**
   a *System Verification Plan* — how each `REQ` will be checked at the system boundary.
3. **L3 Subsystem / High-Level Design** → write `design.md` (design section): the
   modules/components and their interfaces; **allocate every `REQ` to a subsystem.**
   **Paired:** a *Subsystem Verification Plan* — the integration checks per interface.
4. **L4 Component Detailed Design** → extend `design.md`: per-component behavior,
   signatures, and edge cases. **Paired:** a *Component Verification Procedure* — the
   exact unit tests (names + what each asserts), written **before** any implementation.

Then build the **Traceability Matrix** in `traceability.md`:

| REQ | Subsystem | Component | Verification ID | Status |
|-----|-----------|-----------|-----------------|--------|
| REQ-1 | … | … | V-1 (unit) | ☐ |

Every `REQ` must appear. Every row must name the verification that will prove it.

## Bottom — IMPLEMENT

Write the code and the tests already specified in the L4 Component Verification
Procedures. Because the test specs already exist, this is naturally test-first.

## Right side — CLIMB

Verify bottom-up against the plans you wrote on the descent, then validate:

- **R4 Component Verification** — run the unit tests (L4 procedures). Mark matrix rows.
- **R3 Subsystem Verification** — run the integration checks (L3 plan). Mark rows.
- **R2 System Verification** — run the System Verification Plan. Every `REQ` must
  show PASS with evidence (command + result). *"Did we build it right?"*
- **R1 System Validation** — replay the ConOps acceptance scenarios.
  *"Did we build the right thing?"*

## Done gate

Done is decided by your project's **real tools**, not by self-report and not by a
bespoke checker. Three parts:

1. **Every `REQ` states how it is verified** — a concrete test, or a type-check, lint,
   build, or explicit human review. If you can't name how a `REQ` is verified, it
   isn't granular enough — split it. (This is the matrix's verification column.)
2. **The verdict is the real test command's exit code.** Run the project's actual test
   command; the matrix closes only when it **exits 0** *and* every `REQ`'s named test
   is one that command actually runs. Determinism comes from the runner, never from
   prose — only mark a row `☑` against a test you watched the runner pass.
3. **Every System Validation scenario passes** when replayed.

Guard against tests that pass without proving anything — not with more checking code,
but with discipline the test framework already gives you:
- **Red-green:** each test must have *failed* before its code existed (write it, watch
  it fail, then implement). A test that never failed proves nothing.
- **Property-based tests** for edge-heavy requirements: assert over many generated
  inputs, not one hand-picked case.

Trust nothing self-reported. The proof is strongest when something **other than the
implementer** (a separate review pass, or the human) runs the command and reads the
result; a self-reported "done" is a claim, not evidence.

Untraced code is either dead work (remove it) or a missing requirement (add the
`REQ` and its verification). Untested code does not ship.

## Iron Laws

1. No level skipped above your tailoring tier.
2. No verification defined *after* the thing it verifies — always paired on descent.
3. No code without a `REQ` it traces to.
4. No `REQ` without a verification.
5. **Verify ≠ Validate** — "built it right" (against requirements) is not "built the
   right thing" (against the ConOps). Keep them separate.

## Artifacts

Write everything under `docs/v-model/<slug>/`:

- `conops.md` — intent + validation scenarios (L1)
- `requirements.md` — `REQ-id`s + system verification plan (L2)
- `design.md` — subsystem + component design + their verification plans (L3, L4)
- `traceability.md` — the live matrix; update Status as you climb the right side

Trivial tier writes only `traceability.md`.
