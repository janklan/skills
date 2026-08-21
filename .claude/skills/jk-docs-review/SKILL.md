---
description: Verify project documentation against reality and report findings for review
version: 1
---

Scope: every documentation file in this project (README, CLAUDE.md, docs/, other Markdown)
plus code comments that state facts about behaviour. Open the artefact with the list of
files reviewed so omissions are visible.

Method: read each documentation file in full and extract every falsifiable claim. Verify
the claims in parallel with subagents: read the referenced code verbatim; where reading
cannot settle a claim, run the relevant command. Never rely on conversation history or
prior knowledge. A claim you cannot settle gets verdict UNVERIFIED with the reason - never
a guess.

Findings: false or outdated statements, ambiguities, verbose prose, LLM slop. Every finding
must quote the claim, cite its file and line, and cite the evidence (code location or
command output). A finding without evidence must not appear.

Identifiers: letter = category (F false, O outdated, A ambiguous, V verbose/slop), number =
sequence. Number proposed solutions within their finding (F3.1, F3.2).

Output: a reviewable HTML artefact, not a terminal response. One row per finding: id,
location, quoted claim, evidence, proposed solutions. Factual findings first, style after.
End with the UNVERIFIED list.
