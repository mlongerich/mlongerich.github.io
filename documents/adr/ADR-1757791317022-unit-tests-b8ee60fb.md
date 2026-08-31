# Architectural Decision Record: 1) Which parts of the repository should be covered by unit tests? (e.g., specific modules, utilities, data models, business logic, UI components). Are there any business-critical areas that must be tested?

2) What does success look like for adding unit tests? For example, target test coverage, key scenarios to verify, integration with CI, and any acceptance criteria or maintenance expectations.

If you’d like, I can adjust these questions to focus on a particular feature you have in mind.

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This plan builds on the existing ADR around unit tests to establish a pragmatic, phased approach for introducing unit testing to mlongerich.github.io. The recommended path prioritizes JS utility tests and CI integration, with optional UI/component tests if a front-end framework is present. The plan mitigates risk by starting with a Phase 1 assessment and a minimal baseline, then expanding test coverage and automation via Phase 2–Phase 4. Clarification from the BA on the exact tech stack and business-critical areas will refine tooling choices and phase timelines. Ensure alignment with ADRs and maintain a lightweight CI gate that respects the GitHub Pages deployment workflow.

## Architecture Decisions

### Adopt a scoped unit testing strategy for JavaScript utilities and, where applicable, UI components, with CI integration via GitHub Actions. Align with existing ADRs (e.g., ADR-1757635978848-unit-tests-e29b77c0.md) to formalize testing goals and impact on maintainability.

**Rationale:** The repository appears to be a GitHub Pages project (mlongerich.github.io) with an existing ADR focused on unit testing. For static sites and front-end components, lightweight, fast feedback unit tests on JS utilities and UI logic provide the best ROI. A CI pipeline ensures code quality and guards against regressions in critical rendering paths. This approach avoids over-engineering for a static site while establishing a solid testing foundation that can scale if a more dynamic stack is adopted later.

**Impact:** Clear testing targets, faster feedback loop on changes, and a maintainable baseline for future feature work. Sets the stage for integrating with CI and reporting coverage.

**Risks:** If the repo is primarily a static site with minimal JS, over-engineering tests could slow down initial progress. Mitigation: scope tests to critical utilities and any public APIs (data loading, URL helpers, templating helpers) first; iterate after baseline is stable.

## Implementation Plan

### Overview
Establish a minimal, scalable unit testing foundation aligned with the ADRs, identify business-critical areas to cover, implement tests for core utilities, and integrate with CI. Prioritize non-invasive changes to avoid disrupting the existing build and deployment flow for GitHub Pages.

### Critical Path
Decision on the exact tech stack (Jekyll/Ruby vs. JS framework) drives test tooling choice; CI configuration depends on the stack; baseline utilities tests are prerequisites for CI gates.

## Technology Recommendations

### Testing framework
**Recommendation:** Vitest (preferred) or Jest for JavaScript utilities; if React components exist, pair with React Testing Library.
**Rationale:** Vitest is fast, works well with Vite/TS, supports ESM, and integrates cleanly with modern front-end setups. React Testing Library emphasizes testing behavior over implementation, improving maintainability.

### CI/CD
**Recommendation:** GitHub Actions with separate workflows for tests and deployment; matrix strategy for Node and Ruby environments if both stacks exist.
**Rationale:** GH Actions is native to the repo hosting, easy to maintain, and provides robust integration with PR checks and artifacts (coverage reports).

### Linting & type safety
**Recommendation:** ESLint + Prettier; TypeScript (optional) for strong typing if utilities or scripts benefit from it.
**Rationale:** Consistent code style, early error catching, and better maintainability for JavaScript utilities.

### Test data & utilities
**Recommendation:** Create a shared test-utils module for common mocks and fixtures; maintain deterministic test data.
**Rationale:** Avoids duplication and speeds up test authoring for multiple tests.

## Technical Risks

### Unclear or mixed tech stack (Jekyll/Ruby and client-side JS) complicates test tool selection and CI setup.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Perform Phase 1 assessment to lock in a single, coherent testing approach; if multiple stacks exist, scope JS tests to client-side code and exclude server-side build steps from unit tests.

### Limited business-critical code to test (low ROI for heavy testing effort).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Prioritize testing for utilities and data processing with clear success criteria; defer UI tests until UI framework is confirmed.

### CI/CD workflow conflicts with GitHub Pages deployment (e.g., build steps clashing with GH Pages cache).
**Probability:** Low–Medium
**Impact:** Medium
**Mitigation:** Design CI to run tests before deployment steps; keep GH Pages deployment isolated (no-nonsense publish step).

### Test flakiness due to environment differences (local vs. CI).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Lock dependencies, pin Node versions, and use deterministic test data; add retry logic sparingly in CI if necessary.

## Performance Considerations

**Scalability:** Static site performance remains dominated by asset size; tests themselves are lightweight. Ensure bundle sizes remain small and scripts are lazy-loaded where possible.

**Performance Targets:** Aim for near-instant test startup times; maintain CI times under a few minutes for baseline tests.

**Optimization Strategies:** Only run unit tests for changed modules with selective testing if tooling supports it,Cache dependencies in CI and reuse artifacts between runs

## Security Considerations

**Threat Model:** Code quality and dependency risks in test tooling; potential exposure of test data in artifacts.

**Security Controls:** Pin dependency versions and use lockfiles (package-lock.json / yarn.lock).,Regularly audit dependencies for known vulnerabilities (npm audit / GitHub advisories).,Avoid executing untrusted code in tests; mock external calls.

**Compliance:** Ensure tests do not expose sensitive data in fixtures; comply with project privacy guidelines.

## Development Strategy

**Methodology:** Agile-like incremental improvements with PR-based validation; emphasize test-first or test-as-you-go within feature work.

**Testing Strategy:** [object Object]

**Deployment Strategy:** Maintain current GitHub Pages deployment flow; ensure tests run before deployment where feasible. Use PR checks to gate changes.

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