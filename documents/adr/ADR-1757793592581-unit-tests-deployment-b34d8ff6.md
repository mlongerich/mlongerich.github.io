# Architectural Decision Record: I’ll gather requirements first. I’ll start with two clarifying questions:

1) What primary business outcomes do you want unit tests to support, and how will you measure success? (Examples: reduce regressions on the site, speed up PR reviews, improve contributor confidence, target a certain test coverage or bug rate, etc. ) Who will rely on these tests to make decisions (site maintainers, contributors, CI systems)?

2) What scope and priority should the tests cover? Which parts of the repository should be tested (e.g., JavaScript utilities, content rendering, build or deployment helpers), and which areas are top priority? Also, do you want these tests wired into CI (e.g., run on pull requests) or should they be runnable locally as well, or both?

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis proposes a pragmatic, incremental approach to introducing unit tests for the mlongerich.github.io site, starting with a lightweight Jest-based foundation focused on JavaScript utilities and build scripts. The plan emphasizes CI integration via GitHub Actions, a baseline coverage target, and a staged expansion path (with optional rendering tests via Playwright). The approach aligns with the existing ADR around unit tests while avoiding duplication and keeping the initial effort scoped and maintainable. Once BA requirements are clarified (via the questions for BA), we will tailor the test scope, coverage targets, and CI configuration accordingly and update ADR references to reflect concrete choices.

## Architecture Decisions

### Adopt a lightweight, JavaScript-focused unit testing strategy for the mlongerich.github.io site with GitHub Actions CI, leveraging Jest for unit tests of Node/JS utilities and optional Playwright for basic rendering checks; defer Ruby/Jekyll-specific testing to a later phase if needed.

**Rationale:** The repository appears to be a static site (GitHub Pages) likely built with a combination of HTML/JS/CSS. A JS-centric unit testing approach provides fast feedback for client-side utilities, build tooling, and small Node scripts without imposing the overhead of a full Ruby/Jekyll test stack. Jest is widely adopted, fast, and integrates cleanly with GitHub Actions. This decision aligns with the existing ADR focus on unit tests and provides a pragmatic, low-friction starting point. If later there is a need to test server-side Ruby/Jekyll behavior or full end-to-end rendering, a separate, scoped testing plan can be introduced (e.g., RSpec for Ruby or Playwright/Cypress for E2E).

**Impact:** Provides quick, maintainable test coverage for JS utilities and build scripts, reduces regressions in client-side behavior, and enables faster PR feedback. Keeps initial setup lean and CI-friendly for a GitHub Pages project.

**Risks:** If there are few or no JS utilities, test value may be limited; potential for flaky tests due to DOM or environment differences; PR CI time could increase if the suite grows too large too quickly. Mitigations include starting with a small, focused baseline and steadily expanding coverage; use deterministic tests and CI caching.

## Implementation Plan

### Overview
 phased plan to introduce a minimal yet scalable unit testing foundation for JS utilities and build scripts, integrate with CI, and progressively extend to rendering checks as needed.

### Critical Path
Discovery of JS utilities and Node scripts to test, baseline Jest setup, first 2–3 unit tests, and CI workflow creation. Ruby/Jekyll components can remain out of scope unless requested; if tested later, plan for a separate ADR addressing Ruby tooling (RSpec) and Jekyll build steps.

## Technology Recommendations

### Testing framework
**Recommendation:** Jest
**Rationale:** Lightweight, fast, widely adopted for JavaScript testing; good TypeScript support if TS is in use; easy to integrate with GitHub Actions.

### CI/CD
**Recommendation:** GitHub Actions
**Rationale:** Native to GitHub-hosted repos; easy to trigger on pull requests; straightforward caching and artifact workflows; aligns with GitHub Pages hosting.

### Optional rendering tests
**Recommendation:** Playwright
**Rationale:** Headless browser automation for basic rendering checks of critical pages; supports cross-browser testing and easy CI integration.

### Code quality
**Recommendation:** ESLint + Prettier
**Rationale:** Maintains consistent test and code hygiene; helps catch issues early and reduces debate in PRs.

## Technical Risks

### Limited return on investment if the repository has few JavaScript utilities or testable logic.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Start with obvious targets (npm scripts, small helper modules, utilities in assets/js). Regularly reassess utility surface and expand tests incrementally.

### Test flakiness due to DOM-dependent behavior or environment differences in CI.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Isolate tests from real DOM where possible; mock dependencies; use deterministic inputs and local fixtures; limit reliance on network I/O.

### Increased PR latency due to test suite growth in CI.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Set initial lightweight baseline; implement selective or parallel test execution; apply coverage gates incrementally.

### Difficulty aligning with existing ADRs or site build steps (e.g., Jekyll/Ruby components).
**Probability:** Low
**Impact:** Low
**Mitigation:** Document scope boundaries in ADRs; keep JS tests decoupled from Ruby-based build components; add a follow-up ADR if Ruby testing becomes necessary.

## Performance Considerations

**Scalability:** Aim for a fast unit test suite (sub-5 minutes in CI for baseline). Expand gradually as codebase grows (more tests should still remain fast).

**Performance Targets:** Initial target: suite completes within a few minutes; CI time under ~5–10 minutes per PR as tests grow.

**Optimization Strategies:** Use Jest's testRunner with parallelization,Cache node_modules in CI,Mock external dependencies and use local fixtures,Split large test suites into smaller, independent files

## Security Considerations

**Threat Model:** Tests run in CI should not expose secrets or rely on production data; avoid network access to untrusted endpoints; ensure test configuration cannot leak credentials.

**Security Controls:** Use repository-scoped tokens with restricted permissions only in CI secrets,Mock external services and network calls,Do not execute arbitrary code from tests; avoid eval in tests

**Compliance:** No PII or sensitive data should be used in tests; use synthetic fixtures

## Development Strategy

**Methodology:** Iterative, incremental adoption with clear gates and documentation.

**Testing Strategy:** Unit tests first for JS utilities and scripts; then optional integration/rendering tests; maintain a 70–80% baseline coverage target for the initial phase and increase gradually.

**Deployment Strategy:** No production deployment changes required; tests run in CI on PRs and can be extended to nightly builds if desired.

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