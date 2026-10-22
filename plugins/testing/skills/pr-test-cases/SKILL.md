---
name: pr-test-cases
description: Turn a Jira ticket and its pull request into data-driven and state-driven UI and API test cases in Given/When/Then form, written to a CSV file for import. Expected results come only from the ticket; if the ticket does not state one, ask the tester. Use when a tester asks for test cases, a CSV of test cases, or to "create test cases for this ticket / PR".
---

# PR test cases

Produce reviewable test cases (UI and API) as a CSV a tester can import, written so a business person can read them. The code decides **what to test**. The ticket and the tester decide **what is expected**.

## The rule that matters most

**Never take an expected result from the code.** Code shows what the system does, not what it should do; using it would turn bugs into passing tests.

- Expected results come only from the Jira ticket: description, acceptance criteria, comments, a developer's QA changelist, linked requirements.
- If the ticket does not state the expected result for a scenario, **ask the tester**. Do not infer it from the diff, the PR description, the PR's own tests or code comments, and do not write "assumed".
- The code may be used to find states, data variations, roles, entry points and affected areas, and to check that a ticket statement is testable. It is never the source of a `Then`.
- If the ticket has no usable expectations at all, say so and ask the tester for them before writing anything.

## Inputs

- A Jira ticket URL or key (the usual input), or a PR number/URL or branch. A ticket is needed for expected results; if only a PR or branch is given, ask for the ticket key or for the expected behaviour in the tester's own words.
- If `pr-test-insights` has already been run in this conversation for the same change, reuse its findings instead of re-reading the diff.

## Steps

1. **Get the ticket and the change.** Follow steps 0 and 1 of `pr-test-insights`: extract the key, fetch the ticket with the Jira tool (or ask the tester to paste it), find the PR(s) by key, read the changes. If no ticket text is available, ask the tester to paste it.
2. **List the expectations the ticket gives.** Extract every statement of expected behaviour (acceptance criteria, the described fix, "QA changelist" items, comments that restate behaviour). Number them E1, E2... Each test case must trace to one.
3. **Find what varies and what changes state, using the code.**
   - *Data-driven:* inputs that follow the same rule with different values (types, options, roles, boundaries, empty or long values, several records sharing a value).
   - *State-driven:* flags, statuses, modes and transitions the change touches, and the events that move between them (open/closed, online/offline, loading/loaded, has/has no records, session switched, repeated action).
   - *UI level:* checked by a person using the screen: changed screens, controls, navigation, messages, accessibility and responsive behaviour the ticket mentions.
   - *API level:* checked by calling an endpoint directly, with no screen: changed endpoints, request and response fields, validations, roles and status codes. If nothing changed on the server, write no API cases and say so in the summary; never invent endpoints or fields.
   - Put each scenario at exactly one level. A result seen on screen is a UI case even if a server call sits behind it. If the ticket expects server behaviour that can be checked without the screen (a status code, a stored value, a validation error), that is an API case. If the ticket is silent on server behaviour that the change depends on, ask the tester whether they want API coverage and what the expected response is.
4. **Match scenarios to expectations.** For each scenario from step 3 decide: (a) the ticket states the expected result, so write the case; (b) the ticket is silent or vague, so it is an **open question**; (c) it is internal with no user-visible effect, so skip it.
5. **Ask the tester about the open questions, once, in one batch.** Give each question the scenario in plain words and the information you need, for example: "When a student has no steps left and the trainer returns, what should they see?" Wait for answers. Use only what the tester says. If they cannot say, leave the scenario out of the CSV and list it under "Not written".
6. **Write the cases in Given/When/Then.** Plain business wording; use the exact screen, button and field names that appear in the ticket or code, no function or variable names. Rules below.
7. **Write the CSV file** and show the tester a short summary (below).
8. **Check the file**: read it back with a CSV parser (for example Python's `csv` module) and confirm every row has the same number of columns as the header, there are no empty required cells, and IDs are unique. Fix and recheck if not.

## Writing the cases

- **One behaviour per case.** Cover, in priority order: the change working as intended (P1), negative and boundary cases (P1/P2), state transitions (P1/P2), roles and permissions (P2), regression of what the change could affect (P2/P3).
- **Data-driven cases are expanded, one CSV row per data combination**, because importers need concrete rows. Give the combination in `Test Data` and share the same `Title` stem. Keep the set meaningful (one row per distinct behaviour or boundary), not every permutation.
- **State-driven cases name the starting state in `Given` and the event in `When`.** Add a case for each transition the ticket gives an expectation for, including the "nothing should happen" ones.
- **Pre-requisites hold everything that must exist before the test**: accounts and roles, products, sessions, records and their states, flags and configuration, environment. List each item as a short numbered line. `Given` describes the situation from the user's side; Pre-requisites says what data to create to get there. Use test accounts only if the ticket names them; never include passwords, tokens or customer data.
- **UI cases:** `Target` names the screen and area. `Given`, `When` and `Then` are what the user sees and does.
- **API cases:** `Target` holds the method and endpoint, taken from the ticket or code. `Given` states the caller and role, `When` the request in business terms, `Then` the response the ticket expects (status and the fields that matter). Put request values in `Test Data`. Do not describe screens in an API case or HTTP details in a UI case.
- Use `Then` for observable results only. Several results go in one cell, one per line.

## CSV format

- File name: `<TICKET-KEY>-test-cases.csv` in the current directory unless the tester names another location. UTF-8, comma-separated, header row.
- Quote every field. Escape a double quote inside a field by doubling it. Line breaks inside a cell are allowed (used for numbered pre-requisites).
- Columns, in this order:

| Column | Content |
|--------|---------|
| `Test ID` | The level is in the ID: `<KEY>-UI-01`, `<KEY>-API-01`... numbered separately per level. Data-driven rows use `-03a`, `-03b`... |
| `Title` | Short, outcome-oriented |
| `Type` | `UI` or `API` |
| `Target` | UI: the screen and area. API: method and endpoint |
| `Category` | `Functional`, `Data-driven`, `State-driven`, `Negative`, `Role`, `Regression`, `Accessibility` or `Responsive` |
| `Priority` | `P1`, `P2` or `P3` |
| `Pre-requisites` | Data and setup that must exist, numbered |
| `Given` | Starting situation |
| `When` | Action or event |
| `Then` | Expected results |
| `Test Data` | Values for this row (blank if none) |
| `Expected Result Source` | `Ticket <KEY>` (name the section or comment author), or `Tester` for answers given in the session |
| `Covers` | Expectation id, for example `E3` |

Order the rows UI first, then API, so the two levels read as separate blocks and filter cleanly on `Type`. If the tester prefers separate files, write `<KEY>-test-cases-ui.csv` and `<KEY>-test-cases-api.csv` instead (same columns, same IDs).

If the tester's import tool needs different column names or order, ask for the header row and map to it instead of arguing with it.

## Output after writing the file

1. The file path and the number of cases, split **UI level / API level** and by priority. If there are no API cases, say why (for example "the ticket states no server or data changes").
2. A table of the expectations E1... and which cases cover each one. Flag any expectation with no case.
3. **Not written**: scenarios skipped because the expected result was unknown, each as a plain question.
4. Open assumptions: none should exist, because every `Then` comes from the ticket or the tester. If anything slipped in, remove it.

## Rules

- Every `Then` traces to the ticket or to something the tester said in this session. No exceptions.
- Do not invent screens, fields, messages, endpoints, data or users. Quote exact names from the ticket or code, or leave them out.
- No secrets, credentials or customer data in the file.
- Keep the file reviewable: prefer a prioritised, non-redundant set over exhaustive coverage. Skip internal-only changes.
- Writes only the CSV file. Do not post it to Jira, a PR or chat, and do not commit or push, unless the user asks. Do not build or run tests.
- One ticket or change per file. For release-wide planning use a release-analysis skill if the repo has one.
