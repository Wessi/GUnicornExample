# OpenAI Codex: Complete Step-by-Step Reading Guide

## Based on the Net Ninja 11-video playlist, updated for the 2026 Codex workflow

**Playlist supplied by the reader:** OpenAI Codex Tutorial by Net Ninja  
**Playlist structure:** 11 lessons, roughly 1 hour 20 minutes total  
**Guide version:** September 20, 2026  
**Purpose:** turn the video course into a practical text manual you can read, search, highlight, and follow at your own pace.

> **Important note about this guide.** This is not a verbatim transcript of the videos. It is a comprehensive, lesson-by-lesson study guide that reconstructs the teaching sequence, explains the concepts in original wording, and updates the workflow against current OpenAI Codex documentation. Where the 2025 playlist and the 2026 product differ, the guide explicitly marks the difference.

---

# How to use this guide

You can read the guide from beginning to end, but it works best if you have a small Git repository open while reading. Do not begin with your most important production project. Use a toy project, a side project, or a branch that you can safely reset.

For every lesson, use the same learning loop:

1. **Read the concept.** Understand what Codex surface is being used and why.
2. **Run the smallest possible exercise.** Ask for an explanation or a tiny change before delegating a large feature.
3. **Review the diff.** Treat Codex like a collaborator whose work must be inspected, not like an infallible compiler.
4. **Run verification.** Tests, linting, type checks, build commands, or a manual smoke test should decide whether a change is acceptable.
5. **Commit or revert.** Keep good work in Git; discard bad work cleanly.
6. **Only then increase autonomy.** Move from a local edit to cloud delegation, code review, MCP tools, or parallel tasks.

The most important idea running through the entire playlist is that Codex is not one single chat box. It is a set of coding workflows that can share the same project context across the terminal, editor, cloud, and pull-request review process.

---

# Playlist map

| Lesson | Playlist topic | What you should be able to do afterward |
|---|---|---|
| 1 | Introduction & Setup | Understand the Codex ecosystem and connect a repository/environment |
| 2 | Running Cloud Tasks | Delegate a coding task to an isolated cloud environment and review its result |
| 3 | Codex Code Review | Ask Codex to review a pull request and act on high-priority findings |
| 4 | Using the Codex CLI | Work with Codex directly inside a local terminal repository |
| 5 | CLI Commands & Resuming Sessions | Control sessions, models, permissions, reviews, and continuation workflows |
| 6 | Using the AGENTS.md File | Give Codex persistent repository instructions and conventions |
| 7 | Codex IDE Extension | Work beside the code in VS Code-compatible editors |
| 8 | Context, Reasoning & TODOs | Supply better context, choose reasoning depth, and structure multi-step work |
| 9 | MCP Servers | Give Codex access to external tools and data through Model Context Protocol |
| 10 | Delegating Tasks to the Cloud | Move a task from the local/IDE workflow into Codex Cloud |
| 11 | Running Tasks in Parallel | Split independent work so several tasks can proceed simultaneously |

---

# Part I - The mental model before you touch anything

## 1. What Codex is

A useful way to think about Codex is: **an AI coding agent plus several places from which you can operate it**.

The 2025 course emphasizes four primary surfaces:

- **Codex Cloud** for remote, isolated coding tasks.
- **Codex CLI** for local terminal work.
- **Codex IDE extension** for working next to your code in an editor.
- **Codex code review on GitHub** for reviewing pull requests.

By 2026, the ecosystem is broader. The same core idea remains, but Codex can also be launched through current ChatGPT desktop workflows, can work with local/worktree/cloud execution modes, can connect to GitLab in supported workflows, can use MCP tools, skills/plugins, web search, images, and subagents, and can hand work between local and cloud environments.

The crucial distinction is **where the code is being executed**:

- **Local:** Codex reads, edits, and runs tools on your machine in the chosen project directory.
- **Isolated local worktree:** changes happen in a separate Git worktree, reducing interference with your active working tree.
- **Cloud:** Codex receives a configured repository environment in a remote container and works there independently.

The right surface depends on the task. A two-line CSS change is usually easier locally. A long refactor that needs installation, testing, and repeated iteration may be more suitable for the cloud. A review is naturally performed on the pull request. A question about the file you are already staring at is often easiest in the IDE.

## 2. The safety model you should adopt

Before using any coding agent, build a reversible workflow. The minimum is Git.

A good baseline routine is:

```bash
git status
git add -A
git commit -m "checkpoint before codex task"
```

If you do not want a commit yet, at least make sure you know which uncommitted changes are yours before Codex starts. The risk is not usually that Codex intentionally damages a repository. The practical risk is that you cannot distinguish your own half-finished work from the agent's edits, or you accept a plausible-looking change without verifying it.

Use these rules from the start:

- Never paste production secrets into prompts.
- Prefer environment variables or secret stores for credentials.
- Keep destructive operations behind approvals until you trust the workflow.
- Review diffs before committing or merging.
- Require tests or other verification for behavior changes.
- Ask Codex to explain uncertainty instead of inventing project facts.
- Give it the narrowest scope that can solve the problem.

## 3. A prompt is a task specification, not a magic spell

Good Codex prompts are usually structured like miniature tickets. They answer five questions:

**Goal:** What outcome should exist?  
**Scope:** Which area of the project is relevant?  
**Constraints:** What must not change?  
**Acceptance criteria:** How do we know the task is complete?  
**Verification:** What should Codex run or inspect before finishing?

A weak request:

```text
Fix the form.
```

A much better request:

```text
In the signup form, prevent submission when email is empty or invalid.
Keep the existing visual design and API contract unchanged.
Show an inline error below the email input.
Add or update tests for the validation behavior.
Run the relevant tests and summarize the files changed.
```

This pattern will appear throughout the rest of the guide.

---

# Lesson 1 - Introduction & Setup

## What the lesson is trying to teach

The first video introduces the Codex ecosystem and gets a repository connected so Codex can actually work on code. The central lesson is not a particular button. It is understanding that the cloud, CLI, IDE, and pull-request reviewer are different interfaces around a related coding-agent workflow.

The original course starts by showing Codex Cloud and connecting a GitHub repository. That still makes sense, but the current product also supports additional entry points and integrations. Treat the following as the current version of the setup process.

## Step 1 - Choose a practice repository

Pick a repository where you can safely create branches and throw work away. Ideal characteristics:

- Small enough that you understand the structure.
- Has a working install command.
- Has at least one test, lint, type-check, or build command.
- Does not require sensitive production credentials.
- Is already under Git version control.

If your project is not under Git:

```bash
git init
git add -A
git commit -m "initial checkpoint"
```

## Step 2 - Set up Codex Cloud

The current cloud workflow is conceptually:

1. Open Codex and sign in with your ChatGPT account.
2. Connect GitHub, or GitLab where the supported beta workflow is available.
3. Choose the repository/project Codex may access.
4. Create a cloud environment for that repository.
5. Configure installation steps, tools, environment variables, and secrets required by the project.
6. Start a small task.
7. Review the summary and diff before opening or merging a pull request.

A cloud environment is not simply a remote chat. When a cloud task starts, Codex works inside an isolated environment prepared for the repository. The environment should be reproducible enough that Codex can install dependencies and run the same checks you would run locally.

### A minimal environment checklist

Write down the answers before you delegate a serious task:

```text
Package manager: npm / pnpm / yarn / pip / uv / poetry / etc.
Install command: __________________________
Test command: _____________________________
Lint command: _____________________________
Build/type-check command: _________________
Runtime version: __________________________
Required non-secret environment variables: _
Required secrets: _________________________
Does the task need internet access? yes/no
```

Do not give the agent credentials it does not need. A test-only key is preferable to a production key. Read-only access is preferable to write access when that is enough.

## Step 3 - Install the CLI for local work

For current macOS/Linux environments, the official quick-start supports the standalone installer:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

Then move into a repository and launch:

```bash
cd path/to/your/project
codex
```

On first launch, choose a supported sign-in method, commonly signing in with ChatGPT.

### Windows note

The current OpenAI documentation provides a Windows/WSL workflow. A straightforward route is:

```powershell
wsl --install
wsl
```

Then inside the WSL shell:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
codex
```

For better filesystem performance, keep larger repositories inside the Linux filesystem (for example `~/code/my-app`) rather than doing intensive work under `/mnt/c/...`.

## Step 4 - Install or enable the IDE integration

Current Codex editor workflows include VS Code and compatible editors such as Cursor and Windsurf through the Codex extension, with separate integrations available for supported Xcode and JetBrains environments.

In a VS Code-compatible editor:

1. Install the Codex extension published by OpenAI.
2. Open your project folder.
3. Open the Codex sidebar. If it is hidden, use the command palette and run **Codex: Open Codex Sidebar**.
4. Sign in.
5. Keep a relevant file open and ask Codex to explain it before requesting an edit.

Your first editor prompt should be harmless:

```text
Explain the purpose of the file I have open, the main data flow through it,
and any other files I should read to understand it. Do not modify anything.
```

## Step 5 - Run a first safe task

Start with analysis, not implementation:

```text
Inspect this repository and tell me:
1. how the application starts,
2. where tests live,
3. what command runs the tests,
4. what command performs linting or type checking,
5. which three files are most important for understanding the main feature.
Do not edit files.
```

This teaches you two things: whether Codex can navigate your project, and whether its model of your repository matches reality.

## Lesson 1 checkpoint

You are ready for Lesson 2 when you can answer all of these:

- Which Codex surface are you using right now?
- Is the code executing locally, in a worktree, or in a cloud environment?
- Can you revert the next change?
- Does Codex know the project's install/test commands?
- Have you limited access to secrets and external systems?

---

# Lesson 2 - Running Cloud Tasks

## What the lesson is trying to teach

The second video moves from setup into actual cloud work. The example project in the lesson is a Next.js application, and the main idea is to let Codex inspect the project, perform a change in the remote environment, and then review what it produced.

The cloud workflow is strongest when the task has a clear end state and can be verified without constant human input.

## What happens when you send a cloud task

At a high level, a configured cloud run does this:

1. Creates or activates an isolated container for the task.
2. Checks out the selected branch or commit.
3. Runs the environment setup process.
4. Applies network/internet-access rules.
5. Lets Codex inspect files, edit code, and run commands within the configured permissions.
6. Produces a summary and a code diff for review.
7. Lets you follow up, request corrections, or open a pull request.

Current cloud environment documentation also distinguishes setup-time internet access from agent-time internet access. Do not assume the agent can reach arbitrary external services simply because dependency installation succeeded.

## The best first cloud task

Do not begin with “rewrite the app.” Choose something like:

- Add one field to a form.
- Fix one reproducible bug.
- Add a small unit test.
- Rename a well-contained concept.
- Improve one error state.
- Update a documentation page and verify links/tests.

### Example task specification

```text
Goal:
Add client-side validation to the newsletter signup form.

Behavior:
- Email is required.
- Email must look syntactically valid.
- On invalid input, show an accessible inline message.
- Do not submit the API request until the input is valid.

Constraints:
- Keep the existing API endpoint and payload shape unchanged.
- Do not add a new runtime dependency unless necessary.
- Preserve the current visual design.

Verification:
- Update/add tests covering empty email, invalid email, and valid submission.
- Run the relevant test suite and lint/type-check command.

Output:
Summarize the approach, files changed, commands run, and any remaining uncertainty.
```

Notice that this prompt gives the agent a decision boundary. It can decide implementation details, but it cannot silently redesign the API or install packages without a reason.

## Reading a cloud result

Do not read only the agent's prose summary. Review in this order:

1. **Changed files.** Are the files you expected the ones that changed?
2. **Diff size.** Is the amount of code proportionate to the task?
3. **Behavioral logic.** Does the implementation actually meet the acceptance criteria?
4. **Tests.** Were meaningful tests added or updated?
5. **Commands run.** Did Codex actually execute the verification it claims?
6. **Failure output.** If a test failed, did the agent acknowledge it or merely stop?
7. **Unrelated changes.** Look for formatting churn, dependency changes, generated files, and accidental refactors.

A good follow-up prompt is specific:

```text
The validation logic is correct, but do not change the shared Button component.
Revert that file, keep the form-level change, and rerun the same tests.
```

A poor follow-up is:

```text
Make it better.
```

## Environment configuration matters more than prompting

If every cloud task fails during setup, the answer is usually not “write a smarter prompt.” The environment is probably incomplete.

Typical environment problems:

- Wrong Node/Python/runtime version.
- Package manager mismatch.
- Missing system package.
- Missing environment variable.
- Test database unavailable.
- Setup script installs dependencies but does not generate required assets.
- Agent network access is disabled but the task requires a live documentation/API call.
- A command assumes a local service that does not exist in the cloud environment.

Keep setup deterministic. If the project requires ten manual steps on every new machine, Codex will inherit that fragility.

## A useful cloud-task prompt template

```text
You are working in [repository / area].

Objective:
[one concrete outcome]

Relevant context:
[bug reproduction, ticket details, important files, expected behavior]

Must preserve:
[API contracts, public interfaces, design conventions, backwards compatibility]

Do not:
[unrelated refactors, dependency changes, schema changes, external writes]

Acceptance criteria:
- ...
- ...
- ...

Verification:
Run [tests/lint/type-check/build]. If any verification cannot run, explain exactly why.

Before finishing:
Review the diff for unrelated changes and summarize the final implementation.
```

## Lesson 2 exercise

Delegate one small bug to the cloud. Do not merge it immediately. Compare the cloud solution with how you would have solved it yourself. Record:

- Which files Codex chose.
- Which commands it ran.
- One implementation decision you agree with.
- One thing you would change.
- Whether the environment was complete enough for verification.

---

# Lesson 3 - Codex Code Review

## What the lesson is trying to teach

This lesson connects Codex to the pull-request stage. Instead of only writing code, Codex becomes an additional reviewer on GitHub.

The 2025 lesson demonstrates enabling review for a repository, requesting review from a pull request, and using Codex's own findings to improve the branch. The current workflow preserves that pattern and adds more explicit review configuration.

## Set up review

The current GitHub review workflow is:

1. Configure Codex Cloud for the repository.
2. Open Codex settings.
3. Enable code review for the repository.
4. Open a pull request.
5. Request review with a pull-request comment:

```text
@codex review
```

Codex reacts and then posts a standard GitHub review. Automatic reviews can also be enabled so new pull requests are reviewed without manually adding the comment.

Current documentation says the GitHub review is deliberately focused on high-priority findings rather than flooding the PR with minor style comments. Treat that as an additional safety net, not a replacement for tests, required approvals, or human domain review.

## Ask for a focused review

You can narrow the request:

```text
@codex review for problems in the database migration and rollback path
```

Or:

```text
@codex review focusing on authentication regressions and missing tests
```

A focused request is useful when a PR contains a sensitive subsystem where generic review would waste attention.

## Turn a finding into a fix

After a finding is posted, a follow-up can ask Codex to fix it in context, for example:

```text
@codex fix the P1 issue
```

That can launch cloud work using the pull request as context and, where permissions allow, update the branch. You still review the resulting diff.

## Add repository-specific review rules with AGENTS.md

Current Codex review can read project instructions. A practical pattern is to add a section such as:

```markdown
## Code Review Rules

### API compatibility
- Flag any change that removes or renames a public response field without an explicit migration.

### Authentication
- Flag endpoints that read user-owned resources without an authorization check.

### Database changes
- Require a safe rollback or backward-compatible rollout plan for schema migrations.
```

Do not fill review rules with checks that deterministic tools already handle better. Formatting, import sorting, lint rules, and simple type errors belong in CI. Save Codex review instructions for semantic problems that require project context.

## Security review

Current Codex documentation also describes a deeper security-review workflow in research preview for supported setups. Where available, a pull-request comment can request a security-oriented review, for example:

```text
@codex security review
```

Use this as an additional security signal, not as proof that code is secure.

## How to evaluate a review comment

For each comment, ask:

1. Is the described execution path real?
2. Can I reproduce the problem?
3. Is it already prevented elsewhere in the codebase?
4. Is the severity appropriate?
5. What test would fail if the issue exists?
6. Does the proposed fix introduce another regression?

The purpose of AI review is not to accept every comment. It is to increase the chance that a meaningful issue is noticed before merge.

## Lesson 3 exercise

Create a branch with a small intentional bug in a practice repository. Open a PR and request Codex review. Before reading the review, write down the bug yourself. Then compare:

- Did Codex identify the actual bug?
- Did it invent any issue that is not real?
- Was the explanation actionable?
- Could an `AGENTS.md` review rule make the result more project-specific?

---

# Lesson 4 - Using the Codex CLI

## What the lesson is trying to teach

This lesson shifts from remote cloud work to local terminal work. The key benefit of the CLI is speed and proximity: Codex can inspect the repository, edit files, and run the tools already installed on your machine without requiring you to leave the terminal.

## Start in the correct directory

Always verify the working directory before launching an agent:

```bash
pwd
git status
codex
```

The CLI operates in relation to the project directory and its instructions. Starting in the wrong folder can give Codex the wrong repository root or the wrong `AGENTS.md` chain.

## Learn the basic interactive loop

A productive local session usually looks like this:

1. Ask Codex to inspect or explain.
2. Ask for a plan if the task is nontrivial.
3. Let it edit a constrained set of files.
4. Let it run verification.
5. Inspect the diff.
6. Ask for one correction if needed.
7. Run `/review` or your own Git/test review.
8. Commit when satisfied.

Start with a read-only request:

```text
Trace how an HTTP request to POST /api/orders is validated, authorized,
and persisted. Name the relevant files and describe the flow. Do not edit anything.
```

Then request a change:

```text
Add a validation rule that quantity must be a positive integer.
Preserve the public error response format.
Add a test for zero, negative, decimal, and valid quantities.
Run the focused tests before finishing.
```

## Permissions are part of the task design

Modern Codex CLI exposes controls for what the agent may edit or execute. Use the `/permissions` command to inspect or adjust boundaries for the current work.

The principle is simple:

- For repository exploration, narrow permissions are enough.
- For normal coding, file edits and routine test commands may be appropriate.
- For destructive shell commands, package publishing, infrastructure changes, database writes, or external systems, require explicit review/approval.

Do not grant broad autonomy just to remove a confirmation dialog. Grant only what the task needs.

## Choose model/reasoning deliberately

The `/model` command lets current Codex workflows choose a model and reasoning effort. You do not need maximum reasoning for every task.

A practical rule:

- **Low/fast:** file discovery, simple explanations, tiny mechanical edits.
- **Medium:** most everyday feature work, tests, and debugging.
- **High:** complicated multi-file changes, ambiguous failures, architecture-sensitive work.

Higher reasoning can cost more time and usage. The right question is not “which setting is smartest?” but “what level is sufficient for this task?”

## Add current external context only when needed

Current CLI workflows can use live web search for tasks that depend on fresh documentation or versions. The CLI also supports image context for screenshots, diagrams, and UI references.

Examples:

```bash
codex --search
```

and, depending on your workflow, an image can be attached at launch or pasted into the interactive editor.

Use current web context when the answer depends on external facts. Do not use it as a substitute for reading your own repository.

## Local CLI prompt patterns

### Explain before editing

```text
Explain why the checkout integration retries requests here.
Show me the call chain and tests that constrain the behavior.
Do not modify files.
```

### Bug fix

```text
Reproduce the failing test first. Identify the smallest root-cause fix.
Do not weaken the assertion or skip the test. Implement the fix and rerun the suite.
```

### Refactor

```text
Refactor the parsing logic into a small pure function without changing behavior.
Keep the public API unchanged. Move existing tests only if necessary and add tests
for any edge case revealed during extraction.
```

### Test generation

```text
Study the existing test style for this module. Add tests for the uncovered error paths.
Do not change production behavior unless you find a real bug; if you find one, stop and explain it first.
```

## Use Git as the arbiter

At any point:

```bash
git diff
git status
```

You should be able to explain every changed file. If you cannot, do not commit yet.

---

# Lesson 5 - CLI Commands & Resuming Sessions

## What the lesson is trying to teach

This lesson is about operating Codex as an ongoing development tool rather than a one-shot prompt. Sessions have configuration, history, and continuation. Learning the control commands gives you a much more predictable workflow.

## Core interactive commands to know

Current Codex CLI highlights these commands directly in its quick-start interface:

```text
/init         create an AGENTS.md file with project instructions
/status       show the current session configuration
/permissions  choose what Codex is allowed to do
/model        choose model and reasoning effort
/review       review changes and identify issues
```

The exact command set can grow over time, so use `?`, help output, or current documentation if your installed version differs.

## `/init` - bootstrap project guidance

Use `/init` when a repository does not yet have agent instructions and you want Codex to create a starting `AGENTS.md`.

Do not accept the generated file blindly. Edit it into a concise project contract. The best `AGENTS.md` is not a novel; it contains the stable rules Codex repeatedly needs.

## `/status` - verify where and how you are operating

Use status before a sensitive task. You want to know:

- project/workspace root,
- active model/reasoning,
- permission/sandbox behavior,
- and any other current session configuration that affects execution.

A surprisingly large number of agent mistakes are actually operator-context mistakes: wrong directory, wrong branch, stale assumptions, or overly broad permissions.

## `/review` - ask for a second pass

After implementation, use the review command as a dedicated bug-finding pass. A review mindset is different from a coding mindset. The agent should look for regressions, edge cases, missing tests, and unintended behavior rather than merely summarizing what it just wrote.

You can also explicitly ask:

```text
Review only the uncommitted diff. Prioritize correctness and regressions.
Do not edit files. Give findings first with file/line references, then a short summary.
```

## Resume a previous session

Current CLI exposes:

```bash
codex resume
```

This is useful when you stop work and later want to continue with the same conversation context. Resume is powerful, but do not confuse conversation continuity with repository truth. The branch may have changed since the previous session.

After resuming, re-anchor the agent:

```text
Before continuing, inspect git status and the current diff.
Summarize what changed in the repository since our last step and restate the remaining task.
```

## Non-interactive work with `codex exec`

For repeatable scripts and CI-style workflows, current Codex supports non-interactive execution through `codex exec`.

Conceptually:

```bash
codex exec "review the current branch for likely regressions and output a concise report"
```

Use non-interactive execution when the task has a deterministic input/output contract. Avoid it for a vague feature that requires frequent human decisions.

## Other useful current CLI capabilities

Current documentation also surfaces workflows such as:

```text
codex --image      add visual context
codex --search     enable live web context for a run
codex cloud        browse/submit/apply cloud work from the terminal
codex mcp          configure or inspect MCP integrations
codex completion   generate shell completion support
```

Again, your installed version is the source of truth for exact flags.

## When to start a new session instead of resuming

Start fresh when:

- The task has changed substantially.
- The branch has been rebased or replaced.
- Old assumptions are contaminating the discussion.
- You want a clean independent review.
- The conversation has accumulated too many irrelevant details.

A fresh session with a good `AGENTS.md` and a precise prompt is often better than dragging months of context forward.

---

# Lesson 6 - Using the AGENTS.md File

## What the lesson is trying to teach

`AGENTS.md` is one of the most important concepts in the course. It is a persistent instruction file for coding agents. Instead of repeatedly telling Codex how to install, test, format, and structure work, you place stable project guidance in the repository.

By 2026, Codex's `AGENTS.md` behavior is more formally documented and supports layered instructions.

## Think of AGENTS.md as the project's operating manual for the agent

Good content includes:

- Which package manager to use.
- How to run tests, lint, type checks, and builds.
- Architectural boundaries.
- Naming conventions that are not enforced automatically.
- Rules for migrations or public APIs.
- Files/directories that should not be edited.
- What counts as “done.”
- Code-review rules that require domain understanding.

Bad content includes:

- Generic advice such as “write clean code.”
- Long tutorials the agent can infer from the repository.
- Duplicated formatter/linter rules.
- Volatile ticket-specific instructions that belong in a prompt.
- Secrets.

## How Codex currently finds AGENTS.md instructions

Current Codex builds an instruction chain when a run/session starts.

### Global scope

By default, Codex looks under its home configuration directory, commonly:

```text
~/.codex/
```

It can load a global `AGENTS.md`, or an `AGENTS.override.md` that temporarily takes precedence at that level.

A global file is good for personal working preferences that apply across repositories, such as:

```markdown
# ~/.codex/AGENTS.md

## Working preferences
- Explain destructive commands before running them.
- Prefer adding focused tests with behavioral changes.
- Do not add a new production dependency without stating the reason.
```

### Project scope

From the project root toward the current working directory, Codex looks for instructions in each directory. Current precedence checks `AGENTS.override.md` before `AGENTS.md` in a directory. More specific nested instructions appear later in the combined instruction chain and therefore take precedence over broader parent instructions.

This means a monorepo can have:

```text
repo/
  AGENTS.md
  apps/
    web/
      AGENTS.md
  services/
    payments/
      AGENTS.override.md
```

If you start Codex inside `services/payments`, it can inherit root rules and then apply the more specific payment-service rules.

Current documentation also sets a default combined instruction size limit, so giant instruction documents can be truncated. Keep rules concise and move specialized rules nearer to the code they govern.

## A practical AGENTS.md template

Use this as a starting point and edit it for your project:

```markdown
# AGENTS.md

## Project summary
This repository contains a web application and API for managing customer orders.

## Tooling
- Use `pnpm`; do not generate npm or yarn lockfiles.
- Install dependencies with `pnpm install`.
- Run unit tests with `pnpm test`.
- Run lint with `pnpm lint`.
- Run type checks with `pnpm typecheck`.

## Development rules
- Keep request validation in the existing validation layer.
- Do not access the database directly from UI components.
- Preserve public API response fields unless the task explicitly includes a migration.
- Prefer existing utilities before adding a new dependency.

## Testing
- Behavior changes require a focused test.
- Bug fixes should reproduce the bug in a failing test when practical.
- Do not skip or weaken existing tests to make the suite pass.

## Scope and safety
- Do not modify generated files manually.
- Do not rotate keys, change cloud infrastructure, or run production migrations.
- Never commit secrets or local `.env` contents.

## Before finishing
- Review `git diff` for unrelated edits.
- Run relevant tests plus lint/type checks for touched code.
- Summarize files changed and verification performed.

## Code Review Rules
- Flag authorization regressions on user-owned resources.
- Flag database migrations that are not backward compatible with the previous application version.
- Flag removal or renaming of documented public API response fields.
```

## Use overrides sparingly

An `AGENTS.override.md` is useful when a subtree genuinely has different requirements. It is not a way to accumulate contradictory instructions.

Examples of valid specialization:

- Payments service uses a different test command.
- Mobile app has platform-specific build requirements.
- Infrastructure directory forbids automatic apply/deploy operations.
- Generated SDK directory should not be hand-edited.

## Verify that Codex loaded the rules

Do not assume. Ask:

```text
Summarize the active project instructions that apply to this directory.
Do not edit anything.
```

If it omits a rule you expect, check whether you started in the right directory, whether an override exists, and whether the file is empty or too large.

---

# Lesson 7 - Codex IDE Extension

## What the lesson is trying to teach

The IDE extension puts the agent next to the code you are actively reading. This reduces prompt overhead because open files and selected code can become part of the working context.

The best use of the IDE is not “chat inside VS Code.” It is **contextual collaboration around the exact code on your screen**.

## Install and open it

In a supported VS Code-compatible editor:

1. Install/enable the Codex extension.
2. Open your repository.
3. Open the Codex sidebar.
4. Sign in.
5. Open the file you want to discuss.

Current docs emphasize that VS Code, Cursor, and Windsurf use the extension, while supported Xcode and JetBrains integrations follow their own interface patterns.

## Use editor context deliberately

Three excellent IDE actions are:

### Explain a selection

Select a function and ask:

```text
Explain this function's preconditions, side effects, and failure paths.
Which tests currently exercise it?
```

### Request a focused edit

```text
Update only the selected function so it returns a typed error for invalid input.
Do not change callers yet. Show me the diff and explain any compatibility concern.
```

### Ask for cross-file context

```text
This component calls `saveProfile`. Trace that function to the network layer,
identify the API contract, and tell me where errors are converted to UI messages.
Do not edit files.
```

## Review diffs in the editor

One of the advantages of the IDE workflow is that source and proposed changes are near each other. Read the diff before accepting it.

When reviewing, ask:

- Did it touch only the intended files?
- Did it preserve local style?
- Did it duplicate an existing helper?
- Did it change behavior beyond the request?
- Are error paths handled?
- Were tests updated?

If you want only part of a change, say so. The agent can iterate; you do not need to accept an all-or-nothing result.

## Local versus cloud from the IDE

Current Codex IDE workflows support a natural escalation path:

- Keep quick feedback loops local.
- Delegate larger or longer-running work to Codex Cloud.
- Return to the editor and review the result.

Use cloud delegation when the task is independent enough to run without your constant decisions.

## A productive IDE session example

Start:

```text
Explain the selected reducer and the state invariants it assumes. Do not edit.
```

Then:

```text
There is a bug when an item is removed twice from an optimistic update queue.
Create a minimal test that reproduces it. Show me the failing test before changing production code.
```

Then:

```text
Implement the smallest fix, rerun that test and the reducer test file,
and show me the final diff.
```

Finally:

```text
Review the diff for any behavioral regression or missing edge case. Do not edit.
```

This is more reliable than a single mega-prompt because each step creates a verification point.

---

# Lesson 8 - Context, Reasoning & TODOs

## What the lesson is trying to teach

This lesson explains why agent quality depends heavily on context and task structure. Codex performs better when it knows the relevant code, understands the desired outcome, and can break a complex change into explicit subproblems.

## Context is not “more text is always better”

Useful context is:

- Relevant files.
- A reproducible error.
- A failing test.
- A stack trace.
- A screenshot.
- An API contract.
- Existing project instructions.
- A short explanation of the expected behavior.

Unhelpful context is a dump of the entire repository plus an ambiguous request.

### Context hierarchy

Give context in this order:

1. Persistent project rules in `AGENTS.md`.
2. Current task goal and constraints in the prompt.
3. Explicit relevant files/selections.
4. Runtime evidence: error output, tests, logs, screenshots.
5. External docs only when the task depends on them.

## Reasoning effort is a resource

Current Codex exposes reasoning choices through the model/config workflow. Higher reasoning is useful when the agent must reconcile many constraints or investigate an unclear failure.

Use more reasoning for:

- multi-module refactors,
- intermittent or non-obvious bugs,
- architecture changes,
- concurrency problems,
- security-sensitive changes,
- tasks where several plausible implementations exist.

Use less for:

- renaming,
- formatting-safe mechanical edits,
- adding an obvious missing test,
- simple explanations,
- small documentation changes.

The goal is not maximum thinking. It is enough thinking to solve the task reliably.

## TODOs and plans

For multi-step work, ask Codex to make the plan explicit before editing:

```text
Before changing files, create a short implementation plan with 4-7 concrete steps.
Identify the files likely to change and the verification for each step.
Then execute the plan, keeping the scope limited to this feature.
```

A good plan is actionable:

```text
1. Reproduce the failing checkout test.
2. Trace retry-state ownership.
3. Add a regression test for the duplicate retry.
4. Change the retry guard in one module.
5. Run checkout tests and type checks.
6. Review the diff for unrelated changes.
```

A bad plan merely repeats the prompt:

```text
1. Understand the issue.
2. Fix the issue.
3. Test the issue.
```

## Plan mode versus implementation mode

When you want analysis before edits, state it clearly:

```text
Do not edit yet. Investigate the bug, list the most likely root cause with evidence,
and propose the smallest fix plus a verification plan.
```

Then, after you agree:

```text
Proceed with option 1 only. Keep the public API unchanged and run the tests listed above.
```

This human-in-the-loop boundary is especially valuable when the first implementation decision is hard to reverse.

## Keep TODOs honest

If a task ends with items still unverified, the final summary should say so. Do not accept vague “should work” language for things that could have been tested.

Ask for a closure report:

```text
Before finishing, reconcile every plan item as Done, Blocked, or Not needed.
For blocked verification, give the exact command you attempted and the reason it could not complete.
```

---

# Lesson 9 - MCP Servers

## What the lesson is trying to teach

MCP stands for **Model Context Protocol**. It is a standard way for an AI client to connect to external tools and information sources. In the playlist, examples such as Context7 and MCP server directories are used to show how Codex can extend beyond local repository files.

The key idea: **MCP gives Codex tools; it does not magically make every tool safe.**

## What MCP is useful for

Common categories include:

- Up-to-date library documentation.
- Issue trackers.
- Design systems.
- Databases or analytics systems.
- Browser/devtools integrations.
- Internal APIs.
- File/document systems.

For a coding task, an MCP documentation server can be better than relying on the model's memory of a library version.

## Current Codex MCP configuration

Current OpenAI documentation says Codex stores MCP configuration alongside other Codex settings, commonly in:

```text
~/.codex/config.toml
```

Trusted projects can also use project-specific configuration under:

```text
.codex/config.toml
```

The ChatGPT desktop Codex workflow, Codex CLI, and the IDE extension can share configured MCP connections, reducing repeated setup across clients.

## Add a server from the CLI

The current docs provide a CLI pattern:

```bash
codex mcp add <server-name> -- <stdio-server-command>
```

For example, a documentation-oriented Context7 setup is shown as:

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

Then inspect configured servers:

```bash
codex mcp list
```

Depending on the server, authentication can involve environment variables or OAuth.

## Use MCP intentionally

After adding a documentation tool, do not simply say “use MCP.” Tell Codex why:

```text
Use the Context7 MCP tool to verify the current API for the version of the library
listed in package.json. Then update only the integration code that is incompatible.
Cite the documentation point you relied on in your summary.
```

This prevents a tool call from becoming an unnecessary detour.

## Security rules for MCP

Treat every MCP server as code/data access that can expand the agent's capabilities.

Before enabling a server, ask:

- Who operates it?
- What data can it read?
- What actions can it perform?
- Does it write or only read?
- What credentials does it receive?
- Can it access production systems?
- Does the task require that access?

Prefer read-only tools until a workflow clearly requires writes. Do not hand an agent a production database mutation tool merely because it is convenient.

## MCP troubleshooting checklist

If a tool is unavailable:

1. Run `codex mcp list`.
2. Confirm the server command or URL is correct.
3. Check required environment variables.
4. Complete OAuth/login if required.
5. Restart the client/session if configuration is loaded at startup.
6. Confirm the project is trusted if using project-local config.
7. Ask Codex to list available MCP tools before requesting a task.

---

# Lesson 10 - Delegating Tasks to the Cloud

## What the lesson is trying to teach

This lesson connects the local/IDE experience back to cloud execution. You can begin while looking at code locally, then hand a longer task to a remote environment instead of keeping your machine and attention occupied.

## When delegation is a good idea

Delegate when the task is:

- Clearly scoped.
- Time-consuming.
- Verifiable by tests/builds.
- Mostly independent of the code you are currently editing.
- Suitable for an isolated branch/environment.

Examples:

- Add tests across several modules.
- Upgrade a dependency and resolve compilation issues.
- Implement a self-contained feature behind an existing interface.
- Investigate a failing CI job.
- Refactor an internal module with stable tests.
- Update documentation across a large set of files.

Keep work local when:

- You are still deciding what the feature should do.
- You need rapid visual/manual feedback every minute.
- The task relies on local hardware or private state unavailable in the cloud.
- The change overlaps heavily with uncommitted local work.

## Prepare the handoff

Before delegating, make the task self-contained:

```text
Objective: ...
Starting branch/commit: ...
Relevant files: ...
Constraints: ...
Acceptance criteria: ...
Verification commands: ...
Known failing behavior: ...
Do not touch: ...
```

If you cannot describe the task this way, it probably needs more local investigation first.

## Delegation from current interfaces

Current Codex workflows can hand work to the cloud from the IDE, and the CLI exposes a `codex cloud` workflow for browsing cloud chats, submitting work to a configured environment, and applying results locally.

The exact UI evolves, but the handoff concept remains:

1. Capture sufficient task context.
2. Select the correct cloud environment/repository.
3. Send the task.
4. Continue other work.
5. Return to the completed result.
6. Review the summary, diff, and verification.
7. Ask for a correction or apply/open a PR.

## Do not confuse delegation with abandonment

Cloud work is asynchronous from your attention, not from responsibility. The human still owns the merge decision.

A useful review prompt after a cloud task finishes:

```text
Before I apply this result locally, give me a risk-oriented review of your own diff.
List any changed public behavior, migration risk, missing verification, or assumption
about the environment. Do not make additional edits.
```

---

# Lesson 11 - Running Tasks in Parallel

## What the lesson is trying to teach

The final playlist lesson shows one of the main advantages of cloud agents: independent work can happen at the same time. Instead of waiting for Task A to finish before starting Task B, you can run multiple isolated tasks and review their results as they complete.

## Parallelize independent work, not conflicting work

Good parallel task set:

- Task A: add API validation tests.
- Task B: update developer documentation.
- Task C: investigate a frontend accessibility issue.

Risky parallel task set:

- Task A: refactor authentication middleware.
- Task B: change authentication error handling.
- Task C: rename authentication types.

The second set touches the same conceptual and file area, so each result is based on a different snapshot/assumption. Merging them may cost more than doing them sequentially.

## A practical parallelization test

Before splitting, ask:

1. Can each task start from the same base commit?
2. Can each task be reviewed independently?
3. Do they touch mostly different files?
4. Does one task's design decision affect the others?
5. Can their tests run independently?

If answers 1-3 are yes and 4 is no, parallel execution is probably a good fit.

## Compare alternative implementations in parallel

Parallelism is not only for different tasks. You can ask for independent solutions to the same problem, then compare them.

Example:

```text
Task A: solve the caching bug with the smallest code change.
Task B: solve the same bug by making cache ownership explicit in the service layer.
Both must preserve the public API and pass the same test suite.
```

Do not merge both. Use the results as competing design proposals.

## Parallel cloud tasks versus current subagents

By 2026, Codex also supports subagent workflows in current clients. These are related but not identical concepts.

- **Parallel cloud tasks:** separate task runs/environments that you review as separate work products.
- **Subagents:** a parent Codex workflow delegates parts of a larger investigation or task to specialized agent threads, then combines their findings.

Subagents are especially useful for parallel reading, exploration, test analysis, or independent review dimensions. Be more cautious when multiple agents write overlapping code because merge/conflict coordination becomes part of the problem.

### Example subagent review request

```text
Review this branch using parallel subagents.
- One agent: security risks.
- One agent: missing or weak tests.
- One agent: maintainability/regression risks.
Wait for all of them, then return one deduplicated findings list with file references.
Do not edit code.
```

## Manage parallel work like a small queue

For each task, track:

```text
Task name:
Base commit:
Owner/agent:
Scope:
Expected files:
Verification:
Status:
Result/PR:
Merge order dependency:
```

The bottleneck quickly becomes review, not generation. Do not start ten tasks if you can only carefully review two.

---

# Part II - The complete everyday Codex workflow

## Step 1 - Start from a clean, understood Git state

```bash
git status
git branch --show-current
git log -1 --oneline
```

Commit or stash unrelated work. For a risky task, create a branch:

```bash
git switch -c codex/fix-order-validation
```

## Step 2 - Ask Codex to understand before editing

```text
Inspect the order validation flow. Identify the request schema, validation logic,
error conversion, and relevant tests. Do not edit files. Tell me what you would change
for the bug described below and why.
```

Read the answer. Correct misunderstandings before implementation.

## Step 3 - Lock the task boundary

```text
Proceed with the smallest fix.
Do not change the response schema.
Do not add dependencies.
Add a regression test.
Run the focused tests and type check.
```

## Step 4 - Let Codex implement and verify

Watch for tool output. If the task is local, do not interrupt every harmless read command, but do pay attention to package installs, deletes, migrations, external writes, and broad refactors.

## Step 5 - Inspect the diff yourself

```bash
git diff --stat
git diff
```

Review logic, not only syntax.

## Step 6 - Run a dedicated review pass

Use `/review`, GitHub review, or a fresh Codex session. A fresh reviewer can catch assumptions the authoring session is biased toward.

Prompt:

```text
Review the uncommitted diff as if you did not write it.
Prioritize correctness, behavior regressions, authorization/data issues, and missing tests.
Do not comment on style already enforced by linting.
Do not edit.
```

## Step 7 - Run verification independently if practical

Even if the agent reports tests passing, you may run them yourself:

```bash
pnpm test
pnpm lint
pnpm typecheck
```

Or the equivalent for your project.

## Step 8 - Commit with a human-readable message

```bash
git add -A
git commit -m "fix order quantity validation"
```

## Step 9 - Use Cloud/PR review for larger changes

For longer work, delegate to Cloud, open a PR, and request Codex review plus human review as appropriate.

## Step 10 - Merge only after the evidence matches the claim

The implementation says “fixed.” The tests, diff, manual behavior, and review should agree.

---

# Part III - Prompt cookbook

## 1. Repository orientation

```text
Map this repository for a new developer.
Explain the application entry points, major modules, persistence layer, test layout,
and the commands used for development and verification.
Name the 8-12 files I should read first.
Do not modify anything.
```

## 2. Reproduce-before-fix bug prompt

```text
Investigate this bug: [description].
First find or add the smallest test that reproduces it. Do not change production code
until you can explain the root cause with file references.
Then implement the smallest fix that preserves existing public behavior.
Run the focused tests and relevant static checks.
```

## 3. Feature prompt

```text
Implement [feature].

User-visible behavior:
- ...
- ...

Constraints:
- Preserve ...
- Do not add ...
- Follow the existing pattern in ...

Acceptance criteria:
- ...
- ...

Testing:
- Add/update tests for ...
- Run ...

Before finishing, review the diff for unrelated changes and summarize any unresolved risk.
```

## 4. Refactor-without-behavior-change prompt

```text
Refactor [module] to reduce duplication while preserving behavior and public interfaces.
Use existing tests as the behavioral contract.
Do not redesign unrelated code.
If tests are insufficient to prove behavior, add characterization tests first.
Run the relevant suite and show the final diff summary.
```

## 5. Test-gap prompt

```text
Do not edit production code initially.
Review the tests for [module] and identify behavior branches that are not exercised.
Prioritize bugs that could escape because of those gaps.
Then add focused tests following the existing test style.
If a new test reveals a real bug, stop and explain it before fixing production code.
```

## 6. Performance investigation prompt

```text
Investigate the slow path in [operation].
Do not optimize blindly.
First identify where time/allocations/queries are likely spent using existing profiling,
logs, or code evidence. Propose up to three interventions ranked by expected impact and risk.
Only implement the smallest high-confidence improvement after explaining the evidence.
```

## 7. Code review prompt

```text
Review this diff for correctness and regressions.
Focus on:
- data integrity,
- authorization/authentication,
- concurrency/state bugs,
- API compatibility,
- missing tests.
Do not report formatting or lint issues that CI will catch.
Give findings first, ordered by severity, with file references.
Do not edit.
```

## 8. Cloud delegation prompt

```text
Work in the configured cloud environment on this self-contained task.
Base your changes on the selected branch and do not change unrelated files.

Goal: ...
Constraints: ...
Acceptance criteria: ...
Verification commands: ...

If the environment cannot run verification, diagnose the environment problem rather than
claiming success. Return a concise summary, changed files, tests run, and remaining risk.
```

## 9. Documentation-with-MCP prompt

```text
Use the configured documentation MCP tool to verify the current API for [library]
matching the version in this repository. Compare that documentation with our current usage.
Make only the compatibility changes needed, then run the relevant tests/type checks.
In the final summary, state which current API behavior required each change.
```

## 10. Parallel investigation prompt

```text
Split this investigation into three independent tracks:
1. reproduce the failure and inspect tests,
2. trace the data/control flow that leads to the failure,
3. inspect recent changes likely related to the regression.
Run the tracks in parallel if supported. Do not edit code during investigation.
Combine the evidence into one root-cause hypothesis with confidence and next steps.
```

---

# Part IV - Troubleshooting

## Problem: Codex edits too much

**Symptoms:** many unrelated files, formatting churn, unsolicited refactor.

**Fix:** reduce scope and explicitly prohibit unrelated work.

```text
Revert changes outside `src/orders/` and `tests/orders/`.
Keep only changes necessary for the stated bug. Do not reformat unrelated code.
```

Also check whether the project's formatter runs across the entire repository.

## Problem: Codex keeps using the wrong package manager

Put the rule in `AGENTS.md`:

```markdown
- Use pnpm only. Do not run npm install or create package-lock.json.
```

Then verify the instruction is active.

## Problem: Cloud task cannot run tests

Investigate environment reproducibility:

- Runtime version.
- Install command.
- System packages.
- Generated code/assets.
- Test database/service.
- Secrets/environment variables.
- Network access.

Ask for a diagnostic task instead of retrying the feature prompt unchanged:

```text
Do not edit application code. Diagnose why the standard test command cannot run in this
cloud environment. Identify the first failing setup prerequisite and propose the minimal
environment configuration change.
```

## Problem: Codex claims tests pass but you cannot reproduce it

Compare:

- exact command,
- working directory,
- environment variables,
- runtime/package versions,
- selected tests versus full suite,
- cached/generated artifacts.

Require exact command output in the task log or summary.

## Problem: A resumed session acts on stale assumptions

After resume:

```text
Re-read git status, the current diff, and the files relevant to our task.
State what has changed since the last session before doing any new edits.
```

Start a new session if the branch has changed dramatically.

## Problem: MCP server is configured but unused

Ask Codex to enumerate available tools and explicitly instruct when to use the server. Some tasks do not require MCP, and the agent may correctly avoid it.

## Problem: MCP server has too much access

Disable it for the task or replace it with a read-only credential/tool. Convenience is not a reason to expose write access to production systems.

## Problem: GitHub review is noisy or generic

Add concise semantic `Code Review Rules` in `AGENTS.md`. Remove rules that duplicate CI. Focus on project-specific failure modes.

## Problem: Parallel tasks are hard to merge

Your tasks were not independent enough. Restart from a shared base and either:

- narrow each task to separate files/interfaces, or
- serialize the dependent changes.

Parallel generation is only useful when integration remains cheaper than sequential work.

## Problem: The agent gets lost in a huge task

Split by deliverable:

```text
Phase 1: investigate and produce a plan only.
Phase 2: implement the data-layer change and tests.
Phase 3: implement the UI change and tests.
Phase 4: integration review.
```

Commit between phases.

---

# Part V - What changed since the 2025 playlist

The playlist is still a good conceptual introduction, but Codex has evolved. The most important 2026 additions/changes to know are:

## 1. More execution surfaces

Current Codex documentation describes local, Git worktree, and cloud execution choices in current desktop workflows. This makes isolation a first-class choice instead of relying only on your active working directory.

## 2. Cloud integrations expanded

Current Cloud workflows can be initiated not only from the Codex web experience but also through supported GitHub/GitLab, Linear, and Slack integrations. Availability can depend on account/workspace configuration.

## 3. CLI workflow is richer

Current CLI documentation highlights session resumption, image context, live web search, cloud handoff, MCP configuration, shell completions, permissions, model/reasoning choices, code review, and non-interactive `codex exec` workflows.

## 4. AGENTS.md behavior is explicitly layered

The current system supports global and project-scoped instructions, nested specialization, and `AGENTS.override.md` precedence. This is more powerful than treating `AGENTS.md` as one flat repository note.

## 5. Code review has custom rules and security-oriented review

Current GitHub review can use repository-specific review rules from `AGENTS.md`. Supported setups also expose a deeper security-review workflow.

## 6. MCP configuration is shared across Codex clients

Current docs describe common MCP configuration that can be used by the desktop Codex experience, CLI, and IDE extension, with local or remote servers and authentication as needed.

## 7. Subagents complement parallel cloud tasks

Current Codex can split suitable investigations into subagents and combine their results. This is useful for parallel reading, analysis, testing, and review, while overlapping code writes still require care.

## 8. Skills/plugins are an additional customization layer

The current customization model goes beyond `AGENTS.md` and MCP. Reusable skills can package repeatable workflows, and plugins can provide connected tools/data in supported environments. You do not need these to learn the playlist, but they are the natural next step after mastering the fundamentals.

---

# Part VI - A 7-day practice plan

## Day 1 - Orientation and safe local use

- Install/launch Codex CLI or IDE integration.
- Use a practice repository.
- Ask for repository mapping without edits.
- Verify `git status` and run the project's tests yourself.
- Make one tiny local change and inspect the diff.

**Goal:** become comfortable with the read -> edit -> diff -> test loop.

## Day 2 - Better prompts and verification

- Fix a small bug using reproduce-before-fix prompting.
- Require a regression test.
- Ask Codex to run focused tests.
- Use a separate review pass.

**Goal:** make evidence, not prose, define completion.

## Day 3 - AGENTS.md

- Create a concise root `AGENTS.md`.
- Add package manager, verification commands, architecture rules, and safety constraints.
- Ask Codex to summarize active instructions.
- Run the same task again and compare consistency.

**Goal:** stop repeating stable project rules in every prompt.

## Day 4 - Cloud task

- Configure a cloud environment for the practice repo.
- Delegate one self-contained change.
- Review summary, logs, diff, and tests.
- Request one follow-up correction.

**Goal:** learn to hand off without losing review discipline.

## Day 5 - Pull-request review

- Open a practice PR.
- Request Codex review.
- Evaluate each finding manually.
- Add one project-specific review rule to `AGENTS.md` and test it on a representative change.

**Goal:** use Codex as a reviewer, not only an author.

## Day 6 - MCP and fresh documentation

- Configure a safe, read-oriented MCP server such as current library documentation.
- Ask Codex to verify an API against the dependency version in your project.
- Inspect the tool use and final change.

**Goal:** learn when external tools improve accuracy.

## Day 7 - Parallelism and handoff

- Define three independent tasks from the same base commit.
- Run them in parallel cloud tasks or use subagents for parallel investigation where appropriate.
- Review and integrate only the useful results.

**Goal:** experience the real constraint: human review and integration capacity.

---

# Part VII - Quick-reference cheat sheet

## Before a Codex task

```text
[ ] Correct repository and branch
[ ] Git state understood / checkpoint exists
[ ] Goal is one concrete outcome
[ ] Constraints are explicit
[ ] Acceptance criteria are testable
[ ] Verification command is known
[ ] Secrets/external systems limited to what is necessary
[ ] Relevant AGENTS.md instructions are active
```

## During a task

```text
[ ] Agent is touching expected files
[ ] Broad/destructive commands receive scrutiny
[ ] Environment failures are diagnosed rather than hidden
[ ] Scope is not silently expanding
[ ] Plan/TODO items remain aligned with the goal
```

## Before accepting a result

```text
[ ] Read the diff
[ ] Understand every changed file
[ ] Relevant tests passed
[ ] Lint/type/build checks passed where applicable
[ ] No secrets or generated junk were added
[ ] No public behavior changed unintentionally
[ ] Review pass found no unresolved high-priority issue
[ ] Final summary matches actual evidence
```

## CLI commands worth remembering

```text
codex                 start interactive CLI
codex resume          resume a saved chat/session
codex exec ...        non-interactive/repeatable run
codex --search        run with live web context when needed
codex cloud           work with cloud chats/tasks from CLI
codex mcp ...         configure/inspect MCP servers
codex completion      shell completion support

Inside interactive Codex:
/init
/status
/permissions
/model
/review
```

## MCP example

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
codex mcp list
```

## GitHub review examples

```text
@codex review
@codex review for issues in the database migration
@codex fix the P1 issue
@codex security review   # where the security-review workflow is available
```

---

# Part VIII - Glossary

**Agent** - A model operating in a loop where it can inspect context, use tools, make changes, and verify results rather than only output text.

**Codex Cloud** - Remote Codex task execution in configured isolated cloud environments.

**Codex CLI** - Terminal interface that lets Codex inspect/edit local repositories and run local tools.

**IDE extension** - Codex interface embedded in supported code editors, using open files/selections as convenient context.

**Code review** - Codex workflow that reviews a pull-request diff and posts findings in GitHub.

**AGENTS.md** - Persistent instruction file used to tell Codex stable repository/workflow rules.

**AGENTS.override.md** - Higher-precedence instruction file used in the current layered instruction system to override the normal file at a scope.

**MCP (Model Context Protocol)** - Protocol for connecting AI clients to external tools and information sources.

**Cloud environment** - Reproducible remote environment containing a repository plus setup, dependencies, variables, secrets, and network rules needed by a Codex task.

**Worktree** - A Git mechanism that allows a separate working directory tied to the same repository, useful for isolating concurrent changes.

**Reasoning effort** - A control that adjusts how much reasoning work the model spends before/while producing a result.

**Subagent** - A delegated agent thread handling a bounded part of a larger workflow, often in parallel with other subagents.

**Acceptance criteria** - Observable conditions that define when a task is complete.

**Verification** - Evidence such as tests, type checks, linting, builds, reproduction steps, or manual validation that supports the claim that a change works.

---

# Part IX - Recommended learning order after this playlist

Once you are comfortable with the 11 lessons, deepen skills in this order:

1. **Git/GitHub fluency.** Branches, commits, diffs, rebase/merge basics, pull requests.
2. **Testing discipline.** Agents become far more reliable when behavior is executable as tests.
3. **Repository instructions.** Improve `AGENTS.md` and team conventions.
4. **Cloud environment reproducibility.** Make setup and verification deterministic.
5. **MCP/tools.** Add external context only when it provides real value.
6. **Non-interactive automation.** Use `codex exec` and CI-style workflows for repeatable tasks.
7. **Parallelism/subagents.** Scale independent investigation while preserving review quality.
8. **Skills/plugins.** Package recurring workflows and connect supported team tools/data.
9. **Security review.** Add domain-specific review rules and threat-oriented checks for sensitive code.

The progression matters. If you jump to maximum parallel autonomy before learning diffs, tests, and task boundaries, you will generate code faster than you can trust it.

---

# Part X - Sources and freshness notes

This guide was prepared from the playlist you supplied, public course metadata/summaries, and current OpenAI Codex documentation checked on **September 20, 2026**.

## Playlist / course

- Net Ninja course page: https://www.netninja.dev/p/openai-codex-tutorial
- YouTube playlist supplied by the reader: https://www.youtube.com/watch?v=tIb_TzVNbDM&list=PL4cUxeGkcC9iDBeA8IyR1IE1kl4w5IDEG

The playlist is organized around these 11 lessons: Introduction & Setup; Running Cloud Tasks; Code Review; Codex CLI; CLI Commands & Resuming Sessions; AGENTS.md; IDE Extension; Context, Reasoning & TODOs; MCP Servers; Delegating Tasks to the Cloud; Running Tasks in Parallel.

## Current OpenAI Codex references

- Codex CLI: https://developers.openai.com/codex/cli
- Codex IDE extension: https://developers.openai.com/codex/ide
- Codex Cloud: https://developers.openai.com/codex/cloud
- GitHub review integration: https://learn.chatgpt.com/docs/third-party/github
- AGENTS.md guidance: https://learn.chatgpt.com/docs/agent-configuration/agents-md
- MCP guidance: https://developers.openai.com/codex/mcp
- Windows / WSL guidance: https://learn.chatgpt.com/docs/windows/wsl
- Codex documentation home: https://developers.openai.com/codex

Because Codex changes rapidly, treat exact button labels, model names, plan limits, and minor CLI flags as time-sensitive. The workflow principles in this guide are more durable: establish context, constrain scope, preserve reversibility, verify with tools/tests, review diffs, and increase autonomy only when the environment and task are well specified.

---

# Final one-page operating philosophy

If you remember nothing else from the course, remember this:

**Codex is most useful when you give it a well-defined engineering loop, not when you merely ask it to “code.”**

The loop is:

```text
Understand -> Plan -> Change -> Verify -> Review -> Integrate
```

Use local/IDE work for tight feedback. Use cloud tasks for independent longer work. Use `AGENTS.md` for stable rules. Use MCP when external tools or current documentation genuinely matter. Use pull-request review for an independent safety pass. Use parallel tasks only when the work is truly separable. Let Git and tests keep the process reversible and evidence-based.
