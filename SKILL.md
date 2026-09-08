---
name: ouroboros
description: The recursive wake-up driver for autonomous agents — keeps an agent working toward a long-term GOAL, fans work out to SUB-AGENTS (handing code review to Camelot and getting the verdict fed back), and arms a TIMER that wakes the agent back up if it stops or goes idle before the goal is met. Named for the serpent devouring its own tail: an endless working loop that, fed right, keeps the agent grinding to true completion. Use when an agent should keep itself working across interruptions, delegate, re-engage after idle, and not stall out mid-task.
---

# Ouroboros: Recursive Wake-Up Driver

> *The serpent devours its own tail so it never runs out of itself. So too must an agent that has been set a goal — it keeps working, divides its labor, and if it stops, it wakes itself up and continues until the goal is done.*

Ouroboros is **not a code reviewer**. It is the **recursive drive** that makes an autonomous agent *keep working to completion*:

- it holds one **goal** (长期目标) the agent keeps pushing toward across many rounds,
- it **delegates** discrete work to **sub-agents** — including handing the actual *code review* to **Camelot** and getting the verdict called back,
- and it arms a **timer** (定时器) so that if the agent stops, idles, or gets waylaid partway, **it wakes itself up and continues** until the goal is actually met.

When a review is needed, Ouroboros **calls Camelot** rather than doing the review itself. Camelot is the blade; Ouroboros is the hand that never lets go of the task.

## Division of labor (this is the whole point)

| | Camelot | **Ouroboros** |
|---|---|---|
| Role | judge code / PR worth | keep the agent **working until done** |
| Reviews | code | nothing itself — delegates review to Camelot |
| Doctor | one pass | recursively re-engages round after round |
| If it stalls | — | **timer wakes it** |
| Long-term | — | holds a **goal** across rounds |
| Fans out | — | sends sub-tasks to **sub-agents** |
| Relation to Camelot | the review authority | **invokes Camelot** and feeds the verdict back |

**One mental model:** Camelot answers *"is this code worthy?"*; Ouroboros answers *"will this task actually get finished — and wake itself up if not?"*

## The three engines

### 1. goal — hold one long-term objective so the agent doesn't stop
- Register the unavoidable long-term objective so the loop is not tied to a single turn. Each round's progress lands in the goal's context; a crash or interruption resumes from the last committed state instead of restarting at zero.
- The goal is the *raison d'être*: the agent keeps producing toward it. Round output that drifts from the goal is a "cardinal" failure the loop must correct.

### 2. sub-agent — divide labor (and hand review to Camelot)
Dispatch independent work to child agents so the main agent is not a bottleneck:
- one module / one test slice / one root-cause farm per sub-agent,
- the requester gathers each sub-agent's result into the main context.

**Review delegation pattern (recommended):** when a round of produced code needs judgment, do *not* re-review it in the main agent. Spawn a **sub-agent running Camelot** over the diff, let it return the Treason/Dishonor/Counsel/Valour verdict + orders, and **feed that verdict back into the main agent's goal context** before the next round. Ouroboros orchestrates *who reviews and when*; it never owns the review itself.

### 3. timer — wake yourself up if you stop or idle
The failure mode Ouroboros exists to kill: **agent goes idle/stops partway and no one resumes it.** Set a recurring self-wake timer for as long as the goal is unmet:
- before a blocking wait, or when the agent would otherwise hand control back mid-task, arm a timer that re-fires this skill;
- on wake, read the goal → the last round's state → decide `proceed`/`halt`/`escalate`, and **keep going** until the goal is done, not until the wall-clock says "good enough";
- guard against *busy-idle* (running forever without progress): a wake that finds no forward motion for N consecutive fires should escalate instead of merely looping.

The **timer** is what makes the recursion *self-started* — the ouroboros that bites its own tail to stay alive.

## The recursive loop protocol

Each cycle produces a compact state record (machine-friendly). Use your harness's actual goal/sub-agent/timer tools (DSH goal · Claude Code subagents/timers · Codex continuable sessions — whichever exist):

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
| `proceed` | **keep working**; arm wake timer, dispatch next sub-agent round | the goal is still unmet and there is forward motion |
| `halt_and_investigate` | pause production; dig a root cause (or send a sub-agent to Merlin/read-source) before re-spinning | whack-a-mole / recurring same-class failure |
| `escalate_to_human` | stop and surface to a person | blocked on something only a human decides, or 3 cycles without forward motion despite prodding |

The default bias is **`proceed`**: unless a genuine `halt`/`escalate` reason exists, the agent keeps itself scheduled and working — that is the recursive promise.

## Anti-idle / anti-laziness oath

Every cycle the agent must honestly answer: *did I advance the goal, or did I stall?*

1. **No "good enough for one turn" exits** — the goal is unmet until it is met; a single interrupted turn is not completion.
2. **Wake-up is mandatory before idle** — if the agent would stop with work outstanding, it arms the timer and resumes rather than waiting to be poked by a human.
3. **Review goes to Camelot, not to the loop** — the main agent does not silently re-judge code; it delegates review and consumes the returned verdict.
4. **Carry context forward** — nothing is "done because last round felt done"; committed progress is where the next round resumes.
5. **Busy-idle is not progress** — if the timer keeps firing but nothing narrows (same class of issue), escalate, don't merely loop louder.
6. **Verification over claims** — "made progress" needs an artifact (diff / log / test line), not an assertion.

## Quick reference

| Situation | What Ouroboros does |
|---|---|
| Long task, must survive interruptions | hold a **goal**, resume from last committed round |
| Lots of independent work | fan out to **sub-agents** |
| Code needs judgment in a round | delegate to a **sub-agent running Camelot**, feed verdict back |
| Agent stops / idles mid-task | **timer** wakes it; it continues toward the goal |
| Recurring failure, no narrowing | `halt_and_investigate` → root cause |
| Same issue persists 3 cycles | `escalate_to_human` |
