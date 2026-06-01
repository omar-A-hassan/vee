<div align="center">

# Vee

**The NASA systems-engineering V-Model, as a Claude Code skill — spec-driven development with a done-gate decided by your real tests, not self-report.**

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-d97757)

</div>

A Claude Code plugin that runs a coding task through the NASA V-Model: define each
level *and its verification* on the way down, implement at the bottom, then climb back
up verifying — and refuse to call it done until the project's real test command passes
and every requirement traces to a test that command actually runs. Install it, type
`/v-model`, and describe what you want built.

## Contents

- [The problem](#the-problem) · [The solution](#the-solution) · [Not just planning](#not-just-planning) · [Tailoring](#tailoring-no-ceremony-on-small-tasks) · [Install](#install) · [Usage](#usage) · [What it produces](#what-it-produces) · [The done-gate](#the-done-gate) · [How it differs](#how-it-differs) · [Limitations](#limitations)

> *The systems-engineering "Vee" was formalized by Forsberg & Mooz (1991) — who named
> it the **Vee**, hence this project's name — and adopted in NASA's Systems Engineering
> Handbook.*

## The problem

LLMs build fast but leave you unable to *trust* the result:

- they make silent assumptions and run with them;
- they claim "done" with no evidence — **believable but unverified**;
- they produce code you can't trace back to any requirement;
- they collapse a problem's distinct levels into one mushy "plan."

## The solution

Bring the discipline of systems engineering to the agent — five guardrails that each
target one of those failures:

| Discipline | Addresses |
|---|---|
| **Leveled altitude** | mushy, collapsed planning |
| **Paired verification on descent** | tests written late, or never |
| **Traceability spine** | untraceable code, orphan requirements |
| **Delegated done-gate** | "believable but unverified" completion claims |
| **Tailoring gate** | ceremony on trivial tasks |

## Not just planning

"Write a spec, build, then test" is already everywhere. The V-Model earns its place
with three disciplines planning modes skip:

1. **Leveled altitude** — ConOps → System Requirements → Subsystem/High-Level Design
   → Component Detailed Design are *distinct* levels. The skill won't let you collapse
   them into one mushy plan.
2. **Paired verification, defined on the way down** — each level writes its own
   verification plan *at the same altitude and same moment*, before descending. You
   never write a test after the thing it tests.
3. **Traceability spine** — a matrix maps every requirement to the verification that
   proves it, and every line of code back to a requirement. Nothing untraced ships;
   nothing untested ships.

```
 DESCEND (define + plan verification together)        CLIMB (verify, then validate)
   ConOps            <----- validates ----->            System Validation
   System Reqs       <----- verifies ----->             System Verification
   Subsystem Design  <----- verifies ----->             Subsystem Verification
   Component Design  <----- verifies ----->             Component Verification
                     \                     /
                       IMPLEMENT (bottom of the V)
                     spine: docs/v-model/<slug>/traceability.md
```

## Tailoring (no ceremony on small tasks)

The skill classifies scope first, so it never sledgehammers a one-liner:

- **trivial** (one-liner, config) → a single traceability file + one test. No ceremony.
- **feature** → the full single V.
- **system** → decompose into sub-Vs under one system-level V.

## Install

Claude-first. Two ways:

**As a plugin (persistent):**
```
/plugin marketplace add /path/to/this/repo
/plugin install v-model@v-model-skills
```
Then `/v-model` is available in any session. Use `/plugin` to manage it.

**For one session (no install):**
```bash
claude --plugin-dir /path/to/this/repo
```

**As a personal skill:** copy `skills/v-model/SKILL.md` into your personal skills
directory and invoke it with `/v-model`.

## Usage

```
/v-model add rate limiting to the public API
```

The skill states its tailoring decision, descends the left side writing the artifacts
below, implements, then climbs the right side updating the matrix until every
requirement shows a passing verification — and finally applies the done-gate.

See [EXAMPLES.md](EXAMPLES.md) for a full worked run.

## What it produces

Per task, under `docs/v-model/<slug>/`:

| File | Level | Contents |
|------|-------|----------|
| `conops.md` | L1 | why + validation scenarios (user-facing acceptance) |
| `requirements.md` | L2 | `REQ-id`s + how each is verified |
| `design.md` | L3/L4 | subsystems, components, and the unit tests named *before* coding |
| `traceability.md` | spine | the live matrix: `REQ → component → test → status` |

(Trivial tier writes only `traceability.md`.)

## The done-gate

What makes this more than paperwork is *how* it decides "done" — by delegating to your
project's real tools, the same way Spec Kit and Kiro do, never by a bespoke checker:

1. **Every `REQ` names how it is verified** — a concrete test (or type-check / lint /
   build / explicit human review). If a requirement can't name its verification, it
   isn't granular enough.
2. **The verdict is your real test command's exit code.** The matrix closes only when
   that command exits `0` *and* every `REQ`'s named test is one it actually runs.
   Determinism comes from the test runner, not from prose.
3. **Tests must be able to fail for the right reason** — written red-green (failing
   first), and property-based for edge-heavy requirements. A test that never failed,
   or only checks one hand-picked input, proves little.

The strongest signal is **independent**: someone other than the implementer runs the
command and reads the result. A self-reported "done" is a claim, not evidence.

## How it differs

A deliberately lightweight, tool-agnostic take on spec-driven development (cf.
GitHub Spec Kit, AWS Kiro). It is *not* trying to out-feature them. Its edges:
- a **done-gate that delegates to your real test runner** instead of trusting the agent;
- a **tailoring gate** so small tasks stay cheap;
- an explicit **verify ≠ validate** split ("built it right" vs "built the right thing").

## Limitations

Stated plainly, because the point of this tool is honesty:
- **It instructs; it doesn't enforce.** A skill asks the agent to follow the gate — it
  can't guarantee it. For hard enforcement, wire your test command into a `Stop` hook.
- **It checks that tests pass, not that they're meaningful.** A test that passes while
  asserting the wrong thing is invisible to any exit-code gate — red-green and
  property-based discipline mitigate this, mutation testing and human review close it.
- **It does not make code more correct** — its value is auditability and a provable
  paper trail, worth its overhead on long-lived/handed-off/regulated work and overkill
  on throwaways (hence the tailoring gate).

## Pairs well with

Optional: the [superpowers](https://github.com/obra/superpowers) skills
(brainstorming, writing-plans, TDD) complement this workflow but are **not** required.

## Other tools

Cursor (`.cursor/rules/*.mdc`) and Codex (`AGENTS.md`) packaging are planned, not yet included.

## License

MIT — see [LICENSE](LICENSE).
