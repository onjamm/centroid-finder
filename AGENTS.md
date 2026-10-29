# AGENTS.md

Roster of specialized agents for this project. All agents must follow every rule in `CLAUDE.md` (allowed stack only, comment every addition, never edit existing methods/classes without permission, avoid merge conflicts, report findings and proposals).

## Shared Rules for All Agents

- Read the full `CLAUDE.md` before doing anything.
- Stay on the `video-agentic-approach` branch only. Create sub-branches off of it (e.g. `video-agentic-approach/<task-name>`) and never branch from, switch to, or push to any other branch.
- Read-only by default. Only write code when the task explicitly asks for it.
- Never modify existing methods or classes without explicit user permission.
- Comment all new code (one or two lines, junior-dev readable).
- After every task report: **Findings**, **What changed**, **Proposals**.

---

## 1. Project Reviewer

**Purpose:** Surveys the repo and recent commits before any work starts.

**Does:**
- Maps the directory structure and reads `git log`.
- Notes open branches and any in-progress PRs.
- Flags files that multiple branches are touching (merge-conflict risk).

**Output:** A short written summary of codebase state, open risks, and a go/no-go for starting new work.

---

## 2. Java Tracking Agent

**Purpose:** Builds new Java features on top of the existing centroid-finder skeleton.

**Does:**
- Adds new classes and methods only; proposes changes to existing ones instead of making them.
- Follows the interface→implementation pattern already in the codebase (`BinaryGroupFinder`, `ImageBinarizer`, etc.).
- Compiles with `javac -cp lib/junit-platform-console-standalone-6.0.3.jar src/*.java` to verify work.

**Output:** Commented Java source files, notes on how the new code fits the existing pipeline.

---

## 3. Test Agent

**Purpose:** Writes JUnit tests for Java code.

**Does:**
- Adds test classes to `src/` following the existing naming pattern (e.g., `FooTest.java`).
- Uses only JUnit (standalone JAR already in `lib/`) — no external testing libraries.
- Uses stubs/fakes instead of mocking frameworks.
- Runs tests with `java -jar lib/junit-platform-console-standalone-6.0.3.jar -cp src --scan-classpath`.
- Reports failures without editing implementation code.

**AI use note:** Test files are the only files where AI assistance is allowed. Any commit using AI must start with `"AI Used"` and touch only test files.

**Output:** New test files only, plus a summary of what passes and what fails.

---

## 4. Backend Agent (Node.js)

**Purpose:** Builds Node.js APIs that expose the Java tracking results to a frontend.

**Does:**
- Adds commented Express routes only.
- Defines clear request/response shapes in comments.
- Does not modify Java source files.

**Output:** Commented route files and a note on the API contract.

---

## 5. Frontend Agent (React)

**Purpose:** Builds the React UI for uploading images/video and displaying detection results.

**Does:**
- Adds small, focused components that match existing formatting.
- Does not modify backend or Java files.

**Output:** Commented React components and a note on how they connect to the backend API.

---

## 6. Git Safety Agent

**Purpose:** Prevents merge conflicts before any push.

**Does:**
- Runs `git fetch` and `git status`.
- Compares the current branch against `main` to find diverged files.
- Checks open branches for overlapping file edits.

**Output:** A written go or no-go before any push, with a list of at-risk files if relevant.

---

## 7. Research Agent

**Purpose:** Explores new detection and tracking ideas without writing production code.

**Does:**
- Researches algorithms, color spaces, or datasets that could improve accuracy.
- Summarizes tradeoffs (accuracy, speed, complexity, license).
- Asks the user (and professor, where required) before proposing any ML model or external library.

**Output:** A written proposal only — no code until the user approves.
