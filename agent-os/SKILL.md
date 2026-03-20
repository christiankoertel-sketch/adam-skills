---
name: agent-os
description: >
  Build autonomous AI agent operating systems with self-healing, automated fix protocols,
  tiered thinking, zombie detection, and structured operational loops. Use this skill whenever
  someone wants to build a long-running autonomous agent, set up a CLAUDE.md for a complex
  project, create self-healing infrastructure, design agent monitoring and recovery systems,
  build an agent that runs on a schedule (cron, launchd, scheduled tasks), or wants their
  AI agent to operate reliably without constant human supervision. Also trigger when the user
  mentions agent reliability, agent orchestration, autonomous operations, self-healing systems,
  or wants to make their Claude Code / AI agent project production-grade and resilient.
---

# Agent Operating System

Build production-grade autonomous AI agents that run reliably without constant human babysitting. This skill encodes battle-tested patterns from an agent system that has run 100+ daily execution windows across 25+ scheduled tasks for months — including the hard-won lessons from outages, zombie processes, and silent failures.

## The Core Problem This Solves

Most AI agent projects work great during development and die quietly in production. The agent runs for a day, hits an error, and stops. Nobody notices for a week. This skill teaches you to build agents that detect their own failures, fix themselves automatically, and only escalate to humans when genuinely stuck.

## Architecture Overview

A production autonomous agent needs five layers:

```
Layer 5: Strategic Thinking (weekly Opus consultations, gap detection)
Layer 4: Operational Loop (scheduled tasks, cron, daily cycles)
Layer 3: Self-Healing (error detection, auto-fix, escalation)
Layer 2: Process Management (zombie detection, heartbeats, restarts)
Layer 1: Foundation (CLAUDE.md, config files, state tracking)
```

Build bottom-up. Each layer depends on the ones below it. Don't try to add self-healing (Layer 3) before you have solid process management (Layer 2).

## Layer 1: Foundation — The CLAUDE.md Operating System

Your CLAUDE.md file is the agent's operating system. It tells the agent who it is, what it's responsible for, and how to behave. A well-structured CLAUDE.md means the agent can pick up any task without losing context.

### CLAUDE.md Structure

Organize the CLAUDE.md with these sections, in this order:

**1. Before Any Work** — What files to read before doing anything. The agent needs context before it can act intelligently.
```markdown
## Before Any Work
1. Read STATUS.md — know current state
2. Read relevant config files — know the rules
3. Read the roadmap — know the endgame
```

**2. Core Rules** — Non-negotiable behaviors. Keep these sharp and unambiguous.
```markdown
## Core Rules
- Always update STATUS.md after making changes
- Back up files before overwriting (cp file file.bak)
- One thing at a time — complete, verify, move to next
- Never modify architecture docs without explicit approval
```

**3. Task Routing** — What the agent handles autonomously vs. what needs human input. Be explicit about boundaries.
```markdown
## Handle Autonomously
- Bug fixes in existing code
- Running pipelines and reporting results
- Updating status files
- Writing tests

## Escalate to Human For
- Architecture decisions
- New module design
- Security changes
- Anything over $X cost
```

**4. Execution Mode** — How the agent should behave during autonomous runs.
```markdown
## Execution Mode
- Run autonomously — don't ask for approval on routine operations
- Only stop if: a test fails, a pipeline breaks, or a decision is ambiguous
- When encountering dependency issues, resolve them
- After completing any task, run relevant tests, then update STATUS.md
```

**5. Security Boundaries** — What the agent must never do, even if instructed.
```markdown
## Security Boundaries
- Never modify identity files without explicit approval
- Never commit secrets or credentials
- All data must include provenance metadata
```

**6. Domain-Specific Sections** — Rules specific to what the agent actually does (trading rules, content rules, deployment rules, etc.)

### CLAUDE.md Anti-Patterns

These patterns cause subtle agent failures over time:

- **Too vague:** "Be careful with data" → Agent doesn't know what "careful" means. Instead: "All ChromaDB items must include provenance metadata: source_url, source_date, ingestion_date."
- **Contradictory rules:** "Run autonomously" + "Always ask before writing files" → Agent freezes. Resolve conflicts explicitly.
- **Missing escalation paths:** Rules that say "don't do X" without saying what to do instead. Always provide the alternative action.
- **Stale instructions:** Rules that reference deleted files, deprecated modules, or old workflows. Review CLAUDE.md monthly.

## Layer 2: Process Management

Any long-running process will eventually crash. The question is whether it restarts automatically or sits dead until a human notices.

### Heartbeat Pattern

Every long-running process should write a heartbeat file at regular intervals:

```python
import json
from datetime import datetime, timezone
from pathlib import Path

def write_heartbeat(process_name: str, status: str = "alive"):
    heartbeat = {
        "process": process_name,
        "status": status,
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "pid": os.getpid()
    }
    path = Path(f"data/heartbeats/{process_name}.json")
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text(json.dumps(heartbeat))
```

**Staleness detection:** If a heartbeat file is older than 2x the expected interval, the process is presumed dead. A watchdog checks this and triggers restart.

### Zombie Detection

A zombie process is one that appears alive (PID exists, heartbeat updates) but isn't actually doing useful work. This is harder to detect than a crash. Common causes: infinite retry loops, swallowed exceptions, blocked I/O.

**Detection patterns:**
- **Output staleness:** If the process should produce output every N minutes but hasn't, it's zombified
- **Exception swallowing:** Catch-all `except Exception: pass` blocks are the #1 cause. Every exception handler should either fix the problem or log it and re-raise after N retries
- **Circuit breaker:** After N consecutive errors (e.g., 20), exit the process and let the supervisor restart it fresh

```python
class CircuitBreaker:
    def __init__(self, max_consecutive_errors=20):
        self.consecutive_errors = 0
        self.max = max_consecutive_errors

    def record_success(self):
        self.consecutive_errors = 0

    def record_error(self, error):
        self.consecutive_errors += 1
        if self.consecutive_errors >= self.max:
            raise SystemExit(f"Circuit breaker: {self.consecutive_errors} consecutive errors. Last: {error}")
```

### 4-Layer Resilience Stack

For critical processes, use four independent restart layers:

| Layer | What | Recovery Time | Tool |
|-------|------|--------------|------|
| 1. OS-level | Process manager (launchd/systemd) | Immediate | KeepAlive=true / Restart=always |
| 2. Supervisor | Parent script monitors children | 30-60 seconds | Shell script checking PIDs |
| 3. Application outer | Retry loop with exponential backoff | 30-600 seconds | Python try/except with sleep |
| 4. Application inner | Per-operation error handling | Immediate | Try/except per API call |

Each layer catches failures that slip through the layer above it. The innermost layer handles transient errors (API timeouts). The outermost layer handles total crashes (segfaults, OOM kills).

## Layer 3: Self-Healing

Self-healing means the agent detects problems and fixes them automatically, without human intervention. This is the difference between an agent that needs daily babysitting and one that runs for weeks unattended.

### The Fix Hierarchy

When the agent detects a problem, apply fixes in this order:

| Priority | Action | Example |
|----------|--------|---------|
| 1. Auto-fix immediately | Reset counter, restart process, rotate logs | Stale state file → delete and regenerate |
| 2. Auto-fix at next cycle | Write fix script for cron to execute | Config drift → write correction script |
| 3. Auto-fix + notify | Fix it AND tell the human it happened | Budget exceeded → cap spending + email |
| 4. Propose fix for approval | Only for irreversible or expensive actions | Architecture change, >$5 spend |
| 5. Escalate to human | Truly can't automate the solution | Needs physical access or account change |

The key insight: most agent failures are fixable without human help. Default to auto-fix, not escalation.

### Every Fix Needs Three Components

A fix that doesn't prevent recurrence is a band-aid. Every fix must include:

| Component | What It Does | Example |
|-----------|-------------|---------|
| **The Fix** | Resolves the immediate problem | Restart the dead process |
| **The Monitor** | Detects if the problem returns | Check heartbeat file every 30 minutes |
| **The Auto-Recovery** | Fixes it next time without humans | Supervisor script auto-restarts on heartbeat staleness |

If a fix is missing the monitor or auto-recovery, it's incomplete.

### Tiered Thinking for Fixes

Don't use the same "brain" for every problem. Simple fixes shouldn't burn expensive API calls. Complex failures need deeper analysis.

| Level | Trigger | Response |
|-------|---------|----------|
| 0 | Known fix, first occurrence | Hardcoded auto-fix. No API call. Reset/restart/rotate. |
| 1 | Known fix, recurring | Auto-fix AND investigate why it keeps happening |
| 2 | Unknown issue, first detection | Analyze the problem, propose fix with monitor |
| 3 | 3+ failed fix attempts | Deeper analysis — standard fixes failed, need novel approach |
| 4 | Systemic/architectural failure | May need entirely new architecture. Research alternatives. |

**Auto-escalation:** If the same issue appears 3+ times in 48 hours at the same level, automatically escalate to the next tier.

### Drift Detection

"Drift" is when the agent's configuration, data, or behavior gradually diverges from what it should be. It's the silent killer of autonomous systems.

**Types of drift to monitor:**

- **Config drift:** Values in config files no longer match current objectives. Fix: Validate configs against objectives at every cycle start.
- **Data drift:** Files that should be fresh are stale. Fix: Check timestamps against expected refresh intervals. Auto-trigger refresh if stale.
- **Code drift:** A previous fix gets reverted or overwritten. Fix: Add regression tests that fail if the fix disappears.
- **Process drift:** A service that should be running has stopped. Fix: Heartbeat checks with auto-restart.
- **Priority drift:** The agent is working on outdated priorities. Fix: Re-read priority config at every cycle start.

## Layer 4: Operational Loop

The agent needs a structured execution cycle — a rhythm of check, plan, execute, report.

### Cron / Scheduled Task Architecture

Structure the main execution loop as a series of numbered steps:

```bash
#!/bin/bash
# Step 0: Pre-flight checks
#   0.1: Disk hygiene (prevent disk full)
#   0.2: Process health check (restart dead services)
#   0.3: Budget check (stop if over daily limit)

# Step 1: Read inbox (human commands/overrides)
# Step 2: Read state (what happened last cycle)
# Step 3: Plan (what should happen this cycle)
# Step 4: Execute tasks (one at a time, verify each)
# Step 5: Post-execution
#   5.1: Run tests
#   5.2: Update state files
#   5.3: Apply pending fix scripts
#   5.4: Generate cycle report
# Step 6: Schedule next cycle
```

**Numbered steps matter** because they make debugging easy. When something fails, you immediately know it was "Step 4.2" not "somewhere in the execution phase."

### Inbox Pattern for Human Overrides

Let humans send commands to the agent without breaking the autonomous loop:

```
inbox/
  messages.json    # Timestamped messages from human
  commands.json    # Parsed actionable commands
```

**Standard commands:** `pause`, `resume`, `focus [area]`, `skip [task]`, `run [specific_task]`

The agent reads the inbox at cycle start and adjusts its plan accordingly. This preserves autonomy while giving humans a control channel.

### State Tracking

The agent must track what it's done, what it's doing, and what it plans to do:

```
STATUS.md          # Human-readable current state (updated every session)
state.json         # Machine-readable execution state
task_queue.json    # Pending tasks with priorities
results/           # Output from completed tasks
logs/              # Execution logs with timestamps
```

**STATUS.md** is the most important file. It should answer: "What happened in the last session? What's the current state? What's blocked?" in under 30 seconds of reading.

## Layer 5: Strategic Thinking

Autonomous agents need periodic "step back and think" moments — not just execution.

### Weekly Self-Assessment

Schedule a weekly deep review that evaluates:

1. **What worked this week?** — Successes, completed tasks, metrics improved
2. **What failed?** — Errors, missed opportunities, suboptimal decisions
3. **What did we learn?** — New patterns, insights, corrections
4. **What should change?** — Process improvements, priority shifts, new approaches
5. **What are we missing?** — Blind spots, unexplored opportunities, unasked questions

Store assessment results in a structured format so they accumulate into organizational knowledge over time.

### Gap Detection

Run a systematic gap analysis periodically:

1. **Compare state vs objectives:** For each active goal, does the current state actually support it?
2. **Check data flow:** For each pipeline step, is output actually consumed by the next step?
3. **Check feedback loops:** For each "learn → apply → measure" loop, do all three steps work?
4. **Check downstream consumers:** If you built module A that outputs data, does module B actually read it?

### Interconnected Thinking

Train the agent to think beyond individual tasks:

- **Before executing:** "What connects to this task? What's missing? What could be better?"
- **During execution:** "Does this pattern exist elsewhere? Am I noticing something the broader system should know about?"
- **After execution:** "What did I learn that should change how I operate?"

This prevents the agent from becoming a mechanical task executor and keeps it functioning as an intelligent system.

## Anti-Patterns to Avoid

These patterns will kill your autonomous agent over time:

| Anti-Pattern | What Happens | Instead Do |
|---|---|---|
| "Flagged for human to investigate" | Human never investigates, issue persists | Investigate automatically, propose fix |
| Same error escalated 5 cycles in a row | Alert fatigue, human ignores it | After 3 escalations, auto-attempt the most likely fix |
| "Recommended: run X manually" | Human doesn't run it, system degrades | Write fix to `pending_fixes/` for auto-execution |
| One-time fix without regression test | Fix gets reverted, problem returns | Every fix gets a test |
| Monitor without auto-action | Detects problems but never fixes them | Every monitor has an associated auto-recovery action |
| Catch-all exception handlers | Zombifies the process, hides real errors | Specific handlers that fix or re-raise |

## File Organization for Agent Projects

```
project/
  CLAUDE.md              # The operating system — agent instructions
  STATUS.md              # Current state — updated every session
  WORKFLOW.md            # How work flows through the system
  configs/               # All configuration (JSON, not hardcoded)
  data/
    heartbeats/          # Process health signals
    fixes/
      pending/           # Fix scripts awaiting execution
      executed/          # Successfully applied fixes
      failed/            # Fixes that didn't work
    logs/                # Execution logs
    learning/            # Accumulated knowledge and insights
    alerts/              # Urgent notifications
  scripts/
    cron_runner.sh       # Main execution loop
    disk_hygiene.sh      # Cleanup script
    install_cron.sh      # Cron/scheduler installation
  tests/
    test_regressions.py  # Tests that verify fixes stay in place
```

## Getting Started Checklist

For a new autonomous agent project:

- [ ] Write CLAUDE.md with all 5 sections (Before Work, Core Rules, Task Routing, Execution Mode, Security)
- [ ] Create STATUS.md with initial state
- [ ] Set up the execution loop (cron/launchd/scheduled task)
- [ ] Add heartbeat writing to all long-running processes
- [ ] Build a watchdog that checks heartbeats and restarts dead processes
- [ ] Add at least one self-healing pattern (auto-restart on crash)
- [ ] Set up the inbox pattern for human overrides
- [ ] Create a disk hygiene script (prevent disk full failures)
- [ ] Add a daily summary report
- [ ] Schedule a weekly self-assessment

Start with Layers 1-2 (foundation + process management). Add Layer 3 (self-healing) after you've seen what actually breaks. Add Layers 4-5 as the system matures.
