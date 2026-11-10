# Prompt engineering in 2026: where these rules come from

> Background reading for the `prompt-engineering` skill. The skill tells you **what to do**; this document explains **why**: the official basis for each rule, which older habit it overturns, and how to read the few places where different pages say different things.
>
> Snapshot **2026-11-09**. Several kinds of correction have gone into it: a source page that changed under the snapshot (the effort guidance); an exact-string verification pass that caught two snippet attributions this document had got wrong on its own (the error was ours, not an official change); a page that was withdrawn entirely, taking one rule's only source with it; three model releases (Claude Fable 5.1 on September 1, 2026; Claude Opus 5.5 on September 22, 2026; Claude Fable 5.2 and Mythos 5.2 on November 9, 2026) that re-opened every model-specific claim for adjudication, one claim at a time; the 2026-10-23 round, after a month in which the official pages themselves had been edited a good deal, mostly unannounced; and the 2026-11-09 round, which removed, at the repository owner's request, everything with no official source, at a time when the Claude Code best practices page had also been rewritten from top to bottom. The appendix records them. Everything here comes from Anthropic's official documentation, official articles and support site. One public remark by an employee is kept, and it is labeled wherever it is used. The skill marks three further practices as user rules, not official guidance, and this document marks them the same way where it mentions them. Guidance specific to models before the Claude 5 generation has been removed rather than left in on the assumption that it probably still works. Sources are itemized in [SOURCES.md](../SOURCES.md).
>
> **How to read it.** Sections 1, 2 and 3 build the frame. Section 4 is the most important one, the unlearning chapter: if you learned prompt engineering before 2026, it will overturn part of what you know. Sections 5 and 5b each cover one point: whether to give examples, and how to write verification instructions for each model. Section 6 covers the four current models, and how to choose a model and an effort level. Section 7 onward is reference material.
>
> **Scope:** the four current official models, **Claude Fable 5.2** (including Mythos 5.2), **Claude Opus 5.5**, **Claude Sonnet 5.5** and **Claude Haiku 5.5**, plus general techniques verified as current and non-conflicting. Claude Fable 5.1, Fable 5, Opus 5 and Sonnet 5 are all on the official "Legacy models (still available)" list, and this document mentions them only where one is the source of some wording or of a measurement, the starting point of a migration, or a comparison that a current page makes (a price or a benchmark). Each model-specific claim falls into one of three groups: what the current page states is used directly; what only a predecessor's page states, and the current page neither restates nor retracts, is labeled "measured on that model"; what has been retracted, or conflicts with the current page, is left out. The start of Section 6 explains why it is split this way. Guidance specific to earlier models has been removed.
>
> **One disclaimer.** For specific API parameters, the live documentation is authoritative. The appendix records a real lesson on this point: a passage of official wording quoted in this document was rewritten by Anthropic *during* the verification process.

---

## Contents

1. What actually changed
2. Three layers: prompt → context → harness
3. Foundations: six things that still hold
4. **Obsolete: eight habits to unlearn**
5. Whether to give examples at all
5b. Verification instructions: the official pages differ by model
6. The four current models, and how to choose a model and effort
7. Agents and long tasks: context engineering
8. Claude Code: where to put what
9. How to practice
10. Sources and a reading order

Appendix: the verification record

---

## 1. What actually changed

**A large share of what prompt engineering has taught over the past two years is now being reversed by Anthropic itself.**

The single strongest piece of evidence: a 2026-07-24 post from the Claude Code team reports that they removed **over 80% of Claude Code's system prompt** for models like Opus 5 and Fable 5, **with no measurable loss on their coding evaluations**.

That is not an isolated data point. Two further threads point the same direction:

- As Claude improves, scaffolding that was once necessary in a harness becomes dead weight, which can hold back the model's performance (Lance Martin, Claude Platform team, in a 2026-04 article)
- The Claude Fable 5.2 prompting page (2026-11) states that some instructions written for earlier models `raised cost, caused side effects, or no longer had an effect` on it

One sentence for the direction of travel:

> **From carefully constructing instructions to carefully constructing context and tools.**

Why? Because the model's judgment improved. You used to write `Never write multi-paragraph docstrings or multi-line comment blocks — one short line max` because the model could not make that call well itself. Now you write `Write code that reads like the surrounding code: match its comment density, naming, and idiom` and it can work it out. (These two lines are verbatim from the old and new versions of the Claude Code system prompt, as quoted in that post.)

The post itself names the cost of the first form. For a subset of requests, rules like that were simply wrong: on documentation, the user may have their own preferences, and particularly complex code may really need multi-line comment blocks. Reading transcripts of their own usage, they saw the system prompt, skills and the user's request clash within a single request, with "leave documentation as appropriate" and "DO NOT add comments" both present, and Claude had to think one layer further through these overlapping, conflicting messages before it could decide what to do.

The Fable 5.2 page shows a more concrete side: rules get followed literally, in its words `the model applies them as written`. Anthropic tested this on simulated tasks with no one available to approve anything. A rule that listed `sending messages to people outside this conversation` among the steps that need approval made the model hold back roughly half of its routine requests to colleagues and other teams. However wide a rule is written, that is how wide it gets applied.

---

## 2. Three layers: prompt → context → harness

To understand the 2026 picture you need to tell three layers apart.

### 2.1 Prompt engineering

Writing and organizing instructions to an LLM for optimal results. **A one-time act of writing.** It fits single-turn work with a well-defined task.

### 2.2 Context engineering

The full set of strategies for curating and maintaining the optimal set of tokens during LLM inference, including all the information that falls outside the prompt itself.

The distinction that matters: **prompt engineering is writing done once; context engineering is iterative: every turn, you decide again what goes into the model.**

System instructions, tools, external data, message history, memory: all of it counts. In multi-turn agent work this layer matters far more than the prompt layer does.

The governing concept is **context rot**: the model has a finite attention budget, and every token spends a little of it. Attention in a transformer is quadratic, so the longer the context, the more diluted the attention. The goal is therefore not to tell it everything you know, but to find:

> **the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome.**

### 2.3 Harness and loop engineering

The software infrastructure around the model: the loop, the tools, context management, guardrails. Lance Martin's central observation is this:

As Claude improves, the assumptions in a harness **expire**, and scaffolding that was once necessary becomes dead weight. With each step up in model capability, go back and ask: **what can I stop doing?**

**The practical consequence:** when you catch yourself writing a long list of rules into a prompt, stop and ask whether the problem belongs one layer out. A different tool design? A deterministic check? Or does it need handling at all?

---

## 3. Foundations: six things that still hold

### 3.0 First, the preconditions

The official prompt engineering overview opens with something many people skip. **Before you tune a prompt you should already have:**

1. clear success criteria,
2. a way to test empirically against them,
3. a first draft.

Without the first two, you are tuning blind. (You don't have to write the draft yourself from scratch: the official [metaprompt recipe](https://colab.research.google.com/github/anthropics/claude-cookbooks/blob/main/misc/metaprompt.ipynb) will generate one for you.)

One more line from the same page carries as much weight: **not every success criterion or failing eval is best solved by prompt engineering.** The example given is latency and cost, which can sometimes be improved more easily by choosing a different model.

That framing matters: **the prompt is one tool in the box, not the only hammer.** Sections 6 and 8 keep returning to the same pattern: some problems belong at the level of the model and effort choice, a hook, or permission settings, not in one more sentence in the prompt. On things that must happen or must never happen, the Claude Code best practices put it as `A prompt alone can't guarantee either of those`.

---

The new rules overturn *over-constraining*, not *being clear*. Everything below is still taught in the official documentation, in its current version.

### 3.1 Be clear and direct

Treat Claude as a **brilliant but new employee**: capable, but with no knowledge of your conventions or workflows.

**The key test:** show the prompt to a colleague with minimal context on the task and ask them to follow it. If they would be confused, so will Claude be.

If you want behavior beyond the basics, say so. Do not expect the model to infer it from a vague brief.

- ✗ `Create an analytics dashboard`
- ✓ `Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.`

### 3.2 Give context, and give the reason

Explaining *why* you ask for something lets the model generalize from the explanation. The Fable 5.2 page points the same way for rules that have built up: `rewrite absolute rules so that they state their scope and reason`.

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

The second version still contains "never". The difference is that it hangs on a **reason**. Once the model knows why, it can decide for itself in the edge cases the rule never covered. That is the difference.

### 3.3 Structure with XML tags

> ⚠️ A November 2025 official blog post says XML tags are "less necessary with modern models". **The current documentation contradicts this.** For the adjudication, see reading-list entry 6 in Section 10, and "Known conflicts" in [SOURCES.md](../SOURCES.md).

XML tags reduce misreading, especially when a prompt mixes instructions, background, examples and variable inputs. Official wording:

> "XML tags help Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs. Wrapping each type of content in its own tag (for example, `<instructions>`, `<context>`, `<input>`) reduces misinterpretation."

Use consistent, descriptive tag names, and nest them where the content has a hierarchy.

Markdown headers (`## Tool guidance`, `## Output description`) are an equally sanctioned way to partition a prompt, and they mix freely with XML. That comes from Anthropic's context engineering article.

### 3.4 Give it a role

Setting a role in the system prompt focuses behavior and tone. **Even a single sentence has an effect:**

```
You are a helpful coding assistant specializing in Python.
```

> ⚠️ The same 2025-11 post says `heavy-handed role prompting is often unnecessary`. The current documentation still recommends role prompting; again, see Section 10 for the adjudication. The post does contribute one useful corollary, though: **stating what perspective to focus on is often more effective than attaching an identity label.** The post's own example is `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential` rather than assigning a role. The two are compatible and stack.

### 3.5 Long-context placement

For long documents of 20k+ tokens:

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

Analyze the annual report...   <!-- the question goes here, last -->
```

There is also **grounding in quotes**: have the model first extract the relevant passages from the long document into `<quotes>` tags, then answer from those. It keeps attention on the relevant material and lets the rest fall away.

### 3.6 Say what to do, not what to avoid

This runs through every official source.

- ✗ `Do not use markdown in your response`
- ✓ `Your response should be composed of smoothly flowing prose paragraphs.`

A neat supplementary lever: **match the style of your prompt to the style you want out of it.** For example, removing markdown from the prompt can reduce the markdown you get back.

The Opus 5 page says the same in the specific context of narration style: `Positive examples of the communication style you want tend to be more effective than instructions about what not to do.`

---

## 4. Obsolete: eight habits to unlearn

This section is the main reason the document exists. Each entry names where the old advice came from and what current basis supersedes it, so you can check it yourself.

### 4.1 ❌ Prefill (prefilling the assistant's reply) — rejected by the API

**The old advice:** prefill the assistant turn with `{` to force JSON, or with `Here is the requested summary:` to strip the preamble. The November 2025 official post still lists this in its "Advanced prompt engineering techniques" section.

**Now:** **none of the four current models supports it.** Official wording:

> "Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported. Requests with prefilled assistant messages to these models return a 400 error."

The Haiku 5.5 migration guide adds that Haiku 4.5 still accepted prefill with thinking turned off, while Haiku 5.5 returns 400 even with thinking turned off.

**Why it went away:** the stated reason is that the model's intelligence and instruction following have improved to the point where most prefill use cases no longer need it.

**Replacements:**

| Old use | What to do now |
|---|---|
| Force JSON or a fixed structure | The **Structured Outputs** feature, or state the schema directly (newer models can reliably match complex schemas when you ask them to, especially with retries) |
| Classification | A tool with an enum field, or structured outputs |
| Strip the preamble | `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.` |
| Route around a mistaken refusal | Officially, Claude's refusal judgment is far more accurate now; state the context plainly in the user message |
| Continue a truncated response | Move the prior text into the user message and have it continue, or, in error handling where there is no UX penalty, simply retry |

### 4.2 ❌ `budget_tokens` to control the thinking budget — removed

**The old form:** `thinking: {type: "enabled", budget_tokens: 10000}`

**Now:** none of the four current models accepts manual extended thinking; setting `budget_tokens` returns 400.

**The new form:**

```python
thinking={"type": "adaptive"},       # or omit it: the two are equivalent
output_config={"effort": "medium"},  # API default on Opus 5.5 and Haiku 5.5; "high" on Fable 5.2 and Sonnet 5.5
```

**Whether you can turn thinking off differs across the four models.** On Fable 5.2 and Opus 5.5 thinking is always on, and `{"type": "disabled"}` and `{"type": "between_tools"}` both return 400. On Sonnet 5.5, `disabled` returns 400 at every level; to turn off up-front thinking, send `between_tools`, which works only at `high` and below. On Haiku 5.5, `disabled` works at `high` and below and returns 400 at `xhigh`/`max`.

**What adaptive thinking is:** the model decides when to think and how much, based on two things: the `effort` parameter and the complexity of the query. Simple questions get answered directly and hard ones get more thought, except that Fable 5.2 and Mythos 5.2 think before almost every reply at `high` and above. Anthropic's internal evaluations show **adaptive thinking reliably outperforming manual extended thinking**.

**The main depth control is `effort`, not wording in the prompt.** This is stated for all four models. The Fable 5.2 page says that sentences like `answer directly without deliberating` do not stop it thinking at `medium` and above, and its fix is `Remove these lines and lower the effort level`. The Sonnet 5.5 page says that telling it in the system prompt to think less `doesn't reliably reduce its thinking`. The Haiku 5.5 page says that telling it to answer directly did not stop it thinking. The Opus 5.5 page says that lowering effort is more reliable than prompt instructions. This is the basis of skill 3.10.

For a hard ceiling, use `max_tokens`, noting that it is the **combined limit on thinking plus response text**.

### 4.3 ❌ Forceful wording to prevent undertriggering — now reversed

**The old form:** `CRITICAL: You MUST use this tool when...`

**Why it was written that way:** early models were not eager enough to reach for tools, so people applied pressure.

**Now:** the prompting best practices page says Claude Opus 4.5 and Opus 4.6 are more responsive to the system prompt, so prompts written to prevent undertriggering may overtrigger on them, and the fix is to soften aggressive wording: `Use this tool when...` is enough. For Opus 4.6 the same page also replaces `Default to using [tool]` with `Use [tool] when it would enhance your understanding of the problem`, and says `If in doubt, use [tool]` causes overtriggering. These lines name Opus 4.5 and 4.6, and the pages for the four current models do not restate them, so first check whether your prompt still needs the old form at all.

The Claude Code team's own system prompt went the same way: over 80% removed for models like Opus 5 and Fable 5, with no measurable loss on their coding evaluations.

**Emphasis in CLAUDE.md is no longer recommended either.** Earlier versions of this document kept an exception here: the Claude Code best practices used to allow "IMPORTANT" on the one line Claude kept skipping. The rewritten page reverses that: when Claude keeps ignoring a line, `adding IMPORTANT probably won't fix it`. Either the file has grown long enough that the line is buried, and the file should be shorter, or the line is a rule that cannot have exceptions, and it should be a hook. The same page's "What you can stop doing" table lists adding IMPORTANT or all caps to instructions that keep being missed among the old practices you can stop, with this reason:

> `Current ones follow a plainly worded line, and piling on emphasis mostly makes Claude over-apply the rule.`

**The new form:** one plain statement. If you actually observe a tool undertriggering, add one targeted sentence about when it should be used.

### 4.4 ⚠️ One verification rule for every model — the pages actually say different things

**The old advice:** the general documentation still carries `Before you finish, verify your answer against [test criteria]`. Earlier versions of this document went the other way: they made the Opus 5 page's instruction to delete verification instructions a headline rule, and in Section 5b reconciled it with the Fable 5 page's periodic self-checks into one general pattern.

**Now:** the best practices page still treats self-checking as general advice, and Opus 5 is the only exception it names, in the words `remove these instructions rather than rewriting them`. That sentence names Opus 5, which is now a legacy model. The Opus 5.5 pages neither restate nor retract anything about over-verification. The Claude Code docs say reminders to test or check are usually unnecessary on Fable models. The Sonnet 5.5 and Haiku 5.5 pages go the other way: each gives a verification paragraph to add to the system prompt if you see the model report work done at low levels without running checks. Four current models, four different positions, so skill 3.4 lists each page's wording model by model instead of merging them into one rule. Section 5b has the point-by-point comparison.

This is the clearest case in the whole document of why **changing a model name is not the same as completing a model upgrade**.

### 4.5 ❌ Stacking absolute prohibitions

**The old habit:** writing every boundary as a hard rule with `NEVER` and `always`, for fear the model would overstep.

**Now:** the best practices page's principle is `Tell Claude what to do instead of what not to do`, and to give the reason (Sections 3.2 and 3.6). The Section 1 post explains why this is possible: the old absolute rules were simply wrong for a subset of requests, and newer models `have better judgement and can handle these decisions well without explicit rules`. The Claude Code best practices say the same thing from another direction: `Almost every guideline has exceptions`, and with the actual code in front of it, Claude usually judges better than a rule you wrote in advance.

The Fable 5.2 page gives the most concrete measurement on this. The model follows a rule as written, and an exception added on a separate line does not override a hard rule. In Anthropic's simulated tasks, the person in charge had already approved a step. Under a "never" rule, the model took that step in about one try in ten. A separate line saying that approved steps can go ahead barely changed that rate. With the exception written into the rule itself (the rule saying to get approval first), it took the step in about nine tries in ten. So its fix is to rewrite absolute rules so that they state their scope and reason, and to write any needed exception into the rule itself.

What truly cannot have exceptions belongs to hooks and deny rules; see Section 8.2.

### 4.6 ❌ Repeating an instruction in several places

**The old habit:** afraid the model will miss it, you say it in the system prompt, again in the tool description, and again in the docs.

**Now:** the Section 1 post says earlier models sometimes needed repeated instructions, or were more likely to follow instructions at the end of the context than at the start. The team has since deleted the repeats: guidance on how to use a tool stays **in the tool description only**, and is not written again in the system prompt.

**What does not count as repetition:** when to use the tool. Now that forced `tool_choice` is rejected on three of the current models (see the API-level changes at the end of this section), the official replacement is to state in the prompt when to use the tool. So skill 3.7 lists the trigger condition separately, and it does not count as repetition.

### 4.7 ❌ Hand-maintaining CLAUDE.md as memory

**The old practice:** using the `#` hotkey to write notes into CLAUDE.md.

**Now:** Claude Code now saves memories relevant to the work and to you automatically, so that part no longer needs maintaining by hand (same post).

(To be clear, CLAUDE.md itself is not obsolete. It is still the place for what Claude cannot get by reading the code: the commands that confirm work, the places where the code is misleading, decisions that are not in the repo. What is obsolete is using it as a notepad kept by hand. See Section 8.)

### 4.8 ❌ Using a simplified Markdown write-up as the spec

**The old practice:** write a simplified Markdown description and hand that to the model *in place of* the real material.

**Now:** supply **rich references**: the actual code, a detailed test suite, the function in another codebase that is to be ported, an HTML artifact. The post's reason is that code files give Claude clear, high-fidelity instructions in a language it knows very well, and that an HTML mockup of a design usually works better than a written description of the design or a screenshot. The Claude Code best practices say the same: if you already have what it needs, hand it over instead of describing it.

To be clear about what this does *not* deprecate: a spec the user has read and confirmed is a different thing. Section 8.5 is about producing exactly that, and it works because it functions as an **agreement**, not as a lossy stand-in for the code.

### Additional: a few API-level changes

- **Non-default sampling parameters (`temperature` / `top_p` / `top_k`) return 400 on all four current models.** The models the thinking page lists include these four, and its wording is that such values `return a 400 error on every request, regardless of whether thinking is used`; the Fable 5.2 overview says `Omit temperature, top_p, and top_k`. Two traps sit beside it: `top_p` defaults to `0.99`, so `top_p: 1` is rejected, as is sending `temperature` and `top_p` together; and from API version `2026-10-21`, any of the three at any value returns 400 on every model. Stop relying on sampling parameters for variety and use the prompt instead. For design work, the Sonnet 5 page's recommended approach is to have the model propose several distinct directions first, **let the user pick one**, and then build it (the Sonnet 5.5 page states that the Sonnet 5 patterns remain a reasonable starting point).
- **Forced tool use is gone on three of the current models; Haiku 5.5 is the exception.** `tool_choice` of type `any` or `tool` returns 400 on Fable 5.2, Mythos 5.2, Opus 5.5 and Sonnet 5.5. Haiku 5.5 accepts it, but the reply then starts directly with the tool call. (Earlier versions of this document quoted the reason the Fable 5.1 what's-new page gave. That page now redirects to the Fable 5.1 overview, and the current pages state only the error, with no reason, so the reason is no longer quoted.) Keep `auto`, use strict tool use or structured outputs for schema-valid JSON (in organizations that use customer-managed encryption keys (CMEK), neither is available on Fable 5.2 or Mythos 5.2, so validate the output in your own code), and say in the prompt when the tool should be used. For code coming from Opus 5, Fable 5 or Sonnet 5, this is a breaking change.
- **Coming from a model before Claude 4.7, the same text counts as more tokens.** The Haiku 5.5 migration guide says about 30% more than on Haiku 4.5, and the Opus 5.5 migration guide says up to about 35% more than on models before Opus 4.7. Re-check inherited `max_tokens` values and cost estimates.

### A table of hard rules by model

Before the 2026-11-09 round, this document covered only Opus 5.5 and Fable 5.1, which were identical on these points, so one sentence, "both models return 400", was enough. Once Sonnet 5.5 and Haiku 5.5 came into scope, that sentence stopped holding in three places, so skill Section 3 became a per-model table:

| | Fable 5.2 / Mythos 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Prefill | 400 | 400 | 400 | 400 |
| `budget_tokens` | 400 | 400 | 400 | 400 |
| Turning thinking off | 400 | 400 | 400; use `between_tools` instead, `high` and below only | Works at `high` and below; `xhigh`/`max` return 400 |
| Non-default sampling parameters | 400 | 400 | 400 | 400 |
| `reasoning_extraction` refusal | Yes (the official pages name Fable 5.2) | Yes | Yes | No such category |
| Forced `tool_choice` | 400 | 400 | 400 | Accepted |

### The summary table

| Old habit | Status | Replacement |
|---|---|---|
| Prefill | 400 from 4.6 onward, on all four current models | Structured Outputs / explicit instruction |
| `budget_tokens` | 400 | `effort` alone, with `thinking` omitted or `adaptive` |
| Telling it in the prompt to think less, or not to think | In Anthropic's testing, did not stop Fable 5.2 thinking at `medium` and above; unreliable on Sonnet 5.5 and Haiku 5.5 too | Lower effort |
| `CRITICAL: You MUST` (anti-undertriggering) | No longer needed; adding IMPORTANT in CLAUDE.md is also on the list of old practices you can stop | `Use this tool when...`; make a rule that cannot have exceptions a hook |
| One verification rule for every model | The model pages say different things | Look it up by model (Section 5b) |
| Stacked `Never do X` | Rules are followed as written; an exception on a separate line has almost no effect | Positive description, plus the scope and the reason |
| The same instruction in several places | Redundant | One authoritative location (except the trigger condition) |
| Hand-maintained CLAUDE.md memory | Automatic memory exists | Automatic memory |
| A Markdown description used *in place of* the real material | Worse than references such as code, tests and mockups | Code / tests / artifact |
| temperature for variety | 400 on all four current models | Have the model propose several directions first, pick one yourself, then build it |
| Forced `tool_choice` | 400 on Fable 5.2, Opus 5.5 and Sonnet 5.5; accepted by Haiku 5.5 | `auto` plus strict tool use or structured outputs; say when the tool should be used |

---

## 5. Whether to give examples at all

Earlier versions of this document set the documentation's position here beside a public remark by an employee on the Claude Code team, read the two as appearing to conflict while each actually covered its own ground, and derived a rule of giving no examples in agent system prompts. That employee's remark is not an official page, and both the rule and the reading were removed in the 2026-11-09 round at the repository owner's request (the appendix has the record). This section now covers only what official sources say.

The prompting best practices page is direct: `Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.` It recommends `Include 3–5 examples for best results.`, and sets three requirements: **relevant** (close to the real use case), **diverse** (covering edge cases, so the model does not pick up unintended patterns), and **structured** (wrapped in `<example>`, with `<examples>` around them when there are several). If you are not sure your examples are good, you can have Claude look at them: `You can also ask Claude to evaluate your examples for relevance and diversity, or to generate additional ones based on your initial set`.

| Your goal | How to give them | Basis |
|---|---|---|
| Make the output **fit a fixed frame**: a fixed JSON shape, a fixed voice, a fixed report structure | 3–5, diverse, wrapped in `<example>`. Where a schema can govern the format, Structured Outputs is the other route | Best practices page |
| Demonstrate a **reasoning pattern** | Give complete worked examples, in the page's words `Present each example as a problem, the method to apply, and the expected answer.`; they shape how the model handles similar problems in its own thinking blocks. **Do not** build in a `<thinking>` section for the model to fill in | Best practices page; the refusals page lists a `<thinking>`, `<reasoning>` or scratchpad section that the model fills in before answering as a common source of `reasoning_extraction` refusals (Fable 5.2, Opus 5.5 and Sonnet 5.5 have this category; Haiku 5.5 does not) |

To teach tool use, the skill changes the interface first: the Section 1 post (written about the Opus 5 and Fable 5 generation) says `With our newest models, we've found that giving examples actually constrains them to a certain exploration space`, and suggests making the parameters of tools, scripts and files more expressive instead.

---

## 5b. Verification instructions: the official pages differ by model

This is the other place where earlier versions of this document reconciled the pages on their own. The reconciliation went like this: remove instructions that have the model look at its own work again, and keep checks done in a fresh context; on Opus 5.5, measure before deleting; on long Fable runs, self-check on a periodic schedule. That reconciliation was this document's own reading. No page says it, and it was removed in the 2026-11-09 round. Skill 3.4 now lists only what each page says, model by model:

| Model | What the official pages say | Source |
|---|---|---|
| General advice | Have the model check itself: `This catches errors reliably, especially for coding and math` | Best practices page |
| Opus 5 (legacy) | The only exception the advice above names: it verifies its own work without being prompted, and carried-over verification instructions can cause over-verification; `remove these instructions rather than rewriting them` | Best practices page; Opus 5 page |
| Opus 5.5 | Neither restates nor retracts the over-verification finding. Unattended long runs get a separate **completion** check: state the completion condition up front, then `have a separate, smaller model check the conversation against it at each end of turn`. What it checks is whether the work is done, not its quality | Opus 5.5 page |
| Fable models (in Claude Code) | `Skip the verification reminders: it verifies its own work with less prompting, so reminders to test or check are usually unnecessary` | Claude Code model-config, "Work with Fable" |
| Fable 5.2 | At high levels such as `xhigh` and `max` it has been observed using subagents to review its own work (Fable 5.1 did not on the same tasks); the same paragraph warns that subagents can account for a large part of what a session costs | Fable 5.2 page |
| Sonnet 5.5 | In coding it generally checks before reporting done; at `low` it sometimes reports done without running the checks that would test the change → snippet 4.46. At `xhigh` and `max` it goes the other way and may start its own rounds of review and verification after finishing → snippet 4.42 | Sonnet 5.5 page |
| Haiku 5.5 | At `low` and `medium` it sometimes reports a code change done without running checks → snippet 4.46 | Haiku 5.5 page |

**Why they are not merged into one rule.** These are statements about different models, and for techniques that name a model, the general prompting page says to `treat it as measured on that model and re-check it against your own evals before applying it to another`. Applying the Opus 5 result to Opus 5.5, or the Sonnet 5.5 verification paragraph to Fable 5.2, would in both cases treat a measurement on one model as a fact about another.

The Fable 5 page recommends self-checks at intervals on long autonomous runs, and says that separate, clean-context verifier subagents tend to outperform self-critique. The Fable 5.2 pages do not restate this, and what the Claude Code docs now say for Fable models is the `Skip the verification reminders` line in the table above, so the skill no longer treats periodic self-checks as advice for Fable 5.2.

**One point that does not depend on the model: give it a check it can run.** What the Claude Code best practices ask for is not a reminder to look again, but something whose result is pass or fail, with Claude showing its evidence (Section 8.4). The page's "What you can stop doing" table also separates a check from lines that tell Claude to keep going: `Giving Claude a check to pass is different, and still worth doing.`

---

## 6. The four current models, and how to choose a model and effort

The current official lineup, in the cost page's words: `From lowest to highest cost and capability, the current models are Claude Haiku 5.5, Claude Sonnet 5.5, Claude Opus 5.5, and Claude Fable 5.2 (the frontier model)`. But the most capable model and the one to choose by default are not the same thing (Section 6.5), and each model has tendencies its page states, with prompt fixes to match.

### 6.0 First, how each claim is filed

The general prompting page has one caveat for every technique that names a model:

> "Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another."

Each successor model's page then says, in one sentence, whether its predecessor's practices still apply:

- **Opus 5.5**: `Existing Claude Opus 5 prompts should perform well without changes, and the patterns in Prompting Claude Opus 5 remain a reasonable starting point.` The migration guide adds `Instructions tuned for Claude Opus 5's behavior might no longer be needed`.
- **Sonnet 5.5**: `Existing Claude Sonnet 5 prompts should perform well without changes`, and the second half of the sentence is the same as Opus 5.5's.
- **Haiku 5.5**: `Existing Claude Haiku 4.5 prompts should perform well without changes.`
- **Fable 5.2**: **no such sentence.** What it says is:

> "When you migrate to Claude Fable 5.2, review your prompts. In Anthropic's testing, some instructions written for earlier models raised cost, caused side effects, or no longer had an effect."

The migration guide's `Migration is mostly drop-in` is about the API layer.

Taken together, the two give the skill's three groups. What the current page states is used directly. What only a predecessor's page states, and the current page neither restates nor retracts, goes into a table marked "measured on that model", to be re-checked on your own evals before use. What has been retracted, or conflicts with the current page, is left out. The carry-over sentences say only that the predecessor's *prompting practices* remain a reasonable starting point. They do not say that the predecessor's *behavioral findings* still hold on the new model, which is why even the Opus 5 tendencies table is labeled "measured on Opus 5". And because Fable 5.2 has no carry-over sentence at all, everything on the Fable 5.1 and Fable 5 pages becomes "measured on that model" material.

### 6.1 Claude Opus 5.5 (and what was measured on Opus 5)

Released 2026-09-22 and positioned for long-running agentic coding and knowledge work. **Default effort `medium`** (on both the API and Claude Code), where Opus 5's is `high`; adaptive thinking always on and impossible to disable; 1M token context; 128K max output; $4 / $20 per MTok, against Opus 5's $5 / $25. Officially it generates output tokens more than 30 percent faster than Opus 5 and tends to complete the same task with fewer tokens. The selection matrix makes it the starting point for most workloads, and a new Claude Code session on most plans and providers runs it by default too, at `medium`.

**The one number to carry away:** in Anthropic's testing, **Opus 5.5 at `medium` matches or exceeds Opus 5 at `high`** on coding and knowledge-work evaluations. So the equivalent of an old `high` setting is the new default, and the page tells you to `test several levels against your own evals rather than carrying over the setting you used on Claude Opus 5`.

| Tendency stated on the Opus 5.5 page | What to do about it |
|---|---|
| Tends to think more per turn at a given level than Opus 5, most of all at `xhigh` and `max` | Re-run the sweep; reserve `xhigh` and `max` for work where you have **measured** a gain. To get less thinking, lower effort before reaching for prompt instructions |
| Text between tool calls comes back in **progress-update `thinking` blocks**, not `text` blocks, and is empty at the default `display` | Set `display: "updates"`. A client that renders only `text` blocks can look silent through a long agentic turn, with no error |
| Some progress reports **end the turn with text rather than a tool call** | An unattended loop must not treat that as completion: keep a checklist, and send a short message naming what is still open; stop after two or three automatic continuations |
| Tends to get to work quickly; on loosely specified multi-app tasks it misses context the request did not name | One sentence telling it to explore broadly before acting |
| Highly responsive to information about how much time has passed | In a multi-agent harness, give the task a time budget and report elapsed time at the end of each message; it adjusts its pace to finish inside the budget |
| In chat, thinks at length before answering, and on follow-ups sometimes goes back over earlier answers | Consider removing instructions to think carefully before answering. If you want earlier answers treated as settled, add two sentences (leave them out where you want it to keep re-examining earlier work, such as long analyses or agentic tasks where a later step can reveal an earlier mistake; if it matters, test whether it stops pointing out its own earlier mistakes). These two sentences have a documented side effect on Fable 5.2; see 6.2 |
| Robust against instructions inside pasted text, **once told which text was pasted** | Wrap pasted blocks in tags carrying the same random id, and explain the convention in the system prompt |
| Reads charts, diagrams and screenshots much more precisely than Opus 5 without tools | Re-test vision scaffolding built for earlier models; it may no longer be needed |
| Runs classifiers including biology, cybersecurity and reasoning extraction; of the three, only biology is **new for someone coming from Opus 5** | Handle `stop_reason: "refusal"`; note that server-side fallback does **not** retry a `reasoning_extraction` decline. Asking for a short explanation of the answer, or a summary of the actions taken, is still fine |

**Four breaking changes** land on code that already ran on Opus 5: thinking cannot be disabled, forced `tool_choice` returns 400, thinking blocks are tied to both the model and the conversation, and on the Claude API the older `computer_20251124` returns 400 (Amazon Bedrock still accepts it), so you switch to the `computer_toolset_20260801` toolset.

#### Measured on Opus 5 and not restated for Opus 5.5

Everything below comes from the Opus 5 page. On the strength of the Opus 5.5 page's carry-over sentence, these practices remain a reasonable starting point. But whether the tendencies themselves still exist on Opus 5.5, the official pages do not say either way, so they are **measured on Opus 5**, and two of them point the opposite way from documented Opus 5.5 behavior.

| Tendency (Opus 5) | What to do about it |
|---|---|
| User-facing responses run long by default | Ask for concision explicitly. **Note that lowering effort does not reliably shorten visible output**: effort governs how much it thinks, not how much it says. On the other side, Opus 5.5 `tends to complete the same task with fewer tokens`, so measure first rather than assume the problem is still there |
| Narrates often in agentic work | Describe the narration cadence you want. On Opus 5.5 the documented problem is **the opposite**: narration comes back in `thinking` blocks that are empty by default, so turns look quiet. Check `thinking.display` before adding anything |
| Files written to disk are often longer than on prior models too | Add a separate length calibration |
| Verifies its own work without being told to | The best practices page says `remove these instructions rather than rewriting them` for Opus 5 only; see Section 5b |
| Can expand task scope on its own | Constrain scope explicitly on narrow tasks |
| Delegates to subagents more readily | Set an explicit cap; on small tasks delegation multiplies cost and time |
| Narrates corrections to its own earlier statements more often | Restrict that to corrections that would affect the user's decisions |
| Might follow `only report high-severity issues` **literally** in code review | Have it report everything and filter in a separate pass (the Opus 5 page: `ask it to report everything and filter in a separate pass instead`) |

**One trap, documented only for Opus 5 with thinking disabled.** With thinking disabled, Opus 5 occasionally writes a tool call as ordinary text (the call never runs, and in an agentic loop that text stays in the conversation history and affects later turns), or leaks `<thinking>` tags into visible output. The official advice is to **keep thinking on and control cost with low effort** rather than turning it off: on most tasks, thinking on at `low` effort beats thinking off at similar cost. (At `xhigh` / `max` you cannot turn it off anyway; those requests return 400. Opus 5.5 cannot turn thinking off at all. The Opus 5.5 page says these two artifacts appear only on Opus 5 with thinking disabled, tells you to check whether you still need the mitigating instruction, and either way to **remove any inherited no-thinking rule**.)

If an integration must keep thinking disabled, the official mitigation is one combined instruction covering both problems:

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

Naming the thinking tags specifically works less well than this general form.

### 6.2 Claude Fable 5.2 / Mythos 5.2

Released November 9, 2026, at which point Fable 5.1 moved to the "Legacy models (still available)" list. $8 / $40 per MTok (Fable 5.1 is $10 / $50); 1M token context; 128K max output; adaptive thinking always on and impossible to disable; latency listed as Moderate, the same as Opus 5.5. **The default effort depends on the surface**: `high` on the Claude API, `medium` in Claude Code and the Claude apps. The choosing-a-model page calls it `Anthropic's most capable model open to all customers`, `built for demanding reasoning and long-horizon agentic work`. Mythos 5.2 has the same capabilities and is offered only to organizations verified through Anthropic's verification programs (for example, the Cyber Verification Program). Fable 5.2 requires 30-day data retention and is not available under zero data retention (unless expressly authorized by Anthropic).

Most capable and what to choose by default are not the same thing: the selection matrix still says `Most workloads start with Claude Opus 5.5`. On when to move from Opus 5.5 to Fable 5.2, the official sources do not all say the same thing, and the skill follows the steps in the 2026-11-09 model-selection article (Section 6.5).

**Why Fable 5.1 left scope.** No official page recommends it over Fable 5.2 on any point. It is on the legacy list; its own overview says `you should consider migrating to Claude Fable 5.2`; it costs more; and the Fable 5.2 page says Fable 5.2's scores at `low` `sat close to Claude Fable 5.1's scores at higher effort levels`. The only score and cost measurements pointing the other way are on the cost page. On DeepResearch Bench II, Fable 5.2 cost 20% to 61% less than Fable 5.1 at each effort level and scored 4.7 to 7.0 points lower. On the SWE-bench Pro subset, Fable 5.2 at `high` solved more tasks (96.3% against 92.4%) but cost more per solved task ($1.39 against $1.17). The advice the cost page puts right beside the research figures is not to stay on Fable 5.1, but `Measure what a more expensive model buys on your own workload`. So the skill records these measurements in models.md, and Fable 5.1, like Opus 5 and Fable 5, appears only as the source of specific wording, the starting point of a migration, or a comparison that a current page makes.

**What to do with old prompts: this is where this upgrade differs most from the previous two.** The Fable 5.1 prompting page says that existing Fable 5 prompts should perform well without changes, and on that basis this document and the skill carried the Fable 5 prompt blocks over to Fable 5.1. The Fable 5.2 pages have no such sentence. They have only the line quoted in 6.0, `When you migrate to Claude Fable 5.2, review your prompts.` Its prompting page reprints none of the older Fable-labeled blocks verbatim. So in the skill, the Fable 5.1 and Fable 5 entries are all labeled "measured on Fable 5.1" or "measured on Fable 5"; where the Fable 5.2 pages give new wording for the same problem, the entry points to it.

**Tendencies stated on the Fable 5.2 pages.** The skill's models.md has the full table; this lists only the ones you need in order to understand the skill's rules:

| Tendency (as the Fable 5.2 pages describe it) | Fix | Where in the skill |
|---|---|---|
| On long multistep tasks it sometimes ends the turn before the work is done: it closes with a list of what remains, an offer to continue, or a stop at an ordinary obstacle such as a timeout; more common at `low` and `medium` | Snippet 4.37. It separates two kinds of obstacle: diagnose and work around ordinary obstacles; leave blockers someone set on purpose alone and report them plainly. Anthropic replayed coding sessions at `medium`: about half as many sessions ended with doable work left, and output tokens rose by about 30%; the text removed most endings that ask permission or offer to continue, but not endings that list what remains and then stop. The model asks less often as a result, so keep your own confirmation step for risky or irreversible actions | Section 8, "Symptom to prescription"; Section 7, "Pre-delivery checklist" |
| Approval rules in the system prompt are applied as written | Have each rule name the specific actions it covers. When you tell it to continue, say what you want it to do (the official example: `Yes, run the migration on the staging database`) rather than a bare "continue" | Section 7, "Pre-delivery checklist"; models.md; Scenario A, "Say it all at once" |
| Cautious instructions pile up in memory, notes, instruction files and messages between agents; it follows them closely, so it might ask for approvals it does not need or hold back routine work (a few early testers saw this, mostly where several agents work together) | Delete notes you no longer need; name the steps that need no approval (in Anthropic's testing, a general instruction to act more freely changed nothing); check what reviewer agents attach; write the exception into the rule itself. Keep your own confirmation step for risky or irreversible actions | 3.3, 3.5, 3.13 |
| Responds closely to what the prompt says about who owns the code: with ownership unstated, it tends to add to the existing structure (even when that repeats existing code) and reports nearby problems without fixing them | To have it improve the structure as it works, use 4.38. The cost is about 1.5 times the output tokens, changes nobody asked for, and a closing summary that often leaves out small changes, so review the diff. Leave it out when the user wants changes kept as small as possible | Section 8, "Symptom to prescription" |
| Lines telling it to think less do not work at `medium` and above; at `high` and above it thinks before almost every reply, including a reply to a greeting | `Remove these lines and lower the effort level` | 3.10 |
| It also follows style rules closely: a hard limit (the official example: `one instruction per sentence, in 20 words or fewer`) can make its prose choppy | First delete style rules you no longer need, then add 4.40 | 3.13; Section 8, "Length control" |
| Effort barely changes reply length | `Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.` | Section 1, "Filling in variables"; Section 8, "Length control" |
| During long runs of tool calls it sometimes writes little user-facing text | First make sure the client receives it: the narration comes back as progress-update `thinking` blocks, empty under the default `display: "omitted"`, so set `"updates"` or `"summarized"`. If that is not enough, use the Fable 5.2 block in 4.3, or 4.39 | Section 8, "Symptom to prescription" |
| When the harness writes a token budget into the prompt, the size of the budget changes how much it spends | Try removing the budget: on coding tasks at `high` and `xhigh`, Anthropic measured that removing a large budget lowered cost with equivalent performance | 3.13 |
| The two sentences the Opus 5.5 page gives for chat (4.34, telling it to treat earlier answers as settled) have a side effect on it: less thinking on follow-up turns, but it becomes far less likely than before to point out mistakes in its own earlier answers | Leave them out where you want it to keep re-examining; elsewhere, test first | 3.13 |
| How much it delegates to subagents depends on the harness and the task; at high levels it has been observed using subagents to review its own work | First check the system prompt, skills and agent files for lines that tell it to delegate. The official words: `subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default` | Section 2, "Subagents"; 3.13 |

**Measured on earlier Fable pages.** Several tendencies on the Fable 5.1 prompting page (still online; its what's-new page and migration guide now redirect) are neither restated nor retracted on the Fable 5.2 pages: one tool call per turn, denser prose, less formatting in chat, summaries that reproduce source text without marking quotations, answering from memory at `low`, whole-file rewrites for small changes, and, at `xhigh`/`max`, drafting a long deliverable in thinking first. The skill keeps them in its "measured on Fable 5.1" table. The same goes for what is on the Fable 5 page: long turns, overplanning on ambiguous tasks, fabricated progress on long runs, unrequested actions, suggesting a new session, and skills written for older models being too prescriptive. The original wording of the last one:

> "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality."

**An old skill is not merely useless; it can actively drag quality down.** That was measured on Fable 5. Whether it holds on Fable 5.2 is not stated either way. What the Fable 5.2 page gives is its own review list ("Review instructions written for earlier models", skill 3.13), and skills and agent files are within its scope.

**What is no longer kept.** Earlier Fable pages make four more claims on which the Fable 5.2 pages say something different, so they are no longer offered as advice for Fable 5.2. One is that it dispatches parallel subagents more readily and you should encourage that (the Fable 5 page; 5.2 pages: first check your delegation instructions, and watch subagent cost). One is that one brief instruction can steer a class of behavior (the Fable 5 page; 5.2 pages: rules are applied as written, so name the specific actions; in Anthropic's testing a general line changed nothing). One is that it over-delivers on open-ended features and does unrequested tidying and refactoring at high effort (the Fable 5.1 and Fable 5 pages, snippet 4.6; 5.2 pages: with ownership unstated it tends to add to the existing structure and report nearby problems without fixing them, and changes nobody asked for come up only once the code is said to be its own). One is that on complex async tasks it describes the next step instead of doing it (the Fable 5.1 page, snippet 4.18; the 5.2 pages give new wording for the same problem, 4.37). Snippets 4.6 and 4.18 are still in the skill's snippets.md, each labeled with the model it was measured on. Earlier versions of this document also said that Fable was the slowest model, that it could go quiet for minutes in long turns, and that its gains grew with the effort level. All of that came from predecessor pages, while Fable 5.2's listed latency is Moderate and its page says `Most of the gain in capability is already present at high`, so those were removed as well.

**At the API layer (only if you assemble requests yourself).** The prefix check applies to every Fable 5.2 account: sending thinking blocks back after `system`, `tools` or an earlier message changed can return 400. So keep the history append-only, send per-turn reminders as turn-scoped system messages, and make prompt changes only in new conversations. Only Fable 5.2 and Mythos 5.2 can read Fable 5.2's thinking blocks; once routing or a fallback moves the conversation to another model, the later turns run without its reasoning. Change effort mid-conversation with the per-message setting (beta), not the top-level value. Details are in Section 7.6 and the skill's agents.md.

### 6.3 Claude Sonnet 5.5 and Claude Haiku 5.5

Both had shipped by the 2026-10-23 round, but the default recommendation had not moved, and the skill kept them out of scope. They come into scope in this round because the skill now chooses models across the whole current lineup, subagents included, and the selection matrix lists subagent tasks under Haiku 5.5. In three places their API behavior differs from the other two models, which is exactly why the per-model table in Section 4 exists.

**Sonnet 5.5** (released 2026-09-28, $2 / $10 per MTok, latency listed as Fast). Officially positioned as `The best combination of speed and intelligence`; its page also says `For the hardest long-horizon work, an Opus model is the better choice.` Snippets 4.16 and 4.25 come from the Sonnet 5 page, on the strength of the Sonnet 5.5 page's line `Existing Claude Sonnet 5 prompts should perform well without changes`. What you need to know to understand the skill's rules:

- `disabled` returns 400 at every level; to turn off up-front thinking use `between_tools`, which is available only at `high` and below
- The effort levels have been recalibrated, so the same level does not mean the same amount of thinking as on Sonnet 5; the API default is `high`, the Claude Code default `medium`
- In agentic coding at `low` and `medium` it sometimes stops to confirm before the work is done (4.41); at `low` it sometimes reports done without running checks (4.46); at `xhigh` and `max` it may start its own review rounds after finishing (4.42; Anthropic measured on coding tasks at `max` that it stopped starting reviewer subagents, session cost fell by about a third, and quality was unchanged)
- Telling it to think less `doesn't reliably reduce its thinking`
- A real message the user sends partway through is sometimes treated as an injection, especially when it sits in a `tool_result` block

**Haiku 5.5** (released 2026-10-07; $0.10 / $0.50 per MTok for prompts up to 100,000 tokens, $0.50 / $2.50 above that; latency listed as Fastest; default `medium`). Officially positioned `For high-volume, latency-sensitive tasks such as classification, extraction, and routing`; its page says `Existing Claude Haiku 4.5 prompts should perform well without changes.` It is the first Haiku with effort levels. In three places its API behavior differs from all three other models: it can turn thinking off at `high` and below; it accepts forced `tool_choice`; and it has no `reasoning_extraction` refusal category. At the prompt level: given a search tool, it needs to be given today's date (4.47); at `low` and `medium` it sometimes reports code changes done without running checks (4.46, the same text as on the Sonnet 5.5 page); and deployed as a chatbot, it can be worn down by users into not following the system prompt (4.50).

### 6.4 The effort parameter

`max` > `xhigh` > `high` > `medium` > `low`. All four current models support all five levels.

The effort page's general description of each level (where a model's own advice differs, the model's advice wins):

| Level | Official description | Typical use case |
|---|---|---|
| `max` | Absolute maximum capability with no constraints on token spending | Tasks requiring the deepest possible reasoning and most thorough analysis |
| `xhigh` | Extended capability for long-horizon work | Long-running agentic and coding tasks (**over 30 minutes**) with token budgets in the millions |
| `high` | Spends as many tokens as the task needs for excellent results. **The default on every model that supports effort except Claude Opus 5.5 and Claude Haiku 5.5** | Complex reasoning, difficult coding problems, agentic tasks |
| `medium` | Balanced approach with moderate token savings. **The default on Claude Opus 5.5 and Claude Haiku 5.5** | Agentic tasks that require a balance of speed, cost, and performance |
| `low` | Most efficient. Significant token savings with some capability reduction | Simpler tasks that need the best speed and lowest costs, such as subagents |

**Defaults differ by model, and by surface:**

| | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Claude API | `high` | `medium` | `high` | `medium` |
| Claude Code | `medium` | `medium` | `medium` | `medium` |

In the Claude apps, each model's recommended level is marked "Default" in the menu; the launch announcement says Fable 5.2 defaults to `medium` in the Claude apps and Claude Code. The support article on the app settings adds `For complex coding and agentic tasks on Opus 4.7 or newer, try Extra high (xhigh) first`. The Opus 5.5 and Fable 5.2 pages keep `xhigh` for work where a gain has been measured, and the Sonnet 5.5 and Haiku 5.5 pages use `xhigh` only where evals show a quality gain, so the skill chooses the level by the model pages; when a Scenario B task is complex coding or agentic work, it tells the user both in the delivery note. The same model, on the same request that omits `effort`, can run at a different level in the API and in Claude Code, which is why skill Section 2 treats the two surfaces separately.

> **The starting advice has changed more than once.** The Opus 5 starting point was changed from `xhigh` to `high`, Opus 5.5 then lowered the default to `medium`, and Fable 5.2's default splits by surface. The appendix has the history.

Points that matter:

- **Setting `effort` to the model's default behaves exactly the same as omitting the parameter** (official wording: `Setting effort to the model's default ("medium" on Claude Opus 5.5 and Claude Haiku 5.5, "high" on other models) produces exactly the same behavior as omitting the effort parameter entirely.`). This refers to the API defaults
- Effort is a **behavioral signal, not a hard token budget**. Official wording: `Effort is a behavioral signal, not a strict token budget. At lower effort levels, Claude still thinks on sufficiently difficult problems, but thinks less than it would at higher effort levels for the same problem.`
- It affects **all output** tokens: text, tool calls, and thinking
- **Lower effort means fewer and terser tool calls**; lower levels tend to combine operations, skip preamble and give a terse confirmation when done, while higher levels might make more tool calls, explain the plan before acting and give detailed summaries. This is the part people miss: **effort is not only how long it thinks; it is also how much it does**
- **On Fable 5.2, effort barely changes reply length** (stated on its page; the Opus 5 page also says lowering effort does not reliably shorten visible output). Put the length in the prompt; this is the basis of skill Section 8, "Length control"
- **Set effort explicitly per workload** rather than running one level across everything
- **Hold the top-level effort constant within conversations that rely on the cache.** Official wording: `Changing the top-level effort value between requests invalidates prompt caching, so vary it across workloads rather than within a conversation that relies on cache hits. On models that support it, use a per-message effort change instead, which preserves the cache.` All four current models support a per-message effort change (beta): a `role: "system"` message with empty `content` and the new level in `output_config.effort`. Placed directly after a user turn with new input, it sets the level for the reply to that turn; anywhere else it takes effect from the next user turn with new input (one holding only `tool_result` blocks does not count). A per-message effort change returns 400 when Sonnet 5.5 uses `between_tools` or when Haiku 5.5 has thinking off
- At high effort, leave headroom in `max_tokens` (it is the **combined** limit on thinking plus response text), and thinking counts toward it even when the thinking text is not returned to you. For the long turns of agentic coding on Opus 5.5, the page says `a max_tokens of 128,000, the model's maximum, has worked well in Anthropic's testing`; Sonnet 5.5 doing agentic coding also sets 128,000 and streams; on Fable 5.2 it needs to be set large from `high` up. In Claude Code and the Claude apps the client manages this value

> Aside: `ultracode` in Claude Code is **not** an additional API effort tier. The [dynamic workflows page](https://code.claude.com/docs/en/workflows) defines it as `a Claude Code setting that turns on automatic workflow orchestration for the session, at whichever effort level the session runs at`. It is a session-level setting, and the Messages API has no such level. Typed into a prompt as a keyword, it runs only that one task as a workflow: `To run a single task as a workflow without changing the session's effort level, include the keyword ultracode in your prompt`.

### 6.5 Choosing model and effort: by task

Earlier versions of this document gave a two-condition test for what counts as agentic work heavy enough to warrant `xhigh`. That test had no official source at all, and was removed in this round. The only numerical threshold the official material gives is the effort page's description of `xhigh` (over 30 minutes, token budgets in the millions), and each model's own advice takes precedence over that general table.

Skill Section 2 now has six subsections: a table for choosing the starting model and effort by task; a table of when to raise or lower effort on each model; the order of steps when quality falls short or cost is too high, with the places where the official sources disagree; a subagent table (Section 6.6); bulk processing, with the owner's labeled rules beside the official figures; and usage and cost, with a table of where each parameter is set. The first table follows, row for row, "Pick a model by task" in Anthropic's 2026-11-09 article "Which Claude model should you build with?". The article's starting point is `you should start most work on Opus 5.5 at its default effort level`, that is, `medium`. The platform docs' selection matrix also starts from Opus 5.5 (`Most workloads start with Claude Opus 5.5`), but it files two kinds of work differently: it puts large-scale refactoring under Opus 5.5, and analysis carried through to a finished document, spreadsheet or deck under Fable 5.2.

| Task | Starting model | Starting effort | The article's reason |
|---|---|---|---|
| Everyday engineering and debugging; multi-PR feature work | Opus 5.5 | `medium` (default), then `high` | Balance of cost and capability for everyday tasks |
| Classification, routing, tagging, high-volume extraction | Haiku 5.5 | `medium` (default) | Lowest cost and latency in the model family |
| Subagents that search files, read logs, or summarize | Haiku 5.5 or Sonnet 5.5 | The model's default | Wrong answers are cheap to spot, and the judgment calls stay with the main model |
| Chat and interactive sessions | Sonnet 5.5 (Haiku 5.5 for simple chat at volume) | `low` or `medium` | Sonnet 5.5 is fast, especially at `low` and `medium` |
| Well-scoped coding: bug fixes, feature iteration, spec-driven work | Sonnet 5.5 | `medium`, `high` for harder tasks | It handles scoped tasks well, at half the Opus 5.5 token price |
| Everyday documents, slides and spreadsheets | Sonnet 5.5 | `high` (API default; `medium` in Claude Code) | It iterates fast and has a good eye for design. For a complex financial model, or a document built from a large pile of source material, start on Opus 5.5 |
| Agent tasks you run repeatedly: investigation, review, drafting | Sonnet 5.5 | `medium` | Well-defined, repeated tasks rarely need a bigger model |
| Computer use and vision-heavy workflows | Opus 5.5 | `medium` | These runs are long agentic loops over many screenshots and images, the kind of work Opus 5.5 is built for |
| Major refactors and migrations, architecture planning, leading many subagents | Fable 5.2 | `medium`, then `high` (the API default is `high`) | Anthropic's most capable generally available model for technical fields |
| Deep research, scientific or quantitative analysis | Fable 5.2 | `medium`, then `high` (the API default is `high`) | Built for the hardest technical projects in coding, finance and the sciences |
| You aren't sure yet | Opus 5.5 | `medium` (default) | Claude Code uses it by default on the subscription plans and the Anthropic API, and the platform docs recommend it as the starting point |

The same article adds two points. On moving down: `Just because you have a clear spec doesn't mean you should use Sonnet`. A migration or a performance optimization can have a clear spec and still be hard enough to warrant Opus 5.5, or Fable 5.2 for a major one. On moving up to Fable 5.2, it names three cases: the hardest technical projects in coding, finance and the sciences; Opus 5.5 at `high` still falling short; and long unattended runs.

**Raising and lowering effort on each model** (sources: the per-model sections of the effort page, the four models' prompting pages, and the Claude Code model-config page):

| Model | Default | Up | Down |
|---|---|---|---|
| Haiku 5.5 | `medium` | Knowledge work, longer agent tasks, strict instruction following → `high`. `xhigh`/`max` only where evals show a quality gain, and compare against Sonnet 5.5 at the same time | Chat, short tool tasks, simple high-volume requests → `low` |
| Sonnet 5.5 | API `high`; Claude Code `medium` | Well-specified agentic coding and multistep tool use start at `medium`; harder or longer work → `high`. `xhigh`/`max` only where evals show a quality gain | Chat and latency-sensitive work → `medium` or `low` |
| Opus 5.5 | `medium` | Work where verification matters or edge cases are likely → `high` (the model-config page's example is fixing a bug in an existing codebase). `Reserve xhigh and max for work where you've measured a quality gain`; the model-selection article adds that before you raise Opus 5.5 past `high` on a hard task, you should test Fable 5.2 | Evals show quality holds → `low` |
| Fable 5.2 | API `high`; Claude Code and the Claude apps `medium` | Hard coding, research, agentic work and long unattended runs → `high`. `Most of the gain in capability is already present at high`, `Above high, gains are uneven and depend on the task`; keep `xhigh` for the hardest long-running work where a gain has been measured, and `max` for measuring the ceiling | Where quality holds → `medium`; quick back-and-forth, well-specified tasks, high volume → `low` |

**When a model falls short.** The model-selection article gives three steps, in order:

1. Make sure the model has a way to verify its own work: a test, a build, or a script that calls the endpoint. The article says that giving it a way to check itself is often more cost-effective than changing effort levels, and adds less latency
2. Raise effort one level
3. Move to the next model up, and start it at a lower effort than you used on the smaller model

The official points that go with it:

- **Tune effort first, then change model**: `Tuning effort is often a better lever than switching models` (the choosing-a-model page)
- **Past `high`, compare both routes**: the article says one more effort level can cost as much as a bigger model; once you are past `high`, try the next model up at a lower effort
- **Raising effort does not add knowledge or skill the model lacks**; for that, change model (the article)
- **Costs too high, quality fine**: try lower effort on the model you use now (the cost page)
- **Do not carry over a level tuned on another model**: in the Fable 5.2 page's words, `rerun your effort sweep instead of carrying over a setting you tuned on Claude Fable 5.1 or Claude Opus 5.5`
- **Compare cost per completed task**: `Compare models on cost per completed task, not per token`

**Places where the official sources do not quite agree. Skill Section 2 lists them, and the delivery note points out whichever one the choice relies on (Scenario B has one more, the support site's `try Extra high (xhigh) first`; see Section 6.4):**

- **When to move from Opus 5.5 to Fable 5.2.** The model-selection article says when Opus 5.5 at `high` still falls short. The choosing-a-model page says when evals at `xhigh` or `max` still fall short. The Fable 5.2 what's-new page, the models overview and the cost page say when evals `at higher effort` still fall short. The cost page's "Start here" table puts it another way: if quality isn't good enough and you lowered effort, restore it; otherwise `try the next tier up at its default effort`. The skill follows the article's three steps, because the article is the only one of these sources that puts self-verification first
- **Research tasks.** The task table and the selection matrix both put deep research under Fable 5.2. On DeepResearch Bench II, the cost page measured Fable 5.2 at 2.9 times the cost of Opus 5.5 for a score 1.9 points lower, and the advice right beside those figures is `Measure what a more expensive model buys on your own workload`. (The cost page's other line, `On research tasks, measure the model you ship`, sits in its section on effort and compares one model's results across effort levels, not these figures across models.) Both are official, so the skill does not pick a side for the user
- **Where Fable 5.2 starts.** The task table (the model-selection article) starts it at `medium`, and `medium` is also its default in Claude Code and the Claude apps. The Fable 5.2 prompting page, the what's-new page, the effort page and the choosing-a-model page all say to start at `high`, the Claude API default. The prompting page's words are `Start at high, the default on the Claude API`, and it says `high` suits hard coding, research and agentic work. In Scenarios A and B the skill follows the task table: `medium` first, then `high` if that falls short. A prompt written for the API starts at `high`, as the platform pages say
- **Which model to start from.** The model-selection article and the selection matrix start most work on Opus 5.5. The choosing-a-model page also gives an efficiency-first route: start with Haiku 5.5 and upgrade only for specific capability gaps. The cases it lists are initial prototyping and development, tight latency requirements, cost-sensitive implementations, and high-volume, straightforward tasks. The skill follows the task table, and when the user's task is one of those four cases, it tells the user about this route too
- **Which model for everyday coding in Claude Code.** The model-selection article starts everyday engineering on the default, Opus 5.5. The Claude Code best practices say `the default Opus 5.5 handles most coding well and Sonnet costs less on routine work`. The Claude Code costs page says `Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning.` The skill follows the task table: Sonnet 5.5 for well-scoped coding, and Opus 5.5 for other everyday engineering

**In Claude Code**, all four current models start at `medium`; a Fable model is not the default on any plan and has to be selected explicitly with `/model fable`. The Claude Code best practices say `medium` is `plenty for everyday feature work with a clear scope`. When being right depends on cases that are easy to miss (a bug in code you did not write, a flaky test, a schema change where every reader of the old structure must be found), raise it to `high` or `xhigh`. When the back-and-forth is quick and you look at every result, lower it to `low`. Keep `max` for hard problems you want it to work through on its own, since on ordinary tasks it tends to overthink. And only when the next message needs deeper reasoning, write `ultrathink` in that prompt.

**The skill's three user rules (User rule, not official guidance; confirmed 2026-11-09).** Skill Section 2 has three further practices of the repository owner's own. They govern only how to choose among the options the official material gives, and they are not official guidance. By default the Claude that writes the prompt decides model and effort itself, and asks the user once only when the trade-off cannot be inferred. It neither deliberately saves usage nor raises anything without a basis. And in bulk tasks it chooses models separately for the main session and the batch subagents. The last rule was revised on 2026-11-10. It used to say that batch subagents always run on `opus`, at effort `high`, with `xhigh` for the review; the official material has not measured the combination of a Fable 5.2 main session with Opus 5.5 subagents, and those two levels had no official basis, so the owner replaced it with a table taken from the cost page's defect-search benchmark. The default is `sonnet` at `high` (Fable 5.2 orchestrating Sonnet 5.5 subagents measured 0.833 at $165, and in that measurement Sonnet 5.5 ran at `high`). When a trial run shows many misses or wrong calls, switch to `opus` at `high` (the closest measurement is Opus 5.5 with one session per part, 0.850 at $236). When every item is hard, pass no model so the subagent follows the Fable 5.2 main session, at effort `medium` (0.877 at $378, drawing on the weekly Fable limit). For work that only reads files and summarizes, with output a script can check, use `haiku`. The final independent review follows the main session's model at `high`, and `xhigh` is used only where a quality gain has been measured. When to apply this rule at all is also the user's choice, not an official trigger. The `model` and `effort` parameters apply only to non-fork subagents: a fork runs on the main session's model, and fork mode is on by default, so the skill has the prompt say that the batch goes to non-fork subagents. The skill also says to mention in the delivery note, where they apply to the user's task, two things the official measurements show: on work that fits in one context window and has no long cost tail, the orchestrator's model working alone at lower effort comes out cheaper (measured on earlier models), and Fable 5.2 working alone scored higher than with Sonnet 5.5 subagents, so what splitting buys is a lower cost (Section 6.6).

### 6.6 Subagents and usage

**Why subagents are chosen separately.** In Claude Code, a subagent with no model specified uses the same model as the main session. So once the main session is on Fable 5.2, the subagents it dispatches also run on Fable 5.2 by default (there are exceptions: of the built-in ones, Explore runs on the model the `opus` alias points to when you sign in with a subscription, and statusline-setup and claude-code-guide always use Sonnet and Haiku respectively; setting `CLAUDE_CODE_SUBAGENT_MODEL` also changes the default). The Fable 5.2 page's line `subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default` is aimed at exactly this.

| What the subagent does | Model | Official basis |
|---|---|---|
| Searching files, reading logs, summarizing, checking documentation links | `haiku` or `sonnet`, at that model's default effort | The model-selection article's task table; Claude Code best practices: `Set model: haiku in a subagent's definition if its work is simple` |
| High-volume batch work with outputs you can check | Try `haiku` first | Cost page, "Start here" table |
| Work that exceeds one context window, split into one subagent per part | A model cheaper than the main session's; with an Opus 5.5 orchestrator, `sonnet` is the one officially measured to keep the score, and `haiku` was measured too and scored clearly lower | Cost page: Opus 5.5 with Sonnet 5.5 subagents cost about a third less than one Opus 5.5 session per part, at about the same score; with Haiku 5.5 subagents the score was clearly lower; with Fable 5.2 as the orchestrator instead, cost was 19% higher at about the same score |
| Teammates in an agent team | `sonnet` | Claude Code costs page: `Use Sonnet for teammates` |

The model-selection article's principle for dividing the work: `Keep small models on work where a mistake is cheap to spot, like finding files, running tests and reading logs. Keep code edits and judgment calls on the main model.`

**When delegating is worth it.** The cost page measured that delegation pays off in only two cases: routine work with a long cost tail (measured on earlier models; for that case the official advice is to measure accuracy and cost first), and work that exceeds one context window. When the work is one dependent chain, or fits in one context without a long cost tail, the orchestrator's model on its own at lower effort comes out ahead (also measured on earlier models). On the same defect-search benchmark, Fable 5.2 working alone (`low` to `high`) scored 3.2 to 7.5 points higher than it did with Sonnet 5.5 subagents, for 2.2 to 3.4 times the cost, so what splitting buys is a lower cost, not a higher score. The Claude Code best practices say subagents bring parallelism, keep long output out of the main session, give you a reviewer that has not seen how the code was written, and let a task run with fewer tools; in the page's words, `You get parallelism or a fresh pair of eyes out of it, but not a smarter answer.` The cost page's test for adding a second model: sweep effort on your current model first, and `Build a pairing only if it beats the single model's whole curve of score against spend`.

**Subagent effort**: the model-selection article starts subagents that search, read logs or summarize at the model's default effort. The effort page lists subagents as an example for `low`, and the Fable 5.2 page says `low` suits fast, intelligent subagents. Measure before lowering it: on a defect-search benchmark, Opus 5.5 alone scored 34 points lower at `low` than at `medium`, and Fable 5.2 alone scored 4.3 points lower at `low` than at `high`, for 37% less cost. So the skill starts subagents at the model's default level.

**Usage and price**: the points you need most often when writing a prompt (the full figures are in the skill's models.md):

- Price per million tokens (input / output): Fable 5.2 $8 / $40, Opus 5.5 $4 / $20, Sonnet 5.5 $2 / $10, Haiku 5.5 from $0.10 / $0.50
- On subscriptions, subagent requests count toward the same usage limit as the main session; on Max and Ultra, Fable models can use at most half of the weekly limit, and they use it up faster than other models; the higher the effort, the sooner you reach the limit
- Output tokens cost five times as much as input tokens, and in an agent loop every token the model writes is counted again as input on every later turn, which is why the cost page advises `Ask for the answer you will read, not the one that looks thorough`
- When outputs can be checked (API): run once at `low` and re-run the failures at `high`; officially measured on Opus 5.5 at about 40% cheaper than running everything at `high`, with the same pass rate

---

## 7. Agents and long tasks: context engineering

### 7.1 Context is a finite resource: context rot

The model has a finite attention budget. The longer the context, the more diluted the attention and the worse the performance. This is not an engineering bug: transformer attention gives n² pairwise relationships for n tokens, and models see less training data at long lengths, so performance degrades on a gradient rather than at a hard cliff.

So the goal of context engineering is not "make it fit" but **the smallest possible set of high-signal tokens**.

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

1. **Use a different prompt for the very first window**: set up the framework (write tests, create setup scripts) and use later windows to iterate on a todo list
2. **Have the model write tests in a structured format** (`tests.json`) and tell it explicitly: `It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **Set up scripts such as `init.sh`** that start servers, run tests, and run linters, so each fresh window does not repeat the work
4. **Consider starting fresh instead of compacting.** Counterintuitive but grounded: current models are extremely effective at discovering state from the local filesystem, and in some cases you can take advantage of that instead of compacting (the official wording is `consider` and `in some cases`, not always). When you do, be prescriptive about the opening; the three official lines are `Call pwd; you can only read and write files in this directory.`, `Review progress.txt, tests.json, and the git logs.`, `Manually run through a fundamental integration test before moving on to implementing new features.`
5. **Provide verification tools** (browser automation, computer use, and so on). The longer and more autonomous the task, the more the model needs to verify correctness without continuous human feedback
6. **Encourage complete use of the context**: have it finish each component efficiently before moving on, and tell it that spending its whole output context on the task is encouraged, as long as it does not run out with a lot of uncommitted work

For state: structured data in JSON, progress notes in freeform text, versions and checkpoints in **git** (current models are especially effective at using git to track state across sessions).

### 7.5 Let Claude orchestrate itself

This one is easy to miss because it lives at the harness layer rather than the prompt layer.

The common pattern is to route every tool result back into the context window as tokens, with filtering logic hardcoded into the tool.

**The better pattern:** give Claude code execution (bash, a language REPL) so it can write logic that expresses the tool calls and filters the output itself; **only the execution results reach its attention**.

The official figure: on BrowseComp web-browsing tasks, giving Opus 4.6 the ability to filter its own tool outputs raised accuracy from **45.3% to 61.6%**.

Two more in the same direction, both with numbers:

| Change | Effect |
|---|---|
| Let Claude manage its own context (progressive disclosure through skills, context editing, subagent isolation) | Subagents: BrowseComp +2.8% over the best single-agent runs (Opus 4.6) |
| Let Claude persist its own context (compaction plus memory folders) | Compaction: BrowseComp 84% for Opus 4.6 vs. 43% for Sonnet 4.5; a memory folder: BrowseComp-Plus 60.4% → 67.2% (Sonnet 4.5) |

> **Provenance:** these figures come from Anthropic's experiments on an **earlier model generation** (Opus 4.6 and Sonnet 4.5, as named above), not on the models in scope. The article's argument is that as models improve you can hand more decisions back to them, and that `Assumptions about what Claude can't do need to be re-tested with each step change in its capability`, so treat the figures as a direction to re-test on the current models, not as numbers that carry over.

There is a vivid comparison in the same piece: a newer model playing Pokémon writes organized tactical notes (10 files organized into directories, three gym badges), while an earlier generation treats memory as a running log (31 files, two of them near-duplicates, still in the second town). **Same memory mechanism, very different results, which the article presents as an example of Claude's improved ability to use a memory folder.** This is the other side of scaffolding expiring: as the model gets stronger, hand more of the decisions **back** to it.

### 7.6 Cache hits: a pure cost lever

**Cache reads cost 10% of base input tokens** (5% on Opus 5.5 and Sonnet 5.5, 2.5% on Fable 5.2 and Mythos 5.2). That makes the ordering of your context a direct determinant of your bill. Five principles:

| Practice | Point |
|---|---|
| **Static first** | Stable content (system prompts, tool definitions) before dynamic material |
| **Append in messages rather than editing the prompt** | Inject reminders by appending to messages, not by going back and editing the system prompt |
| **Do not switch models mid-session** | Caches are model-specific |
| **Adding or removing tools invalidates the cache** | Use tool search for dynamic discovery, or mid-conversation tool changes, instead of adding or removing tool definitions |
| **Move breakpoints** | In multi-turn applications, move the cache breakpoint to the latest messages |

Mid-conversation tool changes are the newer, official form of the fourth row: declare the full tool set in `tools` up front, then offer or withdraw a tool from a point in the conversation with `tool_addition` / `tool_removal` blocks in a `role: "system"` message (beta). Since September 22, 2026 a `tool_addition` can also carry a full definition for a tool nobody knew about at the first request. The `tools` array never changes, so the cached prefix, and on the four current models the thinking blocks after it, stay valid.

That last half now matters more than the cache does. On all four current models a thinking block is valid only against the content that came before it when it was produced; send it back after changing `system`, `tools` or an earlier message, and it fails the prefix check. On the Messages API, Fable 5.2 returns 400 for every account, and Opus 5.5, Sonnet 5.5 and Haiku 5.5 return 400 for accounts created on or after 2026-08-31 (in the Message Batches API, these cases drop the failing blocks instead). Most edits that trip the prefix check also restart the cache, and the fix is the same: turn-scoped system messages for reminders, and mid-conversation system messages and tool changes for changing instructions and tools, rather than rewriting `system` or `tools`.

This also explains the otherwise arbitrary-looking reminder in Section 6.4, **hold the top-level effort constant within conversations that rely on the cache**: changing it breaks the cache in the same way (per-message effort exists precisely to change the level without touching the prefix).

### 7.7 When to promote a bash action to a dedicated tool

Bash gives broad leverage, but the harness sees only a command string, the same shape for every action. A dedicated tool gives it an action-specific hook with typed arguments it can intercept, gate, render, or audit. Four cases warrant a dedicated declarative tool:

1. **Security boundaries**: hard-to-reverse actions, such as external API calls, which can be gated by user confirmation
2. **Write operations needing staleness checks**: preventing overwrites of files that changed since they were last read
3. **User-facing presentation**: modal displays, multiple options
4. **Structured logging, tracing, and replay**

There is another path, which should only be used for tasks where users trust the general direction: **use a second Claude instance to judge whether a bash command is safe**. That is what Claude Code's auto mode does, and it can limit the need for dedicated tools; dedicated tools can still earn their place for certain high-stakes actions.

Claude Code makes this trade-off itself. According to the tools reference page, on macOS, Linux and WSL, Glob and Grep are not in the default tool set, and Claude searches by running `find` and `grep` through Bash. File changes, though, still go through the dedicated Edit tool, and an edit has to pass its checks before it applies; the page lists three, read-before-edit, match and uniqueness.

> This section looks like it contradicts 7.5 (one says build fewer tools and use general-purpose bash, the other says build dedicated tools). In fact they are two points from the same article: `ask what you can stop doing`, and `carefully set boundaries with the agent harness`. If the model can handle it, do not constrain it with a tool; if the product must be able to intercept, audit or render it, make it a tool. And the article says that whether to promote an action to a tool should be re-evaluated continually as model capability changes.

### 7.8 Tool design

From Anthropic's articles "Writing effective tools for agents" and "Effective context engineering for AI agents". What a good tool looks like:

- **Each tool has a clear purpose, distinct from every other tool**: `Make sure each tool you build has a clear, distinct purpose`. The test is simple: `If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better`
- **Unambiguous parameter names**: `user_id` rather than `user`
- **Meaningful identifiers in what tools return**: resolving arbitrary UUIDs to interpretable names (or even a 0-indexed ID scheme) reduces hallucinations and significantly improves Claude's precision in retrieval tasks
- **Namespacing** by service and resource type (`asana_projects_search`, `asana_users_search`)
- **Token-efficient returns**: pagination, range selection, filtering, truncation, with sensible default caps (Claude Code's default is 25,000 tokens)
- **A `ResponseFormat` enum** (`concise` / `detailed`): in the official example the detailed version is 206 tokens and the concise one 72, about a third
- **Actionable error messages**: say what to change and show a correctly formatted example, rather than returning a stack trace
- **Tool descriptions written as onboarding for a new team member**: spell out implicit assumptions, specialized query formats, definitions of terms, and resource relationships

---

## 8. Claude Code: where to put what

The practical pain point for most people: the same sentence, placed in CLAUDE.md versus a skill versus a hook versus a subagent, has a completely different effect and a completely different cost.

First, a change: the Claude Code best practices page was rewritten from top to bottom between the 2026-10-23 and 2026-11-09 rounds. The explore / plan / implement / commit section, the five common failure patterns that this document's earlier Section 8.5 rested on, and most of the example prompts this document used to quote are gone. New are deterministic and non-deterministic instructions, the cost of subagents, unattended runs, and a "What you can stop doing" table. The skill's for-claude-code.md was rewritten against the new page, and this chapter follows it.

### 8.1 The tradeoff table

Anthropic's "Steering Claude Code" article lists seven methods (CLAUDE.md takes two rows here, root and subdirectory), with the content checked against the current docs:

| Mechanism | Loaded | Context cost | Best for |
|---|---|---|---|
| **CLAUDE.md (root)** | Session start, resident throughout, re-read after compaction | **High** | The commands that confirm work, places where the code is misleading, decisions that are not in the repo, anything already corrected twice. The article's wording: `Keep CLAUDE.md under 200 lines, give it an owner, and review changes to it like code.` |
| **CLAUDE.md (subdirectory)** | When that directory is touched | Low | Directory-specific conventions |
| **Rules** | Session start, or on file access | Medium | Rules that matter only in one part of the codebase. **Add `paths:`** to scope them, so they load only when Claude is actually working there |
| **Skills** | Name and description resident, body on demand | Low | Team procedures and knowledge: the release process, the incident runbook. For a workflow with side effects, add `disable-model-invocation: true` so it runs only when you type the command yourself |
| **Subagents** | Isolated context; only the final conclusion returns | Low for the main context; its own requests still count toward usage | Side tasks where you want only the conclusion: log analysis, dependency audits |
| **Hooks** | Lifecycle events; bypass compaction | Low | Zero-exception rules and **deterministic** automation: formatting, blocking commands. For real enforcement, use a `command` or `http` hook; on most events, when the hook itself fails or times out the action still goes through, so set `onFailure: "block"` if it should be stopped then too |
| **Output styles** | Session start, never compacted | **High** | Tone, length or format you want on every reply, or having Claude work in a role other than software engineer |
| **Appending to the system prompt** | Passed as a CLI flag at startup | Medium | Instructions in scripts and automation that need to sit at the system-prompt level (`--append-system-prompt`) |

### 8.2 Deterministic and non-deterministic instructions

The rewritten best practices page divides every instruction into two kinds. A non-deterministic instruction is one the model reads and then decides what to do with (the prompt, CLAUDE.md, skills Claude picks up itself); it is usually followed, but there is no guarantee. A deterministic one is executed by the harness, Claude Code (hooks, deny rules, managed settings), and applies no matter what the model decides. The page lists the two forms of the same thing side by side. For deciding whether the work is done, for example, the non-deterministic forms are a request for evidence, `/goal` and `/code-review`, and the deterministic forms are tests, type checks, a `Stop` hook and CI. For controlling how much it spends, the non-deterministic form is the effort level, and the deterministic forms are `--max-turns`, `--max-budget-usd` or a spend limit you set.

The official trade-off is **let Claude decide most of the time**, which is the context of the `Almost every guideline has exceptions` line in Section 4.5. Keep the deterministic kind for things with zero exceptions: `a hook is for a rule with zero exceptions`, and a preference that has exceptions (the official example is `Prefer named exports`) stays in CLAUDE.md. The reverse holds too: if you have a hook for something that is only a preference, delete it and let Claude decide. The page also warns to keep the enforced set small: `If you ever find yourself at forty of them, most are probably preferences that Claude could have weighed on its own.`

The anti-patterns the Steering article lists all make sense in this frame:

- ✗ "Every time X, always do Y" in CLAUDE.md → use a hook; the model choosing to run a formatter is not the same as the formatter running automatically
- ✗ "Never do this" constraints → use a `PreToolUse` hook or permission configuration. The article's wording: `A real guardrail needs to be deterministic`
- ✗ A 30-line procedure in CLAUDE.md → move it to a skill
- ✗ An API-specific rule with no path scoping → add `paths:` frontmatter
- ✗ Personal preferences in a shared project file → put them in `CLAUDE.local.md`, and add that to `.gitignore`

### 8.3 CLAUDE.md: only what it can't see

The tip at the start of the best practices page's "Tell Claude what it can't see" section is `Only write down what Claude can't find by reading the repo, or what it would otherwise spend time rediscovering every session`. When Claude starts a task it reads the code first, and it finds the file layout, the conventions and the test framework on its own. What it cannot find is what was never in the repo: a decision the team made in a meeting, a deploy process that lives in one person's head, the environment variables the integration tests need. Before you write anything down, the official test is two questions: could it find this by reading the code? Would it actually go and look? If both answers are yes, leave it out.

**What belongs:** how to confirm work in this repo (the exact test, type check and lint commands, and how to run a single test file); places where the code is misleading (an old pattern that is everywhere but that new code must not copy, a directory that looks unused but is in use); decisions that are not in the repo; anything you have already had to correct twice. When the repo already has a check that runs, the single most useful line in CLAUDE.md is the command that runs it.

**What does not:** anything Claude can work out by reading the code. Describing architecture and file layout in CLAUDE.md is on the list of old practices you can stop, because Claude reads the repo itself.

**When one line keeps getting ignored:** earlier versions of this document said here to add IMPORTANT to that line alone. The current page says adding emphasis mostly does not help: either shorten the file, or make it a hook. It also gives a signal to watch for: `If you keep adding emphasis to a CLAUDE.md line and Claude still misses it, that's a sign the line belongs in a hook.` An example on the same page shows the difference between the two kinds of instruction well. The type-check command line in CLAUDE.md is there so that Claude can check for itself. If what you want is that it may not finish until the check passes, that is a job for a `Stop` hook; a line like `Always typecheck before you finish` only works most of the time.

**Tools:** `/init` generates a starting version from the codebase, after which the work is mostly removing lines; `/doctor` suggests cutting what Claude can work out from the code; `/doctor prompt-audit` flags instructions written for older models, instructions that point at files that no longer exist, and instructions that contradict each other; in a session, `/context all` confirms the file was actually loaded.

### 8.3b Writing skills: three-tier progressive disclosure

The entire architecture of a skill exists to do one thing: **put each layer of information into context only when it is needed.**

| Tier | Content | Loaded |
|---|---|---|
| **1. Metadata** | YAML frontmatter, only `name` and `description` | Preloaded into the system prompt at startup |
| **2. Primary instructions** | The `SKILL.md` body | When Claude judges the skill relevant |
| **3. Bundled files** | Files referenced from SKILL.md | Only when that specific scenario arises |

The official analogy is apt: **like a well-organized manual: table of contents first, then the chapter, then the detailed appendix.**

Key points:

- **The CLAUDE.md rule applies to skills too**: `only write down what Claude can't find on its own` (Claude Code best practices)
- **`name` and `description` are the two critical fields.** They are what Claude uses to decide whether to activate the skill. Make sure they say clearly what the skill does and when it should be used
- The intended shape is a **lightweight guide that helps Claude find information when it needs it**, not an exhaustive procedure. In the words of the Section 1 post: `Avoid making them overconstrained, except in highly important areas`
- When the body grows unwieldy, **split the file**. Separating mutually exclusive scenarios saves real tokens. The official example: the PDF skill puts form-filling instructions in a separate `forms.md`, keeping the body lean, and Claude reads it only when a form actually needs filling
- **Bundle executable code where it helps.** The reason is concrete: sorting a list via token generation is far more expensive than simply running a sorting algorithm. Give deterministic operations to code
- **Start from evaluations**: measure the capability gap first, then fill it with a skill, rather than imagining requirements in advance
- **Let Claude help write it.** If it goes off track while using a skill, ask it to reflect on what went wrong; that shows you **what context it actually needs**, instead of guessing in advance
- **Security:** install skills only from trusted sources. If the source is uncertain, audit the bundled files, the dependencies, and any instruction that reaches the network

Combine this with Section 6.2: the Fable 5 page states that an over-prescriptive skill written for an older model **can actively degrade** its output quality (measured on Fable 5), and when you migrate to Fable 5.2, skills and agent files are within the scope of its review list. So a skill is not better just for being more detailed.

### 8.3c Turning manual checks into verification loops

The test is simple: **any check you keep having to run by hand is a candidate for hardening into a loop.** Especially deterministic rules a generic linter will not catch; the official example is `Reject any migration that drops a column without a backfill step`.

Four deployment modes:

| Mode | Trigger | Best for |
|---|---|---|
| **Standalone** | Invoked deliberately | Cross-cutting checks that do not apply universally: security scans, accessibility audits, license verification |
| **Embedded** | Fires automatically inside the producing skill | Checks that belong to one specific task |
| **Chained** | One skill calls another on completion | End-to-end handoff; also how you add verification to a **skill you cannot modify** |
| **On every PR** | CI | Turns a personal habit into team infrastructure, removing dependence on author diligence |

The trap: **embedding only works on editable skills.** Built-in or plugin-managed skills get overwritten when they update, taking your change with them; those must use chaining.

The creation flow: pick the manual follow-up you do most often → try the built-in `/verify` first → write the procedure in plain language, as you would for a new teammate on day one → hand it to skill-creator, or put the markdown file in `.claude/skills/` yourself → confirm on a new task that the check actually runs → then try chaining.

### 8.3d Four kinds of loops

| Type | Command | Best for |
|---|---|---|
| **Turn-based** | Manual, turn by turn | Short tasks |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | Tasks with a **verifiable exit criterion** where one turn is not enough |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | Recurring work, interfacing with external systems |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | Well-defined recurring streams: bug reports, issue triage, migrations |

Cost control: **use scripts for deterministic work** (running a script costs less than reasoning through it token by token), match intervals to how fast things actually change, and pilot before running at scale. In the article's words: `Your model and effort level choices are among the biggest levers on what a loop costs.` The Claude Code best practices add that each `/loop` run is another turn in the same session: `Each run is another turn in the same session, so don't leave one going in a long conversation.`

### 8.4 Give Claude a check it can run itself

The rewritten best practices page opens by setting three things side by side: `tell Claude what it can't see, give it a check to run, and enforce the rules that can't have exceptions`. The first is 8.3 and the third is 8.2; this section covers the second:

> Claude stops when the work looks done. If it has nothing to run that tells it whether the work actually *is* done, "looks done" is all it has to go on. In practice that makes you the test suite, and every mistake sits there until you happen to notice it.

In other words: without a check it can run, "looks done" is all it has to go on, you become the test suite, and every mistake waits until you happen to notice it. Give it something whose result is pass or fail, and it can close the loop itself: make the change, run the check, read the output, fix it, run the check again.

The check can be a test file, a build's exit code, a linter, a script that diffs output against a fixture, or a browser screenshot next to the mockup.

**Ask it for evidence**: which command it ran and what the output was, or a screenshot. The official words: `"All tests pass" on its own tells you very little`. Reading the output is faster than rerunning the check yourself, and for a session you weren't watching, it is the only evidence there is. After Claude says it is done, you can also run `/verify` in the session: it builds and runs the app and looks at the result, which catches the kind of problem that tests and type checks miss.

Once the check exists, there is a question the prompt cannot answer by itself: **how hard does it gate the stop?** The official material gives four levels, from Claude deciding for itself to Claude Code enforcing it:

| Level | What it is | What the docs say |
|---|---|---|
| In the prompt | `run the auth tests after, and keep going until they pass` | Claude decides for itself when it is satisfied; works for any task, with no setup |
| As a goal | `/goal all tests in tests/auth pass and lint is clean`: after every turn, a separate model checks the condition against the conversation | Rarely needed while you are watching yourself; it matters for long background runs, where Claude sometimes ends a turn with a progress summary while items are still open, and the goal sends it back |
| A `Stop` hook | Runs your script when Claude tries to end its turn, and sends it back to work if the script exits with code 2 | Enforced by Claude Code. If the hook blocks eight times in a row with no tool call in between, Claude Code lets the next stop through; `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` changes this cap |
| Review by someone who didn't write the code | `/code-review` reviews the diff in a fresh subagent that never saw your conversation | It isn't swayed by the reasoning that produced the code |

When the user describes a long run nobody is watching, the skill adds the official line to the delivery note: runs like that are enforced with `/goal` or a `Stop` hook.

To have the review check the work against the plan rather than hunt for bugs in general, write the review prompt yourself. The official example:

```text
use a subagent to review the rate limiter diff against SPEC.md, check every requirement is implemented and the listed edge cases have tests, report gaps that affect correctness and skip style
```

The docs specifically warn against dropping `skip style`: `A reviewer that's asked to find problems will always find something.` Fix every finding and you end up with extra abstraction layers and tests for cases that can't happen. In this official example the subagent reviews and reports straight back to you. Snippet 4.16, from the "Code review harnesses" section of the Sonnet 5 page, goes the other way: have the reviewer report everything, and move filtering by confidence and severity out of the finding step; that page says the prompt can be used without an actual second step (see also the code-review row in Section 6.1). The Opus 5 page also warns that when a severity bar is set, the model `might follow that instruction literally and report less`; the Opus 5.5 pages neither restate nor retract that tendency.

### 8.5 Writing the prompt itself: say it all at once

The best practices page says a specific prompt helps for the same reason CLAUDE.md does: it gives Claude information it did not have. Naming the file, describing the symptom, and saying up front what fixed looks like usually saves a round of corrections later. The same goes for a preference between two reasonable approaches: Claude cannot read your preference from the code.

| Instead of | Try |
|---|---|
| `fix the login bug` | `users report login fails after the session times out. look at token refresh in src/auth/, write a failing test that reproduces it, then fix it` |
| `add a calendar widget` | `follow the pattern in HotDogWidget.php and build a calendar widget that lets you pick a month and page between years. no new libraries` |

If you already have what it needs, hand it over instead of describing it: `@` plus a path brings in a file's contents; a screenshot can be pasted straight into the prompt; command output can be piped in from the shell, for example `cat error.log | claude -p "what's going on here"`.

There are three official notes on wording. Phrases like `think step by step` and `be thorough` are ordinary prompt text, not keywords; the main control for how deeply it thinks is effort (Section 6.4), and such wording can only nudge it within the current level. One phrase that still helps is `be ambitious`: Claude tends to read a request conservatively and build the smallest thing that satisfies it, so if you want the fuller version, say so. That tells it about **scope**, which only you know:

```text
build the settings page from the mockup. be ambitious: empty and error states, the save flow wired up, keyboard navigation, not just the layout
```

When you are exploring rather than fixing something specific, a vague prompt like `what would you improve in this file?` is completely fine; the cost is a few more corrections along the way.

**When the requirements are not thought through, have Claude interview you:**

```text
I want to add rate limiting to the public API but haven't thought it through yet. Interview me until you could write the spec, then write it to SPEC.md.
```

Claude asks multiple-choice questions about the parts that are not pinned down, and keeps asking until the answers add up to a spec. Then **start a fresh session** to build from `SPEC.md`: the context is clean, and there is a written definition of done. It is also the file you point the reviewer at later (the review prompt in 8.4).

### 8.6 Old practices you can stop, and before you let it run

**"What you can stop doing".** That section opens with `Every one of these was good advice when it was written`: most were patches for things the models could not yet do at the time. When reviewing a user's Claude Code prompt or CLAUDE.md, the skill checks it against this table:

| Old practice | What the docs say now |
|---|---|
| Treating the context window as the main constraint and managing it aggressively | Most current models have a 1M token window, compaction happens on its own, and running out is rare. `/clear` between tasks is still worth doing |
| Adding IMPORTANT or all caps to instructions that keep being missed | Current models follow a plainly worded line, and piling on emphasis mostly makes them over-apply the rule |
| Using subagents to protect the context window | Subagents still suit parallel work, keeping noisy output out of the way, and independent review, but each of those costs extra tokens |
| Delegating to a subagent for a better answer | Handing the same task to a subagent does not make the answer better; a fresh reviewer helps because it has not seen your reasoning |
| `think hard`, `think harder`, `be thorough` in the prompt | That is effort's job; `ultrathink` still works, for that one turn |
| Telling it to keep going, loop until done, don't be lazy | Current models usually carry an agreed task through without being told, and lines like that are more likely to cause over-work. Giving it a check to pass is a different thing |
| Describing architecture and file layout in CLAUDE.md | Claude reads the repo itself |

**Five questions before you let it run on its own** ("Let Claude run without you"): Does it know what done looks like (a goal condition, a spec file, a failing test it has to turn green)? Can it check its own work? What is it allowed to touch? What stops it (`--max-turns`, `--max-budget-usd`, a spend limit)? Where does the result go, and who signs off? The official reasoning is plain. While you are watching, Claude missing a bit of context does not matter much, because you notice within a message or two. In a run started before bed, nobody tells it, and the next morning you find it spent the whole night building on a wrong assumption.

**What subagents cost** ("Know what subagents cost"). Four situations where they are worth it: running independent pieces of work at the same time; keeping long output out of the main session; getting a reviewer that has not seen how the code was written; giving a task fewer tools than the main session has. The cost is mostly tokens, and sometimes time: a fresh subagent first has to gather context the main session already has. For ways to save, see Section 6.6.

**When a session does not go as expected**, the page opens with a diagnosis. If it built something other than what you had in mind, it is most likely missing information it cannot get by reading the repo (8.3). If it said it was done and it was not quite, it most likely had no way to check its own work (8.4). If it did something that must never happen, or skipped a step that must always happen, a prompt alone cannot guarantee that (8.2). If it got it right but spent too much, look at `/usage`. Three controls in a session: `Esc` stops it mid-action and keeps everything up to that point; `/rewind` rolls the code, the conversation, or both back to an earlier prompt; and if you have corrected the same thing twice and it is still wrong, run `/clear` and fold what you learned into one new prompt, since a third correction mostly adds to a conversation that already contains two failed attempts.

---

## 9. How to practice

### 9.1 A four-week path

**Week 1 — detox.** Print the Section 4 table and keep it in front of you. Go through the prompts you already have and delete what is obsolete, line by line. If you are migrating to Fable 5.2, also go through the "Review instructions written for earlier models" list on its page (skill 3.13).

**Week 2 — practice being clear.** Run every prompt you write through two tests:

1. Would someone with minimal context on this task be able to follow it? (The best practices page's test)
2. **How would a well-intentioned human misinterpret it?** This one comes from Cat Wu of the Claude Code team, in a public conversation in 2026-07; it is an employee's remark, not official documentation: `you should think about the ways in which it could be misinterpreted by a well-intentioned human, in order to better understand how the model might interpret it`. The skill keeps it because the Fable 5.2 pages state that rules are applied as written (Section 1)

**Week 3 — practice subtraction.** Take the longest prompt you have written, try cutting it in half, and run the same task both ways. This is how you build the intuition for scaffolding becoming dead weight; go by the comparison, not by feel.

**Week 4 — practice solving it one layer out.** Every time you want to add a rule, ask first whether the problem can be solved without a prompt. A different tool design? A deterministic check? A change in how the information is presented?

### 9.2 Six questions for judging a prompt

1. Is there a decidable success criterion?
2. Does every constraint carry its reason?
3. Is there any sentence whose removal would change nothing? (Then remove it.)
4. How would a well-intentioned human misinterpret it? (See week 2.)
5. Does this really need to load **every time**? (If not, push it down into a skill or on-demand file.)
6. Should this rule live in the prompt at all, or should a tool, schema, or hook enforce it deterministically?

### 9.3 The one durable habit

> **Every time the model gets an upgrade, go back and delete part of your prompt.**

Lance Martin's version: as Claude improves, the assumptions in a harness expire, and scaffolding that was once necessary becomes dead weight that can bottleneck performance. So **re-evaluate continuously.**

That is probably the only advice here that will not go out of date, because what it describes is how to handle things going out of date.

---

## 10. Sources and a reading order

The full source list, the read depth for each, and the adjudication of known conflicts are in [SOURCES.md](../SOURCES.md). What follows is only the reading order.

### A suggested reading order

1. **Start here:** [The new rules of context engineering for Claude 5 generation models](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/) (2026-07-24) — a seven-minute read, and it orients everything else
2. **Then:** [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (official docs, continuously updated, and rewritten from top to bottom between this repository's 2026-10-23 and 2026-11-09 checks) — the densest practical material available
3. **Consult as needed:** [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) plus the page for the model you use plus [Choosing the right model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model) — reference material, not a read-through
4. **For the underlying principles:** [Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents) (2025-09)
5. **To hear it from the people:** [Fireside Chat: Cat Wu & Thariq](https://www.youtube.com/watch?v=uU5Gv2h8-9g) (2026-07), with a [transcript](https://simonwillison.net/2026/Jul/21/cat-and-thariq/). This is a public conversation between employees, not official documentation. This document and the skill keep only one test of Cat Wu's from it (Section 9); everything else from it was removed in the 2026-11-09 round
6. **Read with care:** [Best practices for prompt engineering for 2026](https://claude.com/resources/articles/best-practices-for-prompt-engineering) (2025-11). Its framing sections and its `Common mistakes to avoid` list are still worth reading, Section 3.4 takes a useful corollary from it, and the skill's line that lets research prompts say they do not know is the second sentence of its example. But it **contradicts the current documentation in three places**: it recommends prefill (now rejected by the API); it says XML tags are `less necessary with modern models` (the docs still recommend them); and it suggests structured chain of thought in `<thinking>` tags in the reply, even when extended thinking is available (the best practices page says a prompt like that `might be declined` on Fable 5.2, Opus 5.5 and Sonnet 5.5). On those three, the docs win. A fourth point only looks like a conflict: the post calls heavy-handed role prompting often unnecessary, while the docs recommend a short role. The post argues against long personas, not against roles, so the two stack (Section 3.4). The post also suggests starting with one example and adding more only if the output still falls short, where the best practices page says to include 3–5 for best results; the skill follows the page (Section 5)

### ⚠️ Deliberately excluded

| Excluded | Why |
|---|---|
| [prompt-eng-interactive-tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) | Still targets **Claude 3 Haiku**, and the techniques it teaches most prominently have since been reversed. **In fairness:** the official overview page still lists it as the entry point for interactive learners, so it has not been officially deprecated. But that page now notes that it `was written for Claude 3 models, so follow Prompting best practices where the two differ`, and the same page states plainly that Prompting best practices is `the living reference; start there`. Fine as a conceptual introduction; on every specific technique, defer to the living reference |
| [AI prompt engineering: a deep dive](https://www.youtube.com/watch?v=T9aRN5JkmL8) (2024-12) | The most widely reshared online. The mental models still hold; the tactics have been superseded |
| Third-party "ultimate guide" articles on prompting Claude | Secondhand summaries |

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
- **The effort page had only been skimmed.** Reading it in full added that lower effort reduces tool-call count, that `high` equals omitting the parameter, and the per-model differences. (The `ultracode` aside in Section 6 does **not** come from this page — it was later traced to the Claude Code dynamic-workflows page, and is cited there now.)
- **The overview page had not been read at all**, which is where Section 3.0's preconditions come from, along with `Not every success criterion or failing eval is best solved by prompt engineering`
- **Section 5b is new.** The apparent contradiction between Opus 5's advice to remove verification instructions and Fable 5's advice to make self-verification explicit was not addressed in the original draft at all
- **Four sources had only thin summaries**: the Agent Skills engineering post (never fetched), Agent Harness Design, verification loops, and loop engineering. Reading them produced the three-tier progressive disclosure model, the five cache-hit principles, the three orchestration figures, the four verification-loop deployment modes, and the loop commands
- **The Console prompt improver link was dead** (it redirected), and the page it redirected to still advertised **prefill, which the API now rejects** — a trap someone would walk straight into. That replacement page has since been withdrawn as well: the redirect now lands on Prompting best practices, which tells you to migrate away from prefill. What this document originally took from the withdrawn page had to be re-founded on something that still exists; see [SOURCES.md, "The 2026-08-08 re-check"](../SOURCES.md#the-2026-08-08-re-check)
- Scope was then trimmed: guidance specific to earlier models was removed rather than left in as "probably still fine"

### The 2026-11-09 round: a third model release, all four current models in scope, unofficial content removed

Claude Fable 5.2 and Claude Mythos 5.2 shipped on 2026-11-09, and Fable 5.1 moved to the "Legacy models (still available)" list. This round repeated the procedure of the previous two: nothing renamed, every Fable-specific claim adjudicated against the Fable 5.2 pages. The difference this time was that there was no carry-over sentence to rely on. The Fable 5.1 page says that existing Fable 5 prompts should perform well without changes; the Fable 5.2 pages say only to review your prompts when you migrate, and its prompting page reprints none of the older Fable-labeled blocks verbatim. So all the Fable 5.1 and Fable 5 material was relabeled "measured on that model", and older claims on which the Fable 5.2 pages say something different were dropped (encouraging parallel subagent dispatch, one brief instruction steering a class of behavior, over-delivering on open-ended features and unrequested tidying at high effort, describing the next step on complex async tasks instead of doing it, periodic self-checks on long runs, the slowest latency, going quiet for minutes in long turns, gains that grow with the effort level). Fable 5.1 itself left scope: no page recommends it over Fable 5.2 on any point, and the measurements on the cost page that point the other way are recorded in the skill's models.md, together with the advice that sits right beside the research figures, `Measure what a more expensive model buys on your own workload` (Section 6.2). Every page was fetched again, and the old skill was split into 869 claims, each audited against the saved pages. The skill's snippet library now has 52 entries, 51 of them verbatim, in 75 fenced blocks; 4.37 to 4.52 are new.

**Sonnet 5.5 and Haiku 5.5 came into scope.** The skill now chooses models across the whole current lineup, subagents included, so these two models got their own tables and snippets, and the hard rules in the skill's Section 3 became a per-model table. There are three differences: Sonnet 5.5 turns off up-front thinking with `between_tools`; Haiku 5.5 can turn thinking off at `high` and below, accepts forced `tool_choice`, and has no `reasoning_extraction` category. Every sentence in Section 4 of this document of the form "both models return 400" was rewritten as a result. Choosing model and effort was also rewritten, with the main session and subagents treated separately, and gained official figures for prices, usage limits and ways to save usage (Sections 6.5 and 6.6). The skill first had four tiers by task, one model per tier. A review the same day replaced them with the structure of the model-selection article Anthropic published that day, "Which Claude model should you build with?": a task table taken row for row from its "Pick a model by task", a table of effort changes for each model, the order of steps when a model falls short with the places where the official sources disagree, and a subagent table. The old two-condition test for what counts as agentic work heavy enough had no source, and was removed.

**Unofficial content was removed.** The repository owner asked for everything without an official source to be removed. In this document that meant: the two quotations from Thariq Shihipar's conversation in Sections 1 and 5, the sentence of his borrowed in Section 3.2, and what was derived from them (giving no examples in agent system prompts, and the two positions not really conflicting); the whole reconciliation of verification instructions in Section 5b (remove self-review but keep external checks, measure before deleting on Opus 5.5, the clean-context verifier reading, periodic self-checks on long Fable runs); Cat Wu's remarks in Section 4.5 about rules that are 90% true and about always verify; the passages in Section 7.8 where the Claude Code team talked about the grep / glob tools, the file-editing tool and tool design being more like biology, along with the fix-the-tools-first and low-cardinality framings; the observation in Section 3.3 that every tagged block is a standing policy; the characterization-test procedure in Section 8; and a few judgments with no basis (*the highest-leverage item in this document*, *this step usually yields the biggest return*, *you will usually find the quality did not drop*). Only one employee remark is kept: Cat Wu's test, asking how a well-intentioned human would misinterpret a prompt. It is labeled as an employee's remark, and it is kept because the Fable 5.2 pages state that rules are applied as written. Skill Section 2 has three rules of the repository owner's own, marked as user rules (User rule, not official guidance; confirmed 2026-11-09), and this document labels them the same way when it mentions them (Section 6.5). The full list of what was removed from the skill itself is in [SOURCES.md](../SOURCES.md#the-2026-11-09-re-check).

**The Claude Code best practices page was rewritten from top to bottom.** The explore / plan / implement / commit section, the section on five common failure patterns, the CLAUDE.md diagnostic sentence this document used to quote, and most of its example prompts (including the validateEmail contrast table and the interview prompt) are gone. New are deterministic and non-deterministic instructions, the cost of subagents, unattended runs, and the "What you can stop doing" table. Two items in that table are things this document and the skill used to recommend: adding IMPORTANT to the one line Claude keeps missing, and describing architecture and file layout in CLAUDE.md. The CLAUDE.md exception in Section 4.3 was reversed as a result, and Section 8 was rewritten against the new page. The full record is in [SOURCES.md](../SOURCES.md#the-2026-11-09-re-check).

**The owner's subagent draft, and the source list.** The repository owner had an uncommitted draft on running batch subagents on a different model from the main session, and each sentence in it was checked against the Claude Code documentation. Written as documented: the order in which a subagent's model is resolved; per-invocation `model` (aliases only) and `effort` (Claude Code 2.1.292 or later); Explore running on the model the `opus` alias points to when the main session is on Fable and signed in with a subscription; the concurrent limit of 20 and how to change it, which replaced the draft's "about 8"; and what a subagent returns when an API error such as a usage limit ends it. Corrected: the draft said definitions under `.claude/agents/` are cached at session start, while the documentation says an edit is detected within a few seconds and the next delegation uses it. Removed, because the documentation does not mention them: progress files as checkpoints for a batch, per-batch model records and stratified checks, and the BLOCKED convention. (An Anthropic article does recommend a task-list file for long Opus 5.5 runs, so that the list survives compaction; the skill's for-claude-code.md has that tip as the article states it.) Kept as a labeled user rule at the owner's decision: in bulk tasks the main session and the batch subagents get their models separately; the specific models and levels were replaced on 2026-11-10 by a table taken from the official benchmark (Section 6.5). The source list changed too: claude.dev/blog now counts as an official source, while pages under claude.com/docs and the academy tutorial no longer do, so the skill's Chat and Cowork file was rewritten from support.claude.com and its Chat-versus-Cowork table was removed. What no official page confirms (for example, what happens to running subagents when the main session reaches a usage limit, which only two changelog notes touch on, or how many subagents to run at once), and the section-by-section coverage table for the Fable 5.2 pages, are in [SOURCES.md](../SOURCES.md#the-2026-11-09-re-check).

**The review the same day.** There were three rounds of independent review in all, with 92, 100 and 75 findings; this document was reviewed in the second and third, and the fixes from the third round were afterwards checked line by line against the pages, with no fourth round. The skill was reviewed again after this document was written, and this document was corrected to match in these places. Sections 6.2 and 6.5 and this record used to attach the cost page's `On research tasks, measure the model you ship` to the research-benchmark figures for Fable 5.2 against Opus 5.5; the line beside those figures is actually `Measure what a more expensive model buys on your own workload`, and the first line is about effort levels. Section 7.1's heading used to say "The binding constraint", while the rewritten Claude Code best practices list `The context window is your main constraint, manage it aggressively` among the old advice you can stop following. Section 8.4 used to say a prompt can only reach the first of the four levels, and Section 7.7 used to say the Edit tool's checks are the staleness-check case; both claimed more than the pages say. And a few places that said "makes", "cause" or "has no effect" went back to the pages' own strength (`can make`, `can cause`, `changed nothing`).

### The 2026-10-23 round: no new model in scope, but a month of quiet edits

No model entered scope this time. Claude Sonnet 5.5 (2026-09-28) and Claude Haiku 5.5 (2026-10-07) shipped with their own prompting pages and Sonnet 5 joined the legacy list, but the default recommendation did not move — the selection matrix still opens with "Most workloads start with Claude Opus 5.5" — so the skill keeps both new models out of scope, with a note on where its hard rules do not transfer to them.

What changed was the pages already tracked, mostly without a release note. Thirteen official sentences this repository quoted had been reworded, from one-word edits ("finish" became "complete", "may" became "might") to model lists that grew, and one had disappeared along with the page it was on. One quoted block, the Fable 5 final-report readability snippet, lost a parenthetical. Two of the edits changed facts rather than wording. The `medium` effort default is no longer unique to Opus 5.5 — Haiku 5.5 shares it — so every "`high` on every other model" sentence had to go. And `reasoning_extraction` is no longer described as new on Opus 5.5: the what's-new page now introduces it with "As on Claude Opus 5", so of the classifiers it names, only the biology one is new for someone coming from Opus 5.

One piece of advice in this document was withdrawn rather than reworded. Section 5's table used to say a reasoning pattern is best demonstrated with `<thinking>` tags inside few-shot examples. The documentation no longer says that — it says to present each worked example as the problem, the method to apply, and the expected answer — and the refusals page now names a `<thinking>` section the model fills in as a common source of `reasoning_extraction` declines.

The round also found several errors of this repository's own; three are worth recounting here. Section 6.2 still gave the pre-Opus-5.5 selection advice ("start with Claude Opus 5"), and Section 4 still described the Opus line as unaffected by the forced-tool-use change: the 2026-09-22 round had swept the skill and the single-file editions for leftover Opus 5 claims, but not this document. The sentence the 2026-09-22 round relabeled as the repository's own — `If the data is insufficient to draw conclusions, say so rather than speculating.` — is in fact the second sentence of the example in Anthropic's 2025-11 prompt engineering post, and is now credited to it. And the claim that "the official guidance for output-format skills is that research comes first" turned out to have no source at all; the advice stays, labeled as the skill's own. The full record is in [SOURCES.md](../SOURCES.md#the-2026-10-23-re-check).

### The 2026-09-22 round: a second model release, and a rule that did not survive it

Claude Opus 5.5 shipped on 2026-09-22 and Claude Opus 5 joined Fable 5 on the "Legacy models (still available)" list. The procedure from the Fable round was repeated: nothing renamed, every Opus-specific claim adjudicated against the Opus 5.5 pages, and anything without explicit Opus 5.5 backing either moved into a "measured on Opus 5" table or marked unconfirmed.

**The finding worth reading is Section 4.4.** "Remove self-verification instructions" had been one of this document's headline rules, and it does not carry to Opus 5.5 as a rule. The Opus 5.5 prompting page and what's-new page say nothing about over-verification either way; the best practices page still names Claude Opus 5 as the single exception to its general "Ask Claude to self-check" advice; and the Opus 5.5 migration guide offers only `Instructions tuned for Claude Opus 5's behavior might no longer be needed`. A global find-and-replace from "Opus 5" to "Opus 5.5" would have turned a documented, model-specific measurement into a confident claim about a model nobody had measured. That is the whole argument for adjudicating rather than renaming, in one rule.

Three other things changed shape. The effort default became model-specific (`medium` on Opus 5.5), which invalidates the old "both models start at `high`" framing and the `effort: "high"` ≡ omitted shorthand. Three hard rules widened to cover Opus 5.5's breaking changes — thinking cannot be disabled, forced `tool_choice` returns 400, and the `reasoning_extraction` refusal category now names Opus 5.5 on the thinking page. And the progress-update mechanism changed: text between tool calls now arrives in `thinking` blocks that are empty by default, so a client written for Opus 5 goes quiet through a long agentic turn without any error to point at.

The round also flagged one thing that was this document's neighbour rather than its own: a fenced block in the skill's chat-and-Cowork reference — `If the data is insufficient to draw conclusions, say so rather than speculating.` — sat immediately below an official block introduced as wording to copy verbatim, with nothing marking it as different. The round concluded it appears on no Anthropic page and labeled it the repo's own sentence. That conclusion was itself wrong — the sentence is the second sentence of the example in Anthropic's 2025-11 prompt engineering post, which the round had not searched — and the 2026-10-23 round corrected the label. The full record is in [SOURCES.md](../SOURCES.md#the-2026-09-22-re-check).

### The 2026-09-03 round: a model release re-opened every Fable claim

Claude Fable 5.1 shipped on September 1, 2026, with its own prompting page, and Fable 5 moved to the legacy list. Rather than renaming, every Fable-specific claim in this repository was adjudicated against the Fable 5.1 pages: kept where the Fable 5.1 page republishes or carries it over, rewritten where the Fable 5.1 page says something different, retired where the Fable 5.1 page supersedes it (the autonomous-operation block of snippet 4.18), and — for behavioral claims the Fable 5.1 pages neither restate nor retract — kept with an explicit "measured on Fable 5" label rather than silently carried forward. The record of what was checked, which pages moved, and which rules changed is in [SOURCES.md](../SOURCES.md#the-2026-09-03-re-check). Two attribution errors surfaced in passing: the no-preamble string is on the prompting best practices page, not the migration guide, and the "most capable widely released model" phrase lived on the choosing-a-model page, not the models overview (it has since been reworded there).

### A real lesson: the documentation changed mid-verification

Worth recording in full, because it makes the point better than any of the entries above.

On the Opus 5 effort starting point that Section 6.4 mentions, this document first said that coding and agentic work should start at `xhigh`, on the basis of this wording on the official Opus 5 page: `for coding and agentic work, xhigh remains the recommended starting point`.

During review, two independent passes reported the page said "start at `high`" instead. Three direct fetches of the same page all returned the `xhigh` sentence, and the effort page had no Opus 5 section at the time. The two reports were recorded as unreproducible.

**They were not wrong. The page had changed.** The changed page then read:

> "**Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency of higher settings. Start with the default (`high`) and adjust based on your evals: use `low` and `medium` liberally as your primary control for token cost and response time wherever quality holds, and step up to `xhigh` for demanding coding and agentic work."

The phrase `remains the recommended starting point` no longer appears anywhere on the page, and the effort reference has since gained per-model recommendation sections for Opus 5, Fable 5 and, as of September 2026, Fable 5.1 and Opus 5.5. This document and the skill have both been corrected.

**Two takeaways:**

1. **The difference is material,** not cosmetic. An ordinary coding task moved from `xhigh` to `high` as its starting point, which changes what it costs.
2. **For fast-moving documentation, a snapshot is wrong by default, not by exception.** For anything you are using to plan cost or latency, do not rely on a secondhand summary — including this one. Confirm it against the source page, which is the only surface that tracks model updates.

---

*Apart from the one employee remark and the three user rules that are labeled as such, every technical claim here traces to an official source listed in [SOURCES.md](../SOURCES.md). Where this document and something you read elsewhere disagree, check the publication or revision date of the official page first. Corrections are welcome as issues — the live docs are the authority, not this repository.*
