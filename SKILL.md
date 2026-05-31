---
name: verification-toolchain-selector
description: Pick the optimal verification/testing toolchain for a project BEFORE implementation, by the "shortest agent-visible feedback loop" criterion. Two modes — Default (curated lookup for known stacks, cheap) and Escalate (role-structured cross-vendor consensus: Claude + Codex + Gemini propose → web-ground → score → cross-critique → consensus, for novel/uncertain stacks). The winner is frozen into memory-bank/tech-stack.md and enforced by an AGENTS.md always-rule so agents never drift to a slow/GUI tool (e.g. Xcode GUI debugging an Expo RN app instead of Maestro + simulator). Triggers on "verification toolchain", "how should we verify", "pick test harness", "/verification-toolchain-selector", or from vibe-coding Phase 0.6 (required toolchain freeze).
argument-hint: "[default|escalate] <project type / stack>"
---

# Verification Toolchain Selector

Picks **how a project's steps get verified** — the test/QA harness — and freezes that decision before
the implementation loop starts. The goal is to eliminate the failure mode where agents reach for an
**inefficient verification tool** (slow input→signal, no realtime error console visible to the agent,
GUI-only) when a far tighter loop exists.

> **Canonical example.** Expo / React Native app. **Efficient:** Maestro flows + the simulator on this
> machine (scriptable, realtime logs the agent can read, short feedback loop). **Inefficient:** running
> it through the Xcode GUI and debugging there (slow input, the agent can't see the error console in
> realtime). The selector must pick the former and forbid the latter.

## Selection criterion (the one metric that decides)

Rank candidate harnesses by **shortest agent-visible feedback loop**:

1. **Realtime error/console visibility to the agent** — the agent can read failures programmatically (stdout/logs/exit codes), not via a human-only GUI.
2. **Latency input→signal** — how fast a change produces a pass/fail signal.
3. **Headless / scriptable** — runs without a GUI; automatable in the loop and in CI.
4. **CI compatibility** — the same harness runs unattended.
5. **Maintenance cost** — setup/flakiness/upkeep.

A tool that fails (1) or (3) is disqualified regardless of other merits (this is why Xcode GUI debugging loses).

## Mode is the user's choice

When this runs, **ask the user: Default or Escalate.** Do not auto-pick the mode.

| Mode | When | Cost |
|------|------|------|
| **Default** | Known stack with a well-established harness | Cheap, deterministic — a table lookup |
| **Escalate** | Novel/uncertain stack, conflicting options, or the user wants maximum rigor | Expensive — cross-vendor consensus (many tokens, on purpose) |

---

## Default mode — curated lookup

Map the stack to its established harness. Confirm with the user, then freeze. Starter table (extend as needed):

| Stack / type | Default verification harness |
|--------------|------------------------------|
| Expo / React Native (iOS/Android) | **Maestro flows + simulator/emulator on this machine** + EAS build for release; Jest for units. **Not** Xcode GUI. |
| Native iOS (Swift) | XCUITest **headless via `xcodebuild test`** (CLI, not the GUI) + simulator |
| Web app (React/Next/Vue/Svelte) | **Playwright** (headless) for E2E + component tests; Vitest/Jest units |
| Backend / REST / GraphQL API | contract/HTTP tests (httpie/supertest/pytest), **run headless**; schema checks |
| CLI tool | **bats** / `pexpect` / golden-file tests against the built binary |
| Library / SDK | the language's test runner headless (vitest/pytest/cargo test/go test) + public-API snapshot |
| Game | headless logic tests + scripted scene/action checks where the engine allows |
| MCP server / agent | protocol/tool-call harness exercising tools+resources headless |

If the stack isn't in the table or the user is unsure → recommend **Escalate**.

---

## Escalate mode — role-structured cross-vendor consensus (HARD GUARDRAILS)

Based on `stochastic-multi-agent-consensus`, but **cross-vendor** (different model families = different
blind spots) and **role-structured** (not 30 blind votes — that's weak signal per token for a
convergent question). Defaults: **5 agents per vendor (15 total)**, **web search mandatory**.

### Mechanism (non-negotiable)

- **Claude** agents → spawned via the `Task` tool (`general-purpose`, sonnet).
- **Codex** agents → via `omc ask codex "<prompt>"` (CLI; skill nesting unsupported).
- **Gemini** agents → via `omc ask gemini "<prompt>"` (CLI).
- Artifacts land in `.omc/artifacts/ask/`; Claude reads and aggregates.
- If a vendor's CLI is absent → run that vendor with 0 agents and **note the missing perspective** (never silently treat a 2-vendor run as full consensus).

### The pipeline (run in order)

1. **Propose** — each agent independently proposes a harness for the stack + a one-line rationale. (5×Claude via Task, 5×Codex + 5×Gemini via `omc ask`.) Use framing variations (risk-averse, first-principles, CI-focused, DX-focused, contrarian) so same-vendor agents aren't identical.
2. **Web-ground** — a subset (2–3 per vendor) MUST web-search *current* best practice (tools change fast; cite year). Findings are shared back so proposals are grounded, not from stale memory.
3. **Score** — every proposal is scored on the 5 criteria above (each 1–5). This makes aggregation mechanical. Disqualify any option failing criterion 1 or 3.
4. **Cross-critique** — each vendor critiques the *other* vendors' top picks (catches vendor bias).
5. **Consensus** — aggregate: **mode** (the winner most agents + highest mean score converge on), **divergence** (genuine judgment calls → surface to user), **outlier** (creative single-agent ideas → note, don't auto-adopt).

### Output

A ranked recommendation with the winning harness, its criteria scores, the dissent, and citations from the web-grounding step. Write the consensus report to `.omc/artifacts/verification-toolchain/consensus.md`.

---

## HARD GUARDRAILS (apply to BOTH modes)

These are strict — violating any one means the selection is invalid:

1. **Decide once, freeze.** The chosen harness is written to `memory-bank/tech-stack.md` under a
   `## Verification Toolchain` section, **before** the implementation loop starts. It does not change mid-loop.
2. **Enforce via AGENTS.md.** Add the always-rule: *"Always verify via the toolchain defined in
   memory-bank/tech-stack.md. Never substitute a slower or GUI-only tool (e.g. Xcode GUI debugging) for
   the defined harness. If the defined tool genuinely cannot run, stop and ask the user — do not improvise a slower path."*
3. **Criterion is mandatory.** The decision MUST be justified against the 5 criteria, with the
   feedback-loop length explicit. "It's familiar" is not a valid reason.
4. **User confirms.** Both modes end with the user approving the frozen choice — it's a long-lived decision.
5. **Disqualify GUI-only / no-realtime-console tools** unless the user explicitly overrides with a reason recorded in `tech-stack.md`.
6. **Escalate ≠ silent partial.** If a vendor is unavailable, the report must say so; never present a degraded run as full cross-vendor consensus.

## Configuration

```jsonc
{
  "verificationToolchain": {
    "agentsPerVendor": 5,        // Escalate: Claude/Codex/Gemini each
    "webSearchRequired": true,   // Escalate: grounding step is mandatory
    "vendors": ["claude", "codex", "gemini"]  // DeepSeek added later
  }
}
```

## Related

- `stochastic-multi-agent-consensus` — the base voting mechanism this extends cross-vendor.
- `ccg` — Codex+Gemini+Claude synthesis; the lighter cousin used for spec review (Phase 0.5).
- `vibe-coding` — calls this at the **required Phase 0.6** (Default vs Escalate; always runs before the loop) and consumes the frozen `tech-stack.md` toolchain at the step green gate.
- `expo-react-native-verification-gate-generator`, `verification-core-claim-evidence` — reuse for mobile/native specifics rather than rederiving.
