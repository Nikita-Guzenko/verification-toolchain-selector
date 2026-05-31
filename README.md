# verification-toolchain-selector (Claude Code skill)

A [Claude Code](https://claude.com/claude-code) skill that picks a project's **verification /
testing toolchain BEFORE implementation**, by one criterion: the **shortest agent-visible feedback
loop**. The choice is frozen into `memory-bank/tech-stack.md` and enforced by an `AGENTS.md`
always-rule, so AI agents never drift to a slow or GUI-only tool mid-loop.

> **Why it exists.** Agents otherwise reach for inefficient verification — e.g. debugging an Expo /
> React Native app through the **Xcode GUI** (slow input, no realtime error console the agent can
> read) instead of **Maestro flows + the simulator** (scriptable, realtime logs, tight loop). This
> skill decides the right harness up front and forbids the slow path.

## Selection criterion

Rank candidate harnesses by: realtime error/console visibility to the agent · input→signal latency ·
headless/scriptable · CI compatibility · maintenance cost. A GUI-only / no-realtime-console tool is
disqualified.

## Two modes (user chooses)

- **Default** — curated lookup for known stacks (cheap, deterministic). E.g. Expo RN → Maestro + simulator; web → Playwright; API → headless contract tests.
- **Escalate** — role-structured **cross-vendor consensus** for novel/uncertain stacks: Claude (via `Task`) + Codex + Gemini (via `omc ask`), default 5 agents/vendor, **web-grounded**, pipeline `propose → web-ground → score → cross-critique → consensus`.

## Hard guardrails

Decide-once-and-freeze · enforce via `AGENTS.md` · criterion mandatory · user confirms · disqualify
GUI/no-console tools · Escalate never presents a degraded (missing-vendor) run as full consensus.

## Install

```bash
git clone https://github.com/Nikita-Guzenko/verification-toolchain-selector.git
cp -R verification-toolchain-selector ~/.claude/skills/verification-toolchain-selector
```

Invoke with `/verification-toolchain-selector`, or it is called from the
[`vibe-coding`](https://github.com/Nikita-Guzenko/vibe-coding-skill) skill at Phase 0.5.

## Related

Built to pair with the `vibe-coding` skill (durable memory-bank methodology). Extends
`stochastic-multi-agent-consensus` (cross-vendor) and complements `ccg`.

## Author / Connect

Built by **Nik Guzenko**.

- 🐦 X / Twitter: [@NikGuzenko](https://x.com/NikGuzenko)
- 💬 Telegram: [@nikguzenko](https://t.me/nikguzenko)

If this skill saves you time, a ⭐ on the repo helps others find it.

## Keywords

Claude Code skill · AI agents · agentic coding · verification · testing · QA · test harness ·
Maestro · Expo · React Native · Playwright · Codex · Gemini · multi-agent consensus · LLM tooling ·
feedback loop · vibe coding · oh-my-claudecode.
