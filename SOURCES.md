# Sources

Snapshot date **2026-07-25**. Anthropic's documentation is revised frequently — where this repo and the live docs disagree, **the live docs are correct**.

Read-depth legend:

- **Full** — the page was fetched and read verbatim end to end; snippets were extracted programmatically from it
- **Targeted** — specific sections were extracted and quoted; the rest was not used

---

## Living documentation (tracks model releases)

| Source | Depth | What was taken |
|---|---|---|
| [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) | **Full** | Most of the snippet library; long-context placement; XML structuring; the few-shot guidance; the migration section |
| [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) | **Full** | Verbosity, narration, written-deliverable length, scope, subagent damping, self-correction, thinking-disabled artifacts, effort guidance |
| [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) | **Full** | Long-run autonomy, progress truthfulness, `reasoning_extraction`, memory conventions, the send-to-user tool, scaffolding advice |
| [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) | **Full** | The five effort levels and their stated use cases, the per-model recommendation sections for Opus 5 and Fable 5, effort's effect on tool-call count, the `high` ≡ omitted equivalence, cache invalidation |
| [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) | **Full** | The three preconditions before prompt tuning; "not every failing eval is best solved by prompt engineering" |
| [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) | **Full** | The verification-affordance rule, explore/plan/implement, specificity contrasts, CLAUDE.md guidance, the five failure patterns, the adversarial-review caveat |
| [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | **Full** | The constraints this repo's own skill is built to (name/description limits, one-level references, TOC threshold, progressive disclosure) |
| [Console prompting tools](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-tools) | **Full** | The prompt improver's documented output — including the "Strategic prefills" caveat |
| [Claude Cookbook](https://platform.claude.com/cookbook) | Targeted | Metaprompt, prompt caching, context-engineering recipes |

> Note: the previously published `.../prompt-engineering/prompt-improver` path now redirects; `prompting-tools` is the current location.

## Engineering blog

| Source | Date | What was taken |
|---|---|---|
| [The new rules of context engineering for Claude 5 generation models](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) | 2026-07-24 | The six rules; the >80% system-prompt reduction; `/doctor`; skills as lightweight guides |
| [Steering Claude Code: CLAUDE.md, skills, hooks, subagents](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more) | 2026-06-18 | The seven-mechanism comparison table; the anti-patterns; the 200-line CLAUDE.md guidance |
| [Building verification loops in Claude Code with skills](https://claude.com/blog/building-verification-loops-in-claude-code-with-skills) | 2026-07-22 | The four verification-loop modes; the editable-skill caveat |
| [Loop engineering: getting started with loops](https://claude.com/blog/getting-started-with-loops) | 2026-06-30 | The four loop types and their commands; cost levers |
| [Agent Harness Design: 3 patterns](https://claude.com/blog/harnessing-claudes-intelligence) | 2026-04-02 | Let-the-model-orchestrate (45.3% → 61.6% on BrowseComp); cache-hit design; when to promote a bash action to a dedicated tool |
| [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 2025-09-29 | Context rot; the right-altitude principle; compaction, structured note-taking, sub-agent architectures |
| [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) | 2025-09-11 | Tool naming and namespacing, token-efficient returns, `ResponseFormat`, actionable errors |
| [Equipping agents with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | 2025-10-16 | Three-tier progressive disclosure; why bundle executable code |
| [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | 2025-11-26 | The initializer/coding-agent split; the feature-list JSON; why unit tests alone miss end-to-end breakage |
| [Prompt engineering best practices for 2026](https://claude.com/blog/best-practices-for-prompt-engineering) | 2025-11-10 | **Used with care** — see "Known conflicts" below |
| [Cowork overview](https://claude.com/docs/cowork/overview) · [Desktop app modes](https://claude.com/resources/tutorials/navigating-the-claude-desktop-app) | — | The chat / Cowork / Claude Code capability boundaries |

## Employee talks

| Source | What was taken |
|---|---|
| [Fireside chat: Cat Wu & Thariq Shihipar](https://www.youtube.com/watch?v=uU5Gv2h8-9g) (AI Engineer World's Fair, 2026-07) — [transcript](https://simonwillison.net/2026/Jul/21/cat-and-thariq/) | "removing examples was extremely helpful…" (Thariq); "fewer 'do not do this' instructions…" (Thariq); the well-intentioned-human misreading test (Cat); the grep/glob-vs-bash and file-edit-tool tradeoff |
| [Prompting 101 · Code with Claude](https://www.youtube.com/watch?v=ysPbXH0LpIE) (Applied AI team, recorded 2025-05) | Foundations only; tactics superseded by the living docs |
| [Code with Claude 2026 sessions](https://www.youtube.com/playlist?list=PLmWCw1CzcFim2obQ-w3ohbULOfwp5lApR) | Context and background |

---

## Known conflicts

Where official sources disagree with each other, the resolution used here and why.

### 1. "Prompt engineering best practices for 2026" (2025-11) vs. the living docs

That post recommends three things the current documentation contradicts:

| The post says | The living docs say | Resolution |
|---|---|---|
| Prefill the assistant response as an advanced technique | Prefill on the last assistant turn returns 400 from Claude 4.6 onward | **Docs.** Use Structured Outputs or an explicit instruction |
| XML tags are "less necessary now" | Structuring prompts with XML tags is a recommended technique, and the official snippets use it wherever a block needs to function as one named unit | **Docs** |
| Role prompting is "often unnecessary" | Giving Claude a role focuses behavior; even one sentence helps | **Docs**, with the post's useful corollary: stating the *perspective* usually beats attaching an identity label |

The post's framing sections and its "common mistakes" list remain useful.

### 2. Examples: "give 3–5" vs. "removing them helped"

Both are official and they are not in conflict — they address different situations. Locking a format, tone, or structure calls for examples; an agent system prompt that needs judgment does better without them. **Examples constrain**; use them when constraint is what you want.

### 3. Verification: Opus 5 says remove it, Fable 5 says add it

Also not a conflict. What Opus 5 tells you to remove is *the model reviewing its own work*. What Fable 5's long-run guidance adds is *a separate, fresh-context verifier subagent* — which the same page notes tends to outperform self-critique. Remove self-review; keep external checking.

### 4. The Opus 5 effort starting point — not a conflict, a documentation change

Worth recording in full, because it is the clearest demonstration of why this repo defers to the live docs.

During review, two independent passes reported that the Opus 5 recommendation was "start at `high`." Three direct fetches of the Opus 5 page returned instead that `xhigh` "remains the recommended starting point" for coding and agentic work, and the effort page at that time had no Opus 5 section at all. The reports were recorded here as unreproducible.

They were not wrong. On re-verification the page reads:

> **Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency of higher settings. Start with the default (`high`) and adjust based on your evals: use `low` and `medium` liberally as your primary control for token cost and response time wherever quality holds, and step up to `xhigh` for demanding coding and agentic work.

The phrase "remains the recommended starting point" no longer appears anywhere on the page, and the effort page has since gained per-model sections for both Opus 5 and Fable 5. The documentation changed underneath the snapshot.

**The guidance in this repo now follows the current page: start at the default `high` on both models, and step up to `xhigh` for demanding coding and agentic work.** The practical difference from the earlier wording is real — an ordinary coding task now starts at `high`, not `xhigh`.

The lesson generalizes past this one parameter: **a snapshot of fast-moving documentation is wrong by default, not by exception.** For anything you are using to plan cost or latency, confirm it against the live page.

## Deliberately excluded

| Excluded | Why |
|---|---|
| [prompt-eng-interactive-tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) | Still targets Claude 3 Haiku; the techniques it teaches most prominently have since been reversed. **In fairness:** the official overview page still lists it for interactive learners, and does not deprecate it — but the same page designates Prompting best practices as "the living reference; start there" |
| ["AI prompt engineering: a deep dive"](https://www.youtube.com/watch?v=T9aRN5JkmL8) (2024-12) | The most-shared panel on the topic. Mental models hold up; tactics do not |
| Guidance specific to Opus 4.x and the Sonnet line | Out of scope by design — kept out rather than left in as "probably still fine" |
| Third-party "ultimate guide to prompting Claude" articles | Secondhand summaries; not used |
