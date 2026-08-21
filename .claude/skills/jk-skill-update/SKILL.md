---
description: Pull the latest version of skills from Github
version: 1
---

Source: https://github.com/janklan/skills, branch `main`, directory `.claude/skills/`.
Target: `${CLAUDE_PROJECT_DIR}/.claude/skills/`. Only directories named `jk-*` are in scope.
Do not touch any other skill in the target.

Every fact about the source comes from a fresh clone made in this run. Never rely on
conversation history or prior knowledge of the repository. The target project's own git
state is irrelevant: the checks below hash file content and query the clone only.

## Procedure

1. Clone the source with full history:
   `SRC=$(mktemp -d) && git clone --quiet https://github.com/janklan/skills.git "$SRC"`

2. For each `jk-*` directory in `$SRC/.claude/skills/` and in the target, read `version` from
   the SKILL.md frontmatter. A local `version` names the upstream version the local wording
   is based on; local edits never change it. Build a table: skill, upstream version, local
   version, class.

3. Pristine test for a local file: the command below prints at least one commit. Empty output
   means the user altered the file.
   `git -C "$SRC" log --format=%H --find-object=$(git hash-object <local file>) -- <path in source>`
   Apply the test to SKILL.md and to every other file in the local skill directory.

4. Classify each skill:
   - `new`: upstream only.
   - `current`: local version equals upstream version, every local file pristine.
   - `stale`: local version below upstream, every local file pristine.
   - `ahead`: local version above upstream.
   - `altered`: at least one local file not pristine, or a local file that upstream lacks.

5. Act:
   - `current`, `ahead`: no change. Report `ahead` as unexpected.
   - `stale`: `rm -rf <target>/<skill> && cp -R "$SRC/.claude/skills/<skill>" <target>/`
   - `new`: one multi-select question listing the new skills; copy the ones the user picks.
   - `altered`: re-apply the user's adjustments, see below. Never discard them.

6. Print the table with the action taken per skill. Remove `$SRC`.

## Re-apply user adjustments to an altered skill

Three inputs per file:

- Base: the upstream file at the latest commit on `main` whose frontmatter `version` equals
  the local version. Find it with `git -C "$SRC" log --format=%H -- <path>` and
  `git -C "$SRC" show <commit>:<path>`. No such commit: show local and upstream in full and
  ask the user how to proceed.
- Local: the file in the target.
- Upstream: the file at `main`.

Method:

1. Read base, local, and upstream in full. Diff base against local and state each user
   adjustment as one intent: what it changes and what it is for.
2. Diff base against upstream and state what the new version changed, so each adjustment can
   be placed in the new structure.
3. Write the result starting from upstream, verbatim, and apply each adjustment where it
   belongs in the new version. Keep the user's wording unless the surrounding text changed;
   then change only what the new context requires. Keep the upstream `version`.
4. Sort each adjustment into one of:
   - applied: the target context still exists, possibly moved.
   - superseded: upstream now covers the same intent. Drop it and tell the user.
   - orphaned: the text it modified no longer exists upstream. Ask the user.
   - conflicting: upstream changed the same text with a different intent. Ask the user,
     showing the user's version, the upstream version, and the adjustment's intent.
5. Show the user the result as a diff against upstream together with the sorted list, and
   ask to confirm before writing. The diff must contain the user's adjustments and nothing
   else.

Files the user added to the skill directory stay. Files upstream added are copied.

Do not add text that neither side contains. Do not reword, tighten, or improve upstream text
or the user's text. When a decision has to be made or an ambiguity arises, ask the user.
