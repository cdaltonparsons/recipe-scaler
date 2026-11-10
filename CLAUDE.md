# Recipe Scaler

A recipe scaler with a Java API and a C++ math engine. The app is real, but the main point of this repo is to practice multi-agent development with Claude Code subagents, including running agents in parallel. See `README.md` for the why.

## How work happens here

Features are built with the `/build-feature` command, which runs the subagents in `~/.claude/agents/`:

- `planner` breaks a feature into small tasks. When a feature touches both halves, the first task is always the contract: the C++ header function and the matching Java interface.
- `implementer` writes application code for one task, in one half (`engine/` or `api/`)
- `test-writer` writes tests and never touches app code
- `reviewer` reviews the diff and returns APPROVE or CHANGES REQUESTED
- `debugger` and `researcher` are called in when something breaks or needs looking up

Once a contract task is approved, the `engine/` and `api/` tasks can run in parallel, one implementer per half.

The human (Charlie) reviews every change in their IDE and makes every commit. **No agent ever runs `git commit`, `git push`, or opens a pull request.**

Each feature run is written up in `docs/agent-log/`. The planned features are in `docs/ROADMAP.md`.

## Stack

- `engine/`: C++20 library built with CMake. Tests use GoogleTest, fetched by CMake. Formatting with clang-format, linting with clang-tidy.
- `engine/cli/`: a small command line tool that wraps the library, for trying it by hand.
- `api/`: Java 25 with Spring Boot, built with the Gradle wrapper. Tests use JUnit 5. Formatting with Spotless (google-java-format).
- Java calls the C++ library through the Foreign Function and Memory API (`java.lang.foreign`). Do not use JNI.
- The engine exposes a plain C interface (`extern "C"`) in `engine/include/` so Java can call it. C++ types stay inside the library.
- Data layer: Spring Data JPA with an embedded H2 database. Single user, no login.

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
