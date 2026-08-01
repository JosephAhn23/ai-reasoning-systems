# AI Fullstack Knowledge Base

Read this file for end-to-end agent work that spans **how to engineer** and **how to design UI**. It combines the agent craft / system-architecture guide with the frontend design knowledge base.

- **Part A** — System architecture & agent craft (from `system architecture.md`)
- **Part B** — Frontend design judgment (from `frontend_design_knowledge_base.md`)

Sources remain authoritative on their own; this file is the concatenated context pack.

---

# PART A — SYSTEM ARCHITECTURE

---

# DeepSeek v4 Complete Knowledge Base

Read this file before starting any coding task. All guidance is here.

---

# SYSTEM_PROMPT.md

You are an AI software engineer. Your job is to:

1. **Understand the task** — read the request, ask clarifying questions if needed, propose a plan.
2. **Make targeted changes** — edit only what's necessary, not surrounding code.
3. **Verify your work** — test the change, check for side effects, confirm it solves the problem.
4. **Be confident but not cocky** — if you're unsure, say so. If a change is risky, flag it.

## How you think

- **Read before writing.** Always explore the codebase first. Understand the context, find similar patterns, check for dependencies.
- **Plan out loud.** State your approach before editing. This lets the user catch mistakes early.
- **Make small, reviewable changes.** One focused commit per fix. Avoid bundling unrelated work.
- **Test as you go.** After each change, verify it doesn't break anything.

## What you are NOT

- A rubber stamp. You don't blindly accept a request; you think about whether it's a good idea.
- A copy-paste machine. You don't duplicate code. You find patterns and refactor.
- A code-golf player. You write clear, boring code that's easy to maintain.
- A documentation bot. You don't write comments for obvious code. You fix the code so it doesn't need comments.

## When you're stuck

1. Describe what you expected vs. what you see.
2. Ask the user a specific question (not "what should I do?" but "should the timeout be 5s or 30s?").
3. Propose a concrete next step.

Never silently give up or hand-wave a solution.


---

# PLANNING.md

How to break down a task and execute it step by step.

## The Three-Step Plan

### 1. Understand (5–10 min)
- **What are we changing?** (feature, bug fix, refactor)
- **What should it do after the change?** (expected behavior)
- **What files need to change?** (estimate)
- **What could go wrong?** (side effects, breaking changes)

**Output:** One paragraph explaining the task and approach.

Example:
> "Fix: API client timeout is 30s, but requests often hang. Change to 5s, which matches the server timeout. Files: src/api/client.ts (timeout config), tests/api/client.test.ts (update mock). Risk: any request that takes 5–30s will fail — check logs after deploy."

### 2. Implement (15–45 min)
Execute in order:
1. Read the relevant files (don't edit yet).
2. Make the smallest change that fixes the issue.
3. Test it (run existing tests, or manual check).
4. Commit.

**Go narrow first.** Make it work, then make it better if the user asks.

### 3. Verify (5–10 min)
- Tests pass?
- Logs/output show expected behavior?
- No new errors introduced?
- Can you explain the change in one sentence?

If all yes, you're done. If no, go back to implement.

## When to Decompose Into Sub-Tasks

**Do decompose** if the task is:
- A feature with multiple steps (e.g., add auth → update UI → write tests).
- A bug with multiple files (e.g., frontend form → backend validation → database schema).
- Any task that would take >1 hour of continuous work.

**Example decomposition:**
```
Task: Add user roles to API
  1. Update User type (types/user.ts)
  2. Add role field to database schema (migrations)
  3. Update API endpoints to return role (api/users.ts)
  4. Update frontend to display role (components/UserCard.tsx)
  5. Write tests for each layer
```

Execute in order. Commit after each sub-task. Verify at the end.

**Don't decompose** if the task is:
- A single-file fix.
- Renaming or moving something.
- A documentation update.

Just do it.

## Red Flags (Stop and Ask)

- **"I'm not sure what the user wants."** → Ask for clarification.
- **"This breaks something else."** → Mention it and ask how to proceed.
- **"This is a bigger refactor than a bug fix."** → Propose the refactor as a separate task.
- **"The test doesn't tell me what the expected behavior is."** → Ask the user.

## Estimating Scope

| Scope | Time | Signs |
|---|---|---|
| Trivial | <5 min | Single line, one file, obvious fix |
| Small | 5–15 min | One file, clear logic, tests exist |
| Medium | 15–45 min | 2–3 files, some thinking required, tests may need updating |
| Large | >45 min | 4+ files, architecture question, major refactor |

**If large:** break into sub-tasks and check with the user before starting.

## Common Failure Modes

| Mistake | Why it happens | How to avoid |
|---|---|---|
| Edit before reading | Rushing | Write the plan first, read the code, *then* edit |
| Change too much at once | "While I'm here..." | One logical change per commit. Note other issues, come back. |
| Forget to test | Assuming it works | Always run tests or manual verification after a change. |
| Break something unintentionally | Didn't understand dependencies | Search for all callers/imports before changing a function. |
| Add unnecessary complexity | Over-engineering | Solve the specific problem. Generalize if asked. |

## Success Criteria

After you're done:
- ✅ The change solves the stated problem.
- ✅ Existing tests still pass.
- ✅ The diff is small and focused.
- ✅ You can explain it in one sentence.
- ✅ No obvious side effects.

If all five are true, you're done.


---

# AGENTS.md

How the agent edits code and takes action.

## Edit Format: Search-Replace Over File Rewrites

**ALWAYS use search-replace (find the exact text, replace it).** Never rewrite an entire file.

Why: Smaller diffs are easier to review, less likely to introduce bugs, and can be undone surgically.

**Good:**
```
Find: const timeout = 30000;
Replace with: const timeout = 5000;
```

**Bad:**
```
Rewrite the entire utils.ts file because one line changed.
```

## Searching for the Text to Replace

The text you search for must be:
- **Exact** (whitespace, capitalization, everything)
- **Unique** (no two places in the file have this exact text)
- **Surrounded by context** (include 1–2 lines before/after to make it unmistakable)

**Good search:**
```
Find:
  function getUser(id) {
    return users[id];
  }

Replace with:
  function getUser(id) {
    if (!id) throw new Error("id required");
    return users[id];
  }
```

**Bad search:**
```
Find: return users[id];
Replace with: if (!id) throw new Error("id required"); return users[id];
```
(Too generic, might match unrelated code.)

## Git Discipline

- **Commit after each change.** One logical change = one commit.
- **Write a clear commit message** (1 line, imperative: "Fix timeout bug in API client").
- **Review the diff before committing.** Does it match what you intended?

## When to Ask vs. When to Act

**Ask the user:**
- Before deleting code ("should I remove this unused function?").
- Before changing specs ("the spec says 30s timeout, should I change it to 5s?").
- Before major refactors ("should I split this file into two?").

**Act without asking:**
- Bug fixes (code is broken, fix it).
- Adding a missing import or variable.
- Simple renames (reword a variable for clarity).
- Formatting/style (make it consistent with the codebase).

## Tool Usage

### Reading Files

When you need to understand code:
1. Start with `ls` or similar to see the structure.
2. Read the main file (probably `index.ts`, `main.py`, `App.jsx`, etc.).
3. Read only the files that are directly relevant.

**Don't:** Read the entire codebase on every task.

### Running Commands

- **Check the current state:** `git status`, `npm test`, `python -m pytest`.
- **Build/run:** Whatever the project's normal build command is.
- **Verify your fix:** Run the same test/command that was failing before.

## Error Recovery

If a command fails:
1. **Read the error.** What does it say?
2. **Check for typos** in your change.
3. **Look at similar code** to see if you missed a pattern.
4. **Revert and try again** if needed.

Never ignore an error and move on.

## What Success Looks Like

- The change is small and focused.
- Tests pass (or you've verified it manually).
- The diff is clear and reviewable.
- You can explain *why* you made the change in one sentence.


---

# ARCHITECTURE.md

How to understand and navigate the codebase without drowning in files.

## The Three-Layer Map

Understand any repo in three passes:

### 1. Structure (5 min)
```
project/
├── src/
│   ├── components/  (UI)
│   ├── services/    (business logic)
│   ├── utils/       (helpers)
│   └── types/       (type definitions)
├── tests/
├── docs/
└── package.json
```

Run `find . -type d -maxdepth 2 | head -20` to see the layout. Understand: what does this project *do*?

### 2. Entry Points (10 min)
Find the main files:
- **Frontend:** `index.html`, `main.jsx`, `App.jsx`
- **Backend:** `server.ts`, `main.py`, `app.go`
- **Tests:** `__tests__/`, `test_*.py`, `*_test.go`

Read the entry point first. It shows you the overall flow.

### 3. The Critical Path (10–15 min)
For the specific task, trace through the relevant files:
- Bug report: trace from user input → bug location → fix point.
- Feature: trace from entry point → all files that need to change.

**Don't** read every file. Follow the trail.

## Context Packing Strategy

You have limited context. Use it wisely.

### Must Read
- The specific file(s) you're editing.
- 1–2 files that call/depend on that file.
- Type definitions or interfaces it uses.

### Should Read
- Tests for the feature/file (understanding what's expected).
- Similar files in the same directory (to match style/patterns).

### Skip
- Unrelated modules.
- Third-party library code (unless the bug is in that integration).
- Build/config files (unless you're touching them).
- Documentation that's not directly about your task.

## Repository Patterns to Look For

### Pattern: Layered Architecture
```
services/ → data/ → models/
```
User request → business logic → database.
Trace all three layers if editing middle tier.

### Pattern: Shared Utilities
All files import from `utils/`. Read `utils/index.ts` to understand what's available.

### Pattern: Config-Driven
App behavior controlled by config files. Check `config.ts` or `.env` before assuming constants are hardcoded.

## When You're Unsure What to Edit

1. **Search for the string that needs to change.** Where does it appear? (1 file or 10?)
2. **Follow imports backward.** What calls this function?
3. **Read the test for the feature.** Tests often show you where the logic lives.

## What Not to Do

- **Don't assume file names.** Grep for the logic, don't guess it's in `utils.js`.
- **Don't edit before reading.** Read the file in context. Understand what else depends on it.
- **Don't refactor while fixing.** If you see bad code, fix the bug first, note it, come back later.

## Anti-Patterns That Slow You Down

| Anti-Pattern | What to do instead |
|---|---|
| "Let me read the whole repo" | Trace the specific path for this task |
| "I'll refactor this while I'm here" | Fix the bug. Note the refactor for later. |
| "I'll add error handling for every edge case" | Handle only what can realistically happen. |
| "I'll make it work with three different input types" | Make it work for the actual input, then generalize if asked. |


---

# TOOL_USAGE.md

How to safely use filesystem operations and git.

## File Operations

### Reading Files
```bash
# Check structure
ls -la src/

# Read a file
cat src/index.ts

# Read with line numbers (easier to reference)
head -50 src/index.ts   # first 50 lines
tail -20 src/index.ts   # last 20 lines

# Find a file by name
find . -name "*.test.ts" | head -10

# Find by content (grep)
grep -r "function getUser" src/
grep -r "getUser" --include="*.ts"  # only TypeScript files
```

### Creating/Modifying Files
**Always use search-replace or append-to-file, not `cat >>` or full rewrites.**

```bash
# BAD: rewriting entire file
cp new.js old.js   # overwrites without checking

# GOOD: search-replace within file (most editors/tools support this)
# Find exactly: const x = 5;
# Replace with: const x = 10;

# GOOD: append to file (if it's a config or logs)
echo "new line" >> config.txt

# GOOD: create a new file if it doesn't exist
echo "content" > new-file.ts  # only if file doesn't exist
```

### Deleting Files
**Always ask the user before deleting anything.**
```bash
# DON'T do this without asking:
rm src/old-file.ts

# DO: mention it first
# "Found unused file src/old-file.ts. Should I delete it?"
```

## Git Workflow

### Before You Start
```bash
git status              # see what's changed
git log --oneline -10   # see recent commits (copy their format)
git diff HEAD           # see all local changes (if any)
```

### While You Work
```bash
# See the current state
git status

# Stage your changes (after reading them)
git diff src/index.ts   # review before staging
git add src/index.ts    # stage one file
git add src/           # stage all changes in a directory

# Commit (with a clear message)
git commit -m "Fix: timeout bug in API client"

# See what you just committed
git log --oneline -1
```

### Commit Message Format
```
Type: Short description (imperative, <50 chars)

Optional longer explanation if needed.
- Bullet point if helpful
- Keep it brief
```

**Types:**
- `Fix:` — bug fix
- `Add:` — new feature
- `Update:` — enhancement to existing feature
- `Refactor:` — code restructuring (no behavior change)
- `Test:` — test-related changes
- `Docs:` — documentation updates
- `Style:` — formatting/style changes

**Good commits:**
```
Fix: API client timeout should be 5s not 30s

Requests that took 5-30s were hanging. Updated timeout
to match server timeout. Updated tests accordingly.
```

```
Add: user role field to API response

/api/users endpoint now includes role in response.
Updated type definitions and tests.
```

**Bad commits:**
```
update stuff          # vague
fixed it              # what did you fix?
refactor everything   # huge change, one message
```

### If You Make a Mistake

```bash
# Undo the last commit (keep changes)
git reset --soft HEAD~1
git status            # see what was committed
# fix, re-add, re-commit

# Undo the last commit (throw away changes)
git reset --hard HEAD~1   # WARNING: loses data

# Undo a file change (before commit)
git checkout src/index.ts

# See what changed in the last commit
git show HEAD
```

## Command Discipline

**Always run these before committing:**

### For a TypeScript/JavaScript project
```bash
npm run build         # or: npm run compile, npm run tsc
npm test              # or: npm run test, jest
```

### For a Python project
```bash
python -m pytest      # or: pytest
python -m black .     # or: your linter
```

### For a Go project
```bash
go build ./...
go test ./...
go vet ./...
```

### For any project
```bash
git status            # clean working directory?
git diff              # review all changes
```

## Anti-Patterns

| Do This | Not This |
|---|---|
| Search-replace specific lines | Rewrite entire files |
| Commit related changes together | Dump 10 unrelated fixes in one commit |
| Test after you commit | Assume it works without testing |
| Use git history to understand changes | Guess what previous code did |
| Ask before deleting | Delete and hope for the best |
| Read a file before editing | Edit blindly |

## Command Reference

```bash
# Status and history
git status                        # what changed?
git log --oneline -10             # recent commits
git diff src/index.ts             # what changed in this file?
git diff HEAD~1                   # what's in the last commit?

# Staging and committing
git add src/                       # stage a directory
git add src/index.ts              # stage one file
git commit -m "message"           # commit with message

# Undoing
git checkout src/index.ts         # undo changes to one file
git reset --soft HEAD~1           # undo last commit, keep changes
git reset --hard HEAD~1           # undo last commit, throw away changes

# Inspecting
git show HEAD                     # see the last commit
git blame src/index.ts            # who changed each line?
```

## Safety Rules

1. **Always `git status` before running destructive commands.**
2. **Always `git diff` before committing to see what you're committing.**
3. **Always ask the user before deleting files.**
4. **Never use `--force`, `--hard`, or other destructive flags without being explicit.**
5. **If something goes wrong, ask the user what to do.**


---

# TESTING.md

How to verify your changes work before declaring them done.

## The Testing Ladder

Go through these in order. Stop at the first one that's meaningful for your change.

### 1. Syntax/Build Check (2 min)
```bash
# Does the code compile/parse?
npm run build
# or: python -m py_compile src/file.py
# or: go build ./...
```

If this fails, you have a typo. Fix it. Don't move on.

### 2. Unit Tests (5 min)
```bash
# Run tests for the file you changed
npm test -- src/api/client.test.ts
# or: pytest tests/api/test_client.py -v
```

If existing tests fail, you broke something. Debug it.

### 3. Integration Tests (5–10 min)
```bash
# Run tests for the module/feature
npm test -- src/api/
```

Tests pass? Good. Tests fail? Either you need to update them (if behavior changed) or you broke something (fix it).

### 4. Full Test Suite (10–30 min)
```bash
npm test
# or: pytest
# or: go test ./...
```

This catches unintended side effects in other parts of the codebase.

### 5. Manual Verification (10–30 min)
If there's no test, run it manually:

**Backend/API:**
```bash
npm run dev          # or: python -m flask run
curl http://localhost:3000/api/users
# Does it return what you expect?
```

**Frontend:**
```bash
npm run dev          # or: npm start
# Open browser, navigate to the feature
# Does it work?
```

## When to Write a Test

**Always write a test if:**
- You're fixing a bug (add a test that would fail without the fix).
- You're adding a feature (add tests for the happy path and edge cases).

**Don't write a test if:**
- The code is too simple (one-liner renames).
- It's already covered by existing tests.
- The test would be a mock of a mock (testing internal details instead of behavior).

## Test Structure (Quick Reference)

### Unit Test Example (JavaScript/Jest)
```javascript
describe('getUser', () => {
  it('should return user by id', () => {
    const user = getUser(1);
    expect(user.id).toBe(1);
  });

  it('should throw if id is invalid', () => {
    expect(() => getUser(-1)).toThrow();
  });
});
```

### Test a Bug Fix
```javascript
// BAD: doesn't verify the bug is fixed
it('getUser works', () => {
  const user = getUser(1);
  expect(user).toBeDefined();
});

// GOOD: tests the specific bug that was fixed
it('should throw on negative id', () => {
  // This test would FAIL before the fix.
  expect(() => getUser(-1)).toThrow('id must be positive');
});
```

## Manual Testing Checklist

Before declaring a change done:

### Happy Path
- [ ] Does the main feature work as expected?
- [ ] Does output match the spec?

### Edge Cases
- [ ] Empty input?
- [ ] Null/undefined?
- [ ] Boundary values (0, -1, max int)?
- [ ] Large input (1M items)?

### Error Cases
- [ ] What happens if something fails? (network error, missing file, invalid data)
- [ ] Does the error message help the user?

### Side Effects
- [ ] Does this change affect other features?
- [ ] Are there any warnings/errors in the logs?

## Red Flags That Mean "Stop and Debug"

| Sign | What it means |
|---|---|
| One test fails consistently | You broke something in that module. Fix it. |
| Tests pass locally but you're unsure | Run them again, or run the full suite. |
| You don't have a test for the change | Add one, or explain why it's not testable. |
| You have to restart the app to see the change | You may have a caching issue or missed a reload. |
| Manual test succeeds but automated test fails | The test is wrong, or your manual test missed something. Fix the test. |

## Test-Driven Debugging

If something breaks:

1. **Reproduce the failure** (what input causes it?).
2. **Write a test that captures the failure** (test should fail).
3. **Fix the code** (now the test should pass).
4. **Verify the fix** (run the test again, run related tests).

This is much faster than random debugging.

## When Tests Don't Exist

If the code isn't tested:

1. **Create a minimal test** for the thing you're changing.
2. **Run it** (it might fail at first, that's okay).
3. **Fix the code** to make it pass.
4. **Move on.**

Don't try to backfill 100 tests for untested legacy code. Just test what you touch.

## Success Criteria for Testing

- ✅ All existing tests pass.
- ✅ Any new behavior has a test.
- ✅ Manual verification works (you ran it, you saw it succeed).
- ✅ No new warnings or errors in logs.
- ✅ You can run the test in 1 minute or less.

If all five are true, you're done testing.


---

# CODE_REVIEW.md

What to check before you declare a task "done".

## The Pre-Commit Checklist

Before making a commit, run through this:

### Correctness
- [ ] Does the code do what I said it would?
- [ ] Does it handle the happy path?
- [ ] Did I miss any edge cases the tests catch?
- [ ] Are imports correct (no undefined variables)?

### Safety
- [ ] Did I introduce any unintended side effects?
- [ ] Could this break existing tests?
- [ ] Are there any obvious security issues (SQL injection, XSS, command injection)?
- [ ] Did I accidentally commit sensitive data (passwords, keys)?

### Clarity
- [ ] Is the code obvious? (if not, why?)
- [ ] Would a teammate understand this change?
- [ ] Does it match the style of the rest of the file?

### Completeness
- [ ] Did I update tests if I changed behavior?
- [ ] Did I update docs/comments if the API changed?
- [ ] Are there TODOs or debug code left behind?

## The Post-Commit Checklist

After you commit and (if applicable) run tests:

### Tests
- [ ] Do existing tests pass?
- [ ] Did I add/update tests for new behavior?
- [ ] Are there any tests that *should* fail but don't (false negatives)?

### Integration
- [ ] Does this work with the rest of the system?
- [ ] Are there any config changes needed?
- [ ] Did I miss a database migration or environment variable?

### Performance
- [ ] Did I introduce any new loops or expensive operations?
- [ ] Could this cause a memory leak or hang?

## Common Mistakes to Catch Before the User Sees Them

| Mistake | How to catch it |
|---|---|
| Syntax error | Run `npm run build` or equivalent |
| Import wrong | Check all imports are defined |
| Break existing test | Run full test suite, not just new tests |
| Typo in variable name | Read the diff out loud or search for the word |
| Off-by-one error | Trace through with an example |
| Forgot to commit something | `git status` should be clean |
| Left debug code | Search for `console.log`, `print`, `TODO` |
| Broke a sibling | Search for all callers of what you changed |

## The Diff Review

Before committing, read the actual diff:
- **Are lines added that should be deleted?** (cleanup, debug code)
- **Are lines deleted that should stay?** (accidental removal)
- **Is whitespace correct?** (no random indentation changes)
- **Are there merge conflicts you missed?** (diff should be clean)

## When to Punt Back to the User

Don't force a fix if:
- **You're not sure what the desired behavior is.** → Ask for clarification.
- **The fix has multiple valid approaches.** → Propose one, ask which they prefer.
- **The fix touches multiple systems and you can't test it end-to-end.** → Describe what you did and what they should test.
- **It breaks something else.** → Mention what breaks and ask how to proceed.

## Red Flag Patterns

If you see these, stop and review:

| Pattern | What it might mean |
|---|---|
| Massive diff | You changed too much. Undo, split into smaller commits. |
| File rewritten | You should've used search-replace on specific sections. Redo. |
| Copy-paste code | You found a pattern to extract. Note it for later. |
| Commented-out code | Delete it or commit it, don't leave it hanging. |
| Comments explaining obvious code | Delete the comment, rewrite the code so it's obvious. |

## Definition of Done

Your task is done when:
- ✅ You understand the original problem.
- ✅ Your change solves that problem.
- ✅ Tests pass (or you've manually verified it works).
- ✅ The diff is small and focused.
- ✅ You've checked for side effects.
- ✅ You can explain the change in one sentence.

If all six are true, you can commit and move on.


---

# STYLE_GUIDE.md

Coding conventions and anti-patterns to avoid.

## What NOT to Do

### Don't Add Comments to Obvious Code
```javascript
// BAD
// Get the user ID
const userId = user.id;

// Get all posts
const posts = user.posts.map(p => p);
```

If the code is obvious, delete the comment. If it needs a comment, rewrite the code.

**Good code is self-documenting:**
```javascript
const userId = user.id;
const userPosts = user.posts;
```

### Don't Write Comments Explaining Internal Details
```javascript
// BAD: This just narrates what the code does
// Check if the id is greater than 0
// If it is, return true
function isValidId(id) {
  return id > 0;
}

// GOOD: Code is clear, no comment needed
function isValidId(id) {
  return id > 0;
}
```

### Only Comment the WHY, Not the WHAT
```javascript
// GOOD: Comment explains the non-obvious constraint
// IDs must be positive because they're database PKs.
// Negative values indicate a lookup failure.
function isValidId(id) {
  return id > 0;
}

// BAD: Comment just repeats the code
// Returns true if id is positive
function isValidId(id) {
  return id > 0;
}
```

### Don't Over-Engineer
```javascript
// BAD: Abstract before you need it
class BaseEntity {
  getId() { return this.id; }
  setId(id) { this.id = id; }
}
class User extends BaseEntity { }
class Post extends BaseEntity { }

// GOOD: Just use objects/classes directly
class User {
  constructor(id) { this.id = id; }
}
```

Three similar lines are fine. Only abstract when you see the third use.

### Don't Add Error Handling for "Just in Case"
```javascript
// BAD: Handle errors that can't happen
function getValue(obj) {
  try {
    // obj is always defined when this is called
    return obj.value;
  } catch (e) {
    return null;
  }
}

// GOOD: Only handle realistic errors
function getValue(obj) {
  return obj.value;  // If obj is undefined, that's a caller bug
}
```

### Don't Add TODOs or Leave Debug Code
```javascript
// BAD: Leaves junk behind
function processUser(user) {
  console.log("DEBUG: user =", user);  // Remove this!
  // TODO: optimize this later
  return user.id * 2;
}

// GOOD: Clean code, ready to ship
function processUser(user) {
  return user.id * 2;
}
```

If you think something should be optimized, create a separate task. Don't leave it in the code.

## What TO Do

### Use Clear Names
```javascript
// BAD
const x = users.filter(u => u.a > 5);

// GOOD
const activeUsers = users.filter(u => u.age > 5);
```

### Use Immutability Where It Matters
```javascript
// GOOD: Don't modify the original array
const filtered = users.filter(u => u.active);

// OKAY: Local mutation is fine
let count = 0;
for (const user of users) {
  count += user.posts.length;
}
```

### Keep Functions Small and Focused
```javascript
// BAD: Does too much
function processUser(user) {
  validate(user);
  saveToDb(user);
  sendEmail(user);
  updateCache(user);
  logEvent(user);
}

// GOOD: One job per function
function saveUser(user) {
  validate(user);
  saveToDb(user);
  return user;
}
```

### Prefer Explicit Over Implicit
```javascript
// BAD: Magic number
const timeout = x > 0 ? 30000 : 5000;

// GOOD: Clear intent
const TIMEOUT_MS = {
  withContext: 30000,
  minimal: 5000,
};
const timeout = hasContext ? TIMEOUT_MS.withContext : TIMEOUT_MS.minimal;
```

### Test Edge Cases
```javascript
// GOOD test: covers happy path and edges
it('should validate id', () => {
  expect(isValidId(1)).toBe(true);      // happy path
  expect(isValidId(0)).toBe(false);     // boundary
  expect(isValidId(-1)).toBe(false);    // invalid
});
```

## Consistency Rules

Follow the **existing style in the file**, not your personal preference.

- If the codebase uses 2-space indents, use 2-space indents.
- If variables are `camelCase`, use `camelCase` (not `snake_case`).
- If the team uses semicolons, use semicolons.

When in doubt, run the formatter (Prettier, Black, gofmt) and match what it produces.

## Language-Specific Patterns

### TypeScript/JavaScript
- Use types when available, not as an afterthought.
- Prefer `const` over `let` over `var`.
- Use arrow functions for callbacks, regular functions for exports.

### Python
- Use type hints: `def get_user(id: int) -> User:`.
- Use f-strings over `.format()`.
- Follow PEP 8 or run Black.

### Go
- Export functions are `PascalCase`, private are `camelCase`.
- Always handle errors, don't ignore them.
- Use interfaces for dependencies, not concrete types.

## Common Smells

| Smell | What it means |
|---|---|
| Function has 10+ parameters | Split it up or pass an object |
| Function does three different things | Extract into separate functions |
| Variable named `x`, `tmp`, `data` | Rename to be specific |
| Deeply nested conditionals (4+ levels) | Extract functions or use early returns |
| Copy-paste code in two places | Extract into a function |
| Comment explaining a hack | Fix the hack or mark it with `HACK:` for later |

## One-Sentence Rules

1. **Code is read 10x more than it's written.** Make it obvious.
2. **Delete code before adding code.** Less code = fewer bugs.
3. **Solve the problem, don't solve every problem.** Generalize if asked.
4. **Tests catch bugs, not comments.** Good tests > good comments.
5. **Use boring code.** Clever code is hard to maintain.


---



---

# PART B — FRONTEND DESIGN

---

# Frontend Design Knowledge Base

Read this file before building UI, dashboards, landing pages, or any user-facing React/web interface. All guidance is here.

Frontend is one of the biggest weaknesses of cheaper models. They often:

* make everything look similar,
* have weak visual hierarchy,
* poor spacing,
* mediocre UX,
* generic dashboards,
* inconsistent component composition.

This knowledge base is separate from backend/systems engineering. It teaches **visual judgment and interface composition**, not only component APIs. Screenshots and real product references often teach more than raw React code — a design system is taste under constraints.

**How to use this file:** prefer principles + patterns over copy-paste. Study the cited products and libraries for spacing, hierarchy, typography, and interaction — do not clone their brand.

---

# SHADCN_UI.md

Copy-paste components you own — the modern default for serious React UI.

## The Core Identity

shadcn/ui is not an npm component monolith. You **generate/copy source into your repo** (built on Radix primitives + Tailwind), so you own the code, the tokens, and the variants. That invert of "dependency you can't surgically edit" is the point.

```
  Radix primitive (a11y + behavior)
       +
  Tailwind tokens / variants
       +
  your repo's components/ui/*
```

## What to Steal

- Composition patterns: `Button` variants, `Dialog` structure, form field wiring.
- Consistent reliance on **accessible primitives** underneath styled shells.
- A single source of design tokens (CSS variables) driving light/dark.

## What to Avoid

- Dropping in every block until the app looks like the shadcn docs site.
- Treating the registry as a brand — it's a starting kit; differentiate.

## What to Carry Away

1. **Own your components** when you need design control; wrap primitives, don't only theme from outside.
2. **Behavior and appearance separate** — Radix for behavior, your tokens for look.
3. **Variants as a system** (CVA-style) beat one-off class strings.
4. **Fewer well-tuned components** beat a kitchen-sink UI kit.
5. **Docs sites are not your product** — restyle aggressively.

---

# MAGIC_UI.md

Motion-forward marketing components — use sparingly and with intent.

## The Core Identity

Magic UI (and similar registries) ship polished, animated building blocks for landings and feature sections. They teach **presence**: staggered reveals, text effects, beam/border accents, marquee social proof.

## When They Help

- Hero moments, feature showcases, launch pages where delight is the job.
- Learning timing curves and staggered children from working examples.

## When They Hurt

- App chrome (settings, tables, forms) with constant motion noise.
- Stacking three showpiece effects until hierarchy collapses.

## What to Carry Away

1. **Motion is hierarchy** — animate the one thing that matters.
2. **Marketing ≠ product UI** — borrow craft, not constant spectacle.
3. **Performance budgets** — watch CLS, main-thread cost, reduced-motion.
4. **One signature effect** beats five generic ones.
5. **Study structure under the effect** — layout still does the heavy lifting.

---

# ACETERNITY_UI.md

High-craft React UI patterns for distinctive marketing surfaces.

## The Core Identity

Aceternity UI popularized a wave of distinctive, Tailwind + Framer Motion compositions (spotlight cards, bento grids, tracing beams). The lesson is **composition originality**: backgrounds, masks, and micro-layout tricks that escape the purple-gradient AI default.

## What to Carry Away

1. **Backgrounds and masks create atmosphere** without extra cards.
2. **Bento layouts** organize features without a wall of identical cards.
3. **Restraint** — one Aceternity-grade section per page is often enough.
4. **Steal geometry, not identity** — re-token colors/type to your brand.
5. **Always check mobile** — fancy absolute layouts break first on small screens.

---

# MOTION.md

Tasteful animation with Motion (formerly Framer Motion).

## The Core Identity

Motion gives React a declarative animation model: `animate`, layout transitions, `AnimatePresence` for enter/exit, gestures, and scroll-linked stories. Models fail at animation by either skipping it or animating everything.

```
  presence  -->  AnimatePresence
  layout    -->  layout / layoutId
  feedback  -->  whileHover / whileTap
  story     -->  staggered children / scroll
```

## Principles

- **Duration short** (150–300ms UI chrome; longer only for narrative).
- **Easing with intent** — enter slightly decelerate; exits can be quicker.
- **Honor `prefers-reduced-motion`.**
- **Layout animations** for reordering; don't fake with opacity alone when position changes.

## What to Carry Away

1. **Animate state change, not decoration.**
2. **Exit animations need presence management** — unmount timing matters.
3. **Shared layoutIds** create continuity across routes/modals.
4. **Stagger sparingly** — lists of 3–7, not 40.
5. **Reduced motion is a feature**, not an afterthought.

---

# REFACTORING_UI.md

Design principles for developers (Refactoring UI concepts).

## The Core Identity

Refactoring UI teaches non-designers to make interfaces look deliberate: hierarchy, spacing scales, limited type sizes, empty space, and avoiding "gray on gray on gray." You don't need the book as a dependency — you need the **checklist**.

## High-Leverage Rules

- Start with too much white space; tighten until it feels aligned — not the reverse.
- Establish a **type scale** (e.g. 3–5 sizes) and stop inventing font sizes.
- Establish a **spacing scale** (4/8-based) and stop using magic numbers.
- One accent color for actions; neutrals do the rest.
- De-emphasize secondary text (size, weight, color) instead of shouting with red/blue everywhere.
- Borders and shadows are hierarchy tools — if everything is elevated, nothing is.

## What to Carry Away

1. **Hierarchy first** — if you squint, can you still see primary vs secondary?
2. **Scales beat vibes** — type and space on a system.
3. **Neutrals carry UI** — accent is for action and status.
4. **Empty space is structure**, not waste.
5. **Finish by removing**, not adding.

---

# RADIX_UI.md

Accessible primitives without imposing visuals.

## The Core Identity

Radix UI Primitives provide **unstyled, accessible** components: dialogs, menus, popovers, tabs, toast, etc. Focus traps, keyboard roving tabindex, aria wiring — solved once.

## What to Carry Away

1. **Don't hand-roll modal focus** — use primitives.
2. **Styling belongs to you** — primitives are behavior.
3. **Composition API** (`Root`/`Trigger`/`Content`) scales better than mega-props.
4. **Portal + positioning** are part of the primitive's job.
5. **Accessibility is the floor** for any design system.

---

# TREMOR.md

Dashboard-oriented React components on Tailwind.

## The Core Identity

Tremor focuses on **metrics UI**: KPI cards, charts, trackers, input blocks tuned for admin/analytics. It teaches density, numeric typography, and chart+legend composition for data products.

## What to Carry Away

1. **Dashboards need a data hierarchy** — KPI → trend → table, not 12 equal cards.
2. **Number formatting is UX** — units, deltas, sparklines.
3. **Chart defaults matter** — axes, grids, colorblind-safe series.
4. **Density ≠ clutter** if alignment and grouping are clean.
5. **Don't use marketing hero patterns inside app shells.**

---

# REACT_BITS.md

Small animated building blocks for React.

## The Core Identity

React Bits-style libraries showcase isolated motion/visual patterns (text animators, backgrounds, cursors). Use as a **catalog of micro-techniques**, then reimplement in your tokens.

## What to Carry Away

1. **Isolate the idea** — re-skin before shipping.
2. **Microinteractions teach timing** better than theory alone.
3. **Avoid novelty traps** in productivity UI.
4. **Compose with your design system**, don't parallel it.
5. **Delete anything that doesn't support a user goal.**

---

# TAXONOMY.md

Next.js dashboard taxonomy / reference app structure.

## The Core Identity

Taxonomy-style reference projects (shadcn-era Next.js examples) teach **app information architecture**: marketing pages + auth + dashboard + settings + content sections with consistent nav, tables, and forms.

## What to Carry Away

1. **IA before chrome** — nav groups match mental models.
2. **Settings patterns** — sections, danger zones, save affordances.
3. **Table + filter + empty state** as a triple, not an afterthought.
4. **Auth-gated shells** differ from marketing layouts — separate roots.
5. **Reference apps are curricula** — copy structure, not aesthetics blindly.

---

# REACT_ARIA.md

Behavior-first accessibility from Adobe React Aria / Aria Components.

## The Core Identity

React Aria separates **hooks/behavior** from rendering more aggressively than many kits. It encodes keyboard interactions, i18n-aware formatting, and advanced patterns (collections, date pickers, grids).

## What to Carry Away

1. **Internationalization is part of UI behavior** (focus, selection, formatting).
2. **Headless hooks** let design systems stay branded.
3. **Complex widgets are research problems** — don't invent date picker a11y.
4. **Collections virtualization** patterns matter at scale.
5. **Pair with your visuals** — Aria is not a look.

---

# DESIGN_SYSTEMS.md

How professionals build reusable interface systems.

## Sources to Study

1. shadcn/ui
2. Magic UI
3. Aceternity UI
4. Origin UI
5. HeroUI (formerly NextUI)
6. Radix UI / React Aria / MUI / Chakra / Mantine (broader ecosystem)

## System Layers

```
  tokens (color, space, type, radius, shadow)
    -->
  primitives (button, input, checkbox)
    -->
  patterns (form field, page header, data table)
    -->
  features (billing settings, onboarding step)
```

Rules: tokens never hardcode in features; patterns own spacing rhythm; features compose patterns only.

## What to Carry Away

1. **Tokens first** — without them, "consistency" is wishful.
2. **Patterns > orphan components.**
3. **Document variants and states** (hover/focus/disabled/loading/error).
4. **One radius / shadow language** across the product.
5. **Resist new one-off components** — extend the system.

---

# TYPOGRAPHY.md

Type is hierarchy.

## Rules

- Prefer expressive, purposeful fonts; avoid defaulting to Inter/Roboto/Arial/system for branded surfaces (product apps may stay neutral — be intentional).
- Limit weights (regular/medium/semibold) and sizes (a small scale).
- Line length ~45–75 characters for reading text.
- Pair display + body carefully; don't use display fonts for dense data.
- Tabular nums for dashboards.

## What to Carry Away

1. **Size + weight + color** encode hierarchy together.
2. **Fewer sizes than you think.**
3. **Body readability beats brand hero on app screens.**
4. **Match type to surface** — marketing vs app vs code.
5. **Never randomize letter-spacing** without a system.

---

# SPACING.md

Spacing is structure.

## Rules

- Use a spacing scale (4px foundation is common).
- Consistent padding inside components; consistent gaps in stacks.
- Related items closer; unrelated groups farther (proximity principle).
- Align to a mental grid; avoid 13px / 27px orphans.
- On mobile, increase tap targets before shrinking all gaps blindly.

## What to Carry Away

1. **If it looks "off," check spacing before color.**
2. **Stacks and clusters** beat ad-hoc margins.
3. **Section padding** creates page rhythm.
4. **Dense UI still needs grouping.**
5. **Don't nest cards inside cards to fake structure** — use space.

---

# ANIMATIONS.md

Tasteful motion sources: Motion, React Bits, Animate UI, Motion Primitives, GSAP examples.

## Rules

- Motion supports meaning: feedback, continuity, attention.
- Prefer transform/opacity; avoid layout thrash.
- Interruptible animations; never block input without need.
- GSAP shines for timeline-heavy marketing; Motion shines for React app state.

## What to Carry Away

1. **One job per animation.**
2. **Fast feedback, slower storytelling.**
3. **Shared element transitions** glue navigations.
4. **Test reduced motion.**
5. **Delete motion that doesn't teach or confirm.**

---

# DASHBOARDS.md

Study: Tremor, Taxonomy, Refine, Cal.com, Supabase Dashboard.

## Composition Pattern

```
  Page header (title + primary action)
  KPI row (3–5 max)
  Main chart / calendar / feed
  Secondary table or lists
```

Avoid: six identical KPI cards, charts without context, rainbow categorical colors, every widget in a heavy card chrome.

## What to Carry Away

1. **One primary question per screen.**
2. **KPIs need comparison** (vs yesterday / target).
3. **Tables need empty/loading/error states.**
4. **Filters are part of the design**, not bolted on.
5. **App shell consistency** (nav width, header height) builds trust.

---

# LANDING_PAGES.md

Study composition: Vercel, Linear, Raycast, Stripe, Resend.

## Hero Budget

First viewport usually:

* brand
* one headline
* one short supporting sentence
* one CTA group
* one dominant visual plane

Avoid: stat strips, logo salads, multiple competing CTAs, inset collage heroes, floating badge clutter on the hero image.

## What to Carry Away

1. **One composition, not a dashboard.**
2. **Brand must survive removing the nav.**
3. **Full-bleed visual plane** beats tiled cards in the hero.
4. **Sections have one job each.**
5. **Ship 2–3 intentional motions**, not noise.

---

# MOBILE_UI.md

Study: Mobbin (patterns), platform HIG/Material, responsive landings.

## Rules

- Thumb zones and 44px+ targets.
- Bottom nav vs hamburger — know the job.
- Don't shrink a desktop table; reflow to cards/lists.
- Sticky CTAs carefully — don't cover content.
- Respect safe areas.

## What to Carry Away

1. **Design mobile as its own composition.**
2. **Prioritize tasks**, not pixel parity.
3. **Gesture conflicts** (swipe vs scroll) need care.
4. **Type scales up slightly** for reading on phones.
5. **Test on a real device width**, not only DevTools.

---

# ACCESSIBILITY.md

Sources: Apple HIG, Material Design 3, Radix/React Aria, WAI-ARIA patterns.

## Floor Requirements

- Keyboard path for all actions.
- Visible focus.
- Sufficient contrast.
- Labels for inputs; don't rely on placeholder alone.
- Respect reduced motion; don't convey info by color alone.

## What to Carry Away

1. **A11y is design quality**, not compliance theater.
2. **Primitives carry you far** — use them.
3. **Announce dynamic changes** (live regions) when needed.
4. **Test with keyboard only** before shipping.
5. **Accessible ≠ ugly** — constraints sharpen design.

---

# FORMS.md

Sources: React Hook Form, TanStack Form, Zod, Conform, Vest.

## Pattern

```
  schema (Zod) --> form state --> field UI --> async submit --> server errors
```

- Labels, helper text, inline errors aligned.
- Don't block typing with premature errors; validate on blur/submit thoughtfully.
- Preserve input on failure; map server errors to fields.
- Dangerous actions need confirmation patterns.

## What to Carry Away

1. **Schema as source of truth.**
2. **Field layout is a pattern** — label/control/error alignment.
3. **Disable submit sparingly** — explain why when you do.
4. **Optimistic UI only when rollback is safe.**
5. **Multi-step forms show progress and allow back.**

---

# CHARTS.md

Sources: Tremor, Recharts, Nivo, ECharts, Observable.

## Rules

- Start with the question; pick chart type second.
- Label axes; include units; prefer direct labels over huge legends when possible.
- Avoid 3D and chartjunk.
- Use OKLCH/HSL-tuned palettes; test colorblind safety.
- Empty and loading states for charts, not blank boxes.

## What to Carry Away

1. **Clarity over decoration.**
2. **Time series vs categorical** need different defaults.
3. **Tooltip + annotation** explain spikes.
4. **Observable notebooks** teach analytical craft.
5. **Fewer series** — split charts before spaghetti.

---

# NAVIGATION.md

## Rules

- Information architecture before component choice.
- Active states obvious; groups labeled.
- Don't mix 5 nav paradigms on one shell.
- Breadcrumbs for depth; tabs for peer sections.
- Command palette (Linear/Raycast-like) for power users — optional enhancer, not the only path.

## What to Carry Away

1. **Nav is product strategy made visible.**
2. **Deep-link everything meaningful.**
3. **Mobile nav is a redesign**, not a collapse.
4. **Settings belong in a predictable place.**
5. **Escape hatches** (search/command) reduce nav bloat.

---

# DARK_MODE.md

## Rules

- Design with tokens (`background`, `foreground`, `muted`, `border`, `accent`) — not inverted hex hacks.
- Dark mode is not "pure black + neon" by default; control contrast.
- Elevation via luminance steps more than heavy shadows.
- Images/illustrations need dark-aware variants.
- Honor system preference with manual override.

## What to Carry Away

1. **Tokens make dark mode possible.**
2. **Test charts and statuses** on both themes.
3. **Borders + spacing** replace shadows carefully.
4. **Brand accent may need retuning** for dark surfaces.
5. **Avoid pure #000 unless intentional.**

---

# GLASSMORPHISM.md

## Rules

- Glass is atmosphere, not a layout strategy.
- Need real backdrop content; over blur tanks performance/readability.
- Keep text contrast legal; frosted panes fail often.
- Use on heroes/modals sparingly; avoid glass tables of data.

## What to Carry Away

1. **One glass surface per view** is usually enough.
2. **Fallback solids** for reduced performance/a11y.
3. **Borders + soft highlight** sell the material more than max blur.
4. **Don't put dense forms on glass.**
5. **If hierarchy weakens, remove glass.**

---

# GRADIENTS.md

## Rules

- Gradients for atmosphere and brand — not for body text.
- Prefer subtle meshes / two-stop soft gradients over rainbow streaks.
- Avoid the cliché purple-to-indigo AI look unless it *is* the brand.
- CSS variables for gradient stops; keep them themeable.
- Pair with photography or real product UI for landings when possible — abstract gradient alone is a weak visual anchor.

## What to Carry Away

1. **Restraint reads premium.**
2. **Gradients support, they don't replace content.**
3. **Check banding and contrast on text overlays.**
4. **Animate gradients slowly or not at all.**
5. **Match landing gradient language to product chrome.**

---

# COMPONENT_PATTERNS.md

## High-Value Patterns

- Page header: title, description, actions
- Empty state: explanation + CTA
- Data table: filters, sort, row actions, bulk actions
- Confirm destructive dialog
- Toast / inline alert for feedback
- Skeleton loaders that match layout

## Composition Rule

If removing a border/shadow/radius doesn't hurt interaction or understanding, it shouldn't be a card.

## What to Carry Away

1. **Patterns beat pages made of freeform divs.**
2. **States are the work** (empty/loading/error/success).
3. **Cards are for interaction containers**, not decoration.
4. **Keep action placement predictable.**
5. **Document pattern usage in the design system.**

---

# COLOR_THEORY.md

Sources: Tailwind Colors, Radix Colors, Open Color, Material Color, OKLCH examples.

## Rules

- Neutrals first; semantic colors for success/warn/danger/info.
- OKLCH helps even perceptual lightness across hues.
- Radix-style scales (steps 1–12) teach hover/active/solid/contrast roles.
- Don't use color alone for state.
- Limit accent hues.

## What to Carry Away

1. **Scales > loose hex picks.**
2. **Semantic tokens** (`danger`) over raw `red-500` in features.
3. **Test contrast** early.
4. **Charts need a separate categorical palette.**
5. **Brand color ≠ action color** always — define roles.

---

# VISUAL_HIERARCHY.md

## Squint Test

Blur your eyes: you should still see primary action / headline / clustering.

## Tools

- Size, weight, contrast, proximity, alignment, whitespace
- Not: more borders, more colors, more badges

## What to Carry Away

1. **One focal point per section.**
2. **Secondary content recedes.**
3. **Alignment creates calm.**
4. **Badges/chips inflate noise** — budget them.
5. **Hierarchy is design debt when everything is bold.**

---

# MICROINTERACTIONS.md

## Examples

- Button press depth / color shift
- Successful save check
- Copy-to-clipboard confirmation
- Pull-to-refresh affordance
- Subtle list reorder

## Rules

- Confirm user actions that otherwise feel uncertain.
- Keep under ~200ms for tool-like feedback.
- Don't animate decorative chrome on every hover in dense UIs.

## What to Carry Away

1. **Feedback > flair.**
2. **Same interaction, same response** across the app.
3. **Optimistic only with clear failure recovery.**
4. **Sound/haptics rare on web** — visual is enough.
5. **Micro ≠ micro-clutter.**

---

# CREATIVITY_SOURCES.md

Scrape/study (do not copy wholesale):

* Awwwards
* Godly
* Land-book
* Mobbin
* Lapa Ninja

Learn: spacing, hierarchy, typography, composition, layouts, interaction patterns.

## Screenshot-First Learning

For frontend, **screenshots may be more valuable than code**. Curate examples of dashboards, forms, pricing pages, onboarding, mobile screens, empty states — pair each with a one-paragraph "why it works." That trains judgment raw React often doesn't capture.

## What to Carry Away

1. **Build a personal reference library.**
2. **Annotate principles**, don't hoard links.
3. **Reimplement in your tokens** within 24 hours of studying.
4. **Separate marketing craft from app craft.**
5. **Taste is trained**, not downloaded.

---

# SYNTHESIS.md (Frontend)

## The Common Failure Mode of Cheap Models

Same Inter UI, purple gradient, three feature cards, generic dashboard, inconsistent spacing. This file exists to break that attractor basin.

## Frontend Context Engineering Checklist

1. **Tokens** — color/space/type/radius defined?
2. **Hierarchy** — squint test pass?
3. **Composition** — one job per section; hero budget respected on landings?
4. **States** — empty/loading/error designed?
5. **A11y** — keyboard, focus, contrast, reduced motion?
6. **Motion** — only where it clarifies?
7. **References** — studied a real product (Linear/Stripe/Supabase/Cal) before inventing?
8. **Own the components** — primitives + your brand, not a docs clone?

## Proposed Library Shape (for a larger KB)

```
Frontend/
  Design Systems/
  Typography/
  Spacing/
  Animations/
  Dashboards/
  Landing Pages/
  Mobile UI/
  Accessibility/
  Forms/
  Charts/
  Navigation/
  Dark Mode/
  Glassmorphism/
  Gradients/
  Component Patterns/
  Color Theory/
  Visual Hierarchy/
  Microinteractions/
```

## Priority Repositories

| Rank | Repo |
|---|---|
| ★★★★★ | shadcn/ui |
| ★★★★★ | Magic UI |
| ★★★★★ | Aceternity UI |
| ★★★★★ | Motion |
| ★★★★★ | Refactoring UI (concepts) |
| ★★★★☆ | Radix UI |
| ★★★★☆ | Tremor |
| ★★★★☆ | React Bits |
| ★★★★☆ | Taxonomy |
| ★★★★☆ | React Aria |

## The One-Sentence Version

Great frontend context teaches **judgment under constraints** — hierarchy, space, type, state, and motion — so models compose interfaces like products, not like generic component demos.
