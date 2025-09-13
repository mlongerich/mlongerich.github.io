# Architectural Decision Record: 1) What should be covered by the unit tests?
- Which parts of the site or project should have unit tests (e.g., JavaScript utilities, functions, data processing, API call helpers, React/Vue components if applicable)?
- Are you looking to test only core logic, or also UI interactions, edge cases, and error handling?

2) What does success look like from a business perspective?
- Do you have a target test coverage percentage, a required set of test cases, or specific quality metrics (e.g., catching X% of regressions before release, reducing post-deploy bugs)?
- How will testing be integrated into your workflow (CI/CD, pull request gates, local development standards)? What is the desired timeline and priority for this work?

Optional but helpful (if you have it ready):
- Who are the primary users of these tests (developers, QA, product managers) and what are their main needs from having unit tests?
- Any constraints (time, tooling, hosting, or framework preferences) I should be aware of?

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis proposes a pragmatic, ADR-aligned approach to introducing unit tests for the mlongerich.github.io site by adopting Vitest + Vite, focusing on core JavaScript utilities and DOM-related logic. It outlines a phased implementation plan, CI integration, and governance to ensure maintainability and alignment with the existing ADR focusing on test-driven outcomes (e.g., reducing regressions and enabling safer refactors). The plan prioritizes lightweight tooling, minimal production impact, and a clear path for extending coverage over time, with explicit risk mitigations and performance/security considerations. The recommended path should be refined in collaboration with the BA to align with business priorities and any constraints.

## Architecture Decisions

### Adopt a lightweight JavaScript unit testing strategy using Vitest with Vite for the mlongerich.github.io repository, focusing on testing core JS utilities and any DOM-related logic, while ensuring tests run in CI only and are not bundled into the production static site.

**Rationale:** This aligns with the existing ADR focused on unit tests and targets a fast, ESModule-friendly testing stack that works well with static sites. Vitest provides a fast dev/CI loop, jsdom for DOM testing, and tight integration with Vite if a bundling step is present in the repo. Since GitHub Pages serves static content, tests should live in CI, not in the production build. This approach minimizes overhead while maximizing test feedback and maintainability.

**Impact:** Introduce a test infrastructure under tests/ with npm scripts (test, test:ci). Add a minimal Vite config and Vitest setup. Tests will be executed in CI, not during site generation. Requires minor code adjustments to export testable functions where needed.

**Risks:** Initial setup may require adjusting how utilities are exported for testing; potential test fragmentation if DOM-dependent code tightly coupled to HTML structure. Mitigation: isolate pure logic first, then add DOM tests using DOM Testing Library; ensure testable modules have clean interfaces.

## Implementation Plan

### Overview
Establish a CI-driven unit testing workflow using Vitest + Vite, add testing scaffolding, identify testable components, and integrate with PR gates. Create governance for ongoing test maintenance and coverage tracking aligned with business goals.

### Critical Path
Key dependencies are Phase 0 (tooling) and Phase 2 (CI gating). If CI workflow is blocked or dependencies fail to install, testing cannot be validated in PRs. Ensure Node version compatibility and correct export/import patterns in utilities to enable testing.

## Technology Recommendations

### Testing framework
**Recommendation:** Vitest with Vite
**Rationale:** Fast, ESModule-friendly, works well with small static sites, provides good jsdom support for DOM tests, and easy TypeScript support if needed

### DOM & UI testing
**Recommendation:** DOM Testing Library (with @testing-library/dom) or @testing-library/react if React is used
**Rationale:** Encourages testable DOM interactions with stable queries; works well with Vitest

### Code quality
**Recommendation:** ESLint + Prettier; optional TypeScript for future-proofing
**Rationale:** Maintains consistency in test and production code; reduces bugs

### CI/CD
**Recommendation:** GitHub Actions
**Rationale:** Native to the hosting platform, straightforward to gate PRs on test results

### Test coverage (optional)
**Recommendation:** Vitest coverage reports (text/LCOV) if desired
**Rationale:** Gives visibility into test coverage; can guide test expansion

## Technical Risks

### DOM-dependent code without clear testable boundaries may be hard to unit test
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Start with pure/isolated utilities; gradually introduce DOM tests using DOM Testing Library; refactor code to expose testable interfaces

### Test framework learning curve and maintenance overhead
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Adopt a narrow, pragmatic test scope; provide starter templates and examples; enforce CI to prevent regression of tests

### Tests inadvertently bundled with production build or misconfigured CI
**Probability:** Low
**Impact:** Medium
**Mitigation:** Ensure tests run in isolation; CI config only runs tests, not dev server in production pipeline; ensure Vite config is test-focused

### Dependency drift and security vulnerabilities in test tooling
**Probability:** Low
**Impact:** Medium
**Mitigation:** Pin versions, enable npm audit in CI, and run automated security checks on PRs

## Performance Considerations

**Scalability:** Tests should be fast and isolated; run in parallel in CI; avoid long-running tests

**Performance Targets:** Aim for total test runtime under a few minutes in CI for typical PRs; keep individual test files small

**Optimization Strategies:** Use jsdom environment; mock network requests; limit external dependencies; parallelize test execution

## Security Considerations

**Threat Model:** Tests should not rely on or expose secrets; ensure test data is inert; avoid hitting real endpoints

**Security Controls:** Pin test tooling versions; run npm audit in CI; sandbox test environments

**Compliance:** No user data handling in tests; ensure code follows project security guidelines

## Development Strategy

**Methodology:** Iterative, PR-driven with CI gates; incremental test coverage expansion

**Testing Strategy:** Prioritize core utilities first, then gradually cover edge cases and UI interactions

**Deployment Strategy:** No production deployment changes; tests run in CI and do not affect site hosting

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