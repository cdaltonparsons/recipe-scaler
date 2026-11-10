---
description: Build one roadmap milestone with the agent team, pausing after each task so Charlie can commit
argument-hint: <milestone number or feature description>
---

You are the orchestrator. You coordinate a team of subagents to build this milestone from `docs/ROADMAP.md`:

$ARGUMENTS

If that is a milestone number, read its description from `docs/ROADMAP.md` and use it as the feature request.

You do not write application code or tests yourself. Your job is to hand work to the right agent, pass context between them, and keep score. Subagents cannot see each other's output, so always paste the relevant plan, task and feedback into each prompt.

Follow this process:

0. **Check the branch.** Run `git status -sb`. If I am on `main`, or the working tree has uncommitted changes, stop and tell me before doing anything else. Each milestone gets its own branch, and each task gets its own commit on that branch.
1. **Plan.** Use the `planner` subagent with the feature request. Show me the plan, then continue without waiting unless the planner flagged a decision that needs me.
2. **Build each task in order.** For each task in the plan:
   a. Use the `implementer` subagent with the task title, files and acceptance criteria.
   b. Use the `test-writer` subagent with the same task details plus the implementer's summary.
   c. If the test-writer found bugs, send them back to the `implementer`, then rerun the `test-writer`.
   d. Use the `reviewer` subagent with the task details.
   e. If the verdict is CHANGES REQUESTED, send only the "Must fix" items to the `implementer` (and the `test-writer` if tests are affected), then run the `reviewer` again. Allow at most 2 review rounds per task. If it still is not approved, stop and ask me.
   f. **Pause for my commit.** Once the reviewer approves, stop and wait for me. Tell me the task title, the files changed, the number of review rounds, and a suggested one-line commit message. Keep your own notes for the agent log, since I may take a while. When I say to continue, run `git status` first. If the tree is not clean, tell me and wait. Never start the next task on top of uncommitted changes, because the next reviewer would see them in its diff.
3. **Log the run.** Create `docs/agent-log/NNN-short-feature-name.md` (next number in sequence) with: the feature request, the plan, and for each task which agents ran, how many review rounds it took, and the most interesting thing the reviewer or test-writer caught. Keep it honest, including what went wrong.
4. **Stop before git.** Never commit, push or open a pull request. Finish with a short summary of what changed and which files I should review in my IDE. The agent log is the last commit on the branch, and after that I open one pull request for the whole milestone.
