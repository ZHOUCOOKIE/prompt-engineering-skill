# Scenario B: task prompts for desktop chat or a Cowork session

The user describes work in Claude desktop and the resulting prompt runs directly in **desktop chat** or a **Cowork session**. Non-coding work: research, analysis, writing, organizing source material, producing documents, spreadsheets and decks, scheduling, information synthesis.

## Contents

- Chat or Cowork first
- Output length must be pinned
- Where the material goes
- Give examples when format must be locked
- Research tasks
- Cowork: deliverables and files
- Cowork: skills that take over automatically
- What not to put in
- A skeleton you can lift
- Reusable and recurring prompts
- Sourcing note for this file

---

## Chat or Cowork first

The two are positioned differently, and the writing follows. The table below follows Anthropic's own description of the desktop modes (the third, Claude Code, is Scenario A and is covered in `for-claude-code.md`).

| | desktop chat | Cowork session |
|---|---|---|
| **Positioning** | Quick exchanges and iterative work: asking, brainstorming, drafting, working through a problem back and forth | Work that takes real effort: pulling information from many sources, making sense of it, producing something finished |
| **Local files** | Desktop connectors reach local tools; **cannot** access multiple sources systematically | **Reads and writes local files in connected folders directly**, no manual upload or download |
| **Output** | Content in the conversation | **Finished deliverables**: Excel with functional formulas, PowerPoint, formatted documents |
| **Duration** | Not suited to sustained work | Runs autonomously for a long time; the user can leave. Supports **scheduled recurring tasks** |
| **Parallelism** | None | **Sub-agent coordination**, dividing complex work into parallel workstreams |
| **Extensibility** | Connectors | Connectors, skills, plugins, Claude in Chrome |

**One boundary that matters:** Cowork **cannot** access your full file system or run terminal commands the way Claude Code can — it works inside a protected environment, on connected folders only. So repository-level operations, git, and build or deploy work are not Scenario B; those are Scenario A.

**How to decide:** if the deliverable is "content I can just read," chat is enough; if it is "a file I want to keep" or the work needs systematic sourcing across many materials, Cowork.

When the user does not say and the task clearly needs file output or bulk material handling, **default to writing for Cowork and say so in the delivery note**.

**Two Cowork capabilities worth using deliberately:**

- **Scheduled recurring tasks.** When the underlying need is "give me an X every Monday," write the prompt as a reusable task description and tell the user to set it up as a scheduled task rather than running it by hand each time.
  **A scheduled task means nobody is watching.** Any "shall I continue?" stalls the whole thing until next week. So a recurring prompt must say **how to keep going when something goes wrong** — missing material, an unopenable file, an unresolvable source conflict: record it and continue, then collect them under a "needs your decision" section in the output rather than stopping to ask. (Snippet 4.18 is the full treatment; for something light like a weekly report you do not need its two blocks, just this rule.)
  **One more rule that recurring deliverables need:** have it read the previous cycle's output first and **report only the delta** — what changed, whether last cycle's stated next steps happened, what is still open. Without this, every cycle becomes a flat restatement and the reader has to diff it themselves. This holds for weekly reports, standup notes, status updates, digests, and changelogs.
- **Parallel subagents.** With a lot of material, the prompt can say explicitly: process each source in a parallel subagent and have each return only a structured summary. Faster, and it keeps the main context clean.

## Output length must be pinned

This is the most frequent problem in Scenario B. **Lowering effort does not reliably shorten a reply** — effort governs how much the model thinks, not how much it says — so length has to be prompted for on every model in scope. (The "runs longer by default" finding is published for Opus 5; Opus 5.5 officially `tends to finish the same task with fewer tokens`, so re-measure rather than assuming, but state the length either way.)

So anything meant to be read needs an explicit length constraint. **How to choose and what stacks is in SKILL.md Section 8, "Choosing a length control"**; in short:

- Pick one primary mechanism — **give a number when you can** ("300–400 words", "3–6 bullets, each under 15 words", not "keep it brief"); fall back to snippet 4.2 only when a number is impossible
- When the deliverable is written to a file, **add** snippet 4.4 (a word count usually constrains the body but not the padding sections the model adds on its own)
- When the system prompt is long, **append** a short `<tone_preference>` reminder at the end

## Where the material goes

When the user wants Claude to work through a body of source material — documents, emails, chat logs, saved web pages:

**Long material at the top, the query and instructions at the end.** Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs.

```xml
<documents>
  <document index="1">
    <source>annual_report_2025.pdf</source>
    <document_content>
      {{MATERIAL}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis_q2.xlsx</source>
    <document_content>
      {{MATERIAL}}
    </document_content>
  </document>
</documents>

(query and instructions go here, last)
```

**One more technique for long documents — grounding in quotes:** ask the model to extract the relevant quotes into `<quotes>` tags first, then answer from those. This helps it focus on the relevant content and ignore the rest, an effect that is clearest on "find the supporting evidence across this pile" tasks.

## Give examples when format must be locked

A large share of Scenario B work is "produce this same shape every time" — weekly reports, meeting notes, retrospectives, briefings, cards. This is exactly the "give examples" branch of SKILL.md Section 4.

- 3–5 of them, wrapped in `<example>`
- Make them **diverse**, covering edge cases, so the model does not pick up unintended patterns
- The most valuable examples are **paired**: raw material → finished output. That teaches the compression ratio and the selection criteria, not just the shape
- If the user has real past samples, have them drop two or three in — this is the single highest-return step

**When the user has no samples yet** (common in a one-shot delivery where you cannot ask): **give one paired example you wrote, and do not manufacture three to five.** Invented examples in bulk teach an imagined house style as though it were the standard, and the user will find that hard to detect later. Give one that demonstrates the compression ratio and the selection criteria, leave a clearly marked slot in the prompt, and tell the user in the delivery note: **swapping in two or three real ones is the biggest single improvement available.**

Conversely, when the task is open-ended creation or brainstorming, or you want the model to choose its own angle, **do not give examples** — they narrow the exploration space to their own neighborhood.

## Research tasks

For work that requires sourcing and synthesizing across many sources, there is a ready-made official block. **Copy it verbatim, including the self-critique sentence** — see the carve-out below it:

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

> **Carve-out for SKILL.md 3.4.** That block contains `Regularly self-critique your approach and plan`, and 3.4 says to remove self-verification instructions on Opus 5 (and to measure before removing them on Opus 5.5). This is the same kind of exception as `you MUST read the file` inside snippet 4.7: it is **tuned official wording, so copy it as is**. The two are not the same instruction — 3.4 removes *end-of-answer self-review*, a last look at a finished deliverable; this sentence is *mid-run course correction*, re-aiming the search while evidence is still coming in. If the task is a single-pass write-up rather than a multi-step search, that sentence has nothing to correct, and you may drop that one line.

Three more: **give clear success criteria** (what constitutes a successful answer to this question), **ask for source verification** across multiple sources, and **explicitly allow it to say it does not know**. The line below is this skill's own wording, not an official snippet — adapt it freely:

```
If the data is insufficient to draw conclusions, say so rather than speculating.
```

That last one matters especially for research — without an exit, the model states uncertain things with unwarranted confidence.

## Cowork: deliverables and files

Cowork produces real files. **Name the file type you want**, because it determines the path taken:

| Deliverable | How to say it |
|---|---|
| Editable document | "output as .docx" |
| Data table | "output as .xlsx, header row first, columns are …" |
| Presentation | "build a .pptx, X slides, each slide …" |
| A shareable page, dashboard, or tracker | "build it as a single self-contained HTML file" |
| Just text for me to read | Say so explicitly: "give it to me in the conversation, do not create a file" |

**Whether it lands on disk:** if the user wants the file saved to a specific folder on their machine, name the **target folder** in the prompt; otherwise it is delivered in the conversation only.

**Bulk processing:** with multiple files, state the processing order and naming rule, and **what to do with the ones it cannot handle** — skip and record, or stop and ask. Without that rule, behavior on a bad file is unpredictable.

## Cowork: skills that take over automatically

Cowork ships output-format skills (docx, xlsx, pptx, pdf, and others) that engage automatically when the corresponding file type is involved. That means:

- **Do not teach it how to generate a .docx** in the prompt — that is the skill's job, and writing it is redundant (SKILL.md 3.3)
- What the prompt should specify is the **content requirements**: structure, terminology, data sources, length
- But **do state style preferences** — "use the company template," "avoid the default palette" — because the skill cannot guess those

One ordering rule worth writing in: **gather the facts fully before touching the file mechanics.** The official guidance for output-format skills is that research comes first; starting the document machinery early anchors on layout while the correct content is not yet in hand.

## What not to put in

The three things most often over-written in Scenario B:

1. **Self-verification instructions** ("check it once more," "double-check your answer") — on Opus 5 the model does this on its own and adding them causes over-verification; on Opus 5.5 that finding is not restated, so do not add them reflexively but do not assert the Opus 5 result either. See SKILL.md 3.4. Note the distinction that section draws: **reconciling sources against each other is the task, not self-verification** — do not delete that by mistake
2. **Résumé personas** ("you are a senior analyst with twenty years of experience") — no benefit on current models, pure token cost. To shift the perspective, say what to focus on — the official example is `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential` rather than assigning a role
3. **A long list of "do not"s** — rewrite as positive statements with reasons. See SKILL.md 3.5

## A skeleton you can lift

```
(long material first, if any)
<documents>
  <document index="1">
    <source>…</source>
    <document_content>…</document_content>
  </document>
</documents>

<variables_this_cycle>   (only for a recurring template — see the last section)
Date range: …
Material location: …
</variables_this_cycle>

<task>
One sentence on what to do, who it is for, and what they will do with the output.
</task>

<requirements>
1. …
2. …
(positive statements, each with its reason)
</requirements>

<output>
Shape: (conversation / .docx / .xlsx / .pptx / single-file HTML)
Length: (a specific number)
Structure: (sections or fields)
</output>

<examples>
  <example>…</example>
</examples>
(only when format must be locked)
```

Close with a **success criterion** — what counts as having done this correctly. Without it you cannot check your own work and the user cannot judge whether to send it back.

## Reusable and recurring prompts

When the user says "I want to fix this and use it every week," write a template rather than a one-off instruction. Add a variables block that collects everything that changes each cycle in one place, so the user edits only that block:

```
<variables_this_cycle>
Date range: 2026-07-20 ~ 2026-07-26
Material location: (folder path)
Previous output: (path to last cycle's file, for the delta comparison)
</variables_this_cycle>
```

**Place it immediately before `<task>`** — that is, after the material, in the slot the skeleton above shows. This keeps the long-context rule intact (material first, instructions last) and still puts the lines the user edits each cycle right next to the instruction they parameterize.

---

## Sourcing note for this file

The description of chat and Cowork capability boundaries follows Anthropic's official Cowork overview and its comparison of the desktop modes; the prompting technique sections follow the official prompting best practices. Product capability details can change between releases — when in doubt, defer to what the user actually sees in their interface.
