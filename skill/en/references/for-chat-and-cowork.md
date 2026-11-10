# Scenario B: task prompts for the Claude app (chat or Cowork)

The user describes work in Claude desktop, and the resulting prompt runs directly in the Claude app. Non-coding work: research, analysis, writing, organizing source material, producing documents, spreadsheets and decks, scheduling, information synthesis.

## Contents

- Chat and Cowork are now one conversation
- Three things the docs recommend stating when you hand over a task
- Model, effort, and usage
- Put output length in the prompt
- Where the material goes
- Use examples to fix format, tone, and structure
- Research tasks
- Deliverables and files
- Scheduled tasks
- What not to put in
- A skeleton you can lift
- Sources

---

## Chat and Cowork are now one conversation

The support article "Claude Cowork and chat are one Claude" says the new experience merges the two into a single conversation, so `you don't need to decide which option better suits your task before getting started`: say what you want, and Claude can decide which tool to use. What you could previously do only in Cowork is now available from any conversation. It is rolling out in stages, starting with Pro, Max, and Ultra (web, desktop, and mobile), and even accounts on the same plan see it at different times. An account whose message box still shows "Chat" and "Cowork" options hasn't moved over yet; use it as before, and most of what the support article covers works there too.

So when you write the prompt, you don't need to choose a mode for the user. Write to these officially documented capabilities:

| What it can do | What the docs say | What it means for the prompt |
|---|---|---|
| Take on a whole task | Describe the outcome you want, and Claude works through the steps on its own: searching the web, reading files, running code, putting the results together. You don't need to break the work into steps | Describe the outcome; there is no need to break the work into steps. Give steps when their order or completeness matters (best practices page) |
| Keep working after you step away | More involved tasks keep running in the cloud even if you close your laptop or leave the page; when you come back, the result is waiting in the conversation | Tasks that use local files or local apps need Claude Desktop open |
| Hand back finished files | Documents, spreadsheets with working formulas, presentations you can open in PowerPoint | Say which kind of file you want |
| Designs, decks, and docs (artifacts) | On paid plans you can ask for a design, deck, or doc: Claude Design, Claude Slides, Claude Docs (all three in beta). Edit it yourself or have Claude change it, then share it by link or export it | This is a route for work that will be shown to or shared with others |
| Use connected apps | Google Drive, Gmail, Microsoft 365, Slack, and others, which it pulls from as needed while it carries out the task | Say which app the material is in. For Opus 5.5 on tasks that span several connected apps, the official sentence (look through the relevant sources before acting) is snippet 4.32 |
| Local files and folders | In Claude Desktop, give Claude access to a folder, and it can read, organize, and create files there | To save to the machine, name the folder, and remind the user to grant access first |
| Browsing and computer use | Opens sites, reads pages, clicks, and fills in forms, using the browser built into Claude Desktop or Claude in Chrome; computer use is in beta on Pro, Max, and Ultra | The built-in browser and computer use are available only while Claude Desktop is open |
| Scheduled tasks | Have Claude run a task on a schedule; it can run in the cloud and keeps going when the computer is off | See the Scheduled tasks section |
| Skills and plugins | Teach Claude how you like a task done, and it follows those steps whenever that kind of task comes up | Steps Claude should follow whenever this kind of task comes up can be taught once as a skill |

**How often it checks with you** is set by the permission setting in the message box. Under Auto (the default on Pro, Max, Ultra, and Team), Claude doesn't stop to ask about each step, and automated safety checks run before it takes an action; under Manual, it asks before it takes actions. The support article's caution: for work with real consequences, such as sending messages or changing important files, stay close and review what Claude does.

Repository-level operations, git, and build or deploy work belong in Claude Code, which is Scenario A.

## Three things the docs recommend stating when you hand over a task

In the support article's words: `describe what you need, the way you'd brief a colleague`. Results are better when you include these three:

- **The outcome you want**: what you want to end up with, for example "a one-page summary" or "a spreadsheet with a tab for each region"
- **The format**: how the result should be delivered, for example a Word document, a slide deck, or a message that can be pasted straight into Slack
- **The inputs Claude needs**: the files, links, or apps it should work from

Two more support articles add writing tips. For writing help, give the requirements, the audience and the key points in full, and send a text to be edited whole in one message, not in pieces; for research and analysis, define the research question and focus areas at the start, and put all the relevant data in one well-structured message ("Usage limit best practices"). Keep project instructions short: use them for general context about the project, key guidelines and Claude's role, and keep task-specific instructions in the chat itself ("How do usage and length limits work?").

## Model, effort, and usage

Model and effort are chosen in the model menu next to the send button; choose the level according to SKILL.md section 2. The support article on the levels: Low and Medium suit routine tasks and also stretch usage further; High offers the best balance of quality and speed; Extra high is designed for long-running coding and agentic tasks; Max is the most thorough. Each model's recommended level is marked "Default" in the menu.

The same article's advice by task ("Choose the right settings for your task"): for everyday tasks the defaults work well; simple questions, basic information requests, and general writing don't need extra effort, and lower effort stretches usage further; for complex tasks, raise the effort level, turn on thinking, or both; `For complex coding and agentic tasks on Opus 4.7 or newer, try Extra high (xhigh) first`; for the hardest, most correctness-critical work, choose Max and expect longer response times; whichever setting you choose, be specific about the problem. The xhigh line disagrees with the model pages: the Opus 5.5 and Fable 5.2 pages both keep `xhigh` for work where a gain has been measured, and the Sonnet 5.5 and Haiku 5.5 pages likewise use `xhigh` only where evals show a quality gain. This skill chooses by the model pages (SKILL.md section 2); in the delivery note, also tell the user what the support article says.

In the Claude app, thinking can't be turned off on Fable 5.2, Opus 5.5, Sonnet 5.5, or Haiku 5.5; to make the model think less, choose a lower effort level.

Usage: everything you do in Claude counts toward the plan's usage limits, and longer agentic tasks that search the web, run code, or create files generally use more than a quick question. The higher the effort, the sooner you reach the limit. For the allowance rules for Fable models, see "Subscription usage limits" in `models.md`. For documents, spreadsheets and decks, two official sources give different examples: the selection article puts everyday ones on Sonnet 5.5, and a complex financial model or a document built from a large pile of source material on Opus 5.5; the platform selection matrix lists analysis carried through to a finished document, spreadsheet or deck among its examples for Fable 5.2. For the ways to save usage that the support site gives for the Claude app (combine related questions into one message, give the background up front, use a lower effort for routine tasks, turn off connected apps you don't need, ask Claude not to search the web when you don't need current information, start a new conversation when a long one nears the limit), see "Officially recommended ways to save usage" in `models.md`.

---

## Put output length in the prompt

On Fable 5.2 and Opus 5, do not count on effort to shorten replies. The Fable 5.2 page: `Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.` The Opus 5 page states that lowering effort does not reliably shorten visible output, and says the files the model writes to disk `are often longer than on prior models`. The Opus 5.5 pages do not restate either point; they say only that it `tends to complete the same task with fewer tokens`.

For ready-made wording, see snippets 4.2 and 4.4 (Opus 5 page wording). When the target is Fable 5.2, it follows a hard cap on sentence length closely, which can make its prose choppy; the official step is to remove the style rules you no longer need first. For final reports, use snippet 4.40.

---

## Where the material goes

When the user wants Claude to work through a body of source material (documents, emails, chat logs, web content):

**Long material at the top, the query and instructions at the end.** In Anthropic's tests this can improve response quality by up to 30 percent, especially with complex, multidocument inputs.

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

**One more technique for long documents — grounding in quotes:** ask the model to extract the relevant quotes into `<quotes>` tags first, then answer from those. This helps it focus on the relevant content and ignore the rest of the document.

---

## Use examples to fix format, tone, and structure

The best practices page: `Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.` Tasks that produce the same format every time (weekly reports, meeting notes) are that kind of case. What the page asks for:

- 3–5 of them, wrapped in `<example>` (with `<examples>` around them when there are several)
- **Relevant**: mirror the actual use case
- **Diverse**: cover edge cases, so the model does not pick up unintended patterns
- You can have Claude evaluate whether your examples are relevant and diverse enough, or generate more based on the ones you gave (`You can also ask Claude to evaluate your examples for relevance and diversity, or to generate additional ones based on your initial set`)

---

## Research tasks

The best practices page lists three things for research tasks: provide clear success criteria, defining what counts as a successful answer; encourage source verification, asking Claude to verify information across multiple sources; and use a structured approach for complex research tasks. For the third it gives a sample prompt:

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

Add more depending on the target model:

- **Sonnet 5.5**: it sometimes answers from its training knowledge when a search would show that details have changed. When it has a search tool, use snippet 4.45, and remove prompt language that discourages tool use
- **Haiku 5.5**: when you give it a search tool, also give it today's date; see snippet 4.47
- **Fable 5.2**: the model-selection matrix lists multistep deep research under it, but on one research benchmark (DeepResearch Bench II, each at its Messages API default: Fable 5.2 at `high`, Opus 5.5 at `medium`) the cost page measured it at 2.9 times the cost of Opus 5.5, with a score 1.9 points lower; in the official words, `Measure what a more expensive model buys on your own workload`. Tell the user both points in the delivery note
- **Opus 5.5**: Anthropic's article on using Opus 5.5 suggests adding this line to research and analysis requests: `Mark anything you couldn't confirm, and say where you looked`

**Allow it to say it doesn't know.** The line below comes from Anthropic's 2025-11 prompt engineering article (the example in its "Give permission to Claude to express uncertainty" section) and is not on the documentation pages:

```
If the data is insufficient to draw conclusions, say so rather than speculating.
```

---

## Deliverables and files

The support article states that Claude can create documents, spreadsheets with working formulas, and presentations you can open in PowerPoint; these files appear alongside the conversation, and they can be downloaded, or Claude can keep refining them. State the format when you write the prompt (see the three things above).

Three points from Anthropic's article on using Opus 5.5 (2026-09-22, written about Opus 5.5) bear on this section: when you want a spreadsheet or a document, ask for the finished file, not an outline; attach the chart, diagram, screenshot or slide itself instead of retyping the numbers, and ask a specific question; and to have it check a long plan, report or deck for mistakes, the article's wording is `Check this deck for anything that contradicts itself: numbers, dates and names. Quote each problem and say where it is.`

For work that will be shown to or shared with others, paid plans can also ask for a design, deck, or doc (Claude Design, Claude Slides, Claude Docs; beta), which can be shared by link or exported.

To save a file into a folder on the user's own computer: in Claude Desktop, give Claude access to that folder. Claude can reach local files only while Claude Desktop is open, and by default it asks before permanently deleting any files.

## Scheduled tasks

The support article: you can have Claude run a task on a schedule, for example a summary of your inbox every Monday morning; scheduled tasks can run in the cloud, so they keep going when the computer is off. When what the user really needs is an X every week, tell the user it can be set up as a scheduled task.

While a task runs, the user can check its progress, answer Claude's questions, and redirect the work from their phone (the support article's "Check in from your phone").

---

## What not to put in

1. **Self-verification instructions added, or removed, the same way on every model** — the official pages say different things for different models; see SKILL.md 3.4
2. **Résumé personas** ("you are a senior analyst with twenty years of experience") — the best practices page says a role helps, and that one sentence is enough to make a difference (`Even a single sentence makes a difference`). A long persona is a different matter: Anthropic's 2025-11 article says heavy-handed role prompting is often unnecessary, and that to shift the perspective you should say directly what to focus on; its example is `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential`
3. **A long list of "do not"s** — rewrite as positive statements with reasons. See SKILL.md 3.5
4. **Asking it to write out its thinking** — on Fable 5.2, Opus 5.5, and Sonnet 5.5 this can be refused. See SKILL.md 3.9
5. **When the target is Fable 5.2**: lines telling it to think less and hard style caps (see SKILL.md 3.13); a blanket instruction to act more freely changed nothing in one simulated setup Anthropic tested (see the Fable 5.2 table in `models.md`)

---

## A skeleton you can lift

```
(long material first, if any)
<documents>
  <document index="1">
    <source>…</source>
    <document_content>…</document_content>
  </document>
</documents>

<task>
What to do, who it is for, and what they will do with the output.
</task>

<requirements>
1. …
2. …
(positive statements, each with its reason)
</requirements>

<output>
Shape: (conversation / document / spreadsheet / slide deck / a shareable design, deck, or doc)
Length: (state it)
Structure: (sections or fields)
</output>

<examples>
  <example>…</example>
</examples>
(add when you want to fix format, tone, or structure)
```

Close with a **success criterion**: what counts as having done this correctly. Stating the audience and purpose in `<task>` comes from the "Provide context and motivation" section of Anthropic's 2025-11 article (`Explaining the purpose or audience for the output`, `Describing how the output will be used`).

---

## Sources

The descriptions of the Claude app's capabilities in this file follow five support.claude.com articles: "Claude Cowork and chat are one Claude", "Change the model, effort, and thinking settings", "Claude Fable models on your plan", "Usage limit best practices", and "How do usage and length limits work?". The prompting technique sections follow the official prompting best practices and the per-model prompting pages; the six passages that come from Anthropic articles (three from the 2025-11 prompt engineering article, two from the article on using Opus 5.5, one from the model-selection article) are cited inline. Product capabilities change between releases; when in doubt, defer to what the user actually sees in their interface.
