
## Before Step 1: a 2-minute check

1. In `.agents/skills/` you have 10 folders: seven `speckit-*` (constitution, specify, clarify, plan, tasks, analyze, implement) and three `sdd-*` (test, review, release). `speckit-checklist` and `speckit-converge` may also be there. Leave them alone.
2. RULE-7 in `speckit-constitution/SKILL.md` lists your real browsers.
3. You reloaded the window, and there are no "invalid frontmatter" warnings.
4. Typing `/speckit-` in the agent panel shows the commands.
5. Everything is committed. In Source Control (Ctrl+Shift+G), the changes list is empty.

**The approval routine you'll repeat after every step:**

1. Open the file the agent wrote.
2. At the top, change `status: Draft` to `status: Approved`, and fill in `approved_by` and `approved_on`.
3. Save, then commit in Source Control with the message shown for that step.

---

## Step 1: Constitution

**Type** (new conversation, click **+**):

```text
/speckit-constitution
```

**The agent** fills in `.specify/memory/constitution.md` with your 7 rules.

**Check:**

- every rule can be passed or failed with a yes/no answer
- every rule has a check ID (C-01 to C-07)
- no `[PLACEHOLDER]` text remains
- nothing about grades or marks (that's spec content)

**Fix** in the same conversation if needed, for example: `RULE-3 is still vague. Make it measurable.`

**Approve and commit:** `Step 1: constitution approved`

---

## Step 2: Specify

**Type** (new conversation):

```text
/speckit-specify Help a teacher find a student's grade from their marks.
```

**The agent** runs Spec Kit's script (allow it if asked), creates `specs/001-grade-calculator/`, and writes `spec.md` there. It may also switch you to a Git branch called `001-grade-calculator`. That's normal, so stay on it.

**Check:**

- each REQ uses MUST or SHOULD
- the input table covers every listed input, either decided or listed as an open question
- every message is written out in full
- there's a test for each grade and each boundary (100, 90/89, 75/74, 50/49, 0)
- every REQ has a test

**Don't approve yet.** Commit `Step 2: spec draft`, and **keep this conversation open**.

---

## Step 3: Clarify (same conversation as Step 2)

**Type:**

```text
/speckit-clarify
```

**The agent** asks questions with options and a recommended answer.

**Answer briefly**, for example:

```text
Q-1: B.
Q-2: neither. Accept "080" as 80, reject "+80".
```

Keep answering until it says **CLARIFY COMPLETE - ready for product owner approval**.

**Check:**

- the Clarifications table lists every decision
- every decision has a test
- OPEN QUESTIONS is empty
- the "Error:" prefix is decided

**Approve** `spec.md` with version 1.0, then commit `Step 3: spec approved`.

---

## Step 4: Plan

**Type** (new conversation):

```text
/speckit-plan
```

**The agent** writes `plan.md`, and possibly short helper files such as `research.md`, in `specs/001-grade-calculator/`.

**Check:**

- the coverage table lists every REQ
- the compliance table lists every RULE
- marks use a text input with a numeric keyboard
- results use `textContent`
- the logic is separate from the page code
- there are no extra features or libraries

**Approve and commit:** `Step 4: plan approved`

---

## Step 5: Tasks

**Type** (new conversation):

```text
/speckit-tasks
```

**The agent** writes `tasks.md`.

**Check:**

- tasks are named TASK-1, TASK-2… (not T001)
- each task has one result, self-checks that cite T-IDs, a Respects line and a Must-not line
- the dependencies are in a sensible order
- both coverage tables are complete

**Approve and commit:** `Step 5: tasks approved`

---

## Step 6: Analyze

**Type** (new conversation):

```text
/speckit-analyze
```

**The agent** reports in the chat and changes no files.

**Read the last line first.**

- **NOT READY:** for each Critical or High finding, re-run the step that owns it (for example `/speckit-clarify` for a spec gap), re-approve, then run `/speckit-analyze` again in a new conversation.
- **READY TO IMPLEMENT:** create `specs/001-grade-calculator/analysis.md`, paste the report into it, and commit `Step 6: ready to implement`.

---

## Step 7: Implement (one task at a time)

**Type** (new conversation for each task):

```text
/speckit-implement TASK-1
```

**The agent** creates or edits `index.html`, ticks TASK-1 in `tasks.md`, reports what it changed, and stops.

**Self-review:** in the same conversation, send:

```text
Now do the SELF-REVIEW from your instructions for TASK-1.
```

**Test it yourself:**

1. Double-click `index.html` to open it in Chrome, then press F12 and open the **Console**.
2. Try the task's self-check inputs, either on the page or in the console (for example `getGrade(90)` should give `"A"`).
3. Check there are no red errors in the console.

**If it passes,** commit `TASK-1: <title>`. Then start a **new conversation** with `/speckit-implement TASK-2`. Repeat through the last task. TASK-7 creates `tests.js`: paste it into the console, and every line should say PASS.

**If it fails,** tell the agent in the same conversation the input, what you expected and what you saw, and ask it to fix that task only.

---

## Step 8a: Functional tests

**Type** (new conversation):

```text
/sdd-test
```

**The agent** writes a manual test list, `review-tests.js` and `test-report.md`, with every result marked NOT RUN.

**You run the tests:**

1. Open the app and run each test. Screenshot each result into `specs/001-grade-calculator/evidence/` as `T-01.png`, `T-02.png` and so on (Win+Shift+S).
2. Paste all of `review-tests.js` into the console, press Enter, and copy the output.

**Send the real results** in the same conversation:

```text
Here is the REAL output from running the tests:
<paste console output>
T-01: typed Priya, 95 → page showed "Priya: Grade A" (T-01.png)
...
```

**Check** that every test is PASS. If any fail, go back to Step 7 for that task, then redo 8a.

**Commit:** `Step 8a: spec tests passed`

---

## Step 8b: Constitution review

**Type** (new conversation):

```text
/sdd-review
```

**The agent** adds a checklist to `test-report.md`.

**You run each check** and screenshot it as `C-01.png` and so on:

- search `index.html` for `<script src` and `fetch`
- test at 375 px and 320 px
- enter the name `<img src=x onerror=alert(1)>`; it must show as text with no pop-up
- use only Tab and Enter
- use a screen reader (TalkBack on Android, Narrator on Windows)
- test in each supported browser

**Send** `Here are my REAL results:` followed by your results, in the same conversation.

**Check** that every check passes, with no Blocker or Major findings. Then commit `Step 8b: constitution checks passed`.

---

## Step 8c: Release

1. In the terminal, run `git log -1` and copy the commit ID.
2. **Type** (new conversation):

   ```text
   /sdd-release Tested commit: <paste id>. Analysis: READY TO IMPLEMENT.
   ```

3. **The agent** writes `release.md` with APPROVE or REJECT.
   - **REJECT:** fix what it names, then run it again.
   - **APPROVE:** merge and tag the release:

   ```bash
   git checkout main
   git merge 001-grade-calculator
   git tag v1.0.0
   ```

   Use `master` instead of `main` if that's your default branch's name.

4. Publish `index.html` wherever teachers will open it, and commit `Release v1.0.0`.

---

