# Architectural Decision Record: I’d like to gather some clarifications to define the requirements for adding unit tests. io), and what business goals should unit tests support? For example: reliability for readers, faster and safer releases, easier maintenance for contributors, compliance, or something else?

2) What specific outcomes do you want from introducing unit tests? Examples include protecting core features from regressions, improving test coverage metrics, reducing bug rates, speeding up CI feedback, or making it easier for new contributors to understand the codebase. Also, which parts of the repository should be tested (e.g., JavaScript logic, build scripts, data/assets, or something else) and are there any constraints (preferred CI platform, language/framework, etc.)?

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

The repository (mlongerich.github.io) is a GitHub Pages site likely containing static assets and possibly small JavaScript tooling. To enable robust unit testing without overengineering, propose introducing a lightweight JS unit testing setup (Jest with jsdom), integrated via GitHub Actions. Focus initial tests on JavaScript utilities and build scripts, ensure code quality with ESLint/Prettier, and gate PRs with consistent test results and basic coverage signals. Expand testing scope iteratively as business needs clarify which components (or Ruby/Jekyll components) warrant test coverage. The plan emphasizes maintainability, contributor onboarding, and safe, automated release practices while mitigating risks like flaky tests and increased CI times.

## Architecture Decisions

### Adopt a lightweight JavaScript unit testing framework (Jest) with a jsdom environment to cover JS utilities and Node-based build scripts.

**Rationale:** The repository is a static GitHub Pages site likely containing HTML/JS/CSS with potential small Node-based tooling. Jest provides a minimal setup, broad ecosystem, and jsdom enables DOM-related tests without a real browser. This supports reliable, fast feedback on core logic and build scripts while remaining approachable for contributors.

**Impact:** Establishes a reusable, maintainable test harness aligned with standard JS tooling; lowers barrier for contributors to add tests; enables CI integration.

**Risks:** Overkill if there is no meaningful JavaScript logic to test; potential initial setup overhead; need to ensure test environment mirrors production where DOM is involved.

### Integrate tests into GitHub Actions for PRs and main branch validation; treat tests as a required status check.

**Rationale:** GitHub Actions aligns with the hosting platform, provides straightforward caching and parallelism, and ensures regressions are caught before merging. This supports faster, safer releases and predictable contributor experiences.

**Impact:** Automates test execution on PRs, improves quality gates, and shortens feedback cycles.

**Risks:** Increased CI time if the test suite grows; potential flakiness affecting PR status.

### Enforce code quality gates via ESLint and Prettier; optionally add a pre-commit hook and a minimal code style guide.

**Rationale:** Maintains consistency across contributions, reduces bikeshedding in PRs, and lowers maintenance cost.

**Impact:** Improved maintainability and readability; easier onboarding for new contributors.

**Risks:** Overhead for contributors unfamiliar with tooling; mitigations include CI feedback and clear docs.

### Target test scope to JavaScript logic and Node-based build scripts; treat Ruby/Jekyll components (if any) as optional test targets unless directly executable in Node.

**Rationale:** GitHub Pages sites often rely on static assets. Focus testing where it yields measurable ROI (logic, parsing, config readers, and build tasks). Ruby/Jekyll tests can be added if/when Ruby-based tooling is part of the workflow.

**Impact:** Prevents scope creep; ensures testing effort aligns with code and tooling with executable logic.

**Risks:** If critical functionality exists in Ruby/Jekyll or in HTML rendering logic not covered by JS tests, gaps may appear.

## Implementation Plan

### Overview
Establish a minimal, robust unit testing foundation for JS utilities and build scripts, integrate into CI, and iteratively expand coverage with contributor-friendly guidelines.

### Critical Path
Key dependencies are: defined test framework choice, Node.js version stability, initial test files, and a functional CI workflow. Delays in any phase may push overall schedule by 1–2 weeks.

## Technology Recommendations

### Testing framework
**Recommendation:** Jest (with jsdom environment) for JavaScript unit tests
**Rationale:** Mature ecosystem, good TypeScript support (if used), simple setup, strong community, works well for both utilities and Node scripts.

### DOM & utilities testing
**Recommendation:** jsdom (built into Jest environment) for DOM-related tests
**Rationale:** Enables DOM-like testing without a real browser; aligns with unit test goals.

### CI & workflow
**Recommendation:** GitHub Actions
**Rationale:** Native to GitHub Pages projects, simple caching, parallelism, and easy PR gating.

### Code quality
**Recommendation:** ESLint + Prettier; optional style guide
**Rationale:** Consistent code style, reduces review friction, improves maintainability.

### Node & tooling
**Recommendation:** Pin Node.js version (e.g., 18.x) and use package-lock.json for deterministic installs
**Rationale:** Stability across environments; reproducible builds.

## Technical Risks

### Test suite is too small to justify CI overhead or lacks coverage impact.
**Probability:** Medium
**Impact:** Low to Medium
**Mitigation:** Start with small, deterministic tests tied to concrete utilities; expand gradually; track coverage growth to justify CI investment.

### Flaky tests due to DOM/mocking inconsistencies or race conditions.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Use stable jsdom environment, deterministic mocks, and avoid real network calls in unit tests; isolate tests from time-based dependencies.

### Build script tests may cause CI time to grow significantly if tests are broad.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Prioritize unit tests for critical build steps; run incremental tests; enable caching and parallelization in CI.

### Repository has little to no JavaScript logic to test, leading to wasted effort.
**Probability:** Low
**Impact:** Low
**Mitigation:** Perform an upfront quick assessment; adjust scope to test build tooling, configuration readers, or data parsing logic if present.

## Performance Considerations

**Scalability:** Static site remains highly scalable; test suite should be lightweight and fast to avoid CI delays.

**Performance Targets:** Unit tests should complete within 5–10 minutes in CI for a small repo; aim for sub-5 seconds per individual test file locally.

**Optimization Strategies:** Run tests in parallel; enable caching of node_modules; instrument selective tests via testPathPattern if needed,Mock external calls; avoid network in unit tests

## Security Considerations

**Threat Model:** Supply chain risks from third-party npm packages; potential misconfigurations in CI.

**Security Controls:** Use npm ci for deterministic installs,Lock dependencies (package-lock.json) and audit on CI,Run npm audit and fail on critical vulnerabilities

**Compliance:** If data/assets contain sensitive information, ensure tests do not leak or rely on real data; avoid embedding secrets in tests

## Development Strategy

**Methodology:** Incremental, test-driven improvement with lightweight gating; maintainable growth of test suite.

**Testing Strategy:** Test pyramid: many small unit tests for utilities, a smaller set for integration-like behavior of build scripts; limited E2E if required later.

**Deployment Strategy:** CI runs on PRs and on push to main; ensure tests pass before merge; document test outcomes in PR notes

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