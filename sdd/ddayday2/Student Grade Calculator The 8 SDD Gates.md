# Student Grade Calculator: The 8 SDD Gates

Nov 18, 2026 · @Abhi

## Overview

The Student Grade Calculator case study passes through 8 human gates, and nothing moves forward until the current gate is approved. The original write-up has 6 steps and 4 roles; splitting Tasks, Build and Review into their real checkpoints gives 8 gates. A failure at any gate sends work back to the previous owner.

| # | Gate | Owner | Input checked | Passes when |
| --- | --- | --- | --- | --- |
| 1 | Constitution approval | Architect | 4 ground rules | Rules are clear, testable and permanent |
| 2 | Spec approval | Product owner | Goal, grade bands, validation, Done-when table | Every decision is written down, including boundaries |
| 3 | Plan approval | Architect | Design table | Every design choice obeys the constitution |
| 4 | Task breakdown approval | Architect + developer | 5 tasks | Each task fits one AI prompt and one quick check |
| 5 | Per-task build check | Developer | AI output for each task | Code matches the spec before the next task starts |
| 6 | Functional tests | Reviewer | 6 Done-when tests | Actual result equals expected result |
| 7 | Edge case + constitution review | Reviewer | Mobile layout, errors, rules 1–4 | No crash, friendly errors, fits 375 px |
| 8 | PR approval and release | Reviewer | Evidence screenshots | All 7 tests pass with proof |

In the case study, the owner of Gate 4 is not named; the table assumes the architect and developer share it.

## Gate 1: Constitution approval

**Owner:** Architect. **Output:** the project's permanent ground rules.

The architect approves four rules that every feature, now and later, must follow:

1. Simple HTML and JavaScript in a single file, with no external libraries.
2. Every feature has test examples written in its spec.
3. Friendly error messages; the app never crashes.
4. Works on mobile screens.

**Pass check:** each rule can be verified later. Rule 4 is checked at Gate 7 with a 375 px screen; rule 2 is checked at Gate 2 by the Done-when table.

**Fail action:** rewrite the rule until it is testable. A vague rule here weakens every gate below it.

## Gate 2: Spec approval

**Owner:** Product owner. **Output:** WHAT the app does, approved before any design or code.

The product owner confirms:

- **Goal:** help a teacher find a student's grade from their marks.
- **Inputs:** student name (text) and marks out of 100 (number).
- **Output:** name with grade, for example "Priya: Grade A".
- **Grade bands:** 90–100 = A, 75–89 = B, 50–74 = C, 0–49 = F.
- **Validation:** name not empty; marks a whole number from 0 to 100.
- **Done-when table:** 6 input/expected pairs, including the 90 and 89 boundaries.

**Pass check:** every question vibe coding would leave to guesswork has a written answer: the 90 boundary, 120 marks, an empty name.

**Fail action:** the spec returns to the product owner. No plan starts on an unapproved spec.

## Gate 3: Plan approval and constitution check

**Owner:** Architect. **Output:** HOW the spec will be built.

The architect checks each design decision against the four constitution rules:

| Design decision | Constitution rule it satisfies |
| --- | --- |
| One HTML file with name box, marks box, "Get Grade" button | Rule 1: single file, no libraries |
| `getGrade(marks)` returns A, B, C or F from the spec's bands | Rule 2: maps to spec tests |
| `validate(name, marks)` returns a friendly message or nothing | Rule 3: friendly errors, no crash |
| Green card for a grade, red card for an error | Rule 3: clear feedback |
| Single column, max width 420 px | Rule 4: works on mobile |

**Pass check:** no design choice breaks a rule, and every spec rule has a place in the design.

**Fail action:** the plan is revised, or a justified exception is recorded before work continues.

## Gate 4: Task breakdown approval

**Owner:** Architect and developer. **Output:** 5 tasks, each small enough for one AI prompt.

1. Create the page with the name box, marks box and "Get Grade" button.
2. Write `getGrade()` using the grade bands.
3. Write `validate()` for empty names and invalid marks.
4. Show the result as a green grade card or a red error card.
5. Test every Done-when example and the mobile layout.

**Pass check:** every plan item maps to at least one task, and each task has one clear, checkable result.

**Fail action:** split any task that would need more than one prompt, or add a task for an uncovered plan item.

## Gate 5: Per-task build check

**Owner:** Developer. **Output:** working code, one task at a time.

The developer gives the AI one task per prompt, always with the spec attached, and checks the result before the next task. Example prompt for Task 2:

> Following the attached spec, do Task 2: write a `getGrade(marks)` function that returns A, B, C or F using the exact grade bands in the spec.

```javascript
function getGrade(m) {
  if (m >= 90) return "A";
  if (m >= 75) return "B";
  if (m >= 50) return "C";
  return "F";
}

function validate(name, raw) {
  if (!name.trim()) return "Please enter the student's name";
  if (raw.trim() === "" || isNaN(Number(raw))) return "Marks must be a number";
  const m = Number(raw);
  if (m < 0 || m > 100) return "Marks must be between 0 and 100";
  if (!Number.isInteger(m)) return "Marks must be a whole number";
  return null;
}
```

**Pass check:** the code mirrors the spec line by line: thresholds 90, 75 and 50, and one message per validation rule.

**Fail action:** re-prompt the AI for that task only; later tasks wait.

## Gate 6: Functional Done-when tests

**Owner:** Reviewer. **Output:** pass/fail result for each spec test.

All 6 Done-when tests passed.

| Test | Input | Expected | Actual | Result |
| --- | --- | --- | --- | --- |
| Normal A | Priya, 95 | Priya: Grade A | Priya: Grade A | Pass |
| Normal C | Rahul, 60 | Rahul: Grade C | Rahul: Grade C | Pass |
| Marks too high | Amit, 120 | Marks error | Marks must be between 0 and 100 | Pass |
| Empty name | (empty), 80 | Name error | Please enter the student's name | Pass |
| Boundary A | Sneha, 90 | Sneha: Grade A | Sneha: Grade A | Pass |
| Boundary B | Arjun, 89 | Arjun: Grade B | Arjun: Grade B | Pass |

**Pass check:** actual equals expected for every row, with a screenshot as evidence.

**Fail action:** the failing task returns to Gate 5.

## Gate 7: Edge case and constitution compliance review

**Owner:** Reviewer. **Output:** proof the build obeys the constitution, not just the spec.

| Constitution rule | How the reviewer checks it | Result |
| --- | --- | --- |
| 1. Single HTML file, no libraries | Inspect the file for external imports | Pass |
| 2. Tests written in the spec | Done-when table exists and was used | Pass |
| 3. Friendly errors, never crash | Invalid inputs show red error cards (Amit, 120; empty name) | Pass |
| 4. Works on mobile | Load at 375 px width | Pass: fits screen |

**Pass check:** all four rules hold in the running app.

**Fail action:** a layout or error-handling fix returns to Gate 5; a rule that proved unworkable goes back to the architect at Gate 1.

## Gate 8: PR approval and release

**Owner:** Reviewer. **Output:** an approved pull request.

The reviewer approves the PR once all 7 tests (6 Done-when tests plus the mobile layout) pass, each backed by a screenshot. In the case study, all 7 passed on the first review.

**Evidence pack:** 8 screenshots: the empty app, the 375 px mobile view, and one per test result.

**Pass check:** every earlier gate is signed off and every test has evidence.

**Fail action:** the PR is rejected and returns to the developers. Nothing reaches the final product without passing all 8 gates.

## Gaps to close in the next iteration

The gates worked, but a strict reviewer would flag four items the case study passed over:

- **Error wording:** the spec expects "Error: Marks must be between 0 and 100"; the app shows it without the "Error:" prefix, yet the test passed. Gate 2 should state whether the prefix is required.
- **Untested validation rules:** the spec requires whole numbers, and the problem statement mentions -5 marks, but no Done-when test covers a negative value, a decimal like 85.5, or non-numeric text. Gate 2 should add them.
- **Lower boundaries:** 90/89 is tested, but the 75/74 and 50/49 boundaries and the 0 and 100 extremes are not.
- **Test count:** the spec lists 6 Done-when tests while the review counts 7. The mobile test comes from constitution rule 4, so it belongs at Gate 7, as shown above.
