---
name: jira-test-cases
description: Generate structured manual test cases from a Jira ticket (story, bug or epic). Use when the user gives a Jira key or URL, or pastes ticket text, and asks for test cases, test scenarios or acceptance-criteria coverage.
---

# Jira ticket to test cases

Turn a Jira ticket into a reviewable set of test cases that trace back to the ticket's acceptance criteria.

## Inputs

- A Jira issue key (e.g. `ABC-123`) or URL, **or** pasted ticket text.
- Optional: target format (Markdown table by default; CSV if the user wants to import into a test tool).

## Steps

1. **Get the ticket.** If a Jira/Atlassian tool is available in this session, fetch the issue (summary, description, acceptance criteria, linked issues, comments). If not, ask the user to paste the ticket text. Never guess ticket content.
2. **Extract testable statements.** List every acceptance criterion, business rule, validation, role/permission and error condition as a numbered requirement (R1, R2, ...). If the ticket has no acceptance criteria, derive them from the description and flag them as *assumed*.
3. **Ask before assuming.** If a requirement is ambiguous or contradictory, list the questions in one batch and wait for answers, or mark affected cases as `Needs clarification`.
4. **Design cases.** For each requirement cover, where relevant:
   - Happy path
   - Negative / invalid input
   - Boundary values
   - Permissions / roles
   - Error handling and messages
   - Regression risk in linked or adjacent functionality
5. **Write the cases** in the output format below. Each case maps to at least one requirement ID.
6. **Coverage check.** End with a traceability table (requirement -> case IDs). Any requirement with no case must be called out.

## Output format

| ID | Title | Requirement | Priority | Preconditions | Steps | Expected result |
|----|-------|-------------|----------|---------------|-------|-----------------|
| TC-001 | ... | R1 | High | ... | 1. ... 2. ... | ... |

- IDs: `TC-001`, `TC-002`, ...
- Priority: High / Medium / Low, based on business impact and risk.
- Steps are atomic and written so someone unfamiliar with the feature can follow them.
- Expected results are observable and specific (exact message text, state change, data saved), never "works correctly".

After the table add:

- **Assumptions and open questions**
- **Traceability** (requirement -> cases)
- **Out of scope** (what these cases deliberately do not cover)

## Rules

- Do not invent features, fields or messages that are not in the ticket; mark them as assumed.
- Keep cases independent; each should be runnable on its own given its preconditions.
- Do not include credentials, customer data or internal URLs in the output.
