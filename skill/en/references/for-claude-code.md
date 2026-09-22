# Scenario A: task prompts destined for Claude Code

The user describes work in Claude desktop; you produce a prompt they paste into Claude Code (including the VS Code extension's graphical window).

## Contents

- First priority: give it a check it can run
- How hard the check gates the stop
- When the check does not exist yet
- Say it all at once
- How specific to be
- Values only the user can supply
- Explore, plan, implement, commit
- Larger features: have it interview the user into a SPEC
- Context is the binding constraint
- This prompt versus project configuration
- Where project configuration goes: seven mechanisms
- Turning manual checks into verification loops
- Four kinds of loops
- Five common failure patterns

---

## First priority: give it a check it can run

This is the first rule in the official best practices, and it is the largest difference between this scenario and Scenario B.

Adapted from the official wording — the second person there addresses the reader, which here means your user rather than you:

> Claude stops when the work looks done. Without a check it can run, "looks done" is the only signal available, and **the user becomes the verification loop** — every mistake waits for them to notice it. Give Claude something that produces a pass or fail, and the loop closes on its own.

The check is anything that returns a signal the model can read: a test suite, a build exit code, a linter, a script that diffs output against a fixture, or a browser screenshot compared against a design.

**Contrasts to use when writing the prompt:**

| Weak | Strong |
|---|---|
| `implement a function that validates email addresses` | `write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing` |
| `make the dashboard look better` | `[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them` |
| `the build is failing` | `the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error` |

> **Distinguish this from SKILL.md 3.4.** That section is about removing subjective self-review instructions. This is about supplying **external deterministic checks** — running tests, reading exit codes, comparing screenshots. Remove the former; you must supply the latter.

Also require **evidence rather than assertion**: the test output, the command run and what it returned, a screenshot of the result. Reviewing evidence is faster than re-running the verification yourself, and it works for sessions nobody was watching.

## How hard the check gates the stop

Once the check exists, there is a second decision the prompt alone cannot make. Official guidance gives four levels, trading setup effort for how little attention the run needs:

| Level | What it is | When it earns the setup |
|---|---|---|
| **In one prompt** | Ask it to run the check and iterate in the same message | Any task, today. This is what the prompt you are writing does |
| **Across a session** | Set the check as a `/goal` condition — a separate evaluator re-checks after every turn | The user wants it to keep going until the criterion holds |
| **A deterministic gate** | A `Stop` hook runs the check as a script and blocks the turn from ending until it passes (Claude Code overrides after 8 consecutive blocks) | The check must not be skippable |
| **A second opinion** | A verification subagent — or a dynamic workflow that checks its own findings — so the agent doing the work is not the one grading it. For a plain correctness check, Claude Code's bundled `/code-review` skill reviews the current diff in a fresh subagent | Correctness matters more than speed |

**Mention the level in the delivery note when it matters.** A prompt is level one by definition; if the user described an unattended run, say that `/goal` or a `Stop` hook is what actually lets it finish correctly without them.

## When the check does not exist yet

The rule above assumes a runnable check exists. **The most common Claude Code request is exactly the one where no such check exists** — legacy code, low coverage, a past incident, nobody willing to touch it. Do not skip verification and start editing. Make **building the check the first phase**:

1. **Write characterization tests that pin current behavior.** Regardless of whether that behavior is correct, assert it as it stands — including behavior you believe is a bug. The goal is a net that detects whether behavior changed, not a judgment about what is right
2. **Pin suspected bugs as they are, and mark them separately.** Where you suspect a bug, still assert the actual current behavior, and add a comment flagging it. **This must be stated explicitly in the prompt** — a well-intentioned reader told to "write tests that pin current behavior" will very naturally fix the bugs they find along the way, which destroys the net
3. **Freeze the net once it exists.** The refactor phase must not modify these tests. A changed test means changed behavior, and refactoring is by definition behavior-preserving. When a test genuinely must change, that has to be raised separately with a reason
4. **Only then refactor**, running the net at every step

The corresponding prompt structure is phased with checkpoints between phases, not a single line saying "refactor this module." Behavior changes — fixing the bugs, altering logic — belong in a **separate later round**. If the two are mixed together, a red test leaves you unable to tell whether the refactor broke something or the behavior simply changed.

## Say it all at once

Official guidance is explicit: **specifying the task, intent, and relevant constraints upfront in the first turn maximizes autonomy and intelligence while minimizing extra token usage after user turns.** Ambiguous or underspecified prompts delivered progressively over multiple turns tend to reduce token efficiency and sometimes performance.

This matters more here than anywhere else, because the user pastes once and walks away.

## How specific to be

| Strategy | Weak | Strong |
|---|---|---|
| **Scope the task**: which file, which scenario, testing preferences | `add tests for foo.py` | `write a test for foo.py covering the edge case where the user is logged out. avoid mocks.` |
| **Point to sources**: where the answer lives | `why does ExecutionFactory have such a weird api?` | `look through ExecutionFactory's git history and summarize how its api came to be` |
| **Reference existing patterns**: point at a model in the codebase | `add a calendar widget` | `look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget...` |
| **Describe the symptom**: symptom, likely location, and what "fixed" looks like | `fix the login bug` | `users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it` |

Vague prompts have their place — exploration, when you can afford to course-correct. `what would you improve in this file?` surfaces things you would not have thought to ask. That is exploration, not execution.

**Ways to supply rich input** (mention these to the user): `@` to reference files, paste images directly, give documentation URLs, pipe data in with `cat error.log | claude`, or let Claude fetch what it needs through bash, MCP tools, and file reads.

## Values only the user can supply

Scenario A always has values that are **neither inferable nor askable in advance**: the repo path, the test command, what the two production incidents actually were, internal conventions. These are often the highest-value inputs in the whole prompt.

Leave an **explicit placeholder** in the prompt body, with a fallback for when it is left blank:

```
What the two production incidents looked like: 【fill in if you remember】
If the line above is blank, first search git log and the issue tracker for related fixes,
list the incident scenarios you infer, and confirm them with me before continuing.
```

Two requirements: the placeholder must **look like a placeholder** (use a bracket convention the user's eye catches), and **a fallback is mandatory** — a blank with no fallback gets filled in with invented content, or silently ignored. Filled in is better; blank still runs.

## Explore, plan, implement, commit

When scope is unclear or the change spans files, the prompt should walk through four phases rather than starting to write:

1. **Explore** (plan mode): read without changing, understand the current state
2. **Plan**: produce a detailed implementation plan
3. **Implement**: write it, verifying against the plan
4. **Commit**: descriptive message, open a PR

**When to skip:** when scope is clear and the change is small — a typo, a log line, a rename. The official test is useful: **if you could describe the diff in one sentence, skip the plan.**

Planning earns its cost when you are uncertain about the approach, when the change modifies multiple files, or when you are unfamiliar with the code.

**Does "verify against the plan" count as the self-verification SKILL.md 3.4 removes?** It depends on whether the plan was externally confirmed:

- The model wrote the plan, nobody reviewed it, and it now checks itself against it → **that is self-verification**, and of limited value
- The plan was **reviewed and confirmed by the user** (or written to a SPEC.md and adopted as an agreement) → **that is an external basis**, and checking against it works

So the four-phase flow needs a confirmation step after planning. Write that checkpoint explicitly: have it produce the plan and **stop for confirmation** before implementing. When the user will not be present to confirm, drop "verify against the plan" and rely on tests and other genuinely external checks.

## Larger features: have it interview the user into a SPEC

When the request is large and the user has not fully thought it through, the prompt you hand them can be this one:

```text
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask obvious questions, dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Then execute that spec in a **fresh session** — clean context, focused entirely on implementation.

Two official points worth passing on: **time spent making the spec precise pays off more than time spent watching the implementation**, and the most useful specs are **self-contained** — they name the files and interfaces involved, state what is out of scope, and end with an end-to-end verification step that proves the feature works.

---

## Context is the binding constraint

Most of the Claude Code best practices derive from one constraint: **the context window fills fast, and performance degrades as it fills.** A single debugging session or codebase exploration can consume tens of thousands of tokens.

What the prompt can do about it:

- **Push exploration outside the main context**: `Use subagents to investigate how our authentication system handles token refresh, and whether we have any existing OAuth utilities I should reuse.` The subagent reads many files in its own context and reports back a summary

> **The unified subagent criterion** (this is scattered across sources; stated fully here): **encourage them for wide read-only exploration, forbid them for re-reviewing the model's own work in its own context.** A *separate, fresh-context* verifier is the exception, and officially outperforms self-critique — that is what the "second opinion" level above and the adversarial-review note at the end of this file both refer to.
>
> So: The first is their greatest value — keeping the cost of reading dozens of files out of the main context. The second is something Opus 5 already does, so delegating it merely doubles the cost. For everything in between, snippet 4.10 is the test: if you can finish it yourself in a handful of tool calls, do not delegate.

- **Do not write open-ended investigations**: `investigate X` without a boundary reads hundreds of files and fills the context
- Remind the user: `/clear` between unrelated tasks; after two failed corrections on the same issue, `/clear` and rewrite the prompt with what was learned

## This prompt versus project configuration

The test is simple: **information specific to this task goes in the prompt; conventions that hold every time get sedimented into project configuration.**

If you find yourself writing "we use pnpm, not npm" or "run lint before committing" — things that have to be said **every single time** — recommend the user move them into CLAUDE.md or a hook rather than pasting them again.

## Where project configuration goes: seven mechanisms

| Mechanism | Loaded | Context cost | Best for |
|---|---|---|---|
| **CLAUDE.md (root)** | Session start, stays throughout, re-read after compaction | **High** (always loaded) | Build commands, directory layout, monorepo structure, team conventions. **Keep under 200 lines**, with an owner who reviews changes |
| **CLAUDE.md (subdirectory)** | On demand when that directory is touched | Low | Directory-specific conventions |
| **Rules** | Session start (unscoped) or on file access (scoped) | Medium | Specific constraints. **Add `paths:` frontmatter** to scope them and cut the cost |
| **Skills** | Name and description at start, body on invocation | Low | Procedural workflows: deploy checklists, release processes, review processes |
| **Subagents** | Name and description at start, body when called; only the final message returns | Low (isolated context) | Side tasks where you only want the conclusion: log analysis, dependency audits, deep search |
| **Hooks** | Fire on lifecycle events, bypass compaction | Low (outside main context) | **Deterministic** automation: linters, notifications, blocking specific commands |
| **Output styles** | Session start, never compacted | **High** | Only when Claude's role needs to change substantially |

**Anti-patterns:**

- ✗ "Every time X, always do Y" in CLAUDE.md → use a hook. **Advisory instructions are always weaker than deterministic execution**
- ✗ "Never do this" constraints → use a `PreToolUse` hook or permission configuration
- ✗ A 30-line procedure in CLAUDE.md → move it to a skill
- ✗ An API-specific rule with no path scoping → add `paths:` frontmatter
- ✗ Personal preferences in a shared project file → use the user-level local equivalent

**What belongs in CLAUDE.md:** bash commands Claude cannot guess, code style that differs from defaults, testing instructions, repository etiquette (branch naming, PR conventions), project-specific architectural decisions, environment quirks, non-obvious gotchas.

**What does not:** anything readable from the code, standard language conventions, detailed API documentation (link it), frequently changing information, file-by-file descriptions, platitudes like "write clean code."

**Self-test:** for each line ask, *would removing this cause Claude to make mistakes?* If not, cut it.
**Diagnostic signal:** when a rule keeps being ignored despite repetition, the file is usually **too long and the rule is buried**, not insufficiently emphatic.
**Tool:** `/doctor` helps rightsize skills and CLAUDE.md.

**How to write a skill** (three-tier progressive disclosure):

| Tier | Content | Loaded |
|---|---|---|
| Metadata | YAML frontmatter, only `name` and `description` | Preloaded into the system prompt at startup |
| Primary instructions | The `SKILL.md` body | When Claude judges the skill relevant |
| Bundled files | Files referenced from SKILL.md | Only when that specific scenario arises |

`name` and `description` are what drive activation — if it does not trigger, fix those two lines first. Split the body when it gets unwieldy. Bundle executable code where it helps: sorting a list via token generation is far more expensive than simply running a sorting algorithm. **The Fable 5 page states that skills developed for prior models are often too prescriptive for it and can degrade output quality** — prune periodically when targeting the Fable line. (The Fable 5.1 page does not restate this; it says existing Fable 5 prompts should perform well on Fable 5.1 without changes, and the general prompting page says to treat a model-specific finding as measured on that model.)

## Turning manual checks into verification loops

The test: **anything the user keeps having to enforce by hand is a candidate for turning into a loop.** Especially deterministic rules a generic linter will not catch — the official example is "reject any migration that drops a column without a backfill step."

| Mode | Trigger | Best for |
|---|---|---|
| **Standalone** | Invoked deliberately | Cross-cutting checks that do not apply universally: security scans, accessibility audits, license verification |
| **Embedded** | Fires automatically inside the producing skill | Checks that belong to one specific task |
| **Chained** | One skill calls another on completion | End-to-end handoff; also the only way to add verification to a **skill you cannot modify** |
| **On every PR** | CI | Turns a personal habit into team infrastructure, removing dependence on author diligence |

**Trap:** embedding only works on editable skills. Built-in or plugin-managed skills update automatically and would overwrite your change — those must use chaining.

## Four kinds of loops

| Type | Command | Best for |
|---|---|---|
| **Turn-based** | Manual prompting | Short tasks |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | Tasks with a **verifiable exit criterion** where one turn is not enough |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | Recurring work, interfacing with external systems |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | Well-defined recurring streams: bug reports, issue triage, migrations |

Cost control: **use scripts for deterministic work** (running a script costs less than reasoning through it token by token); match intervals to how fast the underlying state actually changes; pilot before running at scale. The official conclusion: **model and effort selection are among the biggest levers on loop cost.**

## Five common failure patterns

Worth passing on to the user alongside the prompt:

| Pattern | Symptom | Fix |
|---|---|---|
| **The kitchen sink session** | One task, then something unrelated, then back again | `/clear` between unrelated tasks |
| **Correcting over and over** | Corrected once, still wrong, corrected again | **After two failed corrections, `/clear`** and write a better initial prompt incorporating what you learned |
| **The over-specified CLAUDE.md** | Too long; important rules get lost in the noise | Prune ruthlessly |
| **The trust-then-verify gap** | Plausible-looking implementation that misses edge cases | Always provide verification. **If you cannot verify it, do not ship it** |
| **The infinite exploration** | "Investigate this" without scope; hundreds of files read | Narrow the scope, or isolate it in a subagent |

**One important caveat on adversarial review.** Having a subagent review the diff in a fresh context is effective, but the official guidance also warns: **a reviewer prompted to find gaps will usually report some, even when the work is sound, because that is what it was asked to do.** Chasing every finding leads to over-engineering. So the review prompt must say: **flag only gaps that affect correctness or the stated requirements, and treat the rest as optional.** This is a filtering instruction, so it belongs only where **you** are the filter. If the findings feed a downstream ranking or triage step instead, use snippet 4.16 and let the reviewer report everything — `references/models.md` warns that Opus 5 applies a stated severity bar literally and silently drops whatever falls below it.
