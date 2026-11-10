# Recipe Scaler

Paste in a recipe, choose a new number of servings, and get back the recipe with every amount scaled, kept as exact fractions, and converted to sensible units.

**The app is real, but it is not the point of this repo.** I built it to learn how to develop software with a team of AI agents instead of a single AI assistant. The code is written, tested and reviewed almost entirely by Claude Code subagents that I designed. My job is the part a tech lead does: defining the roles, breaking down the work, reviewing every diff, and deciding what gets merged.

## Why I built this

This is my second agent-built project. In the first one ([reading-tracker](https://github.com/cdaltonparsons/reading-tracker)) the agents worked one after another on a single TypeScript codebase. Here I wanted to push further in two ways.

First, two languages. The scaling math lives in a C++ library, and a Java API calls it. I wanted to see whether the same generic agents could work across both just by reading the project's instructions.

Second, parallel work. Because the two halves only meet at a small, agreed interface, two agents can build them at the same time. I wanted to learn how to set up that contract so parallel agents do not step on each other.

## How it is built

```
engine/   C++20 library: parsing, exact fractions, unit conversion, scaling
api/      Java 25 + Spring Boot REST API that calls the engine
```

Java calls the C++ library directly through the Foreign Function and Memory API, the modern replacement for JNI.

The agents are generic and live outside this repo. Each one reads this repo's `CLAUDE.md` to learn the stack and commands.

| Agent | Job | Can edit code? |
|-------|-----|----------------|
| `planner` | Breaks a feature into tasks, starting with the C++/Java contract | No |
| `implementer` | Writes application code for one task in one half | Yes, app code only |
| `test-writer` | Writes tests that prove the acceptance criteria | Yes, tests only |
| `reviewer` | Reviews the diff and approves or requests changes | No |
| `debugger` | Tracks down failures, especially across the language bridge | Yes, when asked |
| `researcher` | Looks things up, such as FFM API details | No |

No agent can commit. I review every change locally in my IDE and make every commit myself.

Every feature run is written up in [`docs/agent-log/`](docs/agent-log/), including the mistakes. The plan is in [`docs/ROADMAP.md`](docs/ROADMAP.md).

## What I learned

<!-- TODO(Charlie): fill this in as you go, in your own words. Good prompts:
     Did the generic agents handle C++ and Java as well as TypeScript?
     What went wrong the first time two implementers ran in parallel?
     What did you have to change in CLAUDE.md to make the agents behave? -->

_In progress. I add to this after each feature._

## Running it locally

Needs Xcode command line tools, plus `brew install openjdk@25 cmake llvm`.

```bash
./check.sh                         # build and test everything
cd api && ./gradlew bootRun        # http://localhost:8080
```

To build a feature with the agent team, open the project in Claude Code and run:

```
/build-milestone <milestone number from docs/ROADMAP.md, or describe the feature>
```
