---
name: write-prd
description: Generate or promote a Product Requirements Document (PRD) for a
  feature or product change in this codebase. Trigger this skill when the user
  describes a feature idea, product improvement, or change ("let's add X",
  "what if we...", "I want to introduce...", "spec out Y", "write a PRD for Z",
  "draft requirements for..."), or asks to promote an existing draft PRD to
  ready ("promote this PRD", "move to ready", "this PRD is ready"). The output
  is a markdown file in docs/requirements/draft/ or docs/requirements/ready/.
  Do not use for bug reports, hotfixes, code refactoring without product
  implications, or routine configuration changes.
---

# Writing a PRD

This skill produces a Product Requirements Document. The flow walks the user through structured discovery, grounds the requirement in the existing codebase, generates the PRD from a fixed template, saves it to git as a draft, and later promotes an approved draft to `ready/`. The user remains in the loop at every transition — discovery confirmation, codebase findings, commit, promotion.

## Language and terminology

The user communicates in Russian or English. Match their language when asking questions, restating answers, and discussing context — do not force English if they write in Russian.

The PRD itself is **always written in English**, regardless of chat language. This applies to every section, every heading, every bullet.

### Translating user input

When the user gives content in Russian that needs to land in the PRD in English:

- Use the project's established English terminology from the codebase. Domain entities (e.g., `invoice`, `customer`, `revenue`, `dashboard`) must match the names used in `app/lib/`, database schemas, and route paths — not generic dictionary translations.
- If you encounter a Russian word and it's unclear whether it's a domain term (translate using project conventions) or a regular word (translate freely), **ask the user before translating**. Do not silently choose.

  Example: user says "счёт". This could mean `invoice`, `account`, or `bill` depending on context. Ask: *"By 'счёт' do you mean the `invoice` entity in /app/lib, or something else?"*

- Once a term is confirmed in a session, reuse the confirmed translation for the rest of that PRD without re-asking.

This rule applies across all steps: discovery answers, codebase findings, PRD text.

## Step 0. Collect source materials

Before discovery, ask the user once whether any source materials should inform the PRD. If none, skip immediately to Step 1.

### Step 0a. Ask

Open your very first turn with:

> "Before I start: do you have any source materials to incorporate? Common things I can work with:
> - Call transcripts (Discovery / SME sync / Pre-Discovery)
> - Diagrams, mockups, wireframes (PNG, SVG, screenshots)
> - Existing tickets or specs (pasted text, PDFs, screenshots)
> - Anything else that frames this feature
>
> Attach whatever's relevant, or reply `no` / `skip` / `скип` if there's nothing — we'll work from your description only."

If the user replies `no`, `none`, `skip`, or `скип` (case-insensitive) — move directly to Step 1a without further prompting. Do not ask about sources again later in the flow.

If the user mentions a source by name but does not attach it ("the JIRA ticket for this"), ask them to paste or attach the content. Do not assume you can fetch it.

If materials arrive later (mid-Step-1 or mid-Step-2), process them with the Step 0b rules and re-confirm any Step 1 answers they affect.

### Step 0b. Process by type

**Transcripts.** Treat as context, not truth. Anything said by a speaker is an assumption requiring confirmation by the user of this skill, not a confirmed answer. Attribute extracted content in the Step 1e restate: *"Problem: ... (from transcript)"*. If two speakers contradict each other, surface the conflict in Open Questions — do not pick a side.

Do not summarize the transcript wholesale. Extract only content relevant to Step 1 questions or technical constraints. Skip greetings, off-topic banter, jokes.

**Diagrams, mockups, wireframes.** Extract entities, relationships, flows. Do not narrate the visual in prose — describing what the picture looks like adds nothing. What goes into the PRD is what those entities and relationships mean for scope, constraints, or solution shape.

If it's an architecture diagram, retain a note for Step 2: compare its entities and connections against the actual codebase. If they disagree, the codebase wins for Constraints, and the disagreement goes to Open Questions.

**Tickets, specs, other pasted/attached text.** Extract answers to Step 1 questions that are already present. If the ticket conflicts with what the user says verbally during discovery, ask the user to resolve before continuing — do not silently merge. Attribute extracted answers to their source in the Step 1e restate.

### Step 0c. Bridge to Step 1

After processing, treat extracted answers as inputs to Step 1a's mirror-back. Even content pulled from a source must be confirmed by the user before it's locked in — sources inform, the user decides.

## Step 1. Gather business context

Surface the business intent through structured Q&A before touching code. The user's framing is a hypothesis to test, not a source of truth — do not infer answers from the initial idea, even if it seems obvious. Extract only what the user explicitly confirms.

### Pacing rule (load-bearing)

Send **exactly one question per turn**. End your turn. Wait for the reply. Then send the next.

Never bundle multiple questions in one message — not with bullets, not with "first... second...", not "and one more thing". One question, then stop. The friction is the point.

**Skip mechanism.** If the user replies with `skip` or `скип` (case-insensitive) to any question or follow-up, accept it without protest and move to the next. Record the section as *Skipped during discovery — to revisit before moving to ready/.* A skip is not an assumption; it's an explicit deferral that must be resolved before the PRD is promoted out of draft/.

### Step 1a. Mirror back what you already have

You now have: the user's original feature description, and (optionally) extracted content from source materials processed in Step 0. Identify which of the Step 1b questions are already answered, fully or partially, across both inputs.

Open Step 1 by mirroring this back, with source attribution where it applies:

> "From your description [and the attached materials], I already have: **[Q label]** — [your reading] [(from [source])], **[Q label]** — [your reading]. Still need: [list of unanswered labels]. Did I get the first ones right?"

Wait for confirmation. If the user corrects something, update internally. Only then proceed to the remaining questions. Skip the ones already confirmed.

If nothing concrete was provided in either the initial description or sources, say so briefly and start at Question 1.

### Step 1b. The questions

Ask in this order. Skip any already confirmed in Step 1a, or any the user explicitly skips during the Q&A.

1. **Problem.** What user or business problem are we solving? Who feels the pain today, and how does it show up in their workflow?
2. **Current workaround.** How is this problem solved today — manually, with another tool, or not at all? This anchors the success-metric baseline and signals how deep the pain is.
3. **Hypothesis.** Push for a single sentence in this exact form: *"If we do X, then Y will happen, because Z."* If the user gives prose, rephrase into the template and ask them to confirm.
4. **Target users.** Specifically — role, scenario, how often they hit this. Push back on "everyone" or "all users" answers.
5. **Success metric.** Must be **numeric or observable** (conversion %, minutes saved per task, error rate, adoption count, completion rate). Anything like "users will like it", "it'll be easier", or "we'll know" is not a metric — follow up immediately with "what would you measure to confirm that?"
6. **Scope & non-goals.** What is explicitly NOT in this iteration? Force at least one concrete non-goal. "Nothing is out of scope" is not an acceptable answer — push once.
7. **Constraints.** Any deadlines, external dependencies, regulatory or compliance constraints to know about? *"None known"* is a valid answer — accept it and move on without pushback.

### Step 1c. Handling vague answers

If an answer is vague, mushy, or restates the question:

1. Send **one** targeted follow-up. Be specific about what's missing — quote the vague phrase and ask what it means concretely. Example: user says "make it faster"; you ask "faster than what, and by how much?"
2. If the next answer is still vague, **stop pushing**. Record the best available version as an assumption (see Step 1d) and move to the next question.

One follow-up per question is the hard ceiling. Do not loop. (The user can also escape with `skip` at any point.)

### Step 1d. Marking assumptions in the PRD

When a section is filled from an assumption rather than a confirmed answer, render it inline in italics with an explicit owner for follow-up:

> The bulk export will primarily serve month-end close workflows. *Assumption — to confirm with the finance lead.*

Place the marker immediately after the sentence it qualifies, in the same section. Do not collect assumptions into a separate section.

Skipped questions use a different marker: *Skipped during discovery — to revisit before moving to ready/.* Both signal draft/ status, but skipped items must be resolved before promotion to ready/, while assumptions can carry forward if confirmed by the listed owner.

### Step 1e. Confirm before moving on

Once all seven questions are answered, assumed, or skipped, restate the result as **7 bullets, one per question, ≤20 words each**:


```
- Problem: ...
- Current workaround: ...
- Hypothesis: If ..., then ..., because ...
- Users: ...
- Metric: ...
- Out of scope: ...
- Constraints: ... (or "none known")
```



Then ask: *"Does this capture it before I dig into the codebase?"* Wait for explicit confirmation ("yes", "go", "looks right"). Only then proceed to Step 2.

## Step 2. Explore the codebase

Once Step 1 is confirmed, investigate the existing code to ground the requirement in technical reality. The output of this step is two things: an internal map of what's reusable, what's new, what's constrained, and what's unclear — and an explicit summary shown to the user for confirmation before any PRD is written.

Read targeted, not exhaustive. Every file you open should answer a specific question raised in Step 1.

### Step 2a. What to read

**Mandatory core** — always read, even if the feature seems unrelated. This is the floor:

- `package.json` — stack, libraries, scripts.
- `app/lib/definitions.ts` — existing entities and types.
- `app/` (directory listing only) — overall routing structure.
- `auth.ts` — the auth contract, so suggestions don't accidentally break it.

**Adaptive expansion** — pick rows based on what Step 1 said the feature touches:

| If the feature involves... | Also read |
|---|---|
| Data (new entity, CRUD, queries, reports) | `app/lib/data.ts`, `app/lib/actions.ts`, schema/seed files (latest state only) |
| UI (new screen, modified screen, component) | Relevant `app/ui/...` components, `tailwind.config.ts` if styling changes |
| Authentication, authorization, or roles | `auth.config.ts`, `middleware.ts` |
| Search, pagination, filtering | Existing query patterns in `app/lib/data.ts` |
| A new route | One or two existing route files (`app/dashboard/*/page.tsx`) to mirror conventions |
| External integration | `app/api/*` if it exists, plus relevant config |

If the feature triggers more than three rows, you're probably scoped too broadly — flag this to the user before continuing exploration.

### Step 2b. How to read

**Depth limits.**
- Never read inside `node_modules/`.
- Tests: read `describe` / `it` titles only. Dive into a test body only if the feature extends an existing tested contract.
- Files over ~500 lines: locate relevant sections via grep/glob first, then view only the relevant range. Do not view the full file.
- Migrations: read the latest schema state only. Do not walk migration history.

**Budget.** Maximum **15 file reads** in Step 2 beyond the mandatory core. If you hit this without a clear picture, stop and declare exploration incomplete (see Step 2d).

**Navigation order.** Start with directory listings, not file content. Use grep/glob to locate patterns before opening individual files. For each file you do open, name the question you're answering — never read out of curiosity.

### Step 2c. Reconcile with Step 0 sources

If Step 0 produced source materials with technical implications (architecture diagram, ticket with implementation hints, mockups showing data flow):

- Cross-check each technical claim from those sources against the codebase.
- Where the source and the code disagree, **the codebase wins** for Constraints — the source may be outdated, aspirational, or wrong.
- Surface the disagreement explicitly in Open questions, e.g.: *"Architecture diagram (Step 0) shows X; current code does Y — needs clarification."*

Skip this sub-step if Step 0 had no source materials or none with technical content.

### Step 2d. Show summary and gate to Step 3

After exploration, present the four buckets to the user using this exact structure:


```
**What can be reused**
- [Existing component / model / utility] — [how it'll be used]

**What needs to be added**
- [New entity / component / route] — [why]

**Constraints from the codebase**
- [Existing limit, architectural friction, or awkward fit]

**Open questions**
- [Things the code doesn't answer, sources that contradict code, unclear paths]
```



Lead in with one line:

> "Here's what I found in the codebase. Confirm or correct anything before I write the PRD."

Wait for explicit confirmation ("looks right", "go", "yes") or corrections. If the user corrects something, update internally. Only then proceed to Step 3.

**If exploration was incomplete** (budget hit, codebase too large, unfamiliar stack, key files missing): say so directly in the summary message:

> "Exploration incomplete: I covered [what], but couldn't form a clear picture on [what]. Want me to dig deeper, or proceed with the gap flagged in Open questions?"

Let the user choose. Do not silently proceed with a partial picture.

## Step 3. Generate the PRD

Once Step 2 is confirmed, generate the PRD in markdown using the exact template below. Fill every section. Keep each one short — bullets where possible, prose only when bullets lose meaning.

### Step 3a. Resolve the author

Before generating the file, run:


```bash
git config user.name
```



Use the returned value for the `Author` field. If the command returns empty or fails, ask the user once: *"What name should I use as the PRD author?"* and use the answer. Do not invent a name.

### Step 3b. Section content rules

These apply across the template — read before filling.

- **No `TBD` in body sections.** Every section must contain real content. If something is unresolved, do not write `TBD` in the body — push it to §8 Open questions as a concrete actionable item. If something is approximately known but not confirmed, write the best version and mark it with the assumption marker from Step 1d.

- **Empty sections — explicit None.** If a section or sub-section legitimately has nothing, write `None — [one-sentence reason]` instead of leaving it blank. Example: *"Add: None — this feature reuses existing infrastructure entirely."*

- **Source attribution.** When content was extracted from Step 0 materials, attribute inline where it matters for traceability: *"...as discussed in the Discovery transcript."* Do not over-attribute every sentence.

### Step 3c. The template

Use exactly this structure:


```
# [Feature name]

**Status:** draft
**Author:** [from `git config user.name`]
**Date:** [today, ISO format YYYY-MM-DD]

## 1. Problem

[1–2 short paragraphs describing the pain: who feels it, when, and how it shows up in their workflow.

End the section with one sentence on how this is solved today, prefixed with "Today, " — e.g., "Today, users export invoices one at a time via the per-row menu."]

## 2. Hypothesis

If [action], then [outcome], because [reasoning].

## 3. Target users

[Who, in what scenario, how often. No "all users" answers.]

## 4. Success metrics

- Primary: [metric] — target [value]
- Secondary: [metric] — target [value]  (or `None — single metric is sufficient`)

## 5. Solution overview

[Describe the workflow and behavior. 2–4 short paragraphs.

In scope of this section: what the user experiences, what the system does, what happens on the happy path and one or two key edge cases.

Out of scope of this section: UI details (button placement, exact wording, colors), implementation (API signatures, database column names, code), and timeline.

The reader should be able to think through the happy path and one or two key edge cases from this section alone.]

## 6. Technical context

### Reuse
- [Existing component / model / utility] — [how it'll be used]

### Add
- [New entity / component / route] — [why]

### Constraints
- [Existing limit, architectural friction, or awkward fit from the codebase]

(Each sub-section may use `None — [reason]` if genuinely empty.)

## 7. Scope & constraints

### Out of scope
- [What we are explicitly NOT doing in this iteration]

### Business constraints
- [Deadlines, external dependencies, regulatory or compliance constraints]

(Each sub-section may use `None — [reason]` if genuinely empty.)

## 8. Open questions
- [ ] [Concrete actionable question — name the person or role who can answer it where possible]

(Use `None — all items resolved` only if genuinely true.)

## 9. Acceptance criteria

Use Given/When/Then for behavioral scenarios (a user action produces an outcome). Use free-form bullets for static criteria (performance, data formats, accessibility, validation rules) where Given/When/Then is unnatural.

**Behavioral**
- [ ] Given [context], when [action], then [expected outcome].

**Static**
- [ ] [Criterion] — [target or specification]

(Either sub-section may use `None — [reason]` if not applicable.)

## 10. Sources
- [Source type], [identifier or short description] — informed [which PRD sections]

(Omit section 10 entirely if no Step 0 source materials were used.)
```



### Step 3d. Final check before saving

Before handing the file off to Step 4, verify internally:

- Every section is filled with real content, an assumption marker, or an explicit `None — [reason]`. No `TBD` anywhere.
- Every unresolved item appears in §8 Open questions as an actionable line.
- §10 is present only if Step 0 materials were used.
- The Author field is populated (not a placeholder).

If any check fails, fix before proceeding to Step 4.

## Step 4. Save and offer to commit

The PRD is saved as a file in `draft/`. A branch and a draft PR are offered to the user — never executed without explicit confirmation.

### Step 4a. Pre-flight checks

Before touching git, run these and pause if anything looks off:


```bash
git fetch
git checkout main
git pull
git status
```



If `git status` shows uncommitted or staged changes unrelated to this PRD, stop and ask the user how to handle them (stash, commit separately, or proceed carrying them along). Do not proceed silently.

If `git checkout main` fails because the branch doesn't exist or the working tree is dirty, stop and ask. The skill assumes `main` as the default branch.

### Step 4b. Save the PRD file

Compute the path:

- Folder: `docs/requirements/draft/`
- Filename: `YYYY-MM-DD-<slug>.md`
- Date: `date +%Y-%m-%d` (today, local).
- Slug rules: 2–5 words, lowercase Latin letters and digits, words separated by hyphens. No underscores, no dots, no leading digit. Example: `invoice-bulk-export`.

Before writing, check if the file already exists. If it does, ask the user:

> "A file at `docs/requirements/draft/<filename>` already exists. Overwrite it, or use a different slug?"

Wait for the decision. Do not overwrite silently.

**Source materials never enter the repo.** Files attached by the user in Step 0 (transcripts, diagrams, mockups, screenshots) stay where they came from — they are referenced in §10 of the PRD by metadata only. Never copy them into `docs/`, never `git add` them.

### Step 4c. Offer the git workflow

Once the file is saved, present this workflow to the user and wait for explicit confirmation before running any of it:


```bash
git checkout -b prd/<slug>
git add docs/requirements/draft/YYYY-MM-DD-<slug>.md
git commit -m "docs(prd): draft requirements for <feature name>"
git push -u origin prd/<slug>
REPO=$(git remote get-url origin | sed 's/.*github\.com[:/]//' | sed 's/\.git$//')
gh pr create --draft \
  --title "Draft PRD: <feature name>" \
  --body "PRD draft: <feature name>."
```



Notes on this block:
- `git add` is **targeted to the single PRD file**, not the folder, to prevent pulling unrelated changes into the commit.
- Commit message follows Conventional Commits (`docs(prd): ...`).
- PR title is `Draft PRD: <feature name>`. PR body is one line containing just the feature name — no TL;DR, no summary. The reviewer opens the PRD file directly.

Wait for explicit confirmation ("yes", "go", "ok") before executing. If the user declines, leave the file saved and stop.

### Step 4d. `gh` CLI fallback

Before running `gh pr create`, check whether `gh` is installed and authenticated:


```bash
command -v gh && gh auth status
```



If both succeed — run the full `gh pr create` from Step 4c.

If either fails — skip the `gh` command, complete only up through `git push`, and tell the user:

> "`gh` is not available. The branch has been pushed. Open the PR manually:
> - URL: `https://github.com/<owner>/<repo>/compare/prd/<slug>`
> - Title: `Draft PRD: <feature name>`
> - Body: `PRD draft: <feature name>.`
> - Mark as draft in the GitHub UI."

Derive `<owner>/<repo>` from `git remote get-url origin` if possible.

## Step 5. Promote draft → ready

Triggered when the user explicitly asks to promote a previously-drafted PRD ("promote the bulk export PRD", "this PRD is ready", "move to ready"). If no specific PRD is named and the context is ambiguous, list the files in `draft/` and ask which one to promote.

This step lives in the same skill as Step 4 — it operates on a PRD that has already gone through Step 4 and a draft PR has been reviewed and approved.

### Step 5a. Verify pre-conditions

Open the draft PRD and check all of the following. If any fails, stop and report what's blocking:

- **No assumption markers remain.** Search the file for the substring `Assumption — to confirm` (italic marker from Step 1d). If any remain, list them and ask the user to resolve them in the PRD first.
- **No `Skipped during discovery` markers remain.** Same handling.
- **§8 Open questions is either `None — all items resolved` or every checkbox is `[x]`.** If any unchecked items remain, list them and stop.
- **The draft PR (from Step 4) has been approved.** Ask the user to confirm approval explicitly: *"Has the draft PR been approved by reviewers?"* Wait for `yes` before proceeding.

Only proceed if all four pass.

### Step 5b. Pre-flight checks

Same as Step 4a — fetch, checkout main, pull, status. Stop and ask if anything is off.

### Step 5c. Update Status in the file

Edit the PRD file at `docs/requirements/draft/YYYY-MM-DD-<slug>.md` — change the `**Status:**` field from `draft` to `ready`. The Status field always mirrors the folder location.

### Step 5d. Offer the promotion workflow

Present this workflow and wait for explicit confirmation before running:


```bash
git checkout -b promote/<slug>
git mv docs/requirements/draft/YYYY-MM-DD-<slug>.md \
       docs/requirements/ready/YYYY-MM-DD-<slug>.md
git commit -m "docs(prd): promote <feature name> to ready"
git push -u origin promote/<slug>
gh pr create \
  --title "Promote PRD: <feature name> to ready/" \
  --body "Promoting <feature name> from draft to ready."
```


Notes on this block:
- Branch prefix is `promote/`, not `prd/`, to distinguish from the original draft PR.
- `git mv` carries the Status edit (from Step 5c) with the file in a single rename. No separate `git add` needed.
- This PR is **not** `--draft` — it's a real merge request for the promotion.
- Same `gh` fallback as Step 4d: if `gh` is not available, push only and give the user the manual URL with title and body.

Wait for explicit confirmation before running. If the user declines, leave the file edit and the rename ready on disk and stop.

## Notes

- Write the PRD in English regardless of the user's chat language (see Language and terminology section).
- One PRD per file. If the user describes two features, ask which to PRD first or whether to split.
- If the idea is too vague for a meaningful PRD, push back and ask for tighter scope before writing.
- **Repo hygiene recommendation:** enable *"Automatically delete head branches"* in your GitHub repository settings. Once a PR is merged, the corresponding `prd/...` or `promote/...` branch will be cleaned up automatically — without this, dead branches accumulate.
