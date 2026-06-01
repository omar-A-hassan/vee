# Worked Examples

## Feature-tier: "Add rate limiting to the public API"

**Phase 0 — Tailor:** `feature` (one coherent capability, single subsystem).

**DESCEND**

`docs/v-model/rate-limiting/conops.md`
> **Why:** the public API is being abused by bursty clients, degrading service for
> everyone. **Success (validation scenarios):**
> - V-SCEN-1: a client exceeding 100 req/min receives HTTP 429 with a `Retry-After`.
> - V-SCEN-2: a client under the limit is never throttled.

`docs/v-model/rate-limiting/requirements.md`
> - **REQ-1:** Reject requests over 100/min per API key with HTTP 429.
>   *Verify:* drive 101 requests in <60s, assert the 101st is 429. (V-1, system)
> - **REQ-2:** Include a `Retry-After` header on every 429.
>   *Verify:* assert header present and integer-valued on the 429. (V-2, system)
> - **REQ-3:** Requests under the limit pass through unchanged.
>   *Verify:* 100 requests in 60s all return their normal status. (V-3, system)

`docs/v-model/rate-limiting/design.md`
> **Subsystem:** a `RateLimiter` middleware in front of the request handler.
> Allocates REQ-1, REQ-2, REQ-3.
> *Subsystem verification (V-4, integration):* middleware mounted, a burst trips it
> and a 429 reaches the client through the real stack.
> **Component — `RateLimiter`:** token-bucket keyed by API key; `allow(key) -> bool`,
> `retry_after(key) -> int`.
> *Component verification procedure (unit):*
> - `test_allows_under_limit` — 100 calls to `allow` return True.
> - `test_blocks_over_limit` — the 101st call returns False.
> - `test_retry_after_positive` — after blocking, `retry_after` > 0.

**Traceability matrix** (`docs/v-model/rate-limiting/traceability.md`)

| REQ | Subsystem | Component | Verification ID | Status |
|-----|-----------|-----------|-----------------|--------|
| REQ-1 | RateLimiter mw | RateLimiter | V-1 (system), test_blocks_over_limit (unit) | ☐ |
| REQ-2 | RateLimiter mw | RateLimiter | V-2 (system), test_retry_after_positive (unit) | ☐ |
| REQ-3 | RateLimiter mw | RateLimiter | V-3 (system), test_allows_under_limit (unit) | ☐ |

**IMPLEMENT** — write `RateLimiter` and the three unit tests named above.

**CLIMB**
- R4 Component: run the 3 unit tests → pass.
- R3 Subsystem: run V-4 integration → a burst returns 429 through the stack.
- R2 System: run V-1/V-2/V-3 → each REQ shows PASS with the command + output.
- R1 Validation: replay V-SCEN-1 and V-SCEN-2 → both hold.

**Done gate:** run the project's real test command — it must exit `0` and every REQ's
named test must be one it actually ran — then replay V-SCEN-1/V-SCEN-2. Only then flip
each Status to ☑. The verdict is the runner's exit code, not the checkmarks.

## System-tier: "A spreadsheet formula engine"

**Phase 0 — Tailor:** `system` (multiple interacting subsystems). Decompose into
sub-Vs under one system-level V: one ConOps and one system-level validation, and each
subsystem gets feature-tier treatment.

**DESCEND**

`conops.md` (one, system-level)
> **Why:** an embeddable engine where cells hold numbers or formulas, and changing a
> cell re-evaluates every formula that transitively depended on it.
> - SV-9: `A1→B1→C1→D1` chained; change `A1=10`; `get_value('D1')` recomputes to `13`.
> - SV-4: `A1='=B1'`, `B1='=A1'` → `set_cell` raises `ValueError` (no infinite loop).

`design.md` — four subsystems, every REQ allocated:
> - **Parser** (formula → AST) ← REQ-2, REQ-16
> - **Evaluator** (AST + cell values → number) ← REQ-5..11, REQ-14, REQ-15
> - **DependencyGraph** (edges, cycle detection, topo order) ← REQ-12, REQ-13
> - **CellStore** (raw + cached values) ← REQ-1, REQ-3, REQ-4

**Traceability matrix** (excerpt — one row per REQ, allocated to its subsystem)

| REQ | Subsystem | Component | Verification ID | Status |
|-----|-----------|-----------|-----------------|--------|
| REQ-12 (recompute transitive dependents) | DependencyGraph | DependencyGraph | `test_update_propagates` | ☑ |
| REQ-13 (circular dependency raises) | DependencyGraph | DependencyGraph | `test_circular_raises` | ☑ |
| REQ-6 (operator precedence) | Evaluator | Evaluator | `test_add_mul_precedence` | ☑ |

**CLIMB** (bottom-up, per subsystem then whole-system)
- R4 Component: each subsystem's unit tests pass.
- R3 Subsystem: integration — `set_cell` → Parser → DependencyGraph → no crash; a cycle path raises through the real stack.
- R2 System: the real test command exits `0`; every REQ's named test ran.
- R1 Validation: replay SV-9 (chained recompute) and SV-4 (cycle raises).

**Done gate:** same as feature — the runner's exit code decides, across all sub-Vs,
then the system-level validation scenarios replay.

## Trivial-tier: "Fix typo: button says 'Sumbit'"

**Phase 0 — Tailor:** `trivial`. No ConOps, no requirements, no design files.

Single file `docs/v-model/fix-submit-typo/traceability.md`:

> **Intent:** the submit button label reads "Sumbit"; it should read "Submit".
>
> | Intent | Change | Verification | Status |
> |--------|--------|--------------|--------|
> | Correct button label | edit label string | grep the rendered/template text shows "Submit" and not "Sumbit" | ☑ |

Implement the one-character fix, run the grep verification, flip to ☑. Done.
