# Top 6 Claude Code Plugins Worth Installing

> Source: [r/AskVibecoders](https://www.reddit.com/r/AskVibecoders/comments/1schn7x/top_6_claude_code_plugins_worth_installing/) — a curated list from testing dozens of plugins in the Claude Code ecosystem (9,600+ repositories, 2,300+ skills as of April 2026).

Each plugin installs in under a minute. They cover architecture, parallel execution, code review, security monitoring, and frontend design.

---

## 1. feature-dev (89,000+ installs) — Planning & Architecture

Instead of jumping straight to code, `feature-dev` runs Claude through a **7-phase workflow**:

| Phase | Purpose |
|-------|---------|
| 1. Discovery | What do you actually need? |
| 2. Codebase Exploration | What's already in the project? |
| 3. Clarifying Questions | What's unclear? |
| 4. Architecture Design | How to build this correctly? |
| 5. Implementation | Now we write code |
| 6. Quality Review | Did we break anything? |
| 7. Summary | What changed and why |

### Under the Hood — 3 Specialized Agents

- **code-explorer** — traces execution paths and maps your architecture
- **code-architect** — proposes multiple approaches with trade-offs
- **code-reviewer** — catches bugs and convention violations with confidence scoring

### Install & Usage

```bash
/plugin install feature-dev@claude-plugins-official
/feature-dev Add user authentication with OAuth
```

### Why It Matters

Without this plugin, Claude guesses your architecture from one sentence. With it, Claude asks the right questions first — discovery before implementation.

---

## 2. code-review (5 Parallel Agents) — Code Review & Quality

Run `/code-review` on a pull request branch and Claude spins up **5 independent agents simultaneously**, each reviewing changes from its own angle:

| Agent | Focus |
|-------|-------|
| 1 | **CLAUDE.md compliance** — are you following your own project rules? |
| 2 | **Bug hunting** — logic errors, edge cases, race conditions |
| 3 | **Git history context** — does this conflict with recent commits? |
| 4 | **Previous PR comments** — were past review notes addressed? |
| 5 | **Comment verification** — do code comments match what the code actually does? |

### Quality Metrics

- Each finding gets a **confidence score (0–100)**; only issues above **80** are shown
- On large PRs (1,000+ lines): **84%** get findings, averaging **7.5 real issues**
- **< 1%** of findings are marked incorrect by engineers
- Used internally at Anthropic on almost every PR

### Install & Usage

```bash
/plugin install code-review@claude-plugins-official
/code-review
/code-review --comment   # posts findings directly to your GitHub PR
```

---

## 3. agent-peer-review — Claude vs Codex Cross-Verification

When Claude is about to present an implementation plan, architectural decision, or code review, this plugin **automatically sends it to OpenAI Codex for verification**. Two models compare findings and classify them:

| Classification | Meaning |
|----------------|---------|
| **Agreement** | Both found the same thing — high confidence |
| **Disagreement** | One found what the other missed — worth investigating |
| **Complement** | Found different things — both correct |

### Escalation Protocol

If the models can't agree, a **2-round discussion protocol** runs. If still no agreement, the plugin escalates to **Perplexity or web search** for external evidence.

### Install & Usage

```bash
/plugin marketplace add jcputney/agent-peer-review
/plugin install codex-peer-review
/codex-peer-review
/codex-peer-review "Should we use microservices or monolith for this project?"
```

---

## 4. /batch (5–30 Parallel Agents) — Parallel Execution

Describe a change across the entire codebase in one sentence. Claude breaks it into **5–30 independent units**, spins up one agent per unit — each in its own **isolated git worktree** — and they all work in parallel.

### 3-Phase Execution

1. **Discovery** — Claude scans the entire codebase, finds every file and pattern the change touches
2. **Execution** — One agent per unit, all working simultaneously. Each gets its own branch and working directory (zero merge conflicts). After implementation, each agent runs tests, runs `/simplify`, commits, pushes, and opens a PR
3. **Tracking** — Status table updates in real time. Final result example: "22/24 units landed as PRs"

### Example Commands

```bash
/batch migrate from React to Vue
/batch replace all uses of lodash with native equivalents
/batch add type annotations to all untyped functions
```

### Key Takeaway

22 pull requests from one command. Each unit is isolated, tested, and delivered as its own PR.

---

## 5. ralph-loop — Autonomous Iteration

An **autonomous loop**: you give Claude a task with clear success criteria, and it works repeatedly — fixing its own mistakes, running tests, iterating — until it either succeeds or hits the iteration limit.

### How the Loop Works

1. Claude works on the task
2. Tries to end the session
3. A **stop hook blocks the exit**
4. The same prompt is fed back with Claude's previous work visible
5. Cycle repeats until `COMPLETE` or max iterations

### Install & Usage

```bash
/plugin install ralph-loop@claude-plugins-official

/ralph-loop "Build a REST API for todos. When complete:
- All CRUD endpoints working
- Input validation in place
- Tests passing (coverage > 80%)
- README with API docs
Output: <promise>COMPLETE</promise>" --max-iterations 20
```

### Overnight Execution

```bash
#!/bin/bash
cd /path/to/project1
claude -p "/ralph-loop 'Task 1...' --max-iterations 50"

cd /path/to/project2
claude -p "/ralph-loop 'Task 2...' --max-iterations 50"
```

### Real Results

- Y Combinator hackathon teams shipped **6+ repos overnight**
- Geoffrey Huntley ran a 3-month loop that built a **full programming language**

---

## 6. security-guidance — Security Monitoring Hook

A **PreToolUse hook** that monitors **9 security patterns** every time Claude touches a file:

1. Command injection
2. Cross-site scripting (XSS) vulnerabilities
3. Use of `eval()`
4. Unsafe HTML rendering
5. Pickle deserialization
6. `os.system()` calls
7. SQL injection patterns
8. Path traversal
9. Insecure deserialization

### Why a Hook, Not Instructions

- Instructions in `CLAUDE.md` are ignored roughly **20% of the time**
- Hooks fire **deterministically, 100% of the time**, before Claude can edit a file
- It's always on once installed — no opt-out, no skip

### Install

```bash
/plugin install security-guidance@claude-plugins-official
```

---

## Quick Selection Guide

| Pain Point | Plugin |
|------------|--------|
| Spending too long planning features? | **feature-dev** |
| PRs merging without proper review? | **code-review** |
| Large migrations eating weeks? | **/batch** |
| Want to ship while you sleep? | **ralph-loop** |
| Security gaps in AI-generated code? | **security-guidance** |
| Need cross-model verification? | **agent-peer-review** |

---

## Architectural Patterns to Note

### Agent Orchestration
Plugins like `feature-dev` and `code-review` demonstrate the **Agent specialization pattern** — instead of one general agent, multiple focused agents each handle a specific concern (exploration, architecture, review). This mirrors the Agent-to-Skill pattern documented in this repository.

### Parallel Isolation
`/batch` and `code-review` use **git worktree isolation** to run agents in parallel without conflicts. Each agent gets its own branch and working directory — a pattern that scales from 5 to 30 concurrent agents.

### Hook-Based Guarantees
`security-guidance` shows that **deterministic hooks** are more reliable than prompt-based instructions for safety-critical checks. This aligns with the hooks architecture documented in the `best-practice/` guides — hooks fire on events, not on model interpretation.

### Autonomous Loops
`ralph-loop` uses a **stop-hook pattern** to create iterative execution — the hook intercepts session exit, feeds context back, and restarts. This is a general pattern for any autonomous workflow with measurable success criteria.
