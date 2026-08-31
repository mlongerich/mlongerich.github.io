# Architectural Decision Record: I’ll start by gathering requirements to ensure the unit-test effort aligns with business goals. I won’t create issues yet; I’ll compile a clear requirements picture first. Two clarifying questions to begin:

1) What business outcomes are you hoping to achieve by adding unit tests? Examples might include reducing production bugs, speeding up releases, making refactors safer, improving customer satisfaction, or meeting quality/regulatory standards.

2) Who are the primary users or stakeholders that will rely on these tests, and what do they need from the testing effort? For example: developers needing fast feedback, QA needing stable interfaces, product owners wanting measurable quality signals. Do you have any target coverage or priority areas (e.g., critical features or core utilities) in mind?

If you have any constraints (timeline, CI requirements, or preferred testing framework) you’d like to surface, feel free to mention them as well.

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis proposes building on the existing ADR around unit tests by implementing a pragmatic JavaScript unit testing strategy for client-side code in mlongerich.github.io. The plan emphasizes a lightweight, CI-driven approach using Jest (or Vitest) with jsdom, targeted unit tests for critical utilities and DOM helpers, and GitHub Actions for automated test execution. The approach avoids duplicating server-side (Ruby/Jekyll) concerns at this stage, focusing on testability where it yields clear business value: faster feedback, reduced regressions, and improved maintainability. The phased plan outlines discovery, framework setup, core tests, CI integration, and quality gates, with measurable targets and risk mitigations. Collaboration with the BA will refine coverage targets, stakeholder needs, and any constraints to finalize the scope and cadence.

## Architecture Decisions

### Adopt a JavaScript unit testing and CI strategy for client-side code, building on ADR-1757635978848-unit-tests-e29b77c0.md to implement targeted unit tests and CI integration while excluding Ruby/Jekyll build layers from unit tests.

**Rationale:** This repository appears to be a static site (likely Jekyll-based) with client-side JavaScript. The existing ADR indicates a focus on unit testing to reduce regressions and improve maintainability. Given a small to mid-sized site, focusing unit tests on JavaScript utilities and DOM-interaction helpers provides measurable quality improvements with minimal maintenance overhead. Jest (or Vitest) with jsdom provides a familiar, fast, and lightweight testing environment for frontend code. CI integration via GitHub Actions ensures tests run on PRs and on mainline merges, delivering fast feedback to contributors and preserving production stability. This decision leverages the existing ADR while avoiding over-architecting the Ruby/Jekyll server-side build, which is outside the scope of unit testing and would require additional complexity if pursued.

**Impact:** Adds a maintainable test harness for client-side code, enabling automated regression checks with low incremental cost. Enables better onboarding for contributors and clearer quality signals. May require small ongoing maintenance for test suite as the site evolves.

**Risks:** Tests may become stale if the site’s JS APIs change frequently; potential CI runtime increase; added dependencies (Node ecosystem) risk if the project is kept minimal. Mitigations include targeted test scope, documentation, and CI caching.

## Implementation Plan

### Overview
 phased approach to establish a practical unit testing layer for client-side code with CI integration, aligned to business goals and the existing ADR.

### Critical Path
The critical path centers on Phase 2 framework setup (config and scripts) and Phase 4 CI integration, as these enable automated feedback and guardrails for Phase 3 test development.

## Technology Recommendations

### JavaScript Testing Framework
**Recommendation:** Jest (with jsdom)
**Rationale:** Wide ecosystem, good compatibility with a variety of JS patterns, strong community support, and built-in coverage reporting. Works well for small to medium codebases and fast feedback in CI.

### End-to-End (optional, later phase)
**Recommendation:** Playwright or Puppeteer for targeted E2E tests
**Rationale:** If there is a need to verify navigation and UI flows beyond unit tests, E2E tests provide end-user scenario validation.

### Code Quality & CI
**Recommendation:** eslint + prettier for linting/formatting; GitHub Actions for CI
**Rationale:** Ensures consistent code style and early defect detection; CI enforces quality gates.

### Code Coverage
**Recommendation:** nyc (Istanbul) integrated with Jest/Vitest
**Rationale:** Quantifies test coverage and helps set targets; integrates easily with CI reporting.

## Technical Risks

### Limited testability due to heavy reliance on Jekyll/Ruby rendering in the static site
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Scope unit tests to client-side JS utilities and DOM interactions; defer Ruby/Jekyll tests to a separate pipeline if needed

### Flaky tests caused by environment differences (jsdom vs real browser)
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Isolate tests to pure logic or DOM primitives; mock browser APIs; pin test environment to a stable Node version

### Additional CI runtime cost and maintenance overhead
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Cache dependencies, run tests selectively on PRs, and keep the test suite focused on critical paths

## Performance Considerations

**Scalability:** Frontend unit tests are generally lightweight; plan for incremental growth. Use test isolation to avoid shared state; parallelize test execution in CI.

**Performance Targets:** Run tests within a few minutes for the initial Phase 3 tests; maintain sub-5-minute CI runs as the suite grows.

**Optimization Strategies:** Use test data factories and mocks to keep tests fast,Enable test file parallelization in the chosen framework,Cache dependencies in CI and reuse node_modules between runs

## Security Considerations

**Threat Model:** Ensure test code cannot leak secrets or access production-like endpoints; avoid embedding sensitive data in test fixtures.

**Security Controls:** Use env marks and secrets in CI securely; do not commit credentials,Run npm audit during CI to catch vulnerable dependencies,Isolate test environment from production scaffolding

**Compliance:** If regulatory standards require, document testing scope and maintain traceability of tests to requirements

## Development Strategy

**Methodology:** Agile-inspired incremental delivery with CI-driven quality gates

**Testing Strategy:** Unit tests for client-side utilities and DOM helpers; optional E2E tests later if needed; aim for clear coverage targets and PR gates

**Deployment Strategy:** Static site on GitHub Pages; tests run in CI before merging to main; deployment remains unchanged unless tests reveal issues

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