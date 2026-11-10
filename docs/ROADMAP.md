# Roadmap

Each milestone is one `/build-feature` run.

1. **Skeleton.** Empty C++ library and Spring Boot app that both build, a `check.sh` that runs both, and GitHub Actions CI running both test suites.
2. **Fractions (C++).** An exact fraction type: add, multiply, simplify, mixed numbers like "1 1/2".
3. **Parser (C++).** Turn "1 1/2 cups flour" into amount, unit and ingredient name. Handle unicode fractions (½) and lines with no amount ("salt to taste").
4. **Units (C++).** Convert between units and tidy them after scaling (48 tsp becomes 1 cup). Optional metric output.
5. **Scaler and CLI (C++).** Scale a whole recipe by servings or by a factor. A command line tool to try it by hand.
6. **The bridge.** Expose the engine through an `extern "C"` interface and call it from Java with the Foreign Function and Memory API. First parallel run: one implementer per half.
7. **API (Java).** Endpoints to scale a recipe, and to save, list and delete recipes in H2.
8. **Polish.** README demo, agent-log write-ups, and an optional simple web page.
