# Architectural Decision Record: 1) Which parts of the site do you want covered by unit tests? For example, core JavaScript utilities, UI components, data fetching/processing, or build-related scripts. Are there any features you want to prioritize for testing?

2) What does success look like from a business perspective? Do you have target metrics (e.g., code coverage goals, reduction in post-release bugs, faster feedback, or a deadline to meet)? Any constraints (time, resources) we should consider as we scope this?

**Date:** 2025-09-12
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis recommends preserving a Jekyll-based GitHub Pages hosting path while introducing a lightweight JavaScript unit testing layer (Jest + DOM tests) and a GitHub Actions-driven CI/CD pipeline. Key goals are to enable test coverage for core utilities, establish quality gates before merges, and implement performance and accessibility checks to improve user experience. The plan prioritizes minimal hosting disruption, gradual test expansion, and automated feedback to support faster, safer site updates. The next steps involve aligning on test coverage targets, setting up the test harness, and implementing the initial CI workflow with phased rollout.

## Architecture Decisions

### Maintain a Jekyll-based GitHub Pages site with an optional Node tooling path for local builds and testing.

**Rationale:** GitHub Pages natively supports Jekyll and Ruby-based builds, aligning with typical user/organization pages. This minimizes hosting friction and ensures smooth auto-publishing. Introducing optional Node tooling provides flexibility for modern testing and build pipelines without forcing a full rebuild of the hosting strategy.

**Impact:** Low to moderate disruption; enables test tooling and CI while preserving existing hosting guarantees. If moved to a different generator, hosting strategy and workflows will need redesign.

**Risks:** Ruby version drift on local/dev vs CI; potential friction if the repo relies on Jekyll plugins unavailable in GitHub Pages environment.

### Introduce a lightweight JavaScript unit test layer focused on core utilities with DOM-based tests where applicable.

**Rationale:** Static sites often ship small utility scripts. Testing core utilities improves reliability without requiring a full frontend framework. DOM tests cover simple UI-related decisions (e.g., DOM manipulation utilities, template helpers).

**Impact:** Adds a maintainable safety net for JS logic and DOM helpers; accelerates regression detection during site changes.

**Risks:** If utilities are minimal, test maintenance overhead may exceed value. Ensure tests stay small and focused.

### Set up a CI/CD workflow using GitHub Actions to build the Jekyll site, execute tests, perform linting, run accessibility/a11y checks, and publish to GitHub Pages.

**Rationale:** Automated validation before merge reduces post-deploy bugs, standardizes quality gates, and enables rapid feedback loops.

**Impact:** Improved quality control; requires maintenance of workflow definitions and environment setup (Ruby, Node).

**Risks:** Workflow failures due to environment changes; potential delays in PR feedback if tests are flaky. Mitigation: pin tool versions, implement retry logic, and provide clear test failure diagnostics.

### Adopt performance and accessibility tooling (Lighthouse/axe-core) with lightweight image optimization and modern asset handling.

**Rationale:** Static sites benefit from fast performance and good accessibility metrics; automated checks help maintain standards across changes.

**Impact:** Higher baseline quality and user experience; incremental gains with small changes (lazy loading, asset optimization).

**Risks:** Overhead in configuring and maintaining tooling; ensure it runs efficiently on PRs without causing long feedback loops.

## Implementation Plan

### Overview
A phased plan to introduce unit tests, CI/CD, and quality tooling while preserving existing GitHub Pages hosting. Emphasizes incremental adoption with clear deliverables per phase.

### Critical Path
Key dependencies are the Phase 1 test harness stability, Phase 4 CI/CD workflow creation, and Phase 2 core utilities test coverage. Delays in establishing tests will stall downstream UI tests and CI feedback loops.

## Technology Recommendations

### Testing framework
**Recommendation:** Jest with @testing-library/dom for unit and DOM tests; optionally Playwright for end-to-end through PR previews.
**Rationale:** Lightweight, well-supported, integrates easily with Node-based tooling and GitHub Actions. DOM Testing Library helps write resilient DOM tests for static sites.

### Linting & quality
**Recommendation:** ESLint for JS, Stylelint for CSS, Prettier for formatting.
**Rationale:** Ensures consistent code quality across tests and site scripts; reduces drift.

### CI/CD & hosting
**Recommendation:** GitHub Actions with a workflow to build Jekyll (Ruby), run Node tests, and publish to gh-pages.
**Rationale:** Seamless integration with GitHub hosting, reproducible builds, and automatic feedback.

### Accessibility & performance tooling
**Recommendation:** Lighthouse (audits) and axe-core for automated a11y checks; image optimization via built-in or webpack tooling as needed.
**Rationale:** Automates essential QA gates and directs performance improvements.

### Security & dependencies
**Recommendation:** Dependabot (or Snyk) for dependency scanning; npm audit/gem audit in CI.
**Rationale:** Proactive vulnerability management in dependencies.

## Technical Risks

### Ruby/Jekyll environment drift between local development and GitHub Pages CI
**Probability:** Medium
**Impact:** High (build failures or mismatched output)
**Mitigation:** Lock Ruby version via .ruby-version, pin Bundler and Jekyll versions in Gemfile, and use CI to install exact versions. Consider using docker-based local dev if needed.

### Flaky or high-maintenance tests for DOM-heavy utilities
**Probability:** Medium
**Impact:** Medium to High (false positives/negatives, flaky PR checks)
**Mitigation:** Keep tests small, deterministic, and isolated. Mock browser APIs where possible. Establish a minimum test stability criterion (no tests flaky > X%).

### Owner time/capacity constraints for maintaining test suites and CI workflows
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Automate as much as possible, start with a small, high-value test set, and gradually expand. Schedule quarterly maintenance sprints and assign ownership.

### Dependency vulnerabilities in npm/gem ecosystems
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Enable Dependabot/audit tooling, pin versions, run npm audit and gem audit in CI. Address critical vulnerabilities promptly.

### Performance regressions on PRs due to asset changes
**Probability:** Low to Medium
**Impact:** Medium
**Mitigation:** Introduce baseline performance checks and enforce caching/minification; use Lighthouse thresholds in CI.

## Performance Considerations

**Scalability:** As a static site, horizontal scaling is not a concern; focus on build-time performance and lean runtime.

**Performance Targets:** Target fastest possible Lighthouse scores for desktop/mobile (e.g., Performance > 90, Accessibility > 95, Best Practices > 90, SEO > 90).

**Optimization Strategies:** Image optimization (compression, modern formats like WebP/AVIF).,Lazy loading for off-screen images and iframes.,CSS critical path: inline critical CSS, defer non-critical CSS.,Minify and compress JS/CSS; enable HTTP caching via proper headers (via GitHub Pages configuration or CDN if used).

## Security Considerations

**Threat Model:** As a static site, threats include supply-chain risks in dependencies, misconfigurations exposing content, and potential XSS via user-provided content in templates.

**Security Controls:** Lock dependencies to specific versions; enable Dependabot/Snyk scans.,Avoid eval-like patterns in JS; sanitize any user-provided content in templates.,Use a Content Security Policy (CSP) header in site meta or via hosting config where supported.,Limit inline scripts; prefer external, integrity-protected scripts when possible.

**Compliance:** If personal data is shown, ensure privacy considerations; otherwise standard public site typically falls under general web compliance.

## Development Strategy

**Methodology:** Agile-inspired with incremental delivery and clear quality gates; PR-driven with feature flags where applicable.

**Testing Strategy:** Unit tests for core utilities; DOM/template tests for critical pages; E2E tests if needed for critical flows; automated accessibility checks; performance baselines.

**Deployment Strategy:** Publish to GitHub Pages via a dedicated gh-pages workflow; ensure builds are reproducible; keep a changelog and documentation.

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