# Prompt engineering in 2026: where these rules come from

> Background reading for the `prompt-engineering` skill. The skill tells you **what to do**; this document explains **why** — the official basis for each rule, which older habit it displaces, and how to read the two places where Anthropic's own guidance appears to contradict itself.
>
> Snapshot **2026-07-26**. Two different kinds of correction went into it: a source page that changed under the snapshot (the effort guidance), and an exact-string verification pass that caught two snippet attributions this document had got wrong on its own. Everything here traces to Anthropic's official documentation and official or employee public material; guidance predating the Claude 5 generation has been removed rather than left in. Sources are itemized in [SOURCES.md](../SOURCES.md).
>
> **How to read it.** Sections 1–3 build the frame. Section 4 is the important one — if you learned prompt engineering before 2026, it will contradict part of what you know. Sections 5 and 5b are the two decisions people most often get wrong, both arising from official guidance that looks contradictory and is not. Section 6 onward is reference material.
>
> **Scope: Claude Opus 5 and Claude Fable 5** (including Mythos 5), plus techniques verified as current and non-conflicting. Guidance specific to earlier models has been deliberately excluded. (Two entries in the skill's snippet library are verbatim from the current Sonnet 5 page — deliberate, and flagged at each entry.)
>
> **One disclaimer.** For specific API parameters, the live documentation is authoritative. The appendix records a real lesson on this point: a passage quoted in this document was rewritten by Anthropic *during* the verification process.

---

## Contents

1. What actually changed
2. Three layers: prompt → context → harness
3. Foundations that still hold
4. **Obsolete: eight habits to unlearn**
5. The most misapplied decision: whether to give examples
5b. The second one: whether to remove verification instructions
6. Model differences: Opus 5 and Fable 5
7. Agents and long tasks: context engineering
8. Claude Code: where to put what
9. How to practice
10. Sources and a reading order

Appendix: the verification record

---

## 1. What actually changed

**A large share of what the field has taught over the past two years is now being reversed by Anthropic itself.**

The single strongest piece of evidence: a 2026-07-24 post from the Claude Code team reports that they cut the Claude Code system prompt by **more than 80% with no loss of performance**.

That is not an isolated data point. Two further threads point the same direction:

- **"removing examples was extremely helpful, because it was just more creative than the examples we gave it"** (Thariq Shihipar, AI Engineer World's Fair, 2026-07)
- **As Claude improves, scaffolding that was once necessary becomes dead weight and turns into a bottleneck** (Lance Martin, Claude Platform team, 2026-04)

One sentence for the direction of travel:

> **From carefully constructing instructions to carefully constructing context and tools.**

Why? Because the model's judgment improved. You used to write `Never write multi-line docstrings` because the model could not make that call itself. Now you write `Write code that reads like the surrounding code` and it can. The older form also carried a hidden cost — **a hard constraint that later collides with what the user actually wants leaves the model badly confused**. Thariq Shihipar, at the same fireside chat:

> "The other thing we did is try to give it more context and fewer 'do not do this' instructions, because that's a very strong impulse for Claude, and especially if it conflicts with user instructions later on, that can be extremely confusing to Claude — 'I've got this skill that says this and the system prompt says this.'"

That last clause is the part worth keeping. The failure mode is not disobedience. It is **the model holding two instructions that contradict each other and having no basis for choosing**.

---

## 2. Three layers: prompt → context → harness

Most of the confusion in this area comes from collapsing three distinct layers into one.

### 2.1 Prompt engineering

Writing and organizing instructions to an LLM for optimal results. **A one-time act of writing.** It fits single-turn work with a well-defined task.

### 2.2 Context engineering

The full set of strategies for curating and maintaining the optimal set of tokens during LLM inference, including all the information that falls outside the prompt itself.

The distinction that matters: **prompt engineering is writing done once; context engineering is iterative — every turn, you decide again what goes into the model.**

System instructions, tools, external data, message history, memory — all of it counts. In multi-turn agent work this layer matters far more than the prompt layer does.

The governing concept is **context rot**: the model has a finite attention budget, and every token spends a little of it. Attention in a transformer is quadratic, so the longer the context, the more diluted the attention. The goal is therefore not to tell it everything you know, but to find:

> **the smallest set of high-signal tokens that maximize the likelihood of the desired outcome.**

### 2.3 Harness and loop engineering

The software infrastructure around the model: the loop, the tools, context management, guardrails. Lance Martin's central observation is that as Claude improves, the assumptions baked into a harness **expire** — scaffolding that was necessary becomes dead weight. The question to ask on a schedule is **what can I stop doing?**

**The practical consequence:** when you catch yourself writing a long list of rules into a prompt, stop and ask whether the problem belongs one layer out. A different tool design? A deterministic check? Or nothing at all?

---

## 3. Foundations that still hold

### 3.0 The preconditions nobody reads

The official prompt engineering overview opens with something most people skip. **Before you tune a prompt you should already have:**

1. clear success criteria,
2. a way to test empirically against them,
3. a first draft to improve.

Without the first two, you are tuning blind. (The third is optional — the official [metaprompt recipe](https://colab.research.google.com/github/anthropics/claude-cookbooks/blob/main/misc/metaprompt.ipynb) will generate a draft for you.)

One more line from the same page carries as much weight: **not every failing eval is best solved by prompt engineering.** The example given is latency and cost, where changing models is often easier than changing words.

That framing matters: **the prompt is one tool in the box, not a hammer for every nail.** Sections 7 and 8 keep returning to the same pattern — problems that look like bad prompt writing usually belong in tool design, a schema, a hook, or the choice of model.

---

The new rules overturn *over-constraining*, not *being clear*. Everything below is still taught in the current documentation.

### 3.1 Be clear and direct

Treat Claude as a **brilliant new employee** — capable, but with no knowledge of your conventions or workflows.

**The golden test:** show the prompt to a colleague with no context on the task and ask them to follow it. If they would be confused, so will Claude be.

If you want behavior beyond the basics, say so. Do not expect it to be inferred from a vague brief.

- ✗ `Create an analytics dashboard`
- ✓ `Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.`

### 3.2 Give context, and give the reason

Explaining *why* lets the model generalize from the explanation. This matters more now than it used to — it is the concrete form of "more context, fewer instructions."

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

The second version still contains "never." The difference is that it hangs on a **reason**. Once the model knows why, it can decide for itself in the edge cases the rule never covered.

### 3.3 Structure with XML tags

> ⚠️ A November 2025 official blog post says XML tags are "less necessary now." **The current documentation contradicts this** — see reading-list entry 6 in Section 10, and "Known conflicts" in [SOURCES.md](../SOURCES.md).

Official wording:

> "XML tags help Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs. Wrapping each type of content in its own tag (for example, `<instructions>`, `<context>`, `<input>`) reduces misinterpretation."

Use consistent, descriptive tag names, and nest them where the content has a natural hierarchy.

A pattern shows up in the official snippet library, but it runs **one way only**. Every tagged block is a standing policy — how to act by default, how to call tools, how to format output, what tone to hold: `<default_to_action>`, `<do_not_act_before_instructions>`, `<use_parallel_tool_calls>`, `<investigate_before_answering>`, `<avoid_excessive_markdown_and_bullet_points>`, `<frontend_aesthetics>`, `<tone_preference>`.

The converse does not hold. Plenty of standing policies are left plain — the guardrails for irreversible actions, the autonomous-operation instruction, the memory-writing conventions. The clearest case is one policy appearing both ways: the verbosity instruction is plain text, while the short end-of-prompt reminder of that same policy is wrapped in `<tone_preference>`.

So the criterion is neither length (the shortest tagged block is a single sentence) nor subject matter. A tag earns its place when the block needs to be **addressed as one named unit** — set apart from what surrounds it, or referred to from elsewhere. This is an observed pattern rather than a documented rule, and a wrapper costs almost nothing, so when in doubt, tag it.

Markdown headers (`## Tool guidance`, `## Output description`) are an equally sanctioned way to partition a prompt, and they mix freely with XML. That comes from Anthropic's context engineering article.

### 3.4 Give it a role

Setting a role in the system prompt focuses behavior and tone. **A single sentence is enough to have an effect:**

```
You are a helpful coding assistant specializing in Python.
```

> ⚠️ The same 2025-11 post calls role prompting "often unnecessary." The current documentation still recommends it — again, see Section 10. The post does contribute one useful corollary, though: **stating the perspective to take usually beats attaching an identity label.** Rather than `You are a financial advisor`, write `Analyze this portfolio, focusing on risk tolerance and long-term growth`. The two are compatible and stack.

### 3.5 Long-context placement (the best return per character)

For documents past roughly 20k tokens:

- **Long material at the top, instructions and the question at the end.**
- Official wording: `Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs.`

Wrap multiple documents like this:

```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>
      {{ANNUAL_REPORT}}
    </document_content>
  </document>
</documents>

Analyze the annual report... (the question goes here, last)
```

There is also **grounding in quotes**: have the model first extract the relevant passages into `<quotes>` tags, then answer from those. It keeps attention on the relevant material and lets the rest fall away.

### 3.6 Say what to do, not what to avoid

This runs through every official source and is the most portable principle in the set.

- ✗ `Do not use markdown in your response`
- ✓ `Your response should be composed of smoothly flowing prose paragraphs.`

A neat supplementary lever: **match the style of your prompt to the style you want out of it.** Strip markdown from the prompt and you get less markdown back.

The Opus 5 guidance says the same, in the specific context of communication style: **positive examples of the style you want tend to be more effective than instructions about what not to do.**

---

## 4. Obsolete: eight habits to unlearn

This section is the main reason the document exists. Each entry names where the old advice came from and what supersedes it, so you can check it yourself.

### 4.1 ❌ Prefill — now rejected by the API

**The old advice:** prefill the assistant turn with `{` to force JSON, or with `Here is the summary:` to strip the preamble. The November 2025 official post still lists this as an advanced technique.

**Now:** official wording —

> "Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported. Requests with prefilled assistant messages to these models return a 400 error."

**Why it went away:** the stated reason is that intelligence and instruction following have improved to the point where most prefill use cases no longer need it.

**Replacements:**

| Old use | What to do now |
|---|---|
| Force JSON or a fixed structure | **Structured Outputs**, or state the schema directly — current models match complex schemas reliably |
| Classification | A tool with an enum field, or structured outputs |
| Strip the preamble | `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.` |
| Route around a mistaken refusal | Refusal judgment is far more accurate now; state the context in the user message |
| Continue a truncated response | Move the prior text into the user message and have it continue, or simply retry |

### 4.2 ❌ `budget_tokens` — removed

**The old form:** `thinking: {type: "enabled", budget_tokens: 10000}`

**Now:** neither model supports manual extended thinking; setting `budget_tokens` returns 400.

**The new form**, on Opus 5:

```python
thinking={"type": "adaptive"},
output_config={"effort": "high"},
```

On Fable 5, thinking is always on and cannot be configured at all — `effort` is the only depth control you have.

**What adaptive thinking is:** the model decides when to think and how much, based on two things — the `effort` parameter and the complexity of the query. Simple questions get answered directly; hard ones get more thought. Anthropic's internal evaluations show adaptive thinking reliably outperforming manual extended thinking.

For a hard ceiling, use `max_tokens` — noting that it caps **thinking plus response text combined**.

### 4.3 ⚠️ Forceful wording to prevent undertriggering — reversed, with a boundary

**The old form:** `CRITICAL: You MUST use this tool when...`

**Why it was written that way:** early models were not eager enough to reach for tools, so people applied pressure.

**Now:** the pressure is unnecessary. Fable 5's page states it plainly —

> "Instruction-following is improved enough that you can steer most behaviors with a brief instruction rather than enumerating each behavior by name."

That is the basis on which the Claude Code team cut their system prompt by more than 80% without losing performance. Stacked emphasis now only consumes attention budget, and it creates confusion when it collides with what the user actually wants.

**The new form:** `Use this tool when...` is enough. The same goes for `If in doubt, use [tool]` and `Default to using [tool]`, which become `Use [tool] when it would enhance your understanding of the problem`.

**The boundary** — this is the entry most often over-applied:

- Default to ordinary declarative wording. Add a targeted clarification only when you actually observe undertriggering
- The Claude Code best practices **still** recommend adding emphasis (`IMPORTANT`, `YOU MUST`) to improve adherence in **CLAUDE.md**

These do not conflict. The first concerns legacy anti-undertriggering scaffolding in an API system prompt; the second concerns marking a handful of critical conventions in a project configuration file. **The test is what problem you are solving, not whether the word is allowed.**

### 4.4 ❌ Telling Opus 5 to check its work — actively harmful

**The old advice:** the general documentation still carries `Before you finish, verify your answer against [test criteria]`.

**Opus 5 is the exception, and the wording is unusually direct:**

> "If your prompt contains explicit verification instructions ('include a final verification step for any non-trivial task,' 'use a subagent to verify'), remove them: instructions like these cause over-verification on Claude Opus 5, and removing them reduces wasted tokens with no loss in quality."

Note the verb: **remove**, not rewrite. It is counterintuitive enough to be worth memorizing on its own.

The same applies to `double-check your answer` and `re-verify before responding`.

### 4.5 ❌ Stacking absolute prohibitions

The principle is in Section 1. What the Claude Code team adds is the operational detail. Cat Wu, describing how they audited their own system prompt:

> "we found a few cases where yes, this statement is 90% true, but there's a real 10% of cases where it's not true"

Their response was to **delete** those 90%-true hard rules, because the remaining 10% collides with real user intent and the collision costs more than the rule was worth.

Where a rule is worth keeping at all, the move is to convert the command into a situation. Instead of `always run the app locally before finishing`:

> "most of the time when you're doing front-end work you can't fully understand the experience by hitting the backend endpoints, so when you make larger changes to the user experience, please run the app locally."

The difference is that the second version supplies **when and why**, and hands the judgment back to the model.

### 4.6 ❌ Repeating an instruction in several places

**The old habit:** afraid the model will miss it, you say it in the system prompt, again in the tool description, and again in the docs.

**Now:** current models do not need the repetition. The Claude Code team's practice is to keep guidance on how to use a tool **in the tool description, once**, and to delete the duplicated copy from the system prompt.

### 4.7 ❌ Hand-maintaining CLAUDE.md as memory

**The old practice:** using the `#` shortcut to write notes into CLAUDE.md.

**Now:** automatic memory captures the relevant context, so that part no longer needs hand-maintenance.

(To be clear, CLAUDE.md itself is not obsolete — it is still where build commands, directory layout, and team conventions belong. What is obsolete is using it as a notepad. See Section 8.)

### 4.8 ❌ A simplified Markdown spec

**The old practice:** write a simplified Markdown description of the system and hand that to the model *in place of* the real material.

**Now:** supply **rich references** instead — the actual code, the test suite, an equivalent function in another repository, an interactive HTML artifact. Current models handle complex reference forms, and the information density is far higher than that of a hand-written summary.

To be clear about what this does *not* deprecate: a spec the user has read and confirmed is a different object. Section 8.4 recommends producing exactly that, and the reason it works is that it functions as an agreement rather than as a lossy stand-in for the code.

### Two API-level changes worth knowing

- **Support for sampling parameters (`temperature`, `top_p`, `top_k`) is narrowing**, and some newer models return 400 for non-default values. Get stylistic variety from the prompt instead — the most effective move is to have the model propose several distinct directions and have *you* pick one before it builds.
- **Newer models may use a different tokenizer**, so the same text has a different token count and an inherited `max_tokens` may now truncate. Check both in the migration guide when you upgrade.

### The summary table

| Old habit | Status | Replacement |
|---|---|---|
| Prefill | 400 from Claude 4.6 onward | Structured Outputs / explicit instruction |
| `budget_tokens` | 400 | `adaptive` + `effort` on Opus 5; `effort` alone on Fable 5 |
| `CRITICAL: You MUST` (anti-undertriggering) | No longer needed; consumes attention. A few marked conventions in CLAUDE.md excepted | `Use this tool when...` |
| Telling Opus 5 to self-check | Causes over-verification | **Remove it** |
| Stacked `Never do X` | Confuses the model when it collides with user intent | Positive description plus the reason |
| The same instruction in several places | Redundant | One authoritative location |
| Hand-maintained CLAUDE.md memory | Superseded | Automatic memory |
| A Markdown description used *in place of* the code | Low information density | Code / tests / artifact |
| `temperature` for variety | Support narrowing; 400 on some models | Have it propose options and pick one yourself |

---

## 5. The most misapplied decision: whether to give examples

This is where the available information is most confusing, because you will find both positions and **both are official**:

- **The documentation:** `Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.` and `Include 3–5 examples for best results.`
- **The Claude Code team:** `removing examples was extremely helpful, because it was just more creative than the examples we gave it.`

Simon Willison was tripped up by this during the fireside chat — giving examples is one of the pieces of prompt advice he most often passes on, and if it no longer holds, his mental model of prompting is on shakier ground than he thought.

**Both are correct, because they address different situations.**

| Your goal | Examples? | Why |
|---|---|---|
| Lock the output into a **fixed shape** — a JSON schema, a house voice, a fixed report structure | **Yes.** 3–5, diverse, wrapped in `<example>` | Examples are constrainers, and constraint is exactly what you want here |
| Let the model **exercise judgment** — an agent system prompt, open-ended creative work | **No.** Improve the tool interface instead | Examples collapse the exploration space toward the ones you gave, even where the model would have done better on its own |
| Demonstrate a **reasoning pattern** | Use `<thinking>` tags inside the few-shot examples; the model generalizes the style to its own thinking blocks | Explicitly supported by the docs |

One line to remember:

> **Examples constrain. Use them when you want constraint; drop them when you want creativity.**

The Claude Code team removed theirs because they were writing the system prompt for a general-purpose coding agent, where what you need is precisely the model's own judgment rather than template-following. If what you want is the same weekly report format every time, examples remain the most effective tool available.

The three official requirements when you do include them: **relevant** (mirroring the real use case), **diverse** (covering edge cases and varied enough that the model does not pick up unintended patterns), and **structured** (wrapped in `<example>` tags).

---

## 5b. The second one: whether to remove verification instructions

This apparent contradiction is subtler than the one about examples.

You will read both of these:

- **The Opus 5 page:** if your prompt contains explicit verification instructions, **remove them** — they cause over-verification, wasting tokens with no gain in quality.
- **The Fable 5 page:** for long-running tasks, explicitly build in self-checking: `Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.`

**The resolution is the clause about subagents, plus the sentence the Fable 5 page puts immediately before that instruction:** officially, **separate, fresh-context verifier subagents tend to outperform self-critique.**

So the actual rule is:

| | Remove | Add |
|---|---|---|
| **Form** | The model **looking at its own work again** ("double-check", "re-verify", "add a final verification step") | A **clean context** doing the checking (a fresh-context verifier subagent) |
| **Situation** | Ordinary tasks, where the model self-corrects in one pass | Any task where correctness matters more than speed — long autonomous runs additionally make the check *periodic* rather than one-off |

One line to remember:

> **Delete "check yourself." Keep "have someone else check."**

This also explains the other pair that looks like a collision: the Opus 5 guidance says not to use subagents to verify or double-check its own work, while the Fable 5 guidance recommends subagents verifying against the specification on an interval. Read them on the same axis, and they stop colliding. What 4.10 forbids is **delegating self-review** — handing a subagent the model's own work *together with its own criteria*, which pays twice for a judgment Opus 5 already makes. A **separate verifier with independent criteria in a fresh context** is a different thing, and stays permitted on both models at any duration. What Fable 5's long runs add is not permission but **periodicity**: run that check on an interval instead of once at the end.

There is a trap one level down: **the word "verification" slides between two meanings.** What you remove is the model subjectively reconsidering whether it got it right. What you must **not** remove is running the tests, reading the build exit code, diffing a screenshot against the design — **external deterministic checks**. Those are not self-verification; they are the signal that lets the loop close. Section 8's headline rule is about exactly those.

---

## 6. Model differences: Opus 5 and Fable 5

Current models are not a linear "bigger is better" progression. They have **different dispositions**, and they want to be tuned in different directions.

### 6.1 Claude Opus 5

Positioned for complex agentic coding and enterprise work, with strength on long-horizon tasks. Default effort `high`; thinking on by default; a 1M token context window as both default and maximum.

| Behavior | What to do about it |
|---|---|
| User-facing responses run long by default | Ask for concision explicitly. **Note that lowering effort does not reliably shorten visible output** — effort governs how much it thinks, not how much it says |
| Narrates readily during agentic work | Describe the cadence you want |
| Files written to disk also run long | Add a separate length calibration |
| Verifies its own work thoroughly | **Remove** every verification instruction |
| Expands task scope on its own | Constrain scope explicitly on narrow tasks |
| Delegates to subagents more readily | Set an explicit cap, or a small task doubles in cost |
| Narrates corrections to its own earlier statements more often | Restrict it to corrections that would change the user's decisions |
| Follows "only report high-severity issues" **literally** in code review | Change to "report everything; filter downstream" |
| `low` and `medium` produce strong quality at a fraction of the tokens and latency of higher settings | **Start at the default `high`** and step up to `xhigh` only for demanding coding and agentic work. If you carried settings over from an earlier model, run a fresh effort sweep |

**One trap.** With thinking disabled, Opus 5 occasionally writes a tool call into user-facing text instead of emitting a `tool_use` block — the call never runs, and in an agentic loop the leaked text stays in history and affects later turns. It can also leak `<thinking>` tags into visible output. The official advice is to **keep thinking enabled and control cost with lower effort** rather than turning it off; for most tasks, thinking enabled at `low` effort beats thinking disabled at similar cost. (At `xhigh` and `max` you cannot turn it off in any case; those requests return 400.)

If an integration must keep thinking disabled, the official mitigation is a single combined instruction covering both artifacts:

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

Naming the thinking tags specifically works less well than that general form.

### 6.2 Claude Fable 5 / Mythos 5

Officially **Anthropic's most capable widely released model**, and positioned very differently from Opus 5: for **problems that were previously too complex, long-running, or ambiguous**, and particularly for end-to-end work that takes a person hours, days, or weeks. Thinking is always on and cannot be configured.

Worth separating two questions that get conflated here. The official selection advice is to **start with Opus 5 for complex agentic coding and enterprise work, and reach for Fable 5 when a workload needs the highest available capability.** "Most capable" is not the same as "what you should default to."

One line worth noting: **the teams seeing the best outcomes apply it to their hardest unsolved problems; testing it only on simpler workloads tends to undersell its capability range.**

| Behavior | What to do about it |
|---|---|
| **Individual requests can run for many minutes; autonomous runs can extend for hours** | Adjust client timeouts, streaming, and progress indicators before migrating; consider checking on runs asynchronously rather than blocking |
| Overplans when the task is ambiguous | Add: when you have enough information to act, act; do not narrate options you will not pursue |
| Instruction following strong enough to **steer a class of behavior with one brief instruction** | Write one short instruction instead of enumerating each behavior |
| Can fabricate progress on long autonomous runs | The highest-value single prescription for this model: **require every progress claim to be audited against a tool result from this session.** In Anthropic's testing this nearly eliminated fabricated status reports |
| Takes unrequested actions (drafting an email, creating defensive git branches) | Define boundaries: when the user is describing a problem or thinking out loud rather than requesting a change, the deliverable is your assessment — report and stop. The official wording carries a second half that is easy to miss: **before running a command that changes system state, check that the evidence actually supports that specific action — a signal that pattern-matches to a known failure may have a different cause** |
| Dispatches parallel subagents more readily | **Encourage it**, but give explicit criteria; prefer asynchronous communication; long-lived subagents save cost through cache reads |
| Suggests a new session when shown a remaining-token countdown | Tell it the context is ample and not to stop on that account; better still, do not surface the countdown at all |
| **Skills written for prior models are often too prescriptive and can degrade output quality** | Review and prune |

That last row deserves emphasis — it is the bluntest official statement of the theme that a stronger model makes scaffolding more costly, not less:

> "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality."

**An old skill is not merely useless; it can actively drag quality down.**

There is also a trap specific to this model: **do not instruct it to reproduce or display its internal reasoning.**

> "Prompts, skills, or harness instructions that tell the model to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category on Claude Fable 5, causing elevated fallbacks to Claude Opus 4.8."

For reasoning visibility, read the structured `thinking` blocks the API returns. To surface progress mid-run, build a send-to-user tool.

### 6.3 The effort parameter

`max` > `xhigh` > `high` (default) > `medium` > `low`

The official description of each level:

| Level | Official description | Typical use case |
|---|---|---|
| `max` | Absolute maximum capability with no constraints on token spending | Tasks requiring the deepest possible reasoning and most thorough analysis |
| `xhigh` | Extended capability for long-horizon work | Long-running agentic and coding tasks (**over 30 minutes**) with token budgets in the millions |
| `high` | High capability. **Equivalent to not setting the parameter** | Complex reasoning, difficult coding problems, agentic tasks |
| `medium` | Balanced approach with moderate token savings | Agentic tasks that require a balance of speed, cost, and performance |
| `low` | Most efficient. Significant token savings with some capability reduction | Simpler tasks that need the best speed and lowest costs, such as subagents |

**Both models start at the default `high`.** What differs is when to step up:

| Model | Step up when |
|---|---|
| **Opus 5** | The coding or agentic work is **demanding** → `xhigh`; a task that justifies unconstrained token spending → `max`. Use `low`/`medium` liberally as the primary cost and latency control wherever your evals show quality holds |
| **Fable 5** | Only the most capability-sensitive workloads → `xhigh`; routine work steps down to `medium`/`low` — its lower effort settings often exceed `xhigh` performance on prior models |

> **This entry was corrected.** An earlier version of this document said Opus 5 coding and agentic work should *start* at `xhigh`, on the basis of the official page's then-current wording, "for coding and agentic work, `xhigh` remains the recommended starting point." That page now reads "Start with the default (`high`) … step up to `xhigh` for demanding coding and agentic work." The practical difference is real: an ordinary coding task now starts at `high`. The full account is in the appendix.

Points that matter:

- `Setting effort to "high" produces exactly the same behavior as omitting the effort parameter entirely.`
- `Effort is a behavioral signal, not a strict token budget. At lower effort levels, Claude will still think on sufficiently difficult problems, but it will think less than it would at higher effort levels for the same problem.`
- It affects **all tokens** in the response: text, tool calls, and thinking
- **Lower effort directly reduces the number of tool calls**, combines operations, skips preamble, and produces terse confirmations. Higher effort means more tool calls, plans explained before acting, and detailed summaries. This is the part people miss — **effort is not only how long it thinks; it is how much it does**
- **Set effort deliberately per workload** rather than letting one setting ride across everything you run
- **Do not change effort mid-session with prompt caching enabled.** Official wording: `Changing the effort value between requests invalidates prompt caching, so vary effort across workloads rather than within a conversation that relies on cache hits.`
- At high effort, leave headroom in `max_tokens` — it is a hard limit on thinking plus response text combined, and without headroom you get a response truncated because thinking consumed the budget. On Opus 5 at `xhigh`/`max`, official guidance is to start at 64k and tune from there

> Aside: `ultracode` in Claude Code is **not** an additional API effort tier. It is `xhigh` plus a standing authorization to launch multi-agent workflows.

---

## 7. Agents and long tasks: context engineering

### 7.1 The binding constraint: context rot

The model has a finite attention budget. The longer the context, the more diluted the attention and the worse the performance. This is not an engineering bug; it falls out of the quadratic complexity of transformer attention.

So the goal of context engineering is not "make it fit" but **the smallest set of high-signal tokens**.

### 7.2 The right altitude

A system prompt can fail in two directions:

- **Too specific**: brittle, hardcoded logic in the prompt, creating fragility and maintenance burden
- **Too vague**: high-level guidance with no concrete signal, or assuming shared context the model does not have

The target band is **specific enough to guide behavior effectively, flexible enough to give the model strong heuristics.**

### 7.3 Three long-horizon strategies

**1. Compaction.** As the conversation nears the context limit, summarize it and restart a fresh window from the summary. Keep architectural decisions, unresolved bugs, and implementation details; discard redundant tool outputs and messages.

The tuning order is the useful part: **maximize recall first**, making sure the compaction prompt captures every relevant piece of information from the trajectory; **then iterate on precision**, eliminating superfluous content.

**2. Structured note-taking.** Have the agent periodically write notes to persistent storage outside the context window and pull them back later. The overhead is minimal and it sustains progress across complex tasks.

**3. Sub-agent architectures.** The main agent holds the high-level plan; subagents do deep technical work in clean contexts and return **condensed summaries, typically 1,000–2,000 tokens**. That is separation of concerns: a subagent can explore extensively without its exploration polluting the main context.

### 7.4 Working across context windows

The official playbook:

1. **Use a different prompt for the very first window** — set up the framework (write tests, create setup scripts) and use later windows to iterate on a todo list
2. **Have the model write tests in a structured format** (`tests.json`) and tell it: `It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **Set up quality-of-life scripts** (`init.sh`) that start servers, run tests, and run linters, so each fresh window does not repeat the work
4. **Consider starting fresh instead of compacting.** Counterintuitive but grounded: current models are extremely effective at discovering state from the local filesystem, and **in some cases** that beats compaction. (The official wording is "consider" and "in some cases," not always.) When you do, be prescriptive about the opening: `Call pwd; you can only read and write files in this directory.`, `Review progress.txt, tests.json, and the git logs.`, `Manually run through a fundamental integration test before moving on to implementing new features.`
5. **Provide verification tools** (browser automation, computer use). The longer and more autonomous the task, the more the model needs to verify correctness without human feedback
6. **Encourage complete use of the context** — tell it explicitly not to wrap up early out of budget anxiety

For state: structured data in JSON, progress notes in freeform text, versions and checkpoints in **git** — current models are especially effective at using git to track state across sessions.

### 7.5 Let the model orchestrate itself

This one is easy to miss because it lives at the harness layer rather than the prompt layer, but it may be the **highest-leverage item in this document**.

The common pattern is to route every tool result back into the context window as tokens, with filtering logic hardcoded into the tool.

**The better pattern:** give Claude code execution (bash, a language REPL) so it can write logic that expresses the tool calls and filters the output itself — **only the execution results reach its attention**.

The official figure: on BrowseComp web-browsing tasks, giving the model the ability to filter its own tool outputs raised accuracy from **45.3% to 61.6%**.

Two more in the same direction, both with numbers:

| Change | Effect |
|---|---|
| Let it manage its own context (progressive disclosure through skills, context editing, subagent isolation) | BrowseComp +2.8% |
| Let it persist its own context (compaction plus memory folders) | agentic search 60.4% → 67.2% |

> **Provenance:** these three figures come from Anthropic's experiments on an **earlier model generation** (the article names the specific models; they are omitted here as out of scope). The article's own argument is that the trend continues as models improve — **the stronger the model, the more decisions you should hand back to it** — so it holds more strongly on Opus 5 and Fable 5, not less.

There is a vivid comparison in the same piece: a newer model playing Pokémon writes organized tactical notes (ten files, tracking gym badges), while an earlier generation treats memory as a running log (31 near-duplicate files, stuck in the second town). **Same memory mechanism, very different results, purely as a function of model capability** — which is the other side of the same coin: scaffolding expires. As the model gets stronger, hand more of the decisions back.

### 7.6 Cache hits: a pure cost lever

**Cached tokens are 10% the cost of base input tokens.** That makes the ordering of your context a direct determinant of your bill. Five principles:

| Practice | Point |
|---|---|
| **Static first** | Stable content (system prompts, tool definitions) before dynamic material |
| **Append in messages rather than editing the prompt** | Inject reminders by appending to messages, not by going back and editing the system prompt |
| **Do not switch models mid-session** | Caches are model-specific |
| **Adding or removing tools invalidates the cache** | Use tool search for dynamic discovery instead of changing tool definitions |
| **Move breakpoints** | In multi-turn applications, move the cache breakpoint to the latest messages |

The otherwise arbitrary-looking rule in Section 6.3 — **do not change effort mid-session** — holds for exactly this reason.

### 7.7 When to promote a bash action to a dedicated tool

Bash has broad leverage but no observability: the harness cannot intercept, gate, or audit it. Four cases warrant a dedicated declarative tool:

1. **Security boundaries** — hard-to-reverse actions that need user confirmation (calling an external API)
2. **Write operations needing staleness checks** — preventing overwrites of files that changed underneath you
3. **User-facing presentation** — modal displays, multiple options
4. **Structured logging, tracing, and replay**

For low-stakes work there is another path: use a secondary Claude instance to judge whether a bash command is safe. That is what Claude Code's auto mode does, and it reduces the number of dedicated tools needed.

> This section looks like it contradicts 7.5 — one says build fewer tools and use general-purpose bash, the other says build dedicated tools. They are two faces of the same judgment: **get out of the way of the model's capability; draw a boundary where the product needs determinism.** If the model can handle it, do not constrain it with a tool. If the product must be able to intercept, audit, or render it, make it a tool.

### 7.8 Where tools exist, fix the tools first

The most valuable item in the new rules: **the expressiveness of a tool interface shapes behavior more than prompt text does.**

What a good tool looks like:

- **Low cardinality** — every tool has a distinct function from every other tool, so Claude can easily distinguish when to call each. The test is blunt: **if engineers cannot definitively identify which tool to use in a situation, agents will not either**
- **Semantically meaningful parameter names** — `user_id` rather than `user`; resolving arbitrary UUIDs to interpretable language significantly improves precision
- **Namespacing** by service and resource type (`asana_projects_search`, `asana_users_search`)
- **Token-efficient returns** — pagination, range selection, filtering, truncation, with sensible default caps (Claude Code's default is 25,000 tokens)
- **A `ResponseFormat` enum** (`concise` / `detailed`) — the official example shows 206 tokens versus 72, a 65% reduction
- **Actionable error messages** — say what to change and show a correctly formatted example rather than returning a stack trace
- **Tool descriptions written as onboarding for a new team member** — make implicit context explicit: specialized query formats, terminology, resource relationships

The Claude Code team described an instructive tradeoff. Thariq, on removing their own search tools:

> "we removed our grep and other search tools — glob tools — in favor of native bash."

The model already knows how to use standard command-line tools. But Cat explained why they **kept** a dedicated file-editing tool:

> "The reason we had a dedicated file edit tool was so that we could deterministically know that Claude was making a file change"

— because you need that determinism to render an "approve this change?" UI.

**The tradeoff is worth remembering:** a tool does not have to exist because the model needs it. It may exist because the product needs certainty.

> On the philosophy of tool design, Thariq put it well: `the models are kind of more of a biology than a physics, and tool design especially is quite hard...tool design is more of an art, maybe — or a biology.`

---

## 8. Claude Code: where to put what

The practical pain point for most people: the same sentence, placed in CLAUDE.md versus a skill versus a hook versus a subagent, has a completely different effect and a completely different cost.

### 8.1 The tradeoff table

| Mechanism | Loaded | Context cost | Best for |
|---|---|---|---|
| **CLAUDE.md (root)** | Session start, resident throughout | **High** | Build commands, directory layout, team conventions. Official wording: `Keep CLAUDE.md under 200 lines, give it an owner, and review changes to it like code.` |
| **CLAUDE.md (subdirectory)** | When that directory is touched | Low | Directory-specific conventions |
| **Rules** | Session start, or on file access | Medium | Specific constraints. **Add `paths:` frontmatter** to scope them and cut the cost sharply |
| **Skills** | Name and description resident, body on demand | Low | Procedural workflows: deploy checklists, release processes, review processes |
| **Subagents** | Isolated context; only the conclusion returns | Low | Side tasks where you want only the conclusion: log analysis, dependency audits |
| **Hooks** | Lifecycle events; bypass compaction | Low | **Deterministic** automation: linters, notifications, blocking commands |
| **Output styles** | Session start, never compacted | **High** | Only when Claude's role needs to change substantially |

### 8.2 Anti-patterns

- ✗ "Every time X, always do Y" in CLAUDE.md → use a hook. Official wording: **a real guardrail needs to be deterministic**
- ✗ "Never do this" constraints → use a `PreToolUse` hook or permission configuration
- ✗ A 30-line procedure in CLAUDE.md → move it to a skill
- ✗ An API-specific rule with no path scoping → add `paths:` frontmatter
- ✗ Personal preferences in a shared project file → use the user-level local equivalent

### 8.3 The CLAUDE.md self-test

For every line, ask: **would removing this cause Claude to make mistakes?** If not, cut it.

**What belongs:** bash commands Claude cannot guess, code style that differs from defaults, testing instructions, repository etiquette (branch naming, PR conventions), project-specific architectural decisions, environment quirks, non-obvious gotchas.

**What does not:** anything readable from the code, standard language conventions, detailed API documentation (link it), frequently changing information, file-by-file descriptions, platitudes like "write clean code."

**The diagnostic signal that matters most:**

> When a rule keeps being ignored despite repetition, the problem is usually **not that the wording is insufficiently emphatic — it is that the file is too long and the rule is buried.**

The official phrasing is blunt: bloated CLAUDE.md files cause Claude to ignore your actual instructions. Relatedly, if Claude keeps asking about something CLAUDE.md already covers, the line covering it is ambiguous.

**Treat CLAUDE.md as code:** review it when something goes wrong, prune it periodically, and check whether Claude's behavior actually changed after you edited it.

**There is a tool for this:** `/doctor` in Claude Code helps rightsize skills and CLAUDE.md. It shipped alongside the July 2026 new-rules post, for precisely this job of cutting legacy scaffolding.

### 8.3b Writing skills: three-tier progressive disclosure

The entire architecture of a skill exists to do one thing — **put each layer of information into context only when it is needed:**

| Tier | Content | Loaded |
|---|---|---|
| **1. Metadata** | YAML frontmatter, only `name` and `description` | Preloaded into the system prompt at startup |
| **2. Primary instructions** | The `SKILL.md` body | When Claude judges the skill relevant |
| **3. Bundled files** | Files referenced from SKILL.md | Only when that specific scenario arises |

The official analogy is apt: **like a well-organized manual — table of contents first, then the chapter, then the detailed appendix.**

Key points:

- **`name` and `description` are the two critical fields.** They are what Claude uses to decide whether to activate the skill. If it should have triggered and did not, fix those two lines rather than editing the body
- The intended shape is a **lightweight guide that helps Claude find information when it needs it**, not an exhaustive procedure. **Unless the domain is unusually critical, do not over-constrain**
- When the body grows unwieldy, **split the file**. Separating mutually exclusive scenarios saves real tokens. The official example: the PDF skill puts form-filling instructions in a separate `forms.md`, keeping the body lean and **trusting Claude to read it only when a form actually needs filling**
- Be explicit about whether code is for Claude to **execute** or to **read as reference**
- **Bundle executable code where it helps.** The reason is concrete: sorting a list via token generation is far more expensive than simply running a sorting algorithm. Give deterministic operations to code
- **Start from evaluations** — measure the capability gap first, rather than imagining requirements in advance
- **Let Claude help write it.** When a skill fails to trigger, have it reflect on what context it actually needed; that beats guessing
- **Security:** install skills only from trusted sources. If the source is uncertain, audit the bundled files, the dependencies, and any instruction that reaches the network

Combine this with Section 6.2: an over-prescriptive skill written for an older model **can actively degrade** Fable 5's output. More detail is not automatically better. Past a point it is worse.

### 8.3c Turning manual checks into verification loops

The test is simple: **anything you keep having to enforce by hand is a candidate.** Especially deterministic rules a generic linter will not catch — the official example is rejecting any migration that drops a column without a backfill step.

Four deployment modes:

| Mode | Trigger | Best for |
|---|---|---|
| **Standalone** | Invoked deliberately | Cross-cutting checks that do not apply universally: security scans, accessibility audits, license verification |
| **Embedded** | Fires automatically inside the producing skill | Checks that belong to one specific task |
| **Chained** | One skill calls another on completion | End-to-end handoff; also the only way to add verification to a **skill you cannot modify** |
| **On every PR** | CI | Turns a personal habit into team infrastructure, removing dependence on author diligence |

The trap: **embedding only works on editable skills.** Built-in or plugin-managed skills update automatically and would overwrite your change; those must use chaining.

The creation flow: note the manual review you perform most often → write the steps as you would for someone on their first day → try the built-in `/verify` → if that is not enough, use `/skill-creator` or hand-write it → confirm on a real task that it actually ran → then try chaining.

### 8.3d Four kinds of loops

| Type | Command | Best for |
|---|---|---|
| **Turn-based** | Manual prompting | Short tasks |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | Tasks with a **verifiable exit criterion** where one turn is not enough |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | Recurring work, interfacing with external systems |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | Well-defined recurring streams: bug reports, issue triage, migrations |

Cost control: **use scripts for deterministic work** (running a script costs less than reasoning through it token by token), match intervals to how fast the underlying state actually changes, and pilot before running at scale. The official conclusion: **model and effort selection are among the biggest levers on loop cost.**

### 8.4 The single most valuable practice in Claude Code

The official best practices put one thing first: **give Claude a check it can run itself.**

> Claude stops when the work looks done. Without a check it can run, "looks done" is the only signal available, and **you become the verification loop** — every mistake waits for you to notice it. Give it something that produces a pass or fail, and the loop closes on its own.

The check can be a test suite, a build exit code, a linter, a script that diffs output against a fixture, or a browser screenshot compared against a design.

The contrasts:

| Weak | Strong |
|---|---|
| `implement a function that validates email addresses` | `write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing` |
| `make the dashboard look better` | `[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them` |
| `the build is failing` | `the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error` |

Once the check exists, there is a second question the prompt cannot answer by itself: **how hard does it gate the stop?** The official ladder runs from weakest to strongest — ask for the check in the prompt; set it as a `/goal` condition so a separate evaluator re-checks after every turn; make it a `Stop` hook that blocks the turn from ending until it passes (overridden after 8 consecutive blocks); or hand it to a verification subagent so the agent doing the work is not the one grading it. Each step trades setup for attention. The prompt version works today on any task; the `/goal` and hook versions are what let an unattended run finish correctly without you.

**What if the check does not exist yet?** That is the most common case — legacy code, low coverage, nobody willing to touch it. Do not skip verification and start editing. **Make building the check the first phase:** write **characterization tests** that pin current behavior, **asserting suspected bugs exactly as they currently behave** (this has to be stated explicitly, because a well-intentioned reader told to pin current behavior will very naturally fix the bugs they find, which destroys the net). Freeze the net once it exists — the refactor phase must not modify these tests. Only then refactor. Fixing the bugs belongs in a **separate later round**; mix the two and a red test leaves you unable to tell whether the refactor broke something or the behavior simply changed.

Another practical technique — **have Claude interview you:**

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask obvious questions, dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Then execute that spec in a **fresh session**, where the context is clean and entirely focused on implementation. The official framing: **time spent making the spec precise pays back more than time spent watching the implementation.**

### 8.5 Five common failure patterns

| Pattern | Symptom | Fix |
|---|---|---|
| **The kitchen sink session** | One task, then something unrelated, then back again | `/clear` between unrelated tasks |
| **Correcting over and over** | Corrected once, still wrong, corrected again | **After two failed corrections, `/clear`** and write a better initial prompt incorporating what you learned |
| **The over-specified CLAUDE.md** | Too long; important rules lost in the noise | Prune ruthlessly; delete instructions for things the model already gets right |
| **The trust-then-verify gap** | Plausible-looking implementation that misses edge cases | Always provide verification. **If you cannot verify it, do not ship it** |
| **The infinite exploration** | "Investigate this" without scope; hundreds of files read | Narrow the scope, or isolate it in a subagent |

**One important caveat on adversarial review.** Having a subagent review the diff in a fresh context is effective, but the official guidance also warns that **a reviewer prompted to find gaps will usually report some, even when the work is sound, because that is what it was asked to do.** Chasing every finding leads to over-engineering. The review prompt must therefore say: **flag only gaps that affect correctness or the stated requirements, and treat the rest as optional.** This is a filtering instruction, so it belongs only where **you** are the filter. If the findings feed a downstream ranking or triage step instead, invert it and have the reviewer report everything (the "report everything; filter downstream" row in Section 6) — Opus 5 applies a stated severity bar literally and silently drops whatever falls below it.

---

## 9. How to practice

### 9.1 A four-week path

**Week 1 — detox.** Keep the Section 4 table in front of you. Go through the prompts you already have and delete what is obsolete, line by line. This step usually yields the biggest return, because most people's prompts carry a stack of legacy scaffolding that actively hurts on current models.

**Week 2 — practice being clear.** Run every prompt you write through two tests:

1. Would someone with no context on this task be able to follow it?
2. **How would a well-intentioned human misinterpret it?** Cat Wu's formulation: `you should think about the ways in which it could be misinterpreted by a well-intentioned human, in order to better understand how the model might interpret it`

**Week 3 — practice subtraction.** Take the longest prompt you have written, cut it in half, and run the same task both ways. You will usually find the quality did not drop. This is how you build the intuition for scaffolding becoming dead weight.

**Week 4 — practice solving it one layer out.** Every time you want to add a rule, ask first whether the problem can be solved without a prompt. A different tool design? A deterministic check? A change in how the information is presented?

### 9.2 Six questions for judging a prompt

1. Is there a decidable success criterion?
2. Does every constraint carry its reason?
3. Is there any sentence whose removal would change nothing? (Then remove it.)
4. How would a well-intentioned human misinterpret it?
5. Does this really need to load **every time**? (If not, push it down into a skill or on-demand file.)
6. Should this rule live in the prompt at all, or should a tool, schema, or hook enforce it deterministically?

### 9.3 The one durable habit

> **Every time the model gets an upgrade, go back and delete part of your prompt.**

Lance Martin's version: as Claude improves, the assumptions in a harness expire, and scaffolding that was once necessary becomes dead weight and a bottleneck. So **re-evaluate continuously.**

That is probably the only advice here that will not go out of date — because what it describes is how to handle things going out of date.

---

## 10. Sources and a reading order

The full source list, the read depth for each, and the adjudication of known conflicts are in [SOURCES.md](../SOURCES.md). What follows is only a suggested order.

1. **Start here:** [The new rules of context engineering for Claude 5 generation models](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) (2026-07-24) — ten minutes, and it orients everything else
2. **Then:** [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (living doc) — the densest practical material available; if you use Claude Code, this is the highest-return page
3. **Consult as needed:** [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) plus the page for the model you use — reference material, not a read-through
4. **For the underlying principles:** [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (2025-09)
5. **To hear it from the people who did it:** [Fireside chat: Cat Wu and Thariq Shihipar](https://www.youtube.com/watch?v=uU5Gv2h8-9g) (2026-07), with a [transcript](https://simonwillison.net/2026/Jul/21/cat-and-thariq/)
6. **Read with care:** [Prompt engineering best practices for 2026](https://claude.com/blog/best-practices-for-prompt-engineering) (2025-11). Its framing sections and its "common mistakes" list are still worth reading, and Section 3.4 borrows a useful corollary from it. But it **contradicts the current documentation in three places** — it recommends prefill (now rejected by the API) and calls XML tags and role prompting unnecessary (both still recommended). On those three, the docs win

### ⚠️ Deliberately excluded

| Excluded | Why |
|---|---|
| [prompt-eng-interactive-tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) | Still targets **Claude 3 Haiku**, and the techniques it teaches most prominently have since been reversed. **In fairness:** the official overview page still lists it for interactive learners and does not deprecate it — but the same page designates Prompting best practices as "the living reference; start there." Fine for concepts; defer to the living reference on every specific technique |
| ["AI prompt engineering: a deep dive"](https://www.youtube.com/watch?v=T9aRN5JkmL8) (2024-12) | The most widely shared panel on the subject. The mental models hold up; the tactics do not |
| Third-party "ultimate guide to prompting Claude" articles | Secondhand summaries |

---

## Appendix: the verification record

This section stays because it is itself a piece of methodology: **with fast-moving documentation, "verified" is not a state, it is a repeated action.**

### Content verification

After the first draft, an independent adversarial fact-check compared every English snippet, every figure, and every model attribution against the source text. It changed, among other things:

- the scope of the claim that forceful wording causes over-triggering, adding the CLAUDE.md exception
- the over-generalization that official snippets almost all use XML
- the speaker attribution on the grep/glob passage (Thariq, not Cat)
- Simon Willison's remark, restored to the strength of the original
- the "fresh window beats compaction" claim, restoring the official hedges
- the `max_tokens` 64k recommendation, scoped to `xhigh`/`max`

### Completeness verification

Later rounds asked whether each page had actually been read in full, and closed these gaps:

- **The Fable 5 page had never been fetched**, despite being cited in the model comparison. Reading it in full produced the entire Fable 5 section, along with the `reasoning_extraction` refusal category, the finding that skills written for prior models are often too prescriptive and can degrade output quality, and progress-truthfulness auditing — none of which were present before
- **The effort page had only been skimmed.** Reading it in full added that lower effort reduces tool-call count, that `high` equals omitting the parameter, the per-model differences, and the `ultracode` clarification
- **The overview page had not been read at all**, which is where Section 3.0's preconditions come from, along with "not every failing eval is best solved by prompt engineering"
- **Section 5b is new.** The apparent contradiction between Opus 5's "remove verification" and Fable 5's "add verification" was not addressed in the original draft at all
- **Four sources had only thin summaries**: the Agent Skills engineering post (never fetched), Agent Harness Design, verification loops, and loop engineering. Reading them produced the three-tier progressive disclosure model, the five cache-hit principles, the three orchestration figures, the four verification-loop deployment modes, and the loop commands
- **The Console prompt improver link was dead** (it now 302-redirects), and the page it redirects to still advertises **prefill, which the API now rejects** — a trap someone would walk straight into
- Scope was then trimmed: guidance specific to earlier models was removed rather than left in as "probably still fine"

### A real lesson: the documentation changed mid-verification

Worth recording in full, because it makes the point better than any of the entries above.

Section 6.3's Opus 5 effort starting point originally said coding and agentic work should start at `xhigh`, on the basis of this wording on the official Opus 5 page: `for coding and agentic work, xhigh remains the recommended starting point`.

During review, two independent passes reported the page said "start at `high`" instead. Three direct fetches of the same page all returned the `xhigh` sentence, and the effort page had no Opus 5 section at the time. The two reports were recorded as unreproducible.

**They were not wrong. The page had changed.** It now reads:

> "**Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency of higher settings. Start with the default (`high`) and adjust based on your evals: use `low` and `medium` liberally as your primary control for token cost and response time wherever quality holds, and step up to `xhigh` for demanding coding and agentic work."

The phrase `remains the recommended starting point` no longer appears anywhere on the page, and the effort reference has since gained per-model recommendation sections for both Opus 5 and Fable 5. This document and the skill have both been corrected.

**Two takeaways:**

1. **The difference is material,** not cosmetic. An ordinary coding task moved from `xhigh` to `high` as its starting point, which changes what it costs.
2. **For fast-moving documentation, a snapshot is wrong by default, not by exception.** For anything you are using to plan cost or latency, do not rely on a secondhand summary — including this one. Confirm it against the source page, which is the only surface that tracks model updates.

---

*Every technical claim here traces to an official source listed in [SOURCES.md](../SOURCES.md). Where this document and something you read elsewhere disagree, check the publication or revision date of the official page first. Corrections are welcome as issues — the live docs are the authority, not this repository.*
