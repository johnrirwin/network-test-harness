# AGENTS.md

This repository uses the following workflow orchestration for human and AI contributors.

## Operating Defaults
- Prefer simple, minimal, root-cause fixes.
- Touch only the files required for the task.
- Keep the repo in a verifiable state before calling work complete.
- If higher-level system or tool policies conflict with this file, follow those higher-level policies first.

## 1. Plan First
Use plan mode, or the closest available equivalent, for any non-trivial task.

### Enter planning for:
- Tasks with 3 or more meaningful steps
- Architectural or design decisions
- Debugging with multiple plausible root causes
- Verification work that needs explicit checklists

### Planning rules
- Write a detailed spec up front to reduce ambiguity.
- Record the plan in `/Users/john/Development/network-test-harness/tasks/todo.md` using checkboxes.
- If true plan mode is unavailable, emulate it with a written checklist before implementation.
- If new information invalidates the plan, stop, update the plan, and only then continue.

## 2. Task Management
`/Users/john/Development/network-test-harness/tasks/todo.md` is the working scratchpad for the current task.

### Requirements
- Keep `tasks/todo.md` out of version control.
- Confirm the plan before major implementation begins when the workflow allows for it.
- Do not block on explicit confirmation for trivial, low-risk work such as small doc edits or isolated single-file changes unless the user asks for a checkpoint.
- Mark checklist items complete as work progresses.
- Add a short review section at the end of the task covering:
  - What changed
  - How it was verified
  - Follow-up risks or open questions

### File roles
- `tasks/todo.md`: current-task plan, progress, and review notes; local only
- `tasks/lessons.md`: durable lessons learned after user corrections; tracked in git

## 3. Subagent Strategy
Use subagents proactively, but deliberately, when platform policy and tool availability allow it.

### Good subagent work
- Research
- Codebase exploration
- Parallel analysis
- Focused one-task investigations

### Rules
- Use one task per subagent.
- Prefer subagents for bounded, parallelizable work where the expected leverage outweighs coordination cost.
- Avoid spawning subagents for trivial edits or work the main thread can complete faster directly.
- Keep the main thread focused on integration and decision-making.
- For complex problems, increase compute with parallel subagents where possible.
- Do not keep pushing on a failing path; gather findings, then re-plan.

## 4. Verification Before Done
Never mark work complete without evidence.

### Always do the best available verification
- Run relevant automated tests.
- Compare baseline vs. changed behavior when relevant.
- Check logs, error output, or generated artifacts.
- Demonstrate correctness in the final summary.
- Ask: "Would a staff engineer approve this?"

If the repo lacks tests, say so explicitly and use the strongest available alternative validation.

## 5. Demand Elegance (Balanced)
For non-trivial changes, pause and review the shape of the solution.

### Ask
- Is there a more elegant way?
- Does this fix the root cause instead of the symptom?
- Knowing what is known now, is this still the correct solution?

Avoid over-engineering simple fixes.

## 6. Autonomous Bug Fixing
When given a bug report:
- Reproduce the issue if possible.
- Use logs, traces, failing tests, and CI evidence.
- Fix the issue without unnecessary back-and-forth.
- Proactively address closely related failing tests or obvious regressions.

Escalate to the user only when blocked by missing access, missing requirements, or irreducible ambiguity.

## 7. Self-Improvement Loop
After any user correction:
- Update `/Users/john/Development/network-test-harness/tasks/lessons.md`.
- Capture the mistake, the corrected rule, and how to avoid repeating it.
- Review relevant lessons at the start of future sessions.

## 8. Communication Style
- Share concise high-level progress updates at meaningful checkpoints.
- Call out re-plans explicitly when the situation changes.
- Summarize what changed, why, and how it was verified.

## 9. Repo Startup Checklist
At the start of any non-trivial task:
1. Review this file.
2. Review relevant entries in `tasks/lessons.md`.
3. Write or refresh `tasks/todo.md`.
4. Inspect the current code and baseline behavior.
5. Implement the smallest correct change.
6. Verify before declaring completion.
