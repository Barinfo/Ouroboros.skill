# Ouroboros.skill

[简体中文](README.md) | **English**

<div align="center">
  <img src="https://img.shields.io/github/stars/barinfo/Ouroboros.skill?style=social&label=Stars" alt="GitHub Stars" />
  <img src="https://img.shields.io/github/forks/barinfo/Ouroboros.skill?style=social&label=Forks" alt="GitHub Forks" />
  <img src="https://img.shields.io/github/watchers/barinfo/Ouroboros.skill?style=social&label=Watchers" alt="GitHub Watchers" />
  <img src="https://img.shields.io/github/license/barinfo/Ouroboros.skill?color=blue" alt="License: MIT" />
  <img src="https://img.shields.io/github/repo-size/barinfo/Ouroboros.skill" alt="Repo Size" />
  <img src="https://img.shields.io/github/commit-activity/m/barinfo/Ouroboros.skill?color=green" alt="Commit Activity" />
</div>

<div align="center">
  <img src="assets/snake.png" alt="Ouroboros — the serpent that devours its own tail" width="180" />
</div>

<div align="center" style="background-color: #1a1a2e; color: #eaeaea; border-radius: 12px; padding: 24px 16px; border: 1px solid #3a3a5e;">

# 🐍 Ouroboros · Recursive Wake-Up Driver

🌙 **It circles and circles, in every pair of dreamless eyes.** 🐍 — ilem, *"〇"*

Keeps the agent working — and wakes it back up if it stops — until the goal is actually done.

</div>

Ouroboros is **not a code reviewer**. It is the **recursive drive** that makes an autonomous agent *keep working to completion*: it holds one **goal** (长期目标), delegates work to **sub-agents** (handing code review to Camelot and getting the verdict called back), and arms a **timer** (定时器) so that if the agent stops, idles, or gets waylaid partway, **it wakes itself up and continues** until the goal is met.

When code needs judging, Ouroboros **calls Camelot** — it never does the review itself. **Camelot is the blade; Ouroboros is the hand that never lets go of the task.**

| | |
|---|---|
| 📄 Format | Markdown Skill (frontmatter `SKILL.md`) + `skill.json` metadata |
| 🧩 Compatible | Claude Code / Codex / DSH presets / any agent that loads Markdown skills |
| 🌏 Language | 简体中文 (README default); en (skill voice) |
| ⚖️ License | MIT |

---

## Why it exists

A single-conversation agent "says its piece and stops." To make it *keep producing toward a goal that doesn't end*, it needs three things — the three coils of the serpent:

1. **goal** — the objective does not vanish when a turn ends; interruption resumes from the last committed progress, not from zero.
2. **sub-agent** — the main agent must not be the bottleneck: independent sub-problems fan out in parallel / relay; **when code needs judging, spawn a sub-agent running Camelot and thread the verdict back into the main context**.
3. **timer** — **if the agent stops, idles, or goes lazy before the task is done, a timer wakes it up** and makes it keep pushing toward the goal, instead of waiting for a human to poke it.

## The three engines

### 1. goal — hold one long-term objective so the agent doesn't stop
Register the unavoidable long-term objective so the loop is not tied to a single turn. Each round's progress lands in the goal's context; a crash or interruption resumes from the last committed state instead of restarting at zero. The goal is the *raison d'être*: drift away from it is a cardinal failure the loop must correct.

### 2. sub-agent — divide labor (and hand review to Camelot)
Dispatch independent work to child agents (one module / one test slice / one root-cause farm per sub-agent) so the main agent is not a bottleneck:
- when a round of produced code needs judgment, do **not** re-review it in the main agent — spawn a **sub-agent running Camelot** over the diff, let it return the Exile/Dishonor/Counsel/Valour verdict + orders, and **feed that verdict back into the main agent's goal context** before the next round.
- Ouroboros orchestrates *who reviews and when*; it never owns the review itself — that always belongs to Camelot.

### 3. timer — wake yourself up if you stop or idle
The failure mode Ouroboros exists to kill: **agent goes idle/stops partway and no one resumes it.** As long as the goal is unmet:
- before a blocking wait, or when the agent would otherwise hand control back mid-task, arm a timer that re-fires this skill;
- on wake: read the goal → the last round's state → decide `proceed` / `halt` / `escalate`, and **keep going** until the goal is done — not until the wall-clock says "good enough";
- guard against *busy-idle* (running forever without progress): a wake that finds no forward motion for N consecutive fires should escalate, not merely loop louder.

**The timer is what makes the recursion self-started — the ouroboros that bites its own tail to stay alive.**

## The recursive loop protocol

Each cycle emits a compact, machine-friendly state record. Use your harness's actual goal / sub-agent / timer tools (DSH goal · Claude Code subagents/timers · Codex continuable sessions — whichever exist):

```
## 🔁 Ouroboros cycle N

goal_state:
  objective: "<the long-term goal>"
  status: incomplete | complete | blocked
  context_tail: "<last committed round's outcome>"

delegation:
  - to_subagent: camelot      # spawn Camelot review on the latest diff
    result_callback: verdict  # its Verdict + Orders come back into goal context
  - to_subagent: <other task arms…>

wakeup:
  timer_armed: true
  next_wake_on: <event / interval before idle would halt>
  last_progress_commit: <where we are in the goal>

directive: proceed | halt_and_investigate | escalate_to_human
```

### directive — what happens next

| value | meaning | when |
|---|---|---|
| `proceed` | **keep working**; arm the wake timer, dispatch the next sub-agent round | the goal is unmet and there is forward motion |
| `halt_and_investigate` | pause production; dig a root cause (or send a sub-agent to read the dependency source) before re-spinning | whack-a-mole / recurring same-class failure |
| `escalate_to_human` | stop and surface to a person | blocked on something only a human decides, or 3 cycles without forward motion despite prodding |

The default bias is **`proceed`**: unless a genuine `halt` / `escalate` reason exists, the agent keeps itself scheduled and working — that is the recursive promise.

## Anti-idle / anti-laziness oath

Every cycle the agent must honestly answer: *did I advance the goal, or did I stall?*

1. **No "good enough for one turn" exits** — the goal is unmet until it is met; an interrupted turn is not completion.
2. **Wake-up is mandatory before idle** — if the agent would stop with work outstanding, it arms the timer and resumes rather than waiting to be poked.
3. **Review goes to Camelot, not to the loop** — the main agent does not silently re-judge code; it delegates review and consumes the returned verdict.
4. **Carry context forward** — committed progress is where the next round resumes; "last round felt done" is not done.
5. **Busy-idle is not progress** — if the timer keeps firing but nothing narrows, escalate rather than merely looping louder.
6. **Verification over claims** — "made progress" needs an artifact (diff / log / test line), not an assertion.

## Recommended companion

> **Use it with [Camelot.skill](https://github.com/Barinfo/Camelot.skill) for best results.** Camelot is the judge — it rules on a single round of code; Ouroboros is the driver — it keeps the task moving, and when judgment is needed it hands Camelot to a sub-agent and threads the verdict back. If you also gate AI pre-delivery (`pre-delivery-selfcheck` / D1–D12), all three sit side by side: Camelot gives judgment, Ouroboros gives momentum, the delivery gates keep every send honest.

## Install

```bash
# Claude Code
cp SKILL.md ~/.claude/skills/ouroboros/SKILL.md

# DSH custom preset (replace <id>)
cp SKILL.md ~/.dsh/.agent-presets/<id>/skills/ouroboros/SKILL.md
```

## Layout

```
Ouroboros.skill/
├── README.md                # 简体中文 (default)
├── README.en.md             # this file (English)
├── skill.json               # metadata: triggers, keywords, companion
├── SKILL.md                 # the Skill itself — copy to install
└── assets/
    └── snake.png            # ouroboros visual (original artwork)
```

## Strengths

1. **Turns "will it actually get done?" into mechanism, not self-discipline** — goal persists, timer self-wakes; the agent cannot just go silent.
2. **No review duplication** — code judgment is always delegated to Camelot; the main agent is not both player and referee.
3. **Division without bottleneck** — sub-agents run parallel/relay; the main agent is not the chokepoint.
4. **Interruption-safe** — the goal lets a crash resume from the last committed progress, not from zero.
5. **Anti-laziness is codified** — "arm the wake-up before idle" is written into the oath.
6. **Complements Camelot** — judgment there, momentum here.

## Honest limits

1. **Needs a runtime that actually provides goal / sub-agent / timer** — on a bare text-only agent with none of these, it is toothless.
2. **Timer requires host support** — no scheduler of its own; use the host's timer service.
3. **"Actually finishing the work" still depends on the model's execution ability** — it can force the agent not to stop, but output quality rests on the underlying model.
4. **English-primary skill voice** — teams in other languages should localize.
5. **Prevents "spinning", not "wrong direction"** — it will not correct a goal that is itself mistaken.

## When NOT to use it

- One-off code review → use **Camelot**.
- Runtime offers no goal / sub-agent / timer to attach → it cannot drive recursion.
- You want emotional support → this is the wrong blade.

## License

[MIT](LICENSE) · © 2026 FuCube

*The ouroboros snake artwork is original by Barinfo and may be reused with this skill.*
