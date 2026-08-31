# Architectural Decision Record: Great — I can help with adding unit tests. I have a couple of clarifying questions to scope this properly:

1) Which parts of the repository should have unit tests? For example, are you targeting JavaScript utilities, frontend components, templates, or any Ruby/Jekyll plugins if this is a GitHub Pages site?

2) What language and test framework would you like to use (or prefer)? For instance:
- If the site uses JavaScript: Jest or Vitest
- If it’s a Jekyll/Ruby setup: RSpec
- If there are other languages/tools involved: please specify

Also, do you want tests to run locally, in CI (e.g., GitHub Actions), or both?

**Date:** 2025-09-11
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

Cannot access the repository structure due to a missing path parameter in the provided context. Based on a typical mlongerich.github.io GitHub Pages setup (likely Jekyll with optional JavaScript), the recommended approach is a phased, dual-layer unit testing plan covering Ruby/Jekyll components and JavaScript assets, with optional Liquid/template tests. Implement CI (GitHub Actions) to run Ruby and JS tests locally and in CI, and document the setup for maintainers. The BA questions will help tailor scope, framework choice, and CI requirements. Please provide the repository path (or confirm the typical Jekyll/GHP structure) to tailor the plan precisely and to populate exact test cases and module boundaries.

## Architecture Decisions

### Adopt a dual-layer unit testing strategy targeting both Ruby/Jekyll components and any JavaScript frontend assets, with optional Liquid/template testing for GitHub Pages.

**Rationale:** GitHub Pages sites commonly use Jekyll (Ruby) and may include small JS utilities. Testing should cover: (a) Ruby code such as plugins, helpers, and configuration logic; (b) frontend JavaScript logic used on pages; (c) template rendering (Liquid) to guard against regressions in page layout. This approach minimizes risk of changes breaking site generation or UI while remaining aligned with typical tech stacks in GitHub Pages deployments.

**Impact:** Provides early regression detection, improves maintainability, and enables safe CI-driven delivery for the static site.

**Risks:** If the repo has no Ruby components or JS is minimal, effort may be disproportionately allocated to testing; risk mitigated by scoping tests to actual code.

## Implementation Plan

### Overview
Establish a test ecosystem for a typical Jekyll-based GitHub Pages site, then progressively expand coverage to JS utilities and template rendering as needed. Integrate into CI (GitHub Actions) and provide local scripts for developers.

### Critical Path
Phase 2 (Environment Setup) -> Phase 3 (Ruby tests) and Phase 4 (JS tests) + Phase 6 (CI/docs) to enable automated verification on PRs. Phase 5 (Liquid tests) is optional but recommended for template integrity.

## Technology Recommendations

### Ruby/Jekyll
**Recommendation:** Ruby 3.x (e.g., 3.1 or 3.2) with Jekyll 4.x; Bundler for dependency management; RSpec for unit testing.
**Rationale:** Modern Ruby versions ensure performance, security, and compatibility with current Jekyll; RSpec is a mature, expressive testing framework with wide ecosystem.

### JavaScript
**Recommendation:** Vitest (preferred) or Jest for JS testing
**Rationale:** Vitest is fast, modern, integrates well with Vite-based setups; Jest remains widely adopted with strong ecosystem.

### CI/CD
**Recommendation:** GitHub Actions with matrix for Ruby and Node versions; caching for Bundler and Node modules
**Rationale:** Native to GitHub Pages workflow, easy to maintain, integrates with PR checks.

### Template Testing
**Recommendation:** Ruby-based Liquid rendering tests harness (RSpec with a small rendering helper) or a lightweight Ruby script
**Rationale:** Keeps tests close to the runtime environment of Jekyll.

## Technical Risks

### Inconsistent Ruby/Jekyll environment across local development and CI (Ruby version, bundler, plugin compatibility).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Pin Ruby version in .ruby-version; lock Bundler version; include a minimal Gemfile.lock; use Dockerized test runner or GitHub Actions matrix to ensure consistency.

### Testing Liquid templates is non-trivial and brittle across Jekyll versions.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Target key templates with predictable data; use a harness that renders templates with a controlled context; keep templates isolated from dynamic content in tests.

### Flaky tests due to CI timing or environment differences (file system, network, locale).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Make tests deterministic; avoid network access in unit tests; use CI caches; run tests in isolated steps.

### Maintenance burden for keeping Ruby/Jekyll, Node, and test tooling in sync with GitHub Pages constraints.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Prefer widely supported versions; avoid deprecated plugins; communicate a quarterly review cycle for dependencies.

## Performance Considerations

**Scalability:** As a static site, primary considerations are build time and CI duration. Caching and incremental builds help.

**Performance Targets:** CI build time under 5–10 minutes for full test suite; local test runs under 2–3 minutes for focused tests.

**Optimization Strategies:** Use Bundler cache and RubyGems cache in CI,Split test suites so that non-related tests can run in parallel,Limit template tests to essential templates

## Security Considerations

**Threat Model:** Static site with build steps; potential issues include dependency vulnerabilities, remote plugin risks, and misconfigured content.

**Security Controls:** Lock gem versions (Gemfile.lock) and audit dependencies regularly,Avoid remote plugins; prefer built-in or vendored functionality,Run simple static analysis and linting where feasible

**Compliance:** No sensitive data in tests; ensure test fixtures do not leak credentials

## Development Strategy

**Methodology:** Iterative, test-driven where possible; prioritize essential Ruby tests first, then JS and templates

**Testing Strategy:** Unit tests for logic; lightweight tests for templates; unit tests for JS utilities; gradually add integration tests if needed

**Deployment Strategy:** Automated CI runs on PRs; automated site build with Jekyll on main; deployment to GitHub Pages remains unaffected by tests

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