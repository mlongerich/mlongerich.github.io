# Architectural Decision Record: I’ll ask a couple of clarifying questions to understand the business goals behind adding unit tests. 1) Which parts of the project should be covered by unit tests (e.g., JavaScript utilities, DOM interactions, Jekyll templates/plugins, third-party scripts), and are there any specific modules you want to prioritize?

2) How will we measure success for this unit testing effort from a business perspective (e.g., target test coverage %, reducing post-deploy bugs, ensuring critical paths are tested, or integrating tests into CI with visible results)? If you have a desired timeline or priority, please share that as well.

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

The repository (mlongerich.github.io) likely comprises a static site built with Jekyll, including JavaScript utilities, DOM interactions, Liquid templates, and potentially Ruby-based plugins. A comprehensive unit-testing strategy is recommended, covering JavaScript utilities with Jest, DOM interactions with a jsdom environment, and Ruby/Jekyll plugin logic with RSpec. Phase-backed implementation plan includes tooling setup, JS and Ruby test suites, and CI integration with Codecov reporting. Key risks include test flakiness and longer CI times, mitigated by deterministic fixtures, mocks, caching, and selective test execution. Business success metrics should align with predefined coverage targets and visible CI results, enabling better release confidence and reduced post-deploy issues. The BA should confirm coverage targets, priority modules, and any E2E testing requirements to finalize scope and roadmaps.

## Architecture Decisions

### Adopt a multi-layered test architecture covering JavaScript utilities, DOM interactions, and Jekyll/Ruby plugins with dedicated test suites per layer.

**Rationale:** A static site built with Jekyll and GitHub Pages typically comprises JavaScript utilities, DOM interactions, and Ruby-based plugins/filters. Testing each layer with appropriate tooling provides targeted feedback, prevents regressions across the stack, and aligns with CI workflows. This approach also accommodates testing of Liquid/template outputs indirectly via Jekyll builds.

**Impact:** Improved defect detection across all code surfaces, better maintainability, and clearer ownership of test suites. Increased CI time and maintenance overhead, mitigated by caching and selective test execution.

**Risks:** Increased CI duration, flaky tests in DOM rendering, and Ruby/Gem version drift. Mitigation includes deterministic fixtures, stable test environments, and explicit version pinning.

### Integrate testing with GitHub Actions and code coverage reporting (Codecov) across Node (JS) and Ruby (Jekyll/plugins) workflows.

**Rationale:** CI feedback is essential for business goals around quality and release confidence. Centralized reporting enables visible metrics for leadership and stakeholders, and gates PRs via status checks.

**Impact:** Automation of test runs, early detection of regressions, and measurable quality metrics. Requires setup and maintenance of workflows, secrets, and report publishing.

**Risks:** Misconfigured workflows could fail builds; coverage targets might deter contributors. Mitigation includes clear targets, incremental thresholds, and robust workflow tests.

## Implementation Plan

### Overview
Establish a structured test framework across JavaScript and Ruby components, wire tests into CI, and define coverage targets with phased delivery.

### Critical Path
Phase 1 scaffolding must be in place before Phase 2 JS tests; Phase 3 Ruby tests require Phase 1 Ruby tooling; CI workflows enable Phase 4 gating. Delays in CI configuration could stall feedback loops.

## Technology Recommendations

### JavaScript Testing
**Recommendation:** Jest with @testing-library/dom for unit tests; jsdom as the test environment.
**Rationale:** Widely adopted, fast feedback, strong ecosystem, good utilities for DOM assertions.

### Ruby & Jekyll Testing
**Recommendation:** RSpec for unit tests; Capybara for integration-like template rendering checks when needed.
**Rationale:** Solid Ruby testing ecosystem; suitable for plugin logic and rendering checks.

### Code Quality & Security
**Recommendation:** ESLint for JS linting; RuboCop for Ruby; Bundler Audit for dependency checks; npm audit for JS.
**Rationale:** Prevents common issues and enforces consistency across codebase.

### CI & Coverage
**Recommendation:** GitHub Actions with separate jobs for Node and Ruby; Codecov for coverage reporting.
**Rationale:** Integrated, transparent, and widely supported within GitHub ecosystem.

## Technical Risks

### Test flakiness due to DOM rendering timing and external script dependencies.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Use deterministic fixtures, mock external scripts, and stabilize environment; run tests in a jsdom environment for predictability.

### Ruby/Jekyll test environment constraints on GitHub Actions (availability of gems, Ruby versions).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Pin Ruby version, lock gem versions, use Bundler with exact resolutions, and vendor dependencies when feasible.

### Increased CI duration impacting developer velocity.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Cache dependencies, run selective tests on push, full suite on PRs; parallelize jobs; optimize test data.

### Insufficient test coverage or misaligned targets with business goals.
**Probability:** Low
**Impact:** High
**Mitigation:** Align coverage targets with business goals early; review plan with BA; continuously monitor coverage trends.

## Performance Considerations

**Scalability:** Static site tests should scale with repository growth; keep tests modular to parallelize across jobs.

**Performance Targets:** Aim for total test suite runtime under 20–30 minutes on CI with caching; JS unit tests under 5–10 minutes, Ruby tests under 5–15 minutes depending on plugin load.

**Optimization Strategies:** Mock external calls; use small, representative fixtures; run tests in parallel; cache dependencies (npm/yarn, Bundler).

## Security Considerations

**Threat Model:** Malicious test data or fixtures affecting CI; dependency vulnerabilities in JS and Ruby gems; potential leaks of secrets in logs.

**Security Controls:** Pin dependencies; run npm/audit and Bundler audit; secret scanning in CI; guard against leaking credentials in fixtures; sanitize test data.

**Compliance:** Not a regulated domain, but follow general data handling and vulnerability disclosure practices on public repos.

## Development Strategy

**Methodology:** Agile-like incremental delivery with GitHub Flow; feature branches, PRs, and gated merges based on test outcomes.

**Testing Strategy:** Unit tests per module; minimal integration tests where necessary; template rendering checks; avoid brittle end-to-end tests unless required.

**Deployment Strategy:** Tests run on PRs; automated linting; no changes to GitHub Pages publishing flow unless tests are green; documentation updated alongside code.

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