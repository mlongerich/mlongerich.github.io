# Architectural Decision Record: I’d like to understand the business goals behind adding unit tests to mlongerich/mlongerich. To scope this well, here are two clarifying questions:

1) What business outcomes are you hoping to achieve with unit tests? For example, reducing regressions in the site, enabling safer refactoring, improving contributor confidence, or meeting CI/quality standards. How will you measure success (e.g., fewer broken builds, higher test coverage, faster release cycles)?

2) Which parts of the repository should be covered by unit tests? For instance, JavaScriptUtilities, frontend components, template rendering logic, or build/deployment scripts. Who will maintain and use these tests (your team, external contributors, CI system), and in what contexts (local development vs. automated pipelines)?

**Date:** 2025-09-11
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

The repository mlongerich/mlongerich.github.io can benefit from a structured unit testing strategy focused on JavaScript utilities and template rendering logic, using Jest with jsdom and optional Playwright E2E tests. This approach supports business outcomes such as reducing regressions, enabling safer refactoring, and increasing contributor confidence. The plan includes phased implementation, CI integration (GitHub Actions), clear testing conventions, and measurable success criteria (test coverage, PR pass rates, and faster feedback). Practical implementation should start with foundational framework setup (Phase 1), followed by rendering logic unit tests (Phase 2), and CI/build script coverage (Phase 3), with QA and documentation in Phase 4. Questions for the BA focus on scope, governance, coverage thresholds, and maintenance responsibility to tailor the plan to organizational priorities.

## Architecture Decisions

### Introduce a JS-focused unit testing layer using Jest with jsdom for DOM-related tests, and lightweight end-to-end tests (optional) with Playwright for critical rendering paths.

**Rationale:** mlongerich.github.io is a static GitHub Pages site likely composed of vanilla JS, HTML, and CSS. A JS-centric unit test approach enables validating pure functions, utilities, and any template rendering logic in isolation without requiring a full browser. jsdom provides a deterministic DOM environment for unit tests. Optional E2E tests cover real rendering in a headless browser to catch integration issues that unit tests may miss. This combination improves safety of refactors and confidence for contributors while keeping maintenance manageable for a static site.

**Impact:** Establishes a scalable testing foundation that can grow with the site. Enables CI to fail PRs that introduce regressions. Lower risk when refactoring utilities or templates.

**Risks:** Initial setup overhead and potential test flakiness if environment differences arise. Mitigation: start with deterministic unit tests, mock network calls, and gradually add E2E tests with stable selectors.

### Decouple and extract rendering logic into pure functions where possible to maximize testability.

**Rationale:** Pure functions that render strings or DOM fragments are straightforward to unit test and provide a clear boundary between logic and side effects (DOM manipulation, I/O). This improves maintainability and testability of template rendering logic.

**Impact:** Eases testability and reduces coupled maintenance risk. Improves clarity of responsibilities across code.

**Risks:** Refactoring work to extract pure functions may be non-trivial if code is not structured for testability. Mitigation: incrementally extract and guard changes with tests.

### Use GitHub Actions for CI/CD with test gates (unit tests run on PRs, optional E2E tests on specific triggers).

**Rationale:** GitHub Pages sites are often hosted via GitHub, making Actions a natural fit for CI. Running unit tests on PRs provides fast feedback, while optional E2E tests can be introduced later to validate rendering in a real browser environment.

**Impact:** Automates quality checks, reduces regression risk, and provides visibility into test outcomes for contributors.

**Risks:** CI configuration drift if environment assumptions change. Mitigation: keep dependencies and Node version pinned; document setup.

## Implementation Plan

### Overview
 phased plan to introduce unit tests for JS utilities and rendering logic, establish CI gates, and grow test coverage with maintainable processes.

### Critical Path
Phase 1 prerequisites (framework setup, tests for utilities) must complete before Phase 2 tests; Phase 3 relies on Phase 2 testable modules; CI gating is enabled in Phase 1 and matures through Phase 4.

## Technology Recommendations

### Unit testing framework
**Recommendation:** Jest (with jsdom environment)
**Rationale:** Widely adopted in JS ecosystems, simple to configure for static sites, good compatibility with ES modules, and easy to integrate with TypeScript or JSDoc if needed.

### DOM testing utilities
**Recommendation:** @testing-library/dom
**Rationale:** Encourages testing DOM interactions in a user-centric way, handles queries robustly, integrates well with Jest.

### End-to-end testing (optional extension)
**Recommendation:** Playwright
**Rationale:** Reliable cross-browser E2E testing for critical rendering paths; good for CI; supports headless mode.

### Linting & code quality
**Recommendation:** ESLint + Prettier
**Rationale:** Enforces consistent code style and catches potential issues early.

### Type safety (optional but recommended)
**Recommendation:** TypeScript (gradual adoption) or JSDoc type annotations
**Rationale:** Improves maintainability and helps catch issues at compile/test time.

### CI/CD
**Recommendation:** GitHub Actions
**Rationale:** Native integration with GH repositories, convenient for GitHub Pages workflows, easy caching and artifact handling.

## Technical Risks

### Flaky tests due to DOM environment differences or network dependencies.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Use deterministic mocks, isolate tests from real network calls, pin environments, and favor unit tests over integration tests where possible.

### Maintenance burden from adding tests to a static site with evolving content.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Constrain tests to pure utilities and rendering logic; avoid test coupling to dynamic content; maintain a lightweight test scope.

### Incompatibility with GitHub Pages deployment workflows or Node versions.
**Probability:** Low
**Impact:** Medium
**Mitigation:** Pin Node version in CI, use actions/setup-node with explicit version, and keep dependencies aligned with GH Actions runners.

### Overhead delaying PR feedback if tests are slow.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Cache dependencies, run only targeted test subsets for small PRs, parallelize tests, and optimize test suites to minimize runtime.

## Performance Considerations

**Scalability:** Unit tests scale well as codebase grows; keep tests focused on small units. E2E tests can be added later if needed.

**Performance Targets:** Aim for total unit test runtime under 2–5 minutes on CI for PRs; keep local dev test runs fast.

**Optimization Strategies:** Cache npm dependencies between runs in GitHub Actions.,Run targeted test subsets for small PRs; run full suite on main merges.,Use lightweight mocks instead of real network requests.

## Security Considerations

**Threat Model:** Static site assets could be compromised if test harness fetches remote assets; tests should not depend on external endpoints.,DOM-based XSS risk if tests render untrusted content; sanitize or mock inputs.

**Security Controls:** Mock network calls; avoid real endpoints in tests.,Run tests in isolated environment; avoid leaking secrets into logs.,Use content security policies in the deployed site; tests should not bypass CSP in production.

**Compliance:** No PII in tests; avoid using real user data.

## Development Strategy

**Methodology:** Incremental adoption with trunk-based development and PR-based code reviews; feature branches for larger tests additions if needed.

**Testing Strategy:** Unit tests for utilities and rendering logic first; progressively cover more of the codebase. Add E2E tests only for high-value pages after unit test stability.

**Deployment Strategy:** Deploy GitHub Pages via standard workflow; ensure CI runs tests before deployment where applicable. Tests do not block static site deployment unless configured.

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