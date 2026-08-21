# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Jan's personal Claude Code skills, hosted at `github.com/janklan/skills` and used in all of Jan's projects. There is no application code, build, or test suite. Each skill is one directory under `.claude/skills/` that holds a `SKILL.md`.

## Layout

- `.claude/skills/<name>/SKILL.md` - one skill per directory. The directory name is the command name (`/jk-docs-review`). The `name` frontmatter field is not set; Claude Code defaults it to the directory name.
- Supporting files (reference documents, scripts, templates) sit next to `SKILL.md` and are referenced from it with Markdown links or `${CLAUDE_SKILL_DIR}`.
- Skills under `.claude/skills/` load as project skills in every session started in this repository. That is how a skill is exercised before it ships.

## Conventions

- Prefix every skill directory with `jk-` so a name never collides with a built-in, plugin, or other project skill.
- Frontmatter carries `description` and `version`. `description` is one line, written so Claude can decide when the skill applies; the skill listing truncates `description` plus `when_to_use` at 1,536 characters. `version` is an integer that names a published state of the skill. It changes in one place only: here, when an edit to the skill lands on `main`. In a consuming project the field says which upstream version the local wording is based on; `/jk-skill-update` uses it to pick the comparison base and sets it on update. A local edit to a copy never changes it.
- The body is an instruction set for Claude, not documentation for a human. State scope, method, and output format. Apply `~/.claude/rules/comments-and-docs.md`: terse, ASCII only, hyphens not dashes, no emojis.
- Arguments come in through `$ARGUMENTS`, `$0`, or named `arguments`; shell output is injected with `` !`command` ``.

## Commands

- `claude plugin validate --strict .` - run from the repository root. Checks every `SKILL.md` under `.claude/skills/`: fails on unparseable YAML frontmatter and on a missing frontmatter block. It does not flag unknown frontmatter fields or an empty body, so read the skill after editing.
- Do not point `validate` at a single skill directory; it then expects a plugin manifest and fails with "No manifest found".
- To test a skill, start Claude Code in this repository and invoke `/jk-<name>`.

## Distribution

Skills reach other projects by copy, not by symlink. `/jk-skill-update`, run inside a consuming project, clones this repository with full history, compares each published skill's `version` with the local copy under that project's `.claude/skills/`, replaces pristine copies that are behind, and re-applies the user's local adjustments onto the new version of altered copies. It finds the base for that re-application by looking up the commit whose `version` equals the local one, which is why every edit that lands on `main` needs a new `version`. Only what is on `main` is published, so a skill is live for every project as soon as it lands there.
