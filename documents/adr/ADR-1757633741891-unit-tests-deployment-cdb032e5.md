# Architectural Decision Record: I’ll start by clarifying the business goals and scope so we can capture the right requirements before creating any issues. Two clarifying questions:
1) What business outcomes do you want to achieve by adding unit tests (e.g., fewer bugs in production, faster turnaround on changes, more reliable releases, easier onboarding for contributors)? 
2) Which parts of the mlongerich.github.io project should be covered by unit tests (e.g., core JS utilities, front-end components, data fetching logic, build or deployment scripts, or something else)?

**Date:** 2025-09-11
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

The repository, likely a static GitHub Pages site, benefits from a lightweight unit testing strategy focused on front-end JavaScript utilities and any data-fetching helpers. Introduce a minimal test harness (Vitest + Vite), add CI integration, and progressively expand test coverage while preserving the static hosting model. Prioritize modularization of JavaScript to enable effective testing and maintainability. Align testing scope with business goals by confirming from the BA which components are critical to test and ensuring that unit tests deliver tangible improvements in reliability, onboarding, and maintenance.

## Architecture Decisions

### Maintain a static GitHub Pages architecture with optional Jekyll-based workflow for content and a minimal client-side JavaScript layer.

**Rationale:** Most mlongerich.github.io projects on GitHub Pages are static sites. This minimizes operational overhead, hosting risk, and aligns with typical GitHub Actions CI for static sites. If there are posts and content-driven pages, Jekyll is a reasonable standard path that GitHub Pages supports out of the box. Introducing a heavy SSR or serverless backend would add maintenance burden for a personal site and is unnecessary unless new dynamic requirements are introduced.

**Impact:** Keeps deployment simple; ensures compatibility with GitHub Pages. If unit tests are introduced, they will live in a dev tooling layer and not affect the published static content.

**Risks:** Limited dynamic capabilities may constrain testing of client-side interactions if the site becomes more interactive. Mitigation: isolate interactive JS logic into modular utilities that can be unit-tested without requiring a backend.

## Implementation Plan

### Overview
Introduce a lightweight unit testing strategy for front-end JavaScript utilities, establish a minimal test harness, and integrate with CI for automated validation. Preserve the current static hosting model while enabling test-driven improvements to JS utilities and front-end components (if any).

### Critical Path
Phase 1 deliverables enabling Phase 2 work, followed by Phase 3 CI integration. The critical path is defined by establishing a test framework compatible with the existing code structure and ensuring tests can run in CI without requiring a live web server.

## Technology Recommendations

### Testing Framework
**Recommendation:** Vitest + Vite for unit testing of JavaScript utilities.
**Rationale:** Lightweight, fast, supports ES modules, works well with modern JS, and easy to integrate with GitHub Actions. Works well for static sites without requiring a full bundler for production assets.

### Build & Tooling
**Recommendation:** Vite as a dev server/build tool, with optional ES module adoption for utilities.
**Rationale:** Fast dev experience and easy integration with Vitest. Keeps build steps lightweight for a static site.

### Linting & Formatting
**Recommendation:** ESLint + Prettier for code quality and consistency.
**Rationale:** Helps maintainability as tests are added and code evolves.

### TypeScript (optional but beneficial)
**Recommendation:** Introduce TypeScript for new JS utilities or gradually migrate existing utilities to TS.
**Rationale:** Improves maintainability and testability by enforcing types, enabling better unit test reliability.

## Technical Risks

### Insufficient modularization of JS to enable effective unit testing within a static-site project.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Incrementally refactor scripts into ES modules with pure functions and dependency injection. Start with non-UI utilities.

### CI environment constraints for a static site (no server-side execution, limited tooling in GH Actions for certain runtimes).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Choose lightweight test frameworks (e.g., Vitest) and ensure node_modules are cached. Use actions/setup-node to specify Node version.

### Potential performance impact from bundling or test tooling in the repository.
**Probability:** Low
**Impact:** Low
**Mitigation:** Keep devDependencies isolated from production assets; ensure test/build steps are not invoked during normal site deployment.

## Performance Considerations

**Scalability:** As a static site, horizontal scalability is not a concern for hosting. Focus on build-time performance and test-efficiency as codebase grows.

**Performance Targets:** O(100–200ms dynamic content bootstrapping per page on average devices for any added client-side logic; ensure assets load efficiently.

**Optimization Strategies:** Isolate test tooling to development dependencies; avoid bundling test code into production.,Code-split utilities and use tree-shaking to minimize production JS.,Leverage CDN for static assets and image optimization.,Prefer lazy loading for non-critical resources.

## Security Considerations

**Threat Model:** Minimal attack surface in a static site. Potential risks stem from third-party dependencies in dev tooling and any dynamic fetches wired into front-end code.

**Security Controls:** Lock down dependencies to known-good versions; use lockfile (package-lock.json/yarn.lock).,Regularly update dev dependencies; run security audit (npm audit, etc.).,If any data fetching is introduced, implement proper CORS handling and avoid exposing secrets in client code.

**Compliance:** No sensitive data processing on the client. Ensure any analytics or third-party scripts comply with privacy requirements.

## Development Strategy

**Methodology:** Agile-like incremental improvements with a focus on small, testable increments. Begin with unit tests for utilities, then expand.

**Testing Strategy:** Unit tests for pure functions first; mock network calls for data-fetching helpers; expand to UI interaction tests only if needed.

**Deployment Strategy:** Continue using GitHub Pages for production; ensure CI does not publish test artifacts to production. Optionally host test results or artifacts in GH Pages or as artifacts in CI.

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