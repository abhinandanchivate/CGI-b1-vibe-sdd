# Case Study: Building a Student Grade Calculator with Spec-Driven Development

Nov 18, 2026 · @Abhi

## Overview

This case study builds a small Student Grade Calculator using Spec-Driven Development (SDD), and every test in the spec passed on the first review. It walks through each SDD stage in order: constitution, spec, plan, tasks, build and review.

The app is deliberately tiny so the process stays visible. A teacher enters a student's name and marks out of 100, and the app returns a grade (A, B, C or F). The screenshots in the Build and Review sections are taken from the real, working app.

## The problem

A one-line prompt like "make a grade calculator" leaves every important decision to the AI's guess. That is the core weakness of vibe coding.

Left to guess, the AI may get these wrong without anyone noticing:

- Whether exactly 90 marks is an A or a B (a boundary case).
- What happens when someone types 120 or -5 marks.
- What happens when the name box is left empty.
- Whether the page works on a phone.

SDD fixes this by writing these decisions down before any code exists, so the AI builds to a plan and reviewers test against it.

## Roles: the SDD chain

Work passes through four roles, and each one is a human checkpoint before the next step starts.

| Role | Responsibility | Gate they control | In this case study |
| --- | --- | --- | --- |
| Product owner | Decides WHAT to build | Approves the spec | Confirms the grade bands and the 0–100 rule |
| Architect | Decides HOW to build it | Approves the constitution and plan | Chooses plain HTML + JavaScript, mobile-first |
| Developers + AI | Build the code task by task | Implement each task | Developer prompts the AI one task at a time |
| Reviewers | Check the code against the spec | Approve or reject the pull request (PR) | Run every "Done when" test and edge case |

If a reviewer finds a failure, the work goes back to the developers. Nothing reaches the final product without passing every gate.

## Step 1: Constitution

The constitution holds the project's permanent ground rules, approved by the architect. Every feature, now and later, must follow them.

1. Use simple HTML and JavaScript in a single file, with no external libraries.
2. Every feature must have test examples written in its spec.
3. Show friendly error messages and never crash.
4. Must work on mobile screens.

The constitution changes rarely. Specs change per feature, but they must always respect these rules.

## Step 2: Spec

The spec defines WHAT the app does, and the product owner approved it before any design or code.

**Goal:** Help a teacher find a student's grade from their marks.

**Inputs:** Student name (text) and marks out of 100 (number).

**Output:** The student's name with their grade, for example "Priya: Grade A".

**Grade rules:**

| Marks | Grade |
| --- | --- |
| 90–100 | A |
| 75–89 | B |
| 50–74 | C |
| 0–49 | F |

**Validation rules (edge cases):**

- The name cannot be empty.
- Marks must be a number between 0 and 100.
- Marks must be a whole number.

**Done when:**

| Input | Expected result |
| --- | --- |
| Priya, 95 | Priya: Grade A |
| Rahul, 60 | Rahul: Grade C |
| Amit, 120 | Error: Marks must be between 0 and 100 |
| (empty name), 80 | Error: Please enter the student's name |
| Sneha, 90 | Sneha: Grade A (boundary) |
| Arjun, 89 | Arjun: Grade B (boundary) |

## Step 3: Plan

The plan defines HOW the spec will be built, and the architect checked it against the constitution.

| Part | Design decision |
| --- | --- |
| Page | One HTML file with a name box, a marks box and a "Get Grade" button |
| `getGrade(marks)` | Returns A, B, C or F using the spec's grade bands |
| `validate(name, marks)` | Returns a friendly error message, or nothing if the input is valid |
| Result area | Green card for a grade, red card for an error |
| Layout | Single column, max width 420 px, so it fits phone screens |

## Step 4: Tasks

The plan was split into five small tasks, each small enough for one AI prompt and one quick check.

1. Create the page with the name box, marks box and "Get Grade" button.
2. Write the `getGrade()` function using the grade bands.
3. Write the `validate()` function for empty names and invalid marks.
4. Show the result as a green grade card or a red error card.
5. Test every "Done when" example and the mobile layout.

## Step 5: Build

The developer gave the AI one task at a time, always with the spec attached, and checked each result before moving on. A typical prompt for Task 2 looked like this:

> Following the attached spec, do Task 2: write a `getGrade(marks)` function that returns A, B, C or F using the exact grade bands in the spec.

The two core functions the AI produced map directly to the spec's rules:

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

**Screenshot 1: the finished app before any input.** It follows the plan: a name box, a marks box and one button, in a single column.

&#91;image: The finished Student Grade Calculator before any input\]

**Screenshot 2: the mobile view (375 px wide).** The layout fits a phone screen, which satisfies constitution rule 4.

&#91;image: Mobile view at 375 px showing Priya: Grade A\]

## Step 6: Review

The reviewer ran all six "Done when" tests from the spec, and all six passed, so the pull request was approved.

| Test | Input | Expected | Actual | Result |
| --- | --- | --- | --- | --- |
| Normal A | Priya, 95 | Priya: Grade A | Priya: Grade A | Pass |
| Normal C | Rahul, 60 | Rahul: Grade C | Rahul: Grade C | Pass |
| Marks too high | Amit, 120 | Marks error | Marks must be between 0 and 100 | Pass |
| Empty name | (empty), 80 | Name error | Please enter the student's name | Pass |
| Boundary A | Sneha, 90 | Sneha: Grade A | Sneha: Grade A | Pass |
| Boundary B | Arjun, 89 | Arjun: Grade B | Arjun: Grade B | Pass |
| Mobile layout | 375 px screen | Fits screen | Fits screen | Pass |

The screenshots below are the reviewer's evidence for each test.

**Screenshot 3: Priya, 95 gives Grade A.**

&#91;image: Priya, 95 gives Grade A\]

**Screenshot 4: Rahul, 60 gives Grade C.**

&#91;image: Rahul, 60 gives Grade C\]

**Screenshot 5: Amit, 120 shows the marks error.** The app shows a friendly message instead of a wrong grade.

&#91;image: Amit, 120 shows the marks error\]

**Screenshot 6: an empty name shows the name error.**

&#91;image: Empty name shows the name error\]

**Screenshot 7: Sneha, 90 gives Grade A.** Exactly 90 is the lowest A, a classic boundary case.

&#91;image: Sneha, 90 gives Grade A\]

**Screenshot 8: Arjun, 89 gives Grade B.** One mark below the boundary drops to B, as the spec requires.

&#91;image: Arjun, 89 gives Grade B\]

## SDD vs vibe coding for this app

The same app built by vibe coding would likely work for normal inputs, but its edge cases would be left to chance.

| Point | Vibe coding | SDD (this case study) |
| --- | --- | --- |
| Starting point | "Make a grade calculator" | Constitution, spec, plan and tasks |
| Grade at exactly 90 | Whatever the AI guesses | Decided in the spec: A |
| Marks of 120 | May show a grade or crash | Friendly error, tested |
| Empty name | Often not handled | Friendly error, tested |
| Mobile support | Only if remembered | Required by the constitution |
| Proof it works | "Looks fine" | Seven tests with screenshots |
| Time to first version | Faster | Slightly slower |

## Lessons learned and conclusion

Writing the spec first made the review fast: every test was known before coding began, and all seven passed.

- **Edge cases are cheaper on paper.** Deciding "90 is an A" in the spec took one line; finding it as a bug later costs far more.
- **Small tasks keep the AI accurate.** One task per prompt gave focused, checkable output.
- **The spec doubles as the test plan.** The "Done when" table became the reviewer's checklist.
- **Human gates still matter.** The AI wrote the code, but people approved the spec, the plan and the final PR.

SDD costs a little time upfront and returns it as predictable, tested software. For anything beyond a throwaway prototype, it is the safer way to build with AI.
