---
name: brain-init
description: >-
  Register the current repository (or one the user names) with a central Obsidian "second brain" vault.
  Creates the vault project note from a template AND scaffolds the .brain/ bridge folder in the repo.
  Use when the user says "/brain-init", "register this repo with the brain / my second brain",
  "add this to the brain", "scaffold .brain here", "trackear este repo en el cerebro",
  "registra este proyecto en mi segundo cerebro", or starts a new project that should be tracked centrally.
  The vault location comes from the BRAIN_PATH env var — never hardcode a path.
---

# Initialize a project in your second brain

You are registering a repository with a **central Obsidian vault** (the "brain"). The repo becomes a tracked project with a bidirectional `.brain/` bridge.

## Language / Idioma
Reply in the user's language. If they wrote Spanish, run the whole interaction in Spanish (questions, confirmations, summary); if English, English. Keep frontmatter field *names* in English (the sync script parses them).

## Locate the vault (no hardcoded paths)

1. Read `BRAIN_PATH` from the environment. That is the vault root.
2. If `BRAIN_PATH` is unset: ask the user once for the absolute path to their Obsidian vault, and suggest they add it to their `.env` / shell so future sessions skip this. Do **not** invent or assume a path.
3. The vault must contain `Projects/` and `Templates/Project.md`. If `Templates/Project.md` is missing, offer to copy the starter from this skill category's `templates/Project.md`.

Throughout this skill, `$BRAIN` = the resolved `BRAIN_PATH`.

## What you'll do

1. **Detect the host repo** — current working directory, or a path the user passes after `/brain-init`.
2. **Gather metadata** (ask one question at a time, multiple-choice where possible).
3. **Create the project note** at `$BRAIN/Projects/<Slug>.md` from `$BRAIN/Templates/Project.md`.
4. **Run the sync** so `.brain/` gets scaffolded in the repo.

## Step-by-step

### Step 1 — Detect / confirm host repo
- Default to the current working directory; if invoked with a path argument, use that.
- Confirm: *"Register brain tracking for `<path>`?"*

### Step 2 — Ask for metadata (one question at a time)
Use `AskUserQuestion`. These options are **suggested defaults — adapt them to the user's domain**, they are not fixed:

**Type:** `client-delivery` · `client-support` · `internal-tool` · `research` · `legacy` · `admin`
**Status:** `planning` · `active` · `on-hold` · `archived`
**Priority:** `p0` (drop everything) · `p1` (this week) · `p2` (later)
**Client / Owner:** free-text (blank if internal — do **not** invent one)
**Scope:** one sentence → goes in `## Scope`
**Attention** (optional): integer rank if it belongs in the user's top-active list

### Step 3 — Slug + collision check
```
slug = <ProjectName>: spaces → hyphens, drop chars except [A-Za-z0-9_-]
notePath = $BRAIN/Projects/<slug>.md
```
If `notePath` exists, ask to overwrite or pick a new slug.

### Step 4 — Compute the `folder:` field
`folder:` is the repo path **relative to `$BRAIN/Projects/`**, so the sync can resolve the repo from the vault. Examples (vault at `$BRAIN`, sibling repos one level up):
- repo `<parent>/Aclara` → `folder: "../../Aclara"`
- nested `<parent>/Aclara/exp-x` → `folder: "../../Aclara/exp-x"`

Compute it correctly for the actual relative location — do not assume the example layout.

### Step 5 — Write the project note
Read `$BRAIN/Templates/Project.md`, substitute `{{title}}` → display name, `{{date}}` → today (YYYY-MM-DD), fill frontmatter from the answers, set `started:` and `last-activity:` to today, inject the computed `folder:`. Leave unanswered optional fields empty. Write with the `Write` tool, UTF-8.

### Step 6 — Run the sync
```powershell
pwsh <path-to>/Sync-Project-Brains.ps1 -BrainRoot "$env:BRAIN_PATH"
```
This pushes `CONTEXT.md`, `TASKS.md`, `ASKS.md`, `INSTRUCTIONS.md` into the repo's `.brain/` and seeds empty `STATUS.md`, `LOG.md`, `ANSWERS.md`.

### Step 7 — Confirm + brief
Tell the user: note created at `$BRAIN/Projects/<slug>.md`, `.brain/` scaffolded in the repo, and that the agent in this repo now follows the `brain-sync` protocol. Suggest opening the note to fill milestones, team, and the `Brain → Repo · Tasks` / `· Asks` sections.

## Rules
- ⚠ Don't pre-fill metadata the user didn't give. No client? Leave it blank.
- ⚠ Don't manually edit vault dashboards/indexes — they auto-aggregate (Dataview) on sync.
- ⚠ Don't duplicate `INSTRUCTIONS.md` into project docs — the sync generates it.
- ✅ Do verify `BRAIN_PATH` resolves to a real vault before writing anything.
