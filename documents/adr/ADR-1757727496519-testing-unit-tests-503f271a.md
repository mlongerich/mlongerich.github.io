# Architectural Decision Record: 1) What are the top outcomes you want to achieve with adding unit tests for this project? (e.g., prevent regressions before release, increase developer confidence when refactoring, meet stakeholder quality expectations, reduce manual testing time)

2) Who will rely on or use the results of these unit tests, and how will they use them? (e.g., developers for code changes and quick feedback, QA for regression checks, product/management for quality demonstrations)

If it helps, you can describe any constraints or targets you have in mind (e.g., a target release window, minimum coverage, or preferred ways test results should be surfaced).

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This plan proposes introducing a focused unit test strategy for the mlongerich.github.io repository, treating it as a static site with client-side JS utilities. The approach uses Jest with jsdom, integrates tests into GitHub Actions, and surfaces test results as PR checks with coverage reporting. The initial scope targets core JS utilities and DOM helpers, with incremental expansion to broader site interactions. Risks include potential lack of existing Node tooling, test flakiness, and CI build time impact, mitigated by a phased implementation, caching, and discipline in test design. The plan aligns with business goals to prevent regressions, increase developer confidence, and demonstrate quality to stakeholders, while enabling developers and QA to rely on automated tests for quick feedback.

## Architecture Decisions

### Introduce a lightweight JavaScript unit test framework (Jest with jsdom) and place tests under a dedicated tests/ directory, decoupled from the static site build.

**Rationale:** The repository is a GitHub Pages static site (likely HTML/CSS/JS). Unit tests should target pure JS utilities and DOM-related helpers without impacting the Jekyll/Ruby-based site build. Jest provides a robust, widely adopted JS testing environment with jsdom for DOM-related tests, built-in coverage, and easy CI integration. Keeping tests separate from the site build avoids coupling the test pipeline with the static site generation process.

**Impact:** Adds devDependencies (Jest, possibly Babel or ts-jest if needed) and test configuration. Test results will be surfaced in CI (pull request checks and coverage reports). Minimal impact on the existing site build; tests run in a separate node-based runner.

**Risks:** Test runner and environment drift from the site’s deployment environment.,Flaky tests due to DOM timing or environment differences between local and CI.,Maintenance overhead if the test suite expands rapidly without clear scope.

## Implementation Plan

### Overview
Plan to establish a maintainable unit test setup for the static site, surface test results in PRs, and iteratively expand coverage to core JS utilities and client-side behavior.

### Critical Path
Key dependencies are: presence of a Node-compatible environment (Node.js), a package.json to install dev dependencies, and a GitHub Actions runner with network access to install npm packages. The CI workflow must be approved and merged to enforce PR checks. DOM tests rely on jsdom environment in Jest.

## Technology Recommendations

### Unit testing framework
**Recommendation:** Jest with jsdom environment for DOM-related tests
**Rationale:** Widely adopted, ships with built-in coverage, easy configuration, and excellent ESM/CommonJS support for JS utilities and DOM mocks.

### Code quality & linting
**Recommendation:** ESLint + Prettier
**Rationale:** Ensures consistent code style and catches potential issues early. Can be run in CI and locally.

### Coverage & reporting
**Recommendation:** Jest coverage with nyc (or built-in Jest coverage)
**Rationale:** Clear visibility into test coverage; badge in README and PR checks.

### CI/CD & deployment
**Recommendation:** GitHub Actions
**Rationale:** Seamless integration with GitHub-hosted pages and PR gates; supports caching for faster builds.

### Security & dependencies
**Recommendation:** npm audit / Snyk (optional in CI)
**Rationale:** Automates vulnerability checks for dev dependencies.

## Technical Risks

### Repo lacks package.json or Node-based tooling, making Jest setup non-trivial.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Create a minimal package.json with scripts for test, lint, and audit. Use npm ci in CI to ensure reproducible installs.

### Tests become flaky due to DOM timing or environment differences between local and CI.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Stabilize tests with explicit awaits, deterministic mocks, and a consistent jsdom configuration. Pin versions of test libraries.

### Test suite grows and slows down the CI cycle, delaying PRs.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Adopt selective test execution via test path filtering, parallelize tests, and set sensible coverage gates. Use caching for node_modules in CI.

### Security vulnerabilities in test/dev dependencies.
**Probability:** Low
**Impact:** Medium
**Mitigation:** Run npm audit in CI, lockfile integrity, and routinely update dependencies; use Snyk or similar if adopted.

## Performance Considerations

**Scalability:** Unit tests scale with project; ensure test architecture remains modular to prevent exponential test growth as new features are added.

**Performance Targets:** Local test suite should complete within 1–5 minutes; CI should complete within 8–12 minutes for a typical PR, with longer runs for larger changes.

**Optimization Strategies:** Selective test execution via path-based filtering (e.g., tests related to changed modules).,Mock heavy APIs and avoid real network calls in unit tests.,Enable parallel test execution and caching in CI.

## Security Considerations

**Threat Model:** Dependency supply chain risks in test tooling and dev dependencies.,Potential exposure of test data if tests access real endpoints,False sense of security if tests don’t cover critical paths

**Security Controls:** Lock dependency versions with package-lock.json or yarn.lock,Run npm audit / Snyk in CI,Do not fetch external resources in unit tests; mock network calls

**Compliance:** No special regulatory constraints identified for a static site; ensure data used in tests is non-sensitive or mock data

## Development Strategy

**Methodology:** Iterative, test-driven-like approach where feasible. Start small with core utilities and expand coverage incrementally.

**Testing Strategy:** Prioritize unit tests for deterministic, pure functions; gradually add tests for DOM-related helpers; avoid heavy end-to-end tests in this phase.

**Deployment Strategy:** Static site deployment remains via GitHub Pages; testing pipeline runs on PRs and does not block publishing unless tests fail.

## Alternatives Considered

Alternative approaches not documented

## Decision

This architectural approach is recommended based on the technical analysis above.

## Consequences

### Positive
- Technical implementation aligned with business requirements
- Scalable and maintainable solution

### Negative
- Technical complexity and implementation effort
- Potential technical debt if not properly maintained

---