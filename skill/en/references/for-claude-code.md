# Scenario A: task prompts destined for Claude Code

The user describes work in Claude desktop; you produce a prompt they paste into Claude Code (including the VS Code extension's graphical window).

The main basis here is four pages of the Claude Code docs: Best practices, Create custom subagents, Model configuration, and Manage costs effectively. Anything that comes from elsewhere is cited inline.

## Contents

- Tell it only what it can't see
- First priority: give it a check it can run
- How hard the check gates the stop
- Say it all at once
- How specific to be
- Requirements not thought through: have it interview the user into a SPEC
- Ask for a plan before large changes
- Subagents: model, effort, concurrency, and errors
- This prompt versus project configuration
- Deterministic and non-deterministic instructions
- Where project configuration goes: the seven methods
- Turning manual checks into verification loops
- Four kinds of loops
- Five questions before an unattended run
- Old habits you can drop
- When a session doesn't go as expected

---

## Tell it only what it can't see

When Claude starts a task, it reads the code first. It finds the file layout, the conventions, the test framework, and how existing components are built on its own, usually faster than you could explain them. What it can't find is information that was never in the repo: a decision the team made in a meeting, a deploy process that lives in one person's head, the environment variable the integration tests need.

The official test is two questions to ask before you write anything down: could it find this by reading the code? Would it actually go and look? If both answers are yes, leave it out.

This rule covers the prompt, CLAUDE.md, and skills alike.

## First priority: give it a check it can run

The Claude Code best practices, reworded as if said to your user: Claude stops when the work looks done. If it has nothing to run that tells it whether the work is really done, that is all it has to go on, and in practice the user becomes the test suite, with every mistake waiting until they happen to notice it. Give it something that comes back pass or fail, and it can close the loop itself: make the change, run the check, read the output, fix it, run the check again.

The check can be anything whose result Claude can read: a test file, a build's exit code, a linter, a script that diffs output against a fixture, a browser screenshot next to the mockup. If the repo already has one of these, the single most useful line in CLAUDE.md is the command that runs it.

Also have it **show evidence**: the command it ran and its output, or a screenshot. In the official words, `"All tests pass" on its own tells you very little`; reading the output is faster than rerunning the check yourself, and for a session the user wasn't watching, it is the only evidence there is.

After Claude says it is done, `/verify` can also be run in the session: it has Claude build and run the app and look at the result, which catches the kind of problem that tests and type checks miss.

## How hard the check gates the stop

The docs give four levels, from Claude deciding for itself to Claude Code enforcing it:

| Level | What it is | What the docs say |
|---|---|---|
| **In the prompt** | `run the auth tests after, and keep going until they pass` | Claude decides for itself when it is satisfied. Works for any task, with no setup |
| **As a goal** | `/goal all tests in tests/auth pass and lint is clean`: after every turn, a separate model checks the condition against the conversation | Rarely needed when you are watching the session yourself. It matters for long background runs: Claude sometimes ends a turn with a progress summary while items are still open, and the goal sends it back |
| **A `Stop` hook** | Runs your script when Claude tries to end its turn, and sends Claude back to work if the script exits with code 2 | Enforced by Claude Code. If the hook blocks eight times in a row with no tool call in between, Claude Code lets the next stop through; `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` changes this cap |
| **Review by someone who didn't write the code** | In the terminal, `/code-review` reviews the diff in a fresh subagent that never saw your conversation | It isn't swayed by the reasoning that produced the code |

**This level is worth a line in the delivery note.** When the user describes a long run that nobody will be watching, tell them what the docs say: a run like that is enforced with `/goal` or a `Stop` hook.

To have the review check the work against the plan rather than hunt for bugs in general, write the review prompt yourself. The official example:

```text
use a subagent to review the rate limiter diff against SPEC.md, check every requirement is implemented and the listed edge cases have tests, report gaps that affect correctness and skip style
```

The docs specifically warn against dropping `skip style`: `A reviewer that's asked to find problems will always find something.` If you fix every finding, you end up with extra abstraction layers and tests for cases that can't happen. In this official example the subagent reviews and reports straight back to you. Snippet 4.16 comes from the Sonnet 5 page's "Code review harnesses" section, which is about a harness with a separate filtering step afterwards, and has the reviewer report everything; the Opus 5 page warns that when a review prompt sets a severity bar, the model `might follow that instruction literally and report less` (measured on Opus 5; the Opus 5.5 pages neither restate nor retract it; see `models.md`).

---

## Say it all at once

The Claude Code best practices: name the file, describe the symptom, and say up front what "fixed" looks like, and `you usually save yourself a round of corrections later`. The same goes for a preference between two reasonable approaches — Claude can't read your preference from the code, so if you care, say so up front.

Two predecessor pages put it more directly, and both of their successor pages say the predecessor's patterns remain a reasonable starting point. The Opus 5 page says the model `performs best when given the complete task specification up front and left to run`. The Sonnet 5 page says `Providing well-specified, clear, and accurate task descriptions upfront can help maximize autonomy and intelligence while minimizing extra token usage after user turns`, whereas `ambiguous or underspecified prompts conveyed progressively over multiple user turns tend to relatively reduce token efficiency and sometimes performance`.

Opus 5.5 has an official article of its own that says the same (Anthropic, 2026-09-22, "Getting the most out of Opus 5.5 in Claude and Claude Code"): give the whole task in one message and name the finish line, for example "the tests pass" or "every endpoint is migrated". The article also gives a rule to put in CLAUDE.md, for when a long Opus 5.5 task keeps stopping to report and ask whether to continue, and says to edit it to fit your project:

```text
When a step doesn't need my input, keep going. Put status notes in the same message as your next action.
Stop and ask only when you can't continue without me, or before anything destructive: deleting data, force-pushing, or changing anything outside this repository.
```

The article's notes on it: if a run stops to ask whether to continue, reply continue; if that happens often, the rule helps. The last line of the rule keeps your own check before anything risky or hard to undo, and permission prompts for destructive commands should stay on too. For pair programming you may want the opposite, a one-line plan before it starts and a short recap at the end, also written in CLAUDE.md.

The same article gives two more tips for long Opus 5.5 runs. First, ask it to keep its task list in a file and update it as it goes; the article's wording is `Keep a checklist in TASKS.md. Tick each item when it's done, and add anything new you find.` The reason given is that a long run fills the context window, Claude Code then summarizes older turns, and a list in a file survives that. Second, for an audit, a migration or a review across a large codebase, ask it to split the work across subagents and check each result; the article's example:

```text
Audit every service in services/ for the retry bug in the linked issue.
Give each service to its own subagent. When a subagent reports back, check its evidence before you accept it.
Finish with one table: service, affected yes or no, and the evidence.
```

Be just as clear when you tell Fable 5.2 to keep going or approve a step: the Fable 5.2 page asks you to say what you want it to do rather than send a bare continue. Its example is replying `Yes, run the migration on the staging database` where you might have replied "go" or sent a thumbs-up.

---

## How specific to be

The official contrast (Claude Code best practices):

| Instead of | Try |
|---|---|
| `fix the login bug` | `users report login fails after the session times out. look at token refresh in src/auth/, write a failing test that reproduces it, then fix it` |
| `add a calendar widget` | `follow the pattern in HotDogWidget.php and build a calendar widget that lets you pick a month and page between years. no new libraries` |

**If you already have what it needs, hand it over instead of describing it**: `@` plus a path brings a file's contents into the message; a screenshot can be pasted straight into the prompt; command output can be piped in from the shell, for example `cat error.log | claude -p "what's going on here"`. The support site's Claude Code guide adds a note on usage: the `@` prefix injects the entire file plus its CLAUDE.md tree into context, whereas a bare path lets Claude open the file and read selectively; trim logs and stack traces to the relevant 20 or 30 lines before pasting.

**Three official notes on wording**:

- Phrases like `think step by step` and `be thorough` are ordinary prompt text, not keywords. Effort is the main control for how hard it thinks; such wording can only nudge it within the current level
- One phrase that still helps is `be ambitious`: Claude tends to read a request conservatively and build the smallest thing that satisfies it. If you want the fuller version, say so. That tells it about **scope**, which only you know. The official example:

```text
build the settings page from the mockup. be ambitious: empty and error states, the save flow wired up, keyboard navigation, not just the layout
```

- When you are exploring rather than fixing something specific, a vague prompt like `what would you improve in this file?` is completely fine; the cost is a few more corrections along the way

## Requirements not thought through: have it interview the user into a SPEC

When the user hasn't fully thought it through either, there is no need to work it all out before writing the prompt. The official example:

```text
I want to add rate limiting to the public API but haven't thought it through yet. Interview me until you could write the spec, then write it to SPEC.md.
```

Claude asks multiple-choice questions about the parts that aren't pinned down, and keeps asking until the answers add up to a spec. Once `SPEC.md` exists, **start a fresh session** to build from it: the context is clean, and there is a written definition of done. It is also the file to point the reviewer at later.

## Ask for a plan before large changes

The support site's Claude Code guide: for any change that touches more than two or three files, switch to Plan Mode, or just ask `Before changing anything, list the files you'll touch and what you'll do in each.` Read the list, correct it in plain language, then let it execute. The reason is cost: a plan costs a few hundred tokens, while a wrong 400-line diff that you revert and regenerate costs thousands, twice, plus the turns spent explaining what went wrong. The Claude Code costs page says the same: use plan mode first for complex tasks, because rework after a wrong start is expensive.

The same guide's model pairing: write the plan with Opus and execute it with Sonnet, built in as `/model opusplan`. Anthropic's article "What a task costs on Opus 5.5" (2026-09-25, written before Sonnet 5.5 shipped) adds a caution: `opusplan` puts the code edits on Sonnet, the opposite of the advice to keep code edits on the main model, so `Measure it on your own tasks before you make it a default`. The Claude Code prompt caching page also states that with `opusplan` each plan-mode toggle is a model switch and starts a fresh cache.

---

## Subagents: model, effort, concurrency, and errors

For which model to choose, see SKILL.md section 2. This section covers the mechanics the Claude Code docs state; follow them when you write the prompt.

**The model is resolved in this order** (subagents page, "Choose a model"):

1. The `model` parameter passed on this invocation
2. The `model` field in the subagent's definition, where `inherit` means the main conversation's model
3. The `CLAUDE_CODE_SUBAGENT_MODEL` environment variable
4. The main conversation's model

Key points:

- The `model` on each Agent tool call accepts only aliases: `sonnet`, `opus`, `haiku`, `fable`. It can't take a full model ID or `inherit`. The `model` field in a definition file can be an alias, a full ID, or `inherit`
- A family alias (such as `opus`) passed on the invocation or written in the definition's `model` field resolves to the main conversation's exact model when the main conversation's model belongs to that family. An alias in `CLAUDE_CODE_SUBAGENT_MODEL` always resolves to the version the alias points to
- Which version an alias points to depends on the provider (the table on the Model configuration page): on the Anthropic API, `opus`, `sonnet`, and `haiku` are Opus 5.5, Sonnet 5.5, and Haiku 5.5; on Claude Platform on AWS they are Opus 5.5, Sonnet 4.6, and Haiku 4.5; on Amazon Bedrock and Google Cloud's Agent Platform they are Opus 5.5, Sonnet 4.5, and Haiku 4.5; on Microsoft Foundry they are Opus 4.6, Sonnet 4.5, and Haiku 4.5. Unless `ANTHROPIC_DEFAULT_FABLE_MODEL` is set, the `fable` alias resolves to Fable 5 in Claude apps gateway sessions and to Fable 5.2 everywhere else
- A `model` passed on the invocation still applies when that subagent is resumed or sent a follow-up message
- This order holds from v2.1.251; in earlier versions the environment variable came first. In three cases the `model` passed on the invocation doesn't count: with `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` on, Claude can't pass a model when it starts a subagent, and the `model` field in definitions is ignored as well; a value that the organization's `availableModels` allowlist blocks is replaced with another model; and when an installed mod sets a model in its `agent.spawn` hook, the mod's model is used
- Switching the main conversation's model with `/model` also affects subagents started afterward that inherit the main conversation's model
- To see which model a subagent actually runs on, use `/tasks`: it shows the model on the subagent's row, and the effort level if one is set
- **Forks don't follow this order**. A fork inherits the whole conversation, runs on the main conversation's model, and shares the prompt cache with the main conversation; the docs say it costs less than a fresh subagent on tasks that need the same context. Fork mode is on by default (from v2.1.302; in interactive terminal sessions from v2.1.232), and while it is on, Claude can fork side tasks off on its own. So when the main conversation is on Fable, the forked tasks run on Fable too. To keep Claude from forking on its own, add `"Agent(fork)"` to `permissions.deny`, or set `CLAUDE_CODE_FORK_SUBAGENT` to `0`

**Which model the built-in subagents use**:

| Subagent | Model |
|---|---|
| Explore | The main conversation's model. **When the main conversation runs Fable**: if you connect through a subscription, a Console account, or a gateway reached through `ANTHROPIC_BASE_URL`, it runs on the model the `opus` alias points to; on Amazon Bedrock, Google Cloud, Microsoft Foundry, Claude Platform on AWS, or a Claude apps gateway, it stays on the main conversation's model |
| Plan | Inherits from the main conversation |
| general-purpose | `CLAUDE_CODE_SUBAGENT_MODEL` when it is set and no other source names a model; otherwise the main conversation's model |
| statusline-setup / claude-code-guide | Sonnet / Haiku |
| claude | No model of its own; when Claude spawns it as a subagent, it follows the order above |

Setting `CLAUDE_CODE_SUBAGENT_MODEL` alone does not change the model of Explore and Plan. To put Explore on a cheaper model, the official approach is to define a subagent with the same name, `Explore`, and give it `model: haiku`.

**effort** (subagents page, "Choose an effort level"):

- A definition file can set an `effort` field (`low`, `medium`, `high`, `xhigh`, `max`; which levels are available depends on the model)
- When you ask Claude to run a non-fork subagent at a specific level, it can pass an `effort` parameter on that invocation, which overrides the field in the definition (requires Claude Code 2.1.292 or later). The `CLAUDE_CODE_EFFORT_LEVEL` environment variable takes precedence over both
- When neither is set: a subagent on the same model as the main conversation uses the session's level; on another model the session's level doesn't apply, and it uses that model's saved level or its default (changelog 2.1.317). `CLAUDE_CODE_SUBAGENT_EFFORT` sets one level for every subagent that gets no level from elsewhere (stated only in changelog 2.1.316; not in the environment variables reference)

**When changes to a definition file take effect**: Claude Code watches `~/.claude/agents/` and `.claude/agents/`; in the docs' words, `Claude Code detects the change within a few seconds and the next delegation uses the updated definition, with no restart needed`. Three cases still need a restart: the first file in an `agents` directory created after the session started; `.claude/agents/` inside a directory added with `--add-dir` or `/add-dir`; and a session started with `--disable-slash-commands`.

**Concurrency** (subagents page, "Concurrent subagent limit"): by default, when 20 subagents are running in a session, spawning another with the Agent tool fails with `Concurrent subagent limit reached`, and spawning works again only once the running count drops. `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` changes this limit (any positive whole number; it can be changed but not turned off). Sessions with ultracode active are not bound by it. There is no limit on how many subagents a session can spawn in total. Nesting depth defaults to three levels; change it with `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`.

The docs give no recommended number for how many to run at once. What they give is a split by scale: subagents suit `A few delegated tasks per turn`; to coordinate dozens to hundreds of agents, use a dynamic workflow, which has its own limits (16 at once by default, at most 1,000 per run) and shows a `Large workflow` warning when it schedules more than 25 agents or its projected token total passes 1.5 million. Sixteen at once is the default, and fewer when Claude Code has fewer CPUs available; change it with `CLAUDE_CODE_WORKFLOW_MAX_CONCURRENT_AGENTS` (1 to 256).

Each agent in a workflow gets its model in the same order as a subagent: a model the script names for a stage counts as the first item, the one passed on this invocation, and when nothing names one, the agent runs on the session's model. The Dynamic workflows page gives two ways to control model cost: if you usually switch to a smaller model for routine work, check `/model` before a large run; and when you describe the task, ask Claude to use a smaller model for stages that don't need the strongest one. To estimate the spend before a large task, run it on a small slice first, for example one directory instead of the whole repo. The same page also states that a session with ultracode on uses more tokens on each request, and on a subscription reaches a session or weekly limit sooner.

**How a prompt asks for a workflow** (same page): include the keyword `ultracode` in the prompt, or write "use a workflow" or "run a workflow" in your own words; Claude treats a direct request as the same opt-in. The keyword counts only in a prompt the user types: a prompt passed with `-p` or a scheduled task prompt does not start a workflow this way. The size guideline is advice to Claude, not a cap, so a prompt that calls for a different scale overrides it. One of the page's example prompts:

```text
use a workflow to audit every route handler under src/routes/ for missing authentication checks, and adversarially verify each finding before reporting it
```

The Claude Code best practices give two more ways to run bulk work. For one big mechanical change across many files, use `/batch`; the page's example is `/batch migrate every handler to the v2 response helpers`, which splits the work into 5 to 30 units and shows you the plan, and once you approve it each unit gets its own worktree and opens a PR or reports what happened. When you loop over `claude -p` from a script: `Try it on three files first, see what goes wrong, fix the prompt, then run the rest.`

**How results come back**: subagents the main conversation starts run in the background by default, and their results arrive as completion notices on a later turn. While fork mode is on (the default), Claude can't wait for a subagent in interactive terminal, VS Code or desktop sessions on your machine; to let it wait, turn fork mode off first (`CLAUDE_CODE_FORK_SUBAGENT=0`). The result a subagent returns when it finishes goes into the main conversation's context. The official caution: `Running many subagents that each return detailed results can consume significant context, and each subagent spends tokens of its own while it runs.` So the brief should ask them to report back short summaries.

**What to put in the brief**: a non-fork subagent starts from a fresh context and doesn't see the conversation history, the skills already invoked, or the files Claude has read. It does load CLAUDE.md; the built-in Explore and Plan, and a subagent whose definition sets `omitClaudeMd`, skip the levels the main conversation loads, though a subdirectory's CLAUDE.md still loads when the subagent works with files there. The subagents page puts it this way: the main conversation still has the full CLAUDE.md when it reads these subagents' results, so most rules don't need to reach the subagent; a rule that must reach it (the page's example is ignoring the `vendor/` directory) should be restated in the prompt that asks Claude to delegate. The Agent SDK's subagents page: `The only content you pass from parent to subagent is the Agent tool's prompt string, so include any file paths, error messages, or decisions the subagent needs directly in that prompt.`

**Errors and usage limits** (subagents page, "API errors in subagents"): a subagent whose run ends on an API error (the docs' examples are a usage limit or a repeated server error) reports the failure back to Claude. If a foreground subagent already produced text, the Agent tool returns that partial output with a note that it was cut off and didn't finish; one that produced nothing, or only tool calls, fails with `Agent terminated early due to an API error`. A background subagent is marked failed, and the message Claude receives names the API error and includes the subagent's last output, `so partial work isn't lost`. Once the error clears, ask Claude to retry, or resume that subagent with `SendMessage`; a resumed subagent keeps its full conversation history. Explore and Plan are one-shot and can't be resumed. Subagent transcripts are saved separately from the main conversation, so a subagent can still be resumed after you restart Claude Code and resume the same session.

**When a subagent switches models partway**: a report that opens with `⚠ This agent switched to <model> because its earlier model could not be used.` means the subagent finished on a fallback model (this happens when a fallback model chain is configured and the subagent hits a failure the chain covers, such as its model being unavailable). When a safety classifier flags a request, the subagent doesn't stop to ask. When a fallback model is available: if `/config` is set to switch automatically, or no choice has been made yet, the request reruns on the fallback model and the main conversation's model stays the same; if it is set to ask each time, the subagent fails with `Agent terminated early due to an API error` and the refusal message. When no fallback model is available, the subagent fails the same way with either choice. From 2.1.319, after safeguards stop a subagent, Claude is told to retry with a narrower prompt or tell you, instead of doing the work itself (changelog).

A dynamic workflow is different: when one of its agents hits the claude.ai usage limit, the run pauses instead of failing that agent: the agents that hit the limit wait for the reset, no new agents start, and the run continues on its own after the reset (for the conditions, see "What happens at the limit" in `models.md`).

**When to delegate, per the docs** (Claude Code best practices, "Know what subagents cost"). It pays in four situations: running independent pieces of work at the same time; keeping long output (a full test run, a wide search) out of the main conversation; getting a reviewer that hasn't seen how the code was written; giving a task fewer tools than the main session has. The cost is mostly tokens, and sometimes time as well: a fresh subagent first has to gather context the main conversation already had. From the same page: `by default it's the same model (or a smaller one) on the same task. You get parallelism or a fresh pair of eyes out of it, but not a smarter answer.`

The cache counts toward the cost too: the prompt cache lasts about an hour on a subscription and five minutes on an API key, a cloud provider, or usage credits, while `Subagents get five minutes unless you change it`. Workflow agents also default to five minutes, on a subscription as well. Setting `subagentPromptCacheTtl` to `1h` keeps the cache for an hour, and 1-hour cache writes are billed at a higher rate.

What the Fable 5.2 page asks of prompts: first review the lines in system prompts, skills, and agent files that tell it to delegate; if it delegates more than the task needs, remove the lines you no longer need.

**When writing a subagent definition** (sub-agents page): Claude uses each subagent's `description` to decide when to delegate, so say clearly when it should be used, specifically enough to tell it apart from your other subagents. Keep descriptions short: when the combined descriptions of your custom subagents exceed 15,000 tokens, Claude Code shows a warning at startup; move the detail into the subagent's own system prompt, which loads only when that subagent runs. The same page says that phrases like "use proactively" in the description encourage Claude to delegate proactively; when the target is Fable 5.2, read that together with the previous paragraph, because the Fable 5.2 page says it delegates readily in a coding product whose prompts invite delegation. To guarantee that a given subagent runs for a task, @-mention it; naming it in plain language leaves the choice to Claude.

---

## This prompt versus project configuration

The test is simple: **information specific to this task goes in the prompt; conventions that hold every time get sedimented into project configuration.**

If you find yourself writing "we use pnpm, not npm" or "run lint before committing" — things that have to be said **every single time** — recommend the user move them into CLAUDE.md or a hook rather than pasting them again.

## Deterministic and non-deterministic instructions

The Claude Code best practices split every instruction into two kinds. A non-deterministic instruction is one the model reads and then decides what to do with (the prompt, CLAUDE.md, a skill); it usually follows it, but there is no guarantee. A deterministic instruction is one Claude Code, the harness, executes (a hook, a deny rule, a managed setting); it applies no matter what the model decides.

| | Non-deterministic (Claude decides) | Deterministic (Claude Code enforces) |
|---|---|---|
| What to do | The prompt, CLAUDE.md, skills Claude picks up itself | Hooks, skills only you can start |
| What it can touch | Auto mode's classifier | Deny rules, the sandbox in strict mode, managed settings |
| When it's done | A request for evidence, `/goal`, `/code-review` | Tests, type checks, a `Stop` hook, CI |
| How much it spends | Effort level, Claude's own pace | `--max-turns` or `--max-budget-usd` with `-p`, a spend limit you set |
| Who has the last word | Claude's summary of what it did | Branch protection, the person who approves the merge |

The official trade-off: most of the time, let Claude decide. Almost every guideline has exceptions, and with the actual code in front of it, Claude usually judges better than a rule you wrote in advance. Keep the deterministic kind for things with **zero exceptions**: `a hook is for a rule with zero exceptions`, and a preference that has exceptions (the official example is `Prefer named exports`) stays in CLAUDE.md. The other way round: if you have a hook for something that is only a preference, delete it and let Claude decide.

---

## Where project configuration goes: the seven methods

Anthropic's "Steering Claude Code" article lists seven methods; CLAUDE.md takes two rows here (root and subdirectory).

| Mechanism | Loaded | Context cost | Best for |
|---|---|---|---|
| **CLAUDE.md (root)** | Session start, stays throughout, re-read after compaction | **High** (always loaded) | The commands that confirm work (tests, type check, lint, how to run a single test file), places where the code is misleading, decisions that aren't in the repo, anything already corrected twice. **Keep under 200 lines** |
| **CLAUDE.md (subdirectory)** | On demand when that directory is touched | Low | Directory-specific conventions |
| **Rules** | Session start (unscoped) or on file access (scoped) | Medium | Rules that only matter in one part of the codebase. **Add a `paths:` filter** so they load only when Claude is actually working there |
| **Skills** | Name and description at start, body on invocation | Low | Team procedures and knowledge: the release process, the incident runbook, how API docs are written. For a workflow with side effects, add `disable-model-invocation: true` so it runs only when you type the command yourself |
| **Subagents** | Name and description at start, body when called; only the final message returns | Low for the main context; its own requests still count toward usage | Side tasks where you only want the conclusion |
| **Hooks** | Fire on lifecycle events | Low (outside main context) | Zero-exception rules and deterministic automation: formatting, blocking edits to a directory. For real enforcement, use a `command` or `http` hook |
| **Output styles** | Session start; still in effect after compaction | **High** | Tone, length, or format you want on every reply, or having Claude work in a role other than software engineer |
| **Appending the system prompt** | Passed as a CLI flag at startup | Moderate | Instructions in scripts and automation that need to sit at the system prompt level (`--append-system-prompt`) |

**What belongs in CLAUDE.md** (best practices, "Keep CLAUDE.md short"): how to confirm work in this repo — the exact test, type check, and lint commands, and how to run a single test file; places where the code is misleading — an old pattern that's everywhere but that new code must not copy, a directory that looks unused but isn't; decisions that aren't in the repo; anything you've already had to correct twice.

**How to write it** (Memory page): Claude treats CLAUDE.md as context, not enforced configuration, so how you write an instruction affects how reliably it is followed. Make instructions concrete enough to verify: write "Use 2-space indentation", not "Format code properly". If two instructions contradict each other, Claude may pick one arbitrarily, so remove outdated and conflicting ones from time to time. CLAUDE.md content is delivered as a user message after the system prompt, not as part of the system prompt.

**What does not**: anything Claude can work out by reading the code. The docs list describing architecture and file layout in CLAUDE.md among the old habits you can drop. Personal preferences go in `CLAUDE.local.md`, which you add to `.gitignore`.

**When one line keeps getting ignored**: adding IMPORTANT probably won't help (SKILL.md 3.3). Either the file is too long and the line gets lost, so shorten the file; or it is a rule that can't have exceptions, so make it a hook.

**Tools**: `/init` generates a starting version from the codebase, after which the work is mostly removing lines; `/doctor` suggests cutting what Claude can work out from the code; `/doctor prompt-audit` flags instructions that were written for older models, point at files that no longer exist, or contradict each other; in a session, `/context all` confirms the file was loaded.

**How to write a skill** (three-tier progressive disclosure, from Anthropic's Agent Skills article):

| Tier | Content | Loaded |
|---|---|---|
| Metadata | YAML frontmatter, only `name` and `description` | Preloaded into the system prompt at startup |
| Primary instructions | The `SKILL.md` body | When Claude judges the skill relevant |
| Bundled files | Files referenced from SKILL.md | Only when that specific scenario arises |

The CLAUDE.md rule applies to skills too: only write down what Claude can't find on its own. When you migrate to Fable 5.2, skills and agent files are also within what the Fable 5.2 page asks you to review (SKILL.md 3.13).

Two writing tips from the Claude Code docs that concern truncation: in the skill listing, the combined `description` and `when_to_use` text is cut off past 1,536 characters, so put the main use case first (Skills page); after compaction, large skill bodies that are re-injected get truncated to a per-skill cap that keeps the start of the file, so put the most important instructions near the top of `SKILL.md` (Context window page).

---

## Turning manual checks into verification loops

From Anthropic's "Building verification loops in Claude Code with skills" article. The test: any check the user keeps having to run by hand is a candidate for a verification loop, especially a deterministic rule that a generic linter won't catch; the article's example is `Reject any migration that drops a column without a backfill step`.

| Mode | Trigger | Best for |
|---|---|---|
| **Standalone** | Invoked deliberately | Cross-cutting checks that do not apply universally: security scans, accessibility audits, license verification |
| **Embedded** | Fires automatically inside the producing skill | Checks that belong to one specific task |
| **Chained** | One skill calls another on completion | End-to-end handoff; also how you add verification to a **skill you cannot modify** |
| **On every PR** | CI | Turns a personal habit into team infrastructure, removing dependence on author diligence |

Embedding only works on editable skills. Built-in or plugin-managed skills update automatically and would overwrite your change; for those, use chaining.

## Four kinds of loops

From Anthropic's "Getting started with loops" article.

| Type | Command | Best for |
|---|---|---|
| **Turn-based** | Manual, turn by turn | Short tasks |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | Tasks with a **verifiable exit criterion** where one turn is not enough |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | Recurring work, interfacing with external systems |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | Well-defined recurring streams: bug reports, issue triage, migrations |

Cost control: do deterministic work with scripts, match the interval to how often things actually change, and pilot before running at scale. In the article's words: `Your model and effort level choices are among the biggest levers on what a loop costs.` For proactive loops, the way it gives to manage usage is `Routing routines to smaller, faster models and using the most capable model for judgment calls`. The Claude Code best practices add that each `/loop` run is another turn in the same session, so don't leave one going in a long conversation.

## Five questions before an unattended run

Claude Code best practices, "Let Claude run without you". Before you let a task run on its own, go through these with that specific task in mind:

1. Does it know what done looks like? A goal condition, a spec file, or a failing test it has to turn green
2. Can it check its own work? The test suite, a `Stop` hook, some kind of review step
3. What is it allowed to touch? Permission rules, `dontAsk`, the sandbox, a worktree, the one branch it may push to
4. What stops it? `--max-turns`, `--max-budget-usd`, a spend limit
5. Where does the result go, and who signs off? A PR, a comment, or a summary, and then a person approving the merge, the deploy, or anything else that can't be taken back

The official reasoning: while you are watching, Claude missing some context doesn't matter much, because you notice within a message or two and tell it. In a run started before bed, nobody tells it, and you find out the next morning that it spent the whole night building on the wrong assumption.

## Old habits you can drop

Claude Code best practices, "What you can stop doing". When you review a user's Claude Code prompt or CLAUDE.md, check it against this table:

| Old habit | What the docs say now |
|---|---|
| Treating the context window as the main constraint and managing it aggressively | Most current models have a 1M-token window and compaction happens on its own, so running out is rare. `/clear` between tasks is still worth doing: a focused context gives better answers and costs less |
| Adding IMPORTANT or ALL CAPS to instructions it keeps missing | Current models follow a plainly worded line, and piling on emphasis mostly makes them over-apply the rule. Make a rule that can't have exceptions a hook or a deny rule; if the line is lost in a long file, shorten the file |
| Using subagents to protect the context window | Running out of context is rarely a problem anymore. Subagents still suit parallel work, keeping noisy output out of the way, and independent review, but each of those costs extra tokens |
| Delegating to a subagent for a better answer | Handing the same task to a subagent doesn't make the answer better. A fresh reviewer helps because it hasn't seen your reasoning, and that is a different thing |
| `think hard`, `think harder`, `be thorough` in the prompt | That is effort's job. These are ordinary prompt text. `ultrathink` still works: it asks for deeper reasoning on that one turn |
| Telling it to keep going, loop until done, don't be lazy | Current models usually carry an agreed task through without being told, and lines like that are more likely to cause over-work. Giving it a check to pass is different, and still worth doing. If a run really does stop early, say `continue`; for runs nobody is watching, enforce it with `/goal` or a `Stop` hook. When the target is Fable 5.2, also see what the Fable 5.2 page asks: when you tell it to keep going, say what you want it to do rather than send a bare continue (see "Say it all at once") |
| Describing architecture and file layout in CLAUDE.md | Claude reads the repo itself. Only write down what isn't in there |

## When a session doesn't go as expected

The diagnosis at the top of the Claude Code best practices, worth passing on to the user along with the prompt:

| Symptom | Most likely cause | Section to read |
|---|---|---|
| It built something different from what you had in mind, or every session has to rediscover the same fact | It is missing information it can't get by reading the repo | Tell it only what it can't see |
| It said it was done, and it wasn't quite | It had no way to check its own work | Give it a check it can run |
| It did something that must never happen in this repo, or skipped a step that must always happen | A prompt alone can't guarantee either | Deterministic instructions |
| It got it right, but took more time, tokens, or permission prompts than the task was worth | A default doesn't fit | `/usage` to see where it went; SKILL.md section 2 |

Three controls inside a session ("Steer the session"): `Esc` stops it mid-action and keeps everything up to that point; `/rewind` rolls the code, the conversation, or both back to an earlier prompt; if you've corrected the same thing twice and it's still wrong, run `/clear` and fold what you learned into one new prompt — a third correction mostly adds to a conversation that already contains two failed attempts.
