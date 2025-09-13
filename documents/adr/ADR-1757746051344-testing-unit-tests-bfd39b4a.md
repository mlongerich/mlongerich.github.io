# Architectural Decision Record: Great — adding unit tests can improve quality and speed up releases. 1) Which parts of the codebase should have unit tests added or prioritized?
- Are there specific components, utilities, or features that you want to cover (e.g., core logic, data processing utilities, UI components, API integrations)?
- Who relies on these tests (developers, QA, product owners) and what are their needs? Any known high-risk or frequently changing areas we should focus on first?

2) What does success look like from a business perspective?
- What outcomes do you want to achieve with unit tests (e.g., faster bug detection, reduced post-release defects, improved release confidence, easier refactors)?
- How will we measure success? Are there targets like code coverage, defect leakage after releases, or time saved during testing/QA?
- Do you have any constraints or preferences (e.g., required test coverage percentage, integration with CI, budget or timeline considerations) that we should factor in?

If helpful, feel free to share any current pain points (e.g., flaky tests, slow test suite, gaps in test coverage) so I can tailor the questions and priorities.

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis recommends establishing a Jest-based unit testing baseline focused on core utilities and content logic, with incremental expansion to DOM-related tests if applicable. A GitHub Actions CI pipeline with test execution and coverage gating should be implemented to provide early feedback and release confidence. The plan includes phased deliverables, risk mitigation, and clear alignment with business goals around faster bug detection, reduced post-release defects, and maintainable code. Clarifying priorities with the BA using the listed questions will help tailor coverage targets and prioritize modules for initial tests.

## Architecture Decisions

### Adopt a lightweight JavaScript unit testing framework (Jest) for the repository's JS utilities and content-related logic.

**Rationale:** The repository appears to be a static GitHub Pages site with JavaScript utilities and content rendering helpers. Jest provides a mature, widely-adopted ecosystem, built-in assertion library, jsdom for DOM-like testing when needed, and simple CI integration. This minimizes friction for a small-to-medium codebase and supports snapshots if needed in the future.

**Impact:** Sets a standard testing approach, enables fast feedback in CI, and supports growth (e.g., DOM utilities, data processing helpers).

**Risks:** Initial setup overhead and potential mismatch with existing scripts if not aligned with package management; mitigate with a focused initial scope and clear conventions.

### Keep the codebase in plain JavaScript for MVP with an eye toward incremental TypeScript adoption in new modules.

**Rationale:** If the repo is small, a full TypeScript migration could slow down initial testing efforts. TypeScript can be introduced gradually for new code paths or critical modules, while new tests can leverage typings where available.

**Impact:** Lowers initial friction for writing tests; preserves speed to implement tests for existing logic; future migration can improve maintainability.

**Risks:** Potential type-related issues surface later; mitigate with gradual TS adoption in new modules and strong test coverage for critical paths.

### Prioritize unit tests for core utilities, data processing helpers, and content rendering logic; defer extensive UI component testing unless there are complex DOM interactions.

**Rationale:** These areas are most likely to contain pure logic, deterministic behavior, and business-critical transformations that benefit from automated checks. UI components, if any, can be tested with lightweight DOM tests or can be covered later after core coverage is in place.

**Impact:** Accelerates high-value coverage without overcomplicating the initial setup.

**Risks:** UI behavior may be under-tested; mitigate with targeted DOM tests and incremental expansion as needed.

### Integrate unit tests with GitHub Actions to run on pull requests and gate merges with a conservative coverage threshold.


**Rationale:** CI-driven feedback improves release confidence and catches regressions early. A threshold ensures meaningful coverage growth while avoiding blocking on minimal changes.

**Impact:** Improved quality control and faster feedback for developers and QA stakeholders.

**Risks:** Potential CI flakiness or slow PR checks; mitigate by stable test data, deterministic tests, and sensible thresholds.

## Implementation Plan

### Overview
Establish a minimal, scalable unit testing baseline for the repository, wire CI, and progressively expand coverage to core utilities and content processing. Use Jest as the primary framework with optional DOM testing support. Maintain lightweight, fast tests suitable for a static site build process.

### Critical Path
Phase 1 scaffold must complete before Phase 2; Phase 4 depends on stable Phase 1–3 outputs. CI gating is a gating item for PRs and impacts release confidence and velocity.

## Technology Recommendations

### Testing Framework
**Recommendation:** Jest (with jsdom for DOM-related tests)
**Rationale:** Broad ecosystem, good integration with CI, straightforward setup for a JS-based repo, supports coverage reports and snapshots if needed.

### DOM/UI Testing (if applicable)
**Recommendation:** Testing Library (e.g., @testing-library/dom) with Jest
**Rationale:** Encourages testing user-facing behavior rather than implementation details; integrates well with Jest

### Type Safety
**Recommendation:** Optional TypeScript adoption for new modules or critical utilities
**Rationale:** Improves long-term maintainability and test reliability; can be incremental

### CI/CD
**Recommendation:** GitHub Actions with a test-coverage workflow
**Rationale:** Seamless integration with GitHub Pages workflow; visible quality gates for PRs

## Technical Risks

### Insufficient testable code surface due to tightly coupled or side-effect-heavy utilities.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Refactor to extract pure functions; isolate side effects; introduce small adapters for testing; prioritize testable modules in Phase 1

### Test data/configuration drift leading to flaky tests.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Use deterministic fixtures; mock external data; avoid time-dependent tests; pin dependencies

### CI pipeline failures due to environment differences or secrets leakage.
**Probability:** Low
**Impact:** Medium
**Mitigation:** Use minimal CI environment; store no secrets in tests; mock network calls; cache dependencies

### Scope creep leading to over-engineering test suite for a static site.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Adhere to MVP-focused test plan; set clear coverage thresholds and progressive targets; review quarterly

## Performance Considerations

**Scalability:** Unit test suite should scale with code growth; keep tests fast and isolated to prevent long feedback loops.

**Performance Targets:** Aim for the initial test suite to complete within 1–2 minutes on CI for MVP; progressively shorten as the suite grows.

**Optimization Strategies:** Run tests in parallel (Jest default); use test file patterns to run only affected tests for PRs; cache dependencies in CI

## Security Considerations

**Threat Model:** Primary risk is leakage of sensitive data via test fixtures or CI logs; risk is low for a static site but still prudent.

**Security Controls:** Ensure test fixtures do not contain real secrets; mock external API calls; avoid network I/O in unit tests

**Compliance:** No explicit regulatory requirements for a GitHub Pages site; align with internal policy for code quality and testing

## Development Strategy

**Methodology:** Agile-inspired, with sprints focused on test scaffolding, core utilities, and CI integration; incremental delivery

**Testing Strategy:** Unit tests for pure/virtualized logic first; DOM tests where applicable; integrate coverage thresholds in CI; iterative growth

**Deployment Strategy:** For a static site, deployment remains via GitHub Pages; ensure CI gates tests on PRs before merge

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