# Recipe Scaler

A recipe scaler with a Java API and a C++ math engine. The app is real, but the main point of this repo is to practice multi-agent development with Claude Code subagents, including running agents in parallel. See `README.md` for the why.

## How work happens here

Milestones are built with the project's `/build-milestone` command (`.claude/commands/build-milestone.md`), which runs the subagents in `~/.claude/agents/` and pauses after each approved task for Charlie to commit:

- `planner` breaks a feature into small tasks. When a feature touches both halves, the first task is always the contract: the C++ header function and the matching Java interface.
- `implementer` writes application code for one task, in one half (`engine/` or `api/`)
- `test-writer` writes tests and never touches app code
- `reviewer` reviews the diff and returns APPROVE or CHANGES REQUESTED
- `debugger` and `researcher` are called in when something breaks or needs looking up

Once a contract task is approved, the `engine/` and `api/` tasks can run in parallel, one implementer per half.

The human (Charlie) reviews every change in their IDE and makes every commit. **No agent ever runs `git commit`, `git push`, or opens a pull request.**

Git workflow: one branch per roadmap milestone (for example `feature/fractions`), one commit per approved task, and one pull request per milestone into `main`. Agents assume the working tree is clean at the start of each task, so `git diff` shows only the current task.

Each feature run is written up in `docs/agent-log/`. The planned features are in `docs/ROADMAP.md`.

## Stack

- `engine/`: C++20 library built with CMake. Tests use GoogleTest, fetched by CMake. Formatting with clang-format, linting with clang-tidy.
- `engine/cli/`: a small command line tool that wraps the library, for trying it by hand.
- `api/`: Java 25 with Spring Boot, built with the Gradle wrapper. Tests use JUnit 5. Formatting with Spotless (google-java-format).
- Java calls the C++ library through the Foreign Function and Memory API (`java.lang.foreign`). Do not use JNI.
- The engine exposes a plain C interface (`extern "C"`) in `engine/include/` so Java can call it. C++ types stay inside the library.
- Data layer: Spring Data JPA with an embedded H2 database. Single user, no login.
- CI: GitHub Actions on `ubuntu-latest`. Development happens on macOS (Apple Silicon). The engine builds as `libengine.dylib` on macOS and `libengine.so` on Linux, so anything that loads it (`EngineBridge`, Gradle config) must handle both.

## Versions

These are newer than some agents' training data. Use exactly these, and do not downgrade to versions you remember.

- Java 25 (Temurin). Set the Gradle toolchain to 25.
- Spring Boot 4.1.2 (Spring Framework 7, Jakarta EE 11). Not 3.x.
- Gradle 9.9.0 through the wrapper (`gradle wrapper --gradle-version 9.9.0`). Use the Kotlin DSL (`build.gradle.kts`).
- C++20 and CMake 3.28 or newer (`cmake_minimum_required(VERSION 3.28)`). Charlie has CMake 4.4 installed.
- GoogleTest 1.18.0, fetched with `FetchContent`.
- Compiler: Apple clang on macOS, the default GCC or Clang in CI. clang-format and clang-tidy are LLVM 23.

## Commands

Run from the repo root.

- `./check.sh` runs everything below. It must pass before a task is done.
- C++ build: `cmake -S engine -B engine/build && cmake --build engine/build`
- C++ tests: `ctest --test-dir engine/build --output-on-failure`
- C++ format check: `clang-format --dry-run --Werror engine/src/*.cpp engine/include/*.h`
- Java build and tests: `cd api && ./gradlew build`
- Java tests only: `cd api && ./gradlew test`
- Java format check: `cd api && ./gradlew spotlessCheck`
- Run the API: `cd api && ./gradlew bootRun` (http://localhost:8080)

The Java build needs the engine built first, because the API loads the compiled library from `engine/build/`.

## Conventions

- Amounts are exact fractions everywhere. Never use floating point for ingredient amounts.
- C++: no raw `new`/`delete`. Use values, `std::vector`, `std::string` and smart pointers. Errors inside the library use exceptions, but nothing may throw across the `extern "C"` boundary. Return an error code instead.
- Java: keep FFM calls in one class (`EngineBridge`) so the rest of the API never touches `java.lang.foreign`.
- Validate anything that comes from the user before using it.
- C++ tests live in `engine/tests/`, Java tests in `api/src/test/java/`. Tests never hit the network.
- If a file passes about 200 lines, consider splitting it.
