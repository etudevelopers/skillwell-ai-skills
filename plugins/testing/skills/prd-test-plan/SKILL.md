---
name: prd-test-plan
description: Draft a test plan from a product requirements document (PRD). Use when the user shares a PRD (file, link or pasted text) and asks for a test plan, test strategy or QA scope.
---

# PRD to test plan

Produce a concise, reviewable test plan from a PRD.

## Steps

1. **Read the whole PRD.** If it is a file path, read it; if it is a link and a tool to fetch it is available, fetch it; otherwise ask the user to paste it.
2. **Summarise scope** in 3-5 lines: what is being built, for whom, and what is explicitly out of scope.
3. **List requirements** as numbered items (R1, R2, ...) with their source section in the PRD.
4. **Flag gaps.** Note missing acceptance criteria, undefined error behaviour, unclear non-functional targets, and unstated dependencies. Ask the user about the important ones.
5. **Write the plan** using the structure below.

## Plan structure

1. **Overview and scope** (in / out)
2. **Test approach**: levels (unit, integration, API, UI, E2E), what is manual vs automated, and why
3. **Test areas**: one subsection per feature area, listing the scenarios to cover and linked requirement IDs
4. **Non-functional testing**: performance, security, accessibility, compatibility, localisation, only where the PRD implies them
5. **Test data and environments** needed
6. **Risks and mitigations**
7. **Entry and exit criteria**
8. **Open questions and assumptions**
9. **Traceability matrix**: requirement -> test areas

## Rules

- Every requirement must appear in the traceability matrix; call out any not covered.
- Prioritise by risk (business impact x likelihood of failure) and say so.
- Keep it to what a QA lead can review in ten minutes; link to detailed cases rather than listing hundreds. Use the `jira-test-cases` skill for case-level detail on individual tickets.
- Do not invent requirements; mark inferences as assumptions.
